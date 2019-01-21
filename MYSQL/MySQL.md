# MySQL 优化

## 如何定位慢的原因在sql上面
- 通过慢查询日志的分析，进行分析
- 查看mysql数据库的日志输出类型， log-output, file/table/none, 设置none，其他失效。
```sql
show VARIABLES like '%log_output%';
+---------------------+---------------------+
| Variable_name       | Value               |
+---------------------+---------------------+
| log_output          | FILE                |
+---------------------+---------------------+
```
- slow_query_log ：是否开启慢查询日志，1表示开启，0表示关闭。
- slow_query_log_file: 设置慢查询的日志文件路径
```sql
show variables like 'slow_query%';
+---------------------+---------------------+
| Variable_name       | Value               |
+---------------------+---------------------+
| slow_query_log      | ON                  |
| slow_query_log_file | /usr/local/mysql/data/localhost-slow.log |
+---------------------+---------------------+
```
- long_query_time：sql的执行时间大于（等于不记录）该配置项设置的时间会被记录。 默认10秒;
```sql
set global long_query_time=4
```
- 查询有多少条慢查询记录
```sql
show global status like '%slow_queries%';
```
## EXPLAIN 分析SQL


## 索引 index

### 聚集索引
- InnoDB引擎是聚集索引，数据文件和索引文件在一起，为同一个文件
- InnoDB的表，必须有主键，如果没有主键，找唯一的列，如果没有唯一的列，会创建一个隐含字段作为主键，该字段为6个字节，bigint
- InnoDB的所有辅助索引都引用主键作为data域
- 所有的叶子节点就是数据行
- 一个表中只能有一个聚集索引，一般为主键

### 非聚集索引
- myisam引擎是非聚集索引，数据文件跟索引文件分开。
- 除了聚集索引，都是非聚集索引， 分成普通索引，唯一索引，全文索引
- 非聚集索引的二次查询问题
```sql
    索引：primary key (a), index(b)
    select a, b, c from t where c = 500 // 没有索引，全表扫描
    select a, b, c from t where b = 500 // 会触发二次查询
    select a, b, c from t where a = 500 // 不会触发二次查询
```

### 复合索引 
- 最左侧索引的原则： 查询的时候如果where条件里面没有最左边的一到多列，索引就不会起作用
```sql
    例如： index（a,b,c）
    select a, b, c from t where b=0 and c=1 // 不会使用索引
    select a, b, c from t where a=0 and c=1 
    // 上面这个SQL会使用索引，但是只用到了一个字段a， c没有用到索引，
    // 在a=0的结果里面进行where扫描，过滤出来c=1的数据
    select a, b, c from t where a=0 and b=1 and c=1 // 使用索引
    select a, b, c from t where a=0 and b=1 // 会使用索引
    select a, b, c from t where a=0 // 会使用索引
```
- 查询条件中含有函数或表达式，mysql不会为其使用索引
```sql
SELECT * FROM t WHERE a='10001' AND left(b, 1)='c';
等价于
SELECT * FROM t WHERE a='10001' AND b like 'c%';

第一种不会使用索引，第二种会使用索引
```

## SQL 优化
- 精确匹配： 这里精确匹配指“=”或“IN”匹配
- 索引对顺序是敏感的，但是由于MySQL的查询优化器会自动调整where子句的条件顺序以使用适合的索引