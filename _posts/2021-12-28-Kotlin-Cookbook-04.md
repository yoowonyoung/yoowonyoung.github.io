---
layout: post
title: "Kotlin Cookbook - 5장: 컬렉션"
description: Kotlin Cookbook 5장
date: 2021-12-28 22:01:00 +09:00
categories: Kotlin Cookbook Study
---

# 컬렉션

## 배열 다루기

### 문제
- 코틀린에서 배열을 생성하고 배열에 데이터를 추가하고 싶다

### 해법
- arrayOf 함수를 이용해 배열을 만들고, Array클래스에 들어있는 속성과 메서드를 이용해 배열에 들어있는 값을 다룬다

### 설명
- 코틀린은 배열을 생성하는 arrayOf라는 이름의 간단한 팩토리 메서드를 제공
- 코틀린에서 배열에 접근할때 사용하는 문법은 자바와 똑같지만 코틀린에서 Array는 클래스임
- arrayOfNulls 팩토리 메서드를 사용해 널로만 채워진 배열을 생성할수도 있음. emptyArray 팩토리 메서드도 동일하게 동작

```kotlin
val strings = arrayOf("this","is","an","array","of","strings")
val nullStringArray = arrayOfNulls<String>(5) // 배열에 널만 있음에도 특정 타입을 선택 해야함
```

- Array 클래스에는 public 생성자 하나만 있고, 다음 두개의 인자를 받음
    * Int 타입의 size
    * init, 즉 (Int) -> T 타입의 람다. 배열을 생성할때 인덱스 마다 호출

```kotlin
val squares = Array(5) { i -> (i * i).toString() }
```

- Array 클래스에는 squares[1] 처럼 대괄호를 사용해 배열의 원소에 접근할 수 있는 public 연산자 메서드 get/set이 정의되어 있음
- 코틀린에는 오토박싱과 언박싱 비용을 방지할 수 있는 기본 타입을 booleanArrayOf, byteArrayOf, shotArrayOf, charArrayOf, intArrayOf, longArrayOf, floatArrayOf, doubleArrayOf가 있음
    * 코틀린에서 명시적인 기본 타입은 없지만 널 허용인 경우 바이트코드는 Integer같은 자바 래퍼 클래스를 사용하고, 널 비허용이면 int같은 기본 타입을 사용

- 배열에는 두어개의 고유한 확장함수가 존재하는데 배역의 적법한 인덱스 값을 알 수 있는 indices 속성과, 인덱스와 같이 배열을 순회할 때 사용하는 withIndex가 있음

## 컬랙션 생성하기

### 문제
- 리스트, 셋, 맵을 생성하고 싶음

### 해법
- listOf, setOf, mapOf 처럼 변경 불가능한 컬렉션을 생성하기 위해 만들어진 함수나, mutableListOf, mutableSetOf, mutableMapOf 처럼 변경 가능한 컬랙션을 생성하기 위해 만들어진 함수중 하나를 사용

### 설명
- 어떤 컬랙션의 변경 불가능한 뷰를 얻고 싶다면 kotlin.collections 패키지가 제공하는 유틸리티 함수인 listOf, setOf, mapOf등을 이용하면 됨

```kotlin
public fun <T> listOf(vararg elements: T): List<T> =
    if(elements.size > 0) elements.asList() else emptyList()
```

- 기본적으로 코틀린 컬렉션은 불변. 그런 의메어서 컬렉션은 원소를 추가하거나 제거하는 메서드를 지원하지 않음. 컬렉션 스스로는 오직 읽기 전용 연산만을 지원
- 컬렉션을 변경하는 메서드는 mutableListOf, mutableSetOf, mutableMapOf 같은 가변 인터페이스에 들어 있음

## 컬렉션에서 읽기 전용 뷰 생성하기

### 문제
- 변경 가능한 리스트, 셋, 맵이 있을때 해당 컬렉션의 읽기 전용 버전을 생성하고 싶음

### 해법
- toList, toSet, toMap 메서드를 이용해 새로운 읽기 전용 컬렉션을 생성
- 기존 컬렉션을 바탕으로 읽기 전용 뷰를 만들려면 List, Set, Map 타입의 변수에 기존 컬렉션을 할당

### 설명

```kotlin
val readOnlyNumList: List<Int> = mutableNums.toList() // 내용은 원본과 같지만 더이상 같은 객체는 아님
val readOnlySameList: List<Int> = mutableNums // 내용도 원본과 같고 같은 객체지만 읽기 전용
```

## 컬렉션에서 맵 만들기

### 문제
- 키 리스트가 있을때 각각의 키와 생성한 값을 연관시켜서 맵을 만들고 싶음

### 해법
- associateWith 함수에 각 키에 대해 실행되는 람다를 제공해 사용

### 설명
- 한 묶음의 키가 있고, 각 키를 생성된 값과 매핑하고싶다면 associate 함수를 사용하면 됨

```kotlin
val keys = 'a'..'f'
val map = keys.associate { it to it.toString().repeat(5).capitalize() }
println(map) // {a=Aaaaa, b=Bbbbb, c=Ccccc, d=Ddddd, e=Eeeee}
```

- associate함수는 람다를 받아 T를 ```Pair<K,V>```로 변환하는 ```Iterable<T>```의 인라인 확장함수
- 예제에서 to는 자신의 왼쪽 인자와 오른쪽 인자를 바탕으로 Pair를 생성하는 중위 함수
- associateWith는 더 간단하게 사용가능

```kotlin
val keys = 'a'..'f'
val map = keys.associateWith { it.toString().repeat(5).capitalize() }
println(map) // {a=Aaaaa, b=Bbbbb, c=Ccccc, d=Ddddd, e=Eeeee}
```

## 컬렉션이 빈 경우 기본값 리턴하기

### 문제
- 컬렉션을 처리할떄 컬렉션의 모든 원소가 선택에서 제외 되지만 기본 응답을 리턴하고 싶음

### 해법
- 컬렉션이나 문자열이 비어있는 경우에는 ifEmpty나 ifBlank함수를 이용해 기본값 리턴

### 설명

```kotlin
data class Product(val name: String, val price: Double, val onSale: Boolean = flase)

fun onSaleProduct_ifEmptyCollection(products: List<Product>) =
    products.filter { it.onSlae }
        .map { it.name }
        .ifEmpty { listOf("none") } // 빈 컬렉션에 기본 리스트 제공
        .joinToString(separator = ", ")

fun onSaleProduct_ifEmptyString(products: List<Product>) =
    product.filtst { it.onSale }
        .map { it.name }
        .joinToString(seperator = ", ")
        .ifEmpty { "none" } // 빈 문자열에 기본 문자열 제공
```

## 주어진 범위로 값 제한하기

### 문제
- 값이 주어졌을때, 주어진 값이 특정 범위 안에 들면 해당 값을 리턴하고 그렇지 않다면 범위의 최솟값 또는 최댓값을 리턴하고싶다

### 해법
- kotlin.ranges의 coeceIn 함수를 범위인자 또는 구체적인 최솟값, 최댓값과 함께 사용

### 설명
- 범위함수 coerceIn 에는 2개의 중복이 존재 하는데, 두 중복중 하나는 닫힌 범위를 인자로 받고 다른 하나는 최솟값과 최댓값을 인자로 받음

```kotlin
@Test
fun coerceInGivenRange() {
    val range = 3..8

    assertThat(5, `is`(5.coerceIn(range)))
    assertThat(range.start, `is`(1.coerceIn(range))) // 1은 범위 바깥이므로 경계값이 리턴
    assertThat(range.endInclusive, `is`(9.coerceIn(range))) // 9은 범위 바깥이므로 경계값이 리턴
}

@Test
fun coeceInGivenMinAndMax() {
    val min = 2
    val max = 6
    assertThat(5, `is`(5.coerceIn(min,max)))
    assertThat(min, `is`(1.coerceIn(min,max))) // 1은 범위 바깥이므로 경계값이 리턴
    assertThat(max, `is`(9.coerceIn(min,max))) // 9은 범위 바깥이므로 경계값이 리턴
}
```

## 컬렉션을 윈도우로 처리하기

### 문제
- 값 컬렉션이 주어진 경우 컬렉션을 횡단하는 작은 윈도우를 이용해 컬렉션을 처리하고 싶다

### 해법
- 컬렉션을 같은 크기로 나누고 싶다면 chunked 함수를 사용하고, 정해진 간격으로 컬렉션을 따라 움직이는 블록을 원한다면 windowed 함수를 사용

### 설명
- Iterable 컬렉션에서 chunked함수는 컬렉션을 주어진 크기 또는 그보다 더 작게 리스트의 리스트로 분할. chunked함수는 리스트의 리스트를 리턴할수도 있고, 개발자는 chunked함수의 결과 리스트에 적용할 변환 람다를 제공할수도 있음

```kotlin
fun <T> Iterable<T>.chunked(size: Int): List<List<T>>

fun <T,R> Iterable<T>.chunked(
    size: Int,
    transform: (List<T>) -> R
): List<R>
```

- 정수 0부터 10까지를 범위로 정해서 그 합과 평균을 계산하는 예시

```kotlin
@Test
internal fun chunked() {
    val range = 0..10
    val chunked = range.chunked(3)
    assertThat(chunked, contains(listOf(0,1,2), listOf(3,4,5), listOf(6,7,8), listOf(9,10)))
    assertThat(range.chunked(3) { it.sum() }, `is`(lisOf(3,12,21,19)))
    assertThat(range.chunked(3) { it.average() }, `is`(lisOf(1.0,4.0,7.0,9.5)))
}
```

- chunked 함수는 특별한 경우의 windowed 함수로, chunked의 구현을 windowed에 위임 할 수 있다

```kotlin
public fun<T> Iterable<T>.chunked(size: Int):List<List<T>> {
    return windowed(size, size, partialWindows = true)
}
```

- windowed 함수는 3개의 인자를 받고 그중 2개의 인자는 선택사항
    * size: 각 윈도우에 포함될 원소의 갯수
    * step: 각 단계마다 전진할 원소의 갯수(기본 1개)
    * partialWindows: 나뉘어있는 마지막 부분이 윈도우에 필요한 만큼의 원소 갯수를 가지 못할경우, 해당 부분을 그대로 유지할지 여부(기본 flase)

```kotlin
@Test
fun windowed() {
    val range = 0..10

    assertThat(range.windowed(3,3), contains(listOf(0,1,2), listOf(3,4,5), listOf(6,7,8)))
    assertThat(range.windowed(3,3) { it.average() }, contains(1.0, 4.0, 7.0))
    assertThat(range.windowed(3,1), contains(listOf(0,1,2), listOf(1,2,3), listOf(2,3,4),
        listOf(3,4,5), listOf(4,5,6), listOf(5,6,7),
        listOf(6,7,8), listOf(7,8,9), listOf(8,9,10)))
}
```

## 리스트 구조 분해하기

### 문제
- 리스트의 원소에 접근할 수 있게 구조 분해 하고싶다

### 해범
- 최대 5개의 원소를 가진 그룹에 리스트를 할당한다

### 설명
- 구조 분해는 변수 묶음에 추출한 값을 할당해 객체에서 값을 추출하는 과정

```kotlin
val list = listOf("a","b","c","d","e","f","g")
val (a,b,c,d,e) = list
println("$a $b $c $d $e") // a b c d e
```

- 이 코드가 동작하는 이유는 코틀린 표준 라이브러리 List 클래스에 N이 1에서부터 5까지인 componentN 이라는 이름의 확장함수가 정의되어 있기 때문
- 구조분해는 componentN 함수의 존재에 의존
- 데이터 클래스는 정의된 모든 속성 관련 component 메서드를 자동으로 추가

## 다수의 속성으로 정렬하기

### 문제
- 클래스를 어떤 속성으로 정렬한다음, 동일한 값을 다른 속성으로 정렬하는등 이처럼 계속해서 클래스를 다수의 속성으로 정렬하고싶다

### 해법
- sortedWith와 comparedBy함수를 사용

### 설명
- Golfer라는 이름의 간단한 data클래스와 샘플 컬렉션이 있다고 가정

```kotlin
data class Golfer(val score:Int, val first: String, val last: String)
val golfers = listOf(
    Golfer(70, "Jack", "Nicklaus"),
    Golfer(68, "Tom", "Watson"),
    Golfer(68, "Bubba", "Watson"),
    Golfer(70, "Tiger", "Woods"),
    Golfer(68, "Ty", "Webb")
)
```

- 골프선수를 점수로 정렬한다음, 점수가 같으면 성으로 정렬, 마지막으로 점수와 성이 같으면 이름으로 정렬하려면 다음과 같이 작성

```kotlin
val sorted = golfers.sortedWith(
    comparedBy({ it.score }, { it.last }, { it.first })
)
```

- sortedWith와 comparedBy함수 시그니처는 다음과 같다

```kotlin
fun <T> Iterable<T>.sortedWith(
    comparator: Comparator<in T>
): List<T>

fun <T> compareBy(
    vararg selectors: (T) -> Comparable<*>>
): Comparator<T>
```

- compareBy 함수는 Comparator를 생성하고 sortedWith함수는 Comparator를 인자로 받음
- compareBy에서 눈여겨 볼점은 Comparable을 추출하는 선택자 목록을 제공한다는것과 comparaBy함수가 차례차례 정렬에 사용되는 Comparator를 생성한다는것
- 같은 문제를 thenBy를 통해서 해결도 가능

```kotlin
val comparator = compareBy<Golfer>(Golfer::score)
    .thenBy(Golfer::last)
    .thenBy(Golfer::first)

golfer.sortedWith(comparator).forEach(::println)
```

## 사용자 정의 이터레이터 정의하기

### 문제
- 컬렉션을 감싼 클래스를 손쉽게 순회하고 싶다

### 해법
- next와 hasNext 함수를 모두 구현한 이터레이터를 리턴하는 연산자 함수를 정의

### 설명
- 다음은 코틀린의 이터레이터 정의

```kotlin
interface Iterator<out T> {
    operator fun next(): T
    operator fun hasNext(): Boolean
}
```

- 자바에서 for-each 루프를 사용하면 Iterable을 구현한 모든 클래스를 순회할 수 있고, 코틀린의 for-in 루프에도 비슷한 제약 조건이 있음

```kotlin
data class Player(val name: String)
class Team(val name: String, val players: MutableList<Player> = mutableListOf()) {
    // ...
}

// 선수 여러명이 있는 팀의 선수 목록을 순회하고 싶은 경우
for (player in team.player) {
    // ...
}
```

- iterator라는 이름의 연산자 함수를 정의하면 이 코드를 약간 더 간단하게 만들 수 있다

```kotlin
operator fun Team.iterator() : Iterator<Player> = players.iterator()

for(player in team) {
    // ...
}
```

## 타입으로 컬렉션을 필터링 하기

### 문제
- 여러 타입이 섞여있는 컬렉션에서 특정 타입의 원소로만 구성된 새 컬렉션을 생성하고 싶음

### 해법
- filterIsInstance 또는 filterIsIntanceTo 확장함수를 이용

### 설명

```kotlin
val list = listOf("a", LocalDateTime.now(), 3, 1, 4 "b" ) // List<Any>

val all = list.filterIsInstance<Any>()
val strings = list.filterIsInstance<String>()
val ints = list.filterIsInstance<Int>()
val dates = list.filterIsInstance<LocalDate::class.java>()
```

- filter를 이용한 타입 추론은 ```List<Any>```에 대한 영리한 타입변환이 되지 않으니 filterIsInstance를 사용 해야함
- filterIsInstanceTo는 특정 타입의 컬렉션 인자를 받고 그곳에 원본 컬렉션에 존재하는 해당 타입의 원소를 채움

```kotlin
val list = listOf("a", LocalDateTime.now(), 3, 1, 4 "b" ) // List<Any>

val all = list.filterIsInstanceTo(mutableListOf())
val strings = list.filterIsInstanceTo(mutableListOf<String>())
val ints = list.filterIsInstanceTo(mutableListOf<Int>())
val dates = list.filterIsInstanceTo(mutableListOf<LocalDate>())
```

## 범위를 수열로 만들기

### 문제
- 범위를 순회하고싶지만 범위가 간단한 정수 또는 문자로 구성되어있지 않다

### 해법
- 사용자 저으이 수열을 생성

### 설명
- 코틀린에서는 1..5 처럼 IntRage를 인스턴스화 하는 2개의 점 연산자를 사용해서 범위 생성. 범위는 2개의 끝점으로 정의되는 닫힌 간격이며 두 끝점은 모두 범위에 포함
- 코틀린 표준 라이브러리에는 Comparable 인터페이스를 구현하는 모든 제너릭 타입 T에 rangeTo라는 이름의 확장함수가 추가되어있음

```kotlin
operator fun <T: Compareble<T>> T.rangeTo(that: T): ClosedRange<T> =
    ComparableRange(this, that)
```

- ComparableRange는 그저 ClosedRange를 상속하고 T타입의 start와 endInclude속성을 정의하고, equals, hashCode, toString 함수를 적절하게 정의함

```kotlin
@Test
fun LocalDateInRange() {
    val startDate = LocalDate.now()
    val midDate = startDate.plusDays(3)
    val endDate = startDate.plusDays(5)

    val dateRange = startDate..endDate

    assertTrue(startDate in dateRange)
    assertTrue(midDate in dateRange)
    assertTrue(endDate in dateRange)
}
```

- 위 예제는 잘 동작하지만 ```for(date in dateRange)``` 와 같이 범위를 순회하면 컴파일 에러가 발생. 범위가 수열이 아니기 때문
- 사용자 정의 수열은 코틀린 표준 라이브러리의 IntProgression, LongProgression, CharProgression처럼 Iterable인터페이스를 구현해야함

```kotlin
class LocalDateProgression(
    override val start: LocalDate,
    override val endInclusive: LocalDate,
    val step: Long =1
) : Iterable<LocalDate>, ClosedRange<LocalDate> {
    override fun iterator(): Iterator<LocalDate> =
        LocalDateProgressionIterator(start, endInclusice, step)

    infix fun step(days: Long) = LocalDateProgression(start, endInclusive, days)
}
```

- Iterator 인터페이스에서 반드시 구현해야하는 유일한 함수는 iterator, 여기서 iterator 함수는 다음에 나오는 LocalDateProgressionIterator 클래스를 인스턴스화
- step 중위 함수는 주어진 days만큼 적절하게 증가시킨 LocalDateProgression 클래스를 인스턴스화

```kotlin
internal class LocalDateProgressionIterator(
    val start: LocalDate,
    val endInclusive: LocalDate,
    val step: Long
) : Iterator<LocalDate> {
    private var current = start
    
    override fun hasNext() = current <= endInclusive

    override fun next(): LocalDate {
        val next = current
        current = current.plusDays(step)
        return next
    }
}

operator fun LocalDate.rangeTo(other: LocalDate) =
    LocalDateProgression(this, other)
```

- Iterator 인터페이스는 next와 hasNext를 재정의
- LocalDateProgression인스턴스를 리턴하도록 확장함수를 사용해서 rangeTo 함수를 다시 정의
- 이제 LocalDate는 순회 범위를 만드는데 사용 가능

```kotlin
@Test
fun useLocalDateAsProgression() {
    val startDate = LocalDate.now()
    val endDate = startDate.plusDays(5)

    val dateRange = startDate..endDate
    dateRange.forEachIndexed { index, localDate ->
        assertEquals(localDate, startDate.plusDays(index.toLong()))    
    }
}

@Test
fun useLocalDateAsProgressionWithStep() {
    val startDate = LocalDate.now()
    val endDate = startDate.plusDays(5)

    val dateRange = startDate..endDate step 2
    dateRange.forEachIndexed { index, localDate ->
        assertEquals(localDate, startDate.plusDays(index.toLong() * 2))    
    }
}
```