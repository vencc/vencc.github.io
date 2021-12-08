elasticsearch 和 kibana 的版本要一致
# CentOS 安装 elaticsearch 7.16.0 
[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/7.16/rpm.html#rpm-repo)  
在 /etc/yum.repos.d/ 下创建配置文件 elasticsearch.repo  
填写以下内容：  
```  
[elasticsearch]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md
```  
执行安装命令 sudo yum install --enablerepo=elasticsearch elasticsearch  

设置账号密码
编辑 /etc/elasticsearch/elasticsearch.yml
```
```

不能使用 root 启动 elasticsearch 
```
addgroup es
adduser es
passwd es

chown -R es:es /etc/elasticsearch (根据报错给文件夹授权)
su es
./usr/share/elasticsearch/bin/elasticsearch -d
```

# CentOS 安装 kibana 7.16.0  
[官方文档]（https://www.elastic.co/guide/en/kibana/current/rpm.html）  
在 /etc/yum.repos.d/ 下创建配置文件 kibana.repo  
填写以下内容：  
```
[kibana-7.x]
name=Kibana repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```
执行安装命令 sudo yum install kibana  
启动命令 systemctl start kibana  
