---
title: "Spring Data Persitable Domain"
date: 2023-07-30 19:00:00 +0900
categories: [Level, Junior]
tags: [Spring, Data]
---

The Persistable interface in Spring Framework's data.domain package is a simple interface for entities. It provides two methods:

1. getId(): Returns the identifier of the entity.
2. isNew(): Returns whether the entity is new or not.


These methods are automatically made available as property accessors when the Persistable interface is implemented in combination with the @AccessType(PROPERTY) annotation. Either of these methods can be marked as transient when annotated with @Transient.

The Persistable interface is used by Spring Data to detect the state of an entity. An entity is considered new if its identifier is null. Otherwise, the entity is considered to be persisted.

Here is an example of how to implement the Persistable interface in Java:

```java
@Entity
public class User implements Persistable<Long> {

    @Id
    private Long id;

    @Transient
    private boolean isNew;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public boolean isNew() {
        return isNew;
    }

    public void setNew(boolean isNew) {
        this.isNew = isNew;
    }
}
```

n this example, the User class implements the Persistable interface and provides implementations for the getId() and isNew() methods. The getId() method returns the identifier of the user, which is a Long. The isNew() method returns whether the user is new or not.

The Persistable interface is a simple but useful interface that can be used to simplify the development of entities in Spring Data applications.
