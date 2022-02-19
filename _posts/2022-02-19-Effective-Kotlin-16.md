---
layout: post
title: "Effective Kotlin - 아이템16: 프로퍼티는 동작이 아니라 상태를 나타내야 한다"
description: 프로퍼티는 동작이 아니라 상태를 나타내야 한다
date: 2022-02-19 16:26:00 +09:00
categories: EffectiveKotlin Study
---


# 가독성

## 아이템 16 : 프로퍼티는 동작이 아니라 상태를 나타내야 한다
- 코틀린의 프로퍼티는 자바의 필드와 비슷해보이지만 완전히 다른 개념. 둘다 데이터를 저장한다는 점은 같으나 프로퍼티에는 더 많은 기능이 있음

```kotlin
// 코틀린 프로퍼티
var name: String? = null

// 자바의 필드
String name = null;

// 프로퍼티는 사용자 정의 getter/setter를 가짐
var name: String? = null
    get() = field?.toUpperCase() // field는 프로퍼티의 데이터를 저장해두는 backing field에 대한 레퍼런스. getter/setter의 디폴트 구현에 사용되므로 따로 만들지 않아도 디폴트로 생성
    set(value) {
        if(!value.isNullOrBlank()) {
            field = value
        }
    }

// val을 사용해 읽기 전용 프로퍼티를 만들면 field는 만들어지지 않음
val fullName: String
    get() = "$name $surname"
```

- var를 사용해 만든 읽고 쓸 수 있는 프로퍼티는 파생 프로퍼티라 부르며 getter/setter를 정의 할 수 있음
- 코틀린의 모든 프로퍼티는 디폴트로 캡슐화 되어 있음. 예를들어 자바 표준 라이브러리 Date를 사용하다가 제거해야한다고 할떄 다음과 같이 바꾸면 됨

```kotlin
var date: Date
    get() = Date(millis)
    set(value) {
        millis = value.time
    }
```

- 데이터를 millis라는 별도의 프로퍼티로 옮끼고 이를 활용해서 date프로퍼티에 데이터를 저장하지 않고 warp/unwrap하도록 변경하면 됨
- 프로퍼티는 필드가 필요어없음. 프로퍼티는 개념적으로 접근자(getter/setter)를 나타내기 때문에 코틀린은 인터페이스에도 프로퍼티를 정의 할 수 있는것

```kotlin
interface Person {
    val name: String
}
```

- 이렇게 코드를 작성하면 getter를 가질 것이라는 것을 나타내므로, 다음과 같이 오버라이드가 가능함

```kotlin
open class Supercomputer {
    open val theAnswer: Long = 42
}

class AppleComputer: Supercomputer() {
    override val theAnswer: Long = 1_800_275_2273
}
```

- 마찬가지로 프로퍼티 위임도 가능함. 프로퍼티는 본질적으로 함수이기 때문에 확장 프로퍼티도 가능함

```kotlin
val db: Database by lazy { connectToDb() }

val Context.preferences: SharedPerferences
    get() = PerferenceManager
        .getDefaultSharedPreferences(this)
```

- 프로퍼티는 필드가 아니라 접근자를 나타냄. 프로퍼티를 함수 대신 사용할 수 있지만 그렇다고 완전히 대체해서 사용하는것은 좋지 않음

```kotlin
// 이런식으로 알고리즘의 동작을 나타내는것은 좋지 않다
val Tree<Int>.sum: Int
    get() = when(this) {
        is Leaf -> value
        is Node -> left.sum + right.sum
    }
```

- sum 프로퍼티는 모든 요소를 반복 처리 하므로 알고리즘의 동작을 나타낸다고 할 수 있음. 관습적으로 이런 getter에 이런 계산량이 많은 작업이 필요하다고 예상하지 않으므로 함수로 따로 구현하는것이 좋음

```kotlin
fun Tree<Int>.sum(): Int = when(this) {
    is Leaf -> value
    is Node -> left.sum() + right.sum()
}
```

- 원칙적으로 프로퍼티는 상태를 나타내거나 설정하기 위한 목적으로만 사용하는것이 좋고 다른 로직등을 포함하지 않아야 함
- 이 프로퍼티를 함수로 정의할 경우 접두사로 get/set을 붙일것인가에 대해서 고민해보고 그게 아니라면 그것을 프로퍼티로 만들면 안됨
- 프로퍼티 대신 함수로 사용하는것이 좋은 경우
    * 연산 비용이 높거나, 복잡도가 O(1)보다 큰 경우
    * 비즈니스 로직(애플리케이션의 동작)을 포함하는 경우
    * 결정적이지 않은 경우(같은 동작을 두번 연속 했는데 다른 값이 나온경우)
    * 변환의 경우(변환의 경우 관습적으로 함수를 사용)
    * getter에서 프로퍼티의 상태 변경이 일어나야 하는경우

- 반대로 상태를 추출/설정 할 떄는 프로퍼티를 사용 해야 함. 특별한 이유가 없다면 함수를 사용하면 안됨