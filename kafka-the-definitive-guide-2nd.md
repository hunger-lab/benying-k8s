## Chapter 2. Kafka Producers: Writing Messages to Kafka

### 1 本章内容

> In this chapter we will learn how to use the Kafka producer, starting with an overview of its design and components. We will show how to create `KafkaProducer` and `ProducerRecord` objects, how to send records to Kafka, and how to handle the errors that Kafka may return. We’ll then review the most important configuration options used to control the producer behavior. We’ll conclude with a deeper look at how to use different partitioning methods and serializers, and how to write your own serializers and partitioners.



### 2 Producer Overview

#### 2.1 Kafka应用举例

> recording user activities for auditing or analysis, recording metrics, storing log messages, recording information from smart appliances, communicating asynchronously with other applications, buffering information before writing to a database, and much more.
>
> 记录用户活动，为了后续的审计和数据分析，记录metrics，存储日志数据，记录智能设备的信息，异步通信，作为数据库的缓冲队列等等。



#### 2.2 Kafka使用的考量

> Those diverse use cases also imply diverse requirements: is every message critical, or can we tolerate loss of messages? Are we OK with accidentally duplicating messages? Are there any strict latency or throughput requirements we need to support?
>
> 是否每条信息都很重要？我们能否容忍信息的丢失？重复数据可不可以？我们有没有严格的延迟或者吞吐量的要求？



举例说明

> In the credit card transaction processing example we introduced earlier, we can see that it is critical to never lose a single message nor duplicate any messages. Latency should be low but latencies up to 500ms can be tolerated, and throughput should be very high—we expect to process up to a million messages a second.
>
> 在前面提到的信用卡交易的例子里，我们发现不丢数据，没有重复数据都是很重要的。延迟应该低点，但是500ms是可以接受的。吞吐量应该很高，我们希望可以处理每秒100w的数据。



#### 2.3 架构图

流程：

* 生成record，必须包括topic和value。也可以加上partition或者key。

* 来到serializer，key和value会被序列化为字节数组，方便网络传输

* 来到partitioner，如果指定了partition，直接返回partition，否则按照key计算一个partition。

* 把record加到目的地相同的批里，有异步线程发到broke。
* 如果添加成功，会返回meta信息，包含topic，partition和offset in partition。否则返回出错信息，producer会重试。

![ktdg 0301](https://learning.oreilly.com/library/view/kafka-the-definitive/9781492043072/assets/ktdg_0301.png)



### 3 构建生产者

#### 3.1 主要属性

* 三个主要属性：*bootstrap.servers*，*key.serializer*，*value.serializer*
* *bootstrap.servers*：broke集群的地址。不需要所有的broke地址，连接建立后，会获取所有的broke。但是仍然建议提供2个broke地址，防止单个broke挂掉。
* *key.serializer*：key的序列化器。该类需要实现org.apache.kafka.common.serialization.Serializer接口。设置key的序列化器是必须的，即使你只想发送value，不发送key。
* *value.serializer*：value的序列化器。



#### 3.2 代码示例

```java
Properties kafkaProps = new Properties(); 
kafkaProps.put("bootstrap.servers", "broker1:9092,broker2:9092");

kafkaProps.put("key.serializer",
    "org.apache.kafka.common.serialization.StringSerializer"); 
kafkaProps.put("value.serializer",
    "org.apache.kafka.common.serialization.StringSerializer");

producer = new KafkaProducer<String, String>(kafkaProps); 
```



#### 3.3 发送消息的方法

* 三个。Fire-and-forget，Synchronous send，Asynchronous send
* Fire-and-forget：只发送消息，然后不管了。如果出错，不会得到任何消息。但是producer会自动重试。
* Synchronous send：发送消息，send方法会返回一个future对象，可以使用future的get方法，得到返回信息。
* Asynchronous send：发送消息，send方法会带一个回调函数，kafka返回的时候，会调用这个回调函数。



### 4 发送消息到kafka

#### 4.1 代码示例

```java
ProducerRecord<String, String> record =
    new ProducerRecord<>("CustomerCountry", "Precision Products",
        "France"); 
try {
    producer.send(record); 
} catch (Exception e) {
    e.printStackTrace(); 
}
```

* key和value的类型必须与key，value序列化器相匹配

* > While we ignore errors that may occur while sending messages to Kafka brokers or in the brokers themselves, we may still get an exception if the producer encountered errors before sending the message to Kafka. Those can be a `SerializationException` when it fails to serialize the message, a `BufferExhaustedException` or `TimeoutException` if the buffer is full, or an `InterruptException` if the sending thread was interrupted.
  >
  > 当我们忽略了发送消息到kafka过程中的错误和kafka broker自身的错误时，我们仍然可能在producer把消息发送到kafka之前得到异常。可能是`SerializationException` 序列化异常，可能是`BufferExhaustedException` 或者`TimeoutException` 异常，当缓冲溢出了。或者是`InterruptException` 异常，当发送线程中断的时候。



#### 4.2 同步发送消息

* 生产中基本不用，因为broke会耗费2ms到几秒的时间，才能返回。

* 代码示例

  > ```java
  > ProducerRecord<String, String> record =
  >     new ProducerRecord<>("CustomerCountry", "Precision Products", "France");
  > try {
  >     producer.send(record).get(); 
  > } catch (Exception e) {
  >     e.printStackTrace(); 
  > }
  > ```

* > `KafkaProducer` has two types of errors. *Retriable* errors are those that can be resolved by sending the message again. For example, a connection error can be resolved because the connection may get reestablished. A “not leader for partition” error can be resolved when a new leader is elected for the partition and the client metadata is refreshed. `KafkaProducer` can be configured to retry those errors automatically, so the application code will get retriable exceptions only when the number of retries was exhausted and the error was not resolved. Some errors will not be resolved by retrying. For example, “message size too large.” In those cases, `KafkaProducer` will not attempt a retry and will return the exception immediately.
  >
  > kafka producer有两类错误，一类是可重试的，重新发送消息即可解决。比如连接错误，连接可以重新建立。比如不是分片的leader，leader可以重新选举，然后client的元信息刷新。对于这类错误，kafka可以配置为自动重试。应用只有在重试次数用完，仍然没有解决时才会收到此类错误。另外一类错误，比如消息体太大，producer会直接返回，不会重试。



#### 4.3 异步发送消息

