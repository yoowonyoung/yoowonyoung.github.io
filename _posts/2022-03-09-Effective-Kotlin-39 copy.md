---
layout: post
title: "Effective Kotlin - 아이템39: 태그 클래스보다는 클래스 계층을 사용하라"
description: 태그 클래스보다는 클래스 계층을 사용하라
date: 2022-03-09 12:35:00 +09:00
categories: EffectiveKotlin Study
---


# 클래스 설계

## 아이템 39: 태그 클래스보다는 클래스 계층을 사용하라

- 큰 규모의 프로젝트에서 상수 '모드'를 가진 클래스를 볼 수 있는데, 이러한 상수 모드를 태그라고 부르며 태그를 포함한 클래스를 태그 클래스라고 부름
- 태그 클래스는 다양양한 문제를 내포하고 있는데, 이러한 문제는 서로 다른 책임을 한 클래스에 태그로 구구분해서 넣는다는 것에서 시작함

```kotlin
class ValueMatcher<T> private constructor(
    private val value: T? = null,
    private val matcher: Matcher
) {
    fun match(value: T?) = when(matcher) {
        Matcher.EQUAL -> value == this.value
        Matcher.NOT_EQUAL -> value != this.value
        Matcher.LIST_EMPTY -> value is List<*> && value.isEmpty()
        Matcher.LIST_NOT_EMPTY -> value is List<*> ** value.isNotEmpty()
    }

    enum class Matcher {
        EQUAL,
        NOT_EQUAL,
        LIST_EMPTY,
        LIST_NOT_EMPTY
    }

    companion object {
        fun <T> equal(value: T) =
            ValueMAtcher<T>(value = value, matcher = Matcher.EQUAL)

        fun <T> notEqual(value: T) =
            ValueMAtcher<T>(value = value, matcher = Matcher.NOT_EQUAL)

        fun <T> emptyList(value: T) =
            ValueMAtcher<T>(matcher = Matcher.LIST_EMPTY)

        fun <T> notEmptyList(value: T) =
            ValueMAtcher<T>(matcher = Matcher.LIST_NOT_EMPTY)
    }
}
```

- 이러한 접근법에는 다음과 같은 단점이 있음
    * 한 클래스에 여러 모드를 처리하기 위한 보일러 플레이트가 추가 됨
    * 여러 목적으로 사용해야 하므로 프로퍼티가 일관적이지 않게 사용 될 수 있고, 더 많은 프로퍼티가 필요
    * 요소가 여러 목적을 가지고, 요소를 여러 방법으로 설정할 수 있는 경우에는 상태의 일관성과 정확성을 지키기 어려움
    * 팩토라 메서드를 사용해야 하는 경우가 많음

- 코틀린에서는 일반적으로 태그 클래스 보다 sealed 클래스를 사용 하는데, 한 클래스에 여러 모드를 만드는 방법 대신에 각각의 모드를 여거 클래스로 만들고 타입 시스템과 다형성을 활용 하는 것

```kotlin
sealed class ValueMatcher<T> {
    abstract fun match(value: T): Boolean

    class Equal<T>(val value: T): ValueMatcher<T>() {
        override fun match(value: T): Boolean =
            value == this.value
    }

    class NotEqual<T>(val value: T): ValueMatcher<T>() {
        override fun match(value: T): Boolean =
            value != this.value
    }

    class EmptyList<T>(): ValueMatcher<T>() {
        override fun match(value: T): Boolean =
            value is List<*> && value.isEmpty()
    }

    class NotEmptyList<T>(): ValueMatcher<T>() {
        override fun match(value: T): Boolean =
            value is List<*> && value.isNotEmpty()
    }
}
```

### sealed 한정자
- 반드시 sealed를 쓰지 않고 abstract를 써도 되지만, sealed는 외부 파일에서 서브 클래스를 만드는 행위 자체를 모두 제한 하므로 타입이 추가되지 않을것이라는게 보장이 됨. 이는 when을 이용할 때 else를 따로 만들 필요가 없다는 장점이 됨
- when은 모드를 구분해서 다른 처리를 만들때 굉장히 편리한데, 예를 들어 어떤 처리를 각각의 서브 클래스에 구현할 필요 없이 when을 활용하는 확장함수로 정의하면 한번에 구현 가능

```kotlin
funt <T> ValueMatcher<T>.reversed(): ValueMatcher<T> =
when(this) {
    is ValueMatcher.EmptyList -> ValueMatcher.NotEmptyList<T>()
    is ValueMatcher.NotEmptyList -> ValueMatcher.EmptyList<T>()
    is ValueMatcher.Equal -> ValueMatcher.NotEqual(value)
    is ValueMatcher.NotEqual -> ValueMatcher.Equal(value)
}
```

- abstract 클래스는 계층에 새로운 클래스를 추가할 수 있는 여지를 남기지만, sealed 클래스는 클래스의 서브 클래스가 제어됨

### 태그 클래스와 상태 패턴의 차이
- 상태 패턴은 객체 내부 상태가 변화 할 때, 객체의 동작이 변하는 소프트웨어 디자인 패턴으로 MVC, MVP, MVVM 아키텍처에서 많이 사용
- 상태 패턴을 사용한다면 서로 다른 상태를 나타내는 클래스 계층 구조를 만들게 되고, 현재 상태를 나타내기 위한 읽고 쓸 수 있는 프로퍼티도 만들게 됨

```kotlin
sealed class WorkoutState

class PrepareState(val exercise: Exercise): WorkoutState()

class ExerciseState(val exercise: Exercise): WorkoutState()

object DoneState: WorkoutState()

fun List<Exercise>.toState(): List<WorkoutState> =
    flatMap { exercise ->
        listOf(PrepareState(exercise), ExerciseState(exercise))
    } + DoneState

class WorkoutPresenter( /*..*/ ) {
    private var state: WorkoutState = states.first()
}
```

-  태그 클래스와 상태 패턴의 차이점
    * 상태는 더 많은 책임을 가진 클래스
    * 상태는 변경 할 수 있음

- 구체 상태는 객체를 활용해서 표현하는것이 일반적이고, 태그 클래스 보다는 sealed 클래스 계층으로 만들고, 이를 immutable 객체로 만들고 변경해야 할 때마다 state 프로퍼티를 변경함. 그리고 state 의 변화를 관찰 하는것
