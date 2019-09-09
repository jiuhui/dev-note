# gRPC

gRPC官网 why gRPC？

> gRPC is a modern open source high performance RPC framework that can run in any environment. It can efficiently connect services in and across data centers with pluggable support for load balancing, tracing, health checking and authentication. It is also applicable in last mile of distributed computing to connect devices, mobile applications and browsers to backend services.

gRPC是现在的开源的高性能的RPC框架，它可以运行在任何环境，它可以有效的连接数据中心内部服务和across（我理解为跨越连接数据中心的）数据中心的服务，并且支持可插拔负载均衡、链路追踪、健康检查和权限验证，他也可以应用分布式计算的最后一里，将设备、移动应用和浏览器连接到后端服务。



gRpc使用protobuf作为接口定义语言（IDL）.

```protobuf
service HelloService {
  rpc SayHello (HelloRequest) returns (HelloResponse);
}

message HelloRequest {
  string greeting = 1;
}

message HelloResponse {
  string reply = 1;
}
```

gRPC允许定义四种服务方法:

1、一元rpc，客户端向服务器发送一个请求并得到一个响应，就像普通的函数调用一样。

```protobuf
rpc SayHello(HelloRequest) returns (HelloResponse){
}
```

2、服务器流rpc，客户端向服务器发送请求，并获取流以读取消息序列。客户端从返回的流中读取，直到没有更多的消息为止。grpc保证在单个rpc调用中进行消息排序。

```protobuf
rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse){
}
```

3、和2正好相反，是请求时流 返回的普通消息体

```protobuf
rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse) {
}
```

4、2和3组合的，请求和返回都是流

```protobuf
rpc BidiHello(stream HelloRequest) returns (stream HelloResponse){
}
```



gRPC整合gradle生成代码。

1、首先到gRPC github网站 https://github.com/grpc/grpc-java  按照readme里面的指示，引入gradle依赖

```groovy
"io.grpc:grpc-netty-shaded:1.23.0",
"io.grpc:grpc-protobuf:1.23.0",
"io.grpc:grpc-stub:1.23.0"
```

2、Generated Code代码生成，把proto文件放在`src/main/proto` 和`src/test/proto（测试用的）` 目录下，同时加上下面的插件

