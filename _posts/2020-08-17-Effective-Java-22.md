---
layout: post
title: "Effective Java - 아이템22: 인터페이스는 타입을 정의하는 용도로만 사용해라"
description: 인터페이스는 타입을 정의하는 용도로만 사용해라
date: 2020-08-17 13:30:00 +09:00
categories: EffectiveJava Study
---


# 클래스와 인터페이스

## 아이템 22 : 인터페이스는 타입을 정의하는 용도로만 사용해라

- 인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다. 달리 말해 인터페이스를 구현한다는것은 자신의 인스턴스로 무엇을 할 수 있는지 클라이언트에 이야기 해주는것이다
- 인터페이스는 오직 이 용도로만 사용되어야 하지만, 상수인터페이스와 같이 이  지침에 맞지 않게 사용 되는 경우들도 있다

```java
public interface PhysicalConstant {
    static final double AVOGADROS_NUMBER =  ... ;
    static final double BOLTZMAN_NUMBER = ... ;
    static final double ELECTRON_MASS = ... ;
}
```

- 상수인터페이스 안티패턴은 인터페이스를 잘못 사용한 예 이다
- 클래스 내부에서 사용하는 상수는 외부 인터페이스가 아니라 내부 구현에 해당하며, 상수 인터페이스를 구현하는것은 이내부 구현을 클래스의 API로 노출하는 행위이다
- 클래스가 어떤 상수 인터페이스를 사용하든간에 사용자에게는 아무런 의미가 없으며, 오히려 사용자에게 더 혼란을 주거나 클라이언트 코드가 내부 구현에 해당하는 이 상수에 종속되게 된다
- 상수를 공개할 목적이라면 더 합당한 선택지가 몇개 있다
- 특정 클래스나 인터페이스와 강하게 연관된 상수라면 그 클래스나 인터페이스 자체에 추가해야 한다. 박싱 클래스 Integer의 MAX_VALUE와 같은것이 그 예이다
- 열거 타입으로 나타내기 적합한 상수라면 열거타입으로 만들어 공개하면 된다
- 인스턴스화 할 수 없는 유틸리티 클래스에 담아 공개하는것도 방법이다

```java
public class PhysicalConstant {
    private PhysicalConstant() { }

    public static final double AVOGADROS_NUMBER =  ... ;
    public static final double BOLTZMAN_NUMBER = ... ;
    public static final double ELECTRON_MASS = ... ;
}
```

- 유틸리티 클래스에 정의된 상수를 클라이언트에서 사용하려면 클래스 이름까지 명시해야 하지만, 빈번히 사용하는 상수라면 정적 임포트를 통해 생략 할 수도 있다
