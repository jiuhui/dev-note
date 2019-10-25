# kafka生产者参数调优

一段kafka生产者的代码

```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092"); 
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("buffer.memory", 67108864); 
props.put("batch.size", 131072); 
props.put("linger.ms", 100); 
props.put("max.request.size", 10485760); 
props.put("acks", "1"); 
props.put("retries", 10); 
props.put("retry.backoff.ms", 500);

KafkaProducer<String, String> producer = new KafkaProducer<String, String>(props);
```

Kafka的客户端发送数据到服务器，一般都是要经过缓冲的，也就是说，你通过KafkaProducer发送出去的消息都是先进入到客户端本地的内存缓冲里，然后把很多消息收集成一个一个的Batch，再发送到Broker上去的。

buffer.memory和batch.size配置的单位都是字节 linger.ms的单位是毫秒ms

![](..\img\kafka message send.png)

- #### buffer.memory

  Kafka的客户端发送数据到服务器，不是来一条就发一条，而是经过缓冲的，也就是说，通过KafkaProducer发送出去的消息都是先进入到客户端本地的内存缓冲里，然后把很多消息收集成一个一个的Batch，再发送到Broker上去的，这样性能才可能高。

  　　buffer.memory的本质就是用来约束KafkaProducer能够使用的内存缓冲的大小的，默认值33554432字节就是 32MB。

  　　如果buffer.memory设置的太小，可能导致的问题是：消息快速的写入内存缓冲里，但Sender线程来不及把Request发送到Kafka服务器，会造成内存缓冲很快就被写满。而一旦被写满，就会阻塞用户线程，不让继续往Kafka写消息了。 

  　　所以“buffer.memory”参数需要结合实际业务情况压测，需要测算在生产环境中用户线程会以每秒多少消息的频率来写入内存缓冲。经过压测，调试出来一个合理值。。

- #### batch.size

  每个Batch要存放batch.size大小的数据后，才可以发送出去。比如说batch.size默认值是16384字节就是16KB，那么里面凑够16KB的数据才会发送。

   理论上来说，提升batch.size的大小，可以允许更多的数据缓冲在里面，那么一次Request发送出去的数据量就更多了，这样吞吐量可能会有所提升。但是batch.size也不能过大，要是数据老是缓冲在Batch里迟迟不发送出去，那么发送消息的延迟就会很高。一般可以尝试把这个参数调节大些，利用生产环境发消息负载测试一下。

- #### linger.ms

  一个Batch被创建之后，最多过多久，不管这个Batch有没有写满，都必须发送出去了。

  比如说batch.size是16KB，但是现在某个低峰时间段，发送消息量很小。这会导致可能Batch被创建之后，有消息进来，但是迟迟无法凑够16KB，难道此时就一直等着吗？当然不是，假设设置“linger.ms”是50ms，那么只要这个Batch从创建开始到现在已经过了50ms了，哪怕他还没满16KB，也会被发送出去。 所以“linger.ms”决定了消息一旦写入一个Batch，最多等待这么多时间，他一定会跟着Batch一起发送出去。 

  linger.ms配合batch.size一起来设置，可避免一个Batch迟迟凑不满，导致消息一直积压在内存里发送不出去的情况。

- #### max.request.size

  决定了每次发送给Kafka服务器请求消息的最大大小。

  如果发送的消息都是大报文消息，每条消息都是数据较大，例如一条消息可能要20KB。此时batch.size需要调大些，比如设置512KB，buffer.memory也需要调大些，比如设置128MB。 

  只有这样，才能在大消息的场景下，还能使用Batch打包多条消息的机制。

  此时“max.request.size”也得同步增加。

- #### retries和 retries.backoff.ms

  重试机制，也就是如果一个请求失败了可以重试几次，每次重试的间隔是多少毫秒，根据业务场景需要设置。

- #### acks

  - 0

    Producer 往集群发送数据不需要等到集群的返回，不确保消息发送成功。安全性最低但是效率最高。

  - 1

    Producer 往集群发送数据只要 Leader 应答就可以发送下一条，只确保 Leader 接收成功。

  - -1或all

    Producer 往集群发送数据需要所有的ISR Follower 都完成从 Leader 的同步才会发送下一条，确保 Leader 发送成功和所有的副本都成功接收。安全性最高，但是效率最低。