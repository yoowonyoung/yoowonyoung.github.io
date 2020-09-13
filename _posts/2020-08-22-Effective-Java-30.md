---
layout: post
title: "Effective Java - 아이템30: 이왕이면 제너릭 메서드로 만들어라"
description: 이왕이면 제너릭 메서드로 만들어라
date: 2020-08-22 11:41:00 +09:00
categories: EffectiveJava Study
---


# 제너릭

## 아이템 30 : 이왕이면 제너릭 메서드로 만들어라

- 클래스와 마찬가지로 메서드도 제너릭으로 만들수 있으며, 매개변수화 타입을 받는 정적 유틸리티 메서드는 보통 제너릭이다

```java
public static Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```

- 위 코드는 컴파일은 되지만 경고가 발생한다. 타입 안전하지 않기 떄문이다
- 메서드 선언에서 세 집합의 원소타입을 타입 매개변수로 명시하고 메서드 안에서도 이 타입 매개변수만 사용하도록 하면 안전하다
- 타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 오며, 다음과 같이 사용한다

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```

- 단순한 제너릭 메서드라면 이정도면 충분하고, 이 메서드는 경고없이 컴파일되며 타입 안전하고, 쓰기도 쉽다
- 한정적 와일드 카드를 사용하면 더욱 유연하게 만들수도 있다
- 때때로 불변 객체를 여러 타입으로 활용할 수 있게 만들어야 할 떄가 있다. 제너릭은 런타임에 타입 정보가 소거되므로, 하나의 객체를 어떤 타입으로든 매개변수화 할 수 있다
    * 하지만 이렇게 요청하려면 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리를 만들어야 하며, 이 패턴을 제너릭 싱글턴 팩터리라 한다

- 항등함수를 담은 클래르 만들고 싶다고 할 때, Function.identity를 사용하면 되지만 공부를 위해 한번 만들어보자
- 항등함수 객체는 상태가 없으니 요청할 때마다 새로 정의하는것은 자원낭비이며, 자바의 제너릭이 실체화된다면 항등함수를 타입별로 만들어야햇지만 그게 아니므로 제너릭 싱글턴 하나면 충분하다

```java
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDENTITY_FN;
}
```

- IDENTITY_FN을 UnaryOperator<T>로 형변환 하면 비검사 형변환 경고가 발생한다. T가 어떤타입이든 UnaryOperator<Object>는 UnaryOperator<T>가 아니기 때문이다
- 하지만 항등함수란 입력값 수정없이 그대로 반환하는 븍별한 함수이므로, T가 어떤타입이든 UnaryOperator<T>를 사용해도 안전하다. 그래서 여기서 발생하는 비검사 형변환 경고는 숨겨도 안심할 수 있다

```java
public static void main(String[] args) {
    String[] strings = {"삼베", "대마", "나일론" };
    UnaryOperator<String> sameString = identityFunction();
    for(String s : strings) {
        System.out.println(sameString.apply(s));
    }

    Number[] numbers = { 1, 2.0, 3L };
    UnaryOperator<Number> sameNumber = identityFunction();
    for(Number n : numbers) {
        System.out.println(sameNumber.apply(n));
    }
}
```

- 위 코드는 제너릭 싱글턴을 UnaryOperator<String>과 UnaryOperator<Number>로 활용하는 모습이고, 형변환을 하지 않아도 오류나 경고가 발생하지않는다
- 상대적으로 드물긴 하지만, 자기 자신이 들어간 표현식을 사용해 타입 매개변수의 허용범위를 한정할 수 있다. 이것이 재귀적 타입 한정이라는 개념인데, 이는 주로 타입의 자연적 순서를 정하는 Comparable인터페이스와 함께 쓰인다

```java
public interface Comparable<T> {
    int compareTo(T o);
}
```

- 여기서 타입 매개변수 T는 Comparable<T> 를 구현한 타입이 비교할 수 있는 원소의 타입을 정의한다. 실제로 거의모든 타입은 자기 자신과 같은 타입의 원소와만 비교할수 있으므로, String은 Comparable<String>을 구현하고, Integer는 Comparable<Integer>를 구현하는 형식이다
- Comparable을 구현한 원소의 컬렉션을 입력받는 메서드들은 주로 그원소들을 정렬 혹은 검색하거나, 최댓값이나 최솟값을 구하는 식으로 사용된다. 이 기능을 수행하려면 컬렉션에 담긴 모든 원소가 상호 비교될 수 있어야 하며, 다음은 이 제약을 코드로 표현한 것이다

```java
public static <E extends Comparable<E>> E max(Collection<E> c);
```

- 타입 한정인 ```<E extends Comparable<E>>``` 는 모든 타입 E는 자기 자신과 비교할 수 있다 라는 뜻이다

```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty()) {
        throw new IllegalArgumentException("컬렉션이 비어있음");
    }
    E result = null;
    for(E e: c){
        if(result == null || e.compareTo(result) > 0) 
            result = Objects.requiredNonNull(e);
    }
    return result;
}
```

- 구현은 위와 같으며, 컬렉션의 원소의 자연적 순서를 기준으로 최댓값을 계산하며, 컴파일 오류나 경고는 발생하지 않는다
- 재귀적 타입 한정은 훨씬 복잡해질 가능성이 있지만, 그런일은 잘 일어나지 않으며, 이번 아이템에서 설명한 관용구나 와일드 카드를 사용한 변형, 그리고 아이템2에서 설명한 시뮬레이트한 셀프 타입 관용구를 사용하면 재귀적 타입 한정을 무리없이 다룰 수 있다

