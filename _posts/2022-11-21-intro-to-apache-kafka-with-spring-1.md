---
title: Intro to Apache Kafka with Spring 1. Configuring Topics, Producing Messages 
date: 2022-11-21 20:00:00 +0900
categories: [SlipBox, Spring]
tags: [Kafka, Publish]
publish: false
---

# 1. Overview
- Spring Kafka brings the simple and typical Spring template programming model with a KafkaTemplate and Message-driven POJOs via `@KafkaListener` annotation.

# 2. Configuring Topics
- Ran command-line tools to create topics in Kafka:

```bash
$ bin/kafka-topics.sh --create \
  --zookeeper localhost:2181 \
  --replication-factor 1 --partitions 1 \
  --topic mytopic
```

- Or add the KafkaAdmin Spring bean, which will automatically add topics for all beans of type NewTopic:

```java
@Configuration
public class KafkaTopicConfig {
    
    @Value(value = "${spring.kafka.bootstrap-servers}")
    private String bootstrapAddress;

    @Bean
    public KafkaAdmin kafkaAdmin() {
        Map<String, Object> configs = new HashMap<>();
        configs.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapAddress);
        return new KafkaAdmin(configs);
    }
    
    @Bean
    public NewTopic topic1() {
         return new NewTopic("baeldung", 1, (short) 1);
    }
}
```

# 3. Producing Messages

- To create messages, we first need to configure a `ProducerFactory`. This sets the strategy for creating Kafka `Producer` instances.
- Then we need a `KafkaTemplate`, which wraps a Producer instance and provides convenience methods for sending messages to Kafka topics.
- `Producer` instances are thread safe. So, using a single instance throughout an application context will give higher performance.

## 3.1. Producer Configuration:

```java
@Configuration
public class KafkaProducerConfig {

    @Bean
    public ProducerFactory<String, String> producerFactory() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(
          ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, 
          bootstrapAddress);
        configProps.put(
          ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, 
          StringSerializer.class);
        configProps.put(
          ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, 
          StringSerializer.class);
        return new DefaultKafkaProducerFactory<>(configProps);
    }

    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}
```

## 3.2. Publishing Messages:

- Send messages using the `KafkaTemplate` class:
- The send API returns a `ListenableFuture` object. If we want to block the sending thread and get the result about the sent message, we can call the get API of the `ListenableFuture` object. The thread will wait for the result, but it will slow down the producer.

```java
@Autowired
private KafkaTemplate<String, String> kafkaTemplate;

public void sendMessage(String msg) {
    kafkaTemplate.send(topicName, msg);
}
```

- Kafka is a fast-stream processing platform. Therefore, it's better to handle the results asynchronously so that the subsequent messages do not wait for the result of the previous message.
- We can do this through a callback:

```java
public void sendMessage(String message) {
            
    ListenableFuture<SendResult<String, String>> future = 
      kafkaTemplate.send(topicName, message);
	
    future.addCallback(new ListenableFutureCallback<SendResult<String, String>>() {

        @Override
        public void onSuccess(SendResult<String, String> result) {
            System.out.println("Sent message=[" + message + 
              "] with offset=[" + result.getRecordMetadata().offset() + "]");
        }
        @Override
        public void onFailure(Throwable ex) {
            System.out.println("Unable to send message=[" 
              + message + "] due to : " + ex.getMessage());
        }
    });
}
```

# Reference
- https://www.baeldung.com/spring-kafka