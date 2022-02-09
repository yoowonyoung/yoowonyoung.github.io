---
layout: post
title: "Effective Kotlin - 아이템4: inferred 타입으로 리턴하지 말라"
description: inferred 타입으로 리턴하지 말라
date: 2022-02-09 21:33:00 +09:00
categories: EffectiveKotlin Study
---


# 안정성

## 아이템 4 : inferred 타입으로 리턴하지 말라
- 코틀린의 타입 추론 기능은 몇가지 위험한 부분이 있음. 이러한 위험한 부분을 피하려면 할당때 inferred타입은 정확하게 오른쪽에 있는 피 연산자에 맞게 설정 된다는 것을 기억해야함. 절대로 슈퍼클래스 또는 인터페이스로 설정되지는 않음

```kotlin
open class Animal
class Zebra: Animal()

fun main() {
    var animal = Zebra()
    animal = Animal() // 타입 미스매치 오류
}
```

- 일반적인 경우에는 이러한 것이 문제가 되지 않지만, 직접 라이브러리를 조작할 수 없는 경우 이러한 문제를 간단하게 해결 할 수 없음

```kotlin
interface CarFactory {
    fun produce(): Car
}

val DEFAULT_CAR: Car = Fiat126P()
```

- 위와 같이 CarFatcory 인터페이스와 디폴트값이 존재 한다고 가정. DEFAULT_CAR는 Car로 명시적으로 지정 되어 있으므로 함수의 리턴 타입을 제거 했다고 해보면 아래와 같이 될것

```kotlin
interface CarFactory {
    fun produce() = DEFAULT_CAR
}
```

- 이후 다른 사람이 코드를 보다가 DEFAULT_CAR는 타입 추론에 의해 타입이 지정 될것이므로 Car를 명시적으로 지정하지 않아도 된다고 생각 할 수있음

```
val DEFAULT_CAR = Fiat126P()
```

- 이제 CarFactory에서는 Fiat126P이외의 다른 자동차는 생산하지 못하는 문제가 발생함. 인터페이스를 우리가 직접 사용할 수 있다면 쉽게 수정이 가능하지만 외부 API라면 쉽게 해결할 수 없음
- 리턴 타입은 API를 잘 모르는 사람에게 전달해 줄 수 있는 중요한 정보. 리턴 타입은 외부에서 확인 가능하도록 명시적으로 지정 해줘야함

### 정리
- 타입을 확실하게 지정해야 하는 경우 명시적으로 타입을 지정해야 한다는 원칙만 갖고 있으면 됨. 이는 굉장히 중요한 정보이므로 숨기지 않는것이 좋음
- 안전을 위해 외부 API를 만들 떄에는 반드시 타입을 지정하고, 이렇게 지정한 타입을 특별한 이유와 확실한 확신이 없이는 제거하면 안됨
