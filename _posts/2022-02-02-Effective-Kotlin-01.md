---
layout: post
title: "Effective Kotlin - 아이템1: 가변성을 제한하라"
description: 가변성을 제한하라
date: 2022-02-02 19:15:00 +09:00
categories: EffectiveKotlin Study
---


# 안정성

## 아이템 1 : 가변성을 제한하라
- 코틀린의 모듈(클래스, 객체, 함수, 톱레벨 프로퍼티 등)중 일부는 상태를 가질 수 있음. 이런 요소가 상태를 갖는 경우 해당 요소의 동작은 사용 방법 뿐만 아니라 상태에도 의존하게 됨

```kotlin
class BankAccount {
    var balance = 0.0
        private set

    fun deposit(depositAmount: Double) {
        balance += depositAmount
    }

    @Throw(InsufficientFunds::class)
    fun withdraw(withdrawAmount: Double) {
        if(balance < withdrawAmount) {
            throw InsufficientFunds()
        }
        balance -= withdrawAmount
    }
}

class InsufficientFunds: Excepetion()

val account = BankAccount()
println(account.balance) // 0.0
account.deposit(100.0)
println(account.balance) // 100.0
account.withdraw(50.0)
println(account.balance) // 50.0
```

- 위의 코드에서 BankAccount에는 계좌에 돈이 얼마나 있는지 나타내는 상태가 있음. 이처럼 상태를 갖게 하는것은 양날의 검
- 상태를 가질 경우 가질 수 있는 문제점들
    + 프로그램을 이해하고 디버그 하기 힘들어짐. 상태를 갖는 부분들의 관계를 이해해야하고, 상태 변경이 많아지면 이를 추적하는게 힘들어짐. 이런 클래스는 이해하기도 어렵고, 이후에 코드 수정하기도 어려움. 클래스가 예상하지 못한 상황 또는 오류가 발생할 경우 큰 문제가 됨
    + 가변성이 있으면 코드의 실행을 추론하기 어려움. 시점에 따라서 값이 달라 질 수 있으므로, 현재 어떤값인지 알아야 코드의 실행을 예측 할 수 있으며, 한 시점에서 확인한 값이 계속 동일하게 유지된다고 확신 할 수 없음
    + 멀티 스레드 프로그램일떄 적절한 동기화가 필요함. 변경이 일어나는 모든 부분에서 충돌이 발생 할 수 있음
    + 모든 상태를 테스트 해야 하므로 변경이 많으면 많을수록 더 많은 조합을 테스트 해야 해서 테스트가 어려움
    + 상태 변경이 일어날 때 이러한 변경을 다른 부분에 알려야 하는 경우가 있음

- 변경 가능한 부분에 의한 일관성 문제, 복잡성 증가와 관련된 문제들도 흔함
- 가변성은 생각보다 단점이 많아 순수 함수형 언어에서는 이를 완전하게 제한함(하지만 이런 순수 함수형 언어는 가변성에 너무 많은 제한이 있어, 프로그램을 작성하기가 굉장히 어려움)

### 코틀린에서 가변성 제한하기
- 코틀린은 가변성을 제한할 수 있게 설계되어있어 immutable(불변)객체를 만들거나 프로퍼티를 변경할 수 없게 만드는것이 굉장히 쉬움
- 읽기 전용 프로퍼티(val), 가변 컬렉션과 읽기 전용 컬렉션 구분, 데이터 클래스의 copy등이 대표적인 예시
- 읽기 전용 프로퍼티
    + 코틀린은 val을 통해 읽기 전용 프로퍼티를 만들 수 있으며, 이렇게 선언된 프로퍼티는 일반적인 방법으로는 값이 변하지 않음

    ```kotlin
    val a = 10
    a = 20 // 오류
    ```

    + 읽기 전용 프로퍼티가 완전히 변경 불가능한것은 아님. 읽기 전용 프로퍼티가 mutable 객체를 담고 있다면, 내부적으로 변할 수있음

    ```kotlin
    val list = mutableListOf(1,2,3)
    list.add(4)
    ```

    + 읽기 전용 프로퍼티는 다른 프로퍼티를 활용하는 사용자 정의 getter로도 정의할 수 있음. 이렇게 var프로퍼티를 사용하는 val프로퍼티는 var프로퍼티가 변할때 같이 변할 수 있음

    ```kotlin
    var name: String = "Marcin"
    var surname: String = "Moskala"
    val fulname
        get() = "$name $surname"

    fun main() {
        println(fullName) // Marcin Moskala
        name = "Maja"
        println(fullName) // Maja Moskala
    }
    ```

    + 코틀린의 프로퍼티는 기본적으로 캡슐화 되어 있고ㅡ 추가적으로 사용자 정의 getter/setter를 가질 수 있어 API를 변경하거나 정의할 때 굉장히 유연함. var는 getter/setter모두 제공 되지만, val은 변경 불가능 하므로 getter만 제공
    + val은 var로 오버라이드 가능

    ```kotlin
    interface Element {
        var active: Boolean
    }

    calss ActualElement: Element {
        override var active: Boolean = false
    }
    ```

    + 읽기 전용 프로퍼티 val의 값은 변경될 수 있긴 하지만, 프로퍼티의 레퍼런스 자체를 변경할 수는 없으므로 동기화 문제등을 줄일 수 있음. 그래서 일반적으로는 var보다는 val을 사용
    + val은 읽기 전용 프로퍼티지만 변경 할수 없음(immutable)을 의미하는것은 아니라는것에 주의. 완전히 변경할 필요가 없다면 final을 사용하는것이 좋음
    + val은 정의 옆에 바로 상태가 적히므로 코드의 실행을 예측하는것이 쉽고, 스마트 캐스트 등의 추가적인 기능도 활용 가능

    ```kotlin
    val name: String? = "Marton"
    val surname: String = "Braun"

    val fullName: String?
        get() = name?.let { "$it $surname" } 

    val fullName2: String = name?.let { "$it $surname" }

    fun main() {
        if(fullName != null) { // getter로 정의 했으므로 스마트 캐스트할 수 없음. 값을 사용하는 시점의 name에 따라 다른 결과가 나올수 있기 떄문
            println(fullName.length) // 오류
        }
 
        if(fullName2 != null) { // 스마트 캐스트
            println(fullName2.length) 
        }
    }
    ```

- 가변 컬렉션과 읽기 전용 컬렉션 구분
    + 코틀린은 읽고 쓸 수 있는 컬렉션과 읽기 전용 컬렉션으로 구분됨
    + Iterable, Collection, Set, List 인터페이스는 읽기 전용. 변경을 위한 메서드를 가지지 않음
    + MutableIterable, MutableCollection, MutableSet, MutableList 인터페이스는 읽고 ㅅ쓸 수 있는 컬렉션. 읽기 전용 인터페이스를 상속받아서 변경을 위한 메서드를 추가한 형태
    + 읽기 전용 컬렉션이 내부의 값을 변경할 수 없다는 의미는 아님. 대부분의 경우에는 변경을 할 수 있지만, 읽기 전용 인터페이스가 이를 지원하지 않으므로 변경할 수 없음
    + 컬렉션을 진짜로 불변으로 만들지 않고 읽기 전용으로만 설계한것은 굉장히 중요함. 이로 인해서 더 많은 자유를 얻을 수 있기 때문
    + 코틀린이 내부적으로 immutable하지 않은 컬렉션을 외부적으로 immutable하게 보이게 만들어서 얻어지는 안정성이 있는데, 컬렉션 다운캐스팅은 이러한 안정성을 잃게 만드는 행위이므로 예측하지 못한 결과를 초래

    ```kotlin
    val list = listOf(1,2,3)

    // DO NOT TRY THIS
    if(list is MutableList) { // JVM에서 listOf는 Array.ArrayList가 리턴되는데, 이는 MutableList로 변경 가능하지만 Arrays.ArrayList는 이런 연산을 구현하고 있지 않아서 오류 발생
        list.add(4)
    }
    ```

    + 읽기 전용 컬렉션에서 mutable로 변경 해야 한다면 copy를 통해 새로운 mutable컬렉션을 만드는 toMutable~ 을 이용 해야함

    ```kotlin
    val list = lisOf(1,2,3)

    val mutableList = list.toMutableList()
    mutableList.add(4)
    ```

- 데이터 클래스의 copy
    + immutable 객체를 사용하면 다음과 같은 장점이 있음
        * 한번 정의된 상태가 유지 되므로 코드를 이해하기가 쉬움
        * immutable 객체는 공유 했을때도 충돌이 따로 이뤄지지 않으므로 병렬 처리를 안전하게 할 수 있음
        * immutable 객체에 대한 참조는 변경되지 않으므로 쉽게 캐시가 가능
        * immutable 객체는 방어적 복사본(defensive copy)를 만들 필요가 없음. 객체 복사시 깊은 복사를 따로 하지 않아도 됨
        * immutable 객체는 다른 객체(mutable/immutable)를 만들떄 활용하기 좋음
        * immutable 객체는 set 또는 map의 키로 사용할 수 있음

    + 지금까지 살펴본것처럼 immutable 객체는 변경에 대한 어려움이 있으므로, 자신의 일부를 수정한 새로운 객체를 만들어내는 메서드를 가져야함. 예를들어 Int는 immutable이지만, plus/minus 메서드로 자신을 수정핸 사로운 Int를 리턴 가능
    + 우리가 직접 만드는 immutable 객체도 비슷한 형태로 동작 해야 하지만, 모든 프로퍼티를 대상으로 이런 작업을 계속하는것은 귀찮은일임
    + data 한정자를 사용해서 클래스를 만들면 copy라는 이름의 메서드가 만들어지는데, copy는 모든 기본 생성자 프로퍼티가 같은 새로운 객체를 만들어낼 수 있음

    ```kotlin
    data class User(
        val name: String,
        val surname: String
    )

    var user = User("Maja","Markiewicz")
    user = user.copy(surname = "Moskala")
    print(user) // User(name=Maja, surname=Moskala)
    ```

    + 이와 같은 데이터 모델 클래스는 변경할 수 있다는 측면에서만 보면 Mutable 객체가 더 좋아 보이지만, immutable객체로 만드는것이 더 많은 장점을 가지므로 이렇게 만드는것이 좋음

### 다른 종류의 변경 가능 지점
- 변경 가능한 리스트를 만들어야 한다고 하면 mutable 컬렉션으로 만드는것과, var로 읽고 쓸 수 있는 프로퍼티로 만드는 방법이 있음. 두가지 모두 변경 가능하지만 방법이 다름

```kotlin
val list1: MutableList<Int> = mutableListOf()
var list2: List<Int> = listOf()

list1.add(1)
list2 = list2 + 1

// 이렇게도 가능
list1 += 1 // list1.plusAssign(1)
list2 += 1 // list2 = list2.plus(1)
```

- 두가기 모두 변경 가능 지점이 있지만 그 위치가 다름
- 첫번쨰 코드(MutableList)는 구체적인 리스트 구현 내부에 변경 가능 지점이 있음. 멀티 스레드 처리가 이뤄질경우 내부적으로 적절한 동기화가 되어있는지 알 수 없으므로 위험
- 두번쨰 코드(var immutableList)는 프로퍼티 자체가 변경 가능 지점. 멀티 스레드 처리의 안정성이 더 좋다고 할 수 있음(잘못 만들면 일부 요소가 손실될순있음)
- mutable리스트 대신 mutable프로퍼티를 사용하는 형태는 사용자 정의 setter(또는 이를 사용하는 delegate)를 활용해 변경을 추적할 수 있음

```kotlin
var names by Delegates.observable(listOf<String>()) { _, old, new ->  // Delegates.observable을 사용
    println("Names changed from $old to $new")
}

names += "Fabio"
// Names changed from [] to [Fabio]
names += "Bill"
// Names changed from [Fabio] to [Fabio, Bill]
```

- mutable 컬렉션도 이처럼 observe할 수 있지만 추가적인 구현이 필요함. 따라서 mutable프로퍼티에 읽기 전용 컬렉션을 넣어 사용하는것이 쉬움. 여러 객체를 변경하는 여러 메서드 대신 setter를 쓸 수 있고, 이를 private로도 만들수 있기 떄문

```kotlin
var announcements = lisOf<Announcement>()
    private set
```

- 최악의 방식은 프로퍼티와 컬렉션 모두 변경 가능한 지점으로 만드는것. 변경이 될 수 있는 두 지점 모두에 대한 동기화를 구현해야 하며, 모호성이 발생해서 += 을 사용할 수 없음

### 변경 가능 지점 노출하지 말기
- 상태를 나타내는 mutable 객체를 외부에 노출하는것은 굉장히 위험

```kotlin
data class User(val name: String)

class UserRepository {
    private val storedUsers: MutableMap<Int, String> =
        mutableMapOf()

    fun loadAll(): MutableMap<Int, String> {
        return storedUsers
    }
}

val userRepository = UserRepository()
val storedUsers = userRepository.loadAll()
storedUsers[4] = "Kirill" // 수정 가능!
```

- 이러한 문제를 해결하는 방법은 2가지 있는데, 첫번째는 리턴되는 Mutable객체를 복제 하는것. 이것을 방어적 복제(defensive copying)이라고 함. 이떄 data 한정자로 만들어지는 copy를 사용하면 좋음

```kotlin
class UserHolder {
    private val user: MutableUser()

    fun get(): MatableUser {
        return user.copy()
    }
}
```

- 가능하다면 무조건 가변성을 제한하는게 좋음. 컬렉션은 객체를 읽기 전용 슈퍼타입으로 업캐스트 하여 가변성을 제한 할 수 있음

```kotlin
data class User(val name: String)

class UserRepository {
    private val storedUsers: MutableMap<Int, String> =
        mutableMapOf()

    fun loadAll(): Map<Int, String> {
        return storedUsers
    }
}
```

### 정리
- var보다는 val을 사용하는게 좋음
- mutable 프로퍼티 보다는 immutable 프로퍼티를 사용하는게 좋음
- mutable 객체와 클래스 보다는 immutable 객체와 클래스를 사용하는게 좋음
- 변경이 필요한 대상을 만들어야 한다면 immutable 데이터 클래스로 만들고 copy를 활용하는게 좋음
- 컬렉션에 상태를 저장해야 한다면 mutable 컬렉션 보다는 읽기 전용 컬렉션을 사용하는게 좋음
- 변경 가능 지점을 적절히 설계하고, 불필요한 견경 가능 지점은 만들지 않는게 좋음
- mutable 객체를 외부에 노출하지 않는게 좋음
- 가끔은 효율성 때문에 immutable 객체보다 mutable 객체를 사용하는게 좋을 떄도 있음(3부에서 다룰 예정)
- immutable 객체를 사용할때는 멀티 스레드 환경에서 더 많은 주의를 기울여야함




