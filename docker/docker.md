# docker离线安装

## 下载docker镜像包

#### 1.下载环境
> 访问 https://download.docker.com/linux/static/stable/x86_64/
> 
> 下载对应版本的docker
>
>（注： centos需要内核3.10以上的版本） 
> 
```shell script
    uname -r     #查看centos下的内核版本
```

#### 2.复制tgz 包到任意路径下

```shell script
    tar -zxvf docker.tgz #解压包

    cp docker/* /usr/bin #复制解压的文件到/usr/bin目录下
```

```shell script
    vi /etc/systemd/system/docker.service  #创建docker.service 文件 （一定要在这个路径下）

    #内容
    
    [Unit]
    
    Description=Docker Application Container Engine
    
    Documentation=https://docs.docker.com
    
    After=network-online.target firewalld.service
    
    Wants=network-online.target
    
    [Service]
    
    Type=notify
    
    # the default is not to use systemd for cgroups because the delegate issues still
    
    # exists and systemd currently does not support the cgroup feature set required
    
    # for containers run by docker
    
    ExecStart=/usr/bin/dockerd
    
    ExecReload=/bin/kill -s HUP $MAINPID
    
    # Having non-zero Limit*s causes performance problems due to accounting overhead
    
    # in the kernel. We recommend using cgroups to do container-local accounting.
    
    LimitNOFILE=infinity
    
    LimitNPROC=infinity
    
    LimitCORE=infinity
    
    # Uncomment TasksMax if your systemd version supports it.
    
    # Only systemd 226 and above support this version.
    
    #TasksMax=infinity
    
    TimeoutStartSec=0
    
    # set delegate yes so that systemd does not reset the cgroups of docker containers
    
    Delegate=yes
    
    # kill only the docker process, not all processes in the cgroup
    
    KillMode=process
    
    # restart the docker process if it exits prematurely
    
    Restart=on-failure
    
    StartLimitBurst=3
    
    StartLimitInterval=60s
    
    [Install]
    
    WantedBy=multi-user.target
    
```

#### 启动服务
```shell script
    systemctl daemon-reload    #重新加载配置

    systemctl start docker    # 启动docker服务

    systemctl status docker   # 查看docker 状态

    systemctl stop docker

    systemctl restart docker
    
    docker version #查看
```

##### 输出镜像文件  save保留镜像历史
```shell script
    docker save -o nginx.tar nginx:latest
    #或
    docker save > nginx.tar nginx:latest
    #其中-o和>表示输出到文件，nginx.tar为目标文件，nginx:latest是源镜像名（name:tag）

```

##### 装载镜像 (load import)
```shell script
    docker load -i nginx.tar
    #或
    docker load < nginx.tar
    #其中-i和<表示从文件输入。会成功导入镜像及相关元数据，包括tag信息

    docker import nginx-test.tar nginx:imp    # 指定镜像名
    #或
    cat nginx-test.tar | docker import - nginx:imp
```


#### 输出镜像文件   export不保留镜像历史
```shell script
    docker export -o nginx-test.tar nginx-test
    #其中-o表示输出到文件，nginx-test.tar为目标文件，nginx-test是源容器名（name）
```

#### docker 基础命令

```shell script
    docker ps    #查看启动的镜像
    docker ps -a #查看所有装载的镜像，包括失败的
    
```

#### 安装问题
> 出现
```shell script
    Error response from daemon : OCI runtime ...... /proc/self/attr/keycreate:permission denied 
```
> 修改
```shell script
    vi /etc/selinux/config

    #修改 SELINUX=disabled
```


#### mysql镜像导入
```shell script
    docker load -i mysql.tar

    docker images

    docker run -p {容器绑定的外部ip:端口(可省略ip)}:{容器内部端口} -name mysql \-e MYSQL_ROOT_PASSWORD=1234 {镜像名称}:{镜像版本}

```

