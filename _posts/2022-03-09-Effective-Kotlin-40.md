---
layout: post
title: "Effective Kotlin - 아이템40: equals의 규약을 지켜라"
description: equals의 규약을 지켜라
date: 2022-03-09 13:40:00 +09:00
categories: EffectiveKotlin Study
---


# 클래스 설계

## 아이템 40: equals의 규약을 지켜라

- 코틀린의 Any에는 equals, hashCode, toString 등의 잘 설정된 규약을 가진 메서드들이 있음. Any클래스를 상속받는 모든 메서드라면 이러한 규약은 잘 지켜주는것이 좋음

### 동등성
- 코틀린에는 2가지 동등성이 존재
    * 구조적 동등성: equals 메서드와 이를 기반으로 만들어진 ==, != 으로 확인하는 동등성. a가 nullable이 아니라면 a == b는 a.equals(b)로 변환되고, a가 nullable 이라면 a?.equals(b) ?: (b === null) 로 변환
    * 레퍼런스적 동등성: ===, !== 로 확인하는 동등성. 두 피연산자가 같은 객체를 가리키면 true 리턴

- equals는 모든 클래스의 슈퍼 클래스인 Any에 구현되어 있으므로 모든 객체에서 사용할 수 있지만, 다른 타입의 두객체를 연산자로 비교하는것은 허용되지 않음 ("".equals(1)은 가능하지만 "" == 1 은 불가)

### equals가 필요한 이유
- Any클래스에 구현되어 있는 equals 메서드는 디폴트로 === 처럼 두 인스턴스가 완전히 같은 객체인지를 비교. 이는 모든 객체는 디폴트로 유일한 객체라는것을 의미
- 이러한 동작은 DB 연결, 리포지토리, 스레드 등의 활용 요소를 활용할때 굉장히 유용하나 data클래스 처럼 동등성을 약간 다른 형태로 표현해야 하는 객체들이 있음
- data 클래스는 기본 생성자의 프로퍼티가 같다면 같은 객체로 취급함

```kotlin
data class FullName(val name: String, val surname: String)
val name1 = FullName("Marchin","Moskala")
val name2 = FullName("Marchin","Moskala")

name1 == name2 // true
nane1 === name2 // false
```

- 이렇게 data 한정자를 기반으로 동등성의 조작을 할 수 있으므로 일반적으로 코틀린에서 equals를 직접 구현할 필요가 없지만, 상황에 따라 equals를 직접 구현 해야할수도 있음
- equals를 직접 구현해야 하는 경우
    * 기본적으로 제공되는 동작과 다른 동작을 해야하는 경우
    * 일부 프로퍼티만 비교해야하는 경우
    * data 한정자를 붙이는것을 원하지 않거나, 비교해야하는 프로퍼티가 기본 생성자에 없는 경우

### equals 규약
- 반사적 동작: x가 null이 아닌 값이라면 x.equals(x) 는 true
- 대칭적 동작: x와 y가 null이 아닌 값이라면 x.equals(y)는 y.equals(x)와 같아야 한다
- 연속적 동작: x,y,z가 null이 아닌 값이고 x.equals(y)와 y.equals(z)가 true라면 x.equals(z)도 true
- 일관적 동작: x와 y가 null이 아니라면 x.equals(y)는 여러번 실행 해도 항상 같은 결과를 리턴 해야 함
- 널과 관련된 동작: x가 null이 아니라면 x.equals(null)은 항상 false

### equals 구현하기
- 특별한 이유가 없는한 equals를 직접 구현하는것은 좋지 않음. 기본적으로 제공되는것을 그대로 쓰거나 data 클래스로 만들어서 쓰는것이 좋음
- 꼭 구현해야 한다면 equals의 규약을 지키면서 만들고 final 클래스로 만드는것이 좋음(상속을 하면서 서브클래스에서 equals가 작동하는 방식이 변경되면 안되기 떄문)