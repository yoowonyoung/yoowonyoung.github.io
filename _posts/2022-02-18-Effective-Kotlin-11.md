---
layout: post
title: "Effective Kotlin - 아이템11: 가독성을 목표로 설계하라"
description: 가독성을 목표로 설계하라
date: 2022-02-18 22:34:00 +09:00
categories: EffectiveKotlin Study
---


# 가독성

## 아이템 11 : 가독성을 목표로 설계하라
- 개발자가 코드를 작성하는데에는 1분이 걸리지만, 이를 읽는 데에는 10분이 걸린다 - 로버트 마틴

### 인식 부하 감소

```kotlin
// 구현 A
if (person != null && person.isAdult) {
    view.showPerson(person)
} else {
    view.showError()
}

// 구현 B
person?.takeIf{ it.isAdult }
    ?.let(view::showPerson)
    ?: view.showError()
```

- 위 코드에서 B가 더 짧긴 하지만 읽고 이해하기 어려우므로 좋은 코드가 아님. B에서 사용하는 관용구도 코틀린에서 꽤 일반적으로 사용되지만 경험이 많은 코틀린 개발자만 쉽게 이해 할 수 있으므로 좋은것이 아님
- 또한 구현 A는 수정 하기도 쉬움. if블록에 추가 작업이 필요하다면 쉽게 추가 할 수 있지 구현 B는 그렇지 못하며 만약 else 블록 쪽을 수정하는 일이 생긴다면 엘비스 연산자의 오른쪽 부분이 하나 이상의 표현식을 가져야 하므로 함수를 추가로 사용 해야함

```kotlin
// 구현 A
if (person != null && person.isAdult) {
    view.showPerson(person)
    view.hideProgressWithSuccess()
} else {
    view.showError()
    view.hideProgress()
}

// 구현 B
person?.takeIf{ it.isAdult }
    ?.let {
        view.showPerson(it)
        view.hideProgressWithSuccess()
    }
    ?: run {
        view.showError()
        view.hideProgress()
    }
```

- 이처럼 일반적이지 않고 창의적인 구조는 유연하지 않고 지원도 제대로 받지 못함
- 사실 그리고 구현A와 B는 실행 결과도 다름. let은 람다식의 결과를 리턴 하기 때문에 두번째 구현에서 showPerson이 null을 리턴하면 두번째 구현 때는 showError도 호출하게 됨


### 극단적이 되지 않음
- 극단적이 되어서는 안됨. 위의 예시에서 let으로 인해서 예상하지 못한 결과가 나왔다고 해서 let을 절대 쓰면 안된다는 말이 아님
- 균형을 맞추는것이 중요. 어떤 구조들이 어떤 복잡성을 가져오는지등을 파악하는것이 좋고, 두 구조를 조합해서 사용한다면 단순하게 개별적인 복합성의 합보다 훨씬 커진다는것에 주의

### 컨벤션

```kotlin
val abc = "A" { "B" } and "C"
print(abc) // ABC

// 이런 코드가 필요함
operator fun String.invoke(f: ()->String): String =
    this + f()

infix fun String.and(s: String) = this + s
```

- 위 코드는 저자가 생각하는 코틀린으로 할 수 있는 최악의 코드. 이 코드는 다음과 같은 수많은 규칙을 위반함
    * 연산자는 의미에 맞게 사용 해야함. invoke는 절대 이러한 형태로 사용하면 안됨
    * '람다를 마지막 아규먼트로 사용한다' 라는 컨벤션을 여기에 적용하면 코드가 많이 복잡해짐. invoke와 이러한 컨벤션을 적용 하는것은 신중 해야함
    * 현재 코드에서 and라는 함수 이름이 실제 내부 함수에서 이뤄지는 처리와 맞지 않음
    * 문자열을 결합하는 기능은 이미 언어에 내장 되어 있음. 이를 다시 만들 필요는 없음
