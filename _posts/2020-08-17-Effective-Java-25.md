---
layout: post
title: "Effective Java - 아이템25: 톱레벨 클래스는 한파일에 하나만 담아라"
description: 톱레벨 클래스는 한파일에 하나만 담아라
date: 2020-08-17 18:18:00 +09:00
categories: EffectiveJava Study
---


# 클래스와 인터페이스

## 아이템 25 : 톱레벨 클래스는 한파일에 하나만 담아라

- 소스파일 하나에 톱레벨 클래스를 여러개 선언해도 자바 컴파일러는 불평하지는 않지만, 이것은 득이 없고 위험을 감수해야 하는 행동이다
- 이렇게 하면 한 클래스를 여러가지로 정의할 수 있으먀, 그중 어느것을 사용할지는 어느소스파일을 먼저 컴파일 하느냐에 따라 달라지기 떄문이다

```java
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}

public class Main {
    public static void main(String args[]) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```

- 위의 Utensil과 Dessert를 Utensil.java파일에 선언하였다고 가정할 때, 일단은 이 코드는 문제없이 동작 할 것이다
- 하지만 Dessert.java라는 파일이 새로 생성되고 컴파일 순서를 Dessert.java부터 할 경우, 우리가 원하는대로 동작하지 않을것이다
- 다행이 해결책은 아주 간단한데, 톱레벨 클래스들을 서로 다른 소스파일로 분리하면 되기 때문이다
- 궂이 여러 톱레벨 클래스를 한파일에 담고 싶디면 정적 멤버 클래스를 사용하는 방법을 고민 해봐도 좋다. 읽기 좋고 private로 선언하면 접근 범위도 최소로 관리할 수 있기 떄문이다

```java
public class Main {
    public static void main(String args[]) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }

    private static class Utensil {
        static final String NAME = "pan";
    }

    class Dessert {
        static final String NAME = "cake";
    }
}
```
