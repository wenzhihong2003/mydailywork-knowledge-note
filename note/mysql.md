[MySQL数据库复制概论](https://my.oschina.net/andylucc/blog/631591)
[一致性非锁定读与一致性锁定读](https://my.oschina.net/andylucc/blog/804229)
[关于InnoDB Double Write](https://my.oschina.net/andylucc/blog/808082)
[MySQL线程池内幕](https://my.oschina.net/andylucc/blog/820624)
[MySQL统计信息](https://my.oschina.net/andylucc/blog/841720)


### mysql jdbc 记录sql实际执行各种时间
```
profileSQL=true&gatherPerfMetrics=true&reportMetricsIntervalMillis=500
```
一个完整的配置
```
jdbc:mysql://10.0.0.40:3308/unidsp?profileSQL=true&gatherPerfMetrics=true&reportMetricsIntervalMillis=500&useUnicode=true&zeroDateTimeBehavior=convertToNull&allowMultiQueries=true&characterEncoding=UTF-8
```

### mysql 批量插入
批量插入的情况下, 最理想的速度经测试, 应该是JDBC + executeBatch + JDBC url 加上参数 :
```
useServerPrepStmts=false&rewriteBatchedStatements=true&useCompression=true
```

同时sql语句也写成 insert ... values()()(), 这样的一条语句插件多条记录的形式

### MySQL不重启添加slow log慢查询日志
```
## 开启慢查询
mysql> set global slow_query_log = 'ON';
Query OK, 0 rows affected (0.78 sec)

## 检查变量值
mysql> show global variables like '%slow%';
+---------------------------+------------------------------------------------------+
| Variable_name             | Value                                                |
+---------------------------+------------------------------------------------------+
| log_slow_admin_statements | OFF                                                  |
| log_slow_slave_statements | OFF                                                  |
| slow_launch_time          | 2                                                    |
| slow_query_log            | ON                                                   |
| slow_query_log_file       | /home/username/mysql/data/db/logs/myql-slow.log      |
+---------------------------+------------------------------------------------------+
5 rows in set (0.00 sec)

## 记录不使用索引的语句
mysql> set global log_queries_not_using_indexes = 'ON';
Query OK, 0 rows affected (0.00 sec)

mysql> show global variables like '%indexes%';
+----------------------------------------+-------+
| Variable_name                          | Value |
+----------------------------------------+-------+
| log_queries_not_using_indexes          | ON    |
| log_throttle_queries_not_using_indexes | 0     |
+----------------------------------------+-------+
2 rows in set (0.00 sec)

## 设置慢查询日志的位置

mysql> set global slow_query_log_file ='/home/username/mysql/data/db/logs/myql-slow.log';
Query OK, 0 rows affected (0.01 sec)

mysql> show variables like '%slow%';
+---------------------------+------------------------------------------------------+
| Variable_name             | Value                                                |
+---------------------------+------------------------------------------------------+
| log_slow_admin_statements | OFF                                                  |
| log_slow_slave_statements | OFF                                                  |
| slow_launch_time          | 2                                                    |
| slow_query_log            | ON                                                   |
| slow_query_log_file       | /home/username/mysql/data/db/logs/myql-slow.log |
+---------------------------+------------------------------------------------------+
5 rows in set (0.00 sec)

## 按需要修改记录慢于该时间的语句
mysql> show variables like '%query_time%';
+-----------------+----------+
| Variable_name   | Value    |
+-----------------+----------+
| long_query_time | 1.000000 |
+-----------------+----------+
1 row in set (0.00 sec)

## 刷新即可生效

mysql> FLUSH LOGS;
Query OK, 0 rows affected (0.22 sec)

mysql>
```


