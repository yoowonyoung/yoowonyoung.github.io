---
layout: post
title: "Kotlin Cookbook - 7장: 영역함수"
description: Kotlin Cookbook 7장
date: 2022-01-09 13:43:00 +09:00
categories: Kotlin Cookbook Study
---

# 영역함수

## apply로 객체 생성 후에 초기화 하기

### 문제
- 객체를 사용하기 전에 생성자 인자만으로는 할 수 없는 초기화 작업이 필요하다

### 해법
- apply 함수를 사용 한다

### 설명
- 코틀린 객체에 적용할 수 있는 몇몇의 영역 함수가 있음
- apply는 this를 인자로 전달하고 this를 리턴하는 확장함수

```kotlin
inline fun <T> T.apply(block: T.() -> Unit): T
```

- apply는 모든 제너릭 타입 T에 존재하는 확장 함수, apply 함수는 명시된 블록을 수신자인 this와 함께 호출하고 해당 블록이 완료되면 this를 리턴
- JdbcTemplate 기반의 SimpleJdbcInsert 클래스의 예시

```kotlin
@Repository
class JdbcOfficerDAO(private val jdbcTemplate: JdbcTemplate) {
    private val insertOfficer = SimpleJdbcInsert(jdbcTemplate)
        .withTableName("OFFICERS")
        .usingGeneratedKeyColumns("id)
    
    fun save(officer: Officer) =
        officer.apply {
            id = insertOfficer.executeAndReturnKey(
                mapOf("rank" to rank,
                    "first_name" to first,
                    "last_name" to last))
        }
}
```

- Officer 인스턴스는 this로 apply블록에 전달되기 때문에 블록 안에서 rank, first, last속성에 접근 가능
- apply 블록은 이미 인스턴스화 된 객체의 추가 설정을 위해 사용하는 가장 일반적인 방법

## 부수 효과를 위해 also 사용하기

### 문제
- 코드 흐름을 방해하지 않고 메시지를 출력 하거나 다른 부수 효과를 생성하고 싶다

### 해법
- aslo 함수를 사용해 부수 효과를 생성하는 동작을 수행 한다

### 설명
- also 함수는 코틀린 표준 라이브러리에 있는 확장 함수

```kotlin
public inline fun<T> T.also(
    block: (T) -> Unit
): T
```

- also는 모든 제너릭 타입 T에 추가 되고 block 인자를 실행 시킨후에 자기 자신을 리턴 한다
- also는 일반적으로 객체에 함수 호출을 연쇄시키기 위해 사용 된다

```kotlin
val book = createBook()
    .also{ println(it) }
    .also{ Logger.getAnonymouseLogger().info(it.toString()) }
```

- 다수의 also 호출을 함께 연쇄 할수는 있지만 일반적으로 연속 비즈니스 로직 호출은 함수의 일부분을 추가

```kotlin
class Site(val name: String,
        val latitude: Double,
        val longitude: Double)

@Test
fun LatLongOfBostonMA() = service.getLatLong("Boston", "MA")
    .also{ logger.info(it.toLong()) }
    .run{
        assertThat(latitude, `is`(cloaserTo(42.36, 0.01))
        assertThat(longitude, `is`(colaserTo(-71.06, 0.01))))
    }
```

- run함수는 컨택스트 객체 대신 람다의 값을 리턴하기 떄문에 이 코드의 also호출은 테스트의 run 호출 전에 위치 해야함

## let 함수와 엘비스 연산자 이용하기

### 문제
- 오직 널이 아닌 레퍼런스의 코드 블록을 실행하고 싶지만, 레퍼런스가 널이라면 기본값을 리턴 하고 싶다

### 해법
- 엘비스 연산자를 결합한 안전 호출 연산자와 함께 let 영역 함수를 사용

### 설명
- let함수는 모든 제너릭 T 타입의 확장 함수

```kotlin
public inline fun<T,R> T.let(
    block: (T) -> R
): R
```

- let 함수에 기억해야할 중요한 점은 let 함수는 컨텍스트 객체가 아닌 블록의 결과를 리턴한다는 것
- let은 객체를 위한 map처럼 컨텍스트 객체의 변형 처럼 동작함

```kotlin
fun processString(str: String) =
    str.let {
        when {
            it.isEmpty() -> "Empty"
            it.isBlank() -> "Blank"
            else -> it.capitalize()
        }
    }
```

- 문자열을 받아 해당 문자열을 대문자로 변경하면서 공백문자나 빈 문자열 같은 특수한 입력 처리 하는 경우를 표현. 이때 인자가 Nullable인경우 다음과 같아질것

```kotlin
fun processString(str: String?) =
    str?.let {
        when {
            it.isEmpty() -> "Empty"
            it.isBlank() -> "Blank"
            else -> it.capitalize()
        }
    } ?: "Null"
```

- 두 코드 모드 함수의 리턴 타입은 모두 String. 안전 호출 연산자 ?.과 let 함수 엘비스 연산자 :? 조합으로 모든 경우를 쉽게 처리 할 수 있음

## 임시 변수로 let 사용하기

### 문제
- 연산 결과를 임시 변수에 할당하지 않고 처리 하고 싶음

### 해법
- 연산에 let 호출을 연쇄하고 let에 제공한 람다 또는 함수 레퍼런스 안에서 그 결과를 처리

### 설명
-코틀린 공식 사이트의 영역 함수 문서 페이지에 있는 let 함수의 사용 사례

```kotlin
val numbers = mutableListOf("one", "two", "three", "four", "five")
val resultList = numbers.map { it.length }.filter{ it > 3}
println(resultList)
```

- 위의 코드를 let 블록을 사용하도록 리팩토링 하면 다음과 같이됨

```kotlin
val numbers = mutableListOf("one", "two", "three", "four", "five")
numbers.map { it.length }.filter { it > 3 }.let {
    println(it)
}
```

- 결과를 임시 변수에 할당하는 대신 연쇄시킨 let 호출을 사용하는 목적은, 결과 자체를 컨텍스트 변수로 사용하는것이기 때문에 let에 제공한 블록 안에서 결과를 출력할수있음
- let을 사용하는 경우에 also로도 바꿔 쓸 수 있는 경우가 많지만, also는 부수효과를 사용하기 위한것이니 목적에 맞게 잘 사용하는게 좋다
