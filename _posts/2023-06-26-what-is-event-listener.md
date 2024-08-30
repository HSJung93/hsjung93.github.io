---
title: "What is EventListener in Spring"
date: 2023-06-26 22:00:00 +0900
categories: [Level, Junior]
tags: [Spring, Context]
publish: false
---

In Spring Framework, the @EventListener annotation is used to handle and respond to events that occur within an application. It allows you to define methods that will be executed when specific events are published in the Spring application context. These events can be either standard events provided by Spring or custom events that you create.

Here's how you can use the `@EventListener` annotation in Spring:

1. Define an event class: First, you need to define an event class that represents the event you want to handle. This class can be a simple POJO (Plain Old Java Object) and should typically extend the ApplicationEvent class or its subclasses. For example, let's create a custom event called UserRegisteredEvent:

```java
public class UserRegisteredEvent extends ApplicationEvent {

    private User user;

    public UserRegisteredEvent(User user) {
        super(user);
        this.user = user;
    }

    public User getUser() {
        return user;
    }
}

```

2. Publish the event: Somewhere in your application, you need to publish the event using the ApplicationEventPublisher. This can be done in a service, controller, or any other component. For example:

```java
@Service
public class UserService {

    @Autowired
    private ApplicationEventPublisher eventPublisher;

    public void registerUser(User user) {
        // Perform user registration logic
        // ...

        // Publish the UserRegisteredEvent
        UserRegisteredEvent event = new UserRegisteredEvent(user);
        eventPublisher.publishEvent(event);
    }
}

```

3. Create an event listener: To handle the event, you need to define a method annotated with @EventListener that will be executed when the event is published. This method can be placed in any Spring-managed component, such as a service or a bean. Here's an example of an event listener that handles the UserRegisteredEvent:

```java
@Component
public class UserRegisteredEventListener {

    @EventListener
    public void handleUserRegisteredEvent(UserRegisteredEvent event) {
        User user = event.getUser();
        // Perform actions upon user registration
        // ...
    }
}

```

In the above example, the `handleUserRegisteredEvent` method is annotated with `@EventListener` and takes the `UserRegisteredEvent` as a parameter. Whenever a `UserRegisteredEvent` is published, this method will be invoked and can perform specific actions based on the event data.

Note that you can have multiple event listeners for the same event, and all of them will be invoked when the event is published.

By using the `@EventListener` annotation, you can decouple components in your application and respond to events in a loosely coupled manner. It promotes the separation of concerns and allows you to handle application events without explicitly wiring and invoking event handlers.