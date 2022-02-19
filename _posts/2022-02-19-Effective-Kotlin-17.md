---
layout: post
title: "Effective Kotlin - 아이템17: 이름있는 아규먼트를 사용 하라"
description: 이름있는 아규먼트를 사용 하라
date: 2022-02-19 16:49:00 +09:00
categories: EffectiveKotlin Study
---


# 가독성

## 아이템 17 : 이름있는 아규먼트를 사용 하라
- 코드에서 아규먼트의 의미가 명확하지 않은 경우가있음. 파라미터가 명확하지 않는 경우에는 이를 직접 이름 있는 아규먼트로 지정해서 명확하게 만들 수 있음

```kotlin
val text (1..10).joinToString(separator = "|")
```

### 이름 있는 아규먼트를 언제 사용 해야 할까?
- 이름 있는 아규먼트의 장점
    * 이름을 기반으로 값이 무엇을 나타내는지 알 수 있음
    * 파라미터 입력 순서와 상관이 없으므로 안전

- 아규먼트 이름은 함수를 사용하는 개발자 뿐만 아니라 코드를 읽는 다른 사람들에게도 굉장히 중요한 정보를 줌
- 다음과 같은 경우에 이름 있는 아규먼트를 추천
    * 디폴트 아규먼트의 경우
    * 같은 타입의 파라미터가 많은 경우
    * 함수 타입의 파라미터가 있는 경우(마지막 경우 제외)

### 디폴트 아규먼트의 경우
- 프로퍼티가 디폴트 아규먼트를 가질 경우, 항상 이름을 붙여서 사용하는것이 좋음
- 일반적으로 함수 이름은 필수 파라미터와 관련되어 있기 때문에 디폴트 값을 갖는 옵션 파라미터의 설명이 명확하지 않기 때문에 이름을 붙여 사용하는게 좋음

### 같은 타입의 파라미터가 많은 경우
- 파라미터에 같은 타입이 있다면 잘못 입력 했을떄 문제를 찾아내기 어려울 수 있음

```kotlin
fun sendMail(to: String, message: String) { //... }

sendMail(
    to = "contact@ky.academy"
    message = "Hello"
)
```

### 함수 타입 파라미터
- 일반적으로 함수 타입 파라미터는 마지막 위치에 배치하는것이 좋음
- 함수 이름이 함수 타입 아규먼트를 설명해주기도 하는데, 일반적으로 마지막에 위치하는 함수 파라미터에 대해서만 설명. 그 밖의 모든 함수 타입 아규먼트는 이름 있는 아규먼트를 사용하는게 좋음

```kotlin
val view = linearLayout {
    text("Click below")
    button({ /* 1 */ }, { /* 2 */ }) // 어느 부분이 빌더이고 어떤 부분이 클릭 리스너일것인가?
}

val view = linearLayout {
    text("Click below")
    button(onClick = { /* 1 */ }, { /* 2 */ }) 
}

// 여러 함수 타입의 옵션 파라미터가 있는 경우엔 더 헷갈림
fun call(before: () -> Unit = {}, after: ()-> Unit = {}) {
    before()
    print("Middle")
    after()
}

call({ print("CALL") }) //CALLMiddle
call { print("CALL") } // MiddleCALL

// 이름을 붙여야 이해가 쉽다
call(before = { print("CALL") })
call(after = { print("CALL") })
```

- 마지막 파라미터가 DSL처럼 특별한 의미를 가지는 경우에는 생략한다
