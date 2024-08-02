# CentOS 安装 mqtt服务  

docker 安装/启动  
```
docker pull registry.cn-hangzhou.aliyuncs.com/synbop/emqttd:2.3.6
docker run --name emq -p 18083:18083 -p 1883:1883 -p 8084:8084 -p 8883:8883 -p 8083:8083 -d registry.cn-hangzhou.aliyuncs.com/synbop/emqttd:2.3.6
```

配置mqtt  

1、关闭匿名认证(默认是开启的谁都能够登录)  
```
docker exec -it mqtt /bin/sh
# 更改允许匿名 True -> false
allow_anonymous = false 
```

2、建立用户和权限的 mysql 表  
```
CREATE DATABASE mqtt charset utf8;

use mqtt;

CREATE TABLE mqtt_user ( 
id int(11) unsigned NOT NULL AUTO_INCREMENT, 
username varchar(100) DEFAULT NULL, 
password varchar(100) DEFAULT NULL, 
salt varchar(20) DEFAULT NULL, 
is_superuser tinyint(1) DEFAULT 0, 
created datetime DEFAULT NULL, 
PRIMARY KEY (id), 
UNIQUE KEY mqtt_username (username) 
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

CREATE TABLE mqtt_acl ( 
id int(11) unsigned NOT NULL AUTO_INCREMENT, 
allow int(1) DEFAULT NULL COMMENT '0: deny, 1: allow', 
ipaddr varchar(60) DEFAULT NULL COMMENT 'IpAddress', 
username varchar(100) DEFAULT NULL COMMENT 'Username', 
clientid varchar(100) DEFAULT NULL COMMENT 'ClientId', 
access int(2) NOT NULL COMMENT '1: subscribe, 2: publish, 3: pubsub', 
topic varchar(100) NOT NULL DEFAULT '' COMMENT 'Topic Filter', 
PRIMARY KEY (id) 
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

3、插入ACL规则  
```
INSERT INTO `mqtt_acl` (`id`, `allow`, `ipaddr`, `username`, `clientid`, `access`, `topic`) VALUES 
(1,1,NULL,'$all',NULL,2,'#'),
(2,0,NULL,'$all',NULL,1,'$SYS/#'),
(3,0,NULL,'$all',NULL,1,'eq #'),
(5,1,'127.0.0.1',NULL,NULL,2,'$SYS/#'),
(6,1,'127.0.0.1',NULL,NULL,2,'#'),
(7,1,NULL,'dashboard',NULL,1,'$SYS/#');
```

4、插入用户  
```
# 密码为sha256加密后的字符串
insert into mqtt_user (`username`, `password`) values ('','');
```

5、修改mysql配置  
```
auth.mysql.server = 172.19.28.74
auth.mysql.username = 
auth.mysql.password = 
auth.mysql.database = mqtt
auth.mysql.password_hash = sha256
```

6、重启  
```
/opt/emqttd/bin/emqttd stop
/opt/emqttd/bin/emqttd start
/opt/emqttd/bin/emqttd_ctl plugins load emq_auth_mysql   #开启mysql认证插件
```
