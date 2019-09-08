# Thrift

- Thrift最初是由Facebook研发，主要用于各个服务之间的RPC通信，支持跨语言，常用的语言比如C++,Java,Python,PHP,Ruby,Erlang,Perl,C#,javascript,Node.js等。

- Thrift是一个典型的CS结构，客户端和服务端可以使用不同的语言开发，既然客户端和服务端能使用不同的语言开发，那么一定要有一种中间语言来关联客户端和服务器端的语言，这种语言就是IDL（Interface Description Language）。

- Thrift数据类型不支持无符号的类型，因为很多编程语言不支持无符号类型，比如Java。因为RPC框架要支持多语言的开发，那么它支持的数据类型肯定是这写语言支持的数据类型的交集。

- Thrift容器类型，集合中的元素可以是除了service之外的任何类型，包括exception。容器类型有：list:一系列由T类型的数据组成的有序列表，元素可以重复。set：一系列由T类型的数据组成的无序集合，元素不可以重复；map：一个字典结构，key为k类型，value为V类型，相当于java中的HashMap.

- Thrift工作原理，如何实现多语言之间的通信，数据传输使用socket（多语言均支持），数据再以特定的格式（String等）发送，接收方语言进行解析。

- IDL结构体（struct）.就像C语言一样Thrift支持Struct类型，目的就是将一些数据聚合在一起，方便传输管理。Struct的定义形式如下：

  struct People {

  ​	1:string name;

  ​	2:i32 age;

  ​	3:string gender;

  }

- IDL枚举（enum）枚举的定义形式和java的enum定义类似：

  enum Gender{

  ​		MALE,

  ​		FEMALE

  ​	}

- IDl异常（exception） Thrift支持自定义exception，规则与struct一样

  exception RequestException {

  ​	1:i32 code;

  ​	2:string reason;

  }

- IDL 服务（service）   Thrift定义服务相当于java中创建Interface一样，创建的service经过代码生成命令之后就会生成客户端和服务端的框架代码。定义形式如下：

  service HelloWorldService{

  ​	// service中定义的函数，相当于Javainterface中定义的方法

  string doAction(1:string name,2:i32 age);

  }

- 类型定义 Thrift支持类似C++一样的typedef定义，typedef i32 int   typedef i64 long

- 常量 （const） Thrift也支持常量的定义，使用const关键字：

  const i32 MAX_RETRIES_TIME = 10    const string STR_URL = "http://baidu.com"

- 命名空间   Thrift的命名空间相当于Java中的package的意思，主要目的是组织代码，thrift使用关键字namespace定义命名空间：

  namespace java com.fjh.test.thrift

  格式是：namespace 语言名  路径

- 文件包含  Thrift也支持文件包含，相当于C/C++中的include,Java中的import.使用关键字include的定义：

  include "global.thrift"

- 注释   Thrift注释方式支持shell风格的注释，支持// #开头的注释 /**/包裹的语句也是注释。

- 可选与必填    Thrift提供两个关键字  required,optional,分别用于表示对应字段是必填的还是可选的。例如：

  struct People{

  ​	1:required string name;

   	2:optional i32 age;

  }

- Thrift生成代码的命令 thrift --gen java  src/main/java/com/fjh/netty/thrift/data.thrift

- Thrift传输格式：

  TBinaryProtocol     二进制格式

  TCompactProtocol   压缩格式   效率最高

  TJSONProtocol    JSON格式

  TSimpleJSONProtocol  提供JSON只写协议，生成的文件很容易通过脚本语言解析。

  TDebugProtocol   使用易懂的可读的文本格式，以便于debug。

- Thrift数据传输方式：

  - TSocket   阻塞式socket 效率最低 用的最少
  - TFramedTransport   以frame为单位进行传输，非阻塞式服务中使用。
  - TFileTransport   以文件形式进行传输
  - TMemoryTransport  将内存用于I/O。java实现时内部实际使用了简单的ByteArrayOutputStream.
  - TzlibTransport  使用zlib进行压缩，与其他传输方式联合使用，当前无Java实现。

- Thrift支持的服务模型

  - TSimpleServer   简单的单线程服务模型，常用于测试。
  - TThreadPoolServer   多线程服务模型，使用标准的阻塞式IO
  - TNonblockingServer   多线程服务模型，使用非阻塞式IO(需要使用TFramedTransport数据传输方式)
  - THsHaServer  THsHa引入了线程池去处理，其模型把读写任务放到线程池去处理；Half-sync/Half-async的处理模式，Half-aysnc式在处理IO事件上（accept/read/write io）,half-sync用于handler对rpc的同步处理。it relies on the use of TFramedTransport.