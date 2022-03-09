---
layout: post
title: "Effective Kotlin - 아이템41: hashCode의 규약을 지켜라"
description: hashCode의 규약을 지켜라
date: 2022-03-09 20:40:00 +09:00
categories: EffectiveKotlin Study
---


# 클래스 설계

## 아이템 41: hashCode의 규약을 지켜라

- hashCode 함수는 수 많은 컬렉션과 알고리즘에 사용되는 자료구조인 해시테이블을 구축할 때 사용됨

### 해시 테이블
- 컬렉션에 요소를 빠르게 추가하고 빠르게 추출 해야 할떄를 위해 해시 테이블이 만들어 졌는데, 해시 테이블의 성능은 해시 함수로 인해 좋은것
- 해시 함수의 특징
    * 빠르다
    * 충돌이 적다(다른 값이면 최대한 다른 숫자를 리턴)

### 가변성과 관련된 문제
- 해시 테이블에 요소가 추가될 때만 해시 코드를 계산. 요소가 변경되어도 해시 코드는 계산되지 않으며, 버킷 재배치도 이뤄지지 않음. 따라서 기본적인 LinkedHashSet과 LinkedHashMap의 키는 한번 추가한 요소를 변경할 수 없음

### hashCode 규약
- 어떤 객체를 변경하지 않았다면(equals에서 비교에 사용한 정보가 수정되지 않았다면) hashCode는 여러번 수행해도 그 결과가 항상 같아야 함 -> 일관성 유지를 위해 hashCode가 필요
- equals 메서드의 실행 결과로 두 객체가 같다고 나온다면, hashCode 메서드의 호출 결과도 같다고 나와야함
- hashCode와 equals는 같이 일관성 있는 동작을 해야 함. 즉 같은 요소는 반드시 같은 해시 코드를 가져야 함. 그래서 코틀린은 equals 구현을 오버라이드 할 때 hashCode도 함께 오버라이드 하는것을 추천
- 필수 요구사항은 아니지만 제대로 사용하려면 지켜야 하는 요구사항이 hashCode는 최대한 요소를 넓게 퍼트려야함. 다른 요소라면 최대한 다른 해시값을 가질 수 있도록 해야 함

### hashCode 구현하기
- 일반적으로 data 클래스로 만들면 코틀린이 알아서 equals와 hashCode를 만들어주므로 이를 직접 정의할일은 없음
- 하지만 equals를 따로 정의 했다면 반드시 hashCode도 정의 해줘야 함. equals에서 비교에 사용되는 프로퍼티를 기반으로 해시코드를 만들어야함
- 일반적으로 모든 해시코드의 값을 더하면서, 더하는 과정마다 이전까지의 결과에 31을 곱한뒤 더해줌(관례적으로 31을 많이 사용함)

```kotlin
class DateTime(
    private var millis: Long = 0L,
    private var timeZone: TimeZone? = null
) {
    private var asStringCache = ""
    private var changed = false

    override fun equals(other: Any?): Boolean = 
        other is DateTIme &&
            other.millis == millis &&
            other.timeZone == timeZone

    override fun hashCode(): Int {
        var result = millis.hashCode()
        result = result * 31 + timeZone.hashCode()
        return result
    }
}
```

- hashCode를 구현할 떄 가장 중요한 규칙은 '언제나 equals와 일관된 결과가 나와야 한다' 임
