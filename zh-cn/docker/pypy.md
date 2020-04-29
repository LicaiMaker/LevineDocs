## 下载pypy镜像和添加pypy容器
docker pull pypy:3.6-7.2.0

```dockerfile
docker run -it -d --name p2 --net mynet --ip 172.19.0.4 -p 9600:5000 -v /root/pypy:/root/pypy pypy:3.6-7.2.0 bash
```
> 一般来说加了-d就不用-it命令了。但是在pypy中必须加上-it命令
## 进入pypy容器

```dockerfile
docker exec -it p2 bash
```
在里面查看版本，注意是使用pypy3而不是python3