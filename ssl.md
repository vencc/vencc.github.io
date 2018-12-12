## 自认证

第一步:   生成服务端私钥和证书仓库命令

keytool -genkey -alias securechat -keysize 2048 -validity 365 -keyalg RSA -dname "CN=localhost" -keypass sNetty -storepass sNetty -keystore sChat.jks

第二步：生成服务端自签名证书

keytool -export -alias securechat -keystore sChat.jks -storepass sNetty -file sChat.cer

第三步：生成客户端的密钥对和证书仓库，用于将服务端的证书保存到客户端的授信证书仓库中

keytool -genkey -alias smcc -keysize 2048 -validity 365  -keyalg RSA -dname "CN=localhost" -keypass sNetty  -storepass sNetty -keystore cChat.jks

第四步：将服务端证书导入到客户端的证书仓库中

keytool -import -trustcacerts -alias securechat -file sChat.cer -storepass sNetty -keystore cChat.jks

---
如果你只做单向认证，则到此就可以结束了，如果是双向认证，则还需继续往下走

第五步:生成客户端自签名证书

keytool -export -alias smcc -keystore cChat.jks -storepass sNetty -file cChat.cer

最后一步:将客户端的自签名证书导入到服务端的信任证书仓库中：

keytool -import -trustcacerts -alias smcc -file cChat.cer -storepass sNetty -keystore sChat.jks

## 第三方CA认证

### 服务端证书制作

步骤1：利用OpenSSL生成CA证书：

openssl req -new -x509 -keyout ca.key -out ca.crt -days 365

步骤2：生成服务端密钥对：

keytool -genkey -alias securechat -keysize 2048 -validity 365 -keyalg RSA -dname "CN=localhost" -keypass sNetty -storepass sNetty -keystore sChat.jks

步骤3：生成证书签名请求：

keytool -certreq -alias securechat -sigalg MD5withRSA -file  sChat.csr -keypass sNetty -storepass sNetty -keystore sChat.jks

步骤4：用CA私钥进行签名：

openssl ca -in sChat.csr -out sChat.crt -cert ca.crt -keyfile ca.key -notext

步骤5：导入信任的CA根证书到keystore：

keytool -import -v -trustcacerts -alias ca_root -file ca.crt -storepass sNetty -keystore sChat.jks

步骤6：将CA签名后的server端证书导入keystore：

keytool -import -v -alias securechat -file server.crt -keypass sNetty -storepass sNetty -keystore sChat.jks

### 客户端证书制作

步骤1：生成客户端密钥对：

keytool -genkey -alias smcc -keysize 2048 -validity 365 -keyalg RSA -dname "CN=localhost" -keypass sNetty -storepass sNetty -keystore cChat.jks

步骤2：生成证书签名请求：

keytool -certreq -alias smcc -sigalg MD5withRSA -file  cChat.csr -keypass sNetty -storepass sNetty -keystore cChat.jks

步骤3：用CA私钥进行签名：

openssl ca -in cChat.csr -out cChat.crt -cert ca.crt -keyfile ca.key -notext

步骤4：导入信任的CA根证书到keystore：

keytool -import -v -trustcacerts -alias ca_root -file ca.crt -storepass sNetty -keystore cChat.jks

步骤5：将CA签名后的client端证书导入keystore：

keytool -import -v -alias smcc -file cChat.crt -keypass sNetty -storepass sNetty -keystore cChat.jks

### 证书制作过程中可能遇到的错误

> /etc/pki/CA/index.txt: No such file or directory   
> unable to open '/etc/pki/CA/index.txt'    

touch /etc/pki/CA/index.txt 

> /etc/pki/CA/serial: No such file or directory    
> error while loading serial number    

echo 00 > /etc/pki/CA/serial

> The mandatory countryName field was missing

vim /etc/pki/tls/openssl.cnf   
将   policy          = policy_match   改为  policy          = policy_anything

> failed to update database   
> TXT_DB error number 2   

rm /etc/pki/CA/index.txt
touch /etc/pki/CA/index.txt
