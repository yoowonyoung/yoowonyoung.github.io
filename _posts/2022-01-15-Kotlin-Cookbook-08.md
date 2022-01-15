---
layout: post
title: "Kotlin Cookbook - 9장: 테스트"
description: Kotlin Cookbook 9장
date: 2022-01-15 13:15:00 +09:00
categories: Kotlin Cookbook Study
---

# 테스트

## 테스트 클래스 수명주기설정하기

### 문제
- JUnit5의 테스트 수명주기를 기본값이 테스트 함수당 한번 대신 클래스 인스턴스당 한번으로 변경하고 싶다

### 해법
- ```@TestInstance``` 어노테이션 혹은 junit-platform.properites파일의 lifecycle.default 속성을설정

### 설명
- 기본적으로 JUnit4는 각 테스트 메소드마다 테스트 클래스의 새 인스턴스를 생성. 이러한 테스트 방식으로 각 테스트가 독립적이게 만듬. 하지만 초기화 코드가 매 테스트마다 반복되서 실행된다는 문제
- 자바에서는 static + ```@BeforeClass``` 어노테이션으로 이 문제를 해결하지만 코틀린은 static 키워드가 없으므로 ```companion object```를 써야함

```kotlin
class JUnit4ListTests {
    companion object {
        @JvmStatic
        private val strings = listOf("this", "is", "a", "list", "of", "strings")

        @BeforeClass
        @JvmStatic
        fun runBefore() {
            println("Before class")
        }

        @AftetClass
        @JvmStatic
        fun runAfter() {
            println("After class")
        }
    }
}
```

- ```companion object```는 strings 컬렉션이 클래스 전체에서 단 한번만 인스턴스화 되고 원소가 채워지게 하려고 사용. strings 리스트를 initialize 메소드 안에서 인스턴스화 하려면 strings의 속성을 ```lateinit var```로 바꿔야함
- JUnit5 에서는 ```@TestInstance``` 어노테이션으로 수명주기를 명시할수 있어 더 좋게 수정할수있다


```kotlin
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class JUnit5ListTests{
    private val strings = listOf("this","is","a","list","of","strings")

    private lateinit var modifiable: MutableList<Int>

    @BeforeEach
    fun setUp() {
        modifiable = mutableListOf(3,1,4,2,5)
    }

    @AfterEach
    fun finish() {
        println("After Each")
    }
}
```

- 테스트 인스턴스의 수명 주기를 PER_CLASS 로 설정하면 테스트 함수의 양과 상관 없이 테스트 인스턴스가 딱 하나만 생성
- 테스트 마다 속성 초기화가 필요한 경우 ```@BeforeEach```와 ```@AfterEach```를 사용할수있다
- junit-platform.properties 파일을 생성하고 ```junit.jupiter.testinstance.lifecycle.default = per_class``` 를 명시하는 방법도 있으나 모든 테스트에 적용되므로 주의 해야함


## 테스트에 데이터 클래스 사용하기

### 문제
- 코드를 부풀리지 않고 객체의 여러 속성을 체크하고 싶다

### 해법
- 원하는 모든 속성을 캡슐화 하는 데이터 클래스를 생성

### 설명
- 코틀린의 데이터 클래스는 equals, toString, hashCode, copy와 같은 구조 분해를 위한 component 메소드가 자동으로 포함. 데이터 클래스의 이런 특성은 테스트를 하기 위한 속성을 래핑하기에 적합함

```kotlin
data class Book(
    val isbn: String,
    val title: String,
    val author: String,
    val published: LocalDate
)

@Test
fun useJUnit5AssertAll() {
    val book = service.findBookById("1935182943")
    assertAll(
        { assertThat(book.isbn, `is`("1935182943")) }
        // .. 다른 테스트
    )
}
```

- 위와 같이 개별 테스트 + assertAll을 이용해서 테스트를 작성 할 수 있지만, 코틀린의 data클래스는 equals 메소드가 올바르게 구현되어 있으므로 다음과 같이 간소화 할 수 있다

```kotlin
internall fun useDataClass() {
    val book = service.findBookById("1935181943")
    val expected = Book(
        isbn = "1935181943",
        // 다른 속성들
    )

    assertThat(book, `is`(expected))
}
```

- 이제 assert는 코틀린 데이터 클래스가 정의한 euqals 메소드를 이용한다
- 컬렉션 인스턴스는 Hamcrest matcher가 제공하는 메소드를 이용해 컬렉션의 모든 원소를 확인 할수도 있다

```kotlin
@Test
internal fun checkAllElementsInList() {
    val found = service.findAllBooksById(
        "1934182943", "14911947020", "149197317X"
    )

    val exptected = arrayOf(
        Book("1934182943", "Making Java Groovy","Ken Kousen", LocalDate.perse("2013-09-30")),
        // 다른 책들
    )

    assertThat(found, arrayContainingInAnyOrder(*expected)) // arrayContainingInAnyOrder가 각 원소의 vararg리스트를 인자로 받기 떄문에 배열을 개별 항목으로 확장하기 위해서 * 사용
}
```

## 기본 인자와 함께 헬퍼 메소드 사용하기

### 문제
- 테스트 객체를 빠르게 생성 하고 싶다

### 해법
- copy 함수를 사용하거나, 필요하지 않은 생성자 기본 인자를 명시하지 말고 기본 인자를 가진 헬퍼 함수를 제공

### 설명
- 코틀린에서는 클래스의 생성자 인자에 기본값을 명시 할수도 있지만, 기본값을 생성하는 팩토리 함수를 추가 할수도 있다

```kotlin
fun createBook(
    isbn: String = "149197317X",
    title: String = "Morden java Recipes",
    author: String= "Ken Kousen",
    published: LocalDate = LocalDate.parse("2017-08-26")
) = Book(isbn, title, author, published)

val modern_java_recipes = createBook()
val making_java_groovy = createBooK(
    isbn = "1935182943",
    title = "Making Java Groovy",
    published = LocalDate.parse("2013-09-30")
)
```

- createBook 팩토리 함수 사용은 간편하다. 기본 인자는 오직 테스트 데이터를 생성하기 위해 사용 되엇기 떄문에 도메인 클래스 자체에 기본 인자를 추가할 필요돟 없다
- 똑같은 일을 하기 위해 data 클래스에서 제공하는 copy 함수를 사용 할수도 있지만, copy 함수를 광범위하게 사용하면 중첩 구조에서 가독성이 떨어진다
- 최상위 레벨의 유틸리티 클래스에 팩토리 함수를 위치 시키면 테스트에서 팩토리 함수를 재사용 할 수 있다

## 여러 데이터에서 JUnit5 테스트 반복하기

### 문제
- 데이터 값 세트를 제공해서 JUnit5 테스트를 실행 하고 싶다

### 해법
- JUnit5의 ```@ParameterizedTest```와 동적 테스트를 사용 한다

### 설명
- JUnit5에는 쉼표로 구분된 값(CSV)과 팩토리 메소드가 포함된 옵션과 함꼐 데이터 소스를 명시할 수 있는 파라미터화된 테스트가 있다

```kotlin
@JvmOverloads
tailrec fun fibonacci(n: Int, a: Int = 0, b : Int = 1): Int = 
    when (n) {
        0 -> a
        1 -> b
        else -> fibonacci(n-1,b, a+b)
    }

@ParameterizedTest
@CsvSource("1,1","2,1", "3,2",
    "4,3", "5,5", "6,8", "7,13",
    "8,21", "9,34", "10,55")
fun first10Fibo(n: Int, fib: Int) =
    assertThat(fibonacci(n), `is`(fib))
```

- ```@CsvSource``` 어노테이션은 테스트 함수를 위한 입력 데이터로서 문자열 리스트를 인자로 받음. 각각의 문자열은 해당 함수가 필요한 모든 인자를 제공하고 이때 인자는 문자열에서 쉼표로 구분
- JUnit5에서 팩토리 메소드를 통해 테스트 데이터를 생성 할수도 있는데, ```@TestInstance(Lifecycle.PER_CLASS)```가 명시되있지 않았다면 테스트 클래스의팩토리 메소드는 반드시 static 이여야 한다
- ```@TestInstance(Lifecycle.PER_CLASS)```를 사용하였다면 테스트 데이터를 생성하는 함수를 추가하고 ```@MethodSource```를 해당 함수에 추가해 참조할 수 있도록 한다

```kotlin
private fun fibNumbers() = listOf(
    Arguments.of(1,1), Arguments.of(2,1),
    Arguments.of(3,2), Arguments.of(4,3),
    Arguments.of(5,5), Arguments.of(6,8)
)

@ParameterizedTest(name = "fibonacci({0}) == {1}")
@MethodSource("fibNumbers")
fun first10Fib(n: Int, fib: Int) =
    assertThat(fibonacci(n), `is`(fib))
```

- JUnit은 두 입력 인자를 결합할 수 있는 of 팩토리 메소드를 가진 Arguments 클래스를 제공

## 파라미터화된 테스트에 data 클래스 사용하기

### 문제
- 파라미터화된 테스트를 좀 더 쉽게 읽는 테스트 결과를 생성 하고 싶다

### 해법
- 입력값과 예상값을 감싸는 data 클래스를 만들고, 만든 data 클래스 기반의 테스트 데이터를 생성하는 함수를 테스트 메소드로서 사용한다

### 설명
- 파라미터화된 테스트에서 각각의 테스트는 각각의 계산 값이 예상 값과 같은지 비교하는 단언에 이르게 되므로, 입력과 예상 출력을 담는 데이터 클래스를 정의 할 수 있다

```kotlin
data class FinonacciTestData(val number: Int, val exptected : Int)

@ParameterizedTest
@MethodSource("fibonacciTestData")
fun checkFiboUsingDataClass(data : FibonacciTestData) {
    assertThat(fibonacci(data.number), `is`(data.expected))
}

private fun fibonacciTestData() = Stream.of(
    FinonacciTestData(number = 1, expected = 1),
    FinonacciTestData(number = 2, expected = 1),
    FinonacciTestData(number = 3, expected = 2),
    FinonacciTestData(number = 4, expected = 3)
)
```

- data 클래스는 toString도 자동으로 정의 되어있어서 결과의 가독성도 높다

