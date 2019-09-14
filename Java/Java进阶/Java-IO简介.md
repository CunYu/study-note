### Java IO 概述

Java IO指的是Java程序的输入和输出，这里是指各个方面的输入输出，例如控制台，文件，网络，内存等。

Java IO是阻塞式的。

### 主要API

|类名|说明|
|:----|:----|
|File|文件类|
|InputStream|字节输入流|
|OutputStream|字节输出流|
|Reader|字符输入流|
|Writer|字符输出流|
|RandomAccessFile|可随意操作文件类|
|Socket|网络输入输出类|

### 字节和字符

字节（byte）是计算机存储的一种计量单位。

字符是指计算机使用的文字和符号，其涉及到多种编码格式，不同的编码格式占用的空间不一样。

### 路径分隔符问题

Windows系统的路径分隔符为\，linux系统的路径分隔符为/。

为了兼容不同的系统，java中可以使用File.separator来替代分隔符。

### File

File是文件（目录）的抽象表示，其可以操作文件，但是不能操作文件的内容。

Demo

``` java
File file = new File("F:" + File.separator + "demo");
if (!file.exists()) {
    //mkdirs可以创建多级目录
    file.mkdir();
}
File txtDemo = new File("F:" + File.separator + "demo"+ File.separator+ "txtDemo.txt");
txtDemo.createNewFile();
File jpgDemo = new File("F:" + File.separator + "demo" + File.separator+ "jpgDemo.jpg");
jpgDemo.createNewFile();
String[] files = file.list();
String[] jpgFiles = file.list((dir, name)->{
    if (name.contains(".jpg")) {
        return true;
    }
    return false;
});

System.out.println(file.exists());
System.out.println(txtDemo.exists());
Arrays.stream(files).forEach(System.out::println);
System.out.println();
Arrays.stream(jpgFiles).forEach(System.out::println);
```

``` text
true
true
jpgDemo.jpg
txtDemo.txt

jpgDemo.jpg
```

### 字节流

InputStream和OutputStream可以对字节流进行输入输出操作。

Demo

``` java
File file = new File("F:" + File.separator + "demo");
File demo = new File("F:" + File.separator + "demo" + File.separator+ "demo.txt");
if (!file.exists()) {
    file.mkdir();
}
if (!demo.exists()) {
    demo.createNewFile();
}

OutputStream outputStream = new FileOutputStream(demo);
outputStream.write("Hello World!".getBytes());
outputStream.close();

byte[] content = new byte[(int)demo.length()];
InputStream inputStream = new FileInputStream(demo);
inputStream.read(content);
inputStream.close();
System.out.println(new String(content));
```

``` text
Hello World!
```

### 字符流

Reader和Writer可以对字符流进行输入输出操作。

Demo

``` java
File file = new File("F:" + File.separator + "demo");
File demo = new File("F:" + File.separator + "demo" + File.separator + "demo.txt");
if (!file.exists()) {
    file.mkdir();
}
if (!demo.exists()) {
    demo.createNewFile();
}
Writer writer = new FileWriter(demo);
writer.write("Hello World!");
writer.flush();
writer.close();
Reader reader = new FileReader(demo);
char[] content = new char[(int)demo.length()];
reader.read(content);
for(char ch : content) {
    System.out.print(ch);
}
reader.close();
```

``` text
Hello World!
```

### 字节流与字符流对比

字符流只处理字符类型的数据，字节可以处理所有类型（图片，音乐，视频等）的数据。

字节流IO操作时，直接操作的是数据终端（例如文件），字符流操作时并不直接操作数据终端，其操作的是缓存（内存），最后再由缓冲去操作数据终端，其可以用flush()方法来将缓冲中的操作刷新到数据终端去。

有中文的情况建议使用字符流来处理。

### RandomAccessFile

RandomAccessFile可以对文件进行输入输出操作，其允许操作文件记录指针（标识当前IO操作的位置）。

Demo

``` java
File file = new File("F:" + File.separator + "demo");
File demo = new File("F:" + File.separator + "demo" + File.separator + "demo.txt");
if (!file.exists()) {
    file.mkdir();
}
if (!demo.exists()) {
    demo.createNewFile();
}
RandomAccessFile randomAccessFile = new RandomAccessFile(demo, "rw");
randomAccessFile.write("Hello".getBytes());
randomAccessFile.write(" World!".getBytes());
byte[] content = new byte[(int) demo.length()];
randomAccessFile.seek(0);
randomAccessFile.read(content);

System.out.println(new String(content));
```

``` text
Hello World!
```

### Socket

Socket可以在不同设备之间通过网络来进行输入输出操作。

Demo

* 服务端

``` java
ServerSocket serverSocket = new ServerSocket(8888);
while (true) {
    Socket socket = serverSocket.accept();
    DataInputStream dataInputStream = new DataInputStream(socket.getInputStream());
    System.out.println(dataInputStream.readUTF());
    DataOutputStream dataOutputStream = new DataOutputStream(socket.getOutputStream());
    dataOutputStream.writeUTF("你好");
    socket.close();
}
```

* 客户端

``` java
Socket socket = new Socket("localhost", 8888);
DataOutputStream dataOutputStream = new DataOutputStream(socket.getOutputStream());
dataOutputStream.writeUTF("你好");
DataInputStream dataInputStream = new DataInputStream(socket.getInputStream());
System.out.println(dataInputStream.readUTF());
socket.close();
```