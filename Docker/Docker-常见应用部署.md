本文介绍了Docker常见应用的基本使用命令。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### mysql

```
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql
```

### redis

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

### centos

创建并启动centos

```
docker run -d -i -t centos /bin/bash
```

退出

```
exit
```