---
title: The Spring Qualifier Annotation
date: 2022-11-05 16:58:00 +0900
categories: [SlipBox, Spring]
tags: [Annotation, Qualifier]
publish: false
---

# AutoWire Need for Disambiguation
- The @Autowired annotation alone isn't enough for Spring to understand which bean to inject.
- By default, Spring resolves autowired entries by type.
- If more than one bean of the same type is available in the container, the framework will throw NoUniqueBeanDefinitionException
- two possible candidates exist for Spring to inject as bean collaborators:

```java
@Component("fooFormatter")
public class FooFormatter implements Formatter {
 
    public String format() {
        return "foo";
    }
}

@Component("barFormatter")
public class BarFormatter implements Formatter {
 
    public String format() {
        return "bar";
    }
}

@Component
public class FooService {
     
    @Autowired
    private Formatter formatter;
}
```

- If we try to load FooService into our context, the Spring framework will throw a NoUniqueBeanDefinitionException. 

# @Qualifier Annotation
-  solve the problem by including the @Qualifier annotation to indicate which bean we want to use:

```java
public class FooService {
     
    @Autowired
    @Qualifier("fooFormatter")
    private Formatter formatter;
}
```

- the qualifier name to be used is the one declared in the @Component annotation.

-  could have also used the @Qualifier annotation on the Formatter implementing classes, instead of specifying the names in their @Component annotations, to obtain the same effect:

```java
@Component
@Qualifier("fooFormatter")
public class FooFormatter implements Formatter {
    //...
}

@Component
@Qualifier("barFormatter")
public class BarFormatter implements Formatter {
    //...
}
```

# @Qualifier vs @Primary
- The @Primary annotation is useful when we want to specify which bean of a certain type should be injected by default.
- If we require the other bean at some injection point, we would need to specifically indicate it. We can do that via the @Qualifier annotation.
- if both the @Qualifier and @Primary annotations are present, then the @Qualifier annotation will have precedence.

```java
@Configuration
public class Config {
 
    @Bean
    public Employee johnEmployee() {
        return new Employee("John");
    }
 
    @Bean
    @Primary
    public Employee tonyEmployee() {
        return new Employee("Tony");
    }
}
```

# Reference
- https://www.baeldung.com/spring-qualifier-annotation