---
layout: post
title: "Effective Kotlin - 아이템35: 복합 객체를 생성하기 위한 DSL을 정의 하라"
description: 복합 객체를 생성하기 위한 DSL을 정의 하라
date: 2022-03-05 22:00:00 +09:00
categories: EffectiveKotlin Study
---


# 객체 생성

## 아이템 35: 복합 객체를 생성하기 위한 DSL을 정의 하라

- 코틀린을 활용하면 DSL(Domain Specifin Language)을 직접 만들 수 있는데, DSL은 복잡한 객체, 계층 구조를 갖고있는 객체들을 정의할때 매우 유용. DSL을 이용 하면 보일러플레이트와 복잡성을 숨기면서 개발자의 의도를 명확히 표현 가능

```kotlin
// Kotin DSL로 표현한 HTML

body {
    div {
        a("https://kotlinlang.org") {
            target = ATarget.blank
            +"Main Site"
        }
    }
    +"Some Content"
}
```

- DSL은 자료 또는 설정을 표현할때도 활용 되는데, API 정의, 테스트 케이스 정의, Gradle 설정 정의도 가능함

```kotlin
// 테스트 정의 예시
class MyTest : StringSpec({
    "length should return size of string" {
        "hello".length shouldBe 5
    }
    "startWith should test for a prefix" {
        "world" shoudl starwith("wor")
    }
})
```

- DSL 내부에서도 코틀린이 제공하는것을 모두 활용 할 수 있으며, DSL은 type-safe이므로 여러가지 유용한 힌트를 사용 가능함

### 사용자 정의 DSL 만들기
- DSL을 만드는 방법을 이해 하려면 리시버를 사용하는 합수 타입에 대한 개념을 먼저 이해 해야함. 그 전에 함수 자료형 자체에 대한 개념을 알아야함

```kotlin
inline fun <T> Iterable<T>.filter(
    predicate: (T) -> Boolean // 함수 타입
): List<T> {
    val list = arrayListOf<T>()
    for (elem in this) {
        if(predicate(elem)) {
            list.add(elem)
        }
    }
    return list
}
```

- 함수 타입의 예시
    * () -> Unit: 아규먼트를 갖지 않고 Unit을 리턴
    * (Int) -> Unut: Int 아규먼트를 갖고 Unit을 리턴
    * (Int) -> Int: Int 아규먼트를 갖고 Int를 리턴
    * (Int, Int) -> Int: Int 2개를 가유먼트로 받고 Int를 리턴
    * (Int) -> () -> Unit: Int를 아규먼트로 받고 다른 함수를 리턴. 이때 다른 함수는 아규먼트로 아무것도 받지 않고 Unit을 리턴
    * (() -> Unit) -> Unit: 다른 함수를 아규먼트로 받고 Unit을 리턴. 이때 다른 함순느 아규먼트로 아무것도 받지 않고 Unit을 리턴

- 함수타입을 만드는 방법
    * 람다 표현식
    * 익명 함수
    * 함수 레퍼런스

```kotlin
fun plus(a: Int, b: Int) = a + b
// 프로퍼티 타입이 지정 되어 있어서, 람다 표현식과 익명 함수의 아규먼트 타입을 추론 가능
val plus1: (Int, Int)-> Int = { a, b -> a + b }
val plus2: (Int, Int)-> Int = fun(a,b) = a + b
val plus3: (Int, Int)-> Int = ::plus
// 아규먼트 타입을 지정해서 함수의 형태를 추론하게도 가능
val plus4 = { a: Int, b: Int -> a + b }
val plus5 = fun(a: Int, b: Int) = a + b
```

- 함수 타입은 함수를 나타내는 객체를 표현하는 타입으로, 익명 함수는 일반적인 함수처럼 보이지만 이름을 갖고 있지 않고 람다 표현식은 익명 함수를 짧게 작성하는 표기법
- 확장 함수는 다음과 같이 표현 가능

```kotlin
fun Int.myPlus(other: Int) = this + other
// 익명 확장 함수 => 리시버를 가진 함수 타입
val myPlus = fun Int.(other: Int) = this + other 
```

- 리시버를 가진 함수 타입은 일반 함수 타입과 비슷 하지만 파라미터 앞에 리시버 타입이 추가 되어 있으며 . 으로 구분 되어 있음

```koltin
val myPlus: Int.(Int)-> Int =
    fun Int.(other: Int) = this + other

// 람다 표현식 사용
val myPlus: Int.(Int)-> Int = { this + it }
```

- 리시버를 가진 익명 확장 함수와 람다 표현식은 다음과 같은 방법으로 호출
    * 일반적인 객체처럼 invoke 메서드 사용
    * 확장 함수가 아닌 함수처럼 사용
    * 일반적인 확장 함수처럼 사용

```kotlin
myPlus.invoke(1,2)
myPlus(1,2)
1.myPlus(2)
```

- 리시버를 가진 함수 타입의 중요한 특징은 this의 참조 대상을 변경할 수 있다는 것. this 는 apply 함수에서 리시버 객체의 메서드와 프로퍼티를 간단하게 참조 할 수 있게 함

```koltin
inlien fun<T> T.apply(block: T.() -> Unit):T {
    this.block()
    return this
}

class User(
    var name: String = ""
    var surname: String = ""
)

val user = User().apply {
    name = "Marchin" // 리시버 객체의 프로퍼티 참조
    surname = "Moskala"
}
```

- 리시버를 가진 함수 타입은 코틀린 DSL을 구성하는 가장 기본적인 블록. 이를 활용해 HTML 표를 구성하는 DSL은 다음과 같음

```kotlin
fun createTable(): TableDsl = table { // table 함수. 코드가 톱레벨이고 별도의 리시버가 없으므로 table 함수도 톱 레벨 이여야 함
    tr { // table 정의 내부에서만 사용 되어야 함. table 함수의 아규먼트는 tr 함수를 갖는 리시버를 가져야함
        for (i in 1..2) {
            td {
                +"This is column $i" // 문자열에 적용된 단항 + 연산자
            }
        }
    }
}

fun table(init: TableBuilder.() -> Unit) =
    TableBuilder().apply(init)

class TableBuilder {
    var trs = listOf<TrBuilder>()

    fun td(init: TrBuildler.() -> Unit) {
        trs = trs + TrBuilder().apply(init)
    }
}

class TrBuilder {
    var tds = listOf<TdBuilder>()

    fun td(init: TdBuilder.() -> Unit) {
        tds = tds + tdBuilder().apply(init)
    }
}

class TdBuilder {
    var text = ""

    operator fun String.unaryPlus() {
        text += this
    }
}
```

### 언제 사용 해야 할까

- DSL은 다음과 같은 것을 표현 하는 경우 유용함
    * 복잡한 자료 구조
    * 계층적인 구조
    * 거대한 양의 데이터

- 많이 사용되는 반복되는 코드가 있고(코드를 읽는 사람에게 크게 중요하지 않은 정보를 포함하는 반복되는 코드), 이를 간단하게 만들 수 있는 별도의 코틀린 기능이 없다면 DSL 사용을 고려 해보는것이 좋음