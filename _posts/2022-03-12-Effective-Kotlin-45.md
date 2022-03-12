---
layout: post
title: "Effective Kotlin - 아이템45: 불필요한 객체 생성을 피하라"
description: 불필요한 객체 생성을 피하라
date: 2022-03-12 20:49:00 +09:00
categories: EffectiveKotlin Study
---


# 비용 줄이기

## 아이템 45: 불필요한 객체 생성을 피하라

- 객체 생성은 언제나 비용이 들어가고, 상황에 따라서는 큰 비용이 들어갈수도 있어서 불필요한 객체 생성은 최대한 줄이는것이 최적화의 관점에서 좋음

```kotlin
// JVM에서는 하나의 가상 머신에서 동일한 문자열을 처리하는 코드가 여러개 있다면 기존 문자열을 재사용
var str1 = "String"
var str2 = "String"
print(str1 == str2) // true
print(str2 === str2) // true

// 박스타입인 Integer, Long도 재사용됨 Integer는 -128~127 범위를 캐시
val i1: Int? = 1 // nullable 타입으로 int 대신 Integer 사용을 강제. 
val i2: Int? = 1
print(i1 == i2) // true
print(i1 === i2) // true

// 범위를 벗어나서 캐싱이 되지 않음
val j1: Int? = 1234
val j2: Int? = 1234
print(j1 == j2) // true
print(j1 === j2) //false
```

### 객체 생성 비용은 항상 클까?
- 객체를 wrap 할떄 발생하는 비용
    * 객체는 더 많은 용량을 차지. 기본 int는 4바이트지만, Integer는 16바이트 + 레퍼런스용 8바이트가 필요하므로 5배 이상의 공간 차지
    * 요소가 캡슐화 되어있다면 접근에 추가적인 함수 호출이 필요. 수많은 객체를 처리한다면 이 비용도 커짐
    * 객체는 생성되어야함. 생성되고 메모리에 할당되고 레퍼런스를 만드는 작업이 필요하기 때문에 적은 비용이지만 모이면 커짐

### 객체 선언
- 매순간 객체를 생성하지 않고 객체를 재사용 하는 방법은 객체 선언을 사용하는 싱글톤 패턴

### 캐시를 활용하는 팩토리 함수
- 팩토리 함수는 캐시를 가질 수 있고, 항상 같은 객체를 리턴하게도 만들 수 있음

```kotlin
// stdlib의 emptyList 함수. 항상 같은 객체가 리턴
fun <T> List<T> emptyList() {
    return EMPTY_LIST;
}
```

- 객체의 생성이 무겁거나, 동시에 여러 mutable 객체를 사용해야 하는 경우에는 객체 풀을 사용하는게 좋음
- parameterized 팩토리 메서드도 캐싱을 활용할 수 있는데, map등에 저장 하면 가능함

```kotlin
private val connetcions =
    mutableMapOf<String, Connection>()

fun getConnection(host: String) =
    connections.getOrPut(host) { createConnection(host) }
```

- 모든 순수 함수는 캐싱을 활용할 수 있음. 이를 메모제이션이라 부름
    * 단점으로 캐시를 위한 Map을 저장해야 하므로, 더 많은 메모리를 사용함
    * 메모리 문제로 크래시가 난다면 메모리를 해제 해줘야하는데, 이때 SoftReferenct를 활용하는게 좋음

- 캐시는 언제나 메모리와 성능의 트레이드 오프가 발생 하므로 캐시를 잘 설계하는것은 쉽지 않음

### 무거운 객체를 외부 스코프로 보내기

- 무거운 객체를 외부 스코프로 보내는 방법도 있는데, 컬렉션 처리에서 이뤄지는 무거운 연산을 컬렉션 처리 함수 내부에서 외부로 빼는것등을 말함

```kotlin
fun <T: Comparable<T>>, Iterable<T>.countMax(): Int =
    count { it == this.max() }

// 이렇게 수정 가능
fun <T: Comparable<T>>, Iterable<T>.countMax(): Int {
    val max = this.max() // 가독성 향상
    return count { it == max } // max를 한번만 찾으므로 코드 성능 향상
}
```

### 지연 초기화
- 무거운 클래스를 만들때 지연되게 만드는것이 좋을 때가 있음

```kotlin
class A {
    val b = B()
    val c = C()
    val d = D()
}

// 지연초기화로 A의 생성을 가볍게
class A {
    val b by lzay { B() }
    val c by lazy { C() }
    val d by lazy { D() }
}
```

- 클래스가 무거운 객체를 가졌지만 메서드의 호출은 빨라야 하는 경우 지연초기화는 적합하지 않음(백엔드 어플리케이션 등)

### 기본 자료형 사용하기
- JVM 컴파일러는 내부적으로 기본 자료형을 최대한 사용하지만 nullable 타입을 연산 하거나, 타입을 제너릭으로 사용 한다면 기본 자료형응 wrap 한 박스타입이 쓰이게 됨
- 박스타입 대신에 기본 자료형을 사용하도록 코드를 최적화 할 수있지만, 코틀린/JVM, 일부 코틀린/Native에서만 의미가 있고 숫자에 대한 작업이 여러번 반복될때만 의미가 있음
- 코드와 라이브러리 성능이 굉장히 중요한 부분에서만 이를 적용하는것이 좋음

