---
layout: post
title: "Effective Kotlin - 아이템38: 연산 또는 액션을 전달할 떄는 인터페이스 대신 함수 타입을 사용 하라"
description: 연산 또는 액션을 전달할 떄는 인터페이스 대신 함수 타입을 사용 하라
date: 2022-03-09 11:26:00 +09:00
categories: EffectiveKotlin Study
---


# 클래스 설계

## 아이템 38: 연산 또는 액션을 전달할 떄는 인터페이스 대신 함수 타입을 사용 하라

- 대부분의 프로그래밍 언어에서는 합수 타입이라는 개념이 없기 때문에 연산이나 액션을 전달할때 SAM(Single Abstract Method)라고 부르는 인터페이스를 전달 하는데, 이런 코드를 함수 타입을 사용하는 코드로 변경한다면 더 많은 자유를 얻을 수 있음

```kotlin
// 다음과 같이 파라미터를 전달 가능

// 람다 표현식 또는 익명 함수
setOnClickListener { /*..*/ }
setOnClickListener(fun(view)) { /*..*/ }

// 함수 레퍼런스 또는 제한된 함수 레퍼런스로 전달
setOnClickListener(::println)
setOnClickListener(this::showUsers)

// 선언된 함수 타입을 구현한 객체로 전달
class ClickListener: (View)-> Unit {
    override fun invoke(view: View) {
        // ..
    }
}
setOnClickListener(ClickListener())
```

- SAM의 장점을 아규먼트 이름에 있다고 말할수도 있는데, 타입 별칭을 사용하면 함수 타입도 이름을 붙일 수 있음

```kotlin
typealias OnClick = (View) -> Unit
```

- 람다 표현식을 사용할 떄는 아규먼트 분해를 사용할 수도 있는데, 이런점이 SAM보다 함수 타입을 사용하는게 훨씬 더 좋은 이유임

```kotlin
// 인터페이스 기반
class CalendarView {
    var listener: Listener? = null

    interface Listener {
        fun onDateClicked(date: Date)
        fun onPageChanged(date: Date)
    }
}

// 함수 타입 기반. onDateClicked/onPageChanged 가 독립되어 있기 때문에 각각을 변경 가능
class CalendarView {
    var onDateClicked: ((date:Date) -> Unit) ?= null
    var onPageChanged: ((date:Date) -> Unit) ?= null
}
```

### 언제 SAM을 사용 해야 할까
- 코틀린이 아닌 다른 언어에서 사용할 클래스를 설계 할 때
    * 자바에서는 인터페이스가 더 명확함. 함수타입으로 만들어진 클래스는 IDE의 지원등을 받을 수없음
    * 다른 언어에서 코틀린의 함수 타입을 사용 하려면 명시적으로 Unit을 리턴하는 함수가 필요함

