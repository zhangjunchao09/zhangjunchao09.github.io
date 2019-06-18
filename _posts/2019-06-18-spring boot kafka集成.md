---
layout:     post
title:      kafka
subtitle:   spring boot 集成kafka
date:       2019-06-18
author:     ZJC
header-img: img/post-bg-debug.png
catalog: true
tags:
    - kafka
    - spring boot
---

# pom

```
 <dependency>
     <groupId>org.springframework.kafka</groupId>
     <artifactId>spring-kafka</artifactId>
     <version>1.3.1.RELEASE</version>
 </dependency>
```
# yml配置

```
spring:
  kafka:
    # 指定kafka 代理地址，可以多个
    bootstrap-servers: 192.168.1.144:9092
    producer:
      retries: 0
      batch-size: 16384
      buffer-memory: 33554432
      # 指定消息key和消息体的编解码方式
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    consumer:
      group-id: test-consumer-group
      auto-offset-reset: earliest
      enable-auto-commit: true
      auto-commit-interval: 100
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
```

# 生产者

```
@Component
public class KafkaProducer {

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    public void send(String topic, String msg) {
        kafkaTemplate.send(topic, msg);
    }

}
```

# 消费者

## push消费者
@KafkaListener 注解实现
```
@Component
public class KafkaConsumer {

    @KafkaListener(topics = {"test1"})
    public void listen(String content) {


        System.out.println("================:" + content);

    }
}
```
## pull消费者

```
@Component
public class KafkaCustomConsumer {

    @Autowired
    ConsumerFactory consumerFactory;

    public void subscribe(String topic) {
        List topics = new ArrayList<>();
        topics.add(topic);
        Consumer consumer = consumerFactory.createConsumer();
        consumer.subscribe(Collections.singletonList(topic));
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(100);
            for (ConsumerRecord<String, String> record : records) {
                System.out.printf("offset = %d, key = %s, value = %s", record.offset(), record.key(), record.value());
            }
        }
    }
}
```


