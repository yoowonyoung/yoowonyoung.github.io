---
layout: post
title: "Effective Kotlin - 아이템49: 하나 이상의 처리 단계를 가진 경우에는 시퀀스를 사용 하라"
description: 하나 이상의 처리 단계를 가진 경우에는 시퀀스를 사용 하라
date: 2022-03-13 13:45:00 +09:00
categories: EffectiveKotlin Study
---


# 효율적인 컬렉션 처리

## 아이템 49: 하나 이상의 처리 단계를 가진 경우에는 시퀀스를 사용 하라

- Iterable과 Sequence는 거의 비슷해보이지만 완전히 다른 목적으로 설계되었고, 완전히 다른 형태로 동작함
    * Sequence는 지연처리 되기 떄문에, 시퀀스 처리 함수를 사용하면 데코레이터 패턴으로 꾸며진 새로운 시퀀스가 리턴되며 최종적인 계산은 최종 연산이 이뤄질때 수행됨
    * Iterable 처리 함수를 사용할 때마다 연산이 이뤄져서 List가 만들어짐

```kotlin
public inline fun<T> Iterable<T>.filter(
    predicate: (T) -> Boolean
): List<T> {
    return filterTo(ArrayList<T>(), predicate)
}

public fun <T> Sequence<T>.filter(
    predicate: (T) -> Boolean
): Sequence<T> {
    return FilteringSequence(this, true, predicate)
}
```

- 시퀀스의 지연 처리는 다음과 같은 장점을 가짐
    * 자연스러운 처리 순서를 유지
    * 최소한의 연산만 함
    * 무한 시퀀스 형태로 사용 가능
    * 각각의 단계에서 컬렉션을 만들어내지 않음

### 순서의 중요성
- 시퀀스 처리는 요소 하나 하나에 지정한 연산을 한꺼번에 적용 하는데, 이를 element-by-element order 또는 lazy order라고 부름
- 이터러블 처리는 요소 전체를 대상으로 연산을 차근차근 적용해나가는데, 이를 step-by-step order 또는 eager order 라고 부름

```kotlin
sequenceOf(1,2,3)
    .filter { print("F$it, "); it%2 == 1 }
    .map { print("M$it, "); it*2 }
    .forEach { print("E$it, ") }
// F1, M1, E2, F2, F3, M3, E6

listOf(1,2,3)
    .filter { print("F$it, "); it%2 == 1 }
    .map { print("M$it, "); it*2 }
    .forEach { print("E$it, ") }
// F1, F2, F3, M1, M3, E2, E6
```

- 컬렉션 처리 함수를 사용하지 않고 고전적인 반복문과 조건문을 활용해서 구현하면 이는 시퀀스 처리인 element-by-element order와 같음. 따라서 시퀀스 처리에서 사용되는 elememt-by-element order가 훨씬 자연스러운 처리

### 최소 연산
- 컬렉션에 어떤 처리를 적용햐고 앞에서 10개만 필요한 상황은 굉장이 자주 접할 수 있는 상황인데, 이터러블 처리는 기본적으로 중간연산이라는 개념이 없으므로 원하는 처리를 컬렉션 전체에 적용한 뒤 앞의 요소 10개를 사요앻야 하지만, 시퀀스는 중간연산이라는 개념을 갖고 있으므로 앞의 요소 10개에만 원하는 처리를 적용 가능

```kotlin
(1..10).asSequence()
    .filter { print("F$it, "); it%2 == 1 }
    .map { print("M$it, "); it*2 }
    .find { it > 5 }
// F1, M1, F2, F3, M3

(1..10)
    .filter { print("F$it, "); it%2 == 1 }
    .map { print("M$it, "); it*2 }
    .find { it > 5 }
// F1, F2, F3, F4, F5, F6, F7, F8, F9, F10, M1, M3, M5, M7, M9
```

- 중간 처리 단계를 모든 요소에 적용할 필요가 없는 경우에는 시퀀스를 사용하는것이 좋음
- 처리를 적용하는 요소를 선택하는 연산으로는 first, take, any, all, none, indexOf, find 등이 있음

### 무한 시퀀스
- 시퀀스는 최종 연산이 일어나기 전까지는 컬렉션에 어떤 처리도 하지 않기 때문에 무한 시퀀스를 만들고 필요한 부분 까지만 값을 추출 하는것도 가능
- 무한 시퀀스는 generateSequence 또는 sequence를 사용해서 만듬

```kotlin
generateSequence(1) { it + 1 } // 첫 번쨰 요소 1과, 그 다음 요소를 계산하는 방법인 it + 1을 지정
    .map{ it * 2 }
    .take(10)
    .forEach { print("%it, ") } 
// 2, 4, 6, 8, 10, 12, 14, 16, 18 20
```

- 무한 시퀀스를 실제로 사용할떄는 값을 몇개 활용할지를 지정해야 하는데, 그렇지 않으면 무한하게 반복하기 때문임. take로 활용할 값의 수를 지정 하거나 first, find, any, all, none, indexOf 와 같은 일부 요소만 선택하는 종결 연산을 활용해야함
- 무한 시퀀스는 종결 연산으로 take 또는 first 정도만 쓰는게 좋음

### 각각의 단계에서 컬렉션을 만들어내지 않음
- 표준 컬렉션 처리 함수는 각각의 단계에서 새로운 컬렉션을 만들어 내는데, 이를 활용하거나 저장할수 있다는것은 장점이지만 공간을 차지하는 비용이 드는 단점이 있음
- 컬렉션 처리의 각각의 단계에서 새로운 컬렉션을 만드는데 비용이 들어가는데, 크기가 큰 요소를 처리할수록 그 비용이 커지고, 처리 단계가 많아질수록 비용이 커짐

### 시퀀스가 빠르지 않은 경우
- 컬렉션 전체를 기반으로 처리해야하는 연산은 시퀀스를 사용해도 빨라지지 않는데, 현재 유일한 예시로 stdlib의 stored가 있음
- 참고로 무한 시퀀스처럼 시퀀스의 다음 요소를 lazy하게 구하는 시퀀스에 sorted를 적용하면 무한 반복에 빠지는 문제가 있음

### 자바 스트림의 경우
- 자바8부터 등장한 스트림의 경우 코틀린의 시퀀스와 비슷한 형태로 동작. lazy하게 작동하며 마지막 처리 단계에서 연산이 일어남
- 자바의 스트림과 코틀린의 시퀀스의 차이
    * 코틀린의 시퀀스가 더 많은 처리 함수를 가지고 있고, 사용하기가 더 쉬움
    * 자바 스트림은 병렬 함수를 사용해 병렬 모드로 실행할 수 있어서 멀티코어 환경에서 굉장히 큰 성능 향상을 가져올수있지만, 몇가지 결함이 있으므로 주의 해야함
    * 코틀린의 시퀀스는 코틀린/JVM, 코틀린/JS, 코틀린/네이티브등의 일반적인 모듈에서 모두 사용가능하지만 자바 스트림은 코틀린/JVM에서만 그것도 JVM 8이상에서만 동작

