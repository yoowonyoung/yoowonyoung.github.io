---
layout: post
title: "Effective Kotlin - 아이템22: 일반적인 알고리즘을 구현할 때 제너릭을 사용 하라"
description: 일반적인 알고리즘을 구현할 때 제너릭을 사용 하라
date: 2022-02-21 21:34:00 +09:00
categories: EffectiveKotlin Study
---


# 재사용성

## 아이템 22 : 일반적인 알고리즘을 구현할 때 제너릭을 사용 하라
- 타입 아규먼트를 사용 하는 함수(타입 파라미터를 갖는 함수)를 제너릭 함수라고 하며 대표적인 예시로 stdlib의 filter가 있음

```kotlin
inline fun <T> Iterable<T>.filter(
    predicate: (T) -> Boolean
): List<T> {
    val destination = ArrayList<T>
    for(element in this) {
        if(predicate(element)) {
            destination.add(element)
        }
    }
    return destination
}
```

- 타입 파라미터는 컴파일러에 타입과 관련된 정보를 제공 하여 컴파일러가 타입을 조금이라도 더 정확하게 추측 할 수 있게 해주므로 프로그램이 안전해지고 개발자는 프로그래밍이 편해짐. 물론 컴파일 과정에서 타입 정보가 사라지긴 하지만 개발 중에는 특정 타입을 강제 할 수 있음

### 제너릭 제한
- 타입 파라미터의 중요한 기능중 하나는 구체적인 타입의 서브 타입만 사용하게 타입을 제한 하는것임

```kotlin
fun <T: Comparable<T>> Iterable<T>.sorted(): List<T> { // 슈퍼타입인 Comparable을 지정해서 제한
    /*...*/
}

fun <T,C :MutableCollection<in T>> Iterable<T>.toCollection(destination: C):C { // 슈퍼 타입인 MutableCollection을 지정해서 제한
    /*...*/
}

class ListAdapter<T: ItemAdapter>(/*...*/) { /*...*/ } // 슈퍼 타입인 ItemAdapter를 지정해서 제한
```

- 타입에 제한이 걸리므로, 내부에서는 해당 타입이 제공하는 메서드를 사용할 수 있음
- 많이 사용하는 제한으로는 Any(nullable이 아닌 타입을 나타냄)이 있고, 둘 이상의 제한도 걸 수 있음

```kotlin
inline fun <T, R : Any> Iterable<T>.mapNotNull(
    transform: (T) -> R?
): List<R> {
    return mapNotNullTo(ArrayList<R>(), transform)
}

fun <T: Animal> pet(animal: T) where T: GoodTempered {
    /*...*/
}
// 이렇게도 가능
fun <T> pet(animal: T) where T: Animal, T: GoodTempered {
    /*...*/
}
```



