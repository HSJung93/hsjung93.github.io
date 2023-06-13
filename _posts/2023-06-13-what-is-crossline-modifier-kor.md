---
title: "crossline 지시어"
date: 2023-06-13 22:00:00 +0900
categories: [강의, 주니어]
tags: [코틀린, crossline]
---

- 기본적으로 코틀린은 람다 표현식에서 비지역적인 리턴을 허용한다. 
- 그래서 람다 안에서나 람다가 정의된 메서드 안에서 `return` 키워드가 사용될 수 있다. 
- `crossline` 지시어는 함수라 람다 표현에서 비-지역적인 리턴을 허용하지 않아야할 때 사용한다.
    - 예를 들어 고차 함수를 다룰 때 바디 안에서 흐름 제어를 강제할 필요가 있을 수 있다.
- crossline 지시어가 사용된 함수나 람다는 `return` 키워드를 사용하여 종료할 수 없다. 
    - 파라미터로 들어오는 람다가 `crossline`로 지시되어 있으면, 컴파일러는 람다 자신만 return 할 수 있도록 제한하여, 람다를 감싸는 함수나 메서드에는 영향을 미치지 않도록 한다. 

```kotlin
inline fun higherOrderFunction(crossinline lambda: () -> Unit) {
    val runnable = Runnable {
        lambda() // Invoking the lambda
    }
    // ...
    runnable.run() // Running the runnable
    // ...
}

fun main() {
    higherOrderFunction {
        // Some code
        // return // Error: Return is not allowed here
    }
}

```

- 예를 들어, 위 코드에서 `higherOrderFunction` 은 람다 파라미터를 받는 인라인 함수
- 람다 파라미터를 `crossinline` 으로 지시하여 람다가 지역적인 리턴만 할 수 있도록 보장하고 있다.