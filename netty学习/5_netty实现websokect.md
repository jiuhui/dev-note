# netty实现websocket

服务端

```java
public class WebSocketServer {
    public static void main(String[] args) throws Exception{
        EventLoopGroup boosGroup = new NioEventLoopGroup();
        EventLoopGroup workGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap().group(boosGroup,workGroup)
                    .channel(NioServerSocketChannel.class)
                    .handler(new LoggingHandler(LogLevel.INFO)).childHandler(new WebSocketChannelInitializer());
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
public class WebSocketChannelInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();
        // A  combination of HttpRequestDecoder and HttpResponseEncoder
        // which enables easier server side HTTP implementation.
        pipeline.addLast("httpServerCodec",new HttpServerCodec());
        // A ChannelHandler that adds support for writing a large data stream
        // asynchronously neither spending a lot of memory nor getting OutOfMemoryError.
        // Large data streaming such as file transfer requires complicated state management
        // in a ChannelHandler implementation. ChunkedWriteHandler manages such complicated
        // states so that you can send a large data stream without difficulties.
        pipeline.addLast("chunkedWriteHandle",new ChunkedWriteHandler());
        // 将HttpMessage及其后续HttpContents聚合为单个完整HttpRequest或完整HttpResponse
        pipeline.addLast("httpObjectAggregator",new HttpObjectAggregator(8192));
        pipeline.addLast(new WebSocketServerProtocolHandler("/ws"));
        pipeline.addLast(new WebSocketServerHandler());

    }
}
```

```java
public class WebSocketServerHandler extends SimpleChannelInboundHandler<TextWebSocketFrame> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, TextWebSocketFrame msg) throws Exception {
        System.out.println("收到消息："+msg.text());
        ctx.channel().writeAndFlush(new TextWebSocketFrame("服务器时间："+ LocalDateTime.now()));
    }

    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        System.out.println("handlerAdded "+ ctx.channel().id().asLongText());
    }

    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
        System.out.println("handlerRemoved:"+ctx.channel().id().asLongText());
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        System.out.println("异常发生");
        ctx.close();
    }
}

```



客户端

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>webSock客户端</title>
</head>
<body>
<script type="text/javascript">
    var socket;
    if (window.WebSocket){
        socket= new WebSocket("ws://localhost:8899/ws");

        socket.onmessage = function (ev) {
            var ta = document.getElementById("responseText");
            ta.value = ta.value + "\n" + ev.data;
        }
        socket.onopen = function (ev) {
            var ta = document.getElementById("responseText");
            ta.value = "连接开启";
        }
        socket.onclose = function (ev) {
            var ta = document.getElementById("responseText");
            ta.value = ta.value + "\n" + "连接关闭";
        }
    } else {
        alert("浏览器不支持websocket");
    }

    function send(message) {
        if (!window.WebSocket) {
            return;
        }
        if (socket.readyState == WebSocket.OPEN){
            socket.send(message);
        }

    }

</script>

<form onsubmit="return false;">
    <textarea name="message" style="width: 400px;height: 200px"></textarea>
    <input type="button" value="发送消息" onclick="send(this.form.message.value);">

    <h3>服务器端输出</h3>
    <textarea id="responseText" style="width: 400px;height: 300px"></textarea>
    <input type="button" onclick="javascript:document.getElementById('responseText').value=''" value="清空内容">

</form>
</body>
</html>
```

