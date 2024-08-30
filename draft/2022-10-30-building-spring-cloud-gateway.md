---
title: Building a Spring Cloud Gateway
date: 2022-10-30 05:09:00 +0900
categories: [SlipBox, Spring]
tags: [Cloud, Gateway]
publish: false
---
# Creating A Simple Route
- The Spring Cloud Gateway uses routes to process requests to downstream services
- create a new `Bean` of type `RouteLocator` in `Application.java`.

```java
@Bean
public RouteLocator myRoutes(RouteLocatorBuilder builder) {
  return builder.routes().build();
}
```

- `RouteLocatorBuilder` lets you add predicates and filters to your routes so that you can route handle based on certain conditions
- as well as alter the request/response
- create a route that routes a request to `https://httpbin.org/get` when request is made to the Gateway at `/get`.
- add filter that adds the `Hello` request header with a value of `World` to the request before it is routed:

```java
@Bean
public RouteLocator myRoutes(RouteLocatorBuilder builder) {
  return builder.routes()
    .route( p -> p
      .path("/get")
      .filter( f -> f
        .addRequesetHeader("Hello", "World"))
        .url("http://httpbin.org:80")
      ).build();
}
```
- make a request to `http://localhost:8080/get`

```shell
curl http://localhost:8080/get
```

- receive a response back that looks similar to the following output:

```json
{
  "args": {},
  "headers": {
    "Accept": "*/*",
    "Connection": "close",
    "Forwarded": "proto=http;host=\"localhost:8080\";for=\"0:0:0:0:0:0:0:1:56207\"",
    "Hello": "World", // added request header
    "Host": "httpbin.org",
    "User-Agent": "curl/7.54.0",
    "X-Forwarded-Host": "localhost:8080"
  },
  "origin": "0:0:0:0:0:0:0:1, 73.68.251.70",
  "url": "http://localhost:8080/get"
}
```

# Using Spring Cloud CircuitBreaker
- want to wrap the routes we create in circuit breakers.
- using the Resilience4j Spring Cloud CircuitBreaker implementation.

```gradle
implementation("org.springframework.cloud:spring-cloud-starter-circuitbreaker-reactor-resilience4j")
```

- use HTTPBin’s delay API, which waits a certain number of seconds before sending a response.

```java
@Bean
public RouteLocator myRoutes(RouteLocatorBuilder builder) {
  return builder.routes()
    .route(p -> p
      .path("/get")
      .filters(f -> f.addRequestHeader("Hello", "World"))
      .uri("http://httpbin.org:80")
    ).route(p -> p
      .host("*.circuitbreaker.com")
      .filters(f -> f.circuitBreaker(config -> config.setName("mycmd")))
      .uri("http://httpbin.org:80")
    ).build();
}
```

- use the host predicate instead of the path predicate.
- as long as the host is `circuitbreaker.com`, we route the request to HTTPBin and wrap that request in a circuit breaker.
- configure the circuit breaker filter by using a configuration object.
- make a request to `/delay/3`.
- include a `Host` header that has a host of `circuitbreaker.com`.

```shell
curl --dump-header - --header 'Host: www.circuitbreaker.com'
```

> Use `--dump-header` to see the response header. The `-` after `--dump-header` tells cURL to print the headers to stdout.
{: .prompt-tip }

- see the circuit breaker timed out while waiting for the response from HTTPBin.

```
HTTP/1.1 504 Gateway Timeout
content-length: 0
```

- optionally provide a fallback so that clients do not receive a `504`.
- return a response with the body `fallback` instead.

```java
@Bean
public RouteLocator myRoutes(RouteLocatorBuilder builder) {
  return builder.routes()
    .route(p -> p
      .path("/get")
      .filters(f -> f.addRequestHeader("Hello", "World"))
      .uri("http://httpbin.org:80")
    ).route(p -> p
      .host("*.circuitbreaker.com")
      .filters(f -> f
        .circuitBreaker(config -> config
          .setName("mycmd")
          .setFallbackUri("forward:/fallback")
        )
      ).uri("http://httpbin.org:80")
    ).build();
}
```

- the circuit breaker wrapped route times out calls `/fallback` in the Gateway application.
- Now we can add the `/fallback` endpoint to our application.


```java
@RequestMapping("/fallback")
public Mono<String> fallback() {
  return Mono.just("fallback");
}
```

- issue the following cURL command

```
curl --dump-header - --header 'Host: www.circuitbreaker.com'
```

- now see that we get a `200` back from the Gateway with the response body of `fallback`.

```
HTTP/1.1 200 OK
transfer-encoding: chunked
Content-Type: text/plain;charset=UTF-8
fallback
```
 
# Writing Tests
- make the URI in our routes configurable, so we can change the URI if we need to.
- create a new class called `UriConfiguration`:
 
```java
@ConfigurationProperties
class UriConfiguration {
  private String httpbin = "http://httpbin.org:80";
  
  public String getHttpbin() {
    return httpbin;
  }
  
  public void setHttpbin(String httpbin) {
    this.httpbin = httpbin;
  }
}
```
 
- To enable `ConfigurationProperties`, we need to also add a class-level annotation to `Application.java`.
 
```java
@EnableConfigurationProperties(UriConfiguration.class)
```
 
- use it in the `myRoutes` method:

```java
@Bean
public RouteLocator myRoutes(RouteLocatorBuilder builder, UriConfiguration uriConfiguration) {
  String httpUri = uriConfiguration.getHttpbin();
  return builder.routes()
    .route(p -> p
      .path("/get")
      .filters(f -> f.addRequestHeader("Hello", "World"))
      .uri(httpUri))
    .route(p -> p
      .host("*.circuitbreaker.com")
      .filters(f -> f
        .circuitBreaker(config -> config
          .setName("mycmd")
          .setFallbackUri("forward:/fallback")))
      .uri(httpUri))
    .build();
}
```

- create a new class called `ApplicationTest` in `src/main/test/java/gateway`

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT,
    properties = {"httpbin=http://localhost:${wiremock.server.port}"})
@AutoConfigureWireMock(port = 0)
public class ApplicationTest {

  @Autowired
  private WebTestClient webClient;

  @Test
  public void contextLoads() throws Exception {
    //Stubs
    stubFor(get(urlEqualTo("/get"))
        .willReturn(aResponse()
          .withBody("{\"headers\":{\"Hello\":\"World\"}}")
          .withHeader("Content-Type", "application/json")));
    stubFor(get(urlEqualTo("/delay/3"))
      .willReturn(aResponse()
        .withBody("no fallback")
        .withFixedDelay(3000)));

    webClient
      .get().uri("/get")
      .exchange()
      .expectStatus().isOk()
      .expectBody()
      .jsonPath("$.headers.Hello").isEqualTo("World");

    webClient
      .get().uri("/delay/3")
      .header("Host", "www.circuitbreaker.com")
      .exchange()
      .expectStatus().isOk()
      .expectBody()
      .consumeWith(
        response -> assertThat(response.getResponseBody()).isEqualTo("fallback".getBytes()));
  }
}
```

- WireMock from Spring Cloud Contract stand up a server that can mock the APIs from HTTPBin.
- `@AutoConfigureWireMock(port = 0)` annotation starts WireMock on a random port for us.
- setup “stubs” for the HTTPBin APIs we call through the Gateway and mock the behavior we expect.
- use `WebTestClient` to make requests to the Gateway and validate the responses.

# Reference
- https://spring.io/guides/gs/gateway/