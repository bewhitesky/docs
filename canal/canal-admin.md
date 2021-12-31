## 准备
canal-admin的限定依赖：

   1. MySQL，用于存储配置和节点等相关数据
   2. canal版本，要求>=1.1.4 (需要依赖canal-server提供面向admin的动态运维管理接口)

## 部署
   1. 下载 canal-admin, 访问 release 页面 , 选择需要的包下载, 如以 1.1.4 版本为例
       ```shell script
       wget https://github.com/alibaba/canal/releases/download/canal-1.1.4/canal.admin-1.1.4.tar.gz
      
       内网，自己手动下载release包
        ```
   2. 解压缩
        ```shell script
        mkdir /tmp/canal-admin
        tar zxvf canal.admin-$version.tar.gz  -C /tmp/canal-admin
        ```

        解压完成后，进入 /tmp/canal 目录，可以看到如下结构
        ```shell script
        drwxr-xr-x   6 agapple  staff   204B  8 31 15:37 bin
        drwxr-xr-x   8 agapple  staff   272B  8 31 15:37 conf
        drwxr-xr-x  90 agapple  staff   3.0K  8 31 15:37 lib
        drwxr-xr-x   2 agapple  staff    68B  8 31 15:26 logs
        ```
    
   3.配置修改
```shell script
vi conf/application.yml
```
```yaml
server:
  port: 8089
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8

spring.datasource:
  address: 127.0.0.1:3306
  database: canal_manager
  username: canal
  password: canal
  driver-class-name: com.mysql.jdbc.Driver
  url: jdbc:mysql://${spring.datasource.address}/${spring.datasource.database}?useUnicode=true&characterEncoding=UTF-8&useSSL=false
  hikari:
    maximum-pool-size: 30
    minimum-idle: 1

canal:
  adminUser: admin
  adminPasswd: admin
```
    
    
   4.初始化元数据库
```shell script
mysql -h127.1 -uroot -p

# 导入初始化SQL
> source conf/canal_manager.sql
```

a. 初始化SQL脚本里会默认创建canal_manager的数据库，建议使用root等有超级权限的账号进行初始化 b. canal_manager.sql默认会在conf目录下，也可以通过链接下载 canal_manager.sql

   5.启动
```shell script
sh bin/startup.sh

```
查看 admin 日志
```shell script
vi logs/admin.log

2019-08-31 15:43:38.162 [main] INFO  o.s.boot.web.embedded.tomcat.TomcatWebServer - Tomcat initialized with port(s): 8089 (http)
2019-08-31 15:43:38.180 [main] INFO  org.apache.coyote.http11.Http11NioProtocol - Initializing ProtocolHandler ["http-nio-8089"]
2019-08-31 15:43:38.191 [main] INFO  org.apache.catalina.core.StandardService - Starting service [Tomcat]
2019-08-31 15:43:38.194 [main] INFO  org.apache.catalina.core.StandardEngine - Starting Servlet Engine: Apache Tomcat/8.5.29
....
2019-08-31 15:43:39.789 [main] INFO  o.s.w.s.m.m.annotation.ExceptionHandlerExceptionResolver - Detected @ExceptionHandler methods in customExceptionHandler
2019-08-31 15:43:39.825 [main] INFO  o.s.b.a.web.servlet.WelcomePageHandlerMapping - Adding welcome page: class path resource [public/index.html]
```


> 此时代表canal-admin已经启动成功，可以通过 http://127.0.0.1:8089/ 访问，默认密码：admin/123456 

   6.关闭
```shell script
sh bin/stop.sh

```
   7.canal-server端配置
使用canal_local.properties的配置覆盖canal.properties
```shell script
# register ip
canal.register.ip =

# canal admin config
canal.admin.manager = 127.0.0.1:8089
canal.admin.port = 11110 
canal.admin.user = admin
canal.admin.passwd = 4ACFE3202A5FF5CF467898FC58AAB1D615029441
# admin auto register
canal.admin.register.auto = true
canal.admin.register.cluster =
```

启动admin-server即可。(到bin路径下启动)

或在启动命令中使用参数：sh bin/startup.sh local 指定配置

- 控制台地址
    - http://[ip]:8089
    - 账号/密码： admin/123456
 
- 链接
    - https://github.com/alibaba/canal/wiki/Canal-Admin-QuickStart
- 问题
    > 在C网的服务器中在canal-admin修改instance的position是不会直接生效的，该配置文件只能修改instance的配置。
    
    > server获取偏移量的值是优先获取meta.dat里的内容，如果存在这个文件，则直接获取，不存在，才会从instance的配置里获取偏移量的值
                                                                                       
    - **所以在修改偏移量的操作，需要先停掉服务，然后删掉meta.dat 和h2.mv.db文件，最后启动server服务**
       