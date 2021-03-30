---
title: MYSQL性能优化
date: 2020-03-29 17:44:00
tags: ['知识点','mysql']
categories: ['学习']
---
#### 优化要考虑的问题
##### 优化可能带来的问题
1. 优化不总是对一个单纯的环境进行，还很可能是一个复杂的已投产的系统！
2. 优化手段有很大的风险，一定要意识到和预见到！
3. 任何的技术可以解决一个问题，但必然存在带来一个问题的风险！
4. 对于优化来说调优而带来的问题,控制在可接受的范围内才是有成果。
5. 保持现状或出现更差的情况都是失败！
<!--more-->
##### 优化的需求
1. 稳定性和业务可持续性,通常比性能更重要！
2. 优化不可避免涉及到变更，变更就有风险！
3. 优化使性能变好，维持和变差是等概率事件！
4. 优化应该是各部门协同，共同参与的工作，任何单一部门都不能对数据库进行优化！
#### 优化的思路
##### 优化的方向
1. 安全--->数据安全性
2. 性能--->数据的高性能访问
##### 优化的纬度
- 硬件，系统配置，数据库表结构，SQL及索引
- 从优化成本：硬件>系统配置>数据库表结构>SQL及索引
- 从优化效果：SQL及索引>数据库表结构>系统配置>硬件
##### 检查问题常用工具
###### 检查问题常用工具
```bash
mysqladmin      #mysql客户端，可进行管理操作
mysqlshow       #查看shell命令
show [ SESSION | GLOBAL ] variables     #查看数据库参数信息
SHOW [ SESSION | GLOBAL ] STATUS        #查看数据库得状态信息
SHOW ENGINE INNODB STATUS Innodb        #引擎得所有状态
information_schema      #获取元数据的方法
SHOW PROCESSLIST        #查看当前所有连接session状态
explain          #获取查询语句的执行计划
how index       #查看表的索引信息
slow-log        #记录慢查询语句
mysqldumpslow       #分析slowlog文件
```
###### 不常用
```bash
zabbix      #监控主机、系统、数据库（部署zabbix监控平台）
mysqlslap   #分析慢日志
sysbench    #压力测试工具
workbench   #管理、备份、监控、分析、优化工具（比较费资源）
pt-query-digest     #分析慢日志
mysql profiling      #统计数据库整体状态工具
Performance Schema mysql    #性能状态统计的数据
```
##### 数据库适用优化思路
###### 应急调优思路
1. show processlist（查看链接session状态）
2. explain(分析查询计划)，show index from table（分析索引）
3. 通过执行计划判断，索引问题（有没有、合不合理）或者语句本身问题
4. show status like '%lock%'; # 查询锁状态
5. SESSION_ID; # 杀掉有问题的session
###### 常规调优思路
1. 查看slowlog，分析slowlog，分析出查询慢的语句。
2. 按照一定优先级，进行一个一个的排查所有慢语句。
3. 分析top sql，进行explain调试，查看语句执行时间。
4. 调整索引或语句本身。
#### 查询优化
##### MySQL查询流程
1. 客户端将查询发送到服务器；
2. 服务器检查查询缓存，如果找到了，就从缓存中返回结果，否则进行下一步。
3. 服务器解析，预处理。
4. 查询优化器优化查询
5. 生成执行计划，执行引擎调用存储引擎API执行查询
6. 服务器将结果发送回客户端。
##### 查询优化
###### 慢查询
1. 慢查询日志开启
- 在配置文件my.cnf或my.ini中在[mysqld]一行下面加入两个配置参数
```bash
log-slow-queries=/data/mysqldata/slow-query.log
long_query_time=5   #查询时间大于5s则记录
```
- 还可以在my.cnf或者my.ini中添加log-queries-not-using-indexes参数，表示记录下没有使用索引的查询。
2. 慢查询分析
- 进入log的存放目录，运行：
```bash
[root@mysql_data]# mysqldumpslow slow-query.log
Reading mysql slow query log fromslow-query.log
Count: 2 Time=11.00s (22s) Lock=0.00s (0s)Rows=1.0 (2), root[root]@mysql
select count(N) from t_user; 
```
- mysqldumpslow命令  
`/path/mysqldumpslow -s c -t 10/database/mysql/slow-query.log`
- 这会输出记录次数最多的10条SQL语句，其中：
    - -s, 是表示按照何种方式排序，c、t、l、r分别是按照记录次数、时间、查询时间、返回的记录数来排序，ac、at、al、ar，表示相应的倒叙
    - -t, 是top n的意思，即为返回前面多少条的数据；
    - -g, 后边可以写一个正则匹配模式，大小写不敏感的；
###### EXPLAIN
- EXPLAIN可以帮助开发人员分析SQL问题，EXPLAIN显示了MySQL如何使用使用SQL执行计划  
`EXPLAIN SELECT * FROM products`
1. id
2. select_type
3. table
4. **type**
    - **system > const > eq_ref > ref > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL**
    - 一般来说，得保证查询至少达到range级别,最好能达到ref。
    1. system：表仅有一行，这是const类型的特列，平时不会出现，这个也可以忽略不计。
    2. const：数据表最多只有一个匹配行，因为只匹配一行数据，所以很快
    3. eq_ref：mysql手册是这样说的:"对于每个来自于前面的表的行组合，从该表中读取一行。这可能是最好的联接类型，除了const类型。它用在一个索引的所有部分被联接使用并且索引是UNIQUE或PRIMARY KEY"。eq_ref可以用于使用=比较带索引的列。
    4. ref：查询条件索引既不是UNIQUE也不是PRIMARY KEY的情况。ref可用于=或<或>操作符的带索引的列。
    5. ref_or_null：该联接类型如同ref，但是添加了MySQL可以专门搜索包含NULL值的行。在解决子查询中经常使用该联接类型的优化。
    6. index_merge：该联接类型表示使用了索引合并优化方法。在这种情况下，key列包含了使用的索引的清单，key_len包含了使用的索引的最长的关键元素。
    7. unique_subquery：该类型替换了下面形式的IN子查询的ref: value IN (SELECT primary_key FROM single_table WHERE some_expr) unique_subquery是一个索引查找函数,可以完全替换子查询,效率更高。
    8. index_subquery：该联接类型类似于unique_subquery。可以替换IN子查询,但只适合下列形式的子查询中的非唯一索引: value IN (SELECT key_column FROM single_table WHERE some_expr)
    9. range：只检索给定范围的行,使用一个索引来选择行。
    10. index：该联接类型与ALL相同,除了只有索引树被扫描。这通常比ALL快,因为索引文件通常比数据文件小。
    11. ALL：对于每个来自于先前的表的行组合,进行完整的表扫描。（性能最差）
5. **possible_keys**：指出MySQL能使用哪个索引在该表中找到行。如果是空的，没有相关的索引。这时要提高性能，可通过检验WHERE子句，看是否引用某些字段，或者检查字段不是适合索引。
6. **key**：实际使用到的索引。如果为NULL，则没有使用索引。如果为primary的话，表示使用了主键。
7. key_len
8. ref
9. rows
10. Extra
#### 索引优化
##### 索引类型
1. 主键索引PRIMARY KEY   
`PRIMARY KEY ('id')`
2. 唯一索引 UNIQUE  
`UNIQUE KEY 'num' ('number') USING BTREE`
3. 普通索引 INDEX  
`KEY 'num' ('number') USING BTREE`
4. 组合索引 INDEX  
`KEY 'num' ('number','name') USING BTREE`
5. 全文索引 FULLTEXT
#### 索引的存储结构
##### BTree索引
###### 特点
1. BTREE索引以B+树的结构存储数据
2. BTREE索引能够加快数据的查询速度
3. BTREE索引更适合进行行范围查找
###### 使用场景
1. 全值匹配的查询，例如根据订单号查询 order_sn='98764322119900'
2. 联合索引时会遵循最左前缀匹配的原则,即最左优先
3. 匹配列前缀查询，例如：order_sn like '9876%'
4. 匹配范围值的查找，例如：order_sn > '98764322119900'
5. 只访问索引的查询
##### 哈希索引
###### 特点
1. Hash索引仅仅只能满足“=”,“IN”和“<=>”查询，不能使用范围查询；
2. Hash索引无法被利用来避免数据的排序操作；
3. Hash索引不能利用部分索引键查询；
4. Hash索引在任何时候都不能避免表扫描；
5. Hash索引遇到大量Hash值相等的情况后性能并不一定就会比B-Tree索引高；
#### 索引的使用
##### 使用场景
1. 主键自动建立唯一索引；
2. 经常作为查询条件在WHERE或者ORDER BY 语句中出现的列要建立索引；
3. 作为排序的列要建立索引；
4. 查询中与其他表关联的字段，外键关系建立索引
5. 高并发条件下倾向建立组合索引；
6. 用于聚合函数的列可以建立索引，例如使用count(number)时，number列就要建立索引
##### 不适用场景
1. 有大量重复的列不单独建立索引
2. 表记录太少不要建立索引，因为没有太大作用。
3. 不会作为查询的列不要建立索引
#### 存储优化
#####  InnoDB存储引擎
###### 特点
1. InnoDB存储引擎提供了具有提交、回滚和崩溃恢复能力的事务安全。
2. 提供了对数据库事务ACID（原子性Atomicity、一致性Consistency、隔离性Isolation、持久性Durability）的支持，实现了SQL标准的四种隔离级别。
3. 设计目标就是处理大容量的数据库系统，MySQL运行时InnoDB会在内存中建立缓冲池，用于缓冲数据和索引。
4. 执行“select count(*) from table”语句时需要扫描全表，因为使用innodb引擎的表不会保存表的具体行数，所以需要扫描整个表才能计算多少行。
5. InnoDB引擎是行锁，粒度更小，所以写操作不会锁定全表，在并发较高时，使用InnoDB会提升效率。
6. InnoDB清空数据量大的表时，是非常缓慢，这是因为InnoDB必须处理表中的每一行，根据InnoDB的事务设计原则，首先需要把“删除动作”写入“事务日志”，然后写入实际的表。所以，清空大表的时候，最好直接drop table然后重建。
###### 使用场景
1. 经常UPDETE/INSERT的表，使用处理多并发的写请求
2. 支持事务，必选InnoDB。
3. 可以从灾难中恢复（日志+事务回滚）
4. 外键约束、列属性AUTO_INCREMENT支持
##### MyISAM存储引擎
###### 特点
1. MyISAM不支持事务，不支持外键，SELECT/INSERT为主的应用可以使用该引擎。
2. 每个MyISAM在存储成3个文件
    -  frm：存储表定义（表结构等信息）
    -  MYD(MYData)，存储数据
    -  MYI(MYIndex)，存储索引
3. 不同MyISAM表的索引文件和数据文件可以放置到不同的路径下。
4. MyISAM类型的表提供修复的工具，可以用CHECK TABLE语句来检查MyISAM表健康，并用REPAIR TABLE语句修复一个损坏的MyISAM表。
5. 在MySQL5.6以前，只有MyISAM支持Full-text全文索引
###### 使用场景
1. 经常SELECT/INSERT的表，插入不频繁，查询非常频繁
2. 不支持事务
3. 做很多count 的计算。
##### MyISAM和Innodb区别
1. MyISAM是非事务安全型的，而InnoDB是事务安全型的。
2. MyISAM锁的粒度是表级，而InnoDB支持行级锁定。
3. MyISAM不支持外键，而InnoDB支持外键
4. MyISAM相对简单，所以在效率上要优于InnoDB，小型应用可以考虑使用MyISAM。
5. InnoDB表比MyISAM表更安全。
#### 存储优化
##### 禁用索引
- 对于使用索引的表，插入记录时，MySQL会对插入的记录建立索引。如果插入大量数据，建立索引会降低插入数据速度。
- 禁用索引的语句： `ALTER TABLE table_name DISABLE KEYS`
- 开启索引语句： `ALTER TABLE table_name ENABLE KEYS`
##### 禁用唯一性检查
- 唯一性校验会降低插入记录的速度，可以在插入记录之前禁用唯一性检查，插入数据完成后再开启。
- 禁用唯一性检查的语句：`SET UNIQUE_CHECKS = 0;` 
- 开启唯一性检查的语句：`SET UNIQUE_CHECKS = 1;`
##### 禁用外键检查
- 插入数据之前执行禁止对外键的检查，数据插入完成后再恢复，可以提供插入速度。
- 禁用：`SET foreign_key_checks = 0;`
- 开启：`SET foreign_key_checks = 1;`
##### 批量插入数据
- 插入数据时，可以使用一条INSERT语句插入一条数据，也可以插入多条数据。
##### 禁止自动提交
- 插入数据之前执行禁止事务的自动提交，数据插入完成后再恢复，可以提高插入速度。
- 禁用：`SET autocommit = 0;`
- 开启：`SET autocommit = 1;`
#### 数据库结构优化
##### 优化表结构
- 尽量将表字段定义为NOT NULL约束，这时由于在MySQL中含有空值的列很难进行查询优化，NULL值会使索引以及索引的统计信息变得很复杂。
- 对于只包含特定类型的字段，可以使用enum、set 等数据类型。
- 数值型字段的比较比字符串的比较效率高得多，字段类型尽量使用最小、最简单的数据类型。例如IP地址可以使用int类型。
- 尽量使用TINYINT、SMALLINT、MEDIUM_INT作为整数类型而非INT，如果非负则加上UNSIGNED。但对整数类型指定宽度，比如INT(11)，没有任何用，因为指定的类型标识范围已经确定。
- VARCHAR的长度只分配真正需要的空间
- 尽量使用TIMESTAMP而非DATETIME，但TIMESTAMP只能表示1970 - 2038年，比DATETIME表示的范围小得多，而且TIMESTAMP的值因时区不同而不同。
- 单表不要有太多字段，建议在20以内
- 合理的加入冗余字段可以提高查询速度。
##### 表拆分
###### 垂直拆分
- 垂直拆分按照字段进行拆分，其实就是把组成一行的多个列分开放到不同的表中，这些表具有不同的结构，拆分后的表具有更少的列。
###### 水平拆分
- 水平拆分按照行进行拆分，常见的就是分库分表。
##### 表分区
##### 读写分离
##### 数据库集群
#### 硬件优化
##### 内存
##### 磁盘
##### CPU
##### 网络
#### 缓存优化
##### 查询缓存
- 通过以下命令查看缓存相关变量  
`show variables like '%query_cache%';`
- have_query_cache：表示此版本mysql是否支持缓存
- query_cache_limit ：缓存最大值
- query_cache_size：缓存大小
- query_cache_type：off 表示不缓存，on表示缓存所有结果。
##### 全局缓存
##### 局部缓存
##### 其他缓存
#### 服务器优化
##### MySQL参数
- 通过优化MySQL的参数可以提高资源利用率，从而达到提高MySQL服务器性能的目的。MySQL的配置参数都在my.conf或者my.ini文件的[mysqld]组中，常用的参数如下：
- back_log
- wait_timeout
- max_connections
- max_user_connections
- thread_concurrency
- skip-name-resolve
- default-storage-engine
#### Linux系统优化