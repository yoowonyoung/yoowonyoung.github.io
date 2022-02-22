---
layout: post
title: "Effective Kotlin - 아이템23: 타입 파라미터의 섀도잉을 피하라"
description: 타입 파라미터의 섀도잉을 피하라
date: 2022-02-22 22:20:00 +09:00
categories: EffectiveKotlin Study
---


# 재사용성

## 아이템 23 : 타입 파라미터의 섀도잉을 피하라
- 프로퍼티와 파라미터가 같은 이름을 가져서 지역 파라미터가 외부 스코프에 있는 프로퍼티를 가리는것을 섀도잉 이라고 함. 매우 흔한 경우 이므로 경고도 발생하지 않음

```kotlin
class Forest(val name: String) {
    fun addTree(name: String) { // 섀도잉
        // ...
    }
}
```

- 섀도잉은 클래스 타입 파라미터와 함수 타입 파라미터 사이에서도 발생하는데, 이는 심각한 문제가 될 수 있으며 문제를 찾아내기도 힘듬

```kotlin
interface Tree
class Birch: Tree
class Spruce: Tree

class Forest<T: Tree> {
    fun<T: Tree> addTree(tree: T) {
        // ...
    }
}

val forest = Forest<Birch>()
forest.addTree(Brich())
forest.addTree(Spruce())
```

- 위의 코드에서 Forest와 addTree의 타입 파라미터가 독립적으로 동작하게 되는데, 코드만 봐서는 독립적으로 동작하다는 것을 알아내기 힘듬. addTree가 클래스 타입 파라미터인 T를 사용하는게 올바른것

```kotlin
class Forest<T: Tree> {
    fun addTree(tree: T) {
        // ...
    }
}

val forest = Forest<Birch>()
forest.addTree(Brich())
forest.addTree(Spruce()) // Error, type mismatch
```

- 독립적인 타입 파라미터를 의도 했다면 이름을 아예 다르게 하는게 좋음
- 다음과 같이 타입 파라미터로 다른 타입 파라미터에 제한을 줄 수도 있음

```kotlin
class Forest<T: Tree> {
    fun <ST: T> addTree(tree: ST) {
        // ...
    }
}
```


