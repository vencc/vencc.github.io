Jrebel 的安装部署使用

# 本地部署，当前本地使用环境  
windows11  
idea2023.1  
maven3.8.1  
springboot3.2.0
jdk17.0.9  

**安装插件 Jrebel and XRebel**  
当前版本2024.1.2  
![image](https://github.com/vencc/vencc.github.io/assets/15951328/2f04f834-626f-45f6-bcf1-d4e00bc48cf8)  

**激活Jrebel**  
执行命令<a href="https://www.jpy.wang/page/jrebel.html">参考激活码</a>  
```
curl https://register.jpy.wang/ReRegister/src/main/java/jrebel/JrebelMain.java -o tmp.java && java tmp.java && del tmp.java
或者指定jdk
curl https://register.jpy.wang/ReRegister/src/main/java/jrebel/JrebelMain.java -o tmp.java && D:/Java/java17/jdk-17.0.9/bin/java tmp.java && del tmp.java
```
![image](https://github.com/vencc/vencc.github.io/assets/15951328/4a7de681-8e74-417e-8e33-ee9fd0e1d4f5)  
点击 Get License 按钮进行一键激活， 程序会自动虚拟设备id和环境，获取试用key并存入idea 插件的缓存目录里。  
重启 idea , 点同意即可正常jrebel插件。  
Debug With Jrebel  
![image](https://github.com/vencc/vencc.github.io/assets/15951328/d1f5bb08-9c56-4c3a-a781-fbb674d6bfab)  
通过 Ctrl + Shift + F9 即可实现重新编译热部署。  

# 服务器部署，当前使用环境  
CentOS Linux release 8.4.2105  
jdk17.0.9  

**安装Jrebel**  

[官网下载](https://www.jrebel.com/products/jrebel/download/prev-releases)  
选择和idea插件相同的版本2024.1.2  
![image](https://github.com/vencc/vencc.github.io/assets/15951328/fe15f856-32da-422c-a5b2-346fcd5b2c32)  

**激活Jrebel**  
执行命令<a href="https://www.jpy.wang/page/jrebel.html">参考激活码</a>  
```
unzip jrebel-2024.1.2-nosetup.zip
cd jrebel/bin
./activate.sh http://42.193.18.168:8088/b7c80126-2b3a-4f60-ac18-309bce1c81fa 672545172@qq.com
```
![image](https://github.com/vencc/vencc.github.io/assets/15951328/c9e5e714-840e-46cf-a31a-95ec4456088e)  
设置密码(在jrebel根目录)  
`java -jar jrebel.jar -set-remote-password <password># 例如，设置密码为 12341234java -jar jrebel.jar -set-remote-password 12341234`  

# 远程热部署  
勾选Jrebel远程热部署  
![1710305652307](https://github.com/vencc/vencc.github.io/assets/15951328/b21bb598-3a7e-4021-a704-689d7fecb068)  
生成`rebel.xml`和`rebel-remote.xml`文件  
设置远程连接  
![1710306737941](https://github.com/vencc/vencc.github.io/assets/15951328/aa6743ea-f2bd-4b00-bae6-4726c1c8e986)  
server name 随意取名称  
server url* 项目地址  
password 服务器jrebel的密码  
添加远程调试  
![image](https://github.com/vencc/vencc.github.io/assets/15951328/8a1cd951-e44b-442d-b5a3-9c3c7ae2ed97)  
debug即可  
