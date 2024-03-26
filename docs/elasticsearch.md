elasticsearch 和 kibana 的版本要一致
# CentOS 安装 elaticsearch 8.1.2 
[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/8.1/rpm.html#rpm-repo)  

docker 安装/启动
```
docker pull elasticsearch:8.1.2
# host 模式
docker network create elastic
# discovery.type=single-node 单节点模式运行
docker run -d --name=es1 --net elastic -p 172.19.28.74:9200:9200 -p 172.19.28.74:9300:9300 -e "discovery.type=single-node" elasticsearch:8.1.2
# 使用root用户（ID=0）登录
docker exec -u 0 -it es1 /bin/bash
```

开启https（单节点配置）  

1、生成文件  
```
# 生成 CA
./bin/elasticsearch-certutil ca
# 基于已有 CA 生成压缩包，里面有个elastic-certificates.p12 文件包含节点证书、节点密钥、CA证书
./bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
```

2、修改elasticsearch.yml
```
xpack.security.enabled: true
xpack.security.http.ssl.keystore.path: elastic-certificates.p12
xpack.security.http.ssl.truststore.path: elastic-certificates.p12
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
```

3、设置密码
```
# 添加生成证书时设置的密码，elasticsearch.yml 配置几个证书地址，就添加几个密码
./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
# 设置登录密码
./bin/elasticsearch-setup-passwords interactive
```
# CentOS 安装 kibana 8.1.2  
[官方文档](https://www.elastic.co/guide/en/kibana/8.1/rpm.html)  
 
docker 安装/启动
```
docker pull kibana:8.1.2
docker run -d --name=kibana --net elastic -p 172.19.28.74:5601:5601 kibana:8.1.2
```

连接elasticsearch https  

1、生成文件
```
./bin/elasticsearch-certutil http
```

2、修改elasticsearch.yml
```
xpack.security.http.ssl.keystore.path: http.p12
```

3、修改kibana.yml
```
elasticsearch.ssl.certificateAuthorities: config/elasticsearch-ca.pem
elasticsearch.username: 
elasticsearch.password: 
```  
# CentOS 安装 logstash 8.1.2  

docker 安装/启动
```
docker pull docker.elastic.co/logstash/logstash:8.1.2
docker run -d --name=logstash --net elastic -p 172.19.28.74:4560:4560 docker.elastic.co/logstash/logstash:8.1.2
```
修改config/logstash.yml文件  
```
http.host: "0.0.0.0"
xpack.monitoring.elasticsearch.hosts: [ "http://172.19.28.74:9200" ]
```
修改pipeline/logstash.conf文件  
```
input {
  tcp {
    mode => "server"
    host => "0.0.0.0"
    port => "4560"
    codec => json_lines
  }
}

output {
  elasticsearch {
    hosts => ["172.19.28.74:9200"]
    user => ""
    password => ""
    index => "qsadmin-%{+YYYY.MM.dd}"
  }
  stdout {
    codec => rubydebug
  }
}
```

**springboot将日志上传至logstash**  
```
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.4</version>
</dependency>
```
修改logback-spring.xml文件  
```
<appender name="logstash" class="net.logstash.logback.appender.LogstashAccessTcpSocketAppender">
    <destination>172.19.28.74:4560</destination>
    <encoder charset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder"/>
</appender>
<logger name="com" level="ERROR">
    <appender-ref ref="logstash" />
</logger>
```
