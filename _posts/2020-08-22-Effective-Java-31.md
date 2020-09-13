---
layout: post
title: "Effective Java - 아이템31: 한정적 와일드카드를 사용해 API유연성을 높여라"
description: 한정적 와일드카드를 사용해 API유연성을 높여라
date: 2020-08-22 12:22:00 +09:00
categories: EffectiveJava Study
---


# 제너릭

## 아이템 31 : 한정적 와일드카드를 사용해 API유연성을 높여라

- 아이템 28에서 이야기했듯 매개변수화 타입은 불공변이므로 Type1, Type2가 있을때 ```List<Type1>```, ```List<Type2>```는 어떤 관계도 아니다
- ```List<String>```은 ```List<Object>``` 의 하위타입이 아니라는소린데, 리스코프 치환원칙을 생각해보면 맞는말이다. List<String>은 List<Object>를 대체할 수 없다
- 하지만 때론 불공변 방식보다 유연한 무언가가 필요할때가 있다
- 아이템 29의 스택을 다시 생각해보면, 다음과 같이 일련의 원소를 스택에 넣는 메서드를 추가해야한다고 생각해보자

```java
public void pushAll(Iterable<E> src) {
    for(E e: src)
        push(e);
}
```

- 이 메서드는 깨끗히 컴파일되지만, 모든상황에서 완벽하게 작동하는것은 아니다. Iterable src의 원소 타입이 스택의 원소 타입과 일치할때만 잘 작동한다
- 즉 Stack<Number>로 선언한 후 pushAll에 Integer를 넣을경우 동작하지 않는다
- 논리적으로는 Integer는 Number의 하위타입 이므로 잘 동작해야 할것으로 보이지만, 매개변수화 타입은 불공변이기 때문에 오류가 난다
- 해결책은 한정적 와일드카드라는 특별한 매개변수화 타입을 활용하는 것이다

```java
public void pushAll(Iterable<? extends E> src) {
    for(E e: src)
        push(e);
}
```

- ```Iterable<? extends E> src``` 은 E의 Iterable이 아니라 E의 하위타입의 Iterable이라는 뜻이다
    * extends라는 키워드는 여기에 안어울리긴 한다. 하위타입이란 자기 자신도 포함하지만, 자기 자신을 확장(extend)한것은 아니기 때문이다
    
- 이제 이 코드는 말끔히 컴파일 되며 잘 작동한다. 타입이 안전하기 때문이다
- 이제 pushAll과 짝을 이루는 popAll의 차례이다

```java
public void popAll(Collection<E> dst) {
    while(!isEmpty())
        dst.add(pop());
}
```

- 이번에도 주어진 컬렉션의 원소 타입이 스택의 원소 타입과 일치한다면 말끔히 컴파일되고 문제없이 작동하지만, 완벽하지는 않다. pushAll때와 똑같은 이유이다

```java
public void popAll(Collection<? super E> dst) {
    while(!isEmpty())
        dst.add(pop());
}
```

- ```Collection<? super E> dst``` 는 E의 상위 타입의 Collection이란 의미이다
- 여기서 주는 메시지는 분명하다. 유연성을 극대화 하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드 카드 타입을 사용하라는것이다
- 한편, 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입을 써도 좋을것은 없다
- 어떤 와일드 카드 타입을 써야 할지는 producer-extends, consumer-super : PECS 로 외우면 편하다
- 즉 매개변수화 타입 T가 생산자라면 ```<? extends T>```를 사용하고, 소비자라면 ```<? super T>```를 사용하라
- 이제 이전에 만든 코드들을 볼 차례이다. 아이템 28의 Chooser생성자에서 choices 컬렉션은 T타입의 값을 생산하기만 하니 다음과 같이 수정해야한다

```java
public Chooser(Collection<? extends T> choices)
```

- 이와 같이 수정한 경우, 이전코드에서 Chooser<Number> 생성자에 List<Integer>를 넘기면 컴파일조차 되지 않겟지만 이젠 아니다
- 이제 아이템 30의 union을 보자

```java
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2);
```

- 여기서 반환 타입은 여전히 Set<E> 임에 주목해야한다, 반환타입에는 한정적 와일드카드 타입을 사용하면 안된다. 유연성을 높이는게 아니라 클라이언트 코드에서도 와일드카드 타입을 써야하는 문제가 발생해버린다
- 이처럼 제대로만 사용한다면 클래스 사용자는 와일드카드 타입이 쓰였다는 사실조차 의식하지 못한다
- 클래스 사용자가 와일드카드 타입을 신경써야 한다면 그 API는 문제가 있을 가능성이 크다
- 이번엔 아이템30의 max메서드이다(매개변수 타입을 Collection에서 List로 변경하였다)

```java
public static <E extends Comparable<? super E>> E max(List<? extends E> list)
```

- 이번엔 PECS공식을 2번 사용했는데, 입력 매개변수에서 E 인스턴스를 생산하므로 원래의 List<E>를 List<? extends E>로 수정하였다
- 두번째인 타입 매개변수는 좀 난해한데, 원래는 E가 Comparable<E>를 확장한다고 정의 했는데, 이때 Comparable<E>는 E 인스턴스를 소비한다. 그래서 Comparable<E>를 Comparable<? super E>로 대체 하였다
- 일반적으로 Comparable<E> 보다는 Comparable<? super E>를 사용하는게 낫다. Comparator도 마찬가지인데 Comparator<E> 보다는 Comparator<? super E>가 낫다
- 수정된 max는 충분히 그럴만한 값어치가 있는데, ```List<ScheduledFeature<?>>```와 같은 리스트를 처리할 수 있기 때문이다
- 와일드카드와 관련해 논의해야할 주제가 하나 더 남아있는데, 타입 매개변수와 와일드카드에는 공통되는 부분이 있어, 메서드를 정의할때 둘중 어느것을 사용해도 괜찮을때가 많다
- 이럴 경우 기본 규칙은 다음과 같다
    * 메서드 선언에 타입 매개변수가 한번만 나오면 와일드 카드로 대체한다. 이때 비한정적 타입 매개변수라면 비한정적 와일드카드로, 한정적 타입 매개변수라면 한정적 와일드카드로 대체한다

```java
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```

- 위처럼 똑같은 swap을 타임 배개변수와 와일드카드를 모두 사용해 구현할 수 있다

```java
public static void swap(List<?> list, int i, int j) {
    list.set(i, list.set(j, list(get(i))));
}
```

- 하지만 위와 같은 코드는 컴파일 에러가 난다. 원인은 List<?>에는 null이외의 아무런 값을 넣을수 없기 때문이다
- 따라서 아래와 같이 와일드카드 타입의 실제 타입을 알려주는 메서드를 private 도우미 메서드로 만들어야 하고, 이때 도우미 메서드는 제너릭이여야한다

```java
public static void swap(List<?> list, int i, int j) {
    swapHelper(list,i,j);
}

private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list(get(i))));
}
```

- swapHelper메서드는 리스트가 List<E>임을 알고 있고, 리스트에서 꺼낸 값은 항상 E타입이고, E타입의 값이라면 리스트에 넣어도 안전함을 알고 있기 때문이다
