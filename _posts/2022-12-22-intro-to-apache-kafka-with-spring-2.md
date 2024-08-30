---
title: Intro to Apache Kafka with Spring 2. Consuming Messages, Custom Message Converters, Multi-Method Listeners, 
date: 2022-12-22 20:00:00 +0900
categories: [SlipBox, Spring]
tags: [Kafka, Consume]
publish: false
---

# 4. Consuming Messages
## 4.1 Consumer Configuration

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

## 5. Custom Message Converters

- We can also send and receive custom Java objects. 
- This requires configuring the appropriate serializer in `ProducerFactory` and a deserializer in `ConsumerFactory`.
- A simple bean class, which we will send as messages:

```java
public class Greeting {

    private String msg;
    private String name;

    // standard getters, setters and constructor
}
```

### 5.1. Producing Custom Messages

- Use `JsonSerializer`.
- The code for ProducerFactory and KafkaTemplate
- use new KafkaTemplate to send the Greeting message

```java
@Bean
public ProducerFactory<String, Greeting> greetingProducerFactory() {
    // ...
    configProps.put(
      ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, 
      JsonSerializer.class);
    return new DefaultKafkaProducerFactory<>(configProps);
}

@Bean
public KafkaTemplate<String, Greeting> greetingKafkaTemplate() {
    return new KafkaTemplate<>(greetingProducerFactory());
}

kafkaTemplate.send(topicName, new Greeting("Hello", "World"));
```

### 5.2. Consuming Custom Messages

- modify the ConsumerFactory and KafkaListenerContainerFactory to deserialize the Greeting message correctly:

```java
@Bean
public ConsumerFactory<String, Greeting> greetingConsumerFactory() {
  // ...
    return new DefaultKafakConsumerFactory<>(
        props,
        new StringDeserializer(),
        new JsonDeserializer<>(Greeting.class);
    )
}

@Bean
public ConcurrentKafkaListenerContainerFactory<String, Greeting> greetingKafkaListenerContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<String, Greeting> factory = new ConcurrentKafkaListenerContainerFactory<>();
    facotry.serConsumerFactory(greetingConsumerFactory());
    return factory;
}
```

- The spring-kafka JSON serializer and deserializer use the Jackson library, which is also an optional Maven dependency for the spring-kafka project.

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.7</version>
</dependency>
```
- Finally, we need to write a listener to consume Greeting messages:

```java
@KafkaListener(
  topics = "topicName", 
  containerFactory = "greetingKafkaListenerContainerFactory")
public void greetingListener(Greeting greeting) {
    // process greeting message
}
```

## 6. Multi-Method Listeners

- configure our application to send vaious kinds of objects to the same topic and then consume them.
- add a new class `Farewell`:
- need some extra configuration to be able to send both `Greeting` adn `Farewell` objects to the same topic.

```java
public class Farewell {
    private String message;
    private Integer remainingMinutes;
}
```

### 6.1 Set Mapping Types in the Producer

- In the producer, we have to configure the JSON type mapping:

```java
configProps.put(JsonSerializer.TYPE_MAPPINGS, "greeting:com.baeldung.spring.kafka.Greeting, farewell:com.baeldung.spring.kafka.Farewell");
```

- This way, the library will fill in the type header with the corresponding class name.
- As a result, the ProducerFactory and KafkaTemplate look like this:

```java
@Bean
public ProducerFactory<String, Object> multiTypeProducerFactory() {
    Map<String, Object> configProps = new HashMap<>();
    configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapAddress);
    configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
    configProps.put(JsonSerializer.TYPE_MAPPINGS, 
      "greeting:com.baeldung.spring.kafka.Greeting, farewell:com.baeldung.spring.kafka.Farewell");
    return new DefaultKafkaProducerFactory<>(configProps);
}

@Bean
public KafkaTemplate<String, Object> multiTypeKafkaTemplate() {
    return new KafkaTemplate<>(multiTypeProducerFactory());
}
```

- can use this KafkaTemplate to send a Greeting, Farewell, or any Object to the topic:

```java
multiTypeKafkaTemplate.send(multiTypeTopicName, new Greeting("Greetings", "World!"));
multiTypeKafkaTemplate.send(multiTypeTopicName, new Farewell("Farewell", 25));
multiTypeKafkaTemplate.send(multiTypeTopicName, "Simple string message");
```

### 6.2 Use a Custom `MessageConverter` in the Consumer

- To be able to deserialize the incoming message, we'll need to provide our Consumer with a custom MessageConverter.
- need to tell `Jackson2JavaTypeMapper` explicitly to use the type header to determine the target class for deserialization:
- 

```java
typeMapper.setTypePrecedence(Jackson2JavaTypeMapper.TypePrecedence.TYPE_ID);
```

- also need to provide the reverse mapping information. 
  - Finding “greeting” in the type header identifies a Greeting object, 
  - whereas “farewell” corresponds to a Farewell object:

```java
Map<String, Class<?>> mappings = new HashMap<>(); 
mappings.put("greeting", Greeting.class);
mappings.put("farewell", Farewell.class);
typeMapper.setIdClassMapping(mappings);
```

- have to make sure that mapper contains the location of the target classes:

```java
typeMapper.addTrustedPackages("com.baeldung.spring.kafka");
```

MessageConverter:

```java
@Bean
public RecordMessageConverter multiTypeConverter() {
    StringJsonMessageConverter converter = new StringJsonMessageConverter();
    DefaultJackson2JavaTypeMapper typeMapper = new DefaultJackson2JavaTypeMapper();
    typeMapper.setTypePrecedence(Jackson2JavaTypeMapper.TypePrecedence.TYPE_ID);
    typeMapper.addTrustedPackages("com.baeldung.spring.kafka");
    Map<String, Class<?>> mappings = new HashMap<>();
    mappings.put("greeting", Greeting.class);
    mappings.put("farewell", Farewell.class);
    typeMapper.setIdClassMapping(mappings);
    converter.setTypeMapper(typeMapper);
    return converter;
}
```

need to tell our `ConcurrentKafkaListenerContainerFactory` to use the `MessageConverter` and a rather basic `ConsumerFactory`:

```java
@Bean
public ConsumerFactory<String, Object> multiTypeConsumerFactory() {
    HashMap<String, Object> props = new HashMap<>();
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapAddress);
    props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
    props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
    return new DefaultKafkaConsumerFactory<>(props);
}

@Bean
public ConcurrentKafkaListenerContainerFactory<String, Object> multiTypeKafkaListenerContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<String, Object> factory = new ConcurrentKafkaListenerContainerFactory<>();
    factory.setConsumerFactory(multiTypeConsumerFactory());
    factory.setMessageConverter(multiTypeConverter());
    return factory;
}
```
### 6.3 Use `@KafkaHandler` in the Listener

- in our KafkaListener, we'll create a handler method to retrieve every possible object. 
- Each handler will need to be annotated with @KafkaHandler.
- we can also define a default handler for objects that can't be bound to one of the Greeting or Farewell classes:

```java
@Component
@KafkaListener(id = "multiGroup", topics = "multitype")
public class MultiTypeKafkaListener {

    @KafkaHandler
    public void handleGreeting(Greeting greeting) {
        System.out.println("Greeting received: " + greeting);
    }

    @KafkaHandler
    public void handleF(Farewell farewell) {
        System.out.println("Farewell received: " + farewell);
    }

    @KafkaHandler(isDefault = true)
    public void unknown(Object object) {
        System.out.println("Unkown type received: " + object);
    }
}
```

# Reference
- https://www.baeldung.com/spring-kafka
- https://www.baeldung.com/spring-boot-kafka-testing