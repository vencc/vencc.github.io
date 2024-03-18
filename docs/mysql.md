**mysql版本要相同**  
10.3.28-MariaDB-log  

# 主服务器  
修改配置文件  
```
vim /etc/my.cnf.d/mariadb-server.cnf
[mysqld]
log-bin=mysql-bin #二进制日志文件，master产生，slave使用进行复制操作。 
server-id=1 #给数据库服务的唯一标识，一般为大家设置服务器Ip的末尾号
# 指定同步数据库
binlog_do_db=qs
```
重启数据库  
```
show master status;
```
![image](https://github.com/vencc/vencc.github.io/assets/15951328/9eafe81b-6d14-4c8b-96ae-9e32f43e143e)  

# 从服务器  
修改配置文件  
```
vim /etc/my.cnf.d/mariadb-server.cnf
[mysqld]
log-bin=mysql-bin #二进制日志文件，master产生，slave使用进行复制操作。 
server-id=2 #给数据库服务的唯一标识，一般为大家设置服务器Ip的末尾号
```
重启数据库  
```
CHANGE MASTER TO MASTER_HOST='192.168.16.1', MASTER_PORT = 3306, MASTER_USER='sync', MASTER_PASSWORD='sync',MASTER_LOG_FILE='mysql-bin.000002',MASTER_LOG_POS=1340252;
start slave;
# 查看从库启动状态  
show slave status\G
```
![image](https://github.com/vencc/vencc.github.io/assets/15951328/3daa1f5b-c5b6-4327-b8ae-3b9c84994435)  
都为yes代表连接成功  

**从库重新连接主库** 
```
stop slave;
reset slave;
CHANGE MASTER TO MASTER_HOST='127.0.0.1', MASTER_PORT = 3306, MASTER_USER='sync', MASTER_PASSWORD='sync',MASTER_LOG_FILE='mysql-bin.000002',MASTER_LOG_POS=1340252;
start slave;
show slave status\G
```

