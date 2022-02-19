---
layout: post
title: "Effective Kotlin - 아이템15: 리시버를 명시적으로 참조해라"
description: 리시버를 명시적으로 참조해라
date: 2022-02-19 15:26:00 +09:00
categories: EffectiveKotlin Study
---


# 가독성

## 아이템 15 : 리시버를 명시적으로 참조해라
- 무언가를 더 자세하게 설명하기 위해 명시적으로 긴 코드를 사용할 때가 있는데, 함수와 프로퍼티를 지역 또는 톱레벨 변수가 아닌 다른 리시버로부터 가져온다는것을 나타낼 때가 있음

```kotlin
class User: Person() {
    private var beersDrunk: Int = 0

    fun drinkBeers(num: Int) {
        // ...
        this.beersDrunk += num // 클래스의 것임을 나타내는 this
        // ...
    }
}
```

### 여러개의 리시버
- 스코프 내부에 둘 이상의 리시버가 있는 경우(apply, with, run 등) 리시버를 명시적으로 나타내면 좋음

```kotlin
class Node(val name: String) {
    fun makeChild(childName: String) =
        create("$name.$childName")
            .apply{ print("Created ${name}") }

    fun create(name: String): Node? = Node(name)
}

fun main() {
    val node = Node("parent")
    node.makeChild("child")
}
```

- 위 코드에서 일반적으로 Created parent.child가 출력된다고 예상하지만 실제 결과는 Created parent가 출력될것임. 리시버를 붙여서 보면 확실해짐

```kotlin
class Node(val name: String) {
    fun makeChild(childName: String) =
        create("$name.$childName")
            .apply{ print("Created ${this?.name}") } // apply 내부에서 this의 타입이 Node? 라서 직접 사용 못함

    fun create(name: String): Node? = Node(name)
}

fun main() {
    val node = Node("parent")
    node.makeChild("child") // Created parent.child
}
```

- 사실 이것은 apply의 잘못된 사용법. also 와 파라미터 name을 사용 했으면 이런 문제가 일어나지 않음. also를 사용하면 명시적으로 리시버를 지정하게 됨. 일반적으로 nullable을 처리 할때는 also 또는 let을 사용함

```kotlin
class Node(val name: String) {
    fun makeChild(childName: String) =
        create("$name.$childName")
            .also{ print("Created ${it?.name}") }

    fun create(name: String): Node? = Node(name)
}
```

- 리시버가 명확하지 않다면 명시적으로 리시버를 적어서 이를 명확하게 해줘야함. 레이블 없이 리시버만 사용하면 가장 가까운 리시버를 의미 하므로, 외부에 있는 리시버를 사용 하려면 레이블을 사용 해야 함

```kotlin
class Node(val name: String) {
    fun makeChild(childName: String) =
        create("$name.$childName").apply{ 
            print("Created ${this?.name} in " +
                "${this@Node.name}") 
        }

    fun create(name: String): Node? = Node(name)
}

fun main() {
    val node = Node("parent")
    node.makeChild("child") // Created parent.child in parent
}
```

### DSL 마커
- 코틀린 DSL을 사용할 떄는 여러 리시버를 가진 요소들이 중첩되더라도 리시버를 명시적으로 붙이진 않음. 원래 DSL이 그렇게 사용 하도록 설계 되었기 때문
- DSL에서 외부의 함수를 사용하는 것이 위험한 경우가 있는데, 그런 경우 잘못된 사용을 막기 위해 DslMarker라는 메타 어노테이션을 사용함

```kotlin
@DslMarker
annotation class HtmlDsl

fun table(f: TableDsl.() -> Unit) { // ... }

@HtmlDsl
class DableDsl { // ... }

table {
    tr {
        td { +"Column 1" }
        td { +"Column 2" }
        tr { // 컴파일 에러
            td { +"value 1" }
            td { +"value 2" }
        }
        this@table.tr {
            td { +"value 1" }
            td { +"value 2" }
        }
    }
}
```

- DSL 마커는 가장 가까운 리시버만을 사용하게 하거나 명시적으로 외부 리시버를 사용하지 못하게 할 때 활용 할 수 있는 굉장히 중요한 매커니즘
- 짧게 적을수 있다는 이유 만으로 리시버를 제거하지 말아야함.