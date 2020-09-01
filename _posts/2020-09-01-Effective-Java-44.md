---
layout: post
title: "Effective Java - 아이템44: 표준 함수형 인터페이스를 사용하라"
description: 표준 함수형 인터페이스를 사용하라
date: 2020-09-01 21:31:00 +09:00
categories: EffectiveJava Study
---


# 람다와 스트림

## 아이템 44 : 표준 함수형 인터페이스를 사용하라

- 자바가 람다를 지원하면서 API를 작성하는 모범사례도 많이 변경되었는데, 예컨대 템플릿 메서드 패턴의 매력이 많이 줄게 되었다
    * 이는 같은 효과의 함수 객체를 받는 정적 팩터리나 생성자를 제공하는것이다

- 위의 내용을 일반화해서 말하면 함수 객체를 매개변수로 받는 생성자와 메서드를 더 많이 만들어야 한다는것이다
- LinkedHashMap을 생각해보면 이 클래스의 protected 메서드인 removeEldestEntry를 재정의하면 캐시로 사용할 수 있다. 맴에 새로운 키를 추가하는 put 메서드는 이 메서드를 호출하여 true가 반환되면 맴에서 가장 오래된 원소를 제거한다

```java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return size() > 100;
}
```

- 위의 코드는 잘 동작하지만 람다를 사용하면 훨씰 잘 해낼 수 있다. LinkedHashMap을 오늘날 다시 구현하면 함수 객체를 받는 정적 팩터리나 생성자를 제공 했을 것이다
- removeEldestEntry의 선언을 보면 이 함수 객체는 Map.Entry<K,V>를 받아 boolean을 반환해야 할 것 같지만 꼭 그렇지는 않다
- removeEldestEntry는 size()를 호출해 맵 안의 원소수를 반환하는데, removeEldestEntry가 인스턴스 메서드라서 가능한 방식이다
- 하지만 생성자에 넘기는 함수 객체는 이 맵의 인스턴스 메서드가 아니다. 팩터리나 생성자를 호출 할 때에는 이 맵의 인스턴스가 존재하지 않기 때문에, 맵은 자기 자신도 함수 객체에 전해줘야 한다

```java
@FunctionalInterface
interface EldestEntryRemovalFunction<K,V> {
    boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
}
```

- 이 인터페이스도 잘 동작하지만, 굳이 사용할 이유는 없다. 자바 표준 라이브러리에 이미 같은 모양의 인터페이스가 준비되어 있기 때문이다
- java.util.function 패키지를 보면 다양한 용도의 표준 함수형 인터패이스가 담겨져 있다
- 필요한 용도에 맞는 표준 함수형 인터페이스가 있다면, 직접 구현하지 말고 이를 활용하라. 그러면 API가 다루는 개념의 수가 줄어들어 익히기 더 쉬워지고, 표준 함수형 인터페이스들은 유용한 디폴트 메서드를 제공해주기 때문에, 다른 코드와의 상호운용성도 크게 좋아질 것이다
    * 예컨데 Predicate 인터페이스는 프레디키트들을 조합하는 메서드를 제공한다
    * 앞의 LinkedHashMap의 예에서는 표준 인터페이스인 BiPredicate<Map<M,V>, Map.Entry<K,V>> 를 사용 할 수있다

- java.util.function 패키지에는 총 43개의 인터페이스가 있는데, 이것을 모두 기억하기는 어려우므로 참조 타입용 6개의 기본 인터페이스를 기억하고 나머지를 유추 하는것이 좋다
- 먼저 Operator인터페이스는 인수가 1개인 UnaryOperator와 2개인 BinaryOperator로 나뉘며 반환값과 인수 타입의 타입이 같은 함수를 뜻한다
- Predicate 인터페이스는 인수 하나를 받아 boolean을 반환하는 함수를 뜻하며, Function 인터페이스는 인수와 반환타입이 다른 함수를 뜻한다
- Supplier 인터페이스는 인수를 받지 않고 값을 반환(혹은 제공)하는 함수를, Consumer 인터페이스는 인수를 하나 받고 반환값은 없는(인수를 소비하는) 함수를 뜻한다

|인터페이스|함수 시그니처|예시|
|------|---|------|
|UnaryOperator<T>|T apply(T t)|String::toLowerCase|
|BinaryOperator<T>|T apply(T t1, T t2)|BigInteger::add|
|Predicate<T>|boolean test(T t)|Collection::isEmpty|
|Function<T,R>|R apply(T t)|Arrays::asList|
|Supplier<T>|T get()|Instant::now|
|Consumer<T>|void accept(T t)|System.out::println|

- 기본 인터페이스는 기본 타입인 int, long, double 용으로 각 3개씩 변형이 생겨나며, 그 이름도 기본 인터페이스의 이름 앞에 해당 기본 타입 이름을 붙여 지었다
    * int를 받는 Predicate 는 IntPredicate가 되고, long 을 받아 long을 반환하는 BinaryOperator는 LongBinaryOperator가 되는 식이다
    * 유일하게 Function의 변형만 매개변수화 되었는데, 정확히는 반환 타입만 매개변수화 되었는데 LongFunction<int[]>은 long인수를 받아 int[]를 반환한다

- Function 인터페이스에는 기본 타입을 반환하는 변형이 총 9개가 더 있고, 인수와 같은 타입을 반환하는 함수는 UnaryOperator 이므로, Function 인터페이스의 변형은 입력과 결과의 타입이 항상 다르다
- 입력과 결과가 모두 기본 타입이면 접두어로 src```To```result 를 사용하며, 그 예로 long을 받아 int를 반환하면 LongToIntFunction이 된다
- 입력이 객체참조이고 결과가 int, long, double인 변형들은 입력을 매개변수화 하고 ```To```result의 접두어를 사용하는데, ToLongFunctio<int[]> 은 int[]을 인수로 받아 long을 반환한다
- 기본 함수형 인터페이숭 3개에는 인수를2개씩 받는 변형이 있는데, BiPredicate<T,U> , BiFunction<T,U,R>, BiConsumer<T,U> 가 그것이다
- BiFunction에는 다시 기본타입을 반환하는 세 변형 ToIntBiFunction<T,U>, ToLongBiFunction<T,U>, ToDoubleBiFunction<T,U>가 있다
- Consumer에도 객체 참조와 기본타입 하나, 즉 인수 2개를 받는 변형인 ObjDoubleConsumer<T>, ObjIntConsumer<T>, ObjLongConsumer<T> 가 있다
- 마지막으로 BooleanSupplier 인터페이스는 boolean을 반환하도록 한 Supplier의 변형인데, 이것이 표준 함수형 인터페이스중 boolean을 이르에 명시한 유일한 인터페이스이지만, Pridicate와 그 변형들도 boolean 값을 반환 할 수 있다
- 표준 함수형 인터페이스 대부분은 기본 타입만 지원하지만, 그렇다고 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 말아야 한다. 동작은 하지만 박싱된 기본 타입 대신 기본 타입을사용하라는 아이템 61의 조언을 위배하게 되며, 특히 계산량이 많을떄는 성능이 처참히 느려진다
- 대부분의 상황에서는 직접 작성하는것보다 표준 함수형 인터페이스를 사용하는편이 낫지만, 매개변수 3개를 받는 Predicate같은 용도에 맞는게 없다면 직접 작성해야 한다
- Comparator<T>는 구조적으로는 ToIntBiFunction<T,U>와 동일하고, 심지어 Comparator<T>를 추가할 당시 ToIntBiFunction<T,U>가 이미 존재했더라도 ToIntBiFunction<T,U>를 사용했으면 안됬다
- Comparator<T>가 독자적인 인터페이스로 살아남아야 하는 이유는, API에서 매우 자주 사용되고 이름이 그 용도를 명확히 설명 해주며, 구현하는 쪽에서 반드시 지켜야할 규약을 담고있고, 비교자들과 변환하고 조합해주는 디폴드 메서드들을 듬뿍 담고 있기 때문이다
- 즉 아래와 같은 특성을 하나 이상 만족하는경우 전용 함수형 인터페이스를 구현 해야 하는지 고민 해봐야 한다
    * 자주 쓰이며 이름 자체가 용도를 명확히 설명한다
    * 반드시 따라야 하는 규약이 있다
    * 유용한 디폴트 메서드를 제공 할 수 있다

- 전용 함수형 인터페이스를 작성 하기로 했다면, 본인이 작성하는것이 '인터페이스'임을 명시 해야 한다. 주의해서 설계 해야 한다는 뜻이다
- 위의 예시에서도 ```@FunctionalInterface``` 어노테이션이 달려있음에 주목해야 한다. 이 어노테이션을 사용하는 이유는 ```@Override```와 비슷한데, 프로그래머의 의도를 명시하는것이며, 목적은 3가지 이다
    * 해당 클래스의코드나 설명 문서를 읽을때 그 인터페이스가 람다용으로 설계된것임을 알려준다
    * 해당 인터페이스가 추상메서드를 오직 하나만을 가지고 있어야 컴파일 되게 해준다
    * 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 해준다

- 따라서, 직접 만든 함수형 인터페이스라면 반드시 ```@FunctionalInterface``` 어노테이션을 사용해야 한다
- 마지막으로 함수형 인터페이스를 API에서 사용할 때 주의해야할 점인데, 사로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중으로 정의해서는 안된다. 클라이언트에게 모호함을 주어 문제가 발생하기가 쉽다
    * 실제로 ExcutorSerivce의 submit 메서드는 Collable<T>와 Runnable을 받는것을 다중정의 하게 되어있어, 올바른 메서드를 알려주기 위해 형변환을 해야 하는 경우도 생딘다
    * 위와 같은 문제를 피하는 가장 쉬운 방법은 다중 정의를 피하는 것이다