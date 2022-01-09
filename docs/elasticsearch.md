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
cluster.name: qs
node.name: master
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
# 本机ip地址（云服务器时不能使用localhost）
network.host: 172.19.28.74
discovery.seed_hosts: ["172.19.28.74"]
http.port: 9200
# 开启 elasticsearch 验证
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.license.self_generated.type: basic
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

# elaticsearch 7.16.0 配置双实例
停止 elasticsearch  
在 /etc/elasticsearch 下创建文件夹 config1 和 config2  
将 /etc/elasticsearch 下的文件 elasticsearch.yml、jvm.options、log4j2.properties 分别复制到新建的两个文件夹下  
修改其中一个 elasticsearch.yml
```
cluster.name: qs
# 另一个文件也要添加
node.max_local_storage_nodes: 2
node.name: test
path.data: /var/lib/elasticsearch2
path.logs: /var/log/elasticsearch2
# 本机ip地址（云服务器时不能使用localhost）
network.host: 172.19.28.74
discovery.seed_hosts: ["172.19.28.74"]
http.port: 9201
# 开启 elasticsearch 验证
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: certs/elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: certs/elastic-certificates.p12
xpack.license.self_generated.type: basic
```
为Elasticsearch集群创建一个证书颁发机构。
```
bin/elasticsearch-certutil ca
```
为集群中的每个节点生成证书和私钥
```
bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
```
将 elastic-certificates.p12 拷贝到 config1/certs 和 config2/certs 目录下  
yum 安装的 elasticsearch 默认配置文件由 /usr/lib/systemd/system/elasticsearch.service 指定  
启动第一个实例  
```
vim /usr/lib/systemd/system/elasticsearch.service
Environment=ES_PATH_CONF=/etc/elasticsearch/config1
vim /etc/sysconfig/elasticsearch
ES_PATH_CONF=/etc/elasticsearch/config1
```
启动第二个实例
```
vim /usr/lib/systemd/system/elasticsearch.service
Environment=ES_PATH_CONF=/etc/elasticsearch/config2
vim /etc/sysconfig/elasticsearch
ES_PATH_CONF=/etc/elasticsearch/config2
```

# CentOS 安装 kibana 7.16.0  
[官方文档](https://www.elastic.co/guide/en/kibana/current/rpm.html)  
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
