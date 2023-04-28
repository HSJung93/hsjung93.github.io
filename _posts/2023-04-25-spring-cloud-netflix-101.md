---
title: 스프링 클라우드 넷플릭스 입문
date: 2023-04-26 20:00:00 +0900
categories: [강의, 주니어]
tags: [스프링, 클라우드]
---

- 서비스 디스커버리: `Eureka`
- 서킷 브레이커: `Hystrix`
- 라우팅: `Zuul`
- 클라이언트 사이드 로드 밸런싱: `Ribbon`
- 유레카는 넷플릭스 서비스 디스커버리 서버와 클라이언트로 구성

# 유레카 클라이언트
- 스타터:
  - group ID : `org.springframework.cloud`
  - artifact ID: `spring-cloud-starter-netflix-eureka-client`
- 클라이언트가 Eureka를 등록하면 호스트, 포트, 헬스 체크 URL, 홈페이지 등의 메타 데이터를 제공
- Eureka는 각각의 인스턴스로부터 heartbeat message 들을 받고, 일정 시간 이상 실패하면 registry로부터 인스턴스가 제거된다.
- `spring-cloud-starter-netflix-eureka-client` 가 classpath에 있을시 
  - 앱을 유레카 인스턴스와 클라이언트로 등록
    - 유레카 인스턴스: 자기 자신을 등록
    - 클라이언트: 다른 서비스의 위치를 담은 레지스트리를 쿼리할 수 있음.
  - 자동으로 어플리케이션이 Eureka 서버에 등록되는데 다음과 같은 설정이 필요
```
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```
- `defaultZone`은 클라이언트의 서비스 URL을 제공하는 default fallback 값이다.
- 그 외 기본값:
  - 어플리케이션 명칭(service ID): `${spring.application.name}`
  - 가상 호스트: `${spring.application.name}`
  - 포트: `${server.port}`
- Eureka Discovery Client 해제 방법
  - `eureka.client.enabled`를 false 로 설정하거나
  - `spring.cloud.discovery.enabled`를 false로 설정
- 보다 자세한 설정은 다음 참조:
  - https://github.com/spring-cloud/spring-cloud-netflix/blob/main/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaInstanceConfigBean.java
  - https://github.com/spring-cloud/spring-cloud-netflix/blob/main/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaClientConfigBean.java

## Authenticating
- `eureka.client.serviceUrl.defaultZone` 에 curl style로 보안 정보 넣어서 인증
  - 예: `user:password@localhost:8761/eureka`
  - `DiscoveryClientOptionalArgs` 타입의 빈을 생성하고 `ClientFilter` 인스턴스에 주입하여 커스텀 가능

## 상태 페이지와 헬스 표시
- 기본 path
  - 상태 페이지: `/info`
  - 헬스 표기: `/health`
- 스프링 프로파일로 직접 설정
```
eureka:
  instance:
    statusPageUrlPath: ${server.servletPath}/info
    healthCheckUrlPath: ${server.servletPath}/health
```

## 헬스 체크
- Eureka는 클라이언트의 heartbeat로 클라이언트가 작동함을 확인
- 기본 설정에서는 디스커버리 클라이언트가 자신의 헬스 체크 상태를 전파하지는 않아서 등록 이후에는 정상 상태로 표시
- `eureka.client.healthcheck.enabled: true`로 설정하면 헬스 체크 상태를 전파한다.

# 유레카 서버
- 스타터:
  - group ID : `org.springframework.cloud`
  - artifact ID: ` spring-cloud-starter-netflix-eureka-server`
- 스프링 프로파일 설정
```
spring:
  freemarker:
    template-loader-path: classpath:/templates/
    prefer-file-system-access: false
```
- 다음과 같이 어플리케이션에 `@EnableEurekaServer` 어노테이션을 추가
```
@SpringBootApplication
@EnableEurekaServer
public class Application {

    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class).web(true).run(args);
    }

}
```
- 유레카 서버에게 또다른 백엔드는 없고 메모리 상에서 처리한다.
- 서비스 인스턴스가 보내주는 heartbeats는 레지스트리에 등록된다.
- 기본적으로 유레카 서버는 유레카 클라이언트며 다른 서버를 위치시킬 service URL이 필요하다.

## 스탠드 얼론 구성
- 서버와 클라이언트의 두 캐시를 합쳐 standalone 유레카 서버를 구성할 수 있다.
- standalone에서 클라이언트 설정을 off 하는 방법:
```
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

## 다중 인스턴스 구성
- 다중 인스턴스가 서로를 등록하도록 하여 resilency를 높힐 수 있다.
- 서로 다른 스프링 프로파일에 설정한 호스트 네임에 맞춰서 serviceUrl을 등록한다. 

```
eureka:
  client:
    serviceUrl:
      defaultZone: https://peer1/eureka/,http://peer2/eureka/,http://peer3/eureka/

---
spring:
  profiles: peer1
eureka:
  instance:
    hostname: peer1

---
spring:
  profiles: peer2
eureka:
  instance:
    hostname: peer2

---
spring:
  profiles: peer3
eureka:
  instance:
    hostname: peer3
```

# Reference
- https://cloud.spring.io/spring-cloud-netflix/reference/html/
- https://github.com/spring-cloud/spring-cloud-netflix/blob/main/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaInstanceConfigBean.java
- https://github.com/spring-cloud/spring-cloud-netflix/blob/main/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaClientConfigBean.java
