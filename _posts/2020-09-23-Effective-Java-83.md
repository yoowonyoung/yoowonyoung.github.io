---
layout: post
title: "Effective Java - 아이템83: 지연 초기화는 신중히 사용하라"
description: 지연 초기화는 신중히 사용하라
date: 2020-09-23 21:16:00 +09:00
categories: EffectiveJava Study
---


# 동시성

## 아이템 83 : 지연 초기화는 신중히 사용하라

- 지연 초기화는 필드의 초기화 스점을 그 값이 처음 필요할 때 까지 늦추는 기법이다
    * 값이 전혀 쓰이지 않으면 초기화도 일어나지 않으며, 정적 필드와 인스턴스 필드 모두에 사용할 수 있다
    * 주로 최적화 용도로 쓰이지만, 클래스와 인스턴스 초기화때 발생하는 위험한 순환 문제를 해결하는 효과도 있다

- 다른 모든 최적화와 마찬가지로, 지연 초기화에 해줄 최선의 조언은 "필요 할때까지는 하지 마라" 이다
- 지연 초기화는 양날의 검으로, 클래스 혹은 인스턴스 초기화시 비용은 줄지만, 지연 초기화 하는 필드에 접근하는 비용은 커진다
    * 지연 초기화 하려는 필드들 중 초기화가 이뤄지는 비율이나, 초기화에 드는 비용이나, 초기화된 필드를 호출하는 횟수에 따라 오히려 성능이 느려질 수 있다

- 그럼에도 지연 초기화가 필요할 때가 있는데, 해당 클래스의 인스턴스중 그 필드를 사용하는 인스턴스의 비율이 낮은 반면, 그 필드를 초기화 하는 비용이 크다면 지연 초기화가 해법이다
- 멀티 스레드 환경에서는 지연 초기화가 까다로운데, 지연 초기화 하려는 필드를 둘 이상의 스레드가 공유한다면, 어떤 형태로든 반드시 동기화를 해야한다
- 대부분의 상황에서는 일반적인 초기화가 지연 초기화 보다 낫다

```java
private final FieldType field = computeFieldValue();
```

- 위의 코드는 인스턴스 필드를 선언할 때 수행하는 일반적인 초기화의 모습이다. final을 사용했음에 주목해야 한다
- 지연 초기화가 초기화 순환성을 깨트릴것 같다면 synchronized를 단 접근자를 사용하자. 이것이 가당 간단하고 명확한 대안이다

```java
private FieldType field;

private synchronized FieldType getField() {
    if (field == null)
        field = computeFieldValue();
    return field;
}
```

- 이러한 지연 초기화는 정적 필드에도 똑같이 적용되며, 필드와 접근자 메서드 선언에 static 한정자만 붙여주면 된다
- 성능 때문에 정적 필드를 지연 초기화 해야한다면 지연 초기화 홀더 클래스 관용구를 사용하자. 클래스가 처음 쓰일때 비로소 초기화 된다는 특성을 이용한 관용구이다

```java
private static class FieldHolder {
    static final FieldType field = computeFieldValue();
}

private static FieldType getField() { return FieldHolder.field; }
```

- getField가 처음 호출되는 순간 FieldHolder.field가 처음 읽히면서 비로소 FieldHolder 클래스 초기화를 촉발한다. 이 관용구의 멋진점은 getField 메서드가 필드에 접근하면서 동기화를 전혀 하지 않으니 성능이 느려질거리가 전혀 없다는것이다
- 성능 때문에 인스턴스 필드를 지연 초기화 해야한다면 이중검사 관용구를 사용하라. 이 관용구는 초기화된 필드에 접근할 때의 비용을 없애준다. 이름에서 알 수 있듯이 필드의 값을 두번 검사하는 방식으로, 한번은 동기화 없이 검사하고, 두번째는 동기화 하여 검사한다. 필드가 초기화 된 이후로는 동기화 하지 않으므로 해당 필드는 반드시 volatile로 선언해야 한다

```java
private volatile FieldType field;

private FieldType getField() {
    FieldTpe result = field;
    if(result != null) 
        return result;
    
    synchronized(this) {
        if(field == null)
            field = computeFieldValue();
        return field;
    }
}
```

- 코드가 다소 난해 할 수 있다. result라는 지역 변수가 필요한 이유는, 필드가 이미 초기화 된 상황에서 그 필드를 딱 한번만 읽도록 보장하는 역할을 한다. 반드시 필요하지는 않지만, 성능을 높여주고 저수준 동시성 프로그래밍에도 표준적으로 적용되는 우아한 방식이다
- 이중 검사를 정적 필드에 적용할 수도 있지만, 굳이 그럴 이유는 없다. 지연 초기화 홀더 클래스 방식이 더 낫다
- 이중 검사에는 언급 해둘만한 두가지 변종이 두가지 있다
    * 반복해서 초기화 해도 상관없는 인스턴스 필드를 지연 초기화 할 때가 있는데, 이런 경우라면 두번째 검사를 생략 할 수 있다. 이런 경우 단일 검사 관용구가 되지만 여전히 volatile을 사용한다
    * 모든 스레드가 필드의 값을 다시 계산해도 상관없고, 필드의 타입이 long과 double을 제외한 기본 타입이라면, 단일 검사의 필드 선언에서 volatile 한정자를 없애도 된다. 이 변종은 짜릿한 단일 검사 관용구라 불린다. 보통은 거의 쓰이지 않는다

```java
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if(result == null)
        field = result = computeFieldValue();
    return result;
}
```

- 