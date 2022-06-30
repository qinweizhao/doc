# RabbitMQ-消息确认机制

保证消息的可靠抵达，RabbitMQ 给出了两种方案，使用**事务消息**或**引入确认机制**。但事务模式效率有点低（据听说是 250 倍）。

消息的确认机制包含**消息发送确认**和**消息接收确认**两方面，生产者发送消息到达交换机触发回调。交换机未将消息路由到 Queue 触发回调；消费者开启手动确认确认。

![2022-06-30_180230](https://img.qinweizhao.com/2022/06/2022-06-30_180230.png)



## 一、准备环境

创建 SpringBoot 项目：

### 1、导包

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
```

### 2、配置

```yaml
spring:
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    virtual-host: /
    username: guest
    password: guest
    # 开启消息是否已经发送到交换机的确认机制
    publisher-confirm-type: correlated
    # 开启消息未成功投递到目标队列时将消息返回
    publisher-returns: true
    # 设置消费者需要手动确认消息
    listener:
      simple:
        acknowledge-mode: manual
      direct:
        acknowledge-mode: manual
```

### 3、创建交换机、队列，并完成绑定

```java
/**
 * @author qinweizhao
 * @since 2022/6/30
 */
@Configuration
public class MyMqConfig {

    private static final Logger logger = LoggerFactory.getLogger(MyMqConfig.class);


    public static final String QWZ_EXCHANGE_NAME = "qwz_exchange";
    public static final String QWZ_QUEUE_NAME = "qwz_queue";

    @Bean
    Queue queue() {
        return new Queue(QWZ_QUEUE_NAME, true, false, false);
    }

    @Bean
    DirectExchange directExchange() {
        return new DirectExchange(QWZ_EXCHANGE_NAME, true, false);
    }

    @Bean
    Binding binding() {
        return new Binding(QWZ_QUEUE_NAME,
                Binding.DestinationType.QUEUE,
                QWZ_EXCHANGE_NAME,
                "qwz.send.msg",
                null);
    }

}
```

## 二、消息发送确认

```java
/**
 * @author qinweizhao
 * @since 2022/6/30
 */
@Configuration
public class RabbitConfig implements RabbitTemplate.ConfirmCallback, RabbitTemplate.ReturnsCallback {

    private static final Logger logger = LoggerFactory.getLogger(RabbitConfig.class);

    @Resource
    RabbitTemplate rabbitTemplate;

    @PostConstruct
    public void initRabbitTemplate() {
        rabbitTemplate.setConfirmCallback(this);
        rabbitTemplate.setReturnsCallback(this);
    }

    /**
     * Publisher -> Exchange
     *
     * @param correlationData correlationData
     * @param ack             ack
     * @param cause           cause
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {

        if (ack) {
            logger.info("{}:消息成功到达交换器", correlationData.getId());
        } else {
            logger.error("{}:消息发送失败", correlationData.getId());
        }

    }

    /**
     * Exchange -xxx- Queue
     *
     * @param returned returned
     */
    @Override
    public void returnedMessage(ReturnedMessage returned) {
        logger.error("{}:消息未成功路由到队列", returned.getMessage().getMessageProperties().getMessageId());
    }
}
```

说明：上面方式为全局配置，也可以在发送是发送消息时单独配置。

## 三、消息接收确认

```java
/**
 * @author qinweizhao
 * @since 2022/6/30
 */
@Service
@RabbitListener(queues = MyMqConfig.QWZ_QUEUE_NAME)
public class ConsumerListener {


    private static final Logger logger = LoggerFactory.getLogger(ConsumerListener.class);


    @RabbitHandler
    public void listener(String msg, Channel channel, Message message) {
        logger.info(msg);

        try {
            // int i = 1/0;
            // 确认收到消息
            // 肯定确认；broker将移除此消息
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
            System.out.println("消费者确认收到消息：" + msg);
        } catch (Exception e) {
            try {
                // 拒绝消息
                // 否定确认；可以指定broker是否丢弃此消息，可以批量
                channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, false);
                // 同上，但不能批量
                // channel.basicReject(message.getMessageProperties().getDeliveryTag(),true);
                System.out.println("消费者拒绝消息：" + msg);
            } catch (IOException ioException) {
                ioException.printStackTrace();
            }
        }

    }

}
```

注意：**默认为自动 ack**，消息被消费者收到，就会从 broker 的 queue 中移除。如果 queue 无消费者，消息依然会被存储，直到消费者消费。

## 附

>代码地址：
>
>https://github.com/qinweizhao/qwz-integration/tree/master/spring-boot-rabbitmq

