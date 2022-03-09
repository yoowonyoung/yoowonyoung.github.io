---
layout: post
title: "Effective Kotlin - 아이템42: compareTo의 규약을 지켜라"
description: compareTo의 규약을 지켜라
date: 2022-03-09 22:00:00 +09:00
categories: EffectiveKotlin Study
---


# 클래스 설계

## 아이템 42: compareTo의 규약을 지켜라

- compareTo 메서드는 Any 클래스에 있는 메서드가 아님. 수학적인 부등식으로 변환되는 연산자

```kotlin
obj1 > obj2 // obj1.compareTo(obj2) > 0
obj1 < obj2 // obj1.compareTo(obj2) < 0
obj1 >= obj2 // obj1.compareTo(obj2) >= 0
obj1 <= obj2 // obj1.compareTo(obj2) <= 0
```

- compareTo는 다음과 같이 동작 해야함
    * 비대칭적 동작: a >= b이고, b <= a 라면 a == b여야함. 비교와 동등성 비교에 어떤 관계가 있어야 하고 서로 일관성이 있어야 함
    * 연속적 동작: a >= b이고, b >= c라면 a >= c이여야함. 마찬가지로 a > b 이고 b > c 라면 a > c 여야함
    * 코넥스적 동작: 두 요소는 어떤 확실한 관계를 갖고 있어야함. a >= b 또는 b >= a중에 적어도 하나는 true 여야 함

### 왜 compareTo를 따로 정의해야 할까
- 일반적으로 compareTo를 정의해야하는 상황은 거의 없음. sortedBy를 사용하면 원하는 키로 컬렉션 정렬이 가능하고, 여러 프로퍼티를 기반으로 정렬 하고 싶다면 sortedWith를 사용하면 되기 떄문

```kotlin
class User(val name: String, val surname: String)
val names = listOf<User>(/*..*/)

val sorted = names.sortedBy { it.surename }
val sorted2 = names.sortedWith(compareBy({ it.surname}, { it.name })) // compareBy 를 활용해서 comparator를 만들어야 함
```

### compareTo 구현하기
- compareTo를 구현할 때 유용하게 활용할 수있는 톱레벨 함수가 있음. 두 값을 단순하게 비교할때만 사용 하는 compareValues와 더 많은 값을 비교하거나, selector를 활용해서 비교 하고 싶다면 compareValuesBy

```kotlin
class User(
    val name: String,
    val surname: String
): Comparable<User> {
    override fun compareTo(other: User): Int =
        compareValues(surname, other.surname)
}

class User(
    val name: String,
    val surname: String
): Comparable<User> {
    override fun compareTo(other: User): Int =
        comparaValuesBy(this, other, { it.surname }, { it.name })
}
```

- compareTo는 다음과 같은 값을 리턴해야함을 잊으면 안됨
    * 0: 리시버와 other가 같은 경우
    * 양수: 리시버가 other보다 큰 경우
    * 음수: 리시버가 other보다 작은 경우

