---
layout: post
title: "Effective Kotlin - 아이템21: 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라"
description: 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라
date: 2022-02-20 16:35:00 +09:00
categories: EffectiveKotlin Study
---


# 재사용성

## 아이템 21 : 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라
- 코틀린은 프로퍼티 위임을 제공 하는데, 프로퍼티 위임을 사용 하면 일반적인 프로퍼티의 행위를 추출 해서 재사용 할 수 있음
- 대표적인 예시로 지연 프로퍼티가 있는데, lazy 프로퍼티는 이후에 처음 사용하는 요청이 들어 올 때 초기화 되는 프로퍼티

```kotlin
val value by lazy { createValue() } // 지연 프로퍼티
```

- 프로퍼티 위임을 사용 하면 변화가 있을 때 이를 감지하는 observable 패턴을 stdlib의 observable 델리게이트를 기반으로 쉽게 만들 수 있음

```kotlin
var items: List<Item> by
    Delegates.observable(listOf()) { _,_,_ -> 
        notifyDataSetChanged()
}
```

- 프로퍼티 위임 매커니즘을 사용하면 뷰, 리소스 바인딩, 의존성 주입, 데이터 바인딩등의 패턴을 만들 수 있음

```kotlin
// 안드로이드에서 뷰 리소스 바인딩
private val button: Button by bindView(R.id.button)
private val doctor: Doctor by argExtra(DOCTOR_ARG)

// Koin의 의존성 주입
private val presenter: MainPresenter by inject()
private val vm: MainViewModel by viewModel()

// 데이터 바인딩
private val port by bindConfiguration("port")
private val token: String by preferences.bind(TOKEN_KEY)
```

- 간단한 프로퍼티 델리게이트의 예시. getter/setter에서 로그를 출력 하고 싶다고 가정

```kotlin
var token: String? = null
    get() {
        print("token returned value $field")
        return field
    }
    set(value) {
        print("token changed from $field to $value")
        field = value
    }

var attempts: Int = 0
    get() {
        print("attempts return value $field")
        reutrn field
    }
    set(value) {
        print("attempts changed from $field to $value")
        field = value
    }

```

- 두 프로퍼티는 타입이 다르지만 내부적으로는 거의 같은 처리. 또한 프로젝트에서 자주 반복될 패턴으로 보인다면 프로퍼티 위임을 활용하기 좋은 부분

```kotlin
var token: String? by LoggingProperty(null)
var attempts: Int by LoggingProperty(0)

private class LoggingProperty<T>(var value:T) {
    operator fun getValue(
        thisRef: Any?,
        prop: KProperty<*>
    ): T {
        print("${prop.name} returned value $value")
        return value
    }

    operator fun setValue(
        thisRef: Any?,
        prop: KProperty<*>,
        newValue: T
    ) {
        val name = prop.name
        print("name changed from $value to $newValue")
        value = newValue
    }
}

// token 은 다음과 같은 형태로 컴파일 될 것
@JvmField
private val 'token$delegate' = 
    LoggingProperty<String?>(null)
var token: String?
    get() = 'token$delegate'.getValue(this, ::token) // 컨텍스트와 프로퍼티 레퍼런스의 경계도 같이 사용 하는 형태
    set(value) {
        'token$delegate'.setValue(this, ::token, value)
    }
```

- 컨텍스트와 레퍼런스의 경계를 같이 사용 하기 떄문에 getValue/setValue가 여러개 있어도 컨텍스트에 따라서 상황에 맞는 적절한 메서드가 선택 되기 떄문에 문제가 되지 않음

```kotlin
class SwipeRefreshBinderDelegate(val id:Int) {
    private var cache: SwipeRefreshLayout? = null

    operator fun getValue(
        activity: Activity,
        prop: KProperty<*>
    ): SwipeRefreshLayout {
        return cache ?: activity
            .findViewById<SwipeRefreshLayout>(id)
            .also{ cache = it }
    }

    operator fun getValue(
        fragment: Fragment,
        prop: KProperty<*>
    ): SwipeRefreshLayout {
        return cache ?: fragment.view
            .findViewById<SwipeRefreshLayout>(id)
            .also { cache = it }
    }
}
```

- 객체 프로퍼티를 위임 하려면 val의 경우 getValue, var의 경우 getValue/setValue 연산이 필요한데, 이러한 연산은 확장 함수로도 만들 수 있음
- 코틀린 stdlib에서 다음과 같은 프로퍼티 델리게이트를 알아 두면 좋음
    * lazy
    * Delegates.observable
    * Delegates.vetoable
    * Delegates.notNull



