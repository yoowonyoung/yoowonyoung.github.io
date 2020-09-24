---
layout: post
title: "Effective Java - 아이템87: 커스텀 직렬화 형태를 고려해보라"
description: 커스텀 직렬화 형태를 고려해보라
date: 2020-09-24 21:05:00 +09:00
categories: EffectiveJava Study
---


# 직렬화

## 아이템 87 : 커스텀 직렬화 형태를 고려해보라

- 개발 일정에 쫒기는 상황에서는 API설계에 노력을 집중하는 편이 나을것이다. 다음 릴리즈에서 제대로 다시 구현하기로 하고, 이번 릴리즈에서는 동작만 하도록 만들어놓으란 뜻이다
- 하지만 클래스가 Serializable을 구현하고 기본 직렬화 형태를 사용한다면, 다음 릴리즈때 버리려한 현재의 구현에 영원히 묶일것이다. 기본 직렬화 형태를 버릴수 없는것이다. 실제로 BigInteger같은 일부 자바 클래스가 이 문제에 시달리고있다
- 먼저 고민해보고 괜찮다고 판단할때만 기본 직렬화 형태를 사용하라. 기본 직렬화 형태는 유연성, 성능, 정확성 측면에서 신중히 고민한 후 합당할때만 사용해야 한다. 일반적으로 직접 설계하더라도 기본 직렬화 형태와 거의 같은 형태가 나올때만 기본 형태를 써야 한다
- 어떤 객체의 기본 직렬화 형태는 그 객체를 루트로하는 객체 그래프의 물리적 모습을 나름 효율적으로 인코딩한다. 객체가 포함한 데이터들을 그 객체에서부터 시작해 접근할 수 있는 모든 객체를 담아내며, 이 객체들과 연결된 위상(topology)까지 기술한다. 이상적인 직렬화 형태라면 물리적인 모습과 독립된 논리적인 모습만 표현해야 한다
- 객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태라도 무방하다

```java
public class Name implements Serializable {
    private final String lastName;
    private final String firstName;
    private final String middleName;
}
```

- 위의 코드는 사람으 이름을 표현한 코드로, 이름은 논리적으로 이름, 성, 중간이름 으로 구성되며 위 코드의 인스턴스 필드들은 이 논리적 구성요소를 정확히 반영했다. 이런 경우가 기본 직렬화가 적합한 형태인것이다.
- 기본 직렬화 형태가 적합하다고 결정했다 하더라도, 불변식 보장과 보안을 위해 readObject 메서드를 제공 해야 할떄가 많다. 앞의 Name 클래스의 경우에는 readObject 가 lastName과 firstName이 null이 아님을 보장해야 한다(이름과 성은 null일수 없다!)

```java
public final class StringList implements Serializable {
    private int size = 0;
    private int Entry head = null;

    private static class Entry implements Serializab;e {
        String data;
        Entry next;
        Entry previous;
    }
}
```

- 위 코드는 논리적으로 일련의 문자열을 표현한다. 물리적으로는 문자열들을 이중 링크드 리스트로 연결했다. 이 클래스에 기본 직렬화 형태를 사용하면 각 노드의 양방향 연결 정보를 포함해 모든 엔트리를 철두철미하게 기록한다
- 객체의 물리적 표현과 논리적 표현의 차이가 클때 기본 직렬화 형태를 사용하면 네가지 면에서 문제가 된다
    * 공개 API가 현재의 내부 표현 방식에 영원히 묶인다
    * 너무 많은 공간을 차지 할 수 있다
    * 시간이 너무 많이 걸릴 수 있다
    * 스택 오버플로우를 일으킬 수 있다

- StringList를 위한 합리적인 직렬화 형태는 무엇일까? 단순히 리스트가 포함한 문자열의 개수를 적은 다음, 그 뒤로 문자열들을 나열하는 수준이면 될것이다. StringList의 물리적인 상세 표현은 배제한채, 논리적인 구성만 담는것이다

```java
public final class StringList implements Serializable {
    private transient int size = 0;
    private transient Entry head = null;

    private static class Entry {
        String data;
        Entry next;
        Entry previous;
    }

    public final void add(String s) { ... }

    private void wirteObject(ObjectOutputStream s) throws IOException {
        s.deafultWriteObjct();
        s.writeInt()

        for(Entry e = head; e != null; e = e.next)
            s.writeObject(e.data);
    }

    private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        int numElements = s.readInt();

        for(int i = 0; i < numElements; i++) 
            add((String) s.readObject);
    }
}
```

- StringList의 필드 모두가 transient더라도 writeObject와 readObject는 각각 가장 먼저 defaultWriteObject와 defaultReadObject를 호출한다
    * 클래스의 인스턴스 필드가 모두 transient면 defaultWriteObject와 defaultReadObject를 호출하지 않아도 된다고 들었을지 모르지만, 직렬화 명세는 이 작업을 무조건 하라고 호출한다. 그래야 향후 릴리즈에서 transient가 아닌 인스턴스 필드가 추가되더라도 상호 호환 되기 때문이다

- 문자열들의 길이가 평균 10이라면, 개선 버전의 StringList 직렬화 형태는 원래 버전의 절반 정도의 공간을 차지하며, 속도도 빠르다. 오버플로도 전혀 발생하지 않는다
- StingList에서도 기본 직렬화 형태는 적합하지 않았지만, 상태가 훨씬 심각한 클래스들도 있다. 불변식이 세부 구현에 따라 달라지는 객체에 대해서는 역직렬화를 하면 불변식이 제대로 복원되지 않을 수 있다
    * 해시테이블의 예시가 그렇다. 해시 키를 계산하는 방식이 구현마다 다르기 때문이다

- 기본 직렬화를 수용하든 하지않든 defaultWriteObject 메서드를 호출하면 transient로 선언하지 않은 모든 인스턴스 필드가 직렬화 된다. 따라서 transient로 선언해도 되는 인스턴스 필드에는 모두 붙여야 한다. 캐시된 해시값처럼 다른 필드에서 유도되는 필드도 여기에 해당한다. 해당 객체의 논리적 상태와 무관한 필드라고 확신할때만 transient를 생략해야 한다
- 기본 직렬화를 사용한다면 transient 필드들은 역직렬화 당시 기본 값으로 초기화됨을 잊지 말아야 한다. 객체 참조는 null로, 숫자는 0으로, boolean 은 false로 초기화 된다
- 기본 직렬화 사용 여부와 상관 없이 객체의 전체 상태를 읽는 메서드에 적용해야하는 동기화 매커니즘을 직렬화에도 적용해야 한다. 따라서 모든 메서드를 synchronized로 선언하여 스레드를 안전하게 만든 객체가 있다면, 기본 직렬화를 위해 writeObject도 다음 코드처럼 synchrinized로 선언해야 한다

```java
private synchronized void wirteObject(ObjectOutputStream s) throws IOException {
    s.defaultWriteObject();
}
```

- 어떤 직렬화 형태를 택하든 직렬화 가능 클래스 모두에게 직렬 버전 UID를 명시적으로 부여하자. 이렇게하면 직렬 버전 UID가 일으키는 잠재적인 호환성 문제가 사라진다. 성능도 조금 빨라진다
- 기본 버전 클래스와의 호환성을 끊고 싶다면, 단순히 직렬 버전 UID값을 바꿔주면 된다. 따라서 구버전으로 직렬화된 인스턴스와의 호환성을 끊으려는 경우를 제외하고는 직렬버전 UID를 절대 수정하지 말자
