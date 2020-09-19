---
layout: post
title: "Effective Java - 아이템73: 추상화 수준에 맞는 예외를 던져라"
description: 추상화 수준에 맞는 예외를 던져라
date: 2020-09-19 19:22:00 +09:00
categories: EffectiveJava Study
---


# 예외

## 아이템 73 : 추상화 수준에 맞는 예외를 던져라

- 수행하려는 일과 관련 없어보이는 예외가 튀어나오면 당황스러울것이다. 메서드가 저수준 예외를 처리하지않고 바깥으로 전파해버릴때 종종 일어나는 일이다
    * 이는 단순히 프로그래머를 당황시키는데 그치지않고, 내부 구현 방식을 드러내어 윗레벨 API를 오염시킨다

- 이 문제를 피하려면 상위 계층에서는 저수준 예외를 잡아 자신의 추상화  수준에 맞는 예외를 던져야한다. 이를 예외 번역이라고 한다

```java
try {
    ...
} catch (LowerLevelException e) {
    throw new HigherLevelException(...);
}
```

- 다음은 AbstractSequenctialList에서 수행하는 예외 번역의 예다. AbstractSequentialList는 List인터페이스의 골격 구현 아이템이다. 이 예에서 수행한 예외번역은 List<E> 인터페이스의 get메서드 명세에 명시된 필수사항이다

```java
/**
* 이 리스트 안의 지정한 위치의 원소를 반환한다
* @throws IndexOutOfBoundsException index가 범위 밖이라면 발생한다
*/
public E get(int index) {
    ListIterator<E> i = listIterator(index);
    try {
        return i.next();
    } catch (NoSuchElementException e) {
        throw new IndexOutOfBoundsException("인덱스: " + index);
    }
}
```

- 예외를 번역할때, 저수준 예외가 디버깅에 도움이 된다면 예외 연쇄를 사용하는게 좋다
    * 예외 연쇄란 문제의 근본 원인인 저수준예외를 고수준 예외에 실어 보내는 방식이다, 그러면 별도의 접근자 메서드를 통해 필요하면 언제든 저수준 예외를 꺼내 볼 수 있다

```java
try {
    ...
} catch (LowerLevelException cause) {
    throw new HigherLevelException(cause);
} 
```

- 고수준 예외의 생성자는(예외 연쇄용으로 설계된) 상위 클래스의 셍성자에게 이 원인을 건네주어 최종적으로 Thorwable(Throwable)생성자까지 건네지게 된다

```java
class HigherLevelException extends Exception {
    HigherLevelException(Throwable cause) {
        super(cause);
    }
}
```

- 대부분의 표준 예외는 예외 연쇄용 생성자를 가지고 있다. 그렇지 않은 예외라도 Throwable의 initCause메서드를 이용해 '원인'월 직접 못박을 수 있다. 예외 연쇄는 문제의 원인을 (getCause)메서드로 프로그램에서 접근할 수 있게 해주며, 원인과 고수준 예외의 스택 추적 정보를 잘 통합 해준다
- 무턱대고 예외를 전파하는것보다야 예외 번역이 우수한 방법이지만, 그렇다고 해서 남용하면 곤란하다. 가능하다면 저수준 메서드가 반드시 성공하도록 하여, 아래 계층에서는 예외가 발생하지 않도록하는것이 최우선이다. 때론 생위 계층메서드의 매개변수값을 아래 계층 메서드로 건네기전에 미리 검사하는 방법으로 이 목적을 달성 할 수 있다
- 차선책으로, 아래 계층에서의 예외를 피할 수 없다면, 상위계층에서 그 예외를 조용히 처리하여 API호출자에게까지 전파하지 않는 방법이 있다
    * 이런 경우 발생한 예외는 java.util.logging같은 적절한 로깅 기능을 활용해 기록해두면 좋다. 그렇게 해두면 클라이언트 코드와 사용자에게 문제를 전파하지 않으면서도 프로그래머가 로그를 분석해 추가 조치를 치할 수 있게 해준다