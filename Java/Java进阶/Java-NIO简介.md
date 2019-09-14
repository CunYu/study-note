### 概述

Java 在JDK1.4的时候引入了新的IO类库以提高IO的效率，简称NIO。

其实旧的IO类库也已经用NIO重新实现过。

### IO与NIO对比

IO是面向流的IO，每次读取一个或多个字节，其不能操作流中的数据，除非其将读取到的数据存放到一个中间数据结构中，然后在数据结构中操作数据。

NIO是面向缓存的，其先将数据存放至缓存中，然后再对缓存进行操作，数据在缓存中是可以进行操作的。

IO是阻塞式的，NIO是非阻塞式的。

### NIO核心组件

##### Channel

Channel表示与实体的连接，其是Java程序与操作系统IO做交互的通道。

Channel是双向的。

* 常见Channel

|名称|说明|
|:----|:----|
|FileChannel|文件Channel|
|DatagramChannel|使用UDP协议传输的网络数据Channel|
|SocketChannel|使用TCP协议传输的网络数据Channel|
|ServerSocketChannel|ServerSocket Channel，监听TCP端口，每收到连接都会创建一个SocketChannel|

##### Buffer

Buffer是与Channel交互的缓存区。

* 常见Buffer

|缓存名称|对应基本类型|
|:----|:----|
|ByteBuffer|byte|
|CharBuffer|char|
|ShortBuffer|short|
|IntBuffer|int|
|LongBuffer|long|
|FloatBuffer|float|
|DoubleBuffer|double|

* Buffer重要属性

|名称|说明|
|:----|:----|
|position|指针当前位置|
|limit|边界位置|
|capacity|容量|

<img src="/Java/Java进阶/image/NIO-Buffer结构.png" alt="NIO-Buffer结构"/>

##### Selector

Selector是用来监听各个Channel的状态的。

* Selector监听事件

|名称|说明|
|:----|:----|
|SelectionKey.OP_READ|读|
|SelectionKey.OP_WRITE|写|
|SelectionKey.OP_ACCEPT|存在待接受的连接|
|SelectionKey.OP_CONNECT|连接成功|

### 示例

##### 文件

``` java
File file = new File("F:\\demo.txt");
FileInputStream fileInputStream = new FileInputStream(file);
FileChannel fileChannel = fileInputStream.getChannel();
ByteBuffer byteBuffer = ByteBuffer.allocate(128);
while (-1 != fileChannel.read(byteBuffer)) {
    // 将buffer装换为读模式
    byteBuffer.flip();
    byte[] bytes = new byte[byteBuffer.limit()];
    byteBuffer.get(bytes);
    System.out.print(new String(bytes));
    byteBuffer.clear();
}
fileChannel.close();
fileInputStream.close();
```

输出

``` text
Hello World!
```

##### 网络

* 服务端

``` java
Selector selector = Selector.open();

ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
serverSocketChannel.bind(new InetSocketAddress(8888));
serverSocketChannel.configureBlocking(false);
// 多个事件可以用|分隔
serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
ByteBuffer byteBuffer = ByteBuffer.allocate(128);

while (true) {

    // 阻塞，直到监听的事件发生
    selector.select();
    Iterator<SelectionKey> keyIterator = selector.selectedKeys().iterator();

    while (keyIterator.hasNext()) {

        SelectionKey selectionKey = keyIterator.next();

        if (selectionKey.isAcceptable()) {
            SocketChannel socketChannel = ((ServerSocketChannel) selectionKey.channel()).accept();
            socketChannel.configureBlocking(false);
            socketChannel.register(selector, SelectionKey.OP_READ);
        }

        if (selectionKey.isReadable()) {

            SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
            socketChannel.read(byteBuffer);
            // 将buffer装换为读模式
            byteBuffer.flip();
            byte[] bytes = new byte[byteBuffer.limit()];
            byteBuffer.get(bytes);
            System.out.println(new String(bytes));
            byteBuffer.clear();
            selectionKey.interestOps(SelectionKey.OP_WRITE);
        }

        if (selectionKey.isWritable()) {
            SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
            byteBuffer.put("hello World!".getBytes());
            byteBuffer.flip();
            socketChannel.write(byteBuffer);
            byteBuffer.clear();
            socketChannel.close();
        }
    }
    // 移除已处理的事件
    keyIterator.remove();
}
```

* 客户端

``` java
IntStream.range(0, 10).forEach(i -> {
    new Thread(() -> {
        try {
            Socket socket = new Socket("localhost", 8888);
            DataOutputStream dataOutputStream = new DataOutputStream(socket.getOutputStream());
            Thread.sleep(5000);
            dataOutputStream.write("Hello World!".getBytes());
            DataInputStream dataInputStream = new DataInputStream(socket.getInputStream());

            byte[] bytes = new byte[128];
            dataInputStream.read(bytes);
            System.out.println(new String(bytes));
            socket.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }).start();
});
```