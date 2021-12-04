---
layout: post
title: "Kotlin Cookbook - 1 & 2장: 코틀린 설치와 실행 & 코틀린 기초"
description: Kotlin Cookbook 2장
date: 2021-12-01 21:51:00 +09:00
categories: Kotlin Cookbook Study
---

# 코틀린 설치와 실행


# 코틀린 기초

## 코틀린에서 널 허용 타입 사용하기

### 문제
- 변수가 절대 Null 값을 갖지 못하게 하고 싶다

### 해법
- 물음표를 사용하지 않고 변수의 타입을 정의한다. 또 널 허용 타입은 안전 호출 연산자(?.)나 엘비스 연산자(?:)와 결합해서 사용한다

### 설명
- 코틀린의 가장 매력적인 기능중 하나는 가능한 모든 Null을 제거 한다는것

```kotlin
var name: String // Null 불가
var name: String? // Null 가능

class Person(val first: String, val middle: String?, val last:String)
```

