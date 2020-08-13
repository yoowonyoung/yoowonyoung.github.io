---
layout: post
title: "Effective Java - 아이템20: 추상클래스보다는 인터페이스를 우선하라"
description: 추상클래스보다는 인터페이스를 우선하라
date: 2020-08-13 21:51:00 +09:00
categories: EffectiveJava Study
---


# 클래스와 인터페이스

## 아이템 20 : 추상클래스보다는 인터페이스를 우선하라

- 자바가 제공하는 다중구현 매커니즘은 인터페이스와 추상클래스 이렇게 두가지다
- 자바8 부터 인퍼테이스도 디폴트 메서드를 제공 할 수 있게되어, 이제 두 매커님즘 모두 인스턴스 메서드를 구현 형태로 제공 할 수 있다
- 둘의 가장 큰 차이점은 추상클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다는 점이다. 자바는 단일 상속만 지원하니 이로 인해 새로운 타입을 정의하는데 커다란 제약이 생긴다
- 인터페이스는 인터페이스가 선언한 메서드를 모두 정의하고 그 일반 규약을 잘 지킨 클래스라면 다른 어떤 클래스를 상속했든 같은 타입으로 취급한다
- 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해 넣을 수 있다. 인터페이스가 요구하는 메서드를 추가하고, 클래스 선언에 implements를 추가하면 끝이다
- 하지만 기존 클래스에 새로운 추상 클래스를 끼워넣기는 어려운게 일반적이다. 두 클래스가 같은 추상 클래스를 확장하길 원한다면, 그 추상클래스는 계층구조상 두 클래스의 공통조상 이여야한다
- 인터페이스는 믹스인(Mixin)정의에 안성맞춤이다
    * 믹스인이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 '주입된 타입' 이외에도 특정 선택정 행위를 제공한다고 선언하는 효과를 준다
    * Comparable과 같은것이 대표적인 예이다. 이처럼 대상 타입의 주된 기능에 선택정 기능을 혼합한다 해서 믹스인 이라고 부른다

- 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다
- 타입을 계층적으로 정의하면 수많은 개념을 구조적으로 잘 표현 할 수도 있지만, 현실에는 계층을 엄격히 구분하기 어려운 개념도 있다

```java
public interface Singer {
    AudioClip sing(Song s);
}

public interface Songwriter {
    Song compose(int chartPosition);
}
```

- 위와 같이 가수 인터페이스와 작곡가 인터페이스가 있다고 할 떄, 우리 주변에서 흔히 볼 수있는 작곡하는 가수는 어떻게 구현할까?

```java
public interface SingerSongwirter extneds Singer, Songwirter {
    AudioClip strum();
    void actSensitive();
}
```

- 위와같이 타입을 인터페이스로 정의하면 카수 클래스가 Singer와 Songwriter를 모두 구현해도 전혀 문제가 되지 않으며, 새로운 메서드까지 추가해서 제 3의 인터페이스를 정의 할 수도 있다
- 이정도로 유연성이 필요하지는 않지만, 이렇게 만들어둔 인터페이스가 도움이 될 수도 있다. 같은 구조로 클래스를 만들려면 가능한 조합 전부를 각각의 클래스로 정의해야 하기 떄문이다
- 래퍼클래스와 함꼐 사용하면 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다
- 인터페이스의 메서드중 구현방법이 명백한것이 있다면, 그 구현을 디폴트 메서드로 제공해 프로그래머들의 일감을 덜어줄 수 있다(예시는 다음장에 나온다)
- 디폴트메서드를 제공할떄는 상속하려는 사람을 위한 설명을 @implSpec 자바독 태그를 붙여 문서화 해야 한다
- 디폴트메서드에도 제한은 있다. 많은 인터페이스가 equals와 hashCode같은 Object의 메서드를 정의하고 있지만, 이들은 디폴트메서드로 제공해서는 안된다
- 또한 인터페이스는 인스턴트 필드를 가질수 없고, public이 아닌 정적 멤버도 가질수 없다(private 정적 메서드는 예외이다)
- 마지막으로 우리가 만든 인터페이스 이외에는 디폴트메서드를 추가할 수 없다
- 한편, 인터페이스와 추상 골격 구현 클래스를 함꼐 제공하는식으로 인터페이스와 추상클래스의 장점을 모두 취하는 방법도 있다
    * 인터페이스로는 타입을 정의하고, 필요하면 디폴트 메서드 몇개도 함께 제공한다
    * 골격 구현 클래스는 나머지 메서드까지 구현하면 된다
    * 이렇게 해두면 단순히 골격 구현을 확장하는것 만으로 이 인터페이스를 구현하는데 필요한 일이 대부분 완료되는데, 이것이 바로 템플릿 메서드 페이스이다

- 관례상 인터페이스 이름이 Interface라면 그 골격 구현 클래스의 이름은 AbstractInterface로 짓는다
    * AbstractCollection, AbstractSet, AbstractList, AbstractMap이 핵심 컬렉션 인터페이스의 골격구현이다

```java
static List<Integer> intArrayAsList (int[] a) {
    Objects.requireNonNull(a);

    return new AbstractList<>() {
        @Override
        public Integer get(int i) {
            return a[i];
        }

        @Override
        public Integer set(int i, Integer val) {
            int oldVal = a[i];
            a[i] = val;
            return oldVal;
        }

        @Override
        public int size() {
            return a.length;
        }
    }
}
```

- List구현체가 제공하는 기능들을 잘 생각해보면, 이 코드는 골격 구현의 히믈 잘 보여주는 인상적인 예 임을 알수 있다
- 여담이지만 이 클래스는 int배열을 받아 Integer인스턴스의 리스트 형태로 보여주는 어댑터 이기도 하며, 이 구현에서 익명 클래스를 사용했음에 주목해야 한다
- 골격 구현 클래스의 아름다운은 추상 클래스처럼 구현을 도와주는 동시에, 추상 클래스로 타입을 정의할때 따라오는 심각한 제약에서 자유롭다는 점이다
- 골격 구현을 확장하는것으로 인터페이스 구현은 거의 끝나지만, 꼭 이렇게 해야하는것은 아니다. 구조상 골격 구현을 확장하지 못하는 처지라면 인터페이스를 직접 구현해야 한다
    * 이러한 경우라도 인터페이스가 직접 제공하는 디폴트 메서드의 이점을 여전히 누릴 수 있다

- 골격 구현 클래스를 우회적으로 이용할수도 있는데, 인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 private 내부 클래스를 정의하고, 각 메서드 호출을 내부 클래스의 인스턴스에 전달하는 것이다
    * 이러한 방식을 시뮬레이트한 다중 상속이라고 하며, 다중상속의 많은 장점을 제공하는 동시에 단점은 피하게 해준다

- 골격 구현 작성은 상대적으로 쉽다
    * 가장 먼저 인터페이스를 잘 살펴 다른 메서드들의 구현에 사용되는 기반메서드들을 선정한다. 이 기반 메서드들이 골격 구현에서는 추상 메서드가 된다
    * 그 다음으로 기반 메서드들을 사용해 직접 구현 할 수 있는, 사용해 직접 구현할 수 있는 메서드들을 모두 디폴트메서드로 제공한다. 단 equals와 hashCode같은 Object의 메서드 디폴트 메서드로 제공하면 안된다
    * 이 과정에서 인터페이싀 메서드가 모두 기반 메서드와 디폴트 메서드가 된다면 골격 구현을 별도로 만들 이유는 없다
    * 기반 메서드나 디폴트 메서드로 만들지 못한 메서드가 남아있다면, 이 인터페이스를 구현하는 골격 클래스를 만들어 남은 메서드들을 작성해 넣는다
    * 골격 구현 클래스에는 필요하면 public이 아닌 메서드와 필드를 추가해도 된다

- 간단한 예로 Map.Entry 인터페이스를 살펴보면 getKey와 getValue는 확실한 기반 메서드 이며, 선택적으로 setValue도 포함 할 수 있다. 이 인터페이스는 equals와 hashCode의 동작 방식도 구현해 두었으며, 이 메서드들은 모두 골격 구현 클래스에 구현한다

```java
public abstract class AbstractMapEntry<K,V> implements Map.Entry<K,V> {
    
    //변경 가능한 엔트리는 이 메서드를 반드시 재정의 해야한다
    @Override
    public V setValue(V value) {
        throw new UnsupportedOperationException();
    }

    //Map.Entry.equals의 일반 규약을 구현한다 
    @Override
    public boolean equals(Object o) {
        if (o == this) 
            return true;
        if(!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry) o;
        return Object.equals(e.getKey(), getKey())
            && Object.equals(e.getValue(), getValue());
    }

    //Map.Entry.hashCode의 일반 규약을 구현한다
    @Override
    public int hashCode() {
        return Object.hashCode(gerKey())
            ^ Object.hashCode(getValue());
    }

    @Override
    public String toString() {
        return getKey() + "=" + getValue();
    }
}
```

- 골격 구현은 기본적으로 상속해서 사용하는걸 가정하므로 아이템19에서 이야기한 설계 및 문서화 지침을 모두 따라야 한다
- 단순구현은 골격구현의 작은 변종으로, AbstractMap.SimpleEntry가 좋은 예이다
- 단순구현도 골격구현과 같이 상속을 위해 인터페이스를 구현한 것이지만, 추상클래스가 아니라는점이 다르다. 쉽게 말해 동작하는 가장 단순한 구현이며, 이는 그대로  써도 되고 필요에 맞게 확장해도 된다
- 정리하자면 일반적으로 다중 구현용 타입으로는 인터페이스가 가장 적합하며, 복잡한 인터페이스라면 구현 수고를 덜어주는 골격 구현을 함께 제공하는 방법을 꼭 고려해보자
