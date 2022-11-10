---
title: Testing an Apache Kafka Integration within a Spring Boot Application and JUnit 5
date: 2022-11-10 20:00:00 +0900
categories: [SlipBox, Spring]
tags: [Test, Kafka]
---
# Project Setup
- When you select Spring for Apache Kafka at `start.spring.io` it automatically adds all necessary dependency entries into the maven or gradle file. 
- By now it comes with JUnit 5

```
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka-test</artifactId>
    <scope>test</scope>
</dependency>
```

# Class Configuration
- annotate the class with `@EmbeddedKafka`: inject the EmbeddedKafkaBroker to either your test method or in a setup method at the beginning.

```java
@EmbeddedKafka
public class SimpleKafkaTest {

    private EmbeddedKafkaBroker embeddedKafkaBroker;

    @BeforeEach
    void setUp(EmbeddedKafkaBroker embeddedKafkaBroker) {
        this.embeddedKafkaBroker = embeddedKafkaBroker;
    }

    // ...

}
```
- there is no Spring annotation in this class.
- Without annotating it with @ExtendWith(`SpringExtension.class`) or an extension which implies this (e.g. `@SpringBootTest`) the test is executed outside of the spring context

# Class Configuration with Spring Context
-  like to have the advantages of the Spring context, you need to add the @SpringBootTest annotation to the above test case.
- need to change the way how to autowire your `EmbeddedKafkaBroker`. 
- change the autowiring from the JUnit 5 way to the `@Autowired` annotation from Spring:

```java
@EmbeddedKafka
@ExtendWith(SpringExtension.class)
public class SimpleKafkaTest {

    @Autowired
    private EmbeddedKafkaBroker embeddedKafkaBroker;

    // ...

}
```

# Configure Kafka Consumer

```java
Map<String, Object> configs = new HashMap<>(KafkaTestUtils.consumerProps("consumer", "false", embeddedKafkaBroker));
DefaultKafkaConsumerFactory<String, String> consumerFactory = new DefaultKafkaConsumerFactory<>(
    configs, 
    new StringDeserializer(), 
    new StringDeserializer()
);
```

- KafkaTestUtils.consumerProps is providing you everything what you need to do the configuration. 
- The first parameter is the name of your consumer group, the second is a flag to set auto commit and the last parameter is the EmbeddedKafkaBroker instance.
- Afterwards, you can configure your consumer with the Spring wrapper DefaultKafkaConsumerFactory.

using a `KafkaMessageListenerContainer`:

```java
KafkaMessageListenerContainer<String, String> container = new KafkaMessageListenerContainer<>(consumerFactory, containerProperties);
BlockingQueue<ConsumerRecord<String, String>> records = new LinkedBlockingQueue<>();
container.setupMessageListener((MessageListener<String, String>) records::add);
container.start();
ContainerTestUtils.waitForAssignment(container, embeddedKafkaBroker.getPartitionsPerTopic());
```

- This container has a message listener and writing them as soon as they are received to a queue. 
- read the consumer records from the queue and the queue will block until we are receiving the first record
- By using the `ContainerTestUtil.waitForAssignment` we are waiting for the initial assignment, since we wait explicitly for it.
- We also need to stop() our container afterwards, to ensure that we have a clean context in a multi-test scenario. 
- complete setup could look like:


```java
@EmbeddedKafka
@ExtendWith(SpringExtension.class)
public class SimpleKafkaTest {

    private static final String TOPIC = "domain-events";

    @Autowired
    private EmbeddedKafkaBroker embeddedKafkaBroker;

    BlockingQueue<ConsumerRecord<String, String>> records;

    KafkaMessageListenerContainer<String, String> container;

    @BeforeEach
    void setUp() {
        Map<String, Object> configs = new HashMap<>(KafkaTestUtils.consumerProps("consumer", "false", embeddedKafkaBroker));
        DefaultKafkaConsumerFactory<String, String> consumerFactory = new DefaultKafkaConsumerFactory<>(configs, new StringDeserializer(), new StringDeserializer());
        ContainerProperties containerProperties = new ContainerProperties(TOPIC);
        container = new KafkaMessageListenerContainer<>(consumerFactory, containerProperties);
        records = new LinkedBlockingQueue<>();
        container.setupMessageListener((MessageListener<String, String>) records::add);
        container.start();
        ContainerTestUtils.waitForAssignment(container, embeddedKafkaBroker.getPartitionsPerTopic());
    }

    @AfterEach
    void tearDown() {
        container.stop();
    }

    // our tests...
}
```

# Configure Kafka Producer

```java
Map<String, Object> configs = new HashMap<>(KafkaTestUtils.producerProps(embeddedKafkaBroker));
Producer<String, String> producer = new DefaultKafkaProducerFactory<>(configs, new StringSerializer(), new StringSerializer()).createProducer();
```

# Produce and Consume Messages

now have a consumer and a producer, we are actually able to produce messages:

```java
producer.send(new ProducerRecord<>(TOPIC, "my-aggregate-id", "my-test-value"));
producer.flush();
```

consume messages and doing assertions on them:
```java
ConsumerRecord<String, String> singleRecord = records.poll(100, TimeUnit.MILLISECONDS);
assertThat(singleRecord).isNotNull();
assertThat(singleRecord.key()).isEqualTo("my-aggregate-id");
assertThat(singleRecord.value()).isEqualTo("my-test-value");
```

# Serialize and Deserialize Key and Value

- you can configure your serializers and de-serializers as you want. 
- In case you have inheritance and you have an abstract parent class or an interface your actual implementation might be in the test case.
- solve that by adding the specific package or all packages as trusted:

```java
JsonDeserializer<DomainEvent> domainEventJsonDeserializer = new JsonDeserializer<>(DomainEvent.class);
domainEventJsonDeserializer.addTrustedPackages("*");
```

# Improve Execution Performance ofr Multiple Tests

- with the annotation @TestInstance(TestInstance.Lifecycle.PER_CLASS). 
- We can convert our @BeforeEach and @AfterEach to @BeforeAll and @AfterAll. - The only thing that we need to ensure is, that each test in the class is consuming all messages

```java
@EmbeddedKafka
@ExtendWith(SpringExtension.class)
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
public class SimpleKafkaTest {

    private static final String TOPIC = "domain-events";

    @Autowired
    private EmbeddedKafkaBroker embeddedKafkaBroker;

    BlockingQueue<ConsumerRecord<String, String>> records;

    KafkaMessageListenerContainer<String, String> container;

    @BeforeAll
    void setUp() {
        Map<String, Object> configs = new HashMap<>(KafkaTestUtils.consumerProps("consumer", "false", embeddedKafkaBroker));
        DefaultKafkaConsumerFactory<String, String> consumerFactory = new DefaultKafkaConsumerFactory<>(configs, new StringDeserializer(), new StringDeserializer());
        ContainerProperties containerProperties = new ContainerProperties(TOPIC);
        container = new KafkaMessageListenerContainer<>(consumerFactory, containerProperties);
        records = new LinkedBlockingQueue<>();
        container.setupMessageListener((MessageListener<String, String>) records::add);
        container.start();
        ContainerTestUtils.waitForAssignment(container, embeddedKafkaBroker.getPartitionsPerTopic());
    }

    @AfterAll
    void tearDown() {
        container.stop();
    }

    // …
}
```

