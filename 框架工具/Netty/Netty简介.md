### 概述

Netty是一个基于Java的开源网络通信框架，通过Netty可以快速实现网络通信的服务端和客户端。

### 特点

Netty支持多种协议，并且允许自定义通信协议。

Netty使用了IO多路复用模型，其支持高并发。

Netty封装了Java网络相关代码，其提供了一个简洁方便的API。

Netty提供异步的，事件驱动的框架和工具，Netty的所有IO操作都是异步非阻塞的。

### Netty基本组件

##### Channel

Channel对Java中的Channel进行了封装，其实现了自己的Channel。

Channel主要分为NIO（非阻塞IO）和OIO（非阻塞IO）两类。

* NioSocketChannel

* NioServerSocketChannel

* NioDatagramChannel

* NioSctpChannel

* NioSctpServerChannel

* OioSocketChannel

* OioServerSocketChannel

* OioDatagramChannel

* OioSctpChannel

* OioSctpServerChannel

##### NioEventLoop

NioEventLoop是对Selector的封装。

##### ChannelHandler

ChannelHandler是Netty用来处理Selector监听的事件。

ChannelHandler分为两大类，ChannelInboundHandler和ChannelOutboundHandler。

ChannelInboundHandler针对的是SelectionKey.OP_READ事件，其默认实现为ChannelInboundHandlerAdapter。

ChannelOutboundHandler针对的是SelectionKey.OP_WRITE事件，其默认实现为ChannelOutboundHandlerAdapter。

##### ChannelPipeline

在Netty中，ChannelPipeline是用来处理一个事件多个ChannelHandler处理的情况，ChannelPipeline是由ChannelHandler组成的，每个事件对应一个ChannelPipeline。

在ChannelPipeline中，如果是读事件，则从前往后执行，如果是写事件，则从后往前执行。

##### Bootstrap

Bootstrap是用来组装各个组件的。

客户端使用Bootstrap，服务端使用ServerBootstrap。

### 示例

* gradle依赖

``` groovy
compile("io.netty:netty-all:4.1.43.Final")
```

* Server

``` java
public class NettyServer {

    private int port;

    public NettyServer(int port) {
        this.port = port;
    }

    public void start() {

        EventLoopGroup bossEventLoopGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerEventLoopGroup = new NioEventLoopGroup();

        try {

            ServerBootstrap serverBootstrap = new ServerBootstrap();

            serverBootstrap.group(bossEventLoopGroup, workerEventLoopGroup);
            serverBootstrap.channel(NioServerSocketChannel.class);
            serverBootstrap.childHandler(new ChannelInitializer<SocketChannel>() {

                @Override
                protected void initChannel(SocketChannel ch) {
                    ch.pipeline().addLast(new NettyServerMessageReadHandler());
                }
            });

            ChannelFuture channelFuture = serverBootstrap.bind(port).sync();
            System.out.println("服务端启动，监听" + port + "端口");

            channelFuture.channel().closeFuture().sync();
            System.out.println("服务端关闭");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            bossEventLoopGroup.shutdownGracefully();
            workerEventLoopGroup.shutdownGracefully();
        }
    }
}
```

``` java
public class NettyServerMessageReadHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {

        ByteBuf byteBuf = (ByteBuf) msg;
        System.out.println("服务端收到信息：");
        while (byteBuf.isReadable()) {
            System.out.print((char) byteBuf.readByte());
        }
        System.out.println();
        ctx.writeAndFlush(Unpooled.copiedBuffer("true", CharsetUtil.UTF_8));
    }
}
```

* Client

``` java
public class NettyClient {

    private String host;
    private int port;

    public NettyClient(String host, int port) {
        this.host = host;
        this.port = port;
    }

    public void start() {

        EventLoopGroup eventLoopGroup = new NioEventLoopGroup();

        try {

            Bootstrap bootstrap = new Bootstrap();

            bootstrap.group(eventLoopGroup);
            bootstrap.channel(NioSocketChannel.class);
            bootstrap.handler(new ChannelInitializer<SocketChannel>() {

                @Override
                protected void initChannel(SocketChannel ch) throws Exception {
                    ch.pipeline().addLast(new NettyClientMessageReadHandler());
                }
            });

            ChannelFuture channelFuture = bootstrap.connect(host, port).sync();
            System.out.println("客户端启动");

            channelFuture.channel().closeFuture().sync();
            System.out.println("客户端关闭");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            eventLoopGroup.shutdownGracefully();
        }
    }
}
```

``` java
public class NettyClientMessageReadHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelActive(ChannelHandlerContext ctx) {
        ctx.writeAndFlush(Unpooled.copiedBuffer("Hello", CharsetUtil.UTF_8)).isSuccess();
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {

        ByteBuf byteBuf = (ByteBuf) msg;
        System.out.println("客户端收到反馈：");
        while (byteBuf.isReadable()) {
            System.out.print((char) byteBuf.readByte());
        }
        System.out.println();
        ctx.close();
    }
}
```