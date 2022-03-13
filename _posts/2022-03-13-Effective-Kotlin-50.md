---
layout: post
title: "Effective Kotlin - 아이템50: 컬렉션 처리 단계 수를 제한하라"
description: 컬렉션 처리 단계 수를 제한하라
date: 2022-03-13 14:56:00 +09:00
categories: EffectiveKotlin Study
---


# 효율적인 컬렉션 처리

## 아이템 50: 컬렉션 처리 단계 수를 제한하라

- 모든 컬렉션 처리 메서드는 비용이 많이 듬
    * 표준 컬렉션 처리는 내부적으로 요소들을 활용해 반복을 돌며, 추가적인 컬렉션을 만들어 사용
    * 시퀀스처리도 시퀀스 전체를 랩하는 객체가 만들어지며, 조작을 위해 또다른 추가적인 객체를 만들어냄

- 적절한 메서드를 활용해 컬렉션의 처리 단께 수를 적절하게 제한하는것이 좋음

```kotlin
class Student(val name: String?)

// 동작은 함
fun List<Student>.getNames(): List<String> = this
    .map{ it.name }
    .filter{ it != null }
    .map { it!! }

// 더 좋음
fun List<Student>.getNames(): List<String> = this
    .map{ it.name }
    .filterNotNull()

// 가장 좋음
fun List<Student>.getNames(): List<String> = this
    .mapNotNull { it.name }
```

- 컬렉션 처리와 관련해서 비효율적인 코드를 작성하는 이유는 대부분 어떤 메서드가 있는지 몰라서인 경우가 많음
- 대표적인 예시
    * ```filter{ it != null}.map{ it!!}``` -> ```fitlerNotNull()```
    * ```map{ <Transform> }.filterNotNull``` -> ```mapNotNull{ <Transform> }```
    * ```map{ <Transform> }.joinToString()``` -> ```joinToString{ <Transform> }```
    * ```filter{ <Predicate 1>}.filter{ <Predicate 2>}``` -> ```filter{ <Predicate 1> && <Predicate 2> }```
    * ```filter{ it is Type }.map{ it as Type }``` -> ```filterIsInstance<Type>()```
    * ```sortedBy{ <Key 2> }.sortedBy{ <Key 1> }``` -> ```sortedWith( compareBy({ <Key 1> }, { <Key 2> }))```
    * ```listOf(...).filterNotNull()``` -> ```listOfNotNull(...)```
    * ```withIndex().filter{ (index, elem) -> <Predicate using index> }.map{ it.value }``` -> ```filterIndexed { index, elem -> <Predicate using index>}```

