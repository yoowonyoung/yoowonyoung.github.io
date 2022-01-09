---
layout: post
title: "Kotlin Cookbook - 8장: 대리자"
description: Kotlin Cookbook 8장
date: 2022-01-09 15:43:00 +09:00
categories: Kotlin Cookbook Study
---

# 대리자

## 대리자를 사용해서 합성 구현하기

### 문제
- 다른 클래스의 인스턴스가 포함된 클래스를 만들고, 그 클래스에 연산을 위임 하고 싶음

### 해법
- 연산을 위임할 메소드가 포함된 인터페이스를 만들고, 클래스에서 해당 인터페이스를 구현한 다음 by 키워드를 사용해 바깥쪽에 래퍼 클래스를 만듬

### 설명
- 코틀린에서 by 키워드는 포함된 객체에 있는 모든 public 함수를 이 객체를 담고있는 컨테이너를 통해 노출 가능
- 포함된 객체에 있는 모든 public 함수를 이 객체를 담고 있는 컨테이너를 통해 노출 하려면 포함된 객체의 메소드의 인터페이스를 생성 해야함

```kotlin
interface Dialable {
    fun dial(number String): String
}

class Phone: Dialable {
    override fun dial(number: String) =
        "Dialing $number..."
}

interface Snappable {
    fun takePicture(): String
}

class Camera : Snappable {
    override fun takePicture() = 
        "Taking picture..."
}

class SmartPhone(
    private val phone: Dialable = Phone()
    private val camera: Samppable = Camera()
) : Dialable by phone, Sanppable by camera


class SmartPhoneTest {
    private val smartPhone: SmartPhone = SmartPhone()

    @Test
    fun DialingDelegatesToInternalPhone() {
        assertEquals("Dialing 555-1234...", smartPhone.dial("555-1234"))
    }

    @Test
    fun TakingPictureDelegatesToInternalCamera() {
        assertEquals("Taking Pictures...",smartPhone.takePicture())
    }
}
```

- SmartPhone 클래스가 생성자에서 Phone과 Camera를 인스턴스화 하고 모든 public 함수를 Phone과 Camera 인스턴스에 by 키워드를 사용해 위임함
- 인자없이 SmartPhone 을 인스턴스화 해도 위임 함수가 호출됨
- 포함된 객체는 SmartPhone을 통해 노출된것이 아니라 오직 포함된 객체의 public 함수만 노출 됨
- 내부적으로 SmartPhone 클래스는 위임퇸 속성을 인터페이스 타입으로 정의하고, 인터페이스 타입에 상응하는 클래스 인스턴스는 생성자를 통해 제공. 그다음 대리자 메소드는 해당 하는 메소드를 필드에서 호출

## lazy 대리자 사용하기

### 문제
- 어떤 속성이 필요 할 때까지 해당 속성의 초기화를 지연 시키고 싶음

### 해법
- 코틀린 표준 라이브러리의 lazy 대리자를 사용

### 설명
- 코틀린은 어떤 속성의 획득자와 설정자가 대리자라고 불리는 다른 객체에 구현 되어 있다는 것을 암시 하기 위해서 by 키워드를 사용
- lazy 대리자는 사용하려면 처음 접근이 일어날 때 값을 계산하기 위해 만들어진 () -> T 형식의 초기화 람다를 제공 해야 함

```kotlin
fun <T> lazy(initializer: () -> T): Lazy<T> // 기본, 스스로 동기화

fun <T> lazy(
    mode: LazyThreadSafetyMode, initializer: () -> T // lazy 인스턴스가 다수의 스레드간에 초기화를 동기화 하는 방법
): Lazy<T>

funt <T> lazy(lock: Any?, initializer: () -> T): Lazy<T> // 동기화 락을 위해 제공된 객체를 사용
```

- lazy에 제공된 초기화 람다가 예외를 던지면 다음번에 접근할 때 해당 값의 초기화를 다시 시도

```kotlin
val ultimateAnswer: Int by lazy {
    println("computing the answer")
    42
}
```

- ultimateAnswer의 값을 lazy에 제공된 람다 식이 평가 되는 떄, 즉 해당 변수에 처음 접근할때까지 계산하지 않음
- 이 구현에서 lazy는 람다를 받고 해당 속성에 처음 접근할때 제공된 람다를 실행하는 ```Lazy<Int>``` 타입의 인스턴스를 리턴하는 함수

## 값이 널이 될 수 없게 만들기

### 문제
- 처음 접근이 일어나기 전에 값이 초기화 되지 않았다면 예외를 던지고 싶다

### 해법
- notNull 함수를 이용해 값이 설정되지 않았다면 예외를 던지는 대리자를 제공

### 설명
- 코틀린 클래스의 속성은 보통 생성시에 초기화 되지만, 속성 초기화를 지연시키는 한가지 방법은 속성에 처음 접근하기 전에 속성이 사용되면 예외를 던지는 대리자를 제공하눈 notNull 함수를 사용 하는것

```kotlin
var shouldNotBeNull: String by Delegates.notNull<String>()

@Test
fun uninitializedValueThrowsException() {
    assertThrows<IlligalStateExcetpion>{ shouldNotBeNull }
}
```

- 코틀린 표준 라이브러리의 Delegates의 notNull구현을 살펴보면 흥미로운 부분이 있음

```kotlin
object Delegates{
    fun <T : Any> notNull(): ReadWriteProperty<Any?, T> = NotNullVar()
}

private class NotNullVar<T : Any>() : ReadWriteProperty<Any?, T>{
    private var value: T? = null

    override fun getValue(thisRef: Any?, property: KProperty<*>): T {
        return value ?: throw IllegalStateException(
            "Property ${property.name} should be initialized before get."
        )
    }

    override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        this.value = value
    }
}
```

- object 키워드를 사용해서 Delegates의 싱글톤 인스턴스를 정의 했고, notNull은 그로 인해 자바의 static처럼 동작 함
- notNull의 팩토리 메소드는 ReadWriteProperty 인터페이스를 구현하는 private 클래스인 NotNullVar를 인스턴스화 함

## observable과 vetoable 대리자 사용하기

### 문제
- 속성의 변경을 가로채서 필요에 따라 변경을 거부 하고 싶다

### 해법
- 변경 감지에는 observable 함수를 사용하고, 변경의 적용 여부를 결정할 때는 vetoable 함수와 람다를 사용

### 설명
- 코틀린 표준 라이브러리에서 Delegates 객체의 observable 함수와 votable 함수는 사용하기 쉽다. 하지만 observable 함수와 vetoable 함수의 구현은 개발자가 대리자를 작성할때 참고할만한 좋은 패턴임

```kotlin
fun <T> obserable(
    initialValue: T,
    onChange: (property: KProperty<*>, oldValue: T, newValue: T) -> Unit
): ReadWritePoperty<Any?, T>

fun <T> vetoable(
    initialValue: T,
    onChange: (property: KProperty<*>, oldValue: T, newValue: T) -> Boolean
): ReadWritePoperty<Any?, T>
```

- 두 팩토리 함수 모두 T 타입의 초기화 값과 람다를 인자로 받고 ReadWriteProperty 인터페이스를 구현한 클래스의 인스턴스를 리턴

```kotlin
var watched: Int by Delegates.obserabvle(1) { prop, old, new -> 
    println("${prop.name} changed from $old to $new")    
}

var checked: Int by Delegates.votable(0) { prop, old, new ->
    println("Trying to change ${prop.name} from $old to $new")
    new >= 0
}
```

- watched의 변수 타입은 Int에 1로 초기화 됬고, 이 변수의 값이 변경 될때마다 이전 값과 새로운 값을 보여주는 메시지가 출력
- checked의 변수 타입은 Int에 0로 초기화 됬고, votable의 람다 인자는 새로운 값이 0보다 크거나 같을 때만 true를 리턴

## 대리자로서 Map 제공하기

### 문제
- 여러 값이 들어 있는 맵을 제공해 객체를 초기화 하고 싶음

### 해법
- 코틀린 맵에는 대리자가 되는데 필요한 getValue와 setValue 함수 구현이 있음

### 설명
- 객체 초기화에 필요한 값이 맵 안에 있다면 해당 클래스 속성을 자동으로 맵에 위임할 수 있다

```kotlin
data class Project(val map: MutableMap<String, Any?>) {
    val name: String by map
    var prioirty: Int by map
    var completed: Boolean by map
}

@Test
fun useMapDelegateForProject() {
    val project = Project(
        mutableMapOf(
            "name" to "Learn Kotlin",
            "priority" to 5,
            "completed" to ture
        )
    )

    assertAll(
        { assertEquals("Learn Kotlin", project.name) },
        { assertEquals(5, project.priority) },
        { assertTrue(project.completed) }
    )
}
```


- 이 코드가 동작하는 이유는 MutableMap에 ReadWriteProperty 대리자가 되는데 올바른 시그니처의 setValue와 getValue가 있기 때문


## 사용자 정의 대리자 만들기

### 문제
- 어떤 클래스의 속성이 다른 클래스의 획득자와 설정자를 사용하게끔 만들고 싶음

### 해법
- ReadOnlyProperty 또는 ReadWriteProperty를 구현하는 클래스를 생성함으로써 직접 속성 대리자를 작성

### 설명
- 대체로 클래스의 속성은 지원 필드와 함께 동작하지만 필수는 아니며, 대신 값을 획득 하거나 설정하는 동작을 객체에 위임 할 수 있음
- 사용자 정의 속성 대리자를 생성하려면 ReadOnlyProperty 또는 ReadWriteProperty 인터페이스에 존재하는 함수를 제공 해야 함

```kotlin
interface ReadOnlyProperty<in R, out T> {
    operater fun getValue(thisRef: R, property: KProperty<*>): T
}

interface ReadWriteProperty<in R, T> {
    operater fun getValue(thisRef: R, property: KProperty<*>): T
    operater fun serValue(thisRef: R, property: KProperty<*>, value: T): T

}
```

- 대리자를 만들려고 ReadOnlyProperty나 ReadWriteProperty인터페이스를 구현할 필요는 없으며 코드에서 보여주는 시그니쳐와 동일한 getValue와 setValue 함수 만으로도 충분함
- 코틀린 표준 문서의 속성 대리자 절에는 Delegate 클래스가 있는 간단한 대리자 예시가 있음

```kotlin
class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, delegating '${property.name}' to me!"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value to '${property.name}' in $thisRef")
    } 
}

class Example {
    var p : String by Delegate()
}

fun main() {
    val e = Example()
    println(e.p)
    e.p = "NEW"
}
```

- 대리자를 사용하기 위해서 위임할 클래스 또는 변수를 생성한다음에 해당 변수를 가져오거나 설정하면 됨

