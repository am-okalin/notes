[TOC]
## 帮助文档
[mysql教程](https://juejin.cn/post/6850037271233331208#refetch)  
[歪麦-mysql慢查询](https://www.awaimai.com/1809.html)  
[MySQL架构总览->查询执行流程->SQL解析顺序](https://www.cnblogs.com/annsshadow/p/5037667.html)
[备份](https://juejin.cn/post/6861127903736758285)

## mysql shell
- `mysql --help | grep my.cnf`查找使用的`my.cnf`
- 例:修改字符编码: `\s` 查看当前字符编码，修改`my.cnf`
```shell
# 登录时可不用输入密码
user = root 
password = 123456
```
- `mysql -h[server] -u[username] -p[password] -P3306`进入`mysql shell`
- 进入`mysql shell`后使用`help`查看`快捷命令`
    + `ego` 使数据立起来
    + `clear` 取消当前未完成的工作
    + `status` 显示当前服务器状态 
- `show variables` 显示系统变量(扩展`show variables like '%xxxx%'`)
- `show status` 显示状态信息(扩展`show status like '%xxxx%'`)
- `show innodb status` 显示InnoDB存储引擎的状态
- `show table status from database where name="tablename"` 查看数据状态(包含引擎)
- `show processlist` 查看当前SQL执行，包括执行状态、是否锁表等
- `show databases` 查看所有库
- `use [database]` 使用某库

### `variables`变量
- 在`my.cnf`中写入`variable=xxxx`并重启`mysqld`后生效
- 在`mysql shell`中`SET variables=xxxx`开启新`session`生效，重启后失效
- `show variables like '%variables%'` 显示变量
    + `storage_engine` 默认存储引擎
    + `slow_query_log` 是否开启慢查询
    + `tx_isolation` 事务隔离级别
    + `innodb_page_size` 页的大小有`4K 8K 16K(default)`

#### 慢查询相关变量
- `slow_query_log` 慢查询日志(0-关闭，1-开启)
- `slow_query_log_file` 5.6以上版本的慢查询日志存储路径
- `log_slow_queries` 5.6以下版本的慢查询日志存储路径
- `long_query_time` 慢查询阈值，多余阈值就记录日志，秒为单位
- `log_queries_not_using_indexes` 未用索引`full index scan`的查询也被记录到慢查询日志中(0-关闭，1-开启)
- `log_output` 日志存储方式：文件`FILE`(默认)，数据库`TABLE`(更耗资源)。可多选`FILE,TABLE`

### mysqldumpslow
- `-s ORDER` 排序(默认`at`) `ORDER`如下
    + `al`: average lock time 
    + `ar`: average rows sent
    + `at`: average query time
    + `c` : count 访问次数
    + `l` : lock time
    + `r` : rows sent
    + `t` : query time  
- `-t NUM` 显示数目
- `-g PATTERN` 同`grep`用法相似，`PATTERN`是用引号包裹的字符串
- `mysqldumpslow -s c -t 10 /var/lib/mysql/hostname-slow.log`得到访问次数最多的10个SQL
- `mysqldumpslow -s r -t 10 /var/lib/mysql/hostname-slow.log`获取返回记录集最多的10个SQL
- `mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/hostname-slow.log`得到按照时间排序的前10条里面含有左连接的查询语句。 可配合管道符在后面加`| more`

### mysqldump
- `mysqldump [OPTION] DB [table]` 导出数据库的某张表
- `mysqldump [OPTION] --database [OPTION] DB1 [DB2 DB3...]` 导出指定库
- `mysqldump [OPTION] --all-database [OPTION]` 导出所有库
- `--no-data`/`-d` 不导出数据，仅导出表结构
- `--no-create-db`/`-n` 不创建数据库，仅导出数据
- `--no-create-info`/`-t` 不导出表结构，仅导出数据
- `--replace` 使用 REPLACE 语句代替 INSERT 语句
- `--routines`/`-R` 导出存储过程及自定义函数
- `--lock-all-tables` 设置全局锁，锁定所有的数据表以保证备份数据的完整性
- `--ignore-table DB.table` 忽略某张表

#### 数据迁移脚本
```shell
# 来源库
src_user="root" # 用户名
src_password="root" # 密码
src_host="localhost" # Host
src_port="3306" # 端口
src_database="test" # 数据库名
src_table="edu" # 表名

# 目的库
dst_user="root" # 用户名
dst_password="root" # 密码
dst_host="localhost" # Host
dst_port="3306" # 端口
dst_database="test" # 数据库名

# 导入数据
mysqldump --host=$src_host --port=$src_port -u$src_user -p$src_password $src_database --tables $src_table | mysql --host=$dst_host --port=$dst_port -u$dst_user -p$dst_password $dst_database
```

#### 定时备份数据
使用修改`chmod 611 /root/mysql_backup.sh`权限并写入以下内容，再加入`crontab`
```shell
#!/bin/bash

# 以下配置信息请自己修改
mysql_user="root" #MySQL备份用户
mysql_password="root" #MySQL备份用户的密码
mysql_host="localhost"
mysql_port="3306"
mysql_charset="utf8mb4" #MySQL编码
backup_db_arr=("db1" "db2") #要备份的数据库名称，多个用空格分开隔开 如("db1" "db2" "db3")
backup_location=/var/www/mysql  #备份数据存放位置，末尾请不要带"/",此项可以保持默认，程序会自动创建文件夹
expire_backup_delete="OFF" #是否开启过期备份删除 ON为开启 OFF为关闭
expire_days=3 #过期时间天数 默认为三天，此项只有在expire_backup_delete开启时有效

# 本行开始以下不需要修改
backup_time=`date +%Y%m%d%H%M`  #定义备份详细时间
backup_Ymd=`date +%Y-%m-%d` #定义备份目录中的年月日时间
backup_3ago=`date -d '3 days ago' +%Y-%m-%d` #3天之前的日期
backup_dir=$backup_location/$backup_Ymd  #备份文件夹全路径
welcome_msg="Welcome to use MySQL backup tools!" #欢迎语

# 判断MYSQL是否启动,mysql没有启动则备份退出
mysql_ps=`ps -ef |grep mysql |wc -l`
mysql_listen=`netstat -an |grep LISTEN |grep $mysql_port|wc -l`
if [ [$mysql_ps == 0] -o [$mysql_listen == 0] ]; then
        echo "ERROR:MySQL is not running! backup stop!"
        exit
else
        echo $welcome_msg
fi

# 连接到mysql数据库，无法连接则备份退出
mysql -h$mysql_host -P$mysql_port -u$mysql_user -p$mysql_password <<end
use mysql;
select host,user from user where user='root' and host='localhost';
exit
end

flag=`echo $?`
if [ $flag != "0" ]; then
        echo "ERROR:Can't connect mysql server! backup stop!"
        exit
else
        echo "MySQL connect ok! Please wait......"
        # 判断有没有定义备份的数据库，如果定义则开始备份，否则退出备份
        if [ "$backup_db_arr" != "" ];then
                #dbnames=$(cut -d ',' -f1-5 $backup_database)
                #echo "arr is (${backup_db_arr[@]})"
                for dbname in ${backup_db_arr[@]}
                do
                        echo "database $dbname backup start..."
                        `mkdir -p $backup_dir`
                        `mysqldump -h$mysql_host -P$mysql_port -u$mysql_user -p$mysql_password $dbname --default-character-set=$mysql_charset | gzip > $backup_dir/$dbname-$backup_time.sql.gz`
                        flag=`echo $?`
                        if [ $flag == "0" ];then
                                echo "database $dbname success backup to $backup_dir/$dbname-$backup_time.sql.gz"
                        else
                                echo "database $dbname backup fail!"
                        fi
                        
                done
        else
                echo "ERROR:No database to backup! backup stop"
                exit
        fi
        # 如果开启了删除过期备份，则进行删除操作
        if [ "$expire_backup_delete" == "ON" -a  "$backup_location" != "" ];then
                 #`find $backup_location/ -type d -o -type f -ctime +$expire_days -exec rm -rf {} \;`
                 `find $backup_location/ -type d -mtime +$expire_days | xargs rm -rf`
                 echo "Expired backup data delete complete!"
        fi
        echo "All database backup success! Thank you!"
        exit
fi
```

#### 恢复备份
- 使用`bash`的方式导入
```shell
# 往DB库中导入数据
mysql -uroot -proot DB < backup.sql
# 和建库语句一起导入
mysql -uroot -proot --host=127.0.0.1 --port=33006 < global.sql
```
- 使用`mysql shell`的方式导入
```shell
mysql -u root -p
use dbname;
source dbname.sql
```
 
##  MySQL优化
### SQL语句优化
#### 开启慢查询日志
- 使用慢查询日志对有效率问题的sql进行监控。进入mysql执行如下
- `show variables like 'slow_query_log'` 查看是否开启慢查询日志
- `set global slow_query_log_file='/home/mysql/sql_log/mysql-slow.log'` 设置慢查询日志存储路径
- `set global log_queries_not_using_indexex=on;` 记录未使用索引的查询
- `set global slow_query_log=on` 开启慢查询日志
- `set global log_query_time=1` 通常0.01比较合适

#### 慢查询日志分析工具
- `mysqldumpslow [option] [logs]`这是官方的慢查询工具
    + `-t n` 获取前n条慢查询
    + `mysqldumpslow -t 3 /home/mysql/data/mysql-slow.log | more`
- `pt_query-digest [option] [log]` 一个更好的工具
    + `--limit=A` 限制默认取前n条慢查询，默认是前95%
    + `--review` 将观察日志写入某张表中
    + `--create-reviewtable` 自动建立观察日志存储表
    + `--explain=d` 执行explain分析sql语句
    + `pt_query-digest /home/mysql/data/mysql-slow.log | more` 

#### 通过慢查询结果找出要优化的sql
- 查询次数多且每次查询占用时间长的sql
    + 通常为`pt_query-digest`分析的前几个查询
- IO(数据库的主要瓶颈)最大的sql
    + 注意`pt_query-digest`分析中的`Row examine`(扫描行数)项
- 未命中索引的sql
    + 注意`pt_query-digest`分析中的`Row examine`和 `Rows Send`(发送行数)
    + 若`Rows Send`远远大于`Row examine` 则说明索引命中不高

#### 找出sql后用explain进行分析
- `explain`加在sql前面，就可以查询sql的执行计划
    + 数据库中的sql都需要执行计划的分析，在进行sql语句的查询
- `explain`返回基本列含义
    + `select_type` 
        * `SIMPLE` 简单查询
        * `PRIMARY` 查询中包含复杂子查询的外层部分
        * `SUBQUERY` 在SELECT或WHERE列表中包含了子查询
        * `DERIVED` 在FROM中包含的子查询。MYSQL会递归执行这些子查询把结果放入临时表
        * `UNION` 若第二个SELECT出现在UNION之后，则被标记为`UNION`,若UNION包含在FROM子句的子查询中，外层SELECT被标记为`DERIVED`
        * `UNION RESULT` 从UNION表中获取结果的SELECT
    + `id` 
    + `table` 本行数据来自于哪张报表
    + `type` 连接类型由好至坏
        * `const` 基于主键或唯一索引的常数查找
        * `eq_reg` 基于主键或唯一索引的范围查找
        * `ref` 多见于连接查询，一个表基于一个索引的查找
        * `range` 基于索引的范围查找
        * `index` 基于索引扫描
        * `ALL` 基于表扫描
    + `possible_keys` 表中可能应用的索引
    + `key` 表中实际应用的索引
    + `key_len` 使用索引的长度,在不损失精确性的情况下，越短越好
    + `ref` 显示索引的哪一列被用了，是一个常数的话效果最好
    + `row` 表扫描行数
- `explain`返回扩展列含义
    + `using filesort` 用到文件排序，通常在order by语句中出现，有此项需优化。
    + `using temporary` 用到零时表，通常在group by语句中出现，有此项需优化。

#### max&count的优化方法
- max函数使用`覆盖索引`方法
    + `explain select max(payment_date) from payment` 没索引的话有大量的IO操作
    + `create index idx_paydate on payment(payment_date)` 添加索引
- count()优化 
    + count(\*) 查询结果包含null的行
    + count(id) 查询结果不包含null的行

#### 子查询优化
- 通常要把子查询优化为连接查询(jion)。执行子查询时，MYSQL需要创建临时表，查询完毕后再删除这些临时表，所以，子查询的速度会受到一定的影响，这里多了一个创建和销毁临时表的过程。
- 优化时要注意关联键是否有一对多的关系，要注意数据重复
```sql
select title,release_year,LENGTH
form film 
where film_id in (
    select filem_id from film_actor where actor_id in(
        select actor_id from actor where first_name='sandra'
))
```

### group by优化
- group by 通常会用到临时表
- 这个group by 内连接没有用到索引，执行计划中会有filesort，temporary这样的扩展列
```sql
explain SELECT actor.first_name,actor.last_name, COUNT(*)
FROM sakila.film_actor
INNER JOIN sakila.actor USING(actor_id)
GROUP BY film_actor.actor_id;
```
- 内连接一个子查询临时表，临时表中的索引被用到了
```sql
explain SELECT actor.first_name,actor.last_name, c.cnt
FROM sakila.actor INNER JOIN (
    SELECT actor_id, COUNT(*) AS cnt FROM sakila.film_actor GROUP BY actor_id
) AS c USING(actor_id);
```

### order by优化
- order by会进行filesort。
- 使用索引进行order by操作，加了索引就不会进行filesort
- limit设定起始页也会有大量IO操作，可加上过滤条件比如查看500页后的5条，不用`limit 500 5`,而使用where进行条件限定

### 索引优化
1. `where` `group by` `order by` `on`中出现的列
2. 索引字段越小越好
3. 离散度大的列放到联合索引前面
4. 重复索引:  相同的列以相同的顺序建立的同类型索引
5. 冗余索引:  多个(联合)索引的前缀列相同，或联合索引中包含了主键索引


## 备份`backup`与恢复
- 备份对象
    + 数据
    + 配置文件
    + 代码：存储过程，存储函数，触发器
    + OS相关配置文件 (todo::待明确)
    + 复制相关的配置 (todo::待明确)
    + 二进制日志
- 根据`mysqld状态`区分备份类型(取决于业务需求)
    + `cold`冷备：关闭mysql服务备份
    + `warm`温备：服务在线，但不能进行`DDL`&`DML`
    + `hot` 热备：服务在线，备份时业务不受影响。
        * `myISAM`不支持热备
        * `InnoDB`需要工具进行热备份
- 根据`数据集合的范围`区分备份类型
    + `full`        完全备份：备份全部`备份对象`
    + `incremental` 增量备份：上次`full backup`/`incremental backup`以来改变了的数据，备份的频率取决于数据的更新频率。
    + `differential`差异备份：上次`full backup`以来改变了的数据
- 恢复策略
    + `full`+`incremental` +二进制日志
    + `full`+`differential`+二进制日志
- `mysqldump`
- `mysqlimport`
- `mysql shell`内部导入/导出命令

