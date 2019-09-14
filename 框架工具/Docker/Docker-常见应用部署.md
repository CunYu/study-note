### Mysql

``` shell
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql
```

### Redis

启动redis服务端默认暴露6379

``` shell
docker run --name redis -d redis
```

持久化启动redis服务端

``` shell
docker run --name redis -p 6379:6379  -d redis redis-server --appendonly yes
```

进入redis容器

``` shell
docker exec -it redis /bin/bash
```

进入redis客户端

``` shell
redis-cli 
```

查看redis配置

``` shell
config get *
```

退出redis客户端

``` shell
quit
```

### Centos

创建并启动centos

``` shell
docker run -d -i -t centos /bin/bash
```

退出

``` shell
exit
```

### Zookeeper

##### Zookeeper单机

启动zookeeper

``` shell
docker run --name zoo -d zookeeper
```

进入zookeeper

``` shell
docker exec -it zoo /bin/bash
```

退出zookeeper

``` shell
exit
```

启动zookeeper客户端

``` shell
docker run -it --rm --link zoo:zookeeper zookeeper zkCli.sh -server zookeeper
```

退出zookeeper客户端

``` shell
quit
```

##### Zookeeper集群

docker-compose配置文件

zookeeper.yml

``` yml
version: '3.1'

services:
  zoo1:
    image: zookeeper
    restart: always
    hostname: zoo1
    ports:
      - 2181:2181
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=0.0.0.0:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888

  zoo2:
    image: zookeeper
    restart: always
    hostname: zoo2
    ports:
      - 2182:2181
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=0.0.0.0:2888:3888 server.3=zoo3:2888:3888

  zoo3:
    image: zookeeper
    restart: always
    hostname: zoo3
    ports:
      - 2183:2181
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=0.0.0.0:2888:3888
```

部署集群

``` shell
docker-compose -f zookeeper.yml up -d
```