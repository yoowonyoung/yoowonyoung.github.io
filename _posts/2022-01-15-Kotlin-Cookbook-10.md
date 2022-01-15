---
layout: post
title: "Kotlin Cookbook - 11장: 그 밖의 코틀린 기능"
description: Kotlin Cookbook 11장
date: 2022-01-15 21:33:00 +09:00
categories: Kotlin Cookbook Study
---

# 그 밖의 코틀린 기능

## 코틀린 버전 알아내기

### 문제
- 코드를 작성해 현재 사용중인 코틀린 버전을 알고 싶다

### 해법
- kotlinVersion 클래스의 companion object의 CURRENT 속성을 사용

### 설명

```kotlin
fun main(args: Array<String>) {
    println("${KotlinVersion.CURRENT}")
}
```

- 위와 같이 사용하면 된다
- KotlinVersion 클래스는 Comparable 인터페이스를 구현 하므로 <, > 같은 연산자를 사용할 수 있고 equals와 hashCode 함수도 구현 되어 있다


## 반복적으로 람다 실행하기

### 문제
- 주어진 람다 식을 여러번 실행 하고 싶다

### 해법
- 코틀린 내장 repeat 함수를 사용 한다

### 설명
- repeat 함수는 코틀린 표준 라이브러리에 들어 있다

```kotlin
@kotlin.internal.InlineOnly
public inline fun repeat(times: Int, action: (Int) -> Unit) {
    contract { callsInPlace(action) }

    for ( index in 0 until times) {
        action(index)
    }
}
```

- repeat는 반복할 횟루를 나타내는 정수(times)와 실행할 (Int) -> Unit 형식의 함수 두가지를 받는다

```kotlin
fun main(args: Array<String>) {
    repeat(5) {
        println("$it")
    }
}
```

## 완벽한 when 강제하기

### 문제
- 코틀린 컴파일러가 when 문에서 가능한 모든 절을 가지도록 강제 하고 싶다

### 해법
- 값을 리턴하는 제너릭 타입에 exhaustive라는 간단한 확장 속성을 추가하고 when 블록에 이를 연결 한다

### 설명
- when절은 if문과 마찬가지로 값을 반환한다. 하지만 when식이 값을 리턴하지 않는다면 코틀린은 when식이 완벽하길 요구하지 않는다

```kotlin
fun mod3(n: Int) {
    when( n % 3) {
        0 -> print("0")
        1 -> print("1")
        2 -> print("2")
    }
}

fun returnMod3(n: Int) = when(n % 3) {
    0 -> print("0")
    1 -> print("1")
    2 -> print("2")
    else -> print("opps") // else 없이는 컴파일 되지 않음
}
```

- 코틀린은 리턴을 else 블록을 강요하는것으로 해석하므로 이 점을 이용해 값을 자동으로 리턴하도록 when 블록을 만들면 모든 when 블록이 완벽해지도록 강제 할 수 있다

```kotlin
val <T> T.exhaustive: T
    get() = this

fun mod3(n: Int) {
    when( n % 3) {
        0 -> print("0")
        1 -> print("1")
        2 -> print("2")
        else -> print("opps") // else 없이는 컴파일 되지 않음
    }.exhaustive
}
```

- 확장속성 exhaustive는 모든 제너릭 타입 T에 대해서 객체 자신을 리턴하는 getter가 있고, 이제 이를 이용해서 exhaustive속성을 추가 하면 코틀린 컴파일러가 when 블록 끝에 exhasstive 속성 때문에 해당 객체가 완벽하길 요구하게 된다

## 정규 표현식과 함께 replace 함수 사용하기

### 문제
- 부분 문자열의 모든 인스턴스를 어떤 값으로 교체 하고 싶다

### 해법
- 문자열 인자 또는 정규식을 받도록 중복된 String의 replace 함수를 사용

### 설명
- String 클래스는 CharSequence 인터페이스를 구현한다. 즉 String 클래스에 2가지 버전의 replace 함수가 정의 되어 있음

```kotlin
fun String.replace(
    oldValue: String,
    newValue: String,
    ignoreCase: Boolean = false
): String

fun CharSequence.replace(
    regex: Regex,
    replacement: String
): String
```

- 각 replace함수는 일치하는 모든 문자열 또는 정규 표현식에 일치하는 모든 부분 문자열을 제공한 값으로 대체 한다

```kotlin
@Test
fun replaceTest() {
    assertAll(
        { assertEquals("one*two", "one.two".replace(".","*")) },
        { assertEquals("*******", "one.two".replace(".".toRex(),"*")) }
    )
}
``` 

- 첫번째 assertEquals는 마침표를 *로 전환하고, 두번째 assertEquals는 모든 문자를 *로 전환한다
- replace 함수는 첫번째 항목이 아니라 발생하는 모든 항목을 교체한다(java의 replaceAll). 또한 정규 표현식으로 해석되길 원한다면 toRex함수를 사용 해야 한다

## 바이너리 문자열로 변환하고 되돌리기

### 문제
- 숫자를 바이너리 문자열로 변환 하거나 바이너리 문자열을 정수로 파싱 하고 싶다

### 해법
- radix를 인자로 받는 toString 또는 toInt 을 사용 한다

### 설명
- StringKt 클래스에는 Int에 radix를 인자로 받는 인라인 확장함수 toString이 있다. 이를 반대로 수행하는 확장 함수도 있다

```kotlin
@Test
internal fun toBinary() {
    val str = 42.toString(radix = 2)
    assertThat(str, `is`("101010"))

    val num "101010".toInt(radix = 2)
    assertThat(num, `is`(42))
}
```

- toString이 생성한 문자열은 앞에 나오는 0을 잘라내는데, 이것을 원치 않는다면 padStart함수를 사용해 결과 문자열을 후처리 해야 한다

## 실행 가능한 클래스 만들기

### 문제
- 클래스에서 단일 함수를 간단하게 호출 하고 싶다

### 해법
- 함수를 호출할 클래스에서 invoke 함수를 재정의 한다

### 설명
- invoke 함수를 통해 클래스의 인스턴스를 함수처럼 호출 할 수 있다

```kotlin
class AstroRequest {
    companion object {
        private const val ASTRO_URL = "~~~"
    }

    operator fun invoke(): AstroResult {
        val responseString = URL(ASTRO_URL).readText()
        return Gson().fromJson(responseString, AstroResult::class.java)
    }
}

internal class AstroRequestTest {
    val reuqest = AstroRequest() // 클래스 인스턴스화

    @Test
    internal fun test() {
        val result = request() // 함수처럼 클래스 호출(invoke 호출)
        // test Code
    }
}
```

- invoke 연산자 함수를 제공하고 클래스 래퍼런스에 괄호를 추가하면 클래스 인스턴스를 바로 실행 할 수 있다

## 경과 시간 측정하기

### 문제
- 코드 블록이 실행되는데 걸린 시간을 알고 싶다

### 해법
- 코틀린 표준 라이브러리의 measureTimeMillies 또는 masureNanoTimes 함수 사용

### 설명

```kotlin
fun doubleIt(x: Int): Int {
    Thread.sleep(100L)
    return x*2
}

fun main() {
    var time = measureTimeMillis {
        IntStream.rangeClosed(1,6)
            .map{ doubleIt(it) }
            .sum()
    }
    print(time)
}

public inline fum measureTimeMillis(block: () -> Unit): Long {
    val start = System.currentTimeMillis()
    block()
    return System.currentTimeMillis() - start
}
```

- measureTimeMillis 함수는 람다를 인자로 받기 때문에 고차 함수이며 효율성을 이유로 인라인으로 되어 있음
- measureTimeMillis 함수의 구현은 그저 블록 인자의 전후로 작업을 System.currentTimeMillis 메소드에 위임. measureNanoTime의 구현도 비슷

## 스레드 시작 하기

### 문제
- 코드 블록을 동시적 스레드에서 실행 하고 싶다

### 해법
- kotlin.concurrent 패키지의 thread 함수를 사용

### 설명
- 코틀린은 스레드를 쉽게 생성하고 시작할수있는 thread 확장함수를 제공

```kotlin
fun thread(
    start: Boolean = true,
    isDaemon: Boolean = false,
    contextCalssLoader: ClassLoader? = null,
    name: String? = null,
    priority: Int = -1,
    block: () -> Unit
): Thread

(0..5).forEach { n ->
    val sleepTime = Random.nextLong(range = 0..1000L)
    thread {
        Thread.sleep(sleepTime)
        println("${THread.currentThread().name} for $n after $sleepTime")
    }
}
```

- start 인자의 기본값이 true이므로 다수의 스레드를 쉽게 생성하고 시작 할 수 있음. 각 스레드마다 start를 호출 할 필요가 없음
- isDaemon 파라미터를 통해 데몬 스레드를 생성 할수도 있는데, 모든 잔여 스레드가 데몬 스레드라면 애플리케이션은 종료된다
- thread 확장함수가 요구하는 코드 블록은 인자가 없고 Unit을 리턴하는 람다인데, 이는 Runnable 인터페이스 또는 Thread의 run 메소드 시그니처와 동일
- thread 확장함수는 자신 본문에서 생성한 스레드를 리턴하기 때문에 스레드의 join 메서드를 통해서 모든 스레드를 순차적으호 호출하게 만들 수 있다

```kotlin
(0..5).forEach { n ->
    val sleepTime = Random.nextLong(range = 0..1000L)
    thread {
        Thread.sleep(sleepTime)
        println("${THread.currentThread().name} for $n after $sleepTime")
    }.join() // 각 스레드는 자신 이전의 스레드에 Join을 수행
}
```

## TODO로 완성 강제하기

### 문제
- 개발자가 특정 함수나 테스트 구현을 끝마치게 하고 싶다

## 해법
- 함수 구현을 완성하지 않으면 예외를 던지는 TODO 함수를 사용

### 설명
- 코틀린 표준 라이브러리에는 TODO 라는 함수가 있는데, 다음과 같이 구현되고 사용한다

```kotlin
public inline fun TODO(reason: String): Noting =
    throw NotImplementedError("An operation is not implemented: $reason")

fun main() {
    TODO(reason = "non, really")
}
```

- 개발자의 의도를 선택적 인자 reason을 통해서 설명 가능

## Random의 무작위 동작 이해하기

### 문제
- 난수를 생성 하고싶다

### 해법
- Random 클래스에 있는 함수중 하나를 사용

### 설명
- 정수 범위의 난수를 생성하고 싶을때는 nextInt를 사용

```kotlin
open fun nextInt(): Int
open fun nextInt(until: Int): Int
open fun nextInt(from: Int, until: Int): Int
```

- Random 클래스의 구현은 다음과 같다

```kotlin
companion object Default: Random() {
    private val defaultRandom: Random = defaultPlatformRandom()

    override fun nextInt(): Int = defaultRandom.nextInt()
    override fun nextInt(until: Int): Int = defaultRandom.nextInt(until)
    override fun nextInt(from: Int, until: Int): Int = defaultRandom.nextInt(from, until)
    //...
}
```

- 이 companion object는 기본 구현을 가져오고 추상 클래스에 선언된 모든 메소드를 기본 구현에 위임하기 위해 재정의 한다. 즉 defaultPlatformRandom의 구현은 internal로 선언된다

## 함수 이름에 특수문자 사용하기

### 문제
- 함수 이름을 읽기 쉽게 작성 하고 싶다

### 해법
- 밑줄을 사용 하거나 함수 이름을 백틱으로 감싸면 된다. 단 테스트에서만 사용 하자

### 설명
- 코틀린은 함수 이름을 백틱으로 감싸는것이 가능 하다

```kotlin
fun `olny use backticks on test functions()`() {
    print("this works but not good idea")
}
```

- 함수를 백틱으로 감싸면 가독성을 위해 함수 이름에 공백이 가능하지만, 좋은 방법은 아니다

## 자바에게 예외 알리기

### 문제
- 코틀린 함수가 자바에서 checked exception으로 여겨지는 예외를 던지는 경우 자바에게 해당 예외가 checked exception임을 알려 주고 싶다

### 해법
- 함수 시그니처에 ```@Throws``` 어노테이션을 추가 한다

### 설명
- 코틀린의 모든 예외는 unchecked exception으로 컴파일러는 해당 예외를 처리할 것을 요구하지 않으며, 예외를 잡기위해 try/catch를 추가 할수 있지만 강제 사항은 아니다
- 자바에서 코틀린 함수를 호출 하면서, 호출한 코틀린 함수가 잠재적으로 자바에서 checked exception으로 여겨지는 예외를 던지고 자바에서 해당 예외를 잡고 싶다면 해당 사실을 알려 줘야 한다

```kotlin
@Throws(IOException::class)
fun problem() {
    throw IOException("Not found")
}

//java
public static void useTry() {
    try {
        problem();
    } catch (IOException e) {
        e.printStackTrace();
    }
}

public static void useThrows() throws IOException {
    problem();
}
```

- ```@Throws``` 어노테이션은 그저 자바/코틀린 통합을 위해서만 존재 한다