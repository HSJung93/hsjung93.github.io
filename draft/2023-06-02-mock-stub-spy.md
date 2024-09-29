---
title: "Mock vs Stub vs Spy"
date: 2023-06-02 22:00:01 +0900
categories: [강의, 주니어]
tags: [테스트, 용어]
---

어떤 소프트웨어를 테스트하여 개발할 때, Mock, Stub, Spy와 같은 용어들이 테스트 스턴트나 객체(test doubles or objects)들을 나타날 때에 사용된다. 테스트 스턴트나 객체는 테스트하려는 코드를 고립시켜야 하는 유닛 테스트 시에 실제 객체나 컴포넌트들의 행위를 모사하기 위하여 사용된다. 

먼저 Mock 객체는 해당 객체가 받을 요청에 대한 기대값들을 미리 프로그램해둔 테스트 스턴트 타입이다. 테스트하려는 코드와 모사된 객체 사이의 상호작용을 검증하기 위해서 사용된다. Mock은 어떤 동작에 대한 검증에 주로 사용된다. 테스트하려는 메서드가 정확한 인자들을 기대한 순서대로 요청하는지를 보장한다. Mock은 테스트하려는 메서드가 요청되었을 때에 미리 설정한 값들을 응답하거나 미리 정의된 동작을 실행하도록 할 수 있다.

예를 들어 `UserService` 인터페이스의 `getUserById()`가 데이터베이스로부터 유저 정보를 반환한다고 해보자. 다음과 같이 모의 프레임워크인 Mockito를 사용하여 인터페이스의 모의 구현을 생성할 수 있다. 

```kotlin
interface UserService {
    fun getUserById(userId: String): User
}

// Using Mockito to create a mock implementation
val userServiceMock = mock<UserService>()

// Setting expectations on the mock
val userId = "123"
val user = User("John Doe", "john@example.com")
`when`(userServiceMock.getUserById(userId)).thenReturn(user)

// Testing the code under test with the mock
val userManager = UserManager(userServiceMock)
val result = userManager.getUserDetails(userId)

// Asserting the result
assertEquals(user, result)
verify(userServiceMock).getUserById(userId)
```

`userServiceMock` 가 `UserService` 인터페이스의 모의 객체이며 이 Mock에 대한 기대 동작과 값을 `when`과 `getUserById()`메서드의 특정 응답 값으로 세팅할 수 있다. 그러면 `UserManager` 클래스의 `getUserDetails()` 메서드에서 이 Mock을 사용할 수 있다.

Stub은 메서드 요청에 대하여 미리 정의된 응답들을 제공하는 실제 객체와 컴포넌트의 간단한 버전이다. Mock과는 다르게 동작 검증에 대해서는 고려하지 않는다. 대신 실제 객체의 기대되는 동작을 따라하는 봉인된 응답을 제공하는데 집중한다. Stub은 실제 의존성을 사용하여 재현하기에는 아까운 특정 시나리오나 조건들을 모의하기 위해서 사용된다.

예를 들어 `PaymentProcessor` 클래스가 payment 게이트웨이와 상호작용한다고 해보자. 실제 지불 요청 없이 `processPayment()` 메서드를 테스트하기 위하여 Stub을 사용할 수 있다. `PaymentGatewayStub`은 언제나 `true`를 응답하는 구현체이다.

```kotlin
interface PaymentGateway {
    fun processPayment(amount: Double): Boolean
}

// Creating a stub implementation
class PaymentGatewayStub : PaymentGateway {
    override fun processPayment(amount: Double): Boolean {
        return true // Simulating a successful payment
    }
}

// Testing the code under test with the stub
val paymentGatewayStub = PaymentGatewayStub()
val paymentProcessor = PaymentProcessor(paymentGatewayStub)
val result = paymentProcessor.processPayment(100.0)

// Asserting the result
assertTrue(result)
```

Spy는 실제 객체를 감싸서 테스트 하려는 코드와 스파이된 객체간의 상호작용을 관찰하고 기록할 수 있도록 하는 테스트 스턴트이다. Spy는 해당 객체의 동작을 조작하지 않고 해당 객체와의 상호작용을 모니터하려고 할 때에 사용한다. Spy는 메서드가 요청된 횟수나 지나간 인자들 혹은 응답 정보까지도 기록할 수 있다. 이 기록된 정보들은 나중에 assertion이나 검증에 사용될 수 있다. 

예를 들어 파일에 메세지를 로깅하는 `Logger` 클래스에 스파이를 만들어 로거와의 상호작용을 모니터하도록 할 수 있다.

```kotlin
class Logger {
    fun log(message: String) {
        // Log the message to a file
    }
}

// Creating a spy by wrapping around a real object
val logger = Logger()
val loggerSpy = spy(logger)

// Testing the code under test with the spy
val message = "Test message"
val logManager = LogManager(loggerSpy)
logManager.logMessage(message)

// Verifying the interactions
verify(loggerSpy).log(message)

```

위 예시에서는 `LogManager.logMessage()` 메서드를 테스트하는데에 `Logger` 객체를 감싸서 `loggerSpy`를 만들고 사용하고 있다. 테스트 후에 스파이의 `log()` 메서드가 기대되는 메세지로 요청되었는지 `verify` 함수로 검증할 수 있다.