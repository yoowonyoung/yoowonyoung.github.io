---
layout: post
title: "Effective Kotlin - 아이템47: 인라인 클래스의 사용을 고려하라"
description: 인라인 클래스의 사용을 고려하라
date: 2022-03-13 11:50:00 +09:00
categories: EffectiveKotlin Study
---


# 비용 줄이기

## 아이템 47: 인라인 클래스의 사용을 고려하라

- 하나의 값을 보유하는 객체도 inline으로 만들 수 있음. 기본 생성자 프로퍼티가 하나인 클래스 앞에 inline을 붙이면 해당 객체를 사용하는 위치가 모두 해당 프로퍼티로 교체됨

```kotlin
inline class Name(private val value: String) {
    // ...
}
```

- 이러한 inline 클래스는 타입만 맞다면 그냥 값을 곧바로 집어넣는것도 허용됨

```kotlin
val name: Name = Name("Marchin")

// 컴파일후 이렇게 바뀜
val name: String = "Marchin"
```

- inline 클래스의 메서드는 모두 정적 메서드로 만들어짐

```kotlin
inline class Name(private val value: String) {
    // ...

    fun greet() {
        print("Hello")
    }
}

val name: Name = Name("Marchin")
name.greet()

// 컴파일후 이렇게 바뀜
val name: String = "Marchin"
Name.'greet-impl'(name)
```

- 인라인 클래스는 다른 자료형을 래핑해서 새로운 자료형을 만들 때 많이 사용. 이때 어떠한 오버헤드도 발생하지 않음. 측정 단위를 표현하거나 타입 오용으로 발생하는 문제를 막을때 유용

### 측정 단위를 표현 할 때

```kotlin
inline class Minutes(val minutes: Int) {
    fun toMillis(): Millis = Millis(minutes * 60 * 1000)
    // ...
}

inline class Millis(val milliseconds: Int){
    // ...
}

interface User {
    fun decideAboutTime(): Minutes
    fun wakeUp()
}

interface Timer {
    fun callAfter(timeMillis: Millis, callBack: () -> Unit)
}

fun setUpUserWkaeUpUser(user: User, timer: Timer){
    val time: Minutes = user.decideAboutTime()
    timer.callAfter(time.toMillis()) { // timer.callAfter(time) 은 type mismatch error
        user.wakeUp()
    }
}
```

- 이런식으로 타입을 잘못 사용해서 발생할 수 있는 문제에 대해 타입에 제한을 걸면서 인라인 클래스를 활용하면 좋음

### 타입 오용으로 발생하는 문제를 막을때
- SQL 데이터베이스는 일반적으로 ID를 사용해서 요소를 식별하는데, ID는 일반적으로숫자이므로 다음과 같은 경우 혼동이 발생 할 수 있음

```kotlin
@Entity(tableName = "grades")
class Grades(
    @Column(name = "studentId")
    val studentId: Int,
    @Column(name = "teacherId")
    val teacherId: Int,
    @Column(name = "schoolId")
    val schoolId: Int,
)

// Int 자료형의 값을 inline 클래스로 만들어서 문제 해결
inline class StudentId(val studentId: Int)
inline class TeacherId(val teacherId: Int)
inline class SchoolId(val studentId: Int)

@Entity(tableName = "grades")
class Grades(
    @Column(name = "studentId")
    val studentId: StudentId,
    @Column(name = "teacherId")
    val teacherId: TeacherId,
    @Column(name = "schoolId")
    val schoolId: SchoolId,
)
```

- inline 클래스를 활용하면 ID들을 사용하는것이 굉장히 안전해지며 컴파일 할 때 타입이 Int로 대체 되므로 코드를 바꾸어도 별도의 문제가 발생하지 않음

### 인라인 클래스와 인터페이스
- 인라인클래스도 인터페이스를 구현할 수 있음

```kotlin
interface TimeUnit {
    val millis: Long
}

inline class Minutess(val minutes: Long): TimeUnit {
    override val millis: Long get() = milliseconds
}

fun setUpTimer(time: TimeUnit) {
    val millis = time.millis
    // ...
}
```

- 하지만 이 코드는 클래스가 inline으로 동작하지 않음. 따라서 inline으로 만들었을때 얻을수 있는 장점이 하나도 없음

### typealias
- typealias로 타입에 새로운 이름을 붙여 줄 수 있음

```kotlin
typealias NewName = Int
val n: NewName = 10
```

- 이런 typealias는 길고 반복적으로 사용해야할 때 많이 유용. 하지만 typealias는 안전하지 않음

```kotlin
typealias Seconds = Int
typealias Millis = Int

fun getTime(): Millis = 10
fun setUpTimer(time: Seconds) {}

fun main() {
    val seconds: Seconds = 10
    val millis: Millis = seconds // 컴파일 오류가 발생하지 않음

    setUpTimer(getTime())
}
```

- 위 코드는 오히려 typealias를 사용하지 않는것이 오류를 쉽게 찾을 수 있을것. 이런 형태로 typealias를 쓰면 안되고 단위등을 표현하려면 파라미터 이름 또는 inline 클래스를 사용 해야함

