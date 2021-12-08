CentOS 安装 elaticsearch  
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
启动命令 systemctl start elasticsearch.service  
