---
layout: post
title: "Kotlin Cookbook - 1 & 2장: 코틀린 설치와 실행 & 코틀린 기초"
description: Kotlin Cookbook 2장
date: 2021-12-01 21:51:00 +09:00
categories: Kotlin Cookbook Study
---

# 코틀린 설치와 실행

## Gradle에 코틀린 플러그인 (코틀린 문법) 추가하기
- Gradle 5.0 이상에서는 빌드 파일을 설정할 때 사용할 수 있는 코틀린 DSL이 들어있음
- 코틀린 DSL은 모든 문자열에 큰따옴표를 사용하며, 괄호 사용이 필수이고, : 대신 = 을 사용한다는 차이가있음
- 코틀린 DSL로 작성 가능한 파이은 build.gradle.kts 파일과 settings.gradle.kts파일인데, settings.gradle.kts는 필수는 아니며, 그레이들 빌드 과정의 초가화 단계에서, 어떤 프로젝트 빌드 파일을 분석해야 하는지를 결정하는 순간에 처리

## Gradle을 이용해 코틀린 프로젝트 빌드하기
- dependency 블록에 ```implementation(kotlin("stdlib"))```추가

# 코틀린 기초

## 코틀린에서 널 허용 타입 사용하기

### 문제
- 변수가 절대 Null 값을 갖지 못하게 하고 싶다

### 해법
- 물음표를 사용하지 않고 변수의 타입을 정의한다. 또 널 허용 타입은 안전 호출 연산자(?.)나 엘비스 연산자(?:)와 결합해서 사용한다

### 설명
- 코틀린의 가장 매력적인 기능중 하나는 가능한 모든 Null을 제거 한다는것

```kotlin
var name: String // Null 불가
var name: String? // Null 가능

class Person(val first: String, val middle: String?, val last:String)

val p = Person(first = "North", middle = null, last = "West")
if(p.middle != null) {
    val middleNameLength = p.middle.length // 스마트 캐스트
}
```

- Null Safe의 예시. Nullable 변수를 식에서 사용하려고 할 때, 위와 깉이 p가 val로 선언되어 있고, null이 아님을 확인 했다면 String? 타입 대신 String 타입으로 스마트 캐스트가 일어남

```kotlin
var p = Person(first = "North", middle = null, last = "West")
if(p.middle != null) {
    val middleNameLength = p.middle!!.length
}
```

- p가 val이 아닌 var라면 위와 같이 됨. var로 선언 되었기 때문에 p가 정의된 시점과 p의 middle에 접근하는 시점 중간에 값이 변경되었을숟 ㅗ있다고 가정하기 때문
- 이를 회피하기 위해 널 아님 단언 연산자(!!)를 사용할수도 있는데, 이것은 나쁜 코드 냄새. 이로 인해 코틀린에서 NPE를 만날수도 있음
- 이런 상황에서는 안전 호출 연산자(?.)를 사용하는게 좋음. 안전 호출은 값이 널이면 null을 돌려줌

```kotlin
var p = Person(fist = "North", middle = null, last = "West")
val middleNameLength = p.middle?.length//Int?
```

- 이때 안전 호출 연산자만 쓰면 반환값이 nullable이므로, 엘비스 연산자(?:)를 병행해서 사용하는게 좋다

```kotlin
var p = Person(fist = "North", middle = null, last = "West")
val middleNameLength = p.middle?.length ?: 0//Int
```

- 엘비스 연산자는 자신의 왼쪽에 위치한 식의 값을 확인해 해당 값이 널이 아니면 그 값을 반환하고, 널이면 오른쪽에 위치한 값을 넘긴다
- 코틀린은 안전 타입 변환 연산자(as?)를 제공 하는데, 타입 변환이 올바르게 작동하지 않는 경우 ClassCastException 대신 null을 반환 하게 된다


## 자바에 널 허용성 지시자 추가하기

### 문제
- 코틀린 코드가 자바 코드와 상호 작용이 필요하고, 널 허용성 어노테이션을 강제하고싶다

### 해법
- 코틀린 코드에 JSR-305 널 허용성 어노테이션을 강제하려면 컴파일 타임 파라미터 Xjsr305=strict를 추가 하면 됨

### 설명
- 코틀린의 주요 기능중 하나는 컴파일 타임에 타입 시스템에 널 허용상을 강제 하는것. String 타입으로 변수를 선언하면 절대 널이 될 수 없음

```kotlin
tasks.withType<KotlinCompile> {
    kotlinOptions {
        jvmTarget = "1.8"
        freeCompilerArgs = listOf("-Xjsr305=strict")
    }
}
```

- 위와 같이 설정해주면 됨. 그 후 자바쪽 코드에서는 ```@Nonnull``` 어노테이션을 이용

## 자바를 위한 메소드 중복

### 문제
- 기본 파라미터를 가진 코틀린 함수가 있는데, 자바에서 각 파라미터의 값을 직접적으로 명시하지 않고 해당 코틀린 함수를 호출하고 싶음

### 해법
- ```@JvmOverloads``` 어노테이션을 해당 함수에 추가한다

### 설명

```kotlin
fun addProduct(name: String, price: Double = 0.0, desc: String? = null) =
    "Adding product wuth $name, ${desc ?: "None" }, and " +
        NumberFormat.getCurrencyInstance().format(price)

@Test
fun `check all overloads`() {
    assertAll("Overoad called form kotlin",
        { println(addProduct("Name", 5.0, "Desc")) },
        { println(addProduct("Name", 5.0)) },
        { println(addProduct("Name")) } )
}
```

- 위처럼 addProduct는 문자열 name이 필수이지만, description과 price는 기본값이 있다
- Optional이나 null 허용 속성은 함수 시그니처에 맨 마지막에 위치해야한다. 그렇게 해야만 위치 인자를 사용해서 호출할때 Optional이나 null 허용을 생략할수있다
- 자바는 메소드 기본인자를 지원하지 않기 때문에 자바에서 addProduct를 호출 하려면 모든 인자를 넣어줘야 하는데, 이를 피하기 위해서는 addProduct에 ```@JvmOverloads``` 어노테이션을 추가해야 한다
- ```@JvmOverloads```를 추가한 함수를 디컴파일 해보면 다음과 같이 오버로딩 되있는걸 볼수있다

```java
@JvmOverloads
@NotNull
public static final String addProduct(@NotNull String name, double price, @Nullable String desc) {
    Intrinsics.checkParameterIsNotNull(name,"name");
    //...
}

@JvmOverloads
@NotNull
public static final String addProduct(@NotNull String name, double price) {
    return addProduct$default(name, price, (String)null, 4, (Object)null);
}

@JvmOverloads
@NotNull
public static final String addProduct(@NotNull String name) {
    return addProduct$default(name, 0.0D, (String)null, 6, (Object)null);
}
```

- 생성된 클래스에는 제공된 인자와 함께 모든 인자를 요구하는 addProduct메서드를 호출하는 메서드가 추가되어있다
- 생성자도 동일하게 중복이 가능하지만 명시적으로 constructor 키워드를 사용해야 한다

```kotlin
data class Product @JvmOverloads constructor(
    val name: String,
    val price: Double = 0.0,
    val desc: String? = null
)
```

- 생성자 중복에서 주의해야 할것이 상속과 함께 사용하는 경우, 종종 ```@JvmOverloads```에 의해 생성된 생성자의 경우에 자신의 인자의 개수와 같은 인자의 개수를 갖는 부모 클래스의 생성자 호출이 일어나지 않는다. 호출되는 상위 클래스 생성자는 명시한 기본 인자와 함께 모든 파라미터를 요구하는 생성자

## 명시적으로 타입 변환하기

### 문제
- 코틀린은 자동으로 기본 타입을 더 넓은 타입으로, Int를 Long으로 승격하지 않는다

### 해법
- 더 작은 타입을 명시적으로 변환 하려면, toInt, toLong등 구체적인 변환 함수를 사용해야 한다

### 설명

```java
int myInt = 3;
long myLong = myInt;
```

- 위 코드는 자바에서는 지극히 정상적인 코드이지만 코틀린에서는 아니다
- 코틀린은 기본 타입을 직접적으로 제공하지 않는다. 개잘자가 코드를 작성할떄는 기본 타입이 아닌 클래스를 다룬다는 사실을 명심해야 한다

```kotlin
val intVar: Int = 3
val longVar: Long = intVar.toLong()
```

- 따라서 위와 같이 타입 변환 메소드를 사용해야 한다
- 타입 변환 메소드의 종류는 다음과 같다
    * toByte
    * toChar
    * toShort
    * toInt
    * toLong
    * toFloat
    * toDouble

- 다행이도 코틀린은 타입 변환을 투명하게 수행하는 연산자 중복 장점이 있어서, 다음 코드는 명시적 타입 변환이 필요하지 않다

```kotlin
val longSum = 3L + intVar
```

## 다른 기수로 출력하기

### 문제
- 십진법이 아닌 다른 기수를 사용하는 숫자를 출력하고 싶다

### 해법
- 올바른 기수를 위해 확장 함수 ```toString(radix: Int)```를 사용하자

### 설명
- 코틀린에서는 자바의 정적 메소드를 사용해서 Byte와 Short, Int, Long에 확장함수 ```toString(radix: Int)```가 만들어져 있다

```kotlin
42.toString(2) == "101010"
```

## 숫자를 거듭제곱 하기

### 문제
- 숫자를 거듭제곱 하고 싶지만, 코틀린에서는 미리 정의된 거듭제곱 연산자가 없다

### 해법
- Int와 Long에 정의돼 있는 코틀린 확장함수 pow에 위임하는 중위(infix) 함수를 정의 한다

### 설명
- 코틀린에서는 자바처럼 내장 거듭 제곱 연산자가 없다(자바에는 java.lang.Math에 pow가 있다)
- 코틀린에서는 기본 타입이 없기기때문에 Int같은 클래스 인스턴스가 자동으로 Long이나 Double으로 승격되지 않기 떄문에, Float와 Double에만 확장함수 pow가 있고, Int나 Long에는 없다
- 따라서 다음과 같이 Int와 Long에 확장함수를 정의 하거나, infix를 이용해서 정의해야 한다

```kotlin
fun Int.pow(x: Int) = toDouble().pow(x).toInt()
fun Long.pow(x: Long) = toDouble().pow(x).toLong()

infix fun Int.`**`(x: Int) = toDouble().pow(x).toInt()
infix fun Long.`**`(x: Long) = toDouble().pow(x).toLong()
```

- 백틱으로 감싼 두개의 별표 연산자가 맘에 들지 않는다면 exp같은 실질적인 이름으로 정의해도 된다

## 비트 시프트 연산자 사용하기

### 문제
- 비트 시프트 연산자를 사용하고 싶다

### 해법
- 코틀린에는 비트 시프트를 위한 shr,shl,ushr 같은 비트 중위 연산자가 있다

### 설명
- 코틀린은 확장함수로서 다음 시프트 연산자를 Int와 Long에 정의 해놓았다
    * shl: 부호 있는 왼쪽 시프트
    * shr: 부호 있는 오른쪽 시프트
    * ushr: 부호 없는 오른쪽 시프트

## 비트 불리언 연산자 사용하기

### 문제
- 비트 값에 마스크를 적용하고 싶다

### 해법
- 비트 불리언 연산을 위해 코틀린이 제공하는 and, or, xor, inv 비트 연산자를 활용한다

### 설명
- Int와 Long에 정의되어있는 시프트 연산자 뿐만 아니라 코틀린은 마스킹 연산자 and, or, xor, (not 대신) inv도 정의되어 있다

## to로 Pair 인스턴스 생성하기

### 문제
- Pair 클래스의 인스턴스를 생성하고 싶다

### 해법
- 직접 Pair 인스턴스를 생성하기 보다는 중위 to 함수를 사용한다

### 설명
- 코틀린은 Pair 인스턴스의 리스트로부터 맵을 생성하는 mapOf와 같은 맵 생성을 위한 최상위 함수 몇가지를 제공한다
- Pair는 first와 second라는 이름의 두개의 원소를 갖는 데이터 클래스다

```kotlin
data class Pari<out A, out B>: Serializable
```

- Pair클래스의 first와 second는 A와 B의 제너릭 값으로, 2개의 인자를 받는 생성자를 사용해 Pair 클래스를 생성할 수 있지만, to함수를 사용하는것이 더 일반적이다

```kotlin
public infix fun <A,B> A.to(that: B): Pair<A,B> = Pair(this, that)
```

- to함수는 제너릭 인자 B와 함께 모든 제너릭 타입 A에 추가된 확장함수이고, 이 확장함수는 A와 B를 위해 제공된 값을 결합한 Pair 인스턴스를 돌려준다
- Pair는 데이터 클래스로 개별 원소에 접근하려면 구조 분해를 통해서 접근해야 한다

```kotlin
val pair = "a" to 1
val (x,y) = pair

print(x)
print(y)
```
