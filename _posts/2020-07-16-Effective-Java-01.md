---
layout: post
title: "Effective Java - 02 객체의 생성과 파괴 - 생성자 대신 정적 팩터리 메서드"
description: 2장 객체의 생성과 파괴
date: 2020-07-16 20:22:00 +09:00
categories: EffectiveJava Study
---


# 객체의 생성과 파괴

## 아이템 1 : 생성자 대신 정적 팩터리 메서드를 고려하라

- 클라이언트가 클래스의 인스턴스를 얻는 전통적인 수단은 public 생성자
- 하지만 클래스 생성자와 별도로 정적 팩터리 메서드(Static Factory Method)를 제공 할 수 있다
- 다음 코드는 boolean의 Wrapper class인 Boolean에서 발췌 한것이다

```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

- 정적 팩터리 메서드는 5가지 장점이 있다
    * 이름을 가질 수 있다. 생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 설명하기 힘들지만,
    정적 팩터리 메서드는 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사 할 수 있다 (ex> 소수를 반환하는 BigInteger.probablePrime)
    또한, 같은 시그니처를 가지는 여러 생성자가 필요 할 경우, 생성자를 정적 팩터리 메서드로 바꾸고 각각의 차이를 드러내는 이름으로 지어주면 된다
    * 호출 될 때마다 인스턴스를 생성하지는 않아도 된다. 이를 이용해 불변 클래스는 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱 해서 재활용 하는 형식으로 불필요한 객체 생성을 피할 수 있다
    이런 클래스 인스턴스 통제를 통해, 클래스를 싱글턴으로 만들수도, 인스턴스화 불가 하게 만들수도 있다. 또한 불변 값 클래스에서 동치인 인스턴스가 단 하나임을 보장 할 수 있다
    * 반환 타입의 하위 타입객체를 반환 할 수 있는 능력이 있다. 이로 인해 반환할 객체의 클래스를 자유롭게 선택 할 수 있는 유연성을 가질 수 있다.
    API를 만들떄 이것을 이용해 구현 클래스를 공개하지 않고도 그 객체를 반환 할 수 있어 API를 작게 유지 할 수 있다(Java8 이상에서만 가능)
    * 입력 매개변수에 따라 매던 다른 클래스의 객체를 반환 할 수 있다. 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관이 없다.
    * 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다. 이러한 유연함은 서비스 제공자 프레임워크를 만드는 근간이 된다.
    이러한 서비스 제공자 프레임워크의 대표적인 예가 JDBC이다

- 하지만 2가지 단점도 있다
    * 상속을 하려면 public 이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
    이 제약은 상속보다 컴포지션을 활용하도록 유도하고, 불변 타입으로 만들려면 이 제약을 지켜야 한다는점에서 오히려 장점이라고 볼 수도 있다
    * 정적 팩터리 메서드는 프로그래머가 찾기 어렵다. API설명에 명확하게 드러나지 않기 떄문에 다음과 같은 명명규칙을 활용한다
        + from: 매개변수 하나를 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드 (ex> Date d = Date.from(instant);)
        + of: 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드 (ex> Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);)
        + valueOf: from과 of의 더 자세한 버전 (ex> BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);)
        + instance 혹은 getInstance: (매개변수를 받는다면) 매개변수로 명시한 인스턴스를반환 하지만, 같은 인스턴스임을 보장하지는 않는다 (StackWalker luke = StackWalker.getInstance(options);)
        + create 혹은 newInstance: instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스를 반환함을 보장한다 (EX> Object newArray = Array.newInstance(classObject, arrayLen);)
        + getType: getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다 (ex> FileStore fs = Files.getFileStore(path);)
        + newType: newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 떄 쓴다 (ex> BufferedReader br = Files.newBufferedReader(path);)
        + type: getType과 newType의 간결한 버전 (ex> List<Complaint> litany = Collections.list(legacyLitany);)

- 정적 팩터리 메서드와 public 생성자는 각각의 쓰임새가 있으니, 상대적인 장단점을 이해하고 사용하는것이 좋다. 그렇다고 하더라도 정적 팩터리 메서드를 사용하는게 유리한 경우가 더 많으므로 무작정 public 생성자를 제공했다면 고쳐라