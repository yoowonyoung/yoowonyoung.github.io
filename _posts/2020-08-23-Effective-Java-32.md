---
layout: post
title: "Effective Java - 아이템32: 제너릭과 가변인수를 함께 쓸때는 신중해라"
description: 제너릭과 가변인수를 함께 쓸때는 신중해라
date: 2020-08-23 10:41:00 +09:00
categories: EffectiveJava Study
---


# 제너릭

## 아이템 32 : 제너릭과 가변인수를 함께 쓸때는 신중해라

- 가변인수 메서드와 제너릭은 모두 자바5때 같이 추가되었으나, 서로 잘 어울리지 못한다
- 가변인수 메서드를 호출하면 가변인수를 담기위한 배열이 자동으로 하나 만들어진다. 이 배열은 내부로 감춰지지 않고 클라이언트에 노출된다. 따라서 가변인수 매개변수에 제너릭이나 매개변수화 타입이 포함되면 알기 어려운 컴파일 경고가 발생한다
- 실체화 불가 타입은 런타임에는 컴파일타임보다 적은 타입관련 정보를 담고있고, 거의 모든 제너릭과 매개변수화 타입은 실체화 되지 않는다. 메서드 선언시 실체화 불가 타입으로 가변인수 매개변수를 선언하면 컴파일러가 다음과 같은 경고를 보낼것이다

```
warning: [uncheked] Possible heap pollution form
    parameterized vararg type List<String>
```

- 매개변수화 타입이 다른 타입 객체를 참조하면 힙 오염이 발생하는데, 이런 상황에서는 컴파일러가 자동 생성한 형변환이 실패할 수 있으니, 제너릭의 타입 안전성의 근간이 흔들리게 되어버린다

```java
static void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringLists;
    object[0] = intList;//힙오염 발생
    String s = stringList[0].get(0);//Class cast exception
}
```

- 이 메서드에서는 형변환 하는곳이 보이지가 않음에도 인수를 건네 호출하면 ClassCastException을 던진다. 마지막줄에 컴파일러가 생성한 보이지 않는형변환이 숨어있기 때문이다. 이처럼 타입 안정성이 꺠지니 제너릭 가변인수 매개변수에 값을 저장하는건 안전하지 않다
- 제너릭 배열을 프로그래머가 직접 선언하는것은 허용하지 않으며, 제너릭 가변인수 매개변수를 받는 메서드를 선언 가능하게 해둔것은 실무에서 유용하기 때문이다
- 실제로 자바 라이브러리에서도 이런 메서드를 몇개 제공하는데, Arrays.asList(T... a), Collections.addAll(Collection<? super T> c, T... elements) 와 같은것들이 대표적이며, 그래도 이들은 타입 안전하다
- 자바7 이전에는 메서드의 작성자가 호출자쪽에서 발생하는 경고에 대해 해줄것이 없었지만, 자바7에서 ```@SafeVarargs``` 가 추가됨으로써, 메서드 작성자가 그 메서드가 타입 안전함을 보장할 수 있게 되었다
- 역으로 메서드가 안전한게 확실치 않다면 ```@SafeVarargs``` 을 사용하면 안된다
- 메서드가 안전한지 확인하는 방법은, 가변인수 매개변수를 담는 제너릭 배열이 만들어질때 메서드가 이 배열에 아무것도 저장하지 않고 그 배열의 참조가 바깥으로 노출되지 않는다면 타입 안전한것이다
    * 즉, 가변인수 매개변수 배열이 호출자로부터 그 메서드로 순수하게 인수들을 전달하는 일로만 쓰이면 안전한것이다

- 하지만 다음과 같은 예시에서는 가변인수 매개변수 배열에 아무것도 저장하지 않고도 타입 안정성이 깨질수 있으므로 주의해야 한다

```java
static <T> T[] pickTwo(T a, T b, T c) {
    switch(ThreadLocalRandom.current().nextInt(3)) {
        case 0 : return toArray(a, b);
        case 1 : return toArray(a ,c);
        case 2 : return toArray(b ,c);
    }
    throw new AssertionError();
}

public static void main(String args[]) {
    String attributes = pickTwo("좋은", "빠른", "저렴한");
}
```

- 이 메서드를 본 컴파일러는 toArray에 넘길 T 인스턴스 2개를 담을 가변인수 매개변수 배열을 만드는 코드를 생성한다
- 이때 만들어지는 배열의 타입은 Object[]인데, 어떤 타입의 객체를 넘기더라도 담을수 있는 가장 구체적인 타입이기 때문이다. 즉 pickTwo는 항상 Object[] 를 반환하게된다
- 위 코드는 아무런 문제없이 컴파일 되지만, 실행할 경우 ClassCastException 이 발생할것이다
- pickTwo의 반환값을 attributes에 저장하기 위해 String[]으로 형변환 하는 코드를 컴파일러가 자동생성하고 있기 때문이다
- Object[]는 String[]의 하위타입이 아니므로 형변환이 실패하는것이다
- 이 예시는 제너릭 가변인수 매개변수 배열에 다른 메서드가 접근하도록 하용하면 안전하지 않다는점을 보여주는것이다
    * 예외로는 2개가 있는데, ```@SafeVarargs```가 제대로 선언된 또다른 가변인수 메서드에 넘기는것이나, 이 배열의 내용의 일부를 호출만하는 일반 메서드에 넘기는것은 안전하다

```java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayList<>();
    for(List<? extends T> list : lists) {
        result.addAll(list);
    }
    return result;
}
```

- 위 예시는 제너릭 가변인수 매개변수를 안전하게 사용하는 전형적인 예시이다
- ```@SafeVarargs``` 애너테이션을 사용해야 할떄를 정하는 규칙은 간단한데, 제너릭이나 매개변수화 타입의 가변인수 매개변수를 받는 모든 메서드에  ```@SafeVarargs```를 달아야 한다는것이다. 즉 안전하지 않은 가변인수 메서드는 절대 작성해서는 안된다는 것이다
-  ```@SafeVarargs```이 유일한 정답은 아닌데, 가변인수 매개변수를 List매개변수로 바꿀수도 있다. 이 방식을 적용해 위의 예시를 바꾸면 이렇게된다

```java
@SafeVarargs
static <T> List<T> flatten(List<List<? extends T>> lists) {
    List<T> result = new ArrayList<>();
    for(List<? extends T> list : lists) {
        result.addAll(list);
    }
    return result;
}
```

- 정적 팩터리 메서드인 List.of를 활용하면 이 메서드에 임의의 개수의 인수를 넘길수 있다
- 이 방식의 장점은 컴파일러가 이 메서드의 타입안정성을 검증할 수 있다는것인데, ```@SafeVarargs```를 우리가 직접 달지 않아도 되며, 실수로 안전하다고 판단할 걱정도 없다. 단점이라면 조금 느려질수도 있고, 클라이언트 코드가 조금 지저분해진다는 것이다
- 이 방식은 toArray처럼 가변인수 메서드를 안전하게 작성하는게 불가능한 상황에서도 쓸 수 있는데, 이를 이용해 pickTwo를 바꾸면 다음과 같다

```java
static <T> List<T> pickTwo(T a, T b, T c) {
    switch(ThreadLocalRandom.current().nextInt(3)) {
        case 0 : return List.of(a, b);
        case 1 : return List.of(a ,c);
        case 2 : return List.of(b ,c);
    }
    throw new AssertionError();
}

public static void main(String args[]) {
    List<String> attributes = pickTwo("좋은", "빠른", "저렴한");
}
```

- 결과 코드는 제너릭만 사용하므로 타입안전하다