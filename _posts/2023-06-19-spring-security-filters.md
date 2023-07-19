---
title: "Spring Security Filters"
date: 2023-06-19 22:00:00 +0900
categories: [강의, 시니어]
tags: [Spring, Security]
---

## SecurityContextPersistenceFilter

SecurityContextPersistenceFilter 는 요청 전에 SecurityContextRepository에서 얻은 정보르 SecurityContextHolder를 채워둔다. 그 후, 요청이 SecurityContextHolder를 채우고 정리하면 SecurityContextRepository에 다시 저장한다. 기본값으로는 HttpSessionSecurityContextRepository를 사용한다. 설정 옵션은 HttpSession 클래스 정보에서 볼 수 있다.

이 필터는 특히 웹 로직 서블릿 컨테이너의 비호환성을 해결하기 위해서 요청 하나 당 한번만 실행된다. 인증 절차 전에 반드시 실행되어야 한다. BASIC, CAS 등 인증 절차는 SecurityContextHolder가 필요한 SecurityContext를 실행 시간에 가지고 있다고 기대하며 동작한다. 

## OAuth2AuthorizationRequestRedirectFilter

이 필터는 인가 코드 부여를 시작한다. 혹은 최종 사용자의 user-agent를 인가 서버의 인가 엔드포인트로 리다이렉트하여 암묵적으로 부여 절차를 시작한다. 인가 엔드포인트로의 리다이렉트 URI로 사용되는 OAuth 2.0 인가 요청을 만든다. 이 리다이렉트 URI는 클라이언트 식별자, 요청 범위, 상태, 응답 타입, 리다이렉트 URI를 포함한다. 인가 서버는 최종 사용자(리소스-소유자)에 의하여 user-agent에게 권한이 부여(혹은 거부)되면 이 리다이렉트 URI를 보낸다.

기본적으로 이 필터는 기본 OAuth2Auth2Auth2RequestResolver를 사용하여 /oauth2/authorization/{registrationId} URI에 대한 권한 부여 요청에 응답한다. URI 템플릿 변수 {registrationId}는 OAuth 2.0 권한 부여 요청을 시작하는 데 사용되는 클라이언트의 등록 식별자를 의미한다.

## ExceptionTranslationFilter

필터 체인에서 던져진 AccessDeniedException 과 AuthenticationException 을 관리한다. 자바 exception과 HTTP 응답간의 다리 역할을 하기 때문에 필수적인다. 어떠한 보안적인 강제를 하는 것은 없지만, 유저 인터페이스를 유지하는 역할을 한다. 이 필터는 AuthenticationException을 발견하면, authenticationEntryPoint를 시작하고, authenticationEntryPoint는 AbstractSecurityInterceptor의 서브 클래스에서 발생하는 인증 실패에 대한 일반적인 처리를 담당한다. 반면 이 필터가 AccessDeniedException을 발견하면, 익명의 사용자의 경우에는 authenticationEntryPoint를 시작하고, 익명의 사용자가 아닐 경우 AccessDeniedHandler에게 위임한다. 

이 필터를 사용하기 위해서는 authenticationEntryPoint 프로퍼티와 requestCache 프로퍼티를 설정해야 한다. authenticationEntryPoint는 AuthenticationException가 발견되면 인증 절차를 시작해야하는 핸들러를 지칭한다. authenticationEntryPoint로 인하여 http 프로토콜이 SSL 로그인을 위한 https 프로토콜로 바뀔 수 있다.
requestCache는 인증 절차시에 요청을 저장하는데에 사용되는 전략을 결정한다. 사용자가 인증이 완료되면 요청을 재사용되거나 반환할 수 있도록 한다. 기본적인 구현은 HttpSessionRequestCache를 따른다.