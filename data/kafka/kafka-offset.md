# Kafka 偏移量提交 与 分区再平衡

## 偏移量

### 偏移量概念与问题

offset 偏移量： 是kafka用来确定消息是否被消费过的标识，​偏移量是kafka特别重要的一个概念，特别是在消费者端。

偏移量是一个自增长的ID 用来标识当前分区的哪些消息被消费过了， 这个ID会保存在kafka的broker当中 而且 消费者本地也会存储一份 因为每次消费每一条消息都要更新一下偏移量的话 难免会影响整个broker的吞吐量 所以一般 这个偏移量在每次发生改动时 先由消费者本地改动， 默认情况下 消费者每五秒钟会提交一次改动的偏移量， 这样做虽然说吞吐量上来了， 但是可能会出现重复消费的问题: 因为可能在下一次提交偏移量之前 消费者本地消费了一些消息，然后发生了分区再均衡， 那么就会出现一个问题： 假设上次提交的偏移量是 2000， 在下一次提交之前， 其实消费者又消费了500条数据， 也就是说当前的偏移量应该是2500， 但是这个2500只在消费者本地， 也就是说 假设其他消费者去消费这个分区的时候拿到的偏移量是2000， 那么又会从2000开始消费消息， 那么， 2000到2500之间的消息又会被消费一遍， 这就是重复消费的问题.

### 偏移量手动提交

kafka对于这种问题也提供了解决方案:手动提交

可以关闭默认的自动提交(enable.auto.commit= false) 然后使用kafka提供的API来进行偏移量提交:

Kafka提供了两种方式提交你的偏移量: 同步和异步

1. 同步提交偏移量
   - 等待服务器应答 并且遇到错误会尝试重试，
   - 一定程度上影响性能
   - 能够确保偏移量到底提交成功与否
2. 异步提交偏移量
   - 相比同步提交有性能提升
   - 但是遇到错误没办法重试
   - 可能在收到你这个结果的时候又提交过偏移量了 如果这时候重试 又会导致消息重复的问题了

采用同步+异步的方式来保证提交的正确性以及服务器的性能

因为 异步提交的话， 如果出现问题但是不是致命问题， 可能下一次提交就不会出现这个问题了， 因此 有些异常是不需要解决的， 所以 我们平时可以采用异步提交的方式 等到消费者中断了(遇到了致命问题，或是强制中断消费者) 的时候再使用同步提交(因为这次如果失败了就没有下次了… 所以要让他重试) 。

```java
try {
    while (true) {
        ConsumerRecords<String, String> poll = kafkaConsumer.poll(500);
        for (ConsumerRecord<String, String> context : poll) {
            System.out.println("消息所在分区:" + context.partition()
                    + "-消息的偏移量:" + context.offset()
                    + "key:" + context.key()
                    + "value:" + context.value());
        }
        //正常情况异步提交
        //异步提交偏移量
        kafkaConsumer.commitAsync();
    }
} catch (Exception e) {
    e.printStackTrace();
} finally {
    try {
        //当程序中断时同步提交
        //同步提交偏移量
        kafkaConsumer.commitSync();
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        //关闭当前消费者
        kafkaConsumer.close();
    }
}
```

值得一提的是 在手动提交时kafka提供了你可以传入具体的偏移量来完成提交 也就是指定偏移量提交。

如果指定的偏移量 小于 分区所存储的偏移量大小的话 那么会导致消息重复消费， 如果指定的偏移量大于分区所存储的偏移量的话，那么会导致消息丢失.

```java
Map<TopicPartition, OffsetAndMetadata> offset = new HashMap<>();
//这里就指定了test-topic这个主题下的分区1
//OffsetAndMetadata:
//    第一个参数为你要提交的偏移量
//    第二个参数可以选择性的传入业务ID
//    可以拿来确定这次提交
// 这里直接提交偏移量为0
// 那么会导致下个消费者或者说分区再均衡之后再来读取这个分区的数据会从第一条开始读取
offset.put(new TopicPartition("test-topic", 1), new OffsetAndMetadata(0, "1"));
//指定偏移量提交，参数为map集合
//  key为指定的主题下的分区
//  value为你要提交的偏移量
kafkaConsumer.commitSync(offset);
```

## 分区再平衡 Rebalance

当触发Rebalance kafka重新分配分区所有权

以下情况会触发Rebalance：

1. 组成员发生变更(新consumer加入组、已有consumer主动离开组或已有consumer崩溃了)
2. 订阅主题数发生变更，如果你使用了正则表达式的方式进行订阅，那么新建匹配正则表达式的topic就会触发rebalance
3. 订阅主题的分区数发生变更


**何为分区所有权？**

消费者在消费者组里，消费主题时有以下规则：
- 一个消费者可以消费多个分区
- 一个分区只能被一个消费者消费

如果我有分区 0, 1, 2. 现在有消费者 A, B， kafka可能会让消费者A消费 0，1 这2个分区，那么这时候 我们就会说 消费者A 拥有 分区 0,1的所有权。

当触发Rebalance时，kafka会重新分配这个所有权

比如前面的例子中，消费者B 退出了kafka，这时候kafka会重新分配一下所有权。此时整个消费者组只有A一个消费者，那么0, 1, 2 三个分区的所有权都会属于A。同理，如果这时候有消费者C加入组，那么kafka会确保消费者的负载均衡，让每个消费者A,B,C都能消费一个分区

当触发Rebalance时 由于kafka正在分配所有权 会导致消费者不能消费， 而且 还会引发一个重复消费的问题， 当消费者还没来得及提交偏移量时 分区所有权遭到了重新分配 那么这时候就会导致一个消息被多个消费者重复消费

那么 解决方案就是 在消费者订阅时， 添加一个再均衡监听器， 也就是当kafka在做Rebalance 操作前后 均会调用再均衡监听器 那么这时候 我们可以在kafka Rebalance之前提交我们消费者最后处理的消息来解决这个问题。


## 退出消费

当我们不需要某个消费者继续消费kafka当中的数据时，可以选择调用Close方法来关闭它，在关闭之前close方法会发送一个通知告诉kafka这个消费者推出了，那么kafka就会准备Rebalance，如果采用自动提交偏移量，消费者自身也会关闭自己之前提交最后所消费的偏移量。

消费者进程被中断，kafka也会捕获到消费者的退出。