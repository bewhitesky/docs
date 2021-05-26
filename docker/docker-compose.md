# docker-compose 安装

> 下载 https://github.com/docker/compose/releases 对应版本

```shell script
    #放置任意路径
    mv docker-compose-Linux-x86_64  docker-compose  # 换个名字

    chmod +x docker-compose  # 赋权

    docker-compose --version  #查看版本

    docker-compose -f 

    docker-compose up -d nginx                    # 构建建启动nignx容器
    
    docker-compose exec nginx bash           # 登录到nginx容器中
    
    docker-compose down                             # 删除所有nginx容器,镜像
    
    docker-compose ps                                  # 显示所有容器
    
    docker-compose restart nginx                  # 重新启动nginx容器
    
    docker-compose run --no-deps --rm php-fpm php -v  #在php-fpm中不启动关联容器，并容器执行php -v 执行完成后删除容器
    
    docker-compose build nginx                    # 构建镜像 。        
    
    docker-compose build --no-cache nginx   #不带缓存的构建。
    
    docker-compose logs  nginx                     #查看nginx的日志 
    
    docker-compose logs -f nginx                   #查看nginx的实时日志
    
     
    
    docker-compose config  -q                      #验证（docker-compose.yml）文件配置，当配置正确时，不输出任何内容，当文件配置错误，输出错误信息。 
    
    docker-compose events --json nginx      # 以json的形式输出nginx的docker日志
    
    docker-compose pause nginx               #  暂停nignx容器
    
    docker-compose unpause nginx            # 恢复ningx容器
    
    docker-compose rm nginx                     #  删除容器（删除前必须关闭容器）
    
    docker-compose stop nginx                   # 停止nignx容器
    
    docker-compose start nginx                    #启动nignx容器

```

#### 配置yaml文件
```yaml
#官方示例

version: "3.7"
services:

  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]

  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - "5000:80"
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure

  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - "5001:80"
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]

networks:
  frontend:
  backend:

volumes:
  db-data:
```

# 针对yaml文件内容的解析：

##服务基于已经存在的镜像
services:
  web:
    image: hello-world


##服务基于dockerfile 
build: /path/to/build/dir
build: ./dir
build:
  context: ../
  dockerfile: path/of/Dockerfile

build: ./dir
image: webapp:tag


##command 
command命令可以覆盖容器启动后默认执行的命令
command: bundle exec thin -p 3000
command: [bundle, exec, thin, -p, 3000]

##container_name
Compose 的容器名称格式是：<项目名称><服务名称><序号>
虽然可以自定义项目名称、服务名称，但是如果你想完全控制容器的命名，可以使用这个标签指定
container_name: app

##depends_on
depends_on解决了容器的依赖、启动先后的问题
version: '2'
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:


##dns
dns: 8.8.8.8
dns:
  - 8.8.8.8
  - 9.9.9.9

dns_search: example.com
dns_search:
  - dc1.example.com
  - dc2.example.com

##tmfs
挂载临时目录到容器内部，与run的参数一样效果
tmpfs: /run
tmpfs:
  - /run
  - /tmp

##environment
设置镜像变量，它可以保存变量到镜像里，也就是说启动的容器也会包含这些变量设置
environment:
  RACK_ENV: development
  SHOW: 'true'
  SESSION_SECRET:
 
environment:
  - RACK_ENV=development
  - SHOW=true
  - SESSION_SECRET
 
##expose
用于指定暴露的端口，但是只是作为参考，端口映射的话还得ports标签
expose:
 - "3000"
 - "8000"

##external_links
在使用Docker的过程中，我们会有许多单独使用docker run启动的容器，为了使Compose能够连接这些不在docker-compose.yml中定义的容器，我们需要一个特殊的标签，就是external_links，它可以让Compose项目里面的容器连接到那些项目配置外部的容器（前提是外部容器中必须至少有一个容器是连接到与项目内的服务的同一个网络里面）
 
external_links:
 - redis_1
 - project_db_1:mysql
 - project_db_1:postgresql

##extra_hosts
添加主机名的标签，就是往容器内部/etc/hosts文件中添加一些记录
extra_hosts:
 - "somehost:162.242.195.82"
 - "otherhost:50.31.209.229"

##labels
向容器添加元数据，和Dockerfile的lable指令一个意思

labels:
  com.example.description: "Accounting webapp"
  com.example.department: "Finance"
  com.example.label-with-empty-value: ""
labels:
  - "com.example.description=Accounting webapp"
  - "com.example.department=Finance"
  - "com.example.label-with-empty-value"
 
##links
解决容器连接问题，与docker的–link一样的效果，会连接到其他服务中的容器,使用的别名将会自动在服务容器中的/etc/hosts里创建
links:
 - db
 - db:database
 - redis


##ports
用作端口映射
使用HOST:CONTAINER格式或者只是指定容器的端口，宿主机会随机映射端口
ports:
 - "3000"
 - "8000:8000"
 - "49100:22"
 - "127.0.0.1:8001:8001"
当使用HOST:CONTAINER格式来映射端口时，如果你使用的容器端口小于60你可能会得到错误得结果，因为YAML将会解析xx:yy这种数字格式为60进制。所以建议采用字符串格式

##security_opt
为每个容器覆盖默认的标签。简单说来就是管理全部服务的标签，比如设置全部服务的user标签值为USER
security_opt:
  - label:user:USER
  - label:role:ROLE

##volumes
挂载一个目录或者一个已经存在的数据卷容器，可以直接使用[HOST:CONTAINER]这样的格式，或者使用[HOST:CONTAINER:ro]这样的格式，或者对于容器来说，数据卷是只读的，这样可以有效保护宿主机的文件系统。
compose的数据卷指定路径可以是相对路径，使用 . 或者 … 来指定性对目录
 
volumes:
  // 只是指定一个路径，Docker 会自动在创建一个数据卷（这个路径是容器内部的）。
  - /var/lib/mysql
 
  // 使用绝对路径挂载数据卷
  - /opt/data:/var/lib/mysql
 
  // 以 Compose 配置文件为中心的相对路径作为数据卷挂载到容器。
  - ./cache:/tmp/cache
 
  // 使用用户的相对路径（~/ 表示的目录是 /home/<用户目录>/ 或者 /root/）。
  - ~/configs:/etc/configs/:ro
 
  // 已经存在的命名的数据卷。
  - datavolume:/var/lib/mysql
 
如果你不使用宿主机的路径，你可以指定一个volume_driver。
volume_driver: mydriver

##volumes_from
从其它容器或者服务挂载数据卷，可选的参数是:ro或者:rw，前者表示容器只读，后者表示容器对数据卷是可读可写的，默认是可读可写的

volumes_from:
  - service_name
  - service_name:ro
  - container:container_name
  - container:container_name:rw

##network_mode
网络模式，与docker client的–net参数类似，只是相对多了一个service:[sevice name]的格式
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
 
##networks
加入指定网络
services:
  some-service:
    networks:
     - some-network
     - other-network