---
layout: post
title: "Effective Java - 아이템36: 비트필드 대신 EnumSet을 사용하라"
description: 비트필드 대신 EnumSet을 사용하라
date: 2020-08-24 21:24:00 +09:00
categories: EffectiveJava Study
---


# 열거타입과 어노테이션

## 아이템 36 : 비트필드 대신 EnumSet을 사용하라

- 열거한 값들이 주로 집합으로 사용될경우, 예전에는 각 상수에 서로다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용해왔다

```java
public class Text {
    public static final int STYLE_BOLD = 1 << 0;
    public static final int STYLE_ITALIC = 1 << 1;
    public static final int STYLE_UNDERLINE = 1 << 2;
    public static final int STYLE_STRIKETHROUGH = 1 << 3;

    public void applyStyles(int styles) { ... }
}
```

- 다음과 같은식으로 비트별 OR를 통해 여러 상수를 하나의 집합으로 모을수 있으먀, 이렇게 만들어진 집합을 비트 필드라고 한다

```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

- 비트 필드를 사용하면 비트별 연산을 이용해 합집합과 교집합 같은 집합 연산을 효율적으로 수행할 수 있지만, 비트 필드는 정수 열거 상수의 단점을 그대로 가지고 있으며 추가로 다음과 같은 문제도 지니고 있다
    * 비트 필드값이 그대로 출력되면 단순한 정수 열거 상수를 출력할때보다 해석하기 어렵다
    * 비트 필드 하나에 녹아있는 모든 원소를 순회하기도 어렵다
    * 최대 몇비트가 필요한지를 API작성시에 미리 예측하여 적절한 타입을 선택해야 한다. API수정 없이는 비트수를 늘릴수 없기 때문이다

- Util패키지의 EnumSet 클래스는 열거타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다
- EnumSet은 Set 인터페이스를 완벽히 구현하며, 타입 안전하고 다른 어떤 Set구현체와도 함께 사용 할 수 있다
- EnumSet의 내부는 비트 벡터로 구현되어있어, 원소가 총 64개 이하라면 EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여주며, removeAll과 retainAll과 같은 대량 작업은 비트를 효율적으로 처리할 수 있는 산술 연산을 써서 구현했다
- EnumSet을 사용하면 직접 비트를 다룰때 흔히 겪는 오류들에서 해방될 수 있다. 난해한 작업을 EnumSet이 모두 해결 해주기 때문이다

```java
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

    //어떤 Set을 넘겨도 되나, EnumSet이 제일 좋다
    public void applyStyles(Set<Style> styles) { ... }
}

text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));//EnumSet의 정적 팩터리중 of를 사용
```
