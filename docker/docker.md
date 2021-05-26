#### 1.1 安装fastDfs
在docker里执行docker images查看

```properties
docker pull morunchang/fastdfs
```

运行tracker

```properties
docker run -d --name tracker --net=host morunchang/fastdfs sh tracker.sh
```

运行storage

```properties
docker run -d --name storage --net=host -e TRACKER_IP=192.168.211.132:22122 -e GROUP_NAME=group1 morunchang/fastdfs sh storage.sh
```

- 使用的网络模式是–net=host, 192.168.14.132是宿主机的IP
- group1是组名，即storage的组  
- 如果想要增加新的storage服务器，再次运行该命令，注意更换 新组名



#### 1.2 配置Nginx

Nginx在这里主要提供对FastDFS图片访问的支持，Docker容器中已经集成了Nginx，我们需要修改nginx的配置,进入storage的容器内部，修改nginx.conf

```properties
docker exec -it storage  /bin/bash
```

进入后

```properties
vi /etc/nginx/conf/nginx.conf
```

添加以下内容

![1564792264719](image\1564792264719.png)

上图配置如下：

```properties
location ~ /M00 {
     root /data/fast_data/data;
     ngx_fastdfs_module;
}
```



禁止缓存：解决图片删除后，还能访问的问题

```
add_header Cache-Control no-store;
```



退出容器

```
exit
```



重启storage容器

```properties
docker restart storage
```



查看启动容器`docker ps`

```properties
9f2391f73d97 morunchang/fastdfs "sh storage.sh" 12 minutes ago Up 12 seconds storage
e22a3c7f95ea morunchang/fastdfs "sh tracker.sh" 13 minutes ago Up 13 minutes tracker
```



开启启动设置

```properties
docker update --restart=always tracker
docker update --restart=always storage
```
