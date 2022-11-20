---
title: Kafka and Spring
date: 2022-11-21 20:00:00 +0900
categories: [SlipBox, Spring]
tags: [Test, Kafka]
---

# Intro to Apache Kafka with Spring

## 1. Overview
- Spring Kafka brings the simple and typical Spring template programming model with a KafkaTemplate and Message-driven POJOs via @KafkaListener annotation.

## 2. Configuring Topics
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

## 3. Producing Messages

- To create messages, we first need to configure a `ProducerFactory`. This sets the strategy for creating Kafka `Producer` instances.
- Then we need a `KafkaTemplate`, which wraps a Producer instance and provides convenience methods for sending messages to Kafka topics.
- `Producer` instances are thread safe. So, using a single instance throughout an application context will give higher performance.

### 3.1. Producer Configuration:

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

### 3.2. Publishing Messages:

- Send messages using the KafkaTemplate class:
- The send API returns a ListenableFuture object. If we want to block the sending thread and get the result about the sent message, we can call the get API of the ListenableFuture object. The thread will wait for the result, but it will slow down the producer.

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
## 4. Consuming Messages
### 4.1 Consumer Configuration

- Need to configure a `ConsumerFactory` and a `KafkaListenerContainerFactory`. 
- Once these beans are available in the Spring bean factory, POJO-based consumers can be configured using `@KafkaListener` annotation.
- `@EnableKafka` annotation is required on the configuration class to enable the detection of `@KafkaListener` annotation on spring-managed beans:

```java
@EnableKafka
@Configuration
public class KafkaConsumerConfig {

    @Bean
    public ConsumerFactory<String, String> consumerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(
          ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, 
          bootstrapAddress);
        props.put(
          ConsumerConfig.GROUP_ID_CONFIG, 
          groupId);
        props.put(
          ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, 
          StringDeserializer.class);
        props.put(
          ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, 
          StringDeserializer.class);
        return new DefaultKafkaConsumerFactory<>(props);
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, String> 
      kafkaListenerContainerFactory() {
   
        ConcurrentKafkaListenerContainerFactory<String, String> factory =
          new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }
}
```

### 4.2. Consuming Messages

```java
@KafkaListener(topics = "topicName", groupId = "foo")
public void listenGroupFoo(String message) {
    System.out.println("Received Message in group foo: " + message);
}
```

- We can implement multiple listeners for a topic, each with a different group Id. Furthermore, one consumer can listen for messages from various topics:

```java
@KafkaListener(topics = "topic1, topic2", groupId = "foo")
```

- Spring also supports retrieval of one or more message headers using the @Header annotation in the listener:

```java
@KafkaListener(topics = "topicName")
public void listenWithHeaders(
  @Payload String message, 
  @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition) {
      System.out.println(
        "Received Message: " + message"
        + "from partition: " + partition);
}
```

### 4.3. Consuming Messages from a Specific Partition

- For a topic with multiple partitions, however, a @KafkaListener can explicitly subscribe to a particular partition of a topic with an initial offset:

```java
@KafkaListener(
  topicPartitions = @TopicPartition(topic = "topicName",
  partitionOffsets = {
    @PartitionOffset(partition = "0", initialOffset = "0"), 
    @PartitionOffset(partition = "3", initialOffset = "0")}),
  containerFactory = "partitionsKafkaListenerContainerFactory")
public void listenToPartition(
  @Payload String message, 
  @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition) {
      System.out.println(
        "Received Message: " + message"
        + "from partition: " + partition);
}
```

- Since the initialOffset has been set to 0 in this listener, all the previously consumed messages from partitions 0 and 3 will be re-consumed every time this listener is initialized.
- If we don't need to set the offset, we can use the partitions property of @TopicPartition annotation to set only the partitions without the offset:

```java
@KafkaListener(topicPartitions 
  = @TopicPartition(topic = "topicName", partitions = { "0", "1" }))
```

### 4.4. Adding Message Filter for Listeners

- configure listeners to consume specific message content by adding a custom filter. This can be done by setting a RecordFilterStrategy to the KafkaListenerContainerFactory:

```java
@Bean
public ConcurrentKafkaListenerContainerFactory<String, String>
  filterKafkaListenerContainerFactory() {

    ConcurrentKafkaListenerContainerFactory<String, String> factory =
      new ConcurrentKafkaListenerContainerFactory<>();
    factory.setConsumerFactory(consumerFactory());
    factory.setRecordFilterStrategy(
      record -> record.value().contains("World"));
    return factory;
}
```
- We can then configure a listener to use this container factory:

```java
@KafkaListener(
  topics = "topicName", 
  containerFactory = "filterKafkaListenerContainerFactory")
public void listenWithFilter(String message) {
    System.out.println("Received Message in filtered listener: " + message);
}
```

- In this listener, all the messages matching the filter will be discarded.

WIP

# Reference
- https://www.baeldung.com/spring-kafka
- https://www.baeldung.com/spring-boot-kafka-testing