title: Netty学习（1）
date: 2018-08-09 09:46:02
categories: 编程
tags: Netty
---
Netty是一个网络通讯框架。好多开源项目底层用到了Netty，比如dubbo、akka、spark、finagle、gRPC、Vert.x等。目前官方推荐的版本是4.X。
<!-- more -->


学习的最好方式是实践，这次准备用Netty写个简单的RPC。参考了这篇文章[《一个轻量级分布式RPC框架--NettyRpc》](http://www.cnblogs.com/luxiaoxun/p/5272384.html)。首先，我们看一下传统的阻塞式IO代码是如何组织的。

```
ServerSocket server = new ServerSocket(port);
while (true) {
    Socket socket = server.accept();
    new Thread().start();
}

```

上面的这种方式是阻塞的,只有新连接来了，accept才会返回，主线程才能继续。子线程里的read和write操作也是阻塞的。这种代码容易理解，但是当并发较大时，性能跟不上。来一个客户端连接，就会新建一个线程，而JVM资源是有限的，这种方案不具有可扩展性。并且大部分线程一会儿阻塞，一会儿运行，来回切换上下文，开销巨大。

服务端代码

```
EventLoopGroup bossGroup = new NioEventLoopGroup();
EventLoopGroup workerGroup = new NioEventLoopGroup();
ServerBootstrap bootstrap = new ServerBootstrap();
bootstrap.group(bossGroup, workerGroup)
        .channel(NioServerSocketChannel.class)
        .childHandler(new ChannelInitializer<SocketChannel>() {
            @Override
            public void initChannel(SocketChannel channel) throws Exception {
                channel.pipeline()
                        .addLast(new RpcDecoder(RpcRequest.class))
                        .addLast(new RpcEncoder(RpcResponse.class))
                        .addLast(new RpcHandler(handlerMap));
            }
        })
        .option(ChannelOption.SO_BACKLOG, 128)
        .childOption(ChannelOption.SO_KEEPALIVE, true);

int port = 18866;
ChannelFuture future = bootstrap.bind(port).sync();
logger.info("Server started on port {}", port);

future.channel().closeFuture().sync();
```

从代码中可以看到，与普通的ServerSocket不同，这段代码不需要处理等待连接建立，读数据、写数据（通常来说这些操作都是阻塞的）。那么问题来了，具体的业务逻辑在哪里呢，答案就是pipeline中的各个handler。RpcDecoder负责解码，继承了ByteToMessageDecoder类(ChannelInboundHandlerAdapter的子类)，将字节流解析成java对象，收到字节流的时候触发；RpcEncoder负责编码，继承了MessageToByteEncoder类（ChannelOutboundHandlerAdapter的子类），将java对象编码为字节流，发送字节流时触发；RpcHandler负责真正的业务逻辑，根据decoder后的java对象，利用反射调用本地方法，并将调用结果写回到输出流中。

NioEventLoopGroup实际上是Reactor模式，bossGroup和workerGroup可以看作两个线程池，前者负责接收客户端的TCP连接，后者负责处理I/O相关读写操作，后期可以专门说一下。

backlog参数主要用于底层方法int listen(int sockfd, int backlog)，参数规定了内核为socket排队的最大连接个数。内核要维护两个队列：

1. 未完成连接队列，三次握手还没有完成的队列，其大小通过/proc/sys/net/ipv4/tcp_max_syn_backlog指定
2. 已完成连接队列，三次握手完成的队列，其大小通过/proc/sys/net/core/somaxconn指定，在使用listen函数时，内核会根据传入的backlog参数与系统参数somaxconn，取二者的较小值。

两个队列之和不超过backlog。放个图：

![](/images/TCP队列.png)

channel和channelPipeline的关系，从网上找了段解释，非常清楚，原文出处没找到。

> 一个Channel分配一个ChannelPipeline，每个ChannelPipeline里是多个ChannelHandler组成的链表，每个ChannelHandler会对应一个ChannelHandlerContext。看看下面的图就能明白：
![](/images/channel.png)

解码器：

```
public class RpcDecoder extends ByteToMessageDecoder {
    private Class<?> genericClass;

    public RpcDecoder(Class<?> genericClass) {
        this.genericClass = genericClass;
    }

    @Override
    public final void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        if (in.readableBytes() < 4) {
            return;
        }
        in.markReaderIndex();
        int dataLength = in.readInt();
        if (in.readableBytes() < dataLength) {
            in.resetReaderIndex();
            return;
        }
        byte[] data = new byte[dataLength];
        in.readBytes(data);

        Object obj = SerializationUtil.deserialize(data, genericClass);
        out.add(obj);
    }

}

```

Netty提供了两个变量用于读取和写入：readerIndex用于标识读取索引，writerIndex用于标识写入索引。两个指针将ByteBuf区域分成3部分,从左到右，依次为：已读、可读、可写。你应该能想象出来。两个指针之间的为可读部分，上面的解码器在可读区域先读了4个字节（readInt），这相当于是我们自己定义的数据的头部，记录了数据长度，之后的dataLength个字节才是真正的数据。

markReaderIndex会记录当前的readerIndex，保存到markedReaderIndex,当需要回滚的时候（resetReaderIndex），需要重新把readerIndex赋值为markedReaderIndex。

ByteBuf类中，read和write开头的方法都会移动readerIndex和writerIndex。

SerializationUtil序列化的本期不讲。

编码器：

```
public class RpcEncoder extends MessageToByteEncoder {
    private Class<?> genericClass;

    public RpcEncoder(Class<?> genericClass) {
        this.genericClass = genericClass;
    }

    @Override
    public void encode(ChannelHandlerContext ctx, Object in, ByteBuf out) throws Exception {
        if (genericClass.isInstance(in)) {
            byte[] data = SerializationUtil.serialize(in);
            out.writeInt(data.length);
            out.writeBytes(data);
        }
    }
}
```
业务逻辑：

```
public class RpcHandler extends SimpleChannelInboundHandler<RpcRequest> {
    private final Map<String, Object> handlerMap;

    public RpcHandler(Map<String, Object> handlerMap) {
        this.handlerMap = handlerMap;
    }

    @Override
    public void channelRead0(final ChannelHandlerContext ctx, final RpcRequest request) throws Exception {
        RpcResponse response = new RpcResponse();
        response.setRequestId(request.getRequestId());

        String className = request.getClassName();
        Object serviceBean = handlerMap.get(className);

        String methodName = request.getMethodName();
        Class<?>[] parameterTypes = request.getParameterTypes();
        Object[] parameters = request.getParameters();

        Method method = serviceBean.getClass().getMethod(methodName, parameterTypes);
        method.setAccessible(true);
        Object result = method.invoke(serviceBean, parameters);
        response.setResult(result);
        ctx.writeAndFlush(response);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        ctx.close();
    }
}
```

当数据被成功读取，会触发channelRead0方法。

讨论Netty，避不开NIO，谈NIO，又绕不开I/O多路复用，想不到多年前想绕开的东西，最后还是绕不开，必须攻破。redis、nodejs、ngnix都有I/O多路复用，这个技术值得研究，准备看下I/O多路复用。