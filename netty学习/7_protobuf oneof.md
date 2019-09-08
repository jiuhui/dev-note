# protobuf oneof

MyDataInfo.proto

```protobuf
syntax = "proto2";
package com.fjh.netty.protobuf.oneof;

option optimize_for = SPEED;
option java_package = "com.fjh.netty.protobuf.oneof";
option java_outer_classname = "MyDataInfo";

message Person{
    enum DataType{
        StudentType =1;
        TeacherType = 2;
    }
    required DataType data_type = 1;
    oneof dataBody{
        Student sutdent = 2;
        Teacher teacher = 3;
    }
}

message Teacher{
    optional string name = 1;
    optional string work = 2;
}

message Student{
    optional string name = 1;
    optional int32 age = 2;
    optional string address = 3;
}


```



服务端

```java
public class ProtobufOneOfServer {
    public static void main(String[] args) throws Exception{
        EventLoopGroup boosGrop = new NioEventLoopGroup();
        EventLoopGroup workGrop = new NioEventLoopGroup();

        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(boosGrop,workGrop).channel(NioServerSocketChannel.class)
                    .handler(new LoggingHandler(LogLevel.INFO)).childHandler(new ProtobufOneOfServerInitializer());
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
public class ProtobufOneOfServerInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();
        pipeline.addLast(new ProtobufVarint32FrameDecoder());
        // 这里是MyDataInfo.Person
        pipeline.addLast(new ProtobufDecoder(MyDataInfo.Person.getDefaultInstance()));
        pipeline.addLast(new ProtobufVarint32LengthFieldPrepender());
        pipeline.addLast(new ProtobufEncoder());

        pipeline.addLast(new ProtobufOneOfServerHandler());
    }
}
```

```java
public class ProtobufOneOfServerHandler extends SimpleChannelInboundHandler<MyDataInfo.Person> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, MyDataInfo.Person msg) throws Exception {
        MyDataInfo.Person.DataType dataType = msg.getDataType();
        switch (dataType) {
            case StudentType:
                MyDataInfo.Student sutdent = msg.getSutdent();
                System.out.println(sutdent.getName());
                System.out.println(sutdent.getAge());
                System.out.println(sutdent.getAddress());
            case TeacherType:
                MyDataInfo.Teacher teacher = msg.getTeacher();
                System.out.println(teacher.getName());
                System.out.println(teacher.getWork());
        }
    }
}

```

客户端



```java
public class ProtobufOneOfClient {
    public static void main(String[] args)throws Exception {
        EventLoopGroup eventLoopGroup = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(eventLoopGroup).channel(NioSocketChannel.class)
                    .handler(new ProtobufOneOfClientInitializer());
            ChannelFuture channelFuture = bootstrap.connect("localhost", 8899).sync();
            channelFuture.channel().closeFuture().sync();

        } finally {
            eventLoopGroup.shutdownGracefully();
        }
    }
}
```

```java
public class ProtobufOneOfClientInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        // 客户端的Initializer要和服务端的一样
        ChannelPipeline pipeline = ch.pipeline();
        pipeline.addLast(new ProtobufVarint32FrameDecoder());
        pipeline.addLast(new ProtobufDecoder(MyDataInfo.Person.getDefaultInstance()));
        pipeline.addLast(new ProtobufVarint32LengthFieldPrepender());
        pipeline.addLast(new ProtobufEncoder());

        pipeline.addLast(new ProtobufOneOfClientHandler());
    }
}
```

```java
public class ProtobufOneOfClientHandler extends SimpleChannelInboundHandler<MyDataInfo.Person> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, MyDataInfo.Person msg) throws Exception {

    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        int random = new Random().nextInt(2);
        MyDataInfo.Person person = null;
        if (random == 0){
            person = MyDataInfo.Person.newBuilder()
                    .setDataType(MyDataInfo.Person.DataType.StudentType)
                    .setSutdent( MyDataInfo.Student.newBuilder().setName("学生1")
                            .setAge(20).setAddress("北京").build()
                    ).build();
        } else {
            person = MyDataInfo.Person.newBuilder()
                    .setDataType(MyDataInfo.Person.DataType.TeacherType)
                    .setTeacher(MyDataInfo.Teacher.newBuilder().setName("老师2")
                            .setWork("语文老师").build()).build();

        }
        ctx.writeAndFlush(person);
    }
}

```

