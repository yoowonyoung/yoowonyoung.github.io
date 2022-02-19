---
layout: post
title: "Effective Kotlin - 아이템13: Unit?을 리턴하지 말라"
description: Unit?을 리턴하지 말라
date: 2022-02-19 15:06:00 +09:00
categories: EffectiveKotlin Study
---


# 가독성

## 아이템 13 : Unit?을 리턴하지 말라

- Boolean이 true / false 를 갖는 것처럼 Unit?도 Unit / null을 가질 수 있어서 Boolean과 Unit?은 서로 바꿔서 사용 가능함

```kotlin
fun keyIsCorrect(key: String): Boolean = // ...
if(!keyIsCorrect(key)) return

// 이렇게도 사용 가능
fun verifyKey(key: String): Unit? = // ...
verifyKey(key) ?: return
```

- 일반적으로 Unit?을 사용한다면 저런 경우이지만, 코드를 작성할때는 멋잇게 보이지만 읽을때에는 그렇지 않음
- Boolean을 Unit?으로 표현하는것은 오해의 소지가 있으며 예측하기 어려운 오류를 만들 수 있음
- Unit?을 쉽게 읽을 수 있는 경우는 거의 없으며 Boolean을 사용하는 형태로 변경하는게 좋음


