迁移内网数据库到aws rds
===

> 创建副本
> RDS 配置
> 种种坑!!!


### 创建一个MySQL 副本

+ Setting the Replication Master Configuration

> [mysqld]
> log-bin=mysql-bin
> server-id=1
> `注意: master机器也是slave时  ,要考虑是否配置config   log-slave-updates = 1`

+ Setting the Replication Slave Configuration

> [mysqld]
> server-id=2

+ Creating a User for Replication

> CREATE USER 'repl_wqs'@'%' IDENTIFIED BY 'wqs'; # `密码在这里时举例用的 ,不用试了`
> GRANT all privileges ON *.* TO 'repl_wqs'@'%' IDENTIFIED BY 'wqs';
> flush privileges

+  Obtaining the Replication Master Binary Log Coordinates

> mysql> FLUSH TABLES WITH READ LOCK;
> mysql> SET GLOBAL read_only = ON;
> mysql > SHOW MASTER STATUS;       

+ 自己添加的步骤

> 为了顺利 在考试导入数据 会`	mysqldump --routines=0 --triggers=0 --events=0  --user=repl_wqs  --host=192.168.0.61 --password=wqs123123   --no-data --databases AGE --order-by-primary --single-transaction > AGE.sql`
> 直导出表结构验证一下,有的数据库会产生show table 存在 但是不在 infomation 视图中.
> `如果真发现这样的表,--ignore-table=test.fairies 略过就好`

+ Creating a Data Snapshot Using mysqldump

> mysqldump --all-databases --master-data > dbdump.db
> 注意:`库特别大,特别久远会很危险`
>      我是`mysqldump --routines=0 --triggers=0 --events=0  --user=repl_wqs  --host=192.168.0.61 --password=wqs --databases AGE  --single-transaction > AGE.sql`
>          `gzip AGE.sql` 迁移

+ 导入数据到mster

> mysql -h master < fulldb.dump

+ Setting the Master Configuration on the Slave

> mysql> CHANGE MASTER TO
>    ->     MASTER_HOST='master_host_name',
>    ->     MASTER_USER='replication_user_name',
>    ->     MASTER_PASSWORD='replication_password',
>    ->     MASTER_LOG_FILE='recorded_log_file_name',
>    ->     MASTER_LOG_POS=recorded_log_position;

+ START SLAVE

> mysql> START SLAVE;

### 亚马逊
+ 新建用户并授权
`GRANT REPLICATION CLIENT, REPLICATION SLAVE ON *.* TO 'repl_user'@'mydomain.com' IDENTIFIED BY '<password>';`
+ 开始同步

```mysql
CALL mysql.rds_set_external_master ('mymasterserver.mydomain.com', 3306,
    'repl_user', '<password>', 'mysql-bin-changelog.000001', 107, 0); 
CALL mysql.rds_start_replication;
```
+ 查看状态并停止
* 在 Amazon RDS MySQL 数据库实例上，运行 SHOW SLAVE STATUS 命令以确定副本何时与复制主体保持同步。SHOW SLAVE STATUS 命令的结果包含 Seconds_Behind_Master 字段。当 Seconds_Behind_Master 字段返回 0 时，副本将与主体保持同步。
* CALL mysql.rds_stop_replication;
