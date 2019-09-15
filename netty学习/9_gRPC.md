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

2、Generated Code代码生成，把proto文件放在`src/main/proto` 和`src/test/proto（测试用的）` 目录下，同时加上下面的插件  下面是build.gradle文件。

```groovy
group 'com.fjh'
version '1.0'

apply plugin: 'java'
apply plugin: 'com.google.protobuf'
sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
    maven {
        url "https://maven.aliyun.com/nexus/content/groups/public"
    }
    mavenCentral()
}

dependencies {
    compile(
            "io.netty:netty-all:4.1.0.Final",
            "com.google.protobuf:protobuf-java:3.9.1",
            "com.google.protobuf:protobuf-java-util:3.9.1",
            "org.apache.thrift:libthrift:0.12.0",
            "io.grpc:grpc-netty-shaded:1.23.0",
            "io.grpc:grpc-protobuf:1.23.0",
            "io.grpc:grpc-stub:1.23.0"
    )
}

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'com.google.protobuf:protobuf-gradle-plugin:0.8.10'
    }
}
protobuf {
    protoc {
        artifact = "com.google.protobuf:protoc:3.9.0"
    }
    plugins {
        grpc {
            artifact = 'io.grpc:protoc-gen-grpc-java:1.23.0'
        }
    }
    generateProtoTasks.generatedFilesBaseDir = "src"
    generateProtoTasks {
        all()*.plugins {
            grpc {
                setOutputSubDir 'java'
            }
        }
    }
}
```

GRPC使用举例：

首先定义proto文件：

```java
syntax = "proto3";

package com.fjh.netty.grpc;
option java_package = "com.fjh.netty.grpc";
option java_outer_classname = "StudentProto";
//是否生成多个文件
option java_multiple_files = true;

service StudentService {
    rpc GetRealNameByUsername(MyRequest) returns (MyResponse){}
    rpc GetStudentsByAge(StudentRequest) returns (stream StudentResponse){}
    rpc GetStudentsWrapperByAges(stream StudentRequest) returns (StudentResponseList){}
    rpc BiTalk(stream StreamRequest) returns (stream StreamResponse){}
}

message MyRequest {
    string username = 1;
}

message MyResponse {
    string realname = 1;
}

message StudentRequest {
    string name = 1;
    int32 age = 2;
    string city = 3;
}

message StudentResponse {
    string name = 1;
    int32 age = 2;
    string city = 3;
}

message StudentResponseList {
    repeated StudentResponse studentResponse = 1;
}

message StreamRequest {
    string request_info = 1;
}
message StreamResponse {
    string response_info = 1;
}

```

然后执行 gradle generateProto命令生成对应的java文件。也可以直接在ide上快捷执行命令。

service的实现类。

```java
public class StudentServiceImpl  extends StudentServiceGrpc.StudentServiceImplBase {
    @Override
    public void getRealNameByUsername(MyRequest request, StreamObserver<MyResponse> responseObserver) {
        System.out.println("收到客户端参数："+ request.getUsername());
        responseObserver.onNext(MyResponse.newBuilder().setRealname("张三").build());
        responseObserver.onCompleted();
    }

    @Override
    public void getStudentsByAge(StudentRequest request, StreamObserver<StudentResponse> responseObserver) {
        System.out.println("收到客户端参数："+ request.getAge());
        responseObserver.onNext(StudentResponse.newBuilder().setName("张三").setAge(10).setCity("北京").build());
        responseObserver.onNext(StudentResponse.newBuilder().setName("李四").setAge(20).setCity("上海").build());
        responseObserver.onCompleted();
    }

    @Override
    public StreamObserver<StudentRequest> getStudentsWrapperByAges(StreamObserver<StudentResponseList> responseObserver) {
        return new StreamObserver<StudentRequest>() {;
            @Override
            public void onNext(StudentRequest value) {
                System.out.println("onNext:"+value.getAge());
            }

            @Override
            public void onError(Throwable t) {
                System.out.println(t.getMessage());
            }

            @Override
            public void onCompleted() {
                StudentResponse studentResponse1 = StudentResponse.newBuilder().setName("张三").setAge(10).setCity("北京").build();
                StudentResponse studentResponse2 = StudentResponse.newBuilder().setName("李四").setAge(20).setCity("上海").build();

                StudentResponseList studentResponseList =
                        StudentResponseList.newBuilder().addStudentResponse(studentResponse1)
                        .addStudentResponse(studentResponse2).build();
                responseObserver.onNext(studentResponseList);
                responseObserver.onCompleted();
            }
        };
    }

    @Override
    public StreamObserver<StreamRequest> biTalk(StreamObserver<StreamResponse> responseObserver) {
        return new StreamObserver<StreamRequest>() {
            @Override
            public void onNext(StreamRequest value) {
                System.out.println(value.getRequestInfo());
                responseObserver.onNext(StreamResponse.newBuilder().setResponseInfo(UUID.randomUUID().toString()).build());
            }

            @Override
            public void onError(Throwable t) {
                System.out.println(t.getMessage());
            }

            @Override
            public void onCompleted() {
                responseObserver.onCompleted();
            }
        };
    }
}

```

GRPC服务端 ：

```java
public class GRpcServer {
    private Server server;
    // 服务启动
    private void start() throws IOException {
        this.server = ServerBuilder.forPort(8899).addService(new StudentServiceImpl()).build().start();

        System.out.println("server started");
        // jvm回调钩子  当jvm进程结束关闭连接
        Runtime.getRuntime().addShutdownHook(new Thread(()->{
            System.out.println("关闭jvm");
            GRpcServer.this.stop();
        }));

        System.out.println("执行到这里");
    }
    // 服务关闭
    private void stop(){
        if (this.server != null){
            this.server.shutdown();
        }
    }
    // 等待终止  就是让服务阻塞，等待连接
    private void awaitTermination() throws InterruptedException {
        if (this.server != null){
            this.server.awaitTermination();
        }
    }

    public static void main(String[] args) throws IOException, InterruptedException {
        GRpcServer server = new GRpcServer();
        server.start();
        // 如果没有这行代码 启动之后直接退出了
        server.awaitTermination();
    }
}

```

GRPC客户端调用service方法

```java
public class GRpcClient {
    public static void main(String[] args) {
        ManagedChannel managedChannel = ManagedChannelBuilder.forAddress("localhost",8899)
                .usePlaintext().build();
        // 阻塞的stub
        StudentServiceGrpc.StudentServiceBlockingStub blockingStub = StudentServiceGrpc
                .newBlockingStub(managedChannel);
        // 异步的stub
        StudentServiceGrpc.StudentServiceStub stub = StudentServiceGrpc.newStub(managedChannel);

        // 客户端调用第一个方法
        System.out.println("===============调用第一个方法============");
        MyResponse myResponse
                = blockingStub.getRealNameByUsername(MyRequest.newBuilder().setUsername("zhangsan").build());
        System.out.println(myResponse.getRealname());
        blockingStub.getRealNameByUsername(MyRequest.newBuilder().setUsername("zhangsan").build());

        System.out.println("===========调用第二个方法====================");
        // 客户端调用第二个方法
        Iterator<StudentResponse> studentsByAge = blockingStub.getStudentsByAge(StudentRequest.newBuilder().setAge(10).build());
        studentsByAge.forEachRemaining(studentResponse -> {
            System.out.println(studentResponse.getName());
            System.out.println(studentResponse.getAge());
            System.out.println(studentResponse.getCity());
        });

        // 调用第三个方法 第三个方法要用stub
        System.out.println("=========调用第三个方法===============");
        StreamObserver<StudentResponseList> responseListStreamObserver = new StreamObserver<StudentResponseList>() {
            @Override
            public void onNext(StudentResponseList value) {
                value.getStudentResponseList().forEach( studentResponse -> {
                    System.out.println(
                            studentResponse.getName()+","+ studentResponse.getAge() +","+studentResponse.getCity());
                });
            }

            @Override
            public void onError(Throwable t) {
                System.out.println(t);
            }

            @Override
            public void onCompleted() {
                System.out.println("onCompleted");
            }
        };
        StreamObserver<StudentRequest> studentsWrapperByAges = stub.getStudentsWrapperByAges(responseListStreamObserver);
        studentsWrapperByAges.onNext(StudentRequest.newBuilder().setAge(10).getDefaultInstanceForType());
        studentsWrapperByAges.onNext(StudentRequest.newBuilder().setAge(20).getDefaultInstanceForType());
        studentsWrapperByAges.onCompleted();
        // 第三个方法这里需要等待，因为是流的结果，这里sleep一下，不然不会由结果的
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 调用第四个方法
        System.out.println("=================调用第四个方法================");
        StreamObserver<StreamRequest> requestStreamObserver = stub.biTalk(new StreamObserver<StreamResponse>() {
            @Override
            public void onNext(StreamResponse value) {
                System.out.println(value.getResponseInfo());
            }

            @Override
            public void onError(Throwable t) {
                System.out.println(t.getMessage());
            }

            @Override
            public void onCompleted() {
                System.out.println("onCompleted");
            }
        });

        for (int i = 0; i < 10 ; i++) {
            requestStreamObserver.onNext(StreamRequest.newBuilder().setRequestInfo(LocalDateTime.now().toString()).build());
        }

        // 第三个方法这里需要等待，因为是流的结果，这里sleep一下，不然不会由结果的
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        //

        //managedChannel.shutdown();
    }
}
```

