## docker 网络环境

- docker为容器固定ip地址
docker环境会给容器分配动态的ip地址，导致下次容器时，ip就变了。  
我们可以单独创建一个docker内部的网段（172.18.0.x）
```shell
docker network create --subnet=172.18.0.0/16 mynet
docker network rm mynet
docker run -it --net mynet --ip 172.18.0.2 python:3.8 bash
```
> 子网划分，172.18.0.0/16表示可用的ip为2^(16-1)=65536个ip地址，mynet为网段名  

> 删除网段时，要先将关联的容器删掉  

> docker run -it --name=xx --net mynet --ip 172.18.0.2 python:3.8 bash 用网段创建Python容器，并分配固定ip：172.18.0.2 注：172.18.0.1是网关地址不能使用
- 管理docker下的所有网段
如果在创建时提示：Error response from daemon: Pool overlaps with other one on this address space
则表示网段重复了，则使用一下命令查看：
```shell
docker netwoek ls
docker network inspect 网段名或者网段id
docker network rm 网段名
```

> 在容器里面查看ip：ip addr


## docker 端口映射技术
将宿主机的端口映射到容器的某个端口上  
```shell
docker run -t -p 9500:5000 python:3.8 bash
docker run -t -p 9500:5000 -p 9600:3306 python:3.8 bash
```

## docker 的目录挂载技术
docker环境只支持目录挂载，不支持文件挂载，而且一个容器可以挂载多个目录  
```shell
docker run -it -v /root/test:/root/project --name=xxx pyhton:3.8 bash
```
> /root/test表示宿主机上的文件目录，/root/project表示容器中的目录地址