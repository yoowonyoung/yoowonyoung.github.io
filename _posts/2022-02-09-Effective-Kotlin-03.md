---
layout: post
title: "Effective Kotlin - 아이템3: 최대한 플랫폼 타입을 사용하지 말라"
description: 최대한 플랫폼 타입을 사용하지 말라
date: 2022-02-09 20:53:00 +09:00
categories: EffectiveKotlin Study
---


# 안정성

## 아이템 3 : 최대한 플랫폼 타입을 사용하지 말라
- 코틀린의 Null-Safety 덕분에 코틀린에서는 NPE를 찾아보기 힘들어졌으나, Null-Safety 매커니즘이 없는 자바,C 등과 코틀린을 연결해서 사용할때는 문제가 발생 할 수 있음
- 자바에서 String 타입을 리턴하는 메서드가 있다고 가정할 때, 자바에서는 모든것이 Nullable이므로 최대한 안전하게 접근 한다면 이를 Nullable로 가정하고 다루어야함
- 어떤 메서드가 null을 리턴하지 않는다고 단정할수 있는경우 non-null 단정인 !!을 붙이면 됨
- nullable과 관련해서 자주 문제가 되는 부분은 자바의 제너릭인데, 자바 API에서 ```List<User>```를 리턴한다고 하면 리스트와 리스트 내부의 User 객체들이 Null이 아닌지 모두 확인 해야함

```kotlin

// Java
public class UserRepo {
    public List<User> getUsers() {
        //
    }
}

//Kotlin
val users: List<User> = UserRepo().users!!.filterNotNull()
```


- 함수가 ```List<List<User>>```를 리턴한다면 훨씬 더 복잡해질것

```kotlin
val users: List<List<User>> = UserRepo().groupedUsers!!.map { it!!.filterNotNull() }
```

- 리스트는 적어도 map과 filterNotNull등의 메서드를 제공 하지만, 다른 제너릭 타입이라면 Null 확인 자체가 정말 족잡한 일이 됨
- 코틀린은 자바등의 다른 프로그래밍 언어에서 넘어온 타입들을 특수하게 다루는데 이러한 타입을 플랫폼 타입(다른 프로그래밍 언어에서 전달되어 nullable인지 아닌지 알수 없는 타입)이라고 함
- 플랫폼 타입은 String! 처럼 이름 뒤에 ! 기호를 붙여서 표기. 이러한것이 코드에 직접적으로 나타나지는 않지만 다음과 같이 선택적으로 사용됨

```kotlin
// Java
public class UserRepo {
    public user getUser() {
        //
    }
}


// Kotlin
val repo = UserRepo()
val user1 = repo.user // User! 타입
val user2: User = repo.user // User 타입
val user3: User? = repo.user // User? 타입
```

- 이러한 코드로 인해 전에 언급했던 문제가 사라질 수 있으나, null이 아니라고 생각되는것이 null일 가능성이 있으므로 여전히 위험함. 그래서 플랫폼 타입을 사용할 때에는 항상 주의를 기울여야함
- 설계자가 명시적으로 어노테이션으로 표기 하거나 주석으로 달아두지 않는다면 언제든지 동작이 변경될 수 있고, 함수가 지금 당장은 null을 리턴하지 않아도 미래에는 변경될 수 있다는것을 염두에 둬야함
- 자바와 코틀린을 함께 사용하는 경우엔, 자바 코드를 직접 조작 가능하다면 ```@Nullable``` 과 ```@NotNull``` 어노테이션을 활용 해야함
- 이러한 문제를 해결하는 여러가지 어노테이션들(ex> JSR 305의 ```@ParametersAreNotnullByDefault```)이 있지만, 플랫폼 타입은 안전하지 않으니 최대한 빨리 제거 하는것이 좋음

```kotlin
// Java
public class JavaClass {
    public String getValue() {
        return null;
    }
}

// Kotlin
fun startedType() {
    val value: String = JavaClass().value
    println(value.length)
}

fun platformType() {
    val value = JavaClass().value
    println(value.length)
}
```

- 위의 두가지 모두 NPE가 발생하지만 그 위치가 다름
- startedType의 경우에는 자바에서 값을 가져오는 위치에서 NPE가 발생. 이 위치에서 오류가 발생 한다면 Null이 아니라고 에상을 했지만 null이 나온다는것을 굉장히 쉽게 알 수 있음
- platformType에서는 값을 활용할 때 NPE가 발생. 플랫폼 타입으로 지정된 변수는 nullable일수도 있고 아닐수도 있기 떄문에, 이러한 변수를 한두번은 안전하게 사용했다 하더라도 다른사람이 사용할떄는 NPE를 발생 시킬 가능성이 충분히 존재. 이러한 문제는 타입 검사기에서도 검출할수 없음
- 인터페이스에서 플랫폼 타입을 사용하는 경우도 문제가 발생함


### 정리
- 다른 프로그래밍 언어에서 와서 nullable 여부를 알 수 없는 타입을 플랫폼 타입이라고 하는데, 플랫퐅 타입을 사용하는 코드는 해당 부분만 위험할 뿐만 아니라, 이를 활용하는 다른 곳까지 영향을 줄 수 있음
- 연결되어있는 자바 생성자, 메서드, 필드에 nullable 여부를 지정하는 어노테이션을 활용하는것도 좋음

