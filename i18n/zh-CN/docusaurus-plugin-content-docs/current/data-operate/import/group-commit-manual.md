---
{
    "title": "高并发导入优化（Group Commit）",
    "language": "zh-CN"
}
---

在高频小批量写入场景下，传统的导入方式存在以下问题：

- 每个导入都会创建一个独立的事务，都需要经过 FE 解析 SQL 和生成执行计划，影响整体性能
- 每个导入都会生成一个新的版本，导致版本数快速增长，增加了后台 compaction 的压力

为了解决这些问题，Doris 引入了 Group Commit 机制。Group Commit 不是一种新的导入方式，而是对现有导入方式的优化扩展，主要针对：

- `INSERT INTO tbl VALUES(...)` 语句
- Stream Load 导入

通过将多个小批量导入在后台合并成一个大的事务提交，显著提升了高并发小批量写入的性能。同时，Group Commit 与 PreparedStatement 结合使用可以获得更高的性能提升。

## Group Commit 模式

Group Commit 写入有三种模式，分别是：

* 关闭模式（`off_mode`）

    不开启 Group Commit。

* 同步模式（`sync_mode`）

    Doris 根据负载和表的 `group_commit_interval`属性将多个导入在一个事务提交，事务提交后导入返回。这适用于高并发写入场景，且在导入完成后要求数据立即可见。

* 异步模式（`async_mode`）

    Doris 首先将数据写入 WAL (`Write Ahead Log`)，然后导入立即返回。Doris 会根据负载和表的`group_commit_interval`属性异步提交数据，提交之后数据可见。为了防止 WAL 占用较大的磁盘空间，单次导入数据量较大时，会自动切换为`sync_mode`。这适用于写入延迟敏感以及高频写入的场景。

    WAL 的数量可以通过 FE http 接口查看，具体可见[这里](../../admin-manual/open-api/fe-http/get-wal-size-action)，也可以在 BE 的 metrics 中搜索关键词`wal`查看。

## Group Commit 使用方式

假如表的结构为：
```sql
CREATE TABLE `dt` (
    `id` int(11) NOT NULL,
    `name` varchar(50) NULL,
    `score` int(11) NULL
) ENGINE=OLAP
DUPLICATE KEY(`id`)
DISTRIBUTED BY HASH(`id`) BUCKETS 1
PROPERTIES (
    "replication_num" = "1"
);
```

### 使用 JDBC

当用户使用 JDBC `insert into values`方式写入时，为了减少 SQL 解析和生成规划的开销，我们在 FE 端支持了 MySQL 协议的 `PreparedStatement` 特性。当使用 `PreparedStatement` 时，SQL 和其导入规划将被缓存到 Session 级别的内存缓存中，后续的导入直接使用缓存对象，降低了 FE 的 CPU 压力。下面是在 JDBC 中使用 `PreparedStatement` 的例子：

**1. 设置 JDBC URL 并在 Server 端开启 Prepared Statement**

```
url = jdbc:mysql://127.0.0.1:9030/db?useServerPrepStmts=true&useLocalSessionState=true&rewriteBatchedStatements=true&cachePrepStmts=true&prepStmtCacheSqlLimit=99999&prepStmtCacheSize=500
```

**2. 配置 `group_commit` session 变量，有如下两种方式：**

* 通过 JDBC url 设置，增加`sessionVariables=group_commit=async_mode`

```
url = jdbc:mysql://127.0.0.1:9030/db?useServerPrepStmts=true&useLocalSessionState=true&rewriteBatchedStatements=true&cachePrepStmts=true&prepStmtCacheSqlLimit=99999&prepStmtCacheSize=500&sessionVariables=group_commit=async_mode,enable_nereids_planner=false
```

* 通过执行 SQL 设置

```
try (Statement statement = conn.createStatement()) {
    statement.execute("SET group_commit = async_mode;");
}
```

**3. 使用 `PreparedStatement`**

```java
private static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";
private static final String URL_PATTERN = "jdbc:mysql://%s:%d/%s?useServerPrepStmts=true&useLocalSessionState=true&rewriteBatchedStatements=true&cachePrepStmts=true&prepStmtCacheSqlLimit=99999&prepStmtCacheSize=50&sessionVariables=group_commit=async_mode";
private static final String HOST = "127.0.0.1";
private static final int PORT = 9087;
private static final String DB = "db";
private static final String TBL = "dt";
private static final String USER = "root";
private static final String PASSWD = "";
private static final int INSERT_BATCH_SIZE = 10;

private static void groupCommitInsertBatch() throws Exception {
    Class.forName(JDBC_DRIVER);
    // add rewriteBatchedStatements=true and cachePrepStmts=true in JDBC url
    // set session variables by sessionVariables=group_commit=async_mode in JDBC url
    try (Connection conn = DriverManager.getConnection(
            String.format(URL_PATTERN, HOST, PORT, DB), USER, PASSWD)) {

        String query = "insert into " + TBL + " values(?, ?, ?)";
        try (PreparedStatement stmt = conn.prepareStatement(query)) {
            for (int j = 0; j < 5; j++) {
                // 10 rows per insert
                for (int i = 0; i < INSERT_BATCH_SIZE; i++) {
                    stmt.setInt(1, i);
                    stmt.setString(2, "name" + i);
                    stmt.setInt(3, i + 10);
                    stmt.addBatch();
                }
                int[] result = stmt.executeBatch();
            }
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

注意：由于高频的 insert into 语句会打印大量的 audit log，对最终性能有一定影响，默认关闭了打印 prepared 语句的 audit log。可以通过设置 session variable 的方式控制是否打印 prepared 语句的 audit log。

```sql
# 配置 session 变量开启打印parpared语句的audit log, 默认为false即关闭打印parpared语句的audit log。
set enable_prepared_stmt_audit_log=true;
```

关于 **JDBC** 的更多用法，参考[使用 Insert 方式同步数据](./import-way/insert-into-manual.md)。

### 使用 Golang 进行 Group Commit

Golang 的 prepared 语句支持有限，所以我们可以通过手动客户端攒批的方式提高 Group Commit 的性能，以下为一个示例程序。

```Golang
package main

import (
	"database/sql"
	"fmt"
	"math/rand"
	"strings"
	"sync"
	"sync/atomic"
	"time"

	_ "github.com/go-sql-driver/mysql"
)

const (
	host     = "127.0.0.1"
	port     = 9038
	db       = "test"
	user     = "root"
	password = ""
	table    = "async_lineitem"
)

var (
	threadCount = 20
	batchSize   = 100
)

var totalInsertedRows int64
var rowsInsertedLastSecond int64

func main() {
	dbDSN := fmt.Sprintf("%s:%s@tcp(%s:%d)/%s?parseTime=true", user, password, host, port, db)
	db, err := sql.Open("mysql", dbDSN)
	if err != nil {
		fmt.Printf("Error opening database: %s\n", err)
		return
	}
	defer db.Close()

	var wg sync.WaitGroup
	for i := 0; i < threadCount; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			groupCommitInsertBatch(db)
		}()
	}

	go logInsertStatistics()

	wg.Wait()
}

func groupCommitInsertBatch(db *sql.DB) {
	for {
		valueStrings := make([]string, 0, batchSize)
		valueArgs := make([]interface{}, 0, batchSize*16)
		for i := 0; i < batchSize; i++ {
		    valueStrings = append(valueStrings, "(?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)")
			valueArgs = append(valueArgs, rand.Intn(1000))
			valueArgs = append(valueArgs, rand.Intn(1000))
			valueArgs = append(valueArgs, rand.Intn(1000))
			valueArgs = append(valueArgs, rand.Intn(1000))
			valueArgs = append(valueArgs, sql.NullFloat64{Float64: 1.0, Valid: true})
			valueArgs = append(valueArgs, sql.NullFloat64{Float64: 1.0, Valid: true})
			valueArgs = append(valueArgs, sql.NullFloat64{Float64: 1.0, Valid: true})
			valueArgs = append(valueArgs, sql.NullFloat64{Float64: 1.0, Valid: true})
			valueArgs = append(valueArgs, "N")
			valueArgs = append(valueArgs, "O")
			valueArgs = append(valueArgs, time.Now())
			valueArgs = append(valueArgs, time.Now())
			valueArgs = append(valueArgs, time.Now())
			valueArgs = append(valueArgs, "DELIVER IN PERSON")
			valueArgs = append(valueArgs, "SHIP")
			valueArgs = append(valueArgs, "N/A")
		}
		stmt := fmt.Sprintf("INSERT INTO %s VALUES %s",
			table, strings.Join(valueStrings, ","))
		_, err := db.Exec(stmt, valueArgs...)
		if err != nil {
			fmt.Printf("Error executing batch: %s\n", err)
			return
		}
		atomic.AddInt64(&rowsInsertedLastSecond, int64(batchSize))
		atomic.AddInt64(&totalInsertedRows, int64(batchSize))
	}
}

func logInsertStatistics() {
	for {
		time.Sleep(1 * time.Second)
		fmt.Printf("Total inserted rows: %d\n", totalInsertedRows)
		fmt.Printf("Rows inserted in the last second: %d\n", rowsInsertedLastSecond)
		rowsInsertedLastSecond = 0
	}
}

```

### INSERT INTO VALUES

* 异步模式

```sql
# 配置 session 变量开启 group commit (默认为 off_mode),开启异步模式
mysql> set group_commit = async_mode;

# 这里返回的 label 是 group_commit 开头的，可以区分出是否使用了 group commit
mysql> insert into dt values(1, 'Bob', 90), (2, 'Alice', 99);
Query OK, 2 rows affected (0.05 sec)
{'label':'group_commit_a145ce07f1c972fc-bd2c54597052a9ad', 'status':'PREPARE', 'txnId':'181508'}

# 可以看出这个 label, txn_id 和上一个相同，说明是攒到了同一个导入任务中
mysql> insert into dt(id, name) values(3, 'John');
Query OK, 1 row affected (0.01 sec)
{'label':'group_commit_a145ce07f1c972fc-bd2c54597052a9ad', 'status':'PREPARE', 'txnId':'181508'}

# 不能立刻查询到
mysql> select * from dt;
Empty set (0.01 sec)

# 10 秒后可以查询到，可以通过表属性 group_commit_interval 控制数据可见延迟。
mysql> select * from dt;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Bob   |    90 |
|    2 | Alice |    99 |
|    3 | John  |  NULL |
+------+-------+-------+
3 rows in set (0.02 sec)
```

* 同步模式

```sql
# 配置 session 变量开启 group commit (默认为 off_mode),开启同步模式
mysql> set group_commit = sync_mode;

# 这里返回的 label 是 group_commit 开头的，可以区分出是否使用了 group commit，导入耗时至少是表属性 group_commit_interval。
mysql> insert into dt values(4, 'Bob', 90), (5, 'Alice', 99);
Query OK, 2 rows affected (10.06 sec)
{'label':'group_commit_d84ab96c09b60587_ec455a33cb0e9e87', 'status':'PREPARE', 'txnId':'3007', 'query_id':'fc6b94085d704a94-a69bfc9a202e66e2'}

# 数据可以立刻读出
mysql> select * from dt;
+------+-------+-------+
| id   | name  | score |
+------+-------+-------+
|    1 | Bob   |    90 |
|    2 | Alice |    99 |
|    3 | John  |  NULL |
|    4 | Bob   |    90 |
|    5 | Alice |    99 |
+------+-------+-------+
5 rows in set (0.03 sec)
```

* 关闭模式

```sql
mysql> set group_commit = off_mode;
```

### Stream Load

假如`data.csv`的内容为：

```sql
6,Amy,60
7,Ross,98
```

* 异步模式

```sql
# 导入时在 header 中增加"group_commit:async_mode"配置

curl --location-trusted -u {user}:{passwd} -T data.csv -H "group_commit:async_mode"  -H "column_separator:,"  http://{fe_host}:{http_port}/api/db/dt/_stream_load
{
    "TxnId": 7009,
    "Label": "group_commit_c84d2099208436ab_96e33fda01eddba8",
    "Comment": "",
    "GroupCommit": true,
    "Status": "Success",
    "Message": "OK",
    "NumberTotalRows": 2,
    "NumberLoadedRows": 2,
    "NumberFilteredRows": 0,
    "NumberUnselectedRows": 0,
    "LoadBytes": 19,
    "LoadTimeMs": 35,
    "StreamLoadPutTimeMs": 5,
    "ReadDataTimeMs": 0,
    "WriteDataTimeMs": 26
}

# 返回的 GroupCommit 为 true，说明进入了 group commit 的流程
# 返回的 Label 是 group_commit 开头的，是真正消费数据的导入关联的 label
```

* 同步模式

```sql
# 导入时在 header 中增加"group_commit:sync_mode"配置

curl --location-trusted -u {user}:{passwd} -T data.csv -H "group_commit:sync_mode"  -H "column_separator:,"  http://{fe_host}:{http_port}/api/db/dt/_stream_load
{
    "TxnId": 3009,
    "Label": "group_commit_d941bf17f6efcc80_ccf4afdde9881293",
    "Comment": "",
    "GroupCommit": true,
    "Status": "Success",
    "Message": "OK",
    "NumberTotalRows": 2,
    "NumberLoadedRows": 2,
    "NumberFilteredRows": 0,
    "NumberUnselectedRows": 0,
    "LoadBytes": 19,
    "LoadTimeMs": 10044,
    "StreamLoadPutTimeMs": 4,
    "ReadDataTimeMs": 0,
    "WriteDataTimeMs": 10038
}

# 返回的 GroupCommit 为 true，说明进入了 group commit 的流程
# 返回的 Label 是 group_commit 开头的，是真正消费数据的导入关联的 label
```

关于 Stream Load 使用的更多详细语法及最佳实践，请参阅 [Stream Load](./import-way/stream-load-manual)。


## 自动提交条件

当满足时间间隔 (默认为 10 秒) 或数据量 (默认为 64 MB) 其中一个条件时，会自动提交数据。这两个参数需要配合使用，建议根据实际场景进行调优。

### 修改提交间隔

默认提交间隔为 10 秒，用户可以通过修改表的配置调整：

```sql
# 修改提交间隔为 2 秒
ALTER TABLE dt SET ("group_commit_interval_ms" = "2000");
```

**参数调整建议**:
- 较短的间隔 (如 2 秒):
  - 优点：数据可见性延迟更低，适合对实时性要求较高的场景
  - 缺点：提交次数增多，版本数增长更快，后台 compaction 压力更大

- 较长的间隔 (如 30 秒):
  - 优点：提交批次更大，版本数增长更慢，系统开销更小
  - 缺点：数据可见性延迟更高

建议根据业务对数据可见性延迟的容忍度来设置，如果系统压力大，可以适当增加间隔。

### 修改提交数据量

Group Commit 的默认提交数据量为 64 MB，用户可以通过修改表的配置调整：

```sql
# 修改提交数据量为 128MB
ALTER TABLE dt SET ("group_commit_data_bytes" = "134217728");
```

**参数调整建议**:
- 较小的阈值 (如 32MB):
  - 优点：内存占用更少，适合资源受限的环境
  - 缺点：提交批次较小，吞吐量可能受限

- 较大的阈值 (如 256MB):
  - 优点：批量提交效率更高，系统吞吐量更大
  - 缺点：占用更多内存

建议根据系统内存资源和数据可靠性要求来权衡。如果内存充足且追求更高吞吐，可以适当增加到 128MB 或更大。


## 相关系统配置

### BE 配置

1. `group_commit_wal_path`

   * 描述：group commit 存放 WAL 文件的目录

   * 默认值：默认在用户配置的`storage_root_path`的各个目录下创建一个名为`wal`的目录。配置示例：
  
   ```
   group_commit_wal_path=/data1/storage/wal;/data2/storage/wal;/data3/storage/wal
   ```

## 使用限制

* **Group Commit 限制条件**

  * `INSERT INTO VALUES` 语句在以下情况下会退化为非 Group Commit 方式：
    - 使用事务写入 (`Begin; INSERT INTO VALUES; COMMIT`)
    - 指定 Label (`INSERT INTO dt WITH LABEL {label} VALUES`)
    - VALUES 中包含表达式 (`INSERT INTO dt VALUES (1 + 100)`)
    - 列更新写入
    - 表不支持轻量级模式更改

  * `Stream Load` 在以下情况下会退化为非 Group Commit 方式：
    - 使用两阶段提交
    - 指定 Label (`-H "label:my_label"`)
    - 列更新写入
    - 表不支持轻量级模式更改

* **Unique 模型**
  - Group Commit 不保证提交顺序，建议使用 Sequence 列来保证数据一致性。

* **WAL 限制**
  - `async_mode` 写入会将数据写入 WAL，成功后删除，失败时通过 WAL 恢复。
  - WAL 文件是单副本存储的，如果对应磁盘损坏或文件误删可能导致数据丢失。
  - 下线 BE 节点时，使用 `DECOMMISSION` 命令以防数据丢失。
  - `async_mode` 在以下情况下切换为 `sync_mode`：
    - 导入数据量过大（超过 WAL 单目录 80% 空间）
    - 不知道数据量的 chunked stream load
    - 磁盘可用空间不足
  - 重量级 Schema Change 时，拒绝 Group Commit 写入，客户端需重试。

## 性能

我们分别测试了使用`Stream Load`和`JDBC`在高并发小数据量场景下`group commit`(使用`async mode`) 的写入性能。

### Stream Load 日志场景测试

#### 机器配置

* 1 台 FE：阿里云 8 核 CPU、16GB 内存、1 块 100GB ESSD PL1 云磁盘

* 3 台 BE：阿里云 16 核 CPU、64GB 内存、1 块 1TB ESSD PL1 云磁盘

* 1 台测试客户端：阿里云 16 核 CPU、64GB 内存、1 块 100GB ESSD PL1 云磁盘

* 测试版本为 Doris-3.0.1

#### 数据集

* `httplogs` 数据集，总共 31GB、2.47 亿条

#### 测试工具

* [doris-streamloader](/ecosystem/doris-streamloader.md)

#### 测试方法

* 对比 `非 group_commit` 和 `group_commit `的 `async_mode` 模式下，设置不同的单并发数据量和并发数，导入 `247249096` 行数据

#### 测试结果

| 导入方式    | 单并发数据量  | 并发数  | 耗时 (秒)     | 导入速率 (行/秒) | 导入吞吐 (MB/秒) |
|----------------|---------|------|-----------|----------|-----------|
| `group_commit` | 10 KB   | 10   | 2204      | 112,181   | 14.8 |
| `group_commit` | 10 KB   | 30   | 2176      | 113,625   | 15.0 |
| `group_commit` | 100 KB  | 10   | 283       | 873,671  | 115.1 |
| `group_commit` | 100 KB  | 30   | 244       | 1,013,315  | 133.5 |
| `group_commit` | 500 KB  | 10   | 125       | 1,977,992  | 260.6 |
| `group_commit` | 500 KB  | 30   | 122       | 2,026,631  | 267.1 |
| `group_commit` | 1 MB    | 10   | 119       | 2,077,723  | 273.8 |
| `group_commit` | 1 MB    | 30   | 119       | 2,077,723  | 273.8 |
| `group_commit` | 10 MB   | 10   | 118       | 2,095,331  | 276.1 |
| `非group_commit` | 1 MB    | 10   | 1883  | 131,305 | 17.3|
| `非group_commit` | 10 MB   | 10   | 294       | 840,983  | 105.4 |
| `非group_commit` | 10 MB   | 30   | 118  | 2,095,331 | 276.1|

在上面的`group_commit`测试中，BE 的 CPU 使用率在 10-40% 之间。

可以看出，`group_commit` 模式在小数据量并发导入的场景下，能有效的提升导入性能，同时减少版本数，降低系统合并数据的压力。

### JDBC

#### 机器配置

* 1 台 FE：阿里云 8 核 CPU、16GB 内存、1 块 100GB ESSD PL1 云磁盘

* 1 台 BE：阿里云 16 核 CPU、64GB 内存、1 块 500GB ESSD PL1 云磁盘

* 1 台测试客户端：阿里云 16 核 CPU、64GB 内存、1 块 100GB ESSD PL1 云磁盘

* 测试版本为 Doris-3.0.1

* 关闭打印 parpared 语句的 audit log 以提高性能

#### 数据集

* tpch sf10 `lineitem` 表数据集，30 个文件，总共约 22 GB，1.8 亿行

#### 测试工具

* [DataX](https://github.com/alibaba/DataX)

#### 测试方法

* 通过 `txtfilereader` 向 `mysqlwriter` 写入数据，配置不同并发数和单个 `INSERT` 的行数

#### 测试结果

| 单个 insert 的行数 | 并发数 | 导入速率 (行/秒) | 导入吞吐 (MB/秒) |
|-------------|-----|-----------|----------|
| 100 | 10  | 160,758    | 17.21 |
| 100 | 20  | 210,476    | 22.19 |
| 100 | 30  | 214,323    | 22.92 |

在上面的测试中，FE 的 CPU 使用率在 60-70% 左右，BE 的 CPU 使用率在 10-20% 左右。

### Insert into sync 模式小批量数据

#### 机器配置

* 1 台 FE：阿里云 16 核 CPU、64GB 内存、1 块 500GB ESSD PL1 云磁盘

* 5 台 BE：阿里云 16 核 CPU、64GB 内存、1 块 1TB ESSD PL1 云磁盘。

* 1 台测试客户端：阿里云 16 核 CPU、64GB 内存、1 块 100GB ESSD PL1 云磁盘

* 测试版本为 Doris-3.0.1

#### 数据集

* tpch sf10 `lineitem` 表数据集。

* 建表语句为
```sql
CREATE TABLE IF NOT EXISTS lineitem (
  L_ORDERKEY    INTEGER NOT NULL,
  L_PARTKEY     INTEGER NOT NULL,
  L_SUPPKEY     INTEGER NOT NULL,
  L_LINENUMBER  INTEGER NOT NULL,
  L_QUANTITY    DECIMAL(15,2) NOT NULL,
  L_EXTENDEDPRICE  DECIMAL(15,2) NOT NULL,
  L_DISCOUNT    DECIMAL(15,2) NOT NULL,
  L_TAX         DECIMAL(15,2) NOT NULL,
  L_RETURNFLAG  CHAR(1) NOT NULL,
  L_LINESTATUS  CHAR(1) NOT NULL,
  L_SHIPDATE    DATE NOT NULL,
  L_COMMITDATE  DATE NOT NULL,
  L_RECEIPTDATE DATE NOT NULL,
  L_SHIPINSTRUCT CHAR(25) NOT NULL,
  L_SHIPMODE     CHAR(10) NOT NULL,
  L_COMMENT      VARCHAR(44) NOT NULL
)
DUPLICATE KEY(L_ORDERKEY, L_PARTKEY, L_SUPPKEY, L_LINENUMBER)
DISTRIBUTED BY HASH(L_ORDERKEY) BUCKETS 32
PROPERTIES (
  "replication_num" = "3"
);
```

#### 测试工具

* [Jmeter](https://jmeter.apache.org/)

需要设置的 jmeter 参数如下图所示

![jmeter1](/images/group-commit/jmeter1.jpg)
![jmeter2](/images/group-commit/jmeter2.jpg)

1. 设置测试前的 init 语句，`set group_commit=async_mode`以及`set enable_nereids_planner=false`。
2. 开启 jdbc 的 prepared statement，完整的 url 为: 

    ```
    jdbc:mysql://127.0.0.1:9030?useServerPrepStmts=true&useLocalSessionState=true&rewriteBatchedStatements=true&cachePrepStmts=true&prepStmtCacheSqlLimit=99999&prepStmtCacheSize=50&sessionVariables=group_commit=async_mode,enable_nereids_planner=false`。
    ```

3. 设置导入类型为 prepared update statement。
4. 设置导入语句。
5. 设置每次需要导入的值，注意，导入的值与导入值的类型要一一匹配。

#### 测试方法

* 通过 `Jmeter` 向`Doris`写数据。每个并发每次通过 insert into 写入 1 行数据。

#### 测试结果

* 数据单位为行每秒。

* 以下测试分为 30，100，500 并发。

#### 30 并发 sync 模式 5 个 BE3 副本性能测试

| Group commit interval | 10ms | 20ms | 50ms | 100ms |
|-----------------------|---------------|---------------|---------------|---------------|
|enable_nereids_planner=true| 891.8      | 701.1      | 400.0     | 237.5    |
|enable_nereids_planner=false| 885.8      | 688.1      | 398.7      | 232.9     |

#### 100 并发 sync 模式 5 个 BE3 副本性能测试

| Group commit interval | 10ms | 20ms | 50ms | 100ms |
|-----------------------|---------------|---------------|---------------|---------------|
|enable_nereids_planner=true| 2427.8     | 2068.9     | 1259.4     | 764.9  |
|enable_nereids_planner=false| 2320.4      | 1899.3    | 1206.2     |749.7|

#### 500 并发 sync 模式 5 个 BE3 副本性能测试

| Group commit interval | 10ms | 20ms | 50ms | 100ms |
|-----------------------|---------------|---------------|---------------|---------------|
|enable_nereids_planner=true| 5567.5     | 5713.2      | 4681.0    | 3131.2   |
|enable_nereids_planner=false| 4471.6      | 5042.5     | 4932.2     | 3641.1 |

### Insert into sync 模式大批量数据

#### 机器配置

* 1 台 FE：阿里云 16 核 CPU、64GB 内存、1 块 500GB ESSD PL1 云磁盘

* 5 台 BE：阿里云 16 核 CPU、64GB 内存、1 块 1TB ESSD PL1 云磁盘。注：测试中分别用了 1 台，3 台，5 台 BE 进行测试。

* 1 台测试客户端：阿里云 16 核 CPU、64GB 内存、1 块 100GB ESSD PL1 云磁盘

* 测试版本为 Doris-3.0.1

#### 数据集

* tpch sf10 `lineitem` 表数据集。

* 建表语句为
```sql
CREATE TABLE IF NOT EXISTS lineitem (
  L_ORDERKEY    INTEGER NOT NULL,
  L_PARTKEY     INTEGER NOT NULL,
  L_SUPPKEY     INTEGER NOT NULL,
  L_LINENUMBER  INTEGER NOT NULL,
  L_QUANTITY    DECIMAL(15,2) NOT NULL,
  L_EXTENDEDPRICE  DECIMAL(15,2) NOT NULL,
  L_DISCOUNT    DECIMAL(15,2) NOT NULL,
  L_TAX         DECIMAL(15,2) NOT NULL,
  L_RETURNFLAG  CHAR(1) NOT NULL,
  L_LINESTATUS  CHAR(1) NOT NULL,
  L_SHIPDATE    DATE NOT NULL,
  L_COMMITDATE  DATE NOT NULL,
  L_RECEIPTDATE DATE NOT NULL,
  L_SHIPINSTRUCT CHAR(25) NOT NULL,
  L_SHIPMODE     CHAR(10) NOT NULL,
  L_COMMENT      VARCHAR(44) NOT NULL
)
DUPLICATE KEY(L_ORDERKEY, L_PARTKEY, L_SUPPKEY, L_LINENUMBER)
DISTRIBUTED BY HASH(L_ORDERKEY) BUCKETS 32
PROPERTIES (
  "replication_num" = "3"
);
```

#### 测试工具

* [Jmeter](https://jmeter.apache.org/)

#### 测试方法

* 通过 `Jmeter` 向`Doris`写数据。每个并发每次通过 insert into 写入 1000 行数据。

#### 测试结果

* 数据单位为行每秒。

* 以下测试分为 30，100，500 并发。

#### 30 并发 sync 模式 5 个 BE3 副本性能测试

| Group commit interval | 10ms | 20ms | 50ms | 100ms |
|-----------------------|---------------|---------------|---------------|---------------|
|enable_nereids_planner=true| 9.1K     | 11.1K     | 11.4K     | 11.1K     |
|enable_nereids_planner=false| 157.8K      | 159.9K     | 154.1K     | 120.4K     |

#### 100 并发 sync 模式 5 个 BE3 副本性能测试

| Group commit interval | 10ms | 20ms | 50ms | 100ms |
|-----------------------|---------------|---------------|---------------|---------------|
|enable_nereids_planner=true| 10.0K     |9.2K     | 8.9K      | 8.9K    |
|enable_nereids_planner=false| 130.4k     | 131.0K     | 130.4K      | 124.1K     |

#### 500 并发 sync 模式 5 个 BE3 副本性能测试

| Group commit interval | 10ms | 20ms | 50ms | 100ms |
|-----------------------|---------------|---------------|---------------|---------------|
|enable_nereids_planner=true| 2.5K      | 2.5K     | 2.3K      | 2.1K      |
|enable_nereids_planner=false| 94.2K     | 95.1K    | 94.4K     | 94.8K     |
