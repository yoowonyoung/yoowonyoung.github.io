---
layout: post
title: "Kotlin Cookbook - 3장: 코틀린 객체지향 프로그래밍"
description: Kotlin Cookbook 3장
date: 2021-12-19 19:31:00 +09:00
categories: Kotlin Cookbook Study
---

# 코틀린 객체지향 프로그래밍

## const와 val 차이 이해하기

### 문제
- 런타임 보다는 컴파일 타임에 변수가 상수임을 나타내야 한다

### 해법
- 컴파일 타임 상수에 const를 사용한다
- val은 변수에 한번 할당되면 변경이 불가능함을 알리지만, 이러한 할당은 실행 시간에 일어난다

### 설명
- 코틀린은 val을 통히 변경이 불가능한 상수임을 나타낸다(java의 final)
- 컴파일 타임 상수인 const는 반드시 객체나 companion object선언의 최상위 속성 또는 맨버여야 하며, 물자열 또는 기본 타입으 래퍼클래스이며, 사용자 정의 getter가 불가능하고, 컴파일 시점에 값을 사용할 수 있도록 main을 포함한 모든 함수의 바깥쪽에서 할당되어야 한다

```kotlin
class Task(val name: String, _priority: Int = DEFAULT_PRIORITY) {
    companion object {
        const val MIN_PRIORITY = 1
        const val MAX_PRIORITY = 5
        const val DEFAULT_PRIORITY = 3
    }

    var priority = validPriority(_priority) {
        set(value) {
            field = validPeriority(value)
        }
    }

    private fun validPriority(p: Int) =
        p.ceorceIn(MIN_PRIORITY, MAX_PRIORITY)
}
```

- val 은 키워드이지만 const는 private, inline등과 같은 변경자이므로 const가 val을 대체하는것이 아닌 같이 쓰이는것임에 주의

## 사용자 정의 getter / setter 생성

### 문제
- 값을 할당하거나 리턴하는 방법을 사용자 정의 하고 싶다

### 해법
- 코틀린의 클래스 속성에 get과 set을 추가한다

### 설명
- 코틀린은 기본적으로 public 이지만, 코틀린 클래스에서는 필드를 직접 선언할 수 없다

```kotlin
class Task(val name: String) {
    var priority = 3
}

val myTask = Task().apply{ priority = 4 }
```

- Task는 name과 priority라는 2가지 속성을 정의 하며, 속성 하나는 주 생성자 안에 선언 하고 다른 속성은 클래스의 최상위 멤버로 선언
- 이렇게 priority를 선언 하면 apply블록을 통해 값을 할당 할 수 있지만, 클래스를 인스턴스화 할때 priority에 값을 할당 할 수 없음

```kotlin
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]
```

- 속성을 정의하는 전체 문법은 위와 같으며 초기화 블록, getter/setter는 선택사항. 속성 타입이 초기값 또는 setter의 리턴 타입에 의해 추론 가능하다면 생략 가능. 생성자에서 선언한 속성에서는 타입 선언 필수(기본값이 있더라도)

```kotlin
val isLowPriority
    get() = priority < 3

val priority = 3
    set(value) {
        field = value.ceorceIn(1..5)
    }
```

- isLowPriority의 타입은 get의 리턴타입으로 추론되며(이 경우엔 불리언), 사용자 정의 setter는 속성에 값을 할당할때마다 사용됨
- 일반적으로는 속성에 지원 필드(backing field)가 필요하지만 코틀린은 자동으로 지원 필드를 생성. setter에서 field 식별자가 코틀린이 생성한 지원 필드를 참조하는데 사용되었고, field는 오직 사용자 정의 getter/setter에서만 사용 가능
- 속성에서 코틀린이 생성하는 기본 setter/getter를 사용하거나 사용자 정의 getter/setter에서 field속성을 통해 지원 필드를 참조하는 경우에는 코틀린이 지원 필드를 생성(isLowPriority는 지원 필드가 없음)

```kotlin
class Task(val name: String, _priority: Int = DEFAULT_PRIORITY) {
    companion object {
        const val MIN_PRIORITY = 1
        const val MAX_PRIORITY = 5
        const val DEFAULT_PRIORITY = 3
    }

    var priority = validPriority(_priority) {
        set(value) {
            field = validPeriority(value)
        }
    }

    private fun validPriority(p: Int) =
        p.ceorceIn(MIN_PRIORITY, MAX_PRIORITY)
}
```

- 파라미터 _priority는 속성이 아니라 생성자의 인자. 사용자 정의 setter에서 value는 임시 명칭임에 주의

## 데이터 클래스 정의하기

### 문제
- euqals, hashCode, toString이 완벽하게 갖춰진 엔티티를 나타내는 클래스를 생성하고 싶다

### 해법
- 클래스를 정의할때 data 키워드를 사용

### 설명
- 코틀린은 데이터를 담는 특정 클래스의 용도를 나타내기 위해 data키워드를 제공. 클래스 정의에 data를 추가하면 코틀린 컴파일러가 euqals, hashCode, toString, copy, component등의 함수를 생성

```kotlin
data class Produce (
    val name: String,
    val price: Double,
    val onSale: Boolean = false
)

@Test
fun checkEquivalence() {
    val p1 = Produce("baseball",10.0)
    val p2 = Produce("baseball",10.0)

    assertEquals(p1,p2)
    assertEquals(p1.hashCode(), p2.hashCode())
}

@Test
fun createSetToCHeckEqualsAndHashCOde() {
    val p1 = Produce("baseball", 10.0)
    val p2 = Produce(price = 10.0, onSale = false, name = "baseball")

    val products = setOf(p1,p2)
    aseertEquals(1, products.size)
}
```

- set에 p1, p2 모두 추가했지만 p1과 p2는 동일 하므로 오직 한 product만 추가됨
- copy는 원본과 같은 속성 값으로 시작해서 copy 함수에 제동된 속성값만 변경해 새로운 객체를 생성하는 인스턴스 메서드. copy는 얕은 복사를 수행 함에 주의

```kotlin
data class OrderItem(val product: Product, val quantity: Int)

@Test
fun dataCopyFunctionIsShallow() {
    val item1 = OrderItem(Product("baseball", 10.0), 5)
    val item2 = item1.copy()

    assertAll(
        { assertTrue(item1 == item2) },
        { assertFalse(item1 === item2) },
        { assertTrue(item1.product == item2.product) },
        { assertTrue(item1.product === item2.product) }
    )
}

@Test
fun destructureUsinComponentFunction() {
    val p = Product("baseball", 10.0)

    val (name, price, sale) = p
    aseertAll(
        { assertEquals(p.name, name) },
        { assertEquals(p.price `is`(closeTo(price,0.01))) },
        { assertFalse(sale) }
    )
}
```

- 두개의 OrderItem 객체가 동등하지만 레퍼런스 동등 연산자(===)가 false를 리턴하기때문에 2개는 서로 다른 객체임을 알 수 있음. 하지만 두 OrderItem인스턴스는 Product레퍼런스에 대해서 공유하고 있음
- 속성값을 리턴하는 component1, component2등도 있음
- 원한다면 equals, hashCode, toString, copy 뿐만 아니라 _componentN_ 함수를 재정의 가능. 그 이외의 함수도 추가 가능

## 지원 속성 기법

### 문제
- 클래스의 속성을 클라이언트에게 노출하고 싶지만, 해당 속성을 초기화 하거나 읽는 방법을 제어 해야함

### 해법
- 같은 타입의 속성을 하나 더 정의하고, 사용자 정의 getter/setter를 이용해 원하는 속성에 접근

```kotlin
class Customer(val name: String) {
    private var _messages: List<String>? = null

    val messages: List<String>
        get() {
            if(_messages == null) {
                _messages = loadMessages()
            }
            return messages!!
        }

    private fun loadMessages(): MutableList<String> =
        mutableListOf(
            "initial contact"
        ).also { println("load messages") }
}
```

- 생성 즉시 초기화 되지 않게 하기 위해 message 속성과 같은 타입의 널 허용 _messages를 추가. 사용자 정의 getter에서 message의 로딩 여부를 검사하여 로딩되지 않았다면 메시지를 불러옴
- _message는 private이기 떄문에 생성자 속성을 사용해 messages를 불러올 수 없음

```kotlin
class Customer(val name: String) {

    val messages: List<String> by lazy{ loadMessages() }

    private fun loadMessages(): MutableList<String> =
        mutableListOf(
            "initial contact"
        ).also { println("load messages") }
}
```

- 코틀린 내장 lazy 대리자 함수를 이용하면 더 쉬운 방식으로 위와 같은 지연 로딩을 사용 할 수 있음

## 연산자 오버로딩

### 문제
- 라이브러리에 정의된 클래스와 더불어 +,* 과 같이 연산자를 사용할 수 있는 클라이언트를 만들고 싶음

### 해법
- 코틀린의 연산자 오버로딩 매커니즘을 이용해 +,*등의 연산자와 연관된 함수를 구현

### 설명
- 코틀린에서 +,-,* 등의 연산자를 비롯해 많은 연산자들이 함수로 구현되어 있고, 해당 연산자를 사용하면 연산자와 연관된 함수에 처리를 위임

```kotlin
data class Point(val x: Int, val y: Int)

operator fun Point.unaryMinus() = Point(-x,-y)

val point = Point(10,20)

fun main() {
    println(-point)
}
```

- equals를 제외한 모든 연산자 함수를 재정의 할떄는 operator 키워드가 필수
- 자신이 작성하지 않은 클래스에 연산자 관련 함수를 추가하고 싶다면 확장 함수를 사용하면 됨

```kotlin
import org.apache.commons.match3.complex.Complex

operator fun Complex.plus(c: Complex) = this.add(c)
operator fun Complex.minus(c: Complex) = this.substact(c)
...
```

## 나중에 초기화 하기 위해 lateinit 사용하기

### 문제
- 생성자에 속성 초기화를 위한 정보가 충분하지 않으면 해덩 속성을 널 비허용 속성으로 만들고싶다

### 해법
- 속성에 lateinit을 사용한다

### 설명
- lateinit는 꼭 필요한 경우에만 사용. DI의 경우 유용하지만 일반적으로 가능하다면 지연 평가 같은 다른 대안을 고려 해보는게 좋음
- 널 비허용 속성으로 선언된 클래스 속성은 생성자에서 초기화 되어야 하지만, 가끔 속성에 할당할 값의 정보가 충분하지 않은 경우가 있는데 DI 프레임워크 이거나 유닛테스트의 설정 메소드 안에서 보통 발생. 이러한 경우 lateinit 속성을 사용
- lateinit는 클래스 몸체, 최상위 속성, 지역변수에서만 선언되고, 사용자 정의 getter/setter가 없는 var 속성에만 사용할 수 있음
- lateinit을 사용할 수 있는 속성 타입은 널 비허용 속성이여야 하며 lateinit를 추가하면 해당 변수가 처음 사용되기 전에 초기화 할 수 있고(apply등을 사용), 사용전 초기화에 실패하면 예외를 던짐

```kotlin
class LateInitDemo {
    lateinit var name: String

    fun initalizeName() {
        println(::name.isInitialized)
        name = "world"
        println(::name.isInitialized)
    }

    fun main() {
        LateInitDemo().initializeName()
    }
}
```
- 클래스 내부에서 속성 레퍼런스의 isInitialized를 사용하면 해당 속성의 초기화 여부를 확인 가능

## equals 재정의를 위한 안전 타입 변환, 레퍼런스 동등, 엘비스 사용하기

### 문제
- 논리적으로 동등한 인스턴스인지 확인 하도록 클래스의 equals 메소드를 잘 구현하고 싶음

### 해법
- 레퍼런스 동등 연산자(===), 안전 타입 변환 함수(as?), 엘비스 연산자(?:)를 다같이 사용

### 설명
- 코틀린에서 == 연산자는 자동으로 equals 함수를 호출
- equals 문법에서 equals 구현은 반사성, 대칭성, 추이성, 일관성이 있어야 하며 null도 적절하게 처리 할수 있어야 함

```kotlin
override fun equals(other: Any?): Boolean {
    if(this === other) return true
    val otherVersion = (other as? KotlinVersion) ?: return false
    return this.version == otherVersion.version
}
```

- 좋은 예시인 KotlinVersion 클래스의 euqals
    * === 으로 레퍼런스 동등성을 확인
    * 인자를 원하는 타입으로 변환하거나 널을 리턴하는 안전 타입 변환 연산자와, 안전 타입 변환 연산자가 널을 리턴하면 엘비스 연산자가 false를 리턴
    * 현재 인스턴스의 version속성이 other 객체의 version 속성과 동등 여부를 검사해 구현

## 싱글톤 생성하기

### 문제
- 싱글톤을 만들고 싶다

### 해법
- class 대신 Object를 사용

### 설명
- 코틀린에서 싱글톤 구현은 class 대신 object 키워드를 사용하기만 하면 됨. 이를 객체 선언이라고 함

```kotlin
object MySingleton {
    val myProperty = 3
    
    fun myFunction() = "hello"
}
```

- 싱글톤 안의 코드를 호출할때에는 자바의 static 멤버처럼 객체 이름에서 멤버에 접근 할 수 있다. 자바에서 코틀린 싱글톤에 접근 할때에는 코틀린이 생성한 INSTANCE 속성을 사용

```Kotlin
MySingleton.myFunction()
MySingleton.myProperty

MySingleton.INSTANCE.myFunction()
MySingleton.INSTANCE.myProperty
```

- 코틀린 object는 생성자를 가질수 없어서 인자와 함께 싱글톤을 인스턴스화 하려고 하면 문제가 발생

## Nothing에 관한 야단법석

### 문제
- Nothing 클래스를 사용법에 맞게 적절하게 사용하고 싶다

### 해법
- 절대 리턴하지 않는 함수에 Nothing 을 사용

### 설명
- 코틀린에는 Nothing이라는 이름의 클래스가 있음

```kotlin
public class Noting private constructor()
```

- private 생성자에 의해 클래스 밖에서 절대 인스턴스화 할 수 없고, 클래스 안에서 인스턴스화 하지 않음. 따라서 Noting의 인스턴스는 존재하지 않으며, 코틀린 공식 문서에서는 결코 존재할 수 없는 값을 나타내기 위해 Nothing을 사용 할 수 있다고 명시
- Nothing 클래스의 사용은 함수 몸체가 전적으로 예외를 전지는 코드로 구성된 상황에서 사용 가능

```kotlin
fun doNoting(): Nothing = throw Exception("Nothing at all")
```

- 리턴 타입을 반드시 구체적으로 명시해야하는데, 해당 메서드는 결코 리턴하지 않으므로 이 경우 리턴타입 Nothing이 유효
- 변수에 널을 할당할떄 구체적인 타입을 명시하지 않는 경우에도 Nothing이 사용

```kotnin
val x = null
```

- 이 경우 컴파일러에 의해 추론된 x의 타입은 Nothing? 임
- 흥미로운 사실은 코틀린에서 Nothing은 실제로 모다른 모든 타입의 하위 타입임

```kotlin
for(n in 1...10) {
    val x = when (n % 3) {
        0 -> "0"
        1 -> "1"
        2 -> "2"
        else -> throw Exception()
    }
    assertTrue(x is String)
}
```

- when식은 값을 리턴하기 떄문에 컴파일러는 하나도 빠트리지 않고 값을 요구함. 여기서 else 조건은 절대 발생하지 않지만 해당 경우에는 예외를 던지는게 타당. 예외가 발생하는 경우는 리턴 타입이 Noting이고 String은 String 타입 이므로 컴파일러는 x의 타입이 String임을알수 있음(String의 하위 타입이 Noting 이라서)