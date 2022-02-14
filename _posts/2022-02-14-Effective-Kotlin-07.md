---
layout: post
title: "Effective Kotlin - 아이템7: 결과 부족이 발생할 경우 null과 Failure를 사용 하라"
description: 결과 부족이 발생할 경우 null과 Failure를 사용 하라
date: 2022-02-14 21:40:00 +09:00
categories: EffectiveKotlin Study
---


# 안정성

## 아이템 7 : 결과 부족이 발생할 경우 null과 Failure를 사용 하라
- 다음과 같은 경우에 함수가 원하는 결과를 만들어 내지 못할 때가 있음
    * 서버로 부터 데이터를 읽어 들이려고 하는데, 인터넷 연결이 문제가 되는 경우
    * 조건에 맞는 첫번쨰 요소를 찾으려고 했는데, 조건에 맞는 요소가 없는 경우
    * 텍스트를 파싱해서 객체를 만들려고 했는데, 텍스트의 형식이 맞지 않는 경우

- 이런 상황을 처리하는 매커니즘은 크게 다음과같이 2가지가 있음
    * null 또는 실패를 나타내는 sealed 클래스(일반적으로 Failure라는 이름을 붙임)을 리턴
    * 예외를 throw

- 예외는 정보를 전달하는 방법으로 사용해서는 안됨. 예외는 잘못된 특별한 상황을 나타내야 하며, 처리되어야함. 예외는 정말 예외적인 상황이 발생 할 때 사용 해야함
- null과 Failure는 예상되는 오류를 표현할때 굉장히 좋음. 명시적이고 효율적이며 간단한 방법으로 처리 할 수 있음
- 따라서 충분히 예측할 수 있는 범위의 오류는 null과 Failure를 사용 하고, 예측 하기 어려운 예외적인 범위의 오류는 예외를 throw 하는 것이 좋음

```kotlin
inline fun <reified T> String.readObjectOrNUll(): T? {
    //...
    if(incorrectSign) {
        return null
    }
    //...
    return result
}

inline fun <reified T> String.readObject(): Result<T> {
    //...
    if(incorrectSign) {
        return Failure(JsonParsingException())
    }
    //...
    return Succuss(result)
}

sealed class Result<out T>
class Success<out T>(val result: T): Result<T>()
class Failure(val throwable: Throwable): Result<Nothing>()

class JsonParsingException: Exception()
```

- 이렇게 표시 되는 오류는 다루기 쉬우며 놓치기는 어려움
    * null을 처리 하기 위해 null-safty 기능을 사용 해야 함
    * Result와 같은 Union Type을 리턴 하기로 했다면 when 표현 식을 사용 해서 이를 처리 할 수있음

- null과 sealed result 클래스의 차이는 추가로 정보를 전달 할것이라면 sealed result, 그렇지 않다면 null을 반환 하는것이 일반적임
- 보통 2가지 형태의 함수를 사용 (List의 예시)
    * get: 예상 할 수 있을때. 특정 위치에 있는 요소를 추출. 만약 없다면 IndexOutOfBoundsException을 발생
    * getOrNull: 예상 할 수 없을떄. out of range 오류가 발생할 수 있는 경우에 사용 하며, 발생한 경우에는 Null을 리턴
    * 추가로 getOrDefault 같은 선택지도 있음. 일반적으로는 getOrNull 또는 엘비스 연산자를 사용하는것이 쉬움
