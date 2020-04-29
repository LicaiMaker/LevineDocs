## 完整的Python容器创建命令
- 新建挂载目录(也可以用已有的目录)
现在宿主机上的/root目录下新建文件夹project，用于目录挂载
```shell
cd /root
mkdir project
```
- 创建Python目录
```shell
docker run -it -d --name=p1 -p 9500:5000 -v /root/project:/root/project --net mynet --ip 172.19.0.2 python:3.8 bash
```
> -d 表示在容器中输入exit不会让着停止，守护进程的意思

- 查看容器情况：
```shell
docker ps -a
```
- 进入容器的bash环境
```shell
docker exec -it p1 bash
```

## 加速安装pip 安装扩展程序
进入Python容器后，安装Python扩展程序
```shell
pip install flask -i https://pypi.tuna.tsinghua.edu.cn/simple
```
> 使用清华大学的pip库,安装更快