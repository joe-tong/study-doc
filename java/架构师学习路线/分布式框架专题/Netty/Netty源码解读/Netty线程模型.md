# Netty线程模型以及简单入门

## 1.线程模型

​        什么是线程模型呢？线程模型指定了线程管理的模型。在进行并发编程的过程中，我们需要小心的处理多个线程之间的同步关系，而一个好的线程模型可以大大减少管理多个线程的成本

## 2.Reactor线程模型

Reactor是一种经典的线程模型，Reactor线程模型分为单线程模型、多线程模型以及主从多线程模型.

### 2.1 Reactor单线程模型

​        所有操作都在同一个NIO线程处理，在这个单线程中要负责接收请求，处理IO，编解码所有操作，相当于一个饭馆只有一个人，同时负责前台和后台服务，效率低。

![image.png](https://i.loli.net/2020/11/05/zNLlW982RKFwAyu.png)

### 2.2 Reactor多线程模

​        多线程的优点在于有单独的一个线程去处理请求，另外有一个线程池创建多个NIO线程去处理IO。相当于一个饭馆有一个前台负责接待，有很多服务员去做后面的工作，这样效率就比单线程模型提高很多

![image.png](https://i.loli.net/2020/11/05/eI5uJF4pi9DVUcE.png)

### 2.3 Reactor主从多线程模型（Netty采用的线程模型）

​       多线程模型的缺点在于并发量很高的情况下，只有一个Reactor单线程去处理是来不及的，就像饭馆只有一个前台接待很多客人也是不够的。为此需要使用主从线程模型。
主从线程模型：一组线程池接收请求，一组线程池处理IO。

![image.png](https://i.loli.net/2020/11/05/3ytrbRdOmLQf2XK.png)



**Netty采用了第三种模型：主从线程模型**

![image.png](https://i.loli.net/2020/11/05/26zdZctQSrGYXDU.png)

**模型解释:**

1. Netty 抽象出两组线程池BossGroup和WorkerGroup，BossGroup专门负责接收客户端的连接, WorkerGroup专门负责网络的读写
2. BossGroup和WorkerGroup类型都是NioEventLoopGroup
3.  NioEventLoopGroup 相当于一个事件循环**线程组**, 这个组中含有多个事件循环线程 ， 每一个事件 循环线程是NioEventLoop
4.  每个NioEventLoop都有一个selector , 用于监听注册在其上的socketChannel的网络通讯
5.  每个Boss NioEventLoop线程内部循环执行的步骤有 3 步
   - 处理accept事件 , 与client 建立连接 , 生成 NioSocketChannel
   -  将NioSocketChannel注册到某个worker NIOEventLoop上的selector 
   - 处理任务队列的任务 ， 即runAllTasks
6.  每个worker NIOEventLoop线程循环执行的步骤 
   - 轮询注册到自己selector上的所有NioSocketChannel 的read, write事件
   - 处理 I/O 事件， 即read , write 事件， 在对应NioSocketChannel 处理业务
   -  runAllTasks处理任务队列TaskQueue的任务 ，一些耗时的业务处理一般可以放入TaskQueue中慢慢处理，这样不影响数据在 pipeline 中的流动处理
7. 每个worker NIOEventLoop处理NioSocketChannel业务时，会使用 pipeline (管道)，管道中维护 了很多 handler 处理器用来处理 channel 中的数据

**Netty模块组件 【Bootstrap、ServerBootstrap】:**

Bootstrap 意思是引导，一个 Netty 应用通常由一个 Bootstrap 开始，主要作用是配置整个 Netty 程 序，串联各个组件，Netty 中 Bootstrap 类是客户端程序的启动引导类，ServerBootstrap 是服务端 启动引导类。

**【Future、ChannelFuture】：**

​        正如前面介绍，在 Netty 中所有的 IO 操作都是异步的，不能立刻得知消息是否被正确处理。 但是可以过一会等它执行完成或者直接注册一个监听，具体的实现就是通过 Future 和 ChannelFutures，他们可以注册一个监听，当操作执行成功或失败时监听会自动触发注册的监听事 件。

**【Channel】:**

1. Netty 网络通信的组件，能够用于执行网络 I/O 操作。Channel 为用户提供: 1)当前网络连接的通道的状态(例如是否打开?是否已连接?)

2. 网络连接的配置参数 (例如接收缓冲区大小)

3. 提供异步的网络 I/O 操作(如建立连接，读写，绑定端口)，异步调用意味着任何 I/O 调用都将立即 返回，并且不保证在调用结束时所请求的 I/O 操作已完成。

4. 调用立即返回一个 ChannelFuture 实例，通过注册监听器到 ChannelFuture 上，可以 I/O 操作成 功、失败或取消时回调通知调用方。

5. 支持关联 I/O 操作与对应的处理程序。 不同协议、不同的阻塞类型的连接都有不同的 Channel 类型与之对应。下面是一些常用的 Channel 类型:

   ```
   NioSocketChannel，异步的客户端 TCP Socket 连接。
   NioServerSocketChannel，异步的服务器端 TCP Socket 连接。 
   NioDatagramChannel，异步的 UDP 连接。
   NioSctpChannel，异步的客户端 Sctp 连接。
   NioSctpServerChannel，异步的 Sctp 服务器端连接，这些通道涵盖了 UDP 和 TCP 网络 IO 以及文 件 IO
   ```

**【Selector】:**
       Netty 基于 Selector 对象实现 I/O 多路复用，通过 Selector 一个线程可以监听多个连接的 Channel 事件。
当向一个 Selector 中注册 Channel 后，Selector 内部的机制就可以自动不断地查询(Select) 这些注册 的 Channel 是否有已就绪的 I/O 事件(例如可读，可写，网络连接完成等)，这样程序就可以很简单 地使用一个线程高效地管理多个 Channel 。
 **【NioEventLoop】:**
       NioEventLoop 中维护了一个线程和任务队列，支持异步提交执行任务，线程启动时会调用 NioEventLoop 的 run 方法，执行 I/O 任务和非 I/O 任务:I/O 任务，即 selectionKey 中 ready 的事件，如 accept、connect、read、write 等，由 processSelectedKeys 方法触发。

​       非 IO 任务，添加到 taskQueue 中的任务，如 register0、bind0 等任务，由 runAllTasks 方法触 发

**【NioEventLoopGroup】:**

 NioEventLoopGroup，主要管理 eventLoop 的生命周期，可以理解为一个线程池，内部维护了一组 线程，每个线程(NioEventLoop)负责处理多个 Channel 上的事件，而一个 Channel 只对应于一个线 程。

**【ChannelHandler】:**
 ChannelHandler 是一个接口，处理 I/O 事件或拦截 I/O 操作，并将其转发到其 ChannelPipeline(业 务处理链)中的下一个处理程序。
 ChannelHandler 本身并没有提供很多方法，因为这个接口有许多的方法需要实现，方便使用期间， 可以继承它的子类:

```
ChannelInboundHandler 用于处理入站 I/O 事件。 
ChannelOutboundHandler 用于处理出站 I/O 操作。 
```

或者使用以下适配器类:

```
ChannelInboundHandlerAdapter 用于处理入站 I/O 事件。 
ChannelOutboundHandlerAdapter 用于处理出站 I/O 操作。 
```

**【ChannelHandlerContext】:**
 保存 Channel 相关的所有上下文信息，同时关联一个 ChannelHandler 对象。 

**【ChannelPipline】:**
       保存 ChannelHandler 的 List，用于处理或拦截 Channel 的入站事件和出站操作。 ChannelPipeline 实现了一种高级形式的拦截过滤器模式，使用户可以完全控制事件的处理方式，以 及 Channel 中各个的 ChannelHandler 如何相互交互。
 在 Netty 中每个 Channel 都有且仅有一个 ChannelPipeline 与之对应，它们的组成关系如下:

![image.png](https://i.loli.net/2020/11/05/qc7OmXfdzCvpbHh.png)

一个 Channel 包含了一个 ChannelPipeline，而 ChannelPipeline 中又维护了一个由 ChannelHandlerContext 组成的双向链表，并且每个 ChannelHandlerContext 中又关联着一个 ChannelHandler。 read事件(入站事件)和write事件(出站事件)在一个双向链表中，入站事件会从链表 head 往后传递到最 后一个入站的 handler，出站事件会从链表 tail 往前传递到最前一个出站的 handler，两种类型的 handler 互不干扰。



 **在这个模型下，Netty提供了BootStrap类方便我们快速开发，下面是一个示例代码：**

```tsx
public class Server {
    public static void main(String[] args) throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childOption(ChannelOption.TCP_NODELAY, true)
                    .childAttr(AttributeKey.newInstance("childAttr"), "childAttrValue")
                    .handler(new ServerHandler())
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        public void initChannel(SocketChannel ch) {
                        }
                    });
            ChannelFuture f = b.bind(8888).sync();
            f.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

下面我们对Netty示例代码进行分析：

- 首先定义了两个EventLoopGroup，其中bossGroup对应的就是主线程池，只接收客户端的连接（注册，初始化逻辑），具体的工作由workerGroup这个从线程池来完成。可以理解为老板负责招揽接待，员工负责任务完成。线程池和线程组是一个概念，所以名称里有group 。之后就采用ServerBootstrap启动类，传入这两个主从线程组。
- 客户端和服务器建立连接后，NIO 会在两者之间建立Channel，所以启动类调用channel方法就是为了指定建立什么类型的通道。这里指定的是NioServerSocketChannel这个通道类。
- 启动类还调用了handler()和childHandler()方法，这两个方法中提及的handler是一个处理类的概念，他负责处理连接后的一个个通道的相应处理。handler()指定的处理类是主线程池中对通道的处理类，childHandler()方法指定的是从线程池中对通道的处理类。
- 执行ServerBootstrap的bind方法进行绑定端口的同时也执行了sync()方法进行同步阻塞调用。
- 关闭通道采用Channel的closeFuture()方法关闭。
- 最终优雅地关闭两个线程组，执行shutdownGracefully()方法完成关闭线程组。

**设置ChannelHandler**

![image.png](https://i.loli.net/2020/11/05/Jr1RCjEiUfMS7B9.png)

现在单独分析下处理类handler，每一个Channel由多个handler共同组成管道pipeline。

ChannelHander 管道中的handler可以用netty官方提供的处理类，也可以自行定义处理类。在上面的示例代码中，childHandler方法中传入ChannelInitializer对象，它的initChannel()方法中可以自行扩展，下面是一个具体的例子：

```java
public class CustomerHandler extends SimpleChannelInboundHandler<HttpObject> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, HttpObject msg) throws Exception {

        Channel channel = ctx.channel(); // 获取通道
        System.out.println(channel.remoteAddress()); // 显示客户端的远程地址
        ByteBuf content = Unpooled.copiedBuffer("hello,netty", CharsetUtil.UTF_8);// 定义要返回的数据

        // 定义响应
        FullHttpResponse response =
                new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK,content);

        // 定义请求头
        response.headers().set(HttpHeaderNames.CONTENT_TYPE,"text/plain");
        response.headers().set(HttpHeaderNames.CONTENT_LENGTH,content.readableBytes());

        ctx.writeAndFlush(response);

    }
}
```

这是个自定义的处理类，它继承SimpleChannelInboundHandler这个初始化类，在channelRead0方法内部实现读写缓冲区的操作：首先从上下文中获取连接后的通道，然后创建ByteBuf对象，里面保存要显示的字符串，接着创建一个Http response  对象，设置返回的报文头和内容，最后使用上下文放松请求，注意要使用writeAndFlush而不是write，这是因为没有执行flush操作，数据仍在缓冲区中。
 写好了上面这个自定义的处理类后将它配置到启动类中：

```tsx
            ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childOption(ChannelOption.TCP_NODELAY, true)
                    .childAttr(AttributeKey.newInstance("childAttr"), "childAttrValue")
                    .handler(new ServerHandler())
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        public void initChannel(SocketChannel channel) {
                            ChannelPipeline pipeline = channel.pipeline();
                            pipeline.addLast("HttpServerCodec",new HttpServerCodec());
                            pipeline.addLast("customerHandler",new CustomerHandler());
                        }
                    });
```

可以看出在childHandler中，pipeline添加了两个处理类，一个是HttpServerCodec，是Netty自带的对请求和响应进行编解码的处理类；另一个就是我们创建的自定义处理类。这样启动这个应用后，在浏览器访问http://localhost:8888/地址后，页面上就会显示`hello,netty`的字样，说明我们添加的处理类已经生效了。

