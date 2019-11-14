### 概述

Connector是Tomcat中的连接器，其处理外界的请求并封装为对象交给容器处理。

### Connector运行模式

* bio

阻塞IO，一个线程只处理一个请求，Linux系统中，Tomcat7及以下版本的默认连接器，Tomcat8.5和Tomcat9.0后的版本已完全删除该模式的连接器。

* nio

非阻塞IO，一个线程可以处理多个请求，Linux系统中，Tomcat8及以上版本的默认连接器。

* apr

Apache Portable Runtime，Apache可移植运行时库，其以JNI的形式调用Apache HTTP服务器的核心动态链接库来处理文件读取和网络传输操作，进而极大的提高了静态文件的处理能力。

apr是从操作系统的维度来解决异步IO问题，极大的提高了性能，其是Tomcat处理高并发应用的首选模式。

apr需要安装apr和native。

### 部分源码分析

##### 说明

* 使用的ProtocolHandler为Http11NioProtocol

##### 分析

* Connector构造方法

``` java
/**
 * Defaults to using HTTP/1.1 NIO implementation.
 */
public Connector() {
    this("org.apache.coyote.http11.Http11NioProtocol");
}
```

``` java
public Connector(String protocol) {
    boolean aprConnector = AprLifecycleListener.isAprAvailable() &&
            AprLifecycleListener.getUseAprConnector();

    if ("HTTP/1.1".equals(protocol) || protocol == null) {
        if (aprConnector) {
            protocolHandlerClassName = "org.apache.coyote.http11.Http11AprProtocol";
        } else {
            protocolHandlerClassName = "org.apache.coyote.http11.Http11NioProtocol";
        }
    } else if ("AJP/1.3".equals(protocol)) {
        if (aprConnector) {
            protocolHandlerClassName = "org.apache.coyote.ajp.AjpAprProtocol";
        } else {
            protocolHandlerClassName = "org.apache.coyote.ajp.AjpNioProtocol";
        }
    } else {
        protocolHandlerClassName = protocol;
    }

    // Instantiate protocol handler
    ProtocolHandler p = null;
    try {
        Class<?> clazz = Class.forName(protocolHandlerClassName);
        p = (ProtocolHandler) clazz.getConstructor().newInstance();
    } catch (Exception e) {
        log.error(sm.getString(
                "coyoteConnector.protocolHandlerInstantiationFailed"), e);
    } finally {
        this.protocolHandler = p;
    }

    // Default for Connector depends on this system property
    setThrowOnFailure(Boolean.getBoolean("org.apache.catalina.startup.EXIT_ON_INIT_FAILURE"));
}
```

Connector构造方法主要是加载合适的ProtocolHandler类。

ProtocolHandler构造函数中，会给自己初始化一个NioEndpoint，并给NioEndpoint初始化一个ConnectionHandler。

* Connector的startInternal方法

Connector的start方法会调用startInternal方法，其用了模板方法模式，start是模板方法，Connector实现自己的startInternal方法。

``` java
/**
 * Begin processing requests via this Connector.
 *
 * @exception LifecycleException if a fatal startup error occurs
 */
@Override
protected void startInternal() throws LifecycleException {

    // Validate settings before starting
    if (getPortWithOffset() < 0) {
        throw new LifecycleException(sm.getString(
                "coyoteConnector.invalidPort", Integer.valueOf(getPortWithOffset())));
    }

    setState(LifecycleState.STARTING);

    try {
        protocolHandler.start();
    } catch (Exception e) {
        throw new LifecycleException(
                sm.getString("coyoteConnector.protocolHandlerStartFailed"), e);
    }
}
```

Connector的startInternal方法会调用其protocolHandler的start方法。

* NioEndpoint的start方法

Http11NioProtocol的start方法会调用其endpoint的start方法。

NioEndpoint的start方法会先调用bindWithCleanup方法，然后再调用startInternal方法，其用了模板方法模式，start是模板方法，NioEndpoint实现自己的部分方法。

``` java
private void bindWithCleanup() throws Exception {
    try {
        bind();
    } catch (Throwable t) {
        // Ensure open sockets etc. are cleaned up if something goes
        // wrong during bind
        ExceptionUtils.handleThrowable(t);
        unbind();
        throw t;
    }
}
```

``` java
/**
 * Initialize the endpoint.
 */
@Override
public void bind() throws Exception {
    initServerSocket();

    setStopLatch(new CountDownLatch(1));

    // Initialize SSL if needed
    initialiseSsl();

    selectorPool.open(getName());
}

// Separated out to make it easier for folks that extend NioEndpoint to
// implement custom [server]sockets
protected void initServerSocket() throws Exception {
    if (!getUseInheritedChannel()) {
        serverSock = ServerSocketChannel.open();
        socketProperties.setProperties(serverSock.socket());
        InetSocketAddress addr = new InetSocketAddress(getAddress(), getPortWithOffset());
        serverSock.socket().bind(addr,getAcceptCount());
    } else {
        // Retrieve the channel provided by the OS
        Channel ic = System.inheritedChannel();
        if (ic instanceof ServerSocketChannel) {
            serverSock = (ServerSocketChannel) ic;
        }
        if (serverSock == null) {
            throw new IllegalArgumentException(sm.getString("endpoint.init.bind.inherited"));
        }
    }
    serverSock.configureBlocking(true); //mimic APR behavior
}
```

``` java
/**
 * Start the NIO endpoint, creating acceptor, poller threads.
 */
@Override
public void startInternal() throws Exception {

    if (!running) {
        running = true;
        paused = false;

        if (socketProperties.getProcessorCache() != 0) {
            processorCache = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                    socketProperties.getProcessorCache());
        }
        if (socketProperties.getEventCache() != 0) {
            eventCache = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                    socketProperties.getEventCache());
        }
        if (socketProperties.getBufferPool() != 0) {
            nioChannels = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                    socketProperties.getBufferPool());
        }

        // Create worker collection
        if (getExecutor() == null) {
            createExecutor();
        }

        initializeConnectionLatch();

        // Start poller thread
        poller = new Poller();
        Thread pollerThread = new Thread(poller, getName() + "-ClientPoller");
        pollerThread.setPriority(threadPriority);
        pollerThread.setDaemon(true);
        pollerThread.start();

        startAcceptorThread();
    }
}
```

``` java
public void createExecutor() {
    internalExecutor = true;
    TaskQueue taskqueue = new TaskQueue();
    TaskThreadFactory tf = new TaskThreadFactory(getName() + "-exec-", daemon, getThreadPriority());
    executor = new ThreadPoolExecutor(getMinSpareThreads(), getMaxThreads(), 60, TimeUnit.SECONDS,taskqueue, tf);
    taskqueue.setParent( (ThreadPoolExecutor) executor);
}
```

``` java
protected void startAcceptorThread() {
    acceptor = new Acceptor<>(this);
    String threadName = getName() + "-Acceptor";
    acceptor.setThreadName(threadName);
    Thread t = new Thread(acceptor, threadName);
    t.setPriority(getAcceptorThreadPriority());
    t.setDaemon(getDaemon());
    t.start();
}
```

NioEndpoint的startInternal方法会判断自己的线程池是否为空，为空则创建一个线程池，然后启动一个poller线程，一个acceptor线程。

* Poller部分代码

``` java
public Poller() throws IOException {
    this.selector = Selector.open();
}
```

``` java
/**
 * The background thread that adds sockets to the Poller, checks the
 * poller for triggered events and hands the associated socket off to an
 * appropriate processor as events occur.
 */
@Override
public void run() {
    // Loop until destroy() is called
    while (true) {

        boolean hasEvents = false;

        try {
            if (!close) {
                hasEvents = events();
                if (wakeupCounter.getAndSet(-1) > 0) {
                    // If we are here, means we have other stuff to do
                    // Do a non blocking select
                    keyCount = selector.selectNow();
                } else {
                    keyCount = selector.select(selectorTimeout);
                }
                wakeupCounter.set(0);
            }
            if (close) {
                events();
                timeout(0, false);
                try {
                    selector.close();
                } catch (IOException ioe) {
                    log.error(sm.getString("endpoint.nio.selectorCloseFail"), ioe);
                }
                break;
            }
        } catch (Throwable x) {
            ExceptionUtils.handleThrowable(x);
            log.error(sm.getString("endpoint.nio.selectorLoopError"), x);
            continue;
        }
        // Either we timed out or we woke up, process events first
        if (keyCount == 0) {
            hasEvents = (hasEvents | events());
        }

        Iterator<SelectionKey> iterator =
            keyCount > 0 ? selector.selectedKeys().iterator() : null;
        // Walk through the collection of ready keys and dispatch
        // any active event.
        while (iterator != null && iterator.hasNext()) {
            SelectionKey sk = iterator.next();
            NioSocketWrapper socketWrapper = (NioSocketWrapper) sk.attachment();
            // Attachment may be null if another thread has called
            // cancelledKey()
            if (socketWrapper == null) {
                iterator.remove();
            } else {
                iterator.remove();
                processKey(sk, socketWrapper);
            }
        }

        // Process timeouts
        timeout(keyCount,hasEvents);
    }

    getStopLatch().countDown();
}
```

Poller封装了Selector，其通过Selector监听注册到其的组件，一旦发生相应的事件，则调用processKey方法处理。

``` java
protected void processKey(SelectionKey sk, NioSocketWrapper socketWrapper) {
    try {
        if (close) {
            cancelledKey(sk, socketWrapper);
        } else if (sk.isValid() && socketWrapper != null) {
            if (sk.isReadable() || sk.isWritable()) {
                if (socketWrapper.getSendfileData() != null) {
                    processSendfile(sk, socketWrapper, false);
                } else {
                    unreg(sk, socketWrapper, sk.readyOps());
                    boolean closeSocket = false;
                    // Read goes before write
                    if (sk.isReadable()) {
                        if (socketWrapper.readOperation != null) {
                            if (!socketWrapper.readOperation.process()) {
                                closeSocket = true;
                            }
                        } else if (!processSocket(socketWrapper, SocketEvent.OPEN_READ, true)) {
                            closeSocket = true;
                        }
                    }
                    if (!closeSocket && sk.isWritable()) {
                        if (socketWrapper.writeOperation != null) {
                            if (!socketWrapper.writeOperation.process()) {
                                closeSocket = true;
                            }
                        } else if (!processSocket(socketWrapper, SocketEvent.OPEN_WRITE, true)) {
                            closeSocket = true;
                        }
                    }
                    if (closeSocket) {
                        cancelledKey(sk, socketWrapper);
                    }
                }
            }
        } else {
            // Invalid key
            cancelledKey(sk, socketWrapper);
        }
    } catch (CancelledKeyException ckx) {
        cancelledKey(sk, socketWrapper);
    } catch (Throwable t) {
        ExceptionUtils.handleThrowable(t);
        log.error(sm.getString("endpoint.nio.keyProcessingError"), t);
    }
}
```

NioSocketWrapper的读写操作的process方法是将任务交给Endpoint中的线程池处理。

* Acceptor部分代码

``` java
@Override
public void run() {

    int errorDelay = 0;

    // Loop until we receive a shutdown command
    while (endpoint.isRunning()) {

        // Loop if endpoint is paused
        while (endpoint.isPaused() && endpoint.isRunning()) {
            state = AcceptorState.PAUSED;
            try {
                Thread.sleep(50);
            } catch (InterruptedException e) {
                // Ignore
            }
        }

        if (!endpoint.isRunning()) {
            break;
        }
        state = AcceptorState.RUNNING;

        try {
            //if we have reached max connections, wait
            endpoint.countUpOrAwaitConnection();

            // Endpoint might have been paused while waiting for latch
            // If that is the case, don't accept new connections
            if (endpoint.isPaused()) {
                continue;
            }

            U socket = null;
            try {
                // Accept the next incoming connection from the server
                // socket
                socket = endpoint.serverSocketAccept();
            } catch (Exception ioe) {
                // We didn't get a socket
                endpoint.countDownConnection();
                if (endpoint.isRunning()) {
                    // Introduce delay if necessary
                    errorDelay = handleExceptionWithDelay(errorDelay);
                    // re-throw
                    throw ioe;
                } else {
                    break;
                }
            }
            // Successful accept, reset the error delay
            errorDelay = 0;

            // Configure the socket
            if (endpoint.isRunning() && !endpoint.isPaused()) {
                // setSocketOptions() will hand the socket off to
                // an appropriate processor if successful
                if (!endpoint.setSocketOptions(socket)) {
                    endpoint.closeSocket(socket);
                }
            } else {
                endpoint.destroySocket(socket);
            }
        } catch (Throwable t) {
            ExceptionUtils.handleThrowable(t);
            String msg = sm.getString("endpoint.accept.fail");
            // APR specific.
            // Could push this down but not sure it is worth the trouble.
            if (t instanceof Error) {
                Error e = (Error) t;
                if (e.getError() == 233) {
                    // Not an error on HP-UX so log as a warning
                    // so it can be filtered out on that platform
                    // See bug 50273
                    log.warn(msg, t);
                } else {
                    log.error(msg, t);
                }
            } else {
                    log.error(msg, t);
            }
        }
    }
    state = AcceptorState.ENDED;
}
```

``` java
@Override
protected SocketChannel serverSocketAccept() throws Exception {
    return serverSock.accept();
}
```

``` java
/**
 * Process the specified connection.
 * @param socket The socket channel
 * @return <code>true</code> if the socket was correctly configured
 *  and processing may continue, <code>false</code> if the socket needs to be
 *  close immediately
 */
@Override
protected boolean setSocketOptions(SocketChannel socket) {
    // Process the connection
    try {
        // Disable blocking, polling will be used
        socket.configureBlocking(false);
        Socket sock = socket.socket();
        socketProperties.setProperties(sock);

        NioChannel channel = null;
        if (nioChannels != null) {
            channel = nioChannels.pop();
        }
        if (channel == null) {
            SocketBufferHandler bufhandler = new SocketBufferHandler(
                    socketProperties.getAppReadBufSize(),
                    socketProperties.getAppWriteBufSize(),
                    socketProperties.getDirectBuffer());
            if (isSSLEnabled()) {
                channel = new SecureNioChannel(socket, bufhandler, selectorPool, this);
            } else {
                channel = new NioChannel(socket, bufhandler);
            }
        } else {
            channel.setIOChannel(socket);
            channel.reset();
        }
        NioSocketWrapper socketWrapper = new NioSocketWrapper(channel, this);
        channel.setSocketWrapper(socketWrapper);
        socketWrapper.setReadTimeout(getConnectionTimeout());
        socketWrapper.setWriteTimeout(getConnectionTimeout());
        socketWrapper.setKeepAliveLeft(NioEndpoint.this.getMaxKeepAliveRequests());
        socketWrapper.setSecure(isSSLEnabled());
        poller.register(channel, socketWrapper);
        return true;
    } catch (Throwable t) {
        ExceptionUtils.handleThrowable(t);
        try {
            log.error(sm.getString("endpoint.socketOptionsError"), t);
        } catch (Throwable tt) {
            ExceptionUtils.handleThrowable(tt);
        }
    }
    // Tell to close the socket
    return false;
}
```

Accept会监听相应的端口，一旦有请求过来，会将该请求注册给Poller。