---
layout: post
title: "Effective Java - 아이템29: 이왕이면 제너릭 타입으로 만들어라"
description: 배열보다는 리스트를 사용하라
date: 2020-08-18 20:41:00 +09:00
categories: EffectiveJava Study
---


# 제너릭

## 아이템 29 : 이왕이면 제너릭 타입으로 만들어라

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
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }

    private void ensureCapacity() {
        if(elements.length == size) {
            elements = Arrays.copyOf(elements, 2*size + 1);
        }
    }
}
```

- 위 코드는 아이템7에서 다뤗던 단순한 스택이다. 이 클래스는 원래 제너릭 타입이여야 마땅하며, 제너릭으로 만든다 하더라도 현재 버전을 사용하는 클라이언트엔 아무런 문제가 없다
- 일반 클래스를 제너릭 클래스로 변환하는 첫 단계는 클래스 선언에 타입 매개변수를 추가하는 일이다

```java
public class Stack {
    private E[] elements;
    private int size;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0) 
            throw new EmptyStackException();
        E result = elements[--size];//컴파일 에러. 실체화 불가 타입을 사용함
        elements[size] = null;
        return result;
    }

    private void ensureCapacity {
        if(elements.length == size) {
            elements = Arrays.copyOf(elements, 2*size + 1);
        }
    }
}
```

- 이 단계에서 대체로 하나 이상의 오류나 경고가 뜨는데, 이 코드도 예외가 아니다. 실체화 불가 타입으로 배열을 만들수 없기 때문이다. 배열을 제너릭으로 만들때 항상 이와 같은 문제가 발생한다
- 해결책으로는 2가지 방법이 존재 하는데, 첫번쨰는 제너릭 배열 생성 제약을 우회하는 방법이다. Object 배열을 생성한 후 제너릭 배열로 형변환 하면 된다

```java
@SuppressWarnings("unchecked")
public Stack() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```

- 컴파일러는 이 프로그램이 타입 안전한지 증명할 방법이 없어 오류 대신 경고를 내보내지만, 우리는 할 수 있다. 이 코드에서의 비검사 형변환은 확실히 안전하므로 어노테이션으로 해당 경고를 숨길수있다
- 두번째 방법으로는 elements 필드의 타입을 E[]에서 Object[]로 바꾸는 것이다. 이렇게 하면 첫번째와는 다른 오류가 발생한다. 
배열이 반환하는 원소의 타입이 문제인것인데, 이를 E로 반환하면 오류대신 경고가 뜬다

```java
public E pop() {
    if (size == 0)
        throw new EmptyStackException();
    @SuppressWarnings("unchecked")
    E result = (E)elements[--size];
    element[size] = null;
    return result;
}
```

- 아이템 27의 조언에 따라 비검사 형변환을 수행하는 할당문에서만 해당 오류를 숨기는것이 좋다
- 이러한 2가지 방법 모두 나름의 장단이 있다
    * 첫번째 방법은 가독성이 더 좋고, 코드도 짧다. 현업에서도 이 방식이 자주 쓰이지만, 배열의 런타임 타입이 컴파일타임 타입과 달라 힙 오염이 일어난다

- Stack의 예제는 배열보다는 리스트를 우선하라는 아이템28과는 모순되어 보이는데, 사실 제너릭 타입 안에서 리스트를 사용하는게 항상 가능하지도, 꼭 더좋은것도 아니다
- Stack의 예제처럼 제너릭 타입은 타입 매개변수에 아무런 제약을 두지 않지만, 단 기본 타입은 사용할 수 없어 컴파일 오류가 나고, 래퍼클래스를 이용해야 한다
- 타입 매개변수에 제약을 두는 제너릭 타입도 있는데, 그 예인 DelayQueue는 다음과 같이 선언되어 있다

```java
class DelayQueue<E extends Delayed> implements BlockingQueue<E>
```

- 타입 매개변수 목록인 ```<E extends Delayed>``` 는 Delayed의 하위타입만 받는다는 뜻이며, 이렇게 하면 DelayQuque 자신과 DelayQueue를 사용하는 클라이언트는 DelayQueue의 원소에서 형변환 없이 바로 Delayed클래스의 메서드를 호출 할 수 있다
    * 이러한 매개변수 E를 한정적 타입 매개변수라고 하며, 모든 타입은 자기 자신의 하위 타입이므로 DelayQueue<Delayed도 가능하다
