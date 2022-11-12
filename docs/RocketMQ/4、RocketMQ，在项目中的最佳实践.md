# 1、消息幂等
为了防止消息重复消费导致业务处理异常，消费者在接收到消息后，有必要根据业务上的唯一Key对消息做幂等处理。
例如：
理想场景：库存扣减，生成者发送1条扣减库存100的消息，最终消费成功，数据库减掉100。
如果因网络不稳定等原因导致扣减消息重复投递，消费者重复消费了该扣减消息，最终会导致库存多次扣减，出现超卖的情况。
因此在发送消息时，增加一个分布式唯一Key，消费时判断该Key是否已经消费，已消费则直接跳过，则最终保障了一条信息只消费一次。
```java
Message message = new Message();
message.setKey("1001");
SendResult sendResult = producer.send(message);  

consumer.subscribe("stock_topic", "*", new MessageListener() {
    public Action consume(Message message, ConsumeContext context) {
        String key = message.getKey();
        // 根据业务唯一标识的Key做幂等处理。
    }
});                  
```
![2022111217841.png](https://restest.sx.ink/2022111217841.png)

# 2、指定时间节点消费
一个订单Topic，已经在线上运行许久时间，在正常生产消息、消费消息。
此时新增了一个业务，也需要消费订单的消息，为了减少代码耦合，规避对历史消费者的影响，技术方案设计决定新增一个消费组，消费订单Topic的消息。
RocketMQ消费者默认配置,一个新的订阅组第一次启动从队列的最后位置开始消费，后续再启动接着上次消费的进度开始消费,即跳过历史消息；
```java
consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_LAST_OFFSET);
```
在这种场景下，新消费者没有历史消费定位，会把历史未过期的消息全部消费一遍，其实新的业务不需要历史的订单消息，则需要在新消费组中增加时间节点[bornTimestamp]过滤。
```java
 @Override
public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
    boolean bool = true;
    for (MessageExt msg : list) {
        //小于2021-03-12 12:15:37的消息，则直接返回成功/过滤。
        if (msg.getBornTimestamp() < 1615522537473L) {
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        }
        // 业务逻辑
    }
}
```

# 3、发送延迟消息
该机制应用在定时检查、定时过期的场景下，使用简单，带来惊奇的效果。
如：未支付的订单20分钟失效、视频压缩定时检查压缩结果、支付成功后延迟反查第三方支付成功状态时间，规避回调失败的情况。
```java
Message message = new Message();
message.setTopic("xxx_topic");
message.setBody("".getBytes());
message.setKeys("uuid");
/**
* MQ延迟级别
* 1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h 6h 12h 1d 3d 5d 7d
*/
message.setDelayTimeLevel(2);// 延迟5s
SendResult sendResult = wxMessagePushProducer.send(message);
```
# 4、消息重试
消息在消费方消费失败后，RocketMQ服务端会重新进行消息的投递，直到消费者成功消费消息，重试有次数限制，默认 16 次，每次重试的间隔时间也不一致。

消息重试在一定程度上保证了消息不丢失，通过重试来达到最终被消费的目的。需要注意的是消费者在消费的时候一定要等本地业务成功后才能进行 ACK(消费确认)，不然就会出现消费失败，但是已经 ACK，消息将不会重复投递。

需要做好对应的监控，如果重试了 4，5 次还是失败的，基本上后面重试也是失败的。这个时候需要让开发人员知道，需人工介入处理。或者直接监控死信队列。
# 5、消费类型（顺序、乱序）
顺序消费会损失一定的性能，需根据业务场景选择是否需要顺序消费。
顺序消费案例：电商的订单创建，以订单ID作为Sharding Key，那么同一个订单相关的创建订单消息、订单支付消息、订单退款消息、订单物流消息都会按照发布的先后顺序来消费。

# 6、Tag应用场景
在RocketMQ中，Topic与Tag都是业务上用来归类的标识，区分在于Topic是一级分类，而Tag可以理解为是二级分类。

**Tag适用场景**
- 业务区分：电器类订单、女装类订单、化妆品类订单的消息。
- CRUD区分：增加(Create)、读取(Read)、更新(Update)和删除(Delete)
```java
Message msg = new Message("xxx_topic",
        JSON.toJSONString(object).getBytes(RemotingHelper.DEFAULT_CHARSET));
msg.setKeys(UUIDBuild.getUUID());
msg.setTags("create"); // 区分消息类型：create、update、delete
msgProvider.send(msg);
```