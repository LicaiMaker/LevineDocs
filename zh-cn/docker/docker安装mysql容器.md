## 下载mysql镜像
```shell
docker pull mysql:8.0.18
docker pull mysql
```
> 第二个下载最新版本的mysql

## 创建Mysql容器

```shell
docker run --name m1 -p 4306:3306 --net mynet --ip 172.19.0.3 -v /root/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql
```
> 这里不用使用'-it',因为不需要与mysql交互，MYSQL_ROOT_PASSWORD为mysql的root用户的密码，-e是环境参数

## 创建数据表，导入记录
使用Navicat远程连接docker里面的mysql，也可以使用命令行连接查看，mysql -h 182.92.119.50 -P 4306 -u root -p
一般情况下是可以登录的，但是可能会报错。一般有三种情况导致：1.网络问题 2.权限问题 3.防火墙问题
> 解决办法：  
- 1.网络问题
    检查端口是否正确映射等，进入这个mysql容器，在里面执行`mysql -uroot -p`如果登录成功则没问题
- 2.权限问题
    mysql用户可能不被允许远程登录，需要授权
    ```mysql
    mysql -u root -p //登录MySQL

    mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'WITH GRANT OPTION; //任何远程主机都可以访问数据库
    
    mysql> FLUSH PRIVILEGES; //需要输入次命令使修改生效
    ```
- 3.防火墙的问题
    针对不同的系统，有不同的方法，以Ubuntu为：
    ```shell
    ufw enable 
    ufw allow 4306
    ```
    但是要注意，如果使用ufw开启防火墙的话，需要将常用的80,443,22等常用端口打开，否则影响正常使用，如果因为关闭了22端口后而无法远程连接，且此时已经退出了shell，那么则无法在使用ssh命令登录linux服务器，需要到阿里云行的云主机实例中的远程登录选择vnc用户密码登录，然后再开启22端口或者关闭防火墙。
> 如果上面三个都不能解决问题，那么则需要检查阿里云的服务器(以阿里云为例，其他平台的主机同理)中是否添加了4306/4306这个地址段的安全组了，若没有，则需要添加这个安全组    


​    