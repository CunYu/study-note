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