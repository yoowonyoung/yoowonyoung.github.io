---
layout: post
title: "Effective Kotlin - 아이템43: API의 필수적이지 않는 부분을 확장함수로 추출하라"
description: API의 필수적이지 않는 부분을 확장함수로 추출하라
date: 2022-03-09 22:22:00 +09:00
categories: EffectiveKotlin Study
---


# 클래스 설계

## 아이템 43: API의 필수적이지 않는 부분을 확장함수로 추출하라

- 클래스의 메서드를 정의할 떄는 메서드를 멤버로 정의할 것인지 아니면 확장함수로 정의할것인지 결정해야함

```kotlin
// 멤버로 메서드 정의하기
class Workshop(/*...*/) {
    fun makeEvent(date: DateTime): Event = //..
    val permlink
        get() = "/workshop/$name"
}

// 확장함수로 메서드 정의하기
class Workshop(/*..*/) {
    //..
}

fun Workshop.makeEvent(date: DateTime): Event = //..

val workshop.permalink
    get() = "/workshop/$name"
```

- 두가지 모두 거의 비슷. 호출하는 방법도 비슷하고 리플렉션으로 레퍼런싱 하는것도 비슷
- 두 방법 모두 각각의 장단이 있기 때문에 상황에 맞게 사용하는게 중요
- 멤버와 확장의 가장 큰 차이는 확장은 따로 가져와서 사용 해야 한다는 것. 그래서 일반적으로 확장은 다른 패키지에 위치함. 확장은 우리가 직접 멤버를 추가할 수 없는 경우 데이터와 행위를 분리하도록 설계된 프로젝트에서 사용. 필드가 있는 프로퍼티는 클래스에 있어야 하지만, 메서드는 클래스의 public API만 활용 한다면 어디에 위치해도 상관 없음
- Import 해서 사용한다는 특징 덕분에 확장은 같은 타임에 같은 이름으로 여러개를 만들 수도 있음. 따라서 여러 라이브러리에서 여러 메서드를 받을 수도 있고, 충돌이 발생하지 않는다는 장점이 있음
- 같은 이름으로 다른 동작을 한다는 확장이 있다는것은 위험할수도 있기 때문에, 위험 가능성이 있다면 그냥 멤버 함수로 만들어서 사용하는것이 좋음. 그렇게 하면 컴파일러가 항상 확장 대신 멤버 함수를 호출 할 것
- 확장은 virtual이 아니라는것도 차이점인데, 이로 인해 파생 클래스에서 오버라이드 할 수 없음. 상속을 목적으로 설계된 요소는 확장 함수로 만들면 안됨

```kotlin
open class C
class D: C()
fun C.foo() = "C"
fun D.foo() = "D"

fun main() {
    val d = D()
    print(d.foo()) // D
    val c: C = d
    print(c.foo()) // C
    print(D().foo()) // D
    print((D() as C).foo()) // C
}
```

- 이러한 차이는 확장함수가 '첫번쨰 아규먼트가 리시버로 들어가는 일반 함수'로 컴파일 되기 때문에 발생하는 결과
- 확장함수는 클래스가 아닌 타입에 정의하는것. 그래서 nullable 또는 구체적인 제너릭 타입에도 확장함수를 정의 가능

```kotlin
inline fun CharSequqnce?.isNullOrBlank(): Boolean {
    contract {
        returns(false) implies (this@isNullOrBlank != null)
    }

    return this == null || this.isBlank()
}

public fun Iterable<Int>.sum(): Int {
    var sum: Int = 0
    for(element in this) {
        sum += element
    }
    return sum
}
```

- 확장함수는 클래스 레퍼런스에서 멤버로 표시 되지 않기 떄문에 어토테이션 프로세서가 따로 처리하지 않음. 따라서 필수적이지 않은 요소를 확장함수로 추출하면 어노테이션 프로세스로부터 숨겨짐