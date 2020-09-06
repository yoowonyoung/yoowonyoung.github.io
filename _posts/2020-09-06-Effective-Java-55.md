---
layout: post
title: "Effective Java - 아이템55: 옵셔널 반환은 신중히 하라"
description: 옵셔널 반환은 신중히 하라
date: 2020-09-06 17:41:00 +09:00
categories: EffectiveJava Study
---


# 메서드

## 아이템 55 : 옵셔널 반환은 신중히 하라

- 자바8 이전에는 메서드가 특정 조건에서 값을 반환할 수 없을떄 취할수 있는 선택지는 2가지였다. 예외를 던지거나 null을 반환하는것이다
    * 두가지 모두 허점이 있다, 예외는 진짜 예외적인 상황에서만 사용해야 하며, 예외를 생성할때 스택 추적 전체를 캡쳐하므로 비용도 만만치않다
    * null을 반환하는 메서드는 별도의 null처리 코드를 추가해야 한다. null처리를 하지 않을경우 NullPointerException이 발생할 수도 있다

- 자바8로 버전이 올라가면서 선택지가 추가되었는데, 그 주인공인 Optional<T>는 null이 아닌 T타입 참조를 하나 담거나 혹은 아무것도 담지 않을 수 있다
    * 아무것도 담지 않은 옵셔널은 비었다고 말하며, 어떤 값을 담은 옵셔널은 비지 않았다고 한다
    * 옵셔널은 원소를 최대 1개 가질수있는 '불변'컬렉션이다. Optional<T>가 Collection<T>를 구현하지는 않았지만 원칙적으로 그렇단 말이다

- 보통은 T를 반환해야하지만 특정 조건에서는 아무것도 반환하지 않아야할때 T 대신 Optional<T>를 반환하도록 선언하면 된다. 그러면 유효한 반환값이 없을떄는 빈 결과를 반환하는 메서드가 만들어진다
- 옵셔널을 반환하는 메서드는 예외를 던지는 메서드보다 유연하고 사용하기 쉬우며, null을 반환하는 메서드보다 오류 가능성이 작다

```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if(c.isEmpty())
        throw new IllegalArgumentException("빈 컬렉션");
    E result = null;
    for(E e: c) {
        if(result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);
    }
    return result;
}
```

- 위 코드는 아이템30에서 등장했던 주어진 컬렉션에서 최대값을 구하는 메서드이다. 이 메서드에 빈 컬렉션을 건네면 IllegalArgumentException을 던지는데, 아이템 30에서도 말했듯이 Optional<E>를 반환하는편이 더 낫다

```java
public static <E extends Comparable<E>> Optional<E> max(Colletion<E> c) {
    if(c.isEmpty())
        return Optional.empty();
    E result = null;
    for(E e: c) {
        if(result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);
    }
    return Optional.of(result);
}
```

- 보다시피 옵셔널을 반환하도록 구현하기는 어렵지 않다. 적절한 정적 팩터리를 사용해 옵셔널을 생성해주기만 하면 된다
    * 이 코드에서는 빈 옵셔널을 만드는 팩터리인 Optional.empty()와 값이 든 옵셔널인 Optional.of(value)를 사용했다
    * Optional.of는 null을 넣을경우 NullPointerException을 던지니 주의해야 한다. null값도 허용하려면 Optional.ofNnullable()을 써야한다

- 옵셔널을 반환하는 메서드에서는 절대 null을 반환하지말자. 옵셔널을 도입한 취지를 완전히 무시하는 행위이다
- 스트림의 종단 연산중 상당수가 옵셔널을 반환하는데, 앞의 max를 스트림버전으로 다시 작성한다면 Stream의 max연산이 우리에게 필요한 옵셔널을 생성 해줄것이다

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    return c.stream().max(Comparator.naturalOrder());
}
```

- 그렇다면 null을 반환하거나 예외를 던지는 대신 옵셔널 반환을 선택해야 하는 기준은 무엇인가? 옵셔널은 검사 예외와 취지가 비슷하다. 즉 반환값이 없을수도 있음을 사용자에게 명확히 알려준다
    * 비검사예외를 던지거나 null을 반환한다면 API 사용자가 그 사실을 인지하지 못해 끔직한 결과로 이어질 수 있다

- 비슷하게 메서드가 옵셔넗을 반환한다면 클라이언트는 값을 받지 못했을때 취할 행동을 선택해야 한다. 그중 하나는 기본값을 설정하는 방법이다

```java
String lastWorkdInLexicon = max(words).orElse("단어 없음");
```

- 또한 상황에 맞는 예외를 던질 수 있다. 다음 코드에서 예외가 아닌 예외 팩터리를 건넨것에 주목해야 한다. 이렇게 하면 실제로 예외가 발생하지 않는한 예외 생성 비용은 들지 않는다

```java
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```

- 옵셔널에 항상 값이 채워져있다고 확신한다면 그냥 곧바로 값을 꺼내 사용하는 선택지도 있다. 다만 잘못 판단한것이라면 NoSuchElementException이 발생할것이다

```java
Element lastNobleGas = max(Elements.NOBLE_GLASS).get();
```

- 이따금 기본값을 설정하는 비용이 아주커서 부담이 될 떄가 있다. 그럴때는 Supplier<T>를 인수로 받는 orElseGet을 사용하면 그 값이 처음으로 필요할 때 Supplier<T>를 사용해 생성하므로 초기 설정 비용을 낮출수있다
- 적합한 메서드를 찾지 못했다면 isPresent메서드를 살펴보면 된다. 안전밸브 역할의 메서드로, 옵셔널이 채워져있으면 true를 비어있으면 false를 반환한다. 이 메서드로는 원하는 모든 작업을 수행할 수있지만, 신중히 사용해야 한다
    * 실제로 isPresent를 쓴 코드중 상당수는 앞서 언급한 메서드들로 대체할 수 있으며, 그렇게하면 더 짧고 명확하고 용법에 맞는 코드가 된다

```java
Optional<ProcessHandle> parentProcess = ph.parent();
System.out.println("부모 PID: " + (parentProcess.isPresent() ? 
    String.valueOf(parentProcess.get().pid()) : "N/A"));

System.out.println("부모 PID: " +
    ph.parent().map(h -> String.valueOf(h.pid())).orElse("N/A"));
```

- 스트림을 사용한다면 옵셔널들을 Steam<Optional<T>> 로 받아서 그중 채워진 옵셔널들에서 값을 뽑아 Stream<T>에 건네담아 처리하는 경우가 드물지 않다

```java
streamOfOptionals
    .filter(Optional::isPresent)
    .map(Optional::get)
```

- 보다시피 옵셔널에 값이 있다면 그 값을 꺼내 스트림에 매핑한다
- 자바9에서는 Optional에 stream() 메서드가 추가되었다. 이 메서드는 Optional을 Stream으로 반환해주는 어댑터이다. 옵셔널에 값이 있으면 그 값을 원소로 담은 스트림을, 값이 없다면 빈 스트림으로 변환한다. 이를 Stream의 flatMap 메서드와 조합하면 앞의 코드를 더 명료하게 바꿀수있다

```java
streamOfOptionals.flatMap(Optional::stream)
```

- 반환값으로 옵셔널을 사용한다고 해서 무조건 득이 되는건 아니다. 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로감싸면 안된다. Optional<List<T>>를 반환하기보단 빈 List<T>를 반환하는게 좋다
    * 참고로 ProcessHandle.Info 인터페이스의 arguments 메서드는 Optional<String[]>을 반환하는데 이는 예외적인 경우이다

- 어떤 경우에 메서드 반환 타입을 T 대신 Optional<T>로 선언해야할까? 결과가 없을수 있으며, 클라이언트가 이 상황을 특별하게 처리해야한다면 Optional<T>를 반환한다
    * 이렇게 하더라도 Optional<T>를 반환하는데에는 대가가따른다. Optional도 엄연히 새로 할당하고 초기화 해야하는 객체이고, 그 안에서 값을 꺼내려면 메서드를 호출해야하니 한단계를 더 거치는 셈이다
    * 그래서 성능이 중요한 상황이라면 옵셔널이 맞지 않을수도 있다

- 박싱된 기본타입을 담는 옵셔널은 기본타입 자체보다 무거울수밖에 없다. 값을 두겹이나 감싸기 떄문이다. 그래서 자바 API설계자는 int, long, double 전용 옵셔널 클래스인 OptionalInt, OptionalLong, OptionalDouble이 있다
- 이 옵셔널들도 Optional<T>가 제공하는 메서드는 거의 다 제공한다. 이렇게 대체재까지 있으니 박싱된 기본타입을 담은 옵셔널을 반환하는 일은 없도록 하자(덜 중요한 기본타입인 Boolean, Byte, Character, Short, Flaot은 예외이다)
- 지금까지 옵셔널을 반환하고 반환된 옵셔널을 처리하는 이야기를나눳다. 다른 쓰임에 대해서는 이야기하지 않았는데, 대부분 적절하지 않기 떄문이다
    * 예컨데 옵셔널을 맵의 값으로 절대 사용하면 안된다. 그리한다면 맵 안에 키가 없다는 사실을 나타내는 방법이 2가지가 된다. 하나는 키 자체가 없는경우이며, 다른 하나는 키는 있지만 그 키가 속이 빈 옵셔널인 경우이다
    * 쓸데없이 복잡성만 높여서 혼란과 오류 가능성만 키울뿐이다
    * 더 일반화해서 이야기하면 옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는게 적절한 상황은 거의 없다

- 옵셔너를 인스턴스필드에 저장해두는게 필요할 때가 있을까? 이런 상황은 대부분 필수 필드를 갖는 클래스와, 이를 확장해 선택적 필드를 추가한 하위 클래스를 따로 만들어야 함을 암시하는 나쁜냄새이다
- 하지만 가끔은 적절한 상황도 있는데, 예를들어 아이템2의 NutritionFacts 클래스의 경우 인스턴스 필드중상 당수는 필수가 아니다. 또한 그 필드들은 기본 타입이라 값이 없음을 나타낼 방법이 마땅치 않다. 이런 경우 선택적 필드의 게터를 옵셔널을 반환하게 해주었으면 좋았을것이다