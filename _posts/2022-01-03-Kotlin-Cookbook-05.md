---
layout: post
title: "Kotlin Cookbook - 6장: 시퀀스"
description: Kotlin Cookbook 6장
date: 2022-01-03 18:48:00 +09:00
categories: Kotlin Cookbook Study
---

# 시퀀스

## 지연 시퀀스 사용하기

### 문제
- 특정 조건을 만족시키는데 필요한 최소량의 데이터만 처리하고 싶다

### 해법
- 코틀린 시퀀스를 쇼트 서킷 함수와 함께 사용한다

### 설명
- 코틀린은 기본 컬렉션에 확장함수를 추가해두었기 때문에 List에는 map과 filter같은 함수가 있음. 이 함수는 즉시 처리되는데 컬렉션의 모든 원소를 처리 해야 한다는 의미
- 이러한 방법은 끔직하게 비효율적이지만, 다행스럽게도 predicate를 받는 first 함수도 있음

```kotlin
(100 until 200).map { it * 2 }
    .first{ it % 3 == 0 }
```

- 위 예제에서 first함수는 컬렉션의 각 원소를 처리하기위해 루프를 사용하지만 predicate를 만족하는 첫번째 원소를 발견하는 순간 진행을 멈춤. 이러한 처리 방식을 쇼트 서킷이라고 부름
- 코틀린 시퀀스는 데이터를 다른 방식으로 처리함

```kotlin
(100 until 200).asSequence()
    .map { println("doubling $it" ); it * 2}
    .filter { println("filtering $it"); it % 3 == 0}
    .first()
```

- 이 예제의 시퀀스에서 어떤 filter 함수가 쓰였는지는 중요하지 않음. 어떤 방법을 사용하던 시퀀스의 각 원소는 다음 원소로 진행하기전에 완전한 전체 파이프라인에서 처리되기 때문에 위의 예제에서는 딱 6번만 연산이 수행됨
- 시퀀스는 스트림과 동일하게 중간연산 최종연산으로 나뉘고 최종연산 없이는 시퀀스가 데이터를 처리하지 않음
- 시퀀스는 스트림과 다르게 일부 시퀀스는 여러번 순회가 가능하며, 그렇지 못한 시퀀스는 여러번 순회가 불가능

## 시퀀스 생성하기

### 문제
- 값으로 이뤄진 시퀀스를 생성하고 싶다

### 해법
- 이미 원소가 있다면 sequenceOf를 사용하고 Iterable이 있다면 asSequence를 사용한다. 그 이외의 경우에는 시퀀스 생성기를 사용

### 설명
- 원소가 있거나 Iterable이 있는 경우는 간단. sequenceOf는 arrayOf, listOf와 똑같이 동작하고, asSequence는 기존의 Iterable을 시퀀스로 변환한다

```kotlin
val numSequence1 = sequenceOf(1,2,3,4,5,6,7)
val numSequence2 = listOf(1,2,3,4,5,6,7).asSequence()
```

- 위의 두 구문은 주어진 값 또는 리스트로부터 ```Sequence<Int>```를 생성한다

## 무한 시퀀스 다루기

### 문제
- 무한대의 원소를 갖는 시퀀스의 일부분이 필요 하다

### 해법
- 넝을 리턴하는 시퀀스 생성기를 사용하거나, 시퀀스 확장함수 중에서 takeWhile같은 함수를 사용하자

### 설명
- 자바의 스트림과 비슷한 시퀀스에는 중간연산과 최종연산이 있고, 중간연산은 새로운 시퀀스를 리턴하고 최종연산은 시퀀스가 아닌 어떤것이든 리턴할수 있다. 시퀀스에서 함수 호출로 연결된 파이프라인을 생성할때 최종 연산이 수행될떄까지 어떤 데이터도 시퀀스의 파이프라인을 통과하지 않는다

```kotlin
fun nexrPrime(num: Int) =
    generateSequence(num + 1) { it + 1 }
        .first(Int::isPrime)

fun firstNPrimes(count: Int) =
    generateSequence(2, ::nextPrime)
        .take(count)
        .toList()
```

- firstNPrimes를 통해 생성된 시퀀스는 무한대의 원소를 갖는데, take함수는 상태가 없는 중간 연산이며 처음 count개수의 값만으로 구성된 시퀀스를 리턴한다. 끝에 toList함수 없이 단순히 take함수만 실행한다면 어무런 소수도 계산되지 않는다. 이 경우 시퀀스는 있지만 시퀀스 안에 값은 없다
- 최종 연산인 toList는 실제로 값을 계산하는데 사용되고 계산된 모든 값을 하나의 리스트로 리턴한다
- 무한대의 원소를 갖는 시퀀스를 잘라내는 다른 방법은 마지막에 널을 리턴하는 생성 함수를 사용하는 것

```kotlin
fun primeLessThan(max: Int): List<Int> =
    generateSequence(2) { n -> if(n < max) nextPrime(n) else null }
        .toList()
        .dropLast(1)
```

- primeLessThan 함수는 generateSequence를 호출해 현재 값이 제공된 한계 값보다 작은 값인지 여부를 확인. 현재 값이 한계 값보다 작으면 다음 소수를 계산하고, 한계 값보다 크다면 null을 리턴하는데, 리턴된 null은 시퀀스를 종료
- 다음 소수가 한계값보다 큰 값인지 여부를 미리 알수 없기 떄문에 이 함수는 실제로 한계값을 넘어가는 소수를 하나 포함하는 리스트를 생성해서 dropLast를 통해서 하나를 잘라냄

```kotlin
fun primeLessThan(max: Int): List<Int> =
    generateSequence(2, ::nextPrime)
        .takeWhile { it < max }
        .toList()
```

- 동일한 기능을 하는 primeLessThan. takeWhile함수는 시퀀스에서 제공된 술어가 true를 리턴하는 동안 시퀀스에서 값을 추출

## 시퀀스에서 yield 하기

### 문제
- 구간을 지정해 시퀀스에서 값을 생성하고 싶다

### 해법
- yield 중단 함수와 함께 sequence 함수를 사용

### 설명

```kotlin
fun <T> sequence(
    block: suspend SequenceScope<T>.() -> Unit
): Sequence<T>
```

- sequence함수는 주어진 블록에서 평가되는 시퀀스를 생성, 이 블록은 인자 없는 람다 함수이며 void를 리턴하고 평가 후에 SequenceScope타입을 받음. sequence함수는 필요한때 yield값을 생성하는 람다를 제공 해야함

```kotlin
fun fibonacciSequence() = sequence {
    var terms = Pair(0,1)

    while(true) {
        yield(terms.first)
        terms = terms.second to terms.first + terms.second
    }
}
```

- sequence에 제공된 람다 함수는 피보나치 숫자의 처음 두 수인 0과 1을 담고 있는 Pair로부터 시작해서 무한루프를 이용해 다음 피보나치 수를 생성. 새로운 원소가 생성될때마다 yield함수는 결과 Pair값의 첫번쨰 원소를 리턴
- yield함수는 sequence연산에 제공된 람다를 받는 SequenceScope의 일부로써 SequenceScope이 가진 비슷한 두 함수중 하나

```kotlin
abstract suspend fun yield(value: T)

abstract suspend fun yieldAll(iterator: Iterator<T>)
suspend fun yieldAll(elements: Iterable<T>)
suspend fun yieldAll(sequence: Sequence<T>)
```

- yield함수는 이터레이터에 값을 제공하고 다음 값을 요청할떄까지 값 생성을 중단. 따라서 yield는 suspend함수가 생성한 시퀀스 안에서 각각의 값을 출력되는데 사용
- yield함수가 suspend함수기 때문에 코루틴과도 잘 동작하며, 코틀린 런타임은 코루틴에 값을 제공한후에 다음 값을 요청할떄까지 해당 코루틴을 중단 시킬수있다. 이러한 이유로 while(true)인 무한 루프가 존재할수있고, take연산에 의해 yield가 호출될떄마다 무한 루프는 값을 하나씩 제공

```kotlin
@Test
fun first10Fibonacci {
    val fibs = fibonacciSequence()
        .take(10)
        .toList()

    assertEquals(listOf(0,1,1,2,3,5,8,13,21,34), fibs)
}
```

- yieldAll은 다수의 값을 이터레이터에 넘겨준다

```kotlin
val sequence = sequence {
    val start = 0
    yield(start)  // 단일 값을 yield
    yieldAll(1..5 step 2) // 범위를 통해 순회 가능 (1,3,5)를 yield
    yieldAll(generateSequence(8) { it * 3 }) // 8부터 시작해 각각의 값에 3을 곱한값을 원소로 갖는 무한 시퀀스를 yield
}
```

- 위 코드는 결과적으로 0,1,3,5,8,24,72...을 원소로 갖는 시퀀스이며 take함수를 사용해 이 sequence에 접근하면 주어진 패턴을 사용해서 요청한 수 만큼의 원소를 리턴
- suspend함수 안의 yield와 yieldAll의 조합을 사용하면 시퀀스에 생성된 값을 원하는 조합으로 쉽게 바꿀 수 있음
 