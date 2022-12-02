elasticsearch 和 kibana 的版本要一致
# CentOS 安装 elaticsearch 7.16.0 
[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/7.16/rpm.html#rpm-repo)  

docker 安装/启动
```
docker pull docker.elastic.co/elasticsearch/elasticsearch:7.16.0
docker run -d --name=es1 -p 172.19.28.74:9200:9200 -p 172.19.28.74:9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.16.0
```

开启https  
1、生成文件  
```
# 生成 CA
./bin/elasticsearch-certutil ca
# 基于已有 CA 生成压缩包，里面有个elastic-certificates.p12 文件包含节点证书、节点密钥、CA证书
./bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
```
elasticsearch.yml
```
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
```
# CentOS 安装 kibana 7.16.0  
[官方文档](https://www.elastic.co/guide/en/kibana/current/rpm.html)  
 
docker 安装/启动
```
docker pull docker.elastic.co/kibana/kibana:7.16.0
docker run -d --name=kibana -p 172.19.28.74:5601:5601 docker.elastic.co/kibana/kibana:7.16.0
```
