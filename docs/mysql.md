# 主服务器  
```
show master status;
```
![image](https://github.com/vencc/vencc.github.io/assets/15951328/9eafe81b-6d14-4c8b-96ae-9e32f43e143e)

# 从服务器  
```
CHANGE MASTER TO MASTER_HOST='127.0.0.1', MASTER_PORT = 3306, MASTER_USER='sync', MASTER_PASSWORD='sync',MASTER_LOG_FILE='mysql-bin.000002',MASTER_LOG_POS=1340252;
start slave;
# 查看从库启动状态  
show slave status\G
```
![image](https://github.com/vencc/vencc.github.io/assets/15951328/3daa1f5b-c5b6-4327-b8ae-3b9c84994435)
都为yes代表连接成功  

