---
layout: post
title: "Effective Java - 아이템54: null이 아닌 빈 컬렉션이나 배열을 반환하라"
description: null이 아닌 빈 컬렉션이나 배열을 반환하라
date: 2020-09-06 17:17:00 +09:00
categories: EffectiveJava Study
---


# 메서드

## 아이템 54 : null이 아닌 빈 컬렉션이나 배열을 반환하라

```java
private final List<Cheese> cheesesInStock = ...;

public List<Cheese> getCheese() {
    return cheesesInStock.isEmpty() ? null
        : new ArrayList<>(cheesesInStock);
}
```

- 위는 주변에서 흔히 볼 수있는 메서드이다. 사실 재고가 없다고(리스트가 비어있다고) 해서 특별히 취급할 이유는 없다. 그럼에도 이 코드처럼 null을 반환한다면, 클라이언트는 이 null 상황을 처리하는 코드를 추가 작성해야한다

```java
List<Cheese> cheese = shop.getCheeses();
if(cheeses != null && cheeses.contains(Cheese.STILTON))
    System.out.println("Yes it is");
```

- 컬렉션이나 배열같은 컨테이너가 비었을때 null을 반환하는 메서드를 사용할때면, 항시 이와 같은 방어 코드를 넣어줘야 한다. 클라이언트에서 방어 코드를 빼먹으면 오류가 발생할 수 있다
    * null을 반환하려면 반환하는 쪽에서도 이 상황을 특별하게 취급해줘야 해서 코드가 더 복잡해진다

- 때로는 빈 컨테이너를 할당하는데도 비용이드니 null을 반환하는쪽이 낫다는 주장이 있는데 이는 틀린 주장이다
    * 성능분석결과 이 할당이 성능 저하의 주범이라고 확인되지 않는한, 이정도의 성능차이는 신경쓸 수준이 못된다
    * 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다. 대부분의 상황에서는 다음과 같이 작성하면 된다

```java
public List<Cheese> getCheeses() {
    return new ArrayList<>(cheesesInStock);
}
```

- 가능성은 작지만, 사용 패턴에 따라 빈 컬렉션 할당이 성능을 눈에띄게 떨어트릴수도 있다. 다행이 해법은 간단한데, 매번 똑같은 빈'불변' 컬렉션을 반환한다. 불변 객체는 공유해도 안전하기 때문이다
    * Collections.emptyList 메서드가 그러한예이다. 집합이 필요하면 Collections.emptySet을 맵이 필요하면 Collections.emptyMap을 사용하면 된다
    * 단, 이 역시 성능 최적화에 해당하니 꼭 필요할때에만 사용해야하며, 최적화 전후의 성능 측정을해서 성능이 개선되었는지 확인해봐야 한다

```java
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? Collections.emptyList()
        : new ArrayList<>(cheesesInStock);
}
```

- 배열을 쓸때도 마찬가지이다. 절대 null을 반환하지말고, 길이가 0인 배열을 반환하라. 보통은 단순히 정확한 길이의 배열을 반환하기만 하면 된다, 그 길이가 0일 수도 있을 뿐인것이다

```java
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);
}
```

- 위 코드에서 toArray메서드에 건넨 길이 0짜리 배열이 우리가 원하는 반환타입을 알려주는 역할을 한다
- 이 방식이 성능을 떨어트릴것 같다면, 길이 0짜리 배열을 미리 선언해두고, 매번 그 배열을 반환하면 된다. 길이가 0인 배열은 불변이기 떄문이다

```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

- 이 최적화 버전의 getCheeses는 항상 EMPTY_CHEESE_ARRAY를 인수로 넘겨 toArray를 호출한다. 따라서 cheesesInStock이 비어있을따면 언제나 EMPTY_CHEESE_ARRAY를 반호나하게 된다
- 단순히 성능을 개선할 목적이라면 toArray에 넘기는 배열을 미리 할당하는건 추천하지 않는다