RedHat7离线部署开发环境
=================
需提前准备的

| 包|对应文中标题|
| -----------|:---------:|  
|安装gcc的rpm包|安装gcc编译环境|
|安装gcc-c++的rpm包| 安装gcc-c++环境|
|安装redis的tar.gz包| 部署redis|
|安装mysql的rpm包| 部署数据库（举例为`mysql`）|
|安装JDK的tar.gz包| 安装JDK|
|安装tomcat的tar.gz包| 部署tomcat|
|安装gitlab的rpm包| 部署gitlab|
|安装git的rpm包|部署Jenkins|
|安装maven的tar.gz包|部署Jenkins|
|安装Jenkins的war包|部署Jenkins|
|安装Jenkins插件的hpi包|部署Jenkins|

# 环境准备

查看redhat版本，去下载对应版本的rpm包
```执行sell命令
cat /etc/redhat-release
```

如有类似下面输出，可按下文rpm的版本走
```
Red Hat Enterprise Linux Server release 7.2 (Maipo)
```
否则，建议下载与部署环境版本一致的iso镜像文件，自行安装虚拟系统，并以“可移动设备 -> CD/DVD”挂载，并输入
```
cd /run/media/root/RHEL-7.2 Server.x86_64/Packages/
```
取出所需的rpm包。

**如果找不到一致版本的iso版本的文件，则找个相近版本的iso，版本宜低不宜高，然后像上述操作一样取出rpm包，保持依赖一致性**

推荐几个网站：
[网易开源镜像站](http://mirrors.cn99.com/)
[检测rpm依赖项所在的rpm包名](https://pkgs.org/)

## 安装gcc编译环境

```执行sell命令
gcc -v 
```
没有则安装,安装顺序如标号

|顺序|命令|
| -----------|:---------:|  
| 7|	rpm -ivh gcc-4.8.5-4.el7.x86_64.rpm|
| 6|	rpm -ivh cpp-4.8.5-4.el7.x86_64.rpm|
| 5|	rpm -ivh libmpc-1.0.1-3.el7.x86_64.rpm|
| 4|	rpm -ivh mpfr-3.1.1-4.el7.x86_64.rpm |
| 3|	rpm -ivh glibc-devel-2.17-105.el7.x86_64.rpm|
| 2|	rpm -ivh glibc-headers-2.17-105.el7.x86_64.rpm|
| 1|	rpm -ivh kernel-headers-3.10.0-327.el7.x86_64.rpm|
  
验证是否安装成功
```执行sell命令
gcc -v 
```
  
## 安装gcc-c++环境

```执行sell命令
g++ -v 
```
没有则安装,安装顺序如标号

|顺序|命令|
| -----------|:---------:|  
| 2|	rpm -ivh gcc-c++-4.8.5-4.el7.x86_64.rpm|
| 1|	rpm -ivh libstdc++-devel-4.8.5-4.el7.x86_64.rpm|

## 部署redis
[下载redis源码包](https://github.com/antirez/redis/releases)，这里选择`redis-4.0.1.tar.gz`这个版本

依次输入如下命令：

解压
```
tar -zxvf redis-4.0.1.tar.gz
```

创建安装目录，并编译
```
mkdir /usr/local/redis
cd redis-4.0.1
make PREFIX=/usr/local/redis install
```

复制配置文件至启动目录
```
cp redis.conf /usr/local/redis
cd /usr/local/redis
```

修改配置文件
```
vi ./redis.conf
```
+ 注释bind 127.0.0.1
+ 修改protected-mode yes为protected-mode no
+ 修改daemonize no为daemonize yes //设置保持后台启动


启动
```
cd /usr/local/redis
./bin/redis-server ./redis.conf
```

测试redis是否安装成功
```
cd /usr/local/redis/bin
./redis-cli
```

进入redis命令行，输入测试
```
127.0.0.1:6379> set wpy hello!
OK
127.0.0.1:6379> get wpy
"hello!"
127.0.0.1:6379> quit
```

设置防火墙，在“配置”那里“运行时”和“永久”都设置打开6379端口，或在命令行输入
```
#--permanent永久生效，没有此参数重启后失效
firewall-cmd --zone=public --add-port=6379/tcp --permanent
#重新载入 
firewall-cmd --reload
#查看
firewall-cmd --zone=public --query-port=6379/tcp
#删除，以备设错端口移除
firewall-cmd --zone=public --remove-port=6379/tcp --permanent
```

可在其他ip地址测试连接redis，不展开

## 部署数据库

这要看具体要求，这里提供`mysql`的部署步骤，选择[mysql5.7版本](https://dev.mysql.com/downloads/mysql/5.7.html#downloads)

下载
+ mysql-community-common-5.7.24-1.el7.x86_64.rpm
+ mysql-community-libs-5.7.24-1.el7.x86_64.rpm
+ mysql-community-client-5.7.24-1.el7.x86_64.rpm
+ mysql-community-server-5.7.24-1.el7.x86_64.rpm

删除原系统自带的`mariadb`，如有其他版本的`mysql`一样处理
```
rpm -qa |grep mariadb
#上面查出有安装，则执行卸载
rpm -e mariadb-libs-5.5.44-2.el7.x86_64 --nodeps
...
```

执行安装 common>libs>client>server
```
rpm -ivh mysql-community-common-5.7.24-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-5.7.24-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-5.7.24-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-5.7.24-1.el7.x86_64.rpm
```

修改配置文件`my.cnf`
```
vim /etc/my.cnf
#中文乱码，在[mysqld]下面，添加
character_set_server=utf8

#设置mysql忽略大小写，在[mysqld]下面，添加
lower_case_table_names=1

#其他修改（可选），在[mysqld]下面，添加
skip-external-locking
key_buffer_size = 16M
max_allowed_packet = 1M
table_open_cache = 64
sort_buffer_size = 512K
net_buffer_length = 8K
read_buffer_size = 256K
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 8M
```

启动，并获取随机密码
```
service mysqld start
cat /var/log/mysqld.log | more
# [Note] A temporary password is generated for root@localhost: ,-:Y-vq,V9EF
```
这个`,-:Y-vq,V9EF`，就是`root`用户登录密码
```
mysql -u root -p
```

查询上述配置修改，是否生效
```
show variables like 'character%'
```

设置密码策略
```
set global validate_password_policy=0;
set global validate_password_length=1;
```

修改`root`密码，并授权用户远程访问
```
alter user 'root'@'localhost' IDENTIFIED BY 'handhand';
grant all privileges on *.* to 'root'@'%' identified by 'handhand';
```

创建用户`user_name`，并授权
```
CREATE USER user_name@'%' IDENTIFIED BY 'user_password';
CREATE USER user_name@'localhost' IDENTIFIED BY 'user_password';
GRANT ALL PRIVILEGES ON user_name.* TO user_name@'localhost'; 
GRANT ALL PRIVILEGES ON user_name.* TO user_name@'%'; 
```

执行完，如上所有操作后
```
flush privileges;
exit
```

设置防火墙，开启`3306`端口，图形化界面直接进入`防火墙配置`，在`public`区域，勾选`服务`页下的`mysql`，或者在命令行输入
```
firewall-cmd --zone=public --add-port=3306/tcp --permanent
#重新载入 
firewall-cmd --reload
#查看
firewall-cmd --zone=public --query-port=3306/tcp
#删除，以备设错端口移除
firewall-cmd --zone=public --remove-port=3306/tcp --permanent
```

在其他ip地址，测试连接成功后，创建数据库`database`
```
#root下新建数据库：
create schema database default character set utf8; 
#root赋权，将上面新建的database的权限全部赋予用户user_name
GRANT ALL PRIVILEGES ON database.* TO user_name@'%'; 
GRANT ALL PRIVILEGES ON hap_dev.* TO user_name@'localhost'; 
flush privileges;
```

## 安装JDK
根据需要，[下载对应版本的tar.gz或rpm]((https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html))，这里版本选择`jdk1.8.0_181`

先卸载系统自带jdk，再安装
```
#查看本机安装的jdk
rpm -qa | grep jdk
#卸载
rpm -e java-1.8.0-openjdk-1.8.0.65-3.b17.el7.x86_64 --nodeps
rpm -e java-1.8.0-openjdk-headless-1.8.0.65-3.b17.el7.x86_64
```

解压，修改配置文件`profile`添加系统环境变量，`/home/jdk1.8.0_181`是jdk解压后的目录路径

```
tar -zxvf jdk-8u181-linux-x64.tar.gz
vi /etc/profile
#在文件底部添加，如下内容
#set java environment
export JAVA_HOME=/home/jdk1.8.0_181
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```
保存后，刷新`profile`，再验证`jdk1.8.0_181`是否配置成功
```
source /etc/profile
java -version
javac -version
```

## 部署tomcat
根据需要，选择对应的`core`版本[下载](http://tomcat.apache.org/download-80.cgi)，这里选择`apache-tomcat-8.5.33.tar.gz`

解压  
```
tar zxvf apache-tomcat-8.5.33.tar.gz
```

修改`server.xml`，性能优化和编码
```
vi /home/apache-tomcat-8.5.33/conf/server.xml
```
><Connector **port="8081"** protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" **maxThreads="600"**
	   **URIEncoding="UTF-8"**
/>

修改`catalina.properties`，取消启动前的jar包扫描，提升启动速度，减少内存占用
```
vi /home/apache-tomcat-8.5.33/conf/catalina.properties
tomcat.util.scan.StandardJarScanFilter.jarsToSkip=*.jar
```

修改`catalina.sh`，在开始处添加
```
vi /home/apache-tomcat-8.5.33/bin/catalina.sh
JAVA_OPTS='-Xms512m -Xmx2048m'
```

在`context.xml`，配置数据源，作为演示，用上面创建的mysql数据库为例
```
vi /home/apache-tomcat-8.5.33/conf/context.xml
#在<Context></Context>间，添加如下内容
<Resource auth="Container" driverClassName="com.mysql.jdbc.Driver" url="jdbc:mysql://127.0.0.1:3306/database?allowMultiQueries=true" name="jdbc/database" type="javax.sql.DataSource" username="user_name" password="user_passwork"/>
```

启动`tomcat`
```
/home/apache-tomcat-8.5.33/bin/startup.sh
```

设置防火墙，开启`8081`端口，图形化界面直接进入`防火墙配置`，在`public`区域，打开`端口`页，在其下方添加的`8081`的`tcp`端口，或者在命令行输入
```
firewall-cmd --zone=public --add-port=8081/tcp --permanent
#重新载入 
firewall-cmd --reload
#查看
firewall-cmd --zone=public --query-port=8081/tcp
#删除，以备设错端口移除
firewall-cmd --zone=public --remove-port=8081/tcp --permanent
```

在其他ip地址，用浏览器访问`http:/192.168.XX.XX:8081`，

停止`tomcat`
```
/home/apache-tomcat-8.5.33/bin/shutdown.sh
#若报错，停止失败，则用kill进程方式关闭，下面`12345`是进程id
ps -ef |grep tomcat
kill -9 12345
```

## 部署gitlab
检查依赖环境是否安装，没有则自行安装
```
rpm -qa curl
rpm -qa policycoreutils-python
rpm -qa openssh-server
rpm -qa postfix
```
启动并修改`sshd`和`postfix`服务自启动，设置防火墙
```
systemctl enable sshd
systemctl start sshd
systemctl enable postfix
systemctl start postfix
firewall-cmd --permanent --add-service=http
systemctl reload firewalld
```

选择所需版本[下载](https://packages.gitlab.com/gitlab/gitlab-ce/packages/el/7/gitlab-ce-10.5.2-ce.0.el7.x86_64.rpm)，这里选择`gitlab-ce-10.5.2-ce.0.el7.x86_64.rpm`

安装前提：**4G**空闲运行内存

安装，配置Gitlab的外部URL
```
rpm -ivh gitlab-ce-10.5.2-ce.0.el7.x86_64.rpm
vi /etc/gitlab/gitlab.rb
external_url "http://ip:端口" #如http://192.168.XX.XX:8888
```
重配置并启动`gitlab`
```
gitlab-ctl reconfigure
gitlab-ctl restart
```
[使用gitlab-ctl reconfigure命令报错](https://gitlab.com/gitlab-org/omnibus-gitlab/issues/3937)

访问，若报502，权限问题或者内存不足
```
chmod -R 755 /var/log/gitlab 
gitlab-ctl reconfigure
gitlab-ctl restart
```

访问成功后，设置`root`用户的密码

登录后，在设置栏`User Settings`的`Emails`设置你的邮箱`wpy@example.com`

将已有项目部署到`gitlab`上：

1. 在开发环境，安装`git`，安装过程不过多阐述，打开`Git Bash`，输入
   ```
   ssh-keygen -t rsa -C "wpy@example.com"
   ```

2. 生成`key`，复制生成的公钥`id_rsa.pub`内容，到`gitlab`设置`User Settings`的`SSH Keys`上
   ```
   cat ~/.ssh/id_rsa.pub
   ```

3. 在`Gitlab`上创建一个空项目`helloGitlab`

4. 打开开发环境的`Git Bash`，配置全局的`user.name`和`user.email`
   ```
   git config --global user.name "root"
   git config --global user.email "wpy@example.com"
   ```
   
5. cd到你需要导入的项目目录下，再执行提交命令
   ```
   git init
   git remote add origin git@192.168.XX.XX:root/helloGitlab.git
   git add .
   git commit -m "[ADD]helloGitlab_v1.0_201811111111"
   git push -u origin master
   ```

回到`gitlab`，刷新`helloGitlab`项目页，会出现刚才提交的项目文件

在IDEA上，clone项目，这部分不多阐述

## 部署Jenkins
根据所需选择对应版本[下载](http://mirrors.jenkins.io/war/latest/)，这里下载截至目前最新（2019-03-04 03:08）`jenkins.war`版本

可以`java -jar`命令启动，也可以选择直接放在`tomcat`的`webapp`目录下管理启动停止

在运行状态可以通过浏览器输入下面链接来重启`jenkins`
```
localhost:8080/jenkins/restart
```

启动成功后，复制生成的随机密码
```
cat /root/.jenkins/secrets/initialAdminPassword
```

访问`jenkins`服务，使用前的配置
1. 将复制的随机密码输入，进入下一步
2. 选择手动安装插件
3. 

安装配置**自动构建web项目**需要的依赖

|名|版本+|
| -----------|:---------:| 
| git| git-1.8.3.1-5.el7.x86_64.rpm|
| maven| apache-maven-3.3.9|

安装`git`

|顺序|命令|
| -----------|:---------:| 
|1| rpm -ivh perl-Error-0.17020-2.el7.noarch.rpm|
|2| rpm -ivh perl-Git-1.8.3.1-5.el7.noarch.rpm --nodeps|
|3| rpm -ivh perl-TermReadKey-2.30-20.el7.x86_64.rpm|
|4| rpm -ivh git-1.8.3.1-5.el7.x86_64.rpm|

安装`maven`，[下载](http://maven.apache.org/download.cgi)，解压并配置系统环境变量
```
tar -zxvf apache-maven-3.3.9-bin.tar.gz
vi /etc/profile
export MAVEN_HOME=/home/apache-maven-3.3.9
export MAVEN_OPTS="-Xms256m -Xmx512m"
export PATH=$MAVEN_HOME/bin:$PATH
source /etc/profile
mvn -v
```

将提前准备好web项目所需的jar包，导入maven库`.m2`文件夹内

安装配置**自动构建web项目**需要的插件，[下载](http://updates.jenkins-ci.org/download/plugins/)与`jenkins.war`对应版本插件，这里全部下载最新

插件清单
```
publish-over-ssh

```

`jenkins`的系统配置

`jenkins`的全局安全配置

构建第一个`jenkins`工程

## 部署onlyoffice
不属于开发必须项，不多阐述
## 部署fastdfs
不属于开发必须项，不多阐述
