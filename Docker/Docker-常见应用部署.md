本文介绍了Docker常见应用的基本使用命令。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### Mysql

```
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql
```

### Redis

启动redis服务端默认暴露6379

```
docker run --name redis -d redis
```

持久化启动redis服务端

```
docker run --name redis -p 6379:6379  -d redis redis-server --appendonly yes
```

进入redis容器

```
docker exec -it redis /bin/bash
```

进入redis客户端

```
redis-cli 
```

查看redis配置

```
config get *
```

退出redis客户端

```
quit
```

### Centos

创建并启动centos

```
docker run -d -i -t centos /bin/bash
```

退出

```
exit
```

### Zookeeper

##### Zookeeper单机

启动zookeeper

```
docker run --name zoo -d zookeeper
```

进入zookeeper

```
docker exec -it zoo /bin/bash
```

退出zookeeper

```
exit
```

启动zookeeper客户端

```
docker run -it --rm --link zoo:zookeeper zookeeper zkCli.sh -server zookeeper
```

退出zookeeper客户端

```
quit
```

##### Zookeeper集群

docker-compose配置文件

zookeeper.yml

```
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

```
docker-compose -f zookeeper.yml up -d
```