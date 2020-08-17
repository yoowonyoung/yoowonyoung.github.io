---
layout: post
title: "Effective Java - 아이템26: 로 타입은 사용하지 말라"
description: 로 타입은 사용하지 말라
date: 2020-08-17 20:45:00 +09:00
categories: EffectiveJava Study
---


# 제너릭

## 아이템 26 : 로 타입은 사용하지 말라

- 클래스와 인터페이스 선언에 타입 매개변수가 쓰이면 이를 제너릭 클래스 혹은 제너릭 인터페이스라 한다. 제너릭 클래스와 제너릭 인터페이스를 통틀어 제너릭 타입 이라고 한다
- 각각의 제너릭 타입은 일련의 매개변수화 타입을 정의한다. 먼저 클래스 이름이 나오고, 이어서 꺽쇠괄호 안에 실제 타입 매개변수들을 나열한다
- 마지막으로 제너릭 타입을 하나 정의하면 그에 딸린 로 타입도 함께 정의되는데, 로타입이란 제너릭 타입에서 타입 매개변수를 전혀 사용하지 않을때를 의미한다
- 로타입은 타입선언에서 제너릭 타입 정보가 전부 지워진것처럼 동작하는데, 제너릭이 도래하기 전 코드와 호환도도록 한 궁여지책이라고 할 수 있다

```java
private final Collection stamps = ...;
stamps.add(new Coin(...));

for(Iterator i = stamps.iterator(); i.hasNext();) {
    Stamp stamp = (Stamp) i.next(); //오류 발생
    stamp.cancel();
}
```

- 오류는 가능한 발생 즉시, 이상적으로는 컴파일 타임에 발견하는게 좋다. 런타임에 발견되는 에러는 에러를 만든 코드와 에러를 발생한 코드가 물리적으로 상당히 떨어져 있을 수 있기 때문이다
- 제너릭을 활용하면 이 정보가 주석이 아닌 타입 선언 자체에 녹아든다

```java
private final Collection<String> stamps = ...;
```

- 위와같이 선언 하면, 컴파일러가 잘못된 타입을 넣는것을 막아준다. 또한 컬렉션에서 원소를 꺼내는 모든곳에 보이지 않는 형변환을 추가하여 절대 실패하지 않읆을 보장한다
- 위처럼 로타입을 쓰는걸 언어차원에서 막아두지는 않았지만, 절대 쓰면 안된다. 제너릭이 주는 모든 안정성과 표현력을 잃기 때문이다
- List와 같은 로 타입은 사용해서는 안되나, List<Object>와 같은 임의의 객체를 허용하는 매개변수화 타입은 괜찮다
- 이 둘의 차이는 명확히 다른데, 로 타입은 제너릭 타입에서 완전히 발을 뺸것이고, List<Object> 는 모든 타입을 허용한다라는 의미를 컴파일러에 명확히 전달한 것이다
- List를 매개변수로 받는 메서드에 List<String>은 넘길 수 있으나, List<Object>를 매개변수로 받으면 넘길 수 없다
- List<String>은 List의 하위타입이지만, List<Object>은 아니다. 이는 제너릭의 하위타입 규칙 떄문인데, 이처럼 로타입을 사용하면 안정성을 잃게된다

```java
public static void main(String[] args) {
    List<String> strings = new ArrayList<>();
    unsafeAdd(strings, Integer.valueOf(42));
    String s = strings.get(0);
}

private static void unsafeAdd(List list, Object o) {
    list.add(o);
}
```

- 이 코드는 경고는 발생하지만 컴파일은 되며, 실행시 strings.get(0)를 할 때 ClassCaseException을 발생 시킨다
- 원소의 타입을 모르는채로 사용하고 싶을 경우, 비한정 와일드카드 타입을 사용 하는게 좋다. 실제 타입 매개변수가 무엇인지 신경 쓰고 싶지 않다면 ? 를 사용해라

```java
static int numElementsInCommon(Set s1, Set s2) // 사용하지 말것
static int numElementsInCommon(Set<?> s1, Set<?> s2) // Set<E> 의 비한정적 와일드카드 타입 Set<?>
```

- 와일드 카드 타입은 안전하고, 로 타입은 안정하지 않은데, 로 타입 컬렉션에는 아무것이나 넣을 수 있지만, Collection<?>에는 null을 제외한 어떠한 원소도 넣을 수 없다
- 로 타입을 쓰지 말라는 예외에도 소소한 예외가 몇개 있는데, class 리터럴에는 로 타입을 써야한다. 예를들어 List.class, String[].class 는 허용허지만, List<String>.class는 안된다
- 런타임에는 제너릭 타입 정보가 지워지므로, instanceof연사자는 비한정적 와일드카드 타입 이외의 매개변수화 타입에는 사용 할수가 없고, 로타입이든 와일드 카드던 똑같이 동작한다

```java
if (o instanceof Set) {
    Set<?> s = (Set<?>) o;
}
```