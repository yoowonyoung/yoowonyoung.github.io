---
layout: post
title: "Effective Kotlin - 아이템34: 기본 생성자에 이름 있는 옵션 아규먼트를 사용하라"
description: 기본 생성자에 이름 있는 옵션 아규먼트를 사용하라
date: 2022-03-01 22:05:00 +09:00
categories: EffectiveKotlin Study
---


# 객체 생성

## 아이템 34: 기본 생성자에 이름 있는 옵션 아규먼트를 사용하라

### 점층적 생성자 패턴
- 여러가지 종류의 생성자를 사용하는 굉장히 간단한 패턴

```kotlin
class Pizza {
    val size: String
    val cheese: Int
    val olives: Int
    val bacon: Int

    constructor(size: String, cheese: Int, olives: Int, bacon: Int) {
        this.size = size
        this.cheese = cheese
        this.olives = olives
        this.bacon = bacon
    }
    constructor(size: String, cheese: Int, olives: Int): {
        this(size, cheese, olives, 0)
    }
    constructor(size: String, cheese: Int) {
        this(size, cheese, 0, 0)
    }
    constructor(size: String) {
        this(size, 0, 0, 0)
    }
}
```

- 이는 그렇게 좋은 코드가 아님. 코틀린에서는 일반적으로 디폴트 아규먼트를 사용

```kotlin
class Pizze(
    val size: String,
    val cheese: Int = 0,
    val olives: Int = 0,
    val bacon: Int = 0
)
```

- 디폴트 아규먼트가 점층적 생성자보다 좋은 이유
    * 파라미터들의 값을 원하는대로 지정 가능
    * 아규먼트를 원하는 순서로 지정 가능
    * 명시적으로 이름을 붙여서 아규먼트를 지정하므로 의미가 명확

## 빌더 패턴
- 자바에서는 빌더 패턴을 이용해 파라미터에 이름을 붙이거나, 원하는 순서대로 지정하고, 디폴트 값을 지정함
- 빌더를 사용하는것보다 이름있는 파라미터를 사용하는것이 더 좋은 이유
    * 더 짦음: 디폴트 아규먼트가 있는 생성자 또는 팩토리 메서드가 빌더보다 구현하기가 더 쉬움
    * 더 명확함: 객체가 어떻게 생성되는지 확인 하고 싶다면 생성자 주변 부분만 확인 하면 됨
    * 더 사용하기 쉬움: 생성자는 기본적으로 언어에 내장된 개념이지만, 빌더는 언어 위에 추가로 구현한 개념임
    * 동시성과 관련된 문제가 없음: 코틀린의 함수 파라미터는 항상 immutable 이지만, 빌더 패턴에서의 프로퍼티는 mutable 이라서 thread-safe 하게 구현하긴 힘듬

- 무조건 빌더보다 기본 생성자가 좋다는것은 아님

```kotlin
val dialog = AlertDialog.Builder(context)
    .setMessage(R.string.fire_missiles)
    .setPositiveButton(R.string.fire, { d, id ->
        // 미사일 발사
    })
    .setNegativeButton(R.string.cancel, { d, id ->
        // 취소를 누른 경우
    })
    .create()


// 빌더를 사용하지 않는 코드
val dialog = AlertDialog(context,
    message = R.string.fire_missiles,
    positiveButtonDescription = 
        ButtonDescription(R.string.fire, { d, id ->
            // 미사일 발사
        }),
    negativeButtonDescription = 
        ButtonDescription(R.string.cancel, { d, id ->
            // 취소를 누른 경우
        })
)
```

- 이러한 코드는 다음과 같이 DSL 빌더를 사용하는게 일반적임

```kotlin
val dialog = context.alert(R.string.fire_missile) {
    positiveButton(R.string.fire) {
        // 미사일 발사
    }
    negativeButton {
        // 취소를 누른 경우
    }
}
```

- 코틀린에서 빌더는 거의 사용되지 않으며 다음과 같은 경우에만 사용함
    * 빌더 패턴을 사용하는 다른 언어로 작성된 라이브러리를 그대로 옮길때
    * 디폴트 아규먼트와 DSL을 지원하지 않는 다른 언어에서 쉽게 사용할 수 있게 API를 설계 할 때
