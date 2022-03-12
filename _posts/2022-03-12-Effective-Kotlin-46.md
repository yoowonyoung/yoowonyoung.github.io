---
layout: post
title: "Effective Kotlin - 아이템46: 함수 타입 파라미터를 갖는 함수에 inline 한정자를 붙여라"
description: 함수 타입 파라미터를 갖는 함수에 inline 한정자를 붙여라
date: 2022-03-12 21:49:00 +09:00
categories: EffectiveKotlin Study
---


# 비용 줄이기

## 아이템 46: 함수 타입 파라미터를 갖는 함수에 inline 한정자를 붙여라

- 코틀린의 표준 라이브러리의 고차함수들을 보면 대부분 inline 한정자가 붙어있음

```kotlin
inline fun repeat(times: Int, action: (Int) => Unit) {
    for (index in 0 until times) {
        action(index)
    }
}
```

- inline 한정자의 역할은 컴파일 시점에 함수를 호출하는 부분을 함수의 본문으로 대체 하는것
- inline 한정자를 붙여 함수를 만들면 다음과 같은 장점이 있음
    * 타입 아규먼트에 reified 한정자를 붙여서 사용할 수 있음
    * 함수 타입 파라미터를 가진 함수가 훨씬 빠르게 동작
    * non-local 리턴을 사용할 수 있음

### 타입 아규먼트를 reified로 사용할 수 있다
- JVM 바이트코드에는 제너릭이 존재하지 않기 때문에 컴파일 하면 제너릭 타입과 관련된 내용이 제거가 되는데, 예를 들면 ```List<Int>``` 를 컴파일 하면 ```List```로 바뀜

```kotlin
any is List<Int> // 오류
any is List<*> // OK
```

- 함수를 인라인으로 만들면 reified 한정자를 사용할 수 있고, reified 한정자를 지정하면 타입 파라미터를 사용한 부분이 타입 아규먼트로 대체됨

```kotlin
inline fun <reified T> printTypeName() {
    print(T::class.smipleName)
}

printTypeName<Int>() // Int
printTypeName<Char>() // Char
printTypeName<String>() // String

// 컴파일 되면 이렇게 변함
print(Int::class.simpleName) // Int
print(Char::class.simpleName) // Char
print(String::class.simpleName) // String
```

### 함수 타입 파라미터를 가진 함수가 훨씬 빠르게 동작한다
- 모든 함수는 inline 한정자를 붙이 면 조금 더 빠르게 동작함. 함수 호출과 리턴을 윟 ㅐ점프하는 과정과 백스택을 추적하는 과정이 없기 때문
- 함수 파라미터를 가지지 않는 함수에서는 이러한 차이가 큰 성능 차이를 발생시키지 않음
- 코틀린/JVM에서는 JVM 익명 클래스 또는 일반 클래스를 기반으로 함수를 객체로 만들어내는데, 함수 본문을 객체로 wrap 하는 과정에서 코드의 속도가 더 느려지는것

### non-local return 을 사용할 수 있다
- inline 한정자를 사용하지 않는 함수 리터럴은 컴파일 될 때 함수가 객체로 래핑되기 때문에 내부에서 return을 사용할 수 없음. 함수가 다른 클래스에 위치하므로 return을 사용해서 main으로 돌아올 수 없는것
- inline 한정자를 사용하면 함수가 main 함수 내부에 박히기 때문에 사용 가능

### inline 한정자의 비용
- 인라인 함수는 재귀적으로 동작 할 수 없음. 재귀적으로 사용하면 무한루프에 빠지는데 이러한 무한루프는 인텔리제이가 오류로 잡아주지 못해서 위험
- 인라인 함수는 더 많은 가시성 제한 요소를 사용할 수 없음. public 인라인 함수 내에서는 private와 internal 가시성을 가진 함수와 프로퍼티를 사용 불가
- inline 한정자를 남용하면 코드의 크기가 쉽게 커짐

### crossinline과 noinline
- 함수를 인라인으로 만들고 싶지만 어떤 이유로 일부 함수 타입 파라미터는 inline으로 받고싶지 않은 경우엔 crossinline 이랑 noinline을 사용하면됨
- crossinline: 아규먼트로 인라인 함수를 받지만 비 지역적 리턴을 하는 함수는 받을수 없게 만듬. 인라인으로 만들지 않은 다른 람다 표현식과 조합해서 사용할때 문제가 발생하는 경우 사용
- noinline: 아규먼트로 인라인 함수를 받을 수 없게 만듬. 인라인 함수가 아닌 함수를 아규먼트로 사용하고 싶을떄 활용

```kotlin
inline fun requestNewToken(
    hasToken: Boolean,
    crossinline onRefresh: () -> Unit,
    noinline onGenerate: () -> Unit
) {
    if (hasToken) {
        httpCall("get-token",onGenerate)
        // 인라인이 아닌 함수를 아규먼트로 함수에 전달 하려면 noinline
    } else {
        httpCall("refresh-token") {
            onRefresh() // Non-local 리턴이 허용되지 않는 컨텍스트에서 inline 함수를 사용하고 싶다면 crossinline
            onGenerate()
        }
    }
}

fun httpCall(url: String, callBack: () -> Unit) {
    /*..*/
}
```

