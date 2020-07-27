---
layout: post
title: "Effective Java - 아이템3: Private 생성자나 열거 타입으로 싱글턴임을 보증하라"
description: Private생성자 혹은 열거타입으로 싱글턴 보증
date: 2020-07-27 20:39:00 +09:00
categories: EffectiveJava Study
---


# 객체의 생성과 파괴

## 아이템 3 : Private 생성자나 열거 타입으로 싱글턴임을 보증하라

- 싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다
- 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트 하기가 어려워진다
    * 타입을 인터페이스로 정의한 다음, 그 인터페이스를 구현해서 만든 싱글턴이 아니면 Mock으로 대체할 수 없기 때문

- 싱글턴은 보통 2가지 방식으로 만드는데, 두 방식 모두 생성자는 private로 감춰두고, 유일한 인스턴스에 접근 할 수 있는 수단으로 public static 멤버를 하나 마련 해둔다
    * public static 멤버가 final인 방식, private 생성자는 public static final 필드인 Elvis.INSTANCE를 초기화 할 때 딱 한번만 호출된다
        + 해당 클래스가 싱글턴임이 API에 명백히 드러나며, 간결하다
    * 정적 팩터리 메서드를 public static멤버로 제공, Elvis.getInstance는 항상 같은 객체의 참조를 반환 하므로, 제 2의 Elvis 인스턴스란 결코 만들어지지 않는다
        + API를 바꾸지 않더라도 싱글턴이 아니도록 바꿀 수 있다
        + 정적 팩터리를 제너릭 싱글턴 팩터리로 만들 수 있다
        + 정적 팩터리의 메서드 참조를 공급자로 사용 할 수 있다

```java
// public static 멤버가 final
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public void leaveTheBuilding() { ... }
}

// 정적 팩터리 메서드
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() = { return this.INSTANCE; }
    public void leaveTheBuilding() { ... }
```

- 싱글턴 클래스를 직렬화 하기 위해서는 단순히 Serializable을 구현 한다고 선언 하는것만으로는 부족하며, 모든 인스턴스를 일시적(transient)이라고 선언 하고, readResolve 메서드를 제공 해야한다
    * 이렇게 하지 않으면 매 역직렬화시마다 새로운 인스턴스가 만들어진다
- 세번째 싱글턴을 만드는 방법은 원소가 하나인 열거 타입을 선언 하는 것이다
    * public 필드 방식과 비슷 하지만, 더 간결하고, 추가 노력 없이 직렬화 할 수 있고, 아주 복잡한 직렬화 상황이나, 리플렉션 공격에도 제2의 인스턴스가 생기는 일을 완벽히 막아준다
    * 대부분의 상황에서 원소가 하나뿐인 결거 타입이 싱글턴을 만드는 가장 좋은 방법이다
    * 단, 만들려는 싱글턴이 Enum이외의 클래스를 상속해야 한다면, 이 방법은 사용 할 수 없다

```java
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() { ... }
}
```
