---
layout: post
title: "Effective Java - 아이템35: ordinal메서드 대신 인스턴스 필드를 사용하라"
description: ordinal메서드 대신 인스턴스 필드를 사용하라
date: 2020-08-24 21:06:00 +09:00
categories: EffectiveJava Study
---


# 열거타입과 어노테이션

## 아이템 35 : ordinal메서드 대신 인스턴스 필드를 사용하라

- 대부분의 열거타입 상수는 자연스럽게 정숫값 하나에 대응되고, 모든 열거 타입은 해당 상수가 그 열거타입의 몇번째 위치인지 반환하는 ordinal 메서드를 제공한다

```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET...;

    public int numberOfMusicians() { return ordinal() + 1; }
}
```

- 따라서 위 코드처럼 열거 타입 상수와 연결된 정숫값이 필요하면 ordinal을 활용하곤 하는데, 이 코드는 동작하지만 유지보수하기 끔찍한 코드다
    * 상수 선언 순서가 바뀌는 순간 numberOfMusicians가 오작동 하며, 이미 사용중인 정숫값과 값이 같은 상수는 추가할 방법이 없다
    * 또한 중간에 값을 비워둘수 없다

- 이 방법에 대한 해결책은 간단한데, 열거타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고 인스턴스 필드에 저장하면 된다

```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5)...;

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```

- Enum의 API문서에 보면 ordinal에 대해 이렇게 쓰여있다
    * "대부분의 프로그래머는 이 메서드를 쓸일이 없다. 이 메서드는 EnumSet과 EnumMap과 같이 열거타입 기반의 범용 자료구조에 쓸 목적으로 설계되있다"
    * 즉 이런 용도가 아니라면 ordinal을 쓰면 안된다

