---
layout: post
title: "Effective Java - 아이템28: 배열보다는 리스트를 사용하라"
description: 배열보다는 리스트를 사용하라
date: 2020-08-18 18:52:00 +09:00
categories: EffectiveJava Study
---


# 제너릭

## 아이템 28 : 배열보다는 리스트를 사용하라

- 배열과 제너릭 타입에는 중요한 두가지 차이가 있다
- 첫번째로 배열은 공변(함께 변함)이지만 제너릭은 불공변이다. Sub가 Super의 하위타입이라면 Sub[]도 Super[]의 하위타입이지만, 서로다른 타입 Type1, Type2에 대한 List<Type1>, List<Type2> 는 어떠한 관계도 없다

```java
Object[] objectArray = new Long[1];
ObjectArray[0] = "런타임 에러"; //런타임에 ArrayStoreException 발생

List<Object> ol = new ArrayList<Long>();
ol.add("타입이 달리요");// 컴파일이 되지 않는다
```

- 어느쪽도 Long용 저장소에 String이 저장되지는 않지만, 배열에서는 그 실수가 런타임에 발견되지만 리스트에서는 컴파일 타임에 알아챌 수 있다
- 두번째 차이로 배열은 실체화가 되는데, 배열은 런타임에도 원소의 타입을 인지하고 확인하지만 제너릭에서는 타입 정보가 런타임에 소거된다. 원소 타입을 컴파일 타임에만 검사하면 런타임에는 알수없단 뜻이다
- 이러한 차이로 인해 배열과 제너릭은 잘 어울리지 못하는데, 즉 배열은 제너릭타입, 매개변수화 타입, 타입 매개변수로 사용할 수없다

```java
List<String> stringLists = new List<String>[1];
List<Integer> intList = List.of(42);
Object[] objects = stringLists;
objects[0] = intList;
String s = stringList[0].get(0); 
```

- 위 코드는 컴파일이 되지 않는다. 컴파일이 된다고 하더라도 마지막 라인에서 ClassCastException이 발생할 것이다. String만 저장하기로된 리스트에 List<Integer> 인스턴스가 저장되있기 때문이다
- E, List<E>, List<String>과 같은 타입을 실체화 불가 타입이라고 한다, 실체화 되지 않아 런타임에는 컴파일타임보다 적은 정보를 가진다
- 매개변수화 타입 가운데 실체화가 될 수 있는 타입은 List<?> 와 Map<?,?>와 같은 비한정적 와일드카드 뿐이다
- 제너릭 컬렉션에서는 자신의 원소타입을 담은 배열을 반환하는게 보통은 불가능해서 이를 해결할 아이템은 아이템 33에서 소개할 것이다
- 제너릭과 가변인수 메서드를 호출할떄에는 해석하기 어려운 경고메시지를 받게 되는데, 가변인수 메서드를 호출 할 때마다 가변인수 매개변수를 담을 배열이 하나 만들어지고, 그 배열의 원소가 실체화 불가라면 경고가 발생하는 것이다
- 배열로 형변환 할 떄, 제너릭 배열 생성오류나 비검사 형변환 경고가 뜨는 경우는 대부분 배열인 E[] 대신 List<E>를 쓰면 해결된다. 조금은 복잡해지고 성능이 낮아지지만, 안정성이 생긴다

```java
public class Chooser {
    private final Object[] choiceArray;

    public Chooser(Collection choices) {
        choiceArray = choices.toArray();
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```

- 이 클래스를 사용하려면 choose 메서드를 호출 할 때마다 반환된 Object를 원하는 타입으로 형변환 해야한다. 타입이 다른 원소가 들어있을 경우 런타임에 형변환 오류가 발생할것이다

```java
public class Chooser<T> {
    private final T[] choiceArray;

    public Chooser(Collection<T> choices) {
        choiceArray = (T[])choices.toArray();
    }
}
```

- 위처럼 제너릭을 조금 활용한 코드로 변경할 경우 경고가 뜰 것이다. T가 무슨타입인지 알 수 없으니 컴파일러가 이 형변환이 런타임에도 안전한지 보장할수 없다는 이야기 이다
- 하지만 이 코드는 컴파일러가 안전을 보장하지 못할 뿐이지 동작하며, 이 코드를 작성한사람이 안전하다고 확신한다면 주석을 남기고 어노테이션 처리함으로 경고를 제거할 수 있다

```java
public class Chooser<T> {
    private final List<T> choiceArray;

    public Chooser(Collection<T> choices) {
        choiceArray = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray.get(rnd.nextInt(choiceArray.size()));
    }
}
```

- 비검사 형변환 경고까지 제거하려면 배열대신 리스트를 쓰면 된다. 위의 코드는 오류나 경고없이 컴파일 된다
- 코드양은 조금 늘고 조금은 더 느릴수 있지만, 런타임에 ClassCastException을 만날리는 없으니 그럴만한 가치가 있다
