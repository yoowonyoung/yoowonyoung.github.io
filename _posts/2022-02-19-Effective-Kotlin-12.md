---
layout: post
title: "Effective Kotlin - 아이템12: 연산자 오버로드를 할 때는 의미에 맞게 사용 하라"
description: 연산자 오버로드를 할 때는 의미에 맞게 사용 하라
date: 2022-02-19 14:41:00 +09:00
categories: EffectiveKotlin Study
---


# 가독성

## 아이템 12 : 연산자 오버로드를 할 때는 의미에 맞게 사용 하라

```kotlin
fun Int.factorial(): Int = (1..this).product()
fun Iterable<Int>.product(): Int =
    fold(1) { acc, i -> acc*i }

operator fun Int.not() = factorial()
print(10 * !6)
```

- 위와 같이 팩토리얼을 연산자 오버로딩을 통해 구현 할 수 있지만, 이는 좋은 코드가 아님
- 코틀린의 모든 연산자는 구체적인 이름을 가진 함수에 대한 별칭일 뿐임. 모든 연산자는 연산자 대신 함수로도 호출 가능

```kotlin
x + y == z

x.plus(y).equal(z)
```

### 분명하지 않은 경우
- 관례를 충족하는지 아닌지 확실하지 않을 때가 문제임

```kotlin
operator fun Int.times(operation: () -> Unit): () -> Unit =
    { repeat(this) { operation() } }

val tripledHello = 3 * { print("Hello") }
tripledHello() // HelloHelloHello

operator fun Int.times(operation: () -> Unit_ {
    repeat(this) { operation() }
}

3 * { print("Hello") }
```

- 함수를 세 배 한다는것 (* 연산자)에서 누군가는 함수를 세번 반복하는 새로운 함수를 만들어낸다고 생각 할 수도, 누군가는 함수를 세번 호출하는걸로 이해 할 수도 있음
- 의미가 명확하지 않다면 infix를 활용한 확장 함수를 사용해서 이항 연산자의 형태로 쓰거나 톱레벨 함수로. 사용하는게 좋음. 사실 이미 함수를 n번 호출 하는것은 다음과 같은 형태로 이미 stdlib에 구현 되어 있음

```kotlin
infix fun Int.timesRepeated(operation: ()-> Unit) = {
    repeat(this) { operation() }
}

val tripledHello = 3 timesRepeated { print("Hello") }
tripledHello()

// 톱레벨 함수
repeat(3) { print("Hello")}
```

### 규칙을 무시해도 되는 경우
- 연산자 오버로딩 규칙을 무시해도 되는 중요한 경우가 있는데, 도메인 특화 언어(Domain Specific Language, DSL)을 설계 할 때임
