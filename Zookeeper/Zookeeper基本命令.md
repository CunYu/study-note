本文介绍了Zookeeper基本命令。内容仅供参考使用，有不足之处请及时指出，也欢迎大家交流探讨。

### 列出root下所有znode

```
[zk: zookeeper(CONNECTED) 0] ls /
[zookeeper]
```

### 创建znode

```
[zk: zookeeper(CONNECTED) 1] create /demo ""
Created /demo
[zk: zookeeper(CONNECTED) 2] ls /
[zookeeper, demo]
```

### 删除znode

```
[zk: zookeeper(CONNECTED) 3] delete /demo
[zk: zookeeper(CONNECTED) 4] ls /
[zookeeper]
```

### 查看znode

```
[zk: zookeeper(CONNECTED) 5] get /zookeeper

cZxid = 0x0
ctime = Thu Jan 01 00:00:00 GMT 1970
mZxid = 0x0
mtime = Thu Jan 01 00:00:00 GMT 1970
pZxid = 0x0
cversion = -1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 1
```

### 创建临时znode

```
[zk: zookeeper(CONNECTED) 6] create -e /temp "temp"
Created /temp
[zk: zookeeper(CONNECTED) 7] ls /
[temp, zookeeper]
```

### 设置监视点

```
[zk: zookeeper(CONNECTED) 8] stat /temp true
cZxid = 0x11
ctime = Sat Apr 13 03:45:42 GMT 2019
mZxid = 0x11
mtime = Sat Apr 13 03:45:42 GMT 2019
pZxid = 0x11
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x10000100cbf0005
dataLength = 4
numChildren = 0
[zk: zookeeper(CONNECTED) 9] delete /temp

WATCHER::

WatchedEvent state:SyncConnected type:NodeDeleted path:/temp
```

### 更改znode内容

```
[zk: zookeeper(CONNECTED) 13] create /demo ""
Created /demo
[zk: zookeeper(CONNECTED) 14] set /demo "demo"
cZxid = 0x14
ctime = Sat Apr 13 03:57:57 GMT 2019
mZxid = 0x15
mtime = Sat Apr 13 04:00:19 GMT 2019
pZxid = 0x14
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 0
[zk: zookeeper(CONNECTED) 15] get /demo
demo
cZxid = 0x14
ctime = Sat Apr 13 03:57:57 GMT 2019
mZxid = 0x15
mtime = Sat Apr 13 04:00:19 GMT 2019
pZxid = 0x14
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
```

### 监视znode的子znode/创建有序znode

```
[zk: zookeeper(CONNECTED) 16] ls /demo true
[]
[zk: zookeeper(CONNECTED) 17] create -s /demo/demo "demo"

WATCHER::

WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/demo
Created /demo/demo0000000000
[zk: zookeeper(CONNECTED) 18] ls /demo
[demo0000000000]
[zk: zookeeper(CONNECTED) 19] set /demo ""
cZxid = 0x14
ctime = Sat Apr 13 03:57:57 GMT 2019
mZxid = 0x17
mtime = Sat Apr 13 04:07:38 GMT 2019
pZxid = 0x16
cversion = 1
dataVersion = 2
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 1
```