---
layout: post
title: "Effective Java - 아이템27: 비검사 경고를 제거하라"
description: 비검사 경고를 제거하라
date: 2020-08-17 22:32:00 +09:00
categories: EffectiveJava Study
---


# 제너릭

## 아이템 27 : 비검사 경고를 제거하라

- 제너릭을 사용하기 시작하면 비검사 형변환 경고, 비검사 메서드 호출경고, 비검사 매개변수화 가변인수 타입경고 등수많은 컴파일러 경고를 보게 될것이다
- 대부분의 비검사 경고는 쉽게 제거할 수 있는데, 대부분은 컴파일러가 어디가 잘못됐는지 알려준다(javac 커맨드라인 인수에 -Xlint:uncheck를 추가하면 된다)

```java
Set<Lark> exaltation = new HashSet(); // 비검사 에러

Set<Lark> exaltation = new HashSet<>();
```

- 위와 같은 비검사 경고는 쉽게 제거 할 수 있지만, 제거하기 훨씬 어려운 것들도 있다. 하지만 할수 있는한 모든 비검사 경고는 제거해야 한다
- 비검사 경고가 모두 제거되면 그 코드는 타입 안정성이 확보된다
- 경고를 제거할 수 없지만 타입 안전하다고 확신 할 수 있다면, ```@SuppressWarnings("unchecked")``` 어노테이션으로 경고를 숨기자
- 하지만 타입 안정성 검증없이 어노테이션을 쓴다면 런타임에 ClassCaseException이 발생 할 수있으니, 어노테이션은 항상 가능한 좁은 범위에 적용해야 한다

```java
public <T> T[] toArray(T[] a) {
    if(a.length < size)
        return (T[]) Arrays.copyOf(elements, size, a.getClass());
    System.arraycopy(elements, 0, a, 0, size);
    if(a.length > size)
        a[size] = null;
    return a;
}
```

- 위와 같은 코드는 return문에서 경고가 발생하는데, 어노테이션은 선언에만 달 수 있기 떄문에 메서드 전체에밖에 달 수 없다

```java
public <T> T[] toArray(T[] a) {
    if(a.length < size) {
        @Suppresswarnings("unchecked")
        T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
        return result;
    }
    System.arraycopy(elements, 0, a, 0, size);
    if(a.length > size)
        a[size] = null;
    return a;
}
```

- 위 코드는 깔끔하게 컴파일되고 비검사 경고를 숨기는 범위도 최소한으로 좁혔다
- ```@Suppresswarnings("unchecked")``` 어노테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다
- 주석은 다른사람이 그 코드를 이해하는데 도움이 되며, 다른사람이 그 코드를 잘못 수정하여 타입 안정성을 잃는 상황을 줄여준다
