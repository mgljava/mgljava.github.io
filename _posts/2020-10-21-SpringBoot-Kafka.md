---
layout:     post
title:      SpringBoot集成Kafka
date:       2020-10-21
author:     Monk
header-img: img/404.png
catalog: true
tags:
    - SpringBoot
    - Kafka
---
### SpringBoot集成Kafka

##### 1. 引入依赖包

```groovy
implementation 'org.springframework.kafka:spring-kafka'
implementation 'com.google.code.gson:gson'
```

##### 2. 编写消息实体

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class KafkaMessage {

  private long id;
  private String username;
  private String password;
  private LocalDateTime time;
}
```

##### 3. 编写Produer

```java
@Component
@RequiredArgsConstructor
public class KafkaProducer {

  @Value("${kafka.topic.test}")
  private String topic;
  private final KafkaTemplate<String, String> kafkaTemplate;

  public void sendMessage(KafkaMessage message) {
    kafkaTemplate.send(topic, new Gson().toJson(message));
  }
}
```

##### 4. 编写Consumer

``````java
@Component
public class KafkaConsumer {

  @KafkaListener(topics = "myJavaTopic", groupId = "myGroup")
  public void consumerMessage(ConsumerRecord<String, String> record) {
    System.out.println("consumerMessage method invoked!");
    System.out.println("topic: " + record.topic());
    System.out.println("key: " + record.key());
    System.out.println("value: " + record.value());
    System.out.println("partition: " + record.partition());
    System.out.println("timestamp: " + record.timestamp());
  }
}
``````

##### 5. 配置YML文件

```yaml
spring:  
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    consumer:
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
kafka:
  topic:
    test: myJavaTopic
```

##### 6. 测试结果

```java
@RestController
@RequiredArgsConstructor
@RequestMapping(value = "/kafka")
public class KafkaMessageController {

  private final KafkaProducer kafkaProducer;

  @GetMapping(value = "/send")
  public KafkaMessage send(@RequestParam("id") long id, @RequestParam("username") String username, @RequestParam("password") String password) {
    KafkaMessage message = new KafkaMessage(id, username, password, LocalDateTime.now());
    kafkaProducer.sendMessage(message);
    return message;
  }
}
```