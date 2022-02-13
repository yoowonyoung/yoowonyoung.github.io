---
layout: post
title: "Effective Kotlin - 아이템5: 예외를 활용해 코드에 제한을 걸어라"
description: 예외를 활용해 코드에 제한을 걸어라
date: 2022-02-13 22:47:00 +09:00
categories: EffectiveKotlin Study
---


# 안정성

## 아이템 5 : 예외를 활용해 코드에 제한을 걸어라
- 확실하게 어떤 형태로 동작해야 하는 코드가 있다면 예외를 활용해서 제한을 걸어 두는게 좋음
- 코틀린에선 다음과 같은 방법으로 제한을 걸 수 있음
    * require 블록: 아규먼트를 제한 할 수 있음
    * check 블록: 상태와 관련된 동작을 제한 할 수 있음
    * assert 블록: 어떤 것이 true인지 확인 할 수 있음(테스트 모드에서만)
    * return 또는 throw 와 함께 활용하는 엘비스 연산자

```kotlin
fun pop(num: Int = 1): List<T> {
    require(num <= size) {
        "Cannnot remove more elements than current size"
    }
    check(isOpen) { "Cannot pop from closed stack" }
    val ret = collection.take(num)
    collection = collection.drop(num)
    assert(ret.size == num)
    return ret
}
```

- 제한을 걸어주면 다음과 같은 장점
    * 제한을 걸면 문서를 읽지 않은 개발자도 문제를 확인 할 수 있음
    * 문제가 있을 경우에 함수가 예상하지 못한 동작을 하지 않고 예외를 throw
    * 코드가 어느정도 자체적으로 검사됨. 이와 관련된 단위 테스트 코드도 줄일 수 있음
    * 스마트 캐스트 기능을 활용 할 수 있게 되므로 캐스트(타입 변환)을 적게 할 수 있음

### 아규먼트
- 함수를 정의 할때 타입 시스템을 활용해 아규먼트에 제한을 거는 코드를 많이 사용
    * 숫자를 아규먼트로 받아서 팩토리얼을 계산 한다면 숫자는 양의 정수여야 함
    * 좌표들을 아규먼트로 받아서 클러스터를 찾을 때는 비어있지 않은 좌표 목록이 필요
    * 사용자로부터 이메일 주소를 입력 받을 떄는 값이 입력 되어 있는지, 그리고 이메일 형식이 올바른지 확인 해야 함

- 아규먼트에 이러한 제한을 걸때는 require 함수를 사용 함. require 함수는 제한을 확인 하고, 제한을 만족 하지 못할 경우는 예외를 throw

```kotlin
fun factorial(n: Int): Long {
    require(n <= 0) 
    return if (n <= 1) 1 else fatcorial(n-1)*n
}
```

- 위와 같은 형태의 입력 유효성 검사 코드는 함수의 가장 앞부분에 배치 되므로 읽는 사람도 쉽게 확인 가능
- requeire 함수는 조건을 만족하지 못할 떄 무조건 적으로 IllegalArgumentException을 발생 시키므로 제한을 무시 할 수 없음
- 다음과 같이 람다를 사용 해서 지연 메시지를 정의 할 수도 있음

```kotlin
fun factorial(n: Int): Long {
    require(n <= 0)  { "Cannot calculate factorial of $n because it is smaller than 0" }
    return if (n <= 1) 1 else fatcorial(n-1)*n
}
```

### 상태
- 어떤 구체적인 조건을 만족할 때만 함수를 사용할 수 있게 해야 할때가 있음
    * 어떤 객체가 미리 초기화 되어 있어야만 처리를 하게 하고 싶은 함수
    * 사용자가 로그인 했을때만 처리 하게 하고 싶은 함수
    * 객체를 사용할 수 있는 시점에서 사용 하고 싶은 함수

- 상태와 관련된 제한을 걸 때는 일반적으로 check 함수

```kotlin
fun speak(text: String) {
    check(isInitialized)
    //...
}
```

- check 함수는 requeire함수와 비슷 하지만, 지정된 예측을 만족하지 못할때 IllegalStateException을 throw
- 예외 메시지는 require 와 마찬가지로 지연 메시지를 사용해서 변경 할 수 있음
- 함수 전체에 대한 예측이 있을 떄에는 일반적으로 require 블록 뒤에 배치. check를 나중에

### Assert 계열 함수
- 함수가 올바르게 구현 되어 있다면 확실하게 true를 만들 수 있는 코드들이 있음. 함수가 올바르지 않게 구현되어 발생하는 문제를 예방 하려면 단위 테스트를 사용 하는것이 좋음

```kotlin
class StackTest {
    @Test
    fun `Stack pops correct number of elements`() {
        val stack = Stack(20) { it }
        val ret = stack.pop(10)
        assertEquals(10, ret.size)
    }
}
```

- 단위 테스트는 구현의 정확성을 확인하는 가장 기본적인 방법
- 단위 테스트 대신 함수에서 assert를 사용 한다면 다음과 같은 장점
    * Assert 계열의 함수는 코드를 자체 점검하며 더 효율적으로 테스트 할 수 있게 해줌
    * 특정 상황이 아닌 모든 상황에 대한 테스트
    * 실행 시점에 정확하게 어떻게 되는지 확인 가능
    * 실제 코드가 더 빠른 지점에 실패하게 만듬. 따라서 예상하지 못한 동작이 언제 어디서 실행됬는지 쉽게 찾을 수 있음

### nullability와 스마트 캐스팅
- 코틀린은 require 와 check 블록으로 어떤 조건을 확인해서 true가 나왔다면 해당 조건은 이후로도 true일것이라 가정
- 따라서 이를 활용해서 타입 비교를 했다면 스마트 캐스트가 작동. 이러한특징은 어떤 대상이 null인지 확인 할 때 굉장히 유용

```kotlin
class Person(val email:String?)

fum sendEmail(person: Person, message: String) {
    require(person.email != null)
    val email: String = person.email
}
```

- 이러한 경우 requireNotNull, checkNotNull이라는 특수한 함수를 사용해도 좋고, 둘다 스마트 캐스트를 지우너 하므로 변수를 언팩하는 용도로도 활용 가능

```kotlin
class Person(val email: String?)

fun sendEmail(person: Person, text: String) {
    val email = requireNotNull(person.email)
    // ..
}
```

- nullablility를 목적으로 오른쪽에 throw 또는 return을 두고 엘비스 연산자를 활용하는 경우도 많은데, 이런 코드는 읽기 쉽고 유연하게 사용할 수 있음

```kotlin
fun sendEmail(person: Person, text: String) {
    val email: String = person.email ?: return
    // ...
}
```

- 프로퍼티에 문제가 있어 null일때 여러 처리를 해야 해도 return/throw 와 run 함수를 조합해서 활용 하면 됨

```kotlin
fun sendEmail(person: Person, text: String) {
    val email: String = person.email ?: run {
        log("Email not sent, no email address")
        return
    }
}
```

- 이처럼 return 과 throw 를 활용한 엘비스 연산자는 nullable을 확인할 떄 굉장히 많이 사용되는 관용적인 방법

### 정리
- 예외를 활용에 코드에 제한을 걸면 다음과 같은 이득이 있음
    * 제한을 더 쉽게 확인 가능
    * 애플리케이션을 더 안정적으로 지킬 수 있음
    * 코드를 잘못 쓰는 상황을 막을 수 있음
    * 스마트 캐스팅을 활용 가능

- 이를 위한 매커니즘은 다음과 같음
    * require 블록: 아규먼트와 관련된 에측을 정의 할 떄 사용하는 범용적인 방법
    * check 블록: 상태와 관련된 예측을 정의 할 때 사용하는 범용적인 방법
    * assert 블록: 테스트 모드에서 테스트를 사용할 떄 사용하는 범용적인 방법
    * return과 throw와 함께 엘비스 연산자 사용하기