---
title: 스프링 클라우드 게이트웨이 입문
date: 2023-04-25 20:00:00 +0900
categories: [강의, 주니어]
tags: [스프링, 클라우드]
---

# 주요 개념

## `Route`
- 게이트웨이의 기본 단위. 
- ID, 목적지 URI, predicate의 collection, 필터의 collection 으로 정의
- 하나의 Route는 aggregate predicate가 참이면 매칭된다.

## `Predicate`
- header와 parameter 등 HTTP 요청에 매칭되는 Java 8 Function Predicate.
- 인풋 타입은 스프링의 `ServerWebExchange` 이다.

## `Filter`
- `GatewayFilter`의 인스턴스.
- 요청을 보내기 전후에 요청과 응답을 수정

![](/assets/img/spring-gateway.png)

> 스프링 게이트웨이
{: .prompt-tip }

클라이언트에서 게이트웨이에 요청을 보내면 Gateway Handler Mapping에서 요청에 맞는 route를 찾아서 Gateway Web Handler에 보낸다. 이 핸들러가 요청을 필터 체인애 보내고, 필터 체인은 프록시 요청을 주고(pre-filter) 받을(post-filter) 때 모두 로직을 실행시킬 수 있다. 

# 스프링 profile 설정
```
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - Cookie=mycookie,mycookievalue
```
- `Cookie`라는 route predicate factory 사용
  - name: `mycookie`
  - value: `mycookievalue`

## Route Predicate 팩토리
- HTTP 요청의 속성들로 매칭
- 다양한 route predicate factory 존재

### The After Route Predicate Factory
```
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - After=2017-01-20T17:42:47.789-07:00[America/Denver]
```
- `datetime` 이라는 파라미터 받음
- 특정 시간(Jan 20, 2017 17:42 Mountain Time (Denver)) 뒤에 일어난 요청과 매칭


### The Before Route Predicate Factory
```
spring:
  cloud:
    gateway:
      routes:
      - id: before_route
        uri: https://example.org
        predicates:
        - Before=2017-01-20T17:42:47.789-07:00[America/Denver]
```
- 특정 시간 전

### The Between Route Predicate Factory
```
spring:
  cloud:
    gateway:
      routes:
      - id: between_route
        uri: https://example.org
        predicates:
        - Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
```
- 파라미터로 주어진 두 시간 사이

### The Cookie Route Predicate Factory
```
spring:
  cloud:
    gateway:
      routes:
      - id: cookie_route
        uri: https://example.org
        predicates:
        - Cookie=chocolate, ch.p
```
- 쿠키 이름: `chocolate`
- 쿠키 값: `ch.p`

### The Header Route Predicate Factory
```
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: https://example.org
        predicates:
        - Header=X-Request-Id, \d+
```
- header 이름: `X-Request-Id`
- 정규표현식: `\d+`

## GatewayFileter Factory
- Route filterssms HTTP 요청과 응답을 수정
- 특정한 route에 해당

### The AddRequestHeader GatewayFilter Factory
```
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: https://example.org
        filters:
        - AddRequestHeader=X-Request-red, blue
```
- `X-Request-red:blue` 라는 헤더를 매칭된 요청의 헤더에 추가한다.
- 다음과 같이 Runtime에 URI 변수들을 사용할 수도 있다.

```
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment}
        filters:
        - AddRequestHeader=X-Request-Red, Blue-{segment}
```

### The AddRequestHeadersIfNotPresent GatewayFilter Factory
```
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment}
        filters:
        - AddRequestHeadersIfNotPresent=X-Request-Red:Blue-{segment}
```

- `X-Request-Red`라는 헤더가 있으면 원래 값을, 없으면 헤더를 더해서 보낸다.

### The AddRequestParameter GatewayFilter Factory
```
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_parameter_route
        uri: https://example.org
        predicates:
        - Host: {segment}.myhost.org
        filters:
        - AddRequestParameter=foo, bar-{segment}
```
- `red=blue` 라는 요청 쿼리를 스트링을 더한다.

### Default Filters
```
spring:
  cloud:
    gateway:
      default-filters:
      - AddResponseHeader=X-Response-Default-Red, Default-Blue
      - PrefixPath=/httpbin
```
- 모든 라우트에 필터를 적용하고 싶을 때 `default-filters`를 사용할 수 있다.
- 필터들의 리스트를 등록한다.

## Global Filiters
- `GatewayFilter` 와 같은 signature를 가지고 있다. 
- 모든 라우트에 적용하는 특수한 필터

### Combined Global Filter and GatewayFilter Ordering
```java
@Bean
public GlobalFilter customFilter() {
    return new CustomGlobalFilter();
}

public class CustomGlobalFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("custom global filter");
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return -1;
    }
}
```

- 요청을 라우터에 매칭할 때, filtering web handler가 `GlobalFilter`의 모든 인스턴스와 `GatewayFilter`의 라우터-특정된 인스턴스들을 필터 체인에 등록한다.
- 이 합쳐진 필터 체인은 `org.springframework.core.Ordered` 인터페이스로 정렬되는데 `getOrder()` 메서드로 구현 가능하다.
- pre 단계의 첫번째 필터와, post 단계의 마지막 필터가 우선 순위가 높다.

## Http timeout 설정

### Global timeouts
```
spring:
  cloud:
    gateway:
      httpclient:
        connect-timeout: 1000
        response-timeout: 5s
```
- `connect-timeout`은 milliseconds로
- `response-timeout`은 java.time.Duration 으로 적어주어야 한다.

### route 별 timeout 설정
```
      - id: per_route_timeouts
        uri: https://example.org
        predicates:
          - name: Path
            args:
              pattern: /delay/{timeout}
        metadata:
          response-timeout: 200
          connect-timeout: 200
```
- `connect-timeout`와 response-timeout` 모두 millisecond로 적어준다.
- java 코드로도 설정해줄 수 있다.
```java
import static org.springframework.cloud.gateway.support.RouteMetadataUtils.CONNECT_TIMEOUT_ATTR;
import static org.springframework.cloud.gateway.support.RouteMetadataUtils.RESPONSE_TIMEOUT_ATTR;

      @Bean
      public RouteLocator customRouteLocator(RouteLocatorBuilder routeBuilder){
         return routeBuilder.routes()
               .route("test1", r -> {
                  return r.host("*.somehost.org").and().path("/somepath")
                        .filters(f -> f.addRequestHeader("header1", "header-value-1"))
                        .uri("http://someuri")
                        .metadata(RESPONSE_TIMEOUT_ATTR, 200)
                        .metadata(CONNECT_TIMEOUT_ATTR, 200);
               })
               .build();
      }
```

# Reference
- https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/