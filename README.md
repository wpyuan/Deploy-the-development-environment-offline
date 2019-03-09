RedHat7离线部署开发环境
=================

# 环境准备

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
  
2. gcc-c++环境

```执行sell命令
g++ -v 
```
没有则安装,安装顺序如标号

|顺序|命令|
| -----------|:---------:|  
| 2|	rpm -ivh gcc-c++-4.8.5-4.el7.x86_64.rpm|
| 1|	rpm -ivh libstdc++-devel-4.8.5-4.el7.x86_64.rpm|

3. 部署redis
