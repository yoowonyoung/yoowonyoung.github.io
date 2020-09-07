---
layout: post
title: "Effective Java - 아이템58: 전통적인 for문 보다는 for-each문을 사용하라"
description: 전통적인 for문 보다는 for-each문을 사용하라
date: 2020-09-07 23:40:00 +09:00
categories: EffectiveJava Study
---


# 일반적인 프로그래밍 원칙

## 아이템 58 : 전통적인 for문 보다는 for-each문을 사용하라

- 아이템 45에서 이야기했듯, 스트림이 제격인 작업이 있고, 반복이 제격인 작업이 있다. 다음은 전통적인 for문으로 컬렉션과 배열을 순회하는 코드다

```java
for(Iterator<Element> i = c.iterator(); i.hasNext();) {
    Element e = i.next();
    ...
}

for(int i = 0; i < a.length; i++) {
    ...
}
```

- 이 관용구들은 while문보다는 낫지만 가장 좋은 방법은 아니다. 반복자와 인덱스 변수는 모드 코드를 지저분하게 할 뿐 아니라, 우리에게 진짜 필요한건 원소들 뿐이다
- 더군다나 이처럼 쓰이는 요소 종류가 늘어나면 오류가 생길 가능성이 높아진다. 1회 반복에서 반복자는 세번 등장하며, 인덱스는 네번이나 등장하여 변수를 잘못 사용할 틈새가 넓어진다. 혹시라도 잘못된 변수를 사용했을때 컴파일러가 잡아주리란 보장도 없다. 마지막으로 컬렉션이냐 배열이냐에 따라 코드도 크게 달라진다
- 이상의 문제는 for-each 문을 사용하면 모두 해결된다. 참고로 for-each문의 정식적인 이름은 향상된 for문 이다. 반복자와 인덱스 변수를 사용하지 않으니 코드가 깔끔해지고, 오류가 날 일도 없다. 하나의 관용구로 컬렉션과 배열을 모두 처리할 수 있어 어떤 컨테이너를 다루는지는 신경쓰지 않아도 된다

```java
for(Element e : elements) {
    ...
}
```

- 여기서 : 는 in 으로 읽으면 되며, 이 반복문은 "elements의 각 원소 e에 대해" 라고 읽는다. 반복 대상이 컬렉션이든 배열이든 for-each문을 사용해도 속도는 그대로이다
- 컬렉션을 중첩해 순회해야 한다면 for-each문의 이점이 더욱 커지는데, 먼저 다음과 같은 코드에서 버그를 찾아보자

```java
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING }

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for(Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
    for(Iterator<Rank> j = ranks.iterator(); j.hasNext(); ) {
        deck.add(new Card(i.next(), j.next()));
    }
}
```

- 여기서 문제는 마지막 줄의 i.next 때문에 바깥 컬렉션의(suit) 반복자에서 next가 너무 많이 불린다는것이다. 이 next는 숫자 하나당 한번씩만 불려야 하는데, 안쪽의 반복문에서 호출되는 바람에 카드 하나당 한번씩만 불리고 있다. 그래서 숫자가 바닥나면 반복문에서 NoSuchElementException을 던진다
- 정말 운이 나빠서 바깥 컬렉션의 크기가 안쪽 컬렉션의 크기의 배수라면 이 반복문은 예외를 던지지 않고 종료한다. 우리가 원하는 일을 수행하지 않은채 말이다
- for-each문을 중첩하는것으로 이 문제는 간단히 해결된다. 코드도 놀랄만큼 간결해진다

```java
for(Suit suit : suits) {
    for(Rank rank : ranks) {
        deck.add(new Card(suit, rank));
    }
}
```

- 하지만 안타깝게도 for-each문을 사용할 수 없는 세가지 상황이 존재한다
    * 파괴적인 필터링: 컬렉션을 순회하면서 선택된원소를 제거해야 한다면, 반복자의 remove를 호출해야한다. 자바8 부터는 Collection의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는 일은 피할수 있다
    * 변형: 리스트나 배열을 순회하면서 그 원소의값 일부 혹은 전체를 교체해야 한다면, 리스트의 반복자나 배열의 인덱스를 사용해야한다
    * 병렬반복: 여러 컬렉션을 병렬로 순회해야한다면, 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다
    * 위의 세가지 상황중 하나에 속한다면 일반적인 for문을 사용하되, 이번 아이템에서 언급한 문제들을 주의해야 한다

- for-each문은 컬렉션과 배열은 물론 Iterable 인터페이스를 구현한 객체라면 무엇이든지 순회 할 수 있다. Iterable인터페이슨느 다음과 같이 메서드가 단 하나뿐이다

```java
public interface Iterable<E> {
    Iterator<E> iteator();
}
```

- Iterable을 처음부터 구현하기란 까다롭지만, 원소들의 묶음을 표현하는 타입을 작성해야 한다면, Iterable을 구현하는 쪽으로 고민해보길 바란다. 해당 타입에서 Collection인터페이스를 구현하지 않기로 했더라도 말이다