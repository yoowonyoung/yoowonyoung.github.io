---
layout: post
title: "Effective Kotlin - 아이템44: 멤버 확장 함수의 사용을 피하라"
description: 멤버 확장 함수의 사용을 피하라
date: 2022-03-09 23:00:00 +09:00
categories: EffectiveKotlin Study
---


# 클래스 설계

## 아이템 44: 멤버 확장 함수의 사용을 피하라

- 어떤 클래스에 대한 확장 함수를 정의할떄 이를 멤버로 추가 하는것은 좋지 않음. 확장함수는 첫번째 아규먼트로 리시버를 받는 단순한 일반 함수로 컴파일 되기 떄문

```kotlin
fun String.isPhoneNumber(): Boolean = 
    length == 7 && all { it.isDigit() }

// 컴파일 되면 이렇게 변함
fun isPhoneNumber('$this': String): Boolean =
    '$this'.length == 7 && '$this'.all { it.isDigit() }
```

- 이렇게 단순하게 변환되는 것이기 떄문에 확장 함수를 클래스 멤버로도 정의할 수 있고, 인터페이스 내부에도 정의할 수 있음

```kotlin
interface PhoneBook {
    fun String.isPhoneNumber(): Booelan
}

class Fizz: PhoneBook {
    override fun String.isPhoneNumber(): Boolean = 
        length == 7 && all { it.isDigit() }
}
```

- 이런 코드가 가능은 하지만 DSL을 만들때를 제외하면 이를 사용하지 않는것이 좋음. 특히 가시성 제한을 위해 확장함수를 멤버로 정의하는것은 굉장히 좋지 않음. 가시성이 제한되지 않기 떄문

```kotlin
// DO NOT TRY THIS
class PhoneBookIncorrect {
    fun String.isPhoneNumber() =
        length == 7 && all { it.isDigit() }
}

// 이렇게 사용 해야함
PhoneBookIncorrect().apply { "1234567890".test() }
```

- 확장 함수의 가시성을 제한하고 싶다면 멤버로 만들지 말고 가시성 한정자를 붙여주면 됨

```kotlin
class PhoneBookIncorrect {
    // ...
}

private fun String.isPhoneNumber() = 
    length == 7 && all { it.isDigit() }
```

- 멤버 확장을 피해야 하는 타당한 이유
    * 레퍼런스를 지원하지 않음
    * 암묵적 접근을 할 떄 두 리시버중 어떤 리시버가 선택될지 혼동이 됨
    * 확장함수가 외부에 있는 다른 클래스를 리시버로 받을떄 해당 함수가 어떤 동작을 하는지 명확하지 않음
    * 경험이 적은 개발자의 경우 확장함수를 보면 직관적이지 않거나, 심지어 보기만 해도 겁먹을 수 있음
