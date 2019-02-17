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
- DESCRIBE和EXPLAIN语句是同义词，实际上在平时使用过程中DESCRIBE多用于获取表结构的信息，然后EXPLAIN多用于获取SQL语句的执行计划。MySQL解析器对这两个语句是完全作为同义词对待的。
- EXPLAIN 结果
```sql
mysql> explain select * from employees;
+----+-------------+-----------+------+---------------+------+---------+------+------+-------+
| id | select_type | table     | type | possible_keys | key  | key_len | ref  | rows | Extra |
+----+-------------+-----------+------+---------------+------+---------+------+------+-------+
|  1 | SIMPLE      | employees | ALL  | NULL          | NULL | NULL    | NULL |    1 | NULL  |
+----+-------------+-----------+------+---------------+------+---------+------+------+-------+
row in set (0.03 sec)
```
- select_type: 示查询中每个select子句的类型
```
SIMPLE(简单SELECT,不使用UNION或子查询等)
PRIMARY(查询中若包含任何复杂的子部分,最外层的select被标记为PRIMARY)
UNION(UNION中的第二个或后面的SELECT语句)
DEPENDENT UNION(UNION中的第二个或后面的SELECT语句，取决于外面的查询)
UNION RESULT(UNION的结果)
SUBQUERY(子查询中的第一个SELECT)
DEPENDENT SUBQUERY(子查询中的第一个SELECT，取决于外面的查询)
DERIVED(派生表的SELECT, FROM子句的子查询)
UNCACHEABLE SUBQUERY(一个子查询的结果不能被缓存，必须重新评估外链接的第一行)
```
- table: 显示这一行的数据是关于哪张表的，有时不是真实的表名字,看到的是derivedx(x是个数字,我的理解是第几步执行的结果)

- type(重要)： 表示MySQL在表中找到所需行的方式，又称“访问类型”。结果值从好到坏依次是：
system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL

一般来说，得保证查询至少达到range级别，最好能达到ref，否则就可能会出现性能问题。
```
- ALL：Full Table Scan， MySQL将遍历全表以找到匹配的行
- index: Full Index Scan，index与ALL区别为index类型只遍历索引树
- range: 只检索给定范围的行，使用一个索引来选择行
- ref: 表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值
- eq_ref: 类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用primary key或者 unique key作为关联条件
- const、system: 当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。如将主键置于where列表中，MySQL就能将该查询转换为一个常量,system是const类型的特例，当查询的表只有一行的情况下，使用system
- NULL: MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成。
```
- possible_keys：可能会用到的索引列表，并不代表该SQL已经用了。
- key：显示实际决定使用的索引。如果没有选择索引，就是NULL
- key_len：显示实际决定使用的索引长度。如果键是NULL，则长度为NULL。使用的索引的长度。在不损失精确性的情况下，长度越短越好
- ref：显示使用哪个列或常数与key一起从表中选择行。
- rows：显示MySQL认为它执行查询时必须检查的行数。
- Extra：包含MySQL解决查询的详细信息，也是关键参考项之一。
```
Distinct
一旦MYSQL找到了与行相联合匹配的行，就不再搜索了

Not exists
MYSQL 优化了LEFT JOIN，一旦它找到了匹配LEFT JOIN标准的行，

就不再搜索了

Range checked for each

Record（index map:#）
没有找到理想的索引，因此对于从前面表中来的每一 个行组合，MYSQL检查使用哪个索引，并用它来从表中返回行。这是使用索引的最慢的连接之一

Using filesort
看 到这个的时候，查询就需要优化了。MYSQL需要进行额外的步骤来发现如何对返回的行排序。它根据连接类型以及存储排序键值和匹配条件的全部行的行指针来 排序全部行

Using index
列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表 的全部的请求列都是同一个索引的部分的时候

Using temporary
看到这个的时候，查询需要优化了。这 里，MYSQL需要创建一个临时表来存储结果，这通常发生在对不同的列集进行ORDER BY上，而不是GROUP BY上

Using where
使用了WHERE从句来限制哪些行将与下一张表匹配或者是返回给用户。如果不想返回表中的全部行，并且连接类型ALL或index， 这就会发生，或者是查询有问题
```
## 索引 index

### 聚集索引
- InnoDB引擎是聚集索引，数据文件和索引文件在一起，为同一个文件
```
create table sys_config(···) 会创建一个文件sys_config.idb
```
- InnoDB的表，必须有主键，如果没有主键，找唯一的列，如果没有唯一的列，会创建一个隐含字段作为主键，该字段为6个字节，bigint
- InnoDB的所有辅助索引都引用主键作为data域
- 所有的叶子节点就是数据行
- 一个表中只能有一个聚集索引，一般为主键

### 非聚集索引
- myisam引擎是非聚集索引，数据文件跟索引文件分开（mysql8.0不一样，以8.0以前版本为例）。
```
create table sys_config(···) 会创建3个文件， 
sys_config.MYI索引文件, 
sys_config.MYD数据文件，
sys_config.frm表结构文件
```
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
- mysql会自动优化where后面的查询条件的顺序，
```sql
    例如： index（a,b,c）
    select a, b, c from t where b=0 and a=1 
    select a, b, c from t where a=0 and b=1 
    // 这两个SQL是一样的，mysql的执行器会把第一个变成第二个
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
### SQL中使用索引
- 指定固定索引
```sql
SELECT * FROM employees  FORCE INDEX(idx_first_name) where first_name > 'm' ;
```
- 使用索引列表
```sql
SELECT * FROM employees  USE INDEX(idx_first_name, idx_first_last_name) where first_name > 'm' ;
```
- 不使用指定的索引
```sql
SELECT * FROM employees IGNORE INDEX (priority) ...
```

## 事务
### 特性
- 原子性（Atomicity）
```
在同一个事务中执行的多个更新操作，要么都成功，只要有一个失败的，所有的成功都需要失败，回滚。
```
- 一致性（Consistency）
- 隔离性（Isolation）
- 持久性（Durability）