---
layout: post
title: "Effective Kotlin - 아이템37: 데이터 집합 표현에 data 한정자를 사용하라"
description: 데이터 집합 표현에 data 한정자를 사용하라
date: 2022-03-07 22:45:00 +09:00
categories: EffectiveKotlin Study
---


# 클래스 설계

## 아이템 37: 데이터 집합 표현에 data 한정자를 사용하라

- 데이터들을 한번에 전달해야 할 때는 data 클래스를 사용 함

```kotlin
data class Player(
    val id: Int,
    val name: String,
    val points: Int
)

val player = Player(0, "Gecko", 9999)
```

- data 한정자를 붙이면 다음과 같은 함수가 자동으로 생성 됨
    * toString
    * equals, hashCode
    * copy
    * componentN

- toString 함수는 클래스의 이름이나 기본 생성자 형태로 모든 프로퍼티와 값을 출력
- equals는 기본 생성자의 프로퍼티가 같은지 확인해줌, hashCode는 equals와 같은 결과를 냄
- copy는 immutable 데이터 클래스를 만들때 편리함. 기본 생성자 프로퍼티가 같은 새로운 객체를 복제 하는데, 새로 만들어진 객체의 값은 이름 있는 아규먼트를 이용해서 변경 가능
- componentN 함수는 위치를 기반으로 객체를 해체할 수 있게 해줌. 위치를 기반으로 하기 떄문에 위치를 혼동하지 않게 주의

### 튜플 대신 데이터 클래스 사용하기
- 데이터 클래스는 튜플(Serializable 기반, toString을 사용할 수 있는 제너릭 데이터 클래스)보다 많은것을 제공

```kotlin
public data class Pair<out A, out B>(
    public val first: A,
    public val second: B
): Serializable {
    
    public override fun toString(): String =
        "($first, $second)"
}

public data class Triple<out A, out B, out C>(
    public val first: A,
    public val second: B,
    public val third: C
): Serializable {

    public override fun toString(): String =
        "($first, $second, $third)"
}
```

- Pair와 Triple이 코틀린에 남아있는 유일한 튜플임. Pair와 Triple이 남아있는 이유는 다음과 같음. 이 경우를 제외 한다면 무조건 데이터 클래스를 사용 하는 것이 나음
    * 값에 간단하게 이름을 붙일때
    * 표준 라이브러리에서 볼 수 있는 것처럼 미리알 수 없는 aggregate을 표현할 때

```kotlin
fun String.parseName(): Pair<String, String>? { // Pair<String, String>이 전체 이름을 나타낸다라는것을 인지하기 어렵고, firstName / lastName중 어떤게 앞에 있을지 예측을 못함
    val indexOfLastSpace = this.trim().lastIndexOf(' ')
    if(indexOfLastSapce < 0) return null
    val firstName = this.take(indexOfLastSpace)
    val lastName = this.drop(indexOfLastSapce)
    return Pair(fistName, lastName)
}

val fulName = "Marcin Moskala"
val (firstName, lastName) = fullName.parseName() ?: return

// 다음과 같은 data 클래스를 사용하면 위의 문제를 해결 가능
data class FullName(
    val firstName: String,
    val lastName: String
)

fun String.parseName(): FullName? {
    val indexOfLastSpace = this.trim().lastIndexOf(' ')
    if(indexOfLastSapce < 0) return null
    val firstName = this.take(indexOfLastSpace)
    val lastName = this.drop(indexOfLastSapce)
    return FullName(fistName, lastName)
}
```

- 튜플을 데이터 클래스로 전환해도 추가 비용은 거의 들지 않으며 다음과 같이 함수를 더 명확하게 만들어줌
    * 함수의 리턴 타입이 더 명확
    * 리턴 타입이 더 짧아지며, 전달하기 쉬워짐
    * 사용자가 데이터 클래스에 적혀있는 것과 다른 이름을 활용해 변수를 해제하면 경고가 출력됨