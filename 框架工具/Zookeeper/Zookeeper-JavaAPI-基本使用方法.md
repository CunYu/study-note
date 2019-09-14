### 会话

Zookeeper客户端和Zookeeper服务端之间的通信需要建立在会话的基础上，当会话在设置的超时时间内Zookeeper服务端没有收到任何该会话的信息，那么该会话会被关闭。

### 依赖

``` groovy
compile("org.apache.zookeeper:zookeeper:3.4.14")
```

### Watch

Watch定义了Zookeeper中用于接收Zookeeper会话事件的对象的类型，Watch是一个接口，每当有Zookeeper会话事件发生时，会调用Watch的process方法。

``` java
public class DemoWatcher implements Watcher {

    @Override
    public void process(WatchedEvent event) {
        System.out.println(event);
    }
}
```

### 基本操作

* 同步基本操作

``` java
ZooKeeper zookeeper = new ZooKeeper("192.168.99.100:2181,192.168.99.100:2182,192.168.99.100:2183", 1000000, new DemoWatcher());
// 创建临时节点
zookeeper.create("/demo", "demo".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);

// 查看节点 指定监视对象 当节点数据变化时会发送通知
Stat stat = new Stat();
byte[] data;
data = zookeeper.getData("/demo", new DemoWatcher(), stat);
System.out.println(new String(data));
System.out.println(stat.toString());

// 修改节点，需将当前节点的版本号传入，如版本号对应不一致，则不会修改，并抛出BadVersion异常。
zookeeper.setData("/demo", "".getBytes(), stat.getVersion());

// 查看节点 指定监视对象 当节点数据变化时会发送通知
data = zookeeper.getData("/demo", new DemoWatcher(), stat);
System.out.println(new String(data));
System.out.println(stat.toString());

// 获取子节点列表 true表示对子节点设置监视，当子节点列表发生变化时，会发送通知，也可以不传此参数
List<String> childrenNode = zookeeper.getChildren("/", true);
childrenNode.stream().forEach(System.out::println);

// 判断节点是否存在 指定监视对象，当该节点创建，删除，数据发生变化时，会发送通知
stat = zookeeper.exists("/demo", new DemoWatcher());
System.out.println(stat.toString());

// 删除节点
zookeeper.delete("/demo", stat.getVersion());
zookeeper.close();
```

* 输出

``` text
WatchedEvent state:SyncConnected type:None path:null
demo
17179869228,17179869228,1555169733644,1555169733644,0,0,0,216172785363845125,4,0,17179869228

WatchedEvent state:SyncConnected type:NodeDataChanged path:/demo

17179869228,17179869229,1555169733644,1555169733662,1,0,0,216172785363845125,0,0,17179869228

zookeeper
demo
17179869228,17179869229,1555169733644,1555169733662,1,0,0,216172785363845125,0,0,17179869228

WatchedEvent state:SyncConnected type:NodeDeleted path:/demo
WatchedEvent state:SyncConnected type:NodeDeleted path:/demo
WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/
```

* 每当我们连接Zookeeper后，后台会启动一个守护线程自动去维护Zookeeper会话。

* 发生中断时，Zookeeper客户端会自动管理Zookeeper连接。

* Zookeeper操作相对来说比较费时，可以将操作(包含查询操作)设置为异步方式，使得操作立马返回结果，不影响其他操作。

* 异步操作需要传参时传入一个指定类型的回调对象。