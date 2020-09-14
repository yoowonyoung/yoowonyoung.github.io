---
layout: post
title: "Effective Java - 아이템63: 문자열 연결은 느리니 주의하라"
description: 문자열 연결은 느리니 주의하라
date: 2020-09-13 22:17:00 +09:00
categories: EffectiveJava Study
---


# 일반적인 프로그래밍 원칙

## 아이템 63 : 문자열 연결은 느리니 주의하라

- 문자열 연결 연산자(+)는 여러 문자열을 하나로 합쳐주는 편리한 수단이다
- 그런데 한줄짜리 출력값 혹은 작고 크기가 고정된 객체의 문자열 표현을 만들때라면 괜찮지만, 본격적으로 사용하기 시작하명 성능저하를 감내하기 어렵다
- 문자열 연결 연산자로 문자열 n개를 잇는 시간은 n^2에 비례한다
- 문자열은 불변이라서 두 문자열을 연결할경우 양쪽의 내용을 모두 복사해야 하므로 성능 저하는 피할수 없는 결과이다

```java
public String statement() {
    String result = "";
    for(int i = 0; i < numItems(); i++) {
        result += lineForItem(i);
    }
    return result;
}
```

- 위 메서드는 청구서의 품목을 전부 하나의 문자열로 연결해준다. 품목이 많을경우 이 메서드는 심각하게 느려질수 있다. 성능을 포기하고 싶지 않다면 String 대신 StringBuilder를 사용하자

```java
public String statement2() {
    StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
    for(int i = 0; i < numItems(); i++) {
        b.append(lineForItem(i));
    }
    return b.toString();
}
```

- 자바6 이후 문자열 연결 성능을 다방면으로 개선했지만, 이 두 메서드의 성능 차이는 여전히 크다
- 원칙은 간단하다. 성능에 신경써야 한다면 많은 문자열을 연결할때는 문자열 연결 연산자를 피하자. 그대신 StringBuilder의 append 메서드를 사용하라