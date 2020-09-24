---
layout: post
title: "Effective Java - 아이템90: 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토해라"
description: 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토해라
date: 2020-09-24 22:45:00 +09:00
categories: EffectiveJava Study
---


# 직렬화

## 아이템 90 : 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토해라

- 직렬화를 하면 정상적인 매커니즘인 생정자 이외의 방법으로 인스턴스를 생성할 수 있게된다. 버그와 보안 문제가 일어날 수 있다는 뜻이다. 이 위험을 줄여주는 기법이 직렬화 프록시 패턴이다
- 직렬화 프록시 패턴은 그리 복잡하지 않다
- 먼저 바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스를 설계해 private static으로 선언한다. 이 중첩 클래스가 바로 바깥 클래스의 직렬화 프록시이다. 중첩 클래스의 생성자는 단 하나여야하며, 바깥 클래스를 매개변수로 받아야 한다
    * 이 생성자는 단순히 인수로 넘어온 인스턴스의 데이터를 복사한다. 일관성 검사나 방어적 복사도 필요 없다
    * 설계상, 직렬화 프록시의 기본 직렬화 형태는 바깥 클래스의 직렬화 형태로 쓰기에 이상적이다. 그리고 바깥 클래스와 직렬화 프록시 모두 Serializable을 구현한다고 선언해야 한다
    * 그 후 바깥 클래스에 writeReplace 메서드를 추가한다. 이 메서드는 자바의 직렬화 시스템이 바깥 클래스의 인스턴스 대신 SerailizationProxy의 인스턴스를 반환하며, 직렬화가 이뤄지기전에 바깥 클래스의 인스턴스를 직렬화 프록시로 변환 해주는 것이다. 이 메서드는 범용적이니 모든 직렬화 프록시에 적용 가능하다

```java
private static class SerailizationProxy implements Serializable {
    private final Date start;
    private final Date end;

    SerailizationProxy(Perid p) {
        this.start = p.start;
        this.end = p.end;
    }

    private static final long SerialVersionUID = ...;
}

private Object writeReplace() {
    return new SerailizationProxy(this);
}
```

- writeReplace 덕분에 직렬화 시스템은 결코 바깥 클래스의 직렬화 인스턴스를 생성해 낼 수 없다. 하지만 공격자는 불변식을 훼손하고자 이런 시도를 할 수 있기 때문에, 다음의 readObject 메서드를 바깥 클래스에 추가해 이런 공격을 간단히 막아내보자

```java
private void readObject(ObjectInputStream stream) throws InvalidObjectException {
    throw new InvalidObjectException("프록시가 필요합니다");
}
```

- 마지막으로, 바깥 클래스와 논리적으로 동일한 인스턴스를 반환하는 readResolve메서드를 SerailizationProxy 클래스에 추가한다. 이 메서드는 역직렬화시에 직렬화 시스템이 직렬화 프록시를 다시 바깥 클래스의 인스턴스로 변환하게 해준다

```java
private Object readResolve() {
    return new Period(start,end);
}
```

- readResolve는 공개된 API만을 사용해 바깥 클래스의 인스턴스를 생성 해내는데, 이 패턴이 아름다운 이유가 이것이다. 일반 인스턴스를 만들떄와 똑같은 생성자, 정적팩터리, 혹은 다른 메서드를 사용해 역 직렬화된 인스턴스를 생성하는 것이다. 따라서 역직렬화된 인스턴스가 해당 클래스의 불변식을 만족하는지 검사할 또다른 수단을 강구하지 않아도 된다
- 방어적 복사처럼 직렬화 프록시 패턴은 가짜 바이트 스트림 공격을 내부 필드 탈취 공격을 프록시 수준에서 차단해준다. 또한 직렬화 프록시는 Period의 필드를 final로 선언해도 되므로 Period를 진정한 불변으로 만들 수 있다
- 직렬화 프록시 패턴이 readObject에서의 방어적 복사보다 강력한 경우가 하나 더 있는데, 직렬화 프록시 패턴은 역직렬화한 인스턴스와 워래의 직렬화된 인스턴스의 클래스가 달라도 정상 동작한다
    * EnumSet의 사례를 생각 해보면, 이 클래스는 public 생성자 없이 정적 팩터리들만 제공한다
    * 클라이언트 입장에서는 이 팩터리들이 EnumSet 인스턴스를 반환하는것으로 보이지만, 현재의 OpenJDK는 열거 타입의 크기에 따라 열거 타입의 원소가 64개 이하면 RegularEnumSet을 사용하고, 그보다 크면 JumboEnumSet을 사용해 반환한다
    * 원소 64개짜리 열거타입을 가진 EnumSet을 직렬화 한다음 원소 5개를 추가하고 역직렬화 하면 어떻게 될까?
    * 처음 직렬화된것은 RegularEnumSet이지만 역직렬화는 JumboEnumSet 인스턴스로 하면 좋을것이다
    * 이러한것은 직렬화 프록시를 통해 가능하다

```java
private static class SerializationProxy <E enxtends Enum<E>> implements Serializable {
    private final Class<E> elementType;
    private final Enum<?>[] elements;

    SerializationProxy(EnumSet<E> set) {
        elementType = set.elementType;
        elements = set.toArray(new Enum<?>[0]);
    }    

    private Object readResolve() {
        EnumSet<E> result = EnumSet.noneOf(elementType);
        for(Enum<?> e : elements)
            result.add((E)e);
        return result;
    }

    private static final long serialVersionUID = ...;
}
```

- 직렬화 프록시 패턴에는 한계가 두가지 있다
    * 클라이언트가 멋대로 확장할 수 있는 아이템에는 적용할 수 없다
    * 직렬화 프록시 패턴이 주는 강력함과 안정성에도 대가가 따른다. 느려진다.
