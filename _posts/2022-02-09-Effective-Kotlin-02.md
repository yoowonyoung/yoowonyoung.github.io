---
layout: post
title: "Effective Kotlin - 아이템2: 변수의 스코프를 최소화 하라"
description: 변수의 스코프를 최소화 하라
date: 2022-02-09 20:27:00 +09:00
categories: EffectiveKotlin Study
---


# 안정성

## 아이템 2 : 변수의 스코프를 최소화 하라
- 상태를 정의할떄는 변수와 프로퍼티의 스코프를 최소화 하는것이 좋음
    * 프로퍼티 보다는 지역변수
    * 최대한 좁은 스코프를 갖게 변수를 사용

- 스코프 내부에 스코프가 있을 수도 있음(람다 표현식 내부의 람다 표현식 등). 최대한 변수는 스코프를 좁게 설정하는게 좋음
- 스코프가 좁은게 중요한 이유는 프로그램을 추적하고 관리하기 쉽게 만들기 위함
- mutable 프로퍼티는 좁은 스코프에 걸쳐 있을수록 그변경을 추적하는게 쉬우며, 이렇게 추적이 되어야 코드를 이해하고 변경하는것이 쉬움
- 변수의 스코프 범위가 넓으면 다른 개발자에 의해서 변수가 잘못 사용될 수도 있음
- 변수는 읽기 전용 또는 읽고 쓰기 전용 여부와 상관없이 변수를 정의할때 초기화 되는것이 좋음
- 여러 프로퍼티를 한꺼번에 설정 해야 하는경우 구조분해를 사용하는게 좋음

```kotlin
fun updateWeather(degrees: Int) {
    val (description, color) = when {
        degrees < 5 -> "cold" to Color.BLUE
        degrees < 23 -> "mild" to Color.YELLOW
        else -> "hot" to Color.RED
    }
}
```

### 캠처링
- 시퀀스를 이용해서 에라토스 테네스의 체 알고리즘을 구현한 예제

```kotlin
val primes: Sequence<Int> = sequence {
    var numbers = generateSequence(2) { it + 1 }

    while(true) {
        val prime = numbers.first()
        yeild(prime)
        numbers = numbers.drop(1).filter( it % prime != 0 )
    }
}


print(primes.take(10).toList())
```

- prime을 var로 선언하고, 반복문 외부에 만듦으로써 최적화 하려고 시도 할 수 있음. 하지만 이것은 이상하게 동작할것

```kotlin
val primes: Squence<Int> = sequence {
    var numbers = generateSequence(2) { it + 1 }

    var prime: Int
    while(true) {
        prime = numbers.first()
        yeild(prime)
        numbers = numbers.drop(1).filter{ it % prime != 0 }
    }
}
```

- prime 변수가 캡쳐되었기 떄문에 발생한 문제인데, 시퀀스를 이용하므로 필터링이 지연되고 최종 prime값으 필터링 되어서 발생한 문제임. 스코프 범위가 좁다면 이런 문제를 피할수있음

### 정리
- 여러가지 이유로 변수의 스코프는 좁게 만들어서 활용하는게 좋음
- var보다는 val을 이용하는게 좋음
- 람다에서 변수가 캡쳐되는것에 주의