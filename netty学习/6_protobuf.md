# protobuf

RPC简介：

> rmi: remote method invocation 只针对java序列化与反序列化  也叫做：编码与解码
>
> RPC：Remote procedure Call 远程过程调用，
>
> 很多RPC框架是跨语言的
>
> 1、定义一个接口的说明文档，描述了对象（结构体），对象成员、接口方法等一系列语言
>
> 2、通过RPC框架所提供的编译器，将接口说明文件编译成具体语言文件
>
> 3、在客户端和服务器端各引入RPC编译器所编译的文件，即可像调用本地方法一样调用远程方法



这个是一个netty实现的server和client通过protobuf传输的程序， 其中ProtobufServerInitialiszer和protobufClientInitializer这两个类里面 addLast的handler里面pipeline.addLast(new ProtobufDecoder(DataInfo.Student.getDefaultInstance())); 这个handler是用来解码的，就是把字节数组转成对象的。现在这个程序的问题是 如果我们要传输的对象很多，不仅仅是DataInfo.Student这一个对象，我们应该怎么做呢。 由两种办法： 1、就是自定义解码器，因为socket底层传输的都是字节流，我们可以通过在定义 协议的时候通过魔数来分开，什么意思呢，就是比如AA后面跟一个对象， BB后面跟一个对象，这样在自定义解码器的时候就通过可以通过判断来解码了。 2、protobuf协议里面的关键字oneof定义。通过在消息体里面定义要传输的对象们的 枚举类来指定要传输哪个对象，然后再定义oneof数据体。比如:

```protobuf
 message Person{
    enum DataType{
        StudentType = 1;
        TeacherType = 2;
    }
    required DataType data_type = 1;
    oneof dataBody {
        Student student = 2;
        Teacher teacher = 3;
    }
}

message Student{
    optional string name = 1;
    optional int32 age = 2;
}
message Teacher{
    optional string name =1;
    optional string age = 2;
}
```

通过这种方式定义协议，解码器解码的就是Person，通过DataType区分到底传输的是什么对象

protobuf生成Java代码的命令 protoc --java_out=src/main/java src/main/java/com/fjh/netty/protobuf/Student.proto

一个protobuf使用的最佳实践： 因为我们在平时的开发中，server端和client端往往是不在一个仓库里面的，那么用protobuf工具生成的java类怎么 在两个仓库中同步呢。 往往我们把protobuf的java文件也放在一个单独的git仓库中。 现在我们由三个仓库分别是： serverproject protobufproject clientproject 使用git作为版本控制工具的时候，git submodule命令意思是git仓库中的子仓库 我们可以把protobufprject作为serverproject和clientproject的子仓库 ，这样如果protobufproject由更新的话，我们无论是在server还是client，在pull自己项目的时候就能拉到protobuf的更新了。

还可以用git subtree,但是上面的方法会好点

homebrew mac上面的包管理器



Student.proto

```protobuf
syntax = "proto2";
package com.fjh.netty.protobuf;

option optimize_for = SPEED;
option java_package = "com.fjh.netty.protobuf";
option java_outer_classname = "DataInfo";

message Student{
    required string name = 1;
    optional int32 age = 2;
    optional string address = 3;
}

```

```java
public class ProtobufTest {
    public static void main(String[] args) throws Exception{
        DataInfo.Student student = DataInfo.Student.newBuilder().setName("张三").setAge(30).setAddress("北京").build();
        byte[] bytes = student.toByteArray();

        DataInfo.Student student1 = DataInfo.Student.parseFrom(bytes);
        System.out.println(student1);
        System.out.println(student1.getName());
        System.out.println(student1.getAge());
        System.out.println(student1.getAddress());
    }
}
```

服务器端：

```java
public class ProtobufServer {
    public static void main(String[] args) throws Exception{
        EventLoopGroup boosGrop = new NioEventLoopGroup();
        EventLoopGroup workGrop = new NioEventLoopGroup();

        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(boosGrop,workGrop).channel(NioServerSocketChannel.class)
                    .handler(new LoggingHandler(LogLevel.INFO)).childHandler(new ProtobufServerInitializer());
            ChannelFuture channelFuture = serverBootstrap.bind(8899).sync();
            channelFuture.channel().closeFuture().sync();
        } finally {
            boosGrop.shutdownGracefully();
            workGrop.shutdownGracefully();
        }
    }
}
```

```java
public class ProtobufServerInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();
        pipeline.addLast(new ProtobufVarint32FrameDecoder());
        pipeline.addLast(new ProtobufDecoder(DataInfo.Student.getDefaultInstance()));
        pipeline.addLast(new ProtobufVarint32LengthFieldPrepender());
        pipeline.addLast(new ProtobufEncoder());

        pipeline.addLast(new ProtobufServerHandler());
    }
}
```

```java
public class ProtobufServerHandler extends SimpleChannelInboundHandler<DataInfo.Student> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, DataInfo.Student msg) throws Exception {
        System.out.println(msg.getAddress());
        System.out.println(msg.getName());
        System.out.println(msg.getAddress());

        DataInfo.Student student = DataInfo.Student.newBuilder().setName("李四")
                .setAge(20).setAddress("上海").build();
        ctx.channel().writeAndFlush(student);
    }
}
```

客户端

```java
public class ProtobufClient {
    public static void main(String[] args)throws Exception {
        EventLoopGroup eventLoopGroup = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(eventLoopGroup).channel(NioSocketChannel.class)
                    .handler(new ProtobufClientInitializer());
            ChannelFuture channelFuture = bootstrap.connect("localhost", 8899).sync();
            channelFuture.channel().closeFuture().sync();

        } finally {
            eventLoopGroup.shutdownGracefully();
        }
    }
}
```

```java
public class ProtobufClientInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();
        pipeline.addLast(new ProtobufVarint32FrameDecoder());
        pipeline.addLast(new ProtobufDecoder(DataInfo.Student.getDefaultInstance()));
        pipeline.addLast(new ProtobufVarint32LengthFieldPrepender());
        pipeline.addLast(new ProtobufEncoder());

        pipeline.addLast(new ProtobufClientHandler());
    }
}
```

```java
public class ProtobufClientHandler extends SimpleChannelInboundHandler<DataInfo.Student> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, DataInfo.Student msg) throws Exception {
        System.out.println(msg.getName());
        System.out.println(msg.getAge());
        System.out.println(msg.getAddress());
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        DataInfo.Student student = DataInfo.Student.newBuilder().setName("张三")
                .setAge(30).setAddress("北京").build();

        ctx.writeAndFlush(student);

    }
}

```

