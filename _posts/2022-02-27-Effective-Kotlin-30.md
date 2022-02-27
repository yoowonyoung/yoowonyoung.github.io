---
layout: post
title: "Effective Kotlin - 아이템30: 요소의 가시성을 최소화 하라"
description: 요소의 가시성을 최소화 하라
date: 2022-02-27 21:09:00 +09:00
categories: EffectiveKotlin Study
---


# 추상화 설계

## 아이템 30: 요소의 가시성을 최소화 하라
- API를 설계할때는 가능한 간결한 API를 선호 하는데 그 이유는 다음과 같음. 
    * 작은 인터페이스는 배우기 쉽고 유지보수하기 쉬움
    * 변경을 가할때는 기존의 것을 숨기는것보다 새로운 것을 노출하는게 더 쉽기 때문. 처음에는 작은 API로 개발하도록 강제하는게 더 좋을때도 있음

- 클래스의 상태를 나타내는 프로퍼티를 외부에서 변경할 수 있다면 클래스는 자신의 상태를 보장할 수 없음. 그래서 setter만 private로 만드는 코드는 굉창히 많이 사용됨

```kotlin
class CounterSet<T>(
    private val innerSet: MutableSet<T> = setOf()
): MutableSet<T> by innerSet {
    var elementsAdded: Int = 0
        private set
        // ...
}
```

- 코틀린에서는 이처럼 구체 접근자의 가시성을 제한해서 모든 프로퍼티를 캡슐화 하는것이 좋음
- 가시성이 제한 될수록 클래스의 변경을 쉽게 추적할 수있으며 프로퍼티의 상태를 더 쉽게 이해 할 수 있음. 이는 동시성을 처리할때 중요

### 가시성 한정자 사용하기
- 내부적인 변경 없이 작은 인터페이스를 유지하고 싶다면 가시성을 제한하면 됨
- 클래스 멤버의 경우 public(디폴트), private, protected, internal, 톱 레벨에서는 public(디폴트), private, internal 을 사용해서 가시성을 제한 할 수 있음
- 모듈이 다른 모듈에 의해서 사용될 가능성이 있다면 internal을 사용해서 공개하고싶지 않은 요소를 숨겨야함
- 코틀린은 지역적으로만 사용되고있는 요소에는 private로 만드는것이 좋다는 컨벤션을 제공해줌
- 이러한 규칙은 데이터를 저장하도록 설계된 클래스에는 적용하지 않는게 좋음
- 한가지 큰 제한은 API를 상속할 때 오버라이드 해서 가시성을 제한할 수 없다는것