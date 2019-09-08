# netty心跳检测

服务端

```java
public class HeartBeatServer {

    public static void main(String[] args)throws Exception {
        EventLoopGroup boosGroup = new NioEventLoopGroup();
        EventLoopGroup workGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap().group(boosGroup,workGroup)
                    .channel(NioServerSocketChannel.class)
                    // 增加一个全局日志的handler
                    .handler(new LoggingHandler())
                    .childHandler(new HeartBeatServerInitializer());
            ChannelFuture channelFuture = serverBootstrap.bind(8899).sync();
            channelFuture.channel().closeFuture().sync();
        } finally {
            boosGroup.shutdownGracefully();
            workGroup.shutdownGracefully();
        }
    }
}

```

```java
public class HeartBeatServerInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();
        // add 心跳检查handle 三个参数 readerIdleTimeSeconds  writerIdleTimeSeconds   allIdleTimeSeconds
        pipeline.addLast(new IdleStateHandler(0,0,10));
        pipeline.addLast(new DelimiterBasedFrameDecoder(4096,Delimiters.lineDelimiter()));
        pipeline.addLast(new StringDecoder(CharsetUtil.UTF_8));
        pipeline.addLast(new StringEncoder(CharsetUtil.UTF_8));
        pipeline.addLast(new HeartBeatServerHeartEventhandler());
        pipeline.addLast(new HeartBeatServerHandler());
    }
}
```

```java
public class HeartBeatServerHeartEventhandler extends ChannelInboundHandlerAdapter {

    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        // 在出现超时会触发
        if (evt instanceof IdleStateEvent){
            IdleStateEvent event = (IdleStateEvent)evt;
            if (event.state() == IdleState.READER_IDLE){
                System.out.println("reader_idle");
            } else if (event.state() == IdleState.WRITER_IDLE){
                System.out.println("writer_idle");
            } else if (event.state() == IdleState.ALL_IDLE){
                System.out.println("all_idle");
            }
        }
    }
}
```

```java
public class HeartBeatServerHandler extends SimpleChannelInboundHandler<String> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
        System.out.println(msg);
        Channel channel = ctx.channel();
        channel.writeAndFlush("服务端收到信息 ack:"+msg+"\n");
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        Channel channel = ctx.channel();
        System.out.println(channel.remoteAddress() + "上线");
    }

    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        Channel channel = ctx.channel();
        System.out.println(channel.remoteAddress()+"下线");
    }
}
```



客户端：

```java
public class HeartBeatClient {
    public static void main(String[] args)throws Exception {
        EventLoopGroup eventLoopGroup = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(eventLoopGroup).channel(NioSocketChannel.class)
                    .handler(new HeartBeatClientInitializer());
            ChannelFuture channelFuture = bootstrap.connect("localhost", 8899).sync();
            Channel channel = channelFuture.channel();
            BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
            for (;;){
                String s = br.readLine();
                System.out.println(s);
                channel.writeAndFlush(s+"\n\r");
            }
        } finally {
            eventLoopGroup.shutdownGracefully();
        }
    }
}

```

```java
public class HeartBeatClientInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();
        pipeline.addLast(new IdleStateHandler(0,0,10));
        pipeline.addLast(new DelimiterBasedFrameDecoder(4096,Delimiters.lineDelimiter()));
        pipeline.addLast(new StringDecoder(CharsetUtil.UTF_8));
        pipeline.addLast(new StringEncoder(CharsetUtil.UTF_8));
        pipeline.addLast(new HeartBeatClientHeartEventHandler());
        pipeline.addLast(new HeartBeatClientHandle());
    }
}

```

```java
public class HeartBeatClientHeartEventHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        if (evt instanceof IdleStateEvent){
            ctx.writeAndFlush("心跳\n");
        }
    }
}
```

```java
public class HeartBeatClientHandle extends SimpleChannelInboundHandler<String> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
        System.out.println(msg);
    }
}

```

