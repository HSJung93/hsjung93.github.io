---
title: "Support Pattern"
date: 2023-07-30 19:00:00 +0900
categories: [Level, Junior]
tags: [Spring, Data]
---

In the Spring Framework's data access module, the `@Transient` annotation is used to mark a field or property of an entity class as non-persistent. This means that the field will not be mapped to any database column when using an ORM (Object-Relational Mapping) framework like Spring Data JPA. It's useful when you have certain fields in your entity class that you want to exclude from database storage or data persistence.

Let's say you have an entity class representing a `Person` with various properties, including some fields that you want to be stored in the database, such as `firstName`, `lastName`, and `age`. However, you also have a field like `transientField` that you want to exclude from database mapping. In this case, you can use the `@Transient` annotation on the `transientField` to indicate that it should not be persisted in the database.

Below is a Java code example demonstrating the usage of the `@Transient` annotation in a Spring Data JPA entity class:

```java
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Transient;

@Entity
public class Person {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String firstName;
    private String lastName;
    private int age;

    @Transient
    private String transientField;

    // Constructors, getters, setters, etc.
}

```

In the example above, the transientField is annotated with @Transient, indicating that it should not be persisted in the database when using Spring Data JPA.

When Spring Data JPA encounters this entity, it will create a table with columns for id, firstName, lastName, and age, but it will not include the transientField in the table schema.

Remember that the @Transient annotation is specific to the JPA (Java Persistence API) and works with ORM frameworks like Hibernate, which is commonly used in Spring Data JPA. If you're using different persistence technologies, the behavior of this annotation might vary.