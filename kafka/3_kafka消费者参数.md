# kafka消费者参数

#### 三、consumer主要配置

##### 1.连接配置

| 配置项            | 作用             | 类型   | 默认值 | 示例           |
| ----------------- | ---------------- | ------ | ------ | -------------- |
| bootstrap.servers | 连接的broker地址 | String | null   | localhost:9092 |

##### 2.消费者本身配置

| 配置项    | 作用         | 类型   | 默认值 | 示例       |
| --------- | ------------ | ------ | ------ | ---------- |
| group.id  | 消费者组的ID | String | null   | user-group |
| client.id | 消费者ID     | String | null   | appName    |

##### 3.消息配置

| 配置项                  | 作用                                                         | 类型    | 默认值   | 示例                                                     |
| ----------------------- | ------------------------------------------------------------ | ------- | -------- | -------------------------------------------------------- |
| auto.offset.reset       | 当没有初始偏移量或当前偏移量不存在时设置消费的起始点(earliest、latest、none三个值可选择) | String  | earliest | /earliest                                                |
| enable.auto.commit      | 是否后台自动提交偏移量                                       | Boolean | true     | true                                                     |
| auto.commit.interval.ms | 后台自动提交消费偏移量(后台定时提交)的间隔时间               | int     | 5000     | 5000                                                     |
| key.deserializer        | 指定消息的key的反序列化类(需要实现Deserializer接口)          | class   | null     | org.apache.kafka.common.serialization.StringDeserializer |
| value.deserializer      | 指定消息内容的反序列化类(需要实现Deseri                      |         |          |                                                          |

https://blog.csdn.net/chaiyu2002/article/details/89472416    理解Kafka消费者属性的enable.auto.commit