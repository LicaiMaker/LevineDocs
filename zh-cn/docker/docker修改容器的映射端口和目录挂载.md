## 修改容器的端口

找到`/var/lib/docker/container/<容器ID>/hostconfig.json`,修改里面的内容:  
```dockerfile
 "PortBindings": {
                "80/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "8080"
                    }
                ]
            },
```
修改添加都可以，比如，要添加多个端口映射到容器的80端口，可在`80/tcp的数组中添加{"HostIp": "","HostPort": "8081"}`，如果要添加映射到容器的其他端口，则在`80/tcp同级下添加配置`"443/tcp": [{"HostIp": "","HostPort": "8080"}]``

## 修改或添加容器的挂载目录
找到`/var/lib/docker/container/<容器ID>/config.v2.json`,修改里面的内容:  
```dockerfile
"MountPoints":{
    "/root/nginx":{
        "Source":"/root/nginx",
        "Destination":"/root/nginx",
        "RW":true,
        "Name":"",
        "Driver":"",
        "Type":"bind",
        "Propagation":"rprivate",
        "Spec":{
            "Type":"bind",
            "Source":"/root/nginx",
            "Target":"/root/nginx"
        },
        "SkipMountpointCreation":false
    }
}
```
如果在添加一个挂载目录，如下：

```dockerfile
"MountPoints":{
    "/root/nginx":{
        "Source":"/root/nginx",
        "Destination":"/root/nginx",
        "RW":true,
        "Name":"",
        "Driver":"",
        "Type":"bind",
        "Propagation":"rprivate",
        "Spec":{
            "Type":"bind",
            "Source":"/root/nginx",
            "Target":"/root/nginx"
        },
        "SkipMountpointCreation":false
    },
    "/root/project":{
        "Source":"/root/project",
        "Destination":"/root/project",
        "RW":true,
        "Name":"",
        "Driver":"",
        "Type":"bind",
        "Propagation":"rprivate",
        "Spec":{
            "Type":"bind",
            "Source":"/root/project",
            "Target":"/root/project"
        },
        "SkipMountpointCreation":false
    }
}
```
> source是宿主机上的路径，destination是容器里的路径

> 注： 上述步骤都要先将容器关闭，在修改文件，然后再重启docker和容器即可