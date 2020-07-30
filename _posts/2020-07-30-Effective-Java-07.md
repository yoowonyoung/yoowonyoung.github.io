---
layout: post
title: "Effective Java - 아이템7: 다 쓴 객체를 참조 해제하라"
description: 다 쓴 객체를 참조 해제하라
date: 2020-07-30 21:49:00 +09:00
categories: EffectiveJava Study
---


# 객체의 생성과 파괴

## 아이템 7 : 다 쓴 객체를 참조 해제하라

- Java처럼 가비지 컬렉터를 갖춘 언어는, 자칫 메모리 관리에 더 이상 신경쓰지 않아도 된다고 생각 할 수 있는데, 이는 사실이 아니다

```java
public class Stack {
    private Object[] elements;
    private int size;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0) 
            throw new EmptyStackException();
        return elements[--size];
    }

    private void ensureCapacity {
        if(elements.length == size) {
            elements = Arrays.copyOf(elements, 2*size + 1);
        }
    }
}
```

- 위 코드는 크게 문제는 없어보이지만, 메모리 누수가 숨어있다. 이 코드를 오래 사용 할 수록 가비지컬렉션 활동과 메모리 사용량이 늘어나 성능이 저하 될 것이다
- 위 코드에서 스택이 커졌다가 줄어들떄, 스택에서 꺼내진 객체들은 가비지 컬렉터가 회수하지 않는다. 그 객체들을 더이상 사용하지 않더라도 말이다
- 이 스택이 그 객체들의 객체들을 더이상 사용하지 않더라도, 스택이 그 객체들의 다 쓴 참조를 여전히 가지고 있기 때문이다
- 이렇게 가비지 컬렉션 언어에서는 메모리 누수를 찾아내기가 아주 까다롭다, 객체 참조를 하나 살려두면 가비지 컬렉터는 그 객체 뿐 아니라 그 객체가 참조하는 모든객체를 회수 해가지 못한다
- 이 문제의 해법은 간단하다. 아래와 같이 해당 참조를 다 썻을때 null처리를 하면 된다

```java
public Object pop() {
    if (size == 0) 
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null;
    return result;
}
```

- 다 쓴 참조를 null 처리하면 다른 이점도 있다. 만약 null 처리한 참조를 실수로 사용하려면 바로 NullPointerException을 던지게 된다
- 하지만, 모든 객체를 다 사용 하자마자 null 처리하는것은 프로그램을 필요 이상으로 지저분하게 만든다. 객체 참조를 null 처리하는것은 예외적인 경우여야 한다
- 다 쓴 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 유효범위 밖으로 밀어내는 것이다
- null 처리는 Stack과 같이 자기 메모리를 직접 관리하는 클래스에서 하는것이 바람직 하다
- 캐시 역시 메모리 누수를 일으키는 주범이다. 객체 참조를 캐시에 넣고 이 사실을 까먹은채 그 객체를 다 쓴 뒤로도 한참 놔두는 일을 많이 접할 수 있다
    * 캐시 외부에서 키를 참조하는 동안만 엔트리가 살아있는 캐시가 필요 하면 WeakHashMap을 사용 하면 된다
    * ScheduledThreadPoolExcutor와 같은 백그라운드 쓰레드나 새 엔트리를 추가할떄 부수작업으로 사용하지 않는 엔트리를 이따금 청소해주는것도 방법이다

- 리스너, 콜백 역시 메모리 누수의 주범인데, 클라이언트가 콜백을 등록만 하고 명확히 해지 하지 않는다면 뭔가 조치를 해주지 않는한 콜백은 계속 쌓여만 갈것이다
    * 이럴때 콜백을 약한 참조로 저장하면 가비지 컬렉터가 즉시 회수 해가는데, WeakHashMap에 등록 하는것도 방법이다
