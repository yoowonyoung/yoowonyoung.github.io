---
layout: post
title: "Effective Java - 아이템18: 상속보다는 컴포지션을 사용하라"
description: 상속보다는 컴포지션을 사용하라
date: 2020-08-11 21:39:00 +09:00
categories: EffectiveJava Study
---


# 클래스와 인터페이스

## 아이템 18 : 상속보다는 컴포지션을 사용하라

- 상속은 코드를 재사용하는 강력한 수단이지만, 항상 최선은 아니다. 잘못 사용하면 오류를 내기 쉬운 소프트웨어를 만들게 된다
- 일반적인 구체 클래스를 패키지 경계를 넘어 다른 패키지의 구체 클래스를 상속하는 일은 위험하다
- 메서드 호출과 달리 상속은 캡슐화를 깨트린다. 다르게 말하면 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다
- 상위 클래스는 릴리즈 마다 내부 구현이 바뀔 수 있으며, 그에 따라 하위클래스가 수정되지 않더라도 동작이 바뀔 수 있다

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;
    
    public InstrumentedHashSet() {}

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap,loadFactor);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```

- 이 클래스는 잘 구현 한 것 같지만, addAll로 원소 3개를 더할 경우 getAddCount가 3이 아닌 6을 반환하게 된다. HashSet의 addAll이 add를 사용해서 구현 되어 있기 때문이다
- 이처럼 자신의 다른 부분을 사용하는 자기 사용 여부는 해당 클래스의 내부 구현 방식에 해당하며, 자바의 정책인지 다음 릴리즈에 바뀌는지는 알 수 없다
- 하위 클래스가 꺠지기 쉬운 이유는 하나 더 있다. 상위 클래스에 메서드가 추가 될때, 하위 클래스에서 적절한 재정의가 없을 경우 문제가 될 수 있다
- 메서드 재정의가 원인이기 떄문에, 클래스를 확장 하더라도 메서드를 재정의 하는 대신 새로운 메서드를 추가하는것이 안전하긴 하지만 이 또한 위험이 없는것은 아니다
- 이 문제를 해결하는 방법은, 기존 클래스를 확장하는 대신 새로운 클래스를 만들고 private필드로 기존 클래스의 인스턴스를 참조하게 하는 컴포지션을 구성하면 된다
- 새 클래스의 인스턴스 메서드들은 기존 클래스에 대응하는 메서드들을 호출해 그 결과를 반환한다. 이 방식을 포워딩 이라고 하며 새 클래스의 메서드들은 포워딩 메서드라고 한다
- 그 결과 새로운 클래스는 기존 클래스의 내부 구현 방식에 영향을 받지 않으며, 기존 클래스에 새로운 메서드가 추가 되더라도 전혀 영향받지 않는다

```java
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s };
    public void clear() { s.clear() }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty() { return s.isEmpty(); }
    public int size() { return s.size(); }
    public Iterator<E> iterator() { return s.iterator(); }
    public boolean add(E e) { return s.add(e); }
    public boolean remove(Object o) { return s.remove(o); }
    public boolean containsAll(Collection<?> c) { return s.containsAll(c); }
    public boolean addAll(Collections<? extends E> c) { return s.addAll(c); }
    public boolean removeAll(Collections<?> c) { return s.removeAll(c); }
    public boolean retainAll(Collections<?> c) { return s.retainAll(c); }
    public Object[] toArray() { return s.toArray(); }
    public <T> T[] toArray(T[] a) { return s.toArray(a); }
    @Override
    public int hashCode() { return s.hashCode(); }
    @Override
    public String toString() { return s.toString(); }
}

public class InstrumentedHashSet<E> extends ForwardingSet<E> {
    ...
}
```

- 새로 구현된 InstrumentedHashSet은 HashSet의 모든 기응을 정의한 Set인터페이스를 활용해 설계되어 견고하고 유연하다
- 이렇게 구현 할 경우 어떠한 Set구현체와도 어울릴 수 있다. 이렇게 다른 인터페이스를 감싸고 있다고 해서 InstrumentedHashSet 같은 클래스를 래퍼 클래스라고 하고, 이 패턴이 데코레이터 패턴이다
- 이런 래퍼클래스는 거의 단점이 없지만, 레퍼클래스가 콜백 프레임워크와는 어울리지 않는다. 콜백 프레임워크에서는 자기자신의 참조를 다른 객체에 넘겨서 콜백에 사용하는데,
내부 객체는 래퍼의 존재를 모르니 자기 자신의 참조를 던지게 되고, 그 결과 래퍼가 아닌 자기 자신이 불리게 된다
- 상속은 하위클래스가 반드시 상위 클래스의 진짜 하위 타입인 상황에서만 쓰여야 한다. 이는 자바 라이브러리에서도 위반한 클래스들이 있는데 스택은 벡터가 아니므로 Stack은 Vector를 확장해서는 안됫다
- 컴포지션을 써야 할 상황에서 상속을 사용 하는건 내부 구현을 불필요하게 노출하는 꼴이 된다. 그 결과가 API가 내부 구현에 묶이고 그 클래스의 성능도 영원히 제한 된다
- 더 심각한 문제는 클라이언트가 노출된 내부에 직접 접근 할 수 있다는 것인데, 이는 사용자를 혼란스럽게 할 수 있다
- 컴포지션 대신 상속을 사용하기로 결정하기 전에, 확장하려는 클래스의 API에 아무런 결함이 없는지에 대해 마지막으로 고민 해봐야 한다
- 결함이 있다면 이 결함이 다른 클래스의 API로 전파 될 수 있기 때문이다. 컴포지션으로는 이러한 결함을 숨기는 API를 만들 수있지만, 상속은 상위 클래스의 API를 결함까지도 그대로 승계한다
