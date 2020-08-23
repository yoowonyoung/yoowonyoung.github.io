---
layout: post
title: "Effective Java - 아이템33: 타입 안전 이종 컨테이너를 고려하라"
description: 타입 안전 이종 컨테이너를 고려하라
date: 2020-08-23 17:00:00 +09:00
categories: EffectiveJava Study
---


# 제너릭

## 아이템 33 : 타입 안전 이종 컨테이너를 고려하라

- 제너릭은 Set<E>, Map<K,V> 등의 컬렉션과 ThreadLocal<T>, AutomicReference<T>등의 단일 원소 컨테이너에서도 흔히 쓰인다
- 이런 쓰임에서 매개변수화 되는 대상은 원소가 아닌 컨테이너 자기 자신으로, 하나의 컨테이너에서 매개변수화 할 수 있는 타입의 수가제한된다
- 이는 컨테이너의 일반적인 용도에 맞게 설계될것이니 문제는 없지만, 더 유연한 수단이 필요할때도 종종 있다
    * 데이터베이스의 row는 임의의 개수의 column을 가질수 있는데, 모든 열을 타입 안전하게 이용하면 좋을것이다

- 이에 대한 해법으로는, 컨테이너 대신 키를 매개변수화 한 다음, 컨테이너에 값을 넣거나 뺄때 매개변수화한 키를 함께 제공하면 된다
    * 이렇게 하면 제너릭 타입시스템이 값의 타입이 키와 같음을 보장해 줄것이다. 이러한 설계 방식이 타입 안전 이종 컨테이너 패턴이라고 한다

```java
public class Favorite {
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requiredNonNull(type), instance);
    }
    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}

public static void main(String[] args) {
    Favorite f = new Favorite();

    f.putFavorite(String.class, "Java");
    f.putFavorite(Integer.class, 0xcafebabe);
    f.putFavorite(Class.class, Favorite.class);

    String favoriteString = f.getFavorite(String.class);
    int favoriteInteger= f.getFavorite(Integer.class);
    Class<?> favoriteClass = f.getFavorite(Class.class);

    System.out.printf("%s %x %s%n", favoriteString, favoriteInteger, favoriteClasss.getName());
}
```

- 간단한 예로, 타입별로 즐겨찾는 인스턴스를 저장하고 검색할수 있는 Favorite클래스이다
- 각 타입의 Class객체를 매개변수화한 키 역할로  사용하면 되는데, 이 방식이 동작하는 이유는 class의 클래스가 제너릭이기 떄문이다
- class 리터럴의 타입은 Class가 아닌 Class<T> 이다. 예컨데 String.class의 타입은 Class<String> 이고, Integer.class의 타입은 Class<Integer>이다
- 한편 컴파일타임 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고 받는 class 리터럴을 타입 토큰이라고 한다
- Favorite클래스는 타입 안전하다. String을 요청했는데, Integer를 반환하는 일은 절대 없으며, 모든 키의 타입이 제각각이라 일반적인 맵과 다른 여러가지 타입의 원소를 담을 수 있다. 이것이 타입 안전 이종 컨테이너 이다
- Favorite 내부에서 사용하는 private맵 변수인 favorites는 ```Map<Class<?>, Object>``` 인데, 이는 비 한정적 와일드카드 타입이라 이 맵안에 아무것도 넣을수 없다고 생각할 수 있지만 사실은 그 반대이다
- favorites는 와일드카드 타입이 중첩되어있다. 맵이 아니라 키가 와일드카드 타입이고, 이는 모든 키가 서로 다른 매개변수화 타입일수있다는 뜻이다
- 그리고 favorites맵의 값  타입은 단순히 Object인데, 이는 키와 값 사이의 타입 관계를 보증하지 않는다는 것이다. 즉 모든 값이 키로 명시한 타입임을 보증하지 않는다
- putFavorite의 구현에서 키와 값 사이의 타입 링크정보는 버려지는데, getFavotite에서 이 관계를 다시 되살릴수 있으니 걱정하지 않아도된다
- getFavorite는 주어진 Class객체에 해당하는 값을 favorites 맵에서 꺼내는데, 이 객체가 반화해야할 객체가 맞지만 잘못된 컴파일타임 타입을 가지고 있으므로, 이것을 favorites맵의 값 타입인 Object나 T로 바꿔 반환해야 한다
    * 따라서 getFavorites구현은 Class의 cast메서드를 사용해서 이 객체 참조를 Class 객체가 가리키는 타입으로 동적 형변환 한다
    * cast 메서드는 형변환 연산자의 동적 버전으로 주어진 인수가 Class객체가 알려주는 타입의 인스턴스인지 검사하여 맞으면 그 인수를 그대로 반환, 아니면 ClassCastException을 던진다
    * cast가 단지 인수를 반환하기만 한다면 굳이 사용할 이유가 없어보이지만, cast메서드의 시그니처가 Class 클래스가 제너릭이라는 이점을 사용하고 있기 때문에 사용한다
    * T로 비검사 형변환 하는 손실이 없어도 Favorite를 타입 안전하게 만드는 비결이다

```java
public class Class<T> {
    T cast(Object obj);
}
``` 

- 이 Favorite클래스에는 두가지 제약이 있는데, 첫번째로는 Class객체를 제너릭이 아닌 로타입으로 넘기면 Favorite인스턴스의 타입 안정성이 쉽게 꺠진다
    * 하지만 다행이도 이런 코든느 컴파일할때 비검사 경고가 뜰것이다
    * 이러한 문제는 HashSet과 HashMap등 일반적인 컬렉션 구현체에도 똑같이 발생한다

```java
HashSet<Integer> set = new HashSet<>();
((HashSet)set).add("문자열");
```

- Favorite가 타입 불변식을 어기는일이 없도록 보장하려면 putFavorite 메서드에서 인수로 주어진 instance의 타입이 type으로 명시한 타입과 같은지 동적 형변환으로 확인하면 된다

```java
public <T> void putFavorite(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type), type.cast(instance));
}
```

- Collections에는 checkedSet, checkedList, checkedMap과 같은 메서드가 이 방식을 적용한 컬렉션 레퍼들이다. 이 정적 팩터리들은 컬렉션 혹은 맵과 함께 1개 혹은 2개의 Class 객체를 받고, 이 메서드들은 모두 제너릭이라 Class 객체와 컴파일타임 타입기 같음을 보장한다 
    * 또한 이 래퍼들은 내부 컬렉션들을 실체화한다. 이 래퍼들은 제너릭과 로타입을 섞어 사용하는 애플리케이션에서 클라이언트 코드가 컬렉션에 잘못된 타입 원소를 넣지 못하게 추적하는데 도움을 준다

- 두번째 제약은 실체화 불가 타입에는 사용할수 없다는 것이다. 다시말해 String이나 String[]는 가능해도 List<String>은 저장할 수 없다
    * List<String>.class라고 쓰면 문법 오류가 난다. List<Integer>와 List<String>은 List.class라는 같은 Class객체를 공유하므로, 만약 List<String>.class와 List<Integer>.class를 허용해서 둘다 똑같은 타입의 객체 참조를 반환한다면 아수라장이 될것이다

- 이 두번째 제약에 대한 완벽한 우회로는 없는데, 슈퍼 타입 토큰이라는 방식이 그나마 쓸만하다. 스프링 프레임워크에서는 이를 아예 ParameterizedTypeReference라는 클래스로 미리 구현해두었다
- Favorite가 사용하는 타입 토큰은 비 한정적인데, 즉 getFavorite와 putFavorite은 어떤 Class도 받아들인다
- 때로는 이 메서드들이 허용하는 타입을 제한하고 싶을 수 있는데, 한정적 타입 토큰을 활용하면 가능하다
    * 한정적 타입 토큰이란 한정적 타입 매개변수나 한정적 와일드카드를 사용하여 표현 가능한 타입을 제한한다는 타입토큰이다
    * 어노테이션 API는 한정적 타입 토큰을 적극적으로 사용하는데, 다음 예시는 AnnotatedElement 인터페이스에 선언된 메서드로, 대상 요소에 달려있는 어노테이션을 런타임에 읽어오는 기능을 한다. 이 메서드는 리플렉션의 대상이 되는 타입들, 즉 클래스, 메서드, 필드와 같이 프로그램 요소를 구현하는 타입들에서 구현한다
    * 여기서 annotationType인수는, 어노테이션 타입을 뜻하는 한정적 타입 토큰이다. 이 메서드는 토큰으로 명시한 타입의 어노테이션이 대상 요소에 달려있다면 그 어노테이션을 반환하고, 없다면 null을 반환한다. 어노테이션된 요소는 그 키가 어노테이션 타입인 타입 안전 이종 컨테이너인것이다

```java
public <T extends Annotation>
    T getAnnotation(Class<T> annotationType);
```

- Class<?> 타입의 객체가 있고, 이를 getAnnotation처럼 한정적 타입 토큰을 받는 메서드에 넘기려면, 객체를 Class<? extends Annotation>등으로 형변환 할 수 있겟지만, 이 형변환은 비검사 이므로 컴파일하면 경고가 뜰것이다
- 운좋게도 Class 클래스가 이런 형변환을 안전하게 그리고 동적으로 수행해주는 인스턴스 메서드를 제공하는데 이것이 asSubclass 메서드로, 호출된 인스턴스 자신의 Class 객체를 인수가 명시한 클래스로 형변환 한다
    * 형변환 한다는것은 이 클래스가 인수로 명시한 클래스의 하위클래스라는 뜻이다
    * 형변환에 성공하면 인수로 받은 클래스 객체를 반환하고, 실패하면 ClassCastException을 던진다

- 다음은 컴파일 시점에는 타입을 알 수 없는 어노테이션을 asSubclass메서드를 사용해 런타임에 읽어내는 예 이다

```java
static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
    Class<?> annotationType = null;
    try {
        annotationType = Class.forName(annotationTypeName);
    } catch (Exception e) {
        throw new IllegalArguementException(ex);
    }
    return element.getAnnotation(annotationType.asSubclass(Annotation.class));
}
```
