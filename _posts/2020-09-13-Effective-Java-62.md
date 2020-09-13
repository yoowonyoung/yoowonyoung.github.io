---
layout: post
title: "Effective Java - 아이템62: 다른타입이 적절하다면 문자열 사용을 피하라"
description: 박싱된 기본 타입 보다는 기본타입을 사용하라
date: 2020-09-13 21:53:00 +09:00
categories: EffectiveJava Study
---


# 일반적인 프로그래밍 원칙

## 아이템 62 : 다른타입이 적절하다면 문자열 사용을 피하라

- 문자열은 텍스트를 표현하도록 설계되었고, 그 일을 아주 멋지게 해낸다
- 하지만 문자열이 워낙 흔하고, 자바가 또 잘 지원 해주어 원래 의도하지 않은 용도로 쓰이는 경향이 있다. 이번 아이템에서는 문자열을 쓰지 않아야할 사례를 다룬다
- 믄자열은 다른 값 타입을 대신하기에 적합하지 않다
    * 많은 사람이 파일, 네트워크, 키보드입력으로부터 데이터를 받을때 주로 문자열을사용한다. 사뭇 자연스러워 보이지만, 입력받은 데이터가 진짜 문자열일때만 그렇게 하는게 좋다
    * 받은 데이터가 수치형이라면 int, float, BinInteger등 적장한 수치 타입으로 변환해야 한다. '예/아니오' 질문의 답이라면 적절한 열거타입이나 boolean으로 변환해야 한다

- 문자열은 열거타입을 대신하기에 적합하지 않다
    * 아이템 34에서 이야기 햇듯, 상수를 열거할때는 문자열보다는 열거타입이 월등히 낫다

- 문자열은 혼합 타입을 대신하기에 적합하지 않다
    * 여러 요소가 혼합된 데이터를 하나의 문자열로 표현하는것은 대체로 좋지 않은 생각이다
    * 구분문자를 사용해 혼합 타입을 만들었다고 하자. 구분문자가 두 요소중 하나에서 쓰였다면, 혼란스러운 결과가 나올것이다
    * 각 요소를 개별로 접근하려면 문자열을 파싱해야해서 느리고, 귀찮고, 오류 가능성도 커진다
    * 적절한 equals, toString, compareTo 메서드를제공할 수 없으며, String이 제공하는 기능에만 의존해야 한다

- 문자열은 권한을 표현하기에 적합하지 않다
    * 권한을 문자열로 표현하는경우가 종종있는데, 이런 방식의 문제는 문자열 키가 전역 이름 공간에 공유된다는 문제가 있다
    * 만약 두 클라이언트가 서로 소통하지 못해 같은 키를 써버리면, 의도치 않게 같은 공유하게 된다
    * 보안에도 취약하다. 악의적인 클라이언트라면 의도적으로 같은키를 사용하여 클라이언트의 값을 가져올수도 있다

- 스레드 지역변수 기능을 설계한다고 해보자. 예전에는 클라이언트가 제공한 문자열 키로 스레드별 지역변수를 식별하였다

```java
public class ThreadLocal {
    private ThreadLocal() {}

    public static void set(String key, Object value);

    public static Object get(String key);
}
```

- 이는 앞서 말한 문제점들을 그대로 가지고 있어, 다음과 같이 API가 문자열 대신 위조할 수 없는 키를 사용하도록 해야한다

```java
public class ThreadLocal {
    private ThreadLocal() {}

    public static class Key {
        key() {}
    }

    public static Key getKey() {
        return new Key();
    }

    public static void set(Key key, Object value);
    public static Object get(Key key);
}
```

- 이 방법은 문자열 기반 API의 문제를 모두 해결해주지만, 개선할 여지는 남아있다. set과 get은 정적 메서드일 이유가 없으니, Key클래스의 인스턴스 메서드로 바꾸면 된다. 이렇게하면 Key는 더이상 스레드 지역변수를 구분하기 위한 키가 아니라, 그 자체가 스레드 지역변수가 된다. 결과적으로 지금의 톱레벨 클레스인 TheeadLocal은 별달리 하는일이 없으니 치워버리고 중첩 클래스 Key의 이름을 ThreadLocal로 바꾸면 다음과 같이 된다

```java
public final class ThreadLocal {
    public ThreadLocal();
    public void set(Object value);
    public Object get();
}
```

- 이 API에서는 get으로 얻은 Object를 실제 타입으로 형변환해 써야해서 타입 안전하지않다. ThreadLocal을 배개변수화 타입으로 선언하면 이 문제도 해결된다

```java
public final class ThreadLocal<T> {
    public ThreadLocal();
    public void set(T value);
    public T get();
}
```

