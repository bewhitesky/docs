## redis安装问题

* cc1: error: unrecognized command line option "-std=c11" 报错
> sudo yum install centos-release-scl
>
> sudo yum install devtoolset-7-gcc*
>
> scl enable devtoolset-7 bash
>
> which gcc
>
> gcc --version

* springboot-redis ,yml配置
* 远程密码访问redis时，需要注释redis中的bind,以及开启requirepass
> Redis 配置
> Redis数据库索引（默认为0）
spring.redis.database=0
>
> Redis服务器地址
spring.redis.host=127.0.0.1
>
> Redis服务器连接端口
spring.redis.port=6379
>
> Redis服务器连接密码（默认为空）
spring.redis.password=
>
> 连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=8
>
> 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.pool.max-wait=-1
>
> 连接池中的最大空闲连接
spring.redis.pool.max-idle=8
>
> 连接池中的最小空闲连接
spring.redis.pool.min-idle=0
>
> 连接超时时间（毫秒）
spring.redis.timeout=1200

## oracle 服务重启
>（1） 以oracle身份登录数据库，命令：su – oracle 
>
> （2） 进入Sqlplus控制台，命令：sqlplus /nolog 
>
> （3） 以系统管理员登录，命令：connect /as sysdba 
>
> （4） 启动数据库，命令：startup 
>
> （5） 如果是关闭数据库，命令：shutdown immediate 
>
> （6） 退出sqlplus控制台，命令：exit 
>
> （7） 进入监听器控制台，命令：lsnrctl 
>
> （8） 启动监听器，命令：start 
>
> （9） 退出监听器控制台，命令：exit 
>
> （10）重启数据库结束


#### lua安装
安装步骤,在linux系统中执行下面的命令。

```properties
curl -R -O http://www.lua.org/ftp/lua-5.3.5.tar.gz
tar zxf lua-5.3.5.tar.gz
cd lua-5.3.5
make linux test
```

注意：此时安装，有可能会出现如下错误：

![1560739143890](images\1560739143890.png)

此时需要安装lua相关依赖库的支持，执行如下命令即可：

```properties
yum install libtermcap-devel ncurses-devel libevent-devel readline-devel
```

此时再执行lua测试看lua是否安装成功

```properties
[root@localhost ~]# lua
Lua 5.1.4  Copyright (C) 1994-2008 Lua.org, PUC-Rio
```

#### 安装openResty

linux安装openresty:

1.添加仓库执行命令

```shell
 yum install yum-utils
 yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo
```

2.执行安装

```
yum install openresty
```

3.安装成功后 会在默认的目录如下：

```
/usr/local/openresty
```