## centos上关闭SELINUX服务

修改/etc/seliunx/config文件。设置SELINUX=disabled,然后重启linux即可（reboot指令）

## 安装docker服务

```shell
yum install docker -y
```

> -y 的意思是无须确认的意思
> 管理docker命令

```shell
service docker start
service docker stop
service docker restart
```

> 在Ubuntu上的安装参考；https://www.runoob.com/docker/ubuntu-docker-install.html，其中在执行`sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"`可能报错，`ModuleNotFoundError: No module named 'apt_pkg'`,报错请看：https://qiita.com/miyagaw61/items/665616d769f74158f1a1  

> 另一种使用curl一键安装，参考：https://www.jianshu.com/p/7f920ca189ce

## dockerhub

dockerhub是docker的公共镜像库https://hub.docker.com,由于国内访问DockerHub很慢，无法下载镜像文件，我们可以使用加速器daocloud.

- 配置docker加速器（DaoCloud的安装）

```shell
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
```

> 参考地址：https://www.daocloud.io/mirror#accelerator-doc
> 然后使用vim编辑/etc/docker/daemon.json,如果在文件中的registry-mirrors中的数组结尾若有逗号，去掉

## docker 命令

![docker命令](C:/Users/summer/Desktop/docker%E5%91%BD%E4%BB%A4.png)

### 镜像管理

- docker images 查看docker容器中有哪些镜像
- docker inspect xxx:版本号 查看xxx的版本的信息，docker inspect python:3.8
- docker rmi xxx:版本号，删除镜像（注意此镜像不能和其他容器关联，否则不能删除）

### 下载与上传镜像（和dockerhub交互）

- docker pull xxx:版本号 从dockerhub上下载镜像
- docker push xxx 制作镜像上传dockerhub
- docker search xxx:版本号 查询dockerhub上某个版本的镜像的详细信息

### 镜像的导入与导出

- docker save/export backup.tar.gz导出镜像
- docker load/import backup.tar.gz导入镜像

### 镜像和容器交互

- docker run -it --name=xxx bash python:3.8 使用Python3.8创建一个容器，现在使用run命令,it表示交互,这个容器启动后使用bash；退出容器命令 exit，此时容器也退出了
- docker exec -it 容器名 bash 进入已经创建的容器里面,执行命令行；退出容器命令 exit，但是，但是，但是，此时容器不会退出
- docker pause 暂停容器
- docker unpause 从暂停到运行状态
- docker start 容器名  启动容器
- docker stop 容器名 停止容器
- docker commit 将容器转为镜像
- docker inspect 容器 查看容器信息
- docker ps  查看当前运行的容器信息
- docker ps -a查看所有容器
- docker ps -l查看最新创建的容器
- docker rm 容器 删除容器（容器必须是停止状态）
- docker start/stop $(docker ps -a -q)批量启动或停止容器


## 常用命令

docker cp s d 复制命令（s和d分别代表源文件url和目的文件目录：docker cp /home/1.txt container:/root/1.txt或docker cp container:/root/1.txt /hoome/1.txt）