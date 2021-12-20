---
layout: post
title: "Kotlin Cookbook - 4장: 함수형 프로그래밍"
description: Kotlin Cookbook 4장
date: 2021-12-20 21:01:00 +09:00
categories: Kotlin Cookbook Study
---

# 함수형 프로그래밍

## 알고리즘에서 fold 사용하기

### 문제
- 반복 알고리즘을 함수형 방식으로 구현하고 싶다

### 해법
- fold 함수를 사용해 시퀀스나 컬렉션을 하나의 값으로 축약 시킨다

### 설명
- fold 함수는 배열 또는 반복 가능한 컬렉션에 적용할수 있는 축약 연산이며 문법은 다음과 같다

```kotlin
inline fun <R> Iterable<T>.fold(
    initial: R,
    operation: (acc: R, T) -> R
): R
```

- fold는 2개의 인자를 받는데 첫번쨰는 누적자(accumulator)의 초깃값이며 두번쨰는 두개의 인자를 받아 누적자를 위해 새로운 값을 리턴하는 함수이다

```kotlin
fun sum(vararg nums: Int) =
    nums.fold(0) { acc, n -> acc + n }
```

- 위의 예시에서 초기값은 0이고, 2개의 인자를 받는 람다 함수를 제공 하는데, 람다 함수의 첫번쨰 인자는 누적에서 사용하는 값이며, 두번째 인자는 num 리스트의 각각의 값을 순회하며 첫번째 인자인 누적 값에 순회중인 값을 N 더하는 함수

```kotlin
fun factorialFold(n: Long): BigInteger =
    when(n) {
        0L, 1L -> BigInteger.ONE
        else -> (2..n).fold(BigInteger.ONE) { acc, i -> acc * BigInteger.valueOf(i) }
   }
```

- when 조건식으로 입력 인자가 0 또는 1인지 검사해서 이에 해당하면 BigInteger.ONE을 리턴하며, 2부터는 else 문을 통해 입력 숫자 n까지의 범위를 사용해 fold연산. 람다 안에서 누적값은 이전 누적값과 순회하는 각 원소값의 곱

## reduce 함수를 이용해 축약하기

### 문제
- 비어있지 않은 컬렉션의 값을 축약하고 싶지만 누적자의 초기값을 설정하고 싶지 않다

### 해법
- fold 대신 reduce 연산을 사용

### 설명
- reduce 함수의 시그니처는 다음과 같다

```kotlin
inline fun <S, T : S> Iterable<T>.reduce(
    operation: (acc: S, T) -> S
): S
```

- reduce는 fold 함수랑 거의 같은데 사용 목적도 거의 비슷. reduce 함수에는 누적자의 초기값 인자가 없다는것이 fold와 가장 큰 차이. 누적자의 초기값은 컬렉션의 첫번째 값으로 초기화 됨

```kotlin
public inline fun IntArray.reduce(operation: (acc: Int, Int) -> Int): Int {
    if(isEmpty())
        throw UnsupprotedOperationException("Empty array can't be reduce")
    var accumulator = this[0]
    for(index in 1...lastIndex) {
        accumulator = operation(accumulator, this[index])
    }
    return accumulator
}
```

- 코틀린 표준 라이브러리의 reduce 구현 방법. 따라서 reduce 함수는 누적자를 컬렉션의 첫번째 값으로 초기화 할 수 있는 경우에만 사용 가능
- 컬렉션의 첫번째 값은 누적자를 초기화 하는데만 사용하므로 다음과 같은 경우에는 원하는 값이 나오지 않음

```kotlin
fun sumReduceDouble(vararg nums: Int) =
    nums.reduce { acc, i -> acc + 2*i }
```

- 위의 예시에서 첫번쨰 값은 누적자를 초기화 하는데 사용 되었으므로 2배가 되지 않아서 원하는 값이 나오지 않음. 이런 경우에는 fold를 사용 해야함
- 자바 스트림에는 reduce라는 이름의 메소드가 2개 있는데, 하나는 바이너리 연산자(람다)를 받고, 다른 하나는 바이너리 연산자 뿐만 아니라 fold에 제공할 초기값도 받음. 초기값을 받지 않는 reduce의 리턴타입은 Optional로 비어있는 스트림에서 예외를 던지는것 대신 Optional 인스턴스를 반환

## 꼬리 재귀 적용하기

### 문제
- 재귀 프로세스를 실행하는데 필요한 메모리를 최소화 하고 싶음

### 해법
- 꼬리 재귀를 사용해 프로세스 알고리즘을 표현하고 해당 함수에 atilrec 기워드를 추가

### 설명
- 각각의 새로운 재귀 호출은 콜 스택에 프레임을 추가 하기에, 이런 프로세스는 사용 가능한 메모리를 초과 하게 될수도 있고, 재귀 프로세스가 스택 크기 제한에 다다르게 되면 StackOverflowError 발생
- 꼬리 재귀로 알려진 접근법은 콜스택에 새 스택 프레임이 추가되지 않게 구현하는것으로, 꼬리 재귀의 구현을 위해 재귀 호출이 연산의 마지막에 수행되도록 하면 됨

```kotlin
@JvmOverloads
tailrec fun factorial(n: Long, acc: BigInteger = BigInteger.ONE): BigInteger =
    when(n) {
        0L -> BigInteger.ONE
        1L -> acc
        else -> factorial(n - 1, acc * BigInteger.valueOf(n))
    }
```

- 위 예제에서 팩토리얼 함수는 팩토리얼 연산의 누적자 역할을 하는 2번째 인자가 필요. 마지막 평가식은 더 작은 수와 증가된 누적자를 이용해 자신 스스로를 호출 할수 있게됨
- tailrec키워드를 통해 컴파일러에게 해당 대귀 호출을 최적화 해야 한다고 알려줌
- tailrec 변경자를 적용할 수 있는 함수의 요건은 다음과 같음
    * 해당 함수는 반드시 수행하는 연산의 마지막 연산으로 자신을 호출 해야함
    * try/catch/finally 블록 안에서는 tailrec 사용 불가
    * 오직 JVM 백엔드에만 꼬리 재귀가 지원