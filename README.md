# mysql
Master slave replication of MySQL in Docker
## docker MySQL 8.0
### 主机
新建 mysql-master 容器
```shell
docker run -d -p 3306:3306 -v /xxj/mysql/master/log:/var/log/mysql  -v  /xxj/mysql/master/data:/var/lib/mysql  -v  /xxj/mysql/master/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=123456 --name mysql-master mysql
```
主机 my.cnf
```shell
[mysqld]
server-id=1
# 启用二进制日志
log-bin=mysql-bin
binlog-cache-size=4M
# mixed,statement,row
binlog-format=mixed
binlog-expire-logs-seconds = 2592000
read-only=0
# 设置不要复制的数据库-可选
binlog-ignore-db=mysql 
# 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断，比如1062是指一些主键重复错误
slave-skip-errors=1062
```
建立可访问主机数据的账号，配置权限
```shell
CREATE USER 'slave'@'%' IDENTIFIED WITH mysql_native_password BY '1234556';

GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';

FLUSH PRIVILEGES;

show master status;
```

### 从机
新建 mysql-slave 容器
```shell
docker run -d -p 3307:3306 -v /xxj/mysql/slave/log:/var/log/mysql  -v  /xxj/mysql/slave/data:/var/lib/mysql  -v  /xxj/mysql/slave/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=123456 --name mysql-slave mysql
```
从机 my.cnf
```shell
[mysqld]
server-id=2
log-bin=slave-mysql-bin 
binlog-cache-size=4M 
binlog-format=mixed  
binlog-expire-logs-seconds = 2592000
read-only=1
binlog-ignore-db=mysql 
slave-skip-errors=1062  
# relay_log配置中继日志
relay-log=relay-mysql-bin
# slave将复制事件写进自己的二进制日志
log-slave-updates=1  
```
从机中设置master信息
```shell
change replication source to source_host='192.168.2.2',source_user='slave',source_password='123456',source_port=3306,source_log_file='mysql-bin.000001',source_log_pos=848;

start replica;

show replica status \G
```
