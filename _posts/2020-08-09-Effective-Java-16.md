---
layout: post
title: "Effective Java - 아이템16: public 클래스에서는 public 필드가 아닌 접근자 메서드를 활용하라"
description: public 클래스에서는 public 필드가 아닌 접근자 메서드를 활용하라
date: 2020-08-09 17:53:00 +09:00
categories: EffectiveJava Study
---


# 클래스와 인터페이스

## 아이템 16 : public 클래스에서는 public 필드가 아닌 접근자 메서드를 활용하라

- 이따금 인스턴스필드들을 모아놓는 일 이외에는 아무런 목적도 없는 퇴보한 클래스를 작성하려 할 떄가 있다

```java
Class Point {
    public double x;
    public double y;
}
```

- 이런 클래스는 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못한다
- API를 수정하지 않고는 내부 포현을 바꿀 수 없고, 불변 식을 보장할 수 없으며, 외부 필드에서 접근할 때 부수 작업을 수행 할 수도 없다
- 철저한 객체지향 프로그래머는 이런 클래스를 상당히 싫어해서 필드를 모두 private로 바꾸고 public 접근자를 추가한다

```java
Class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        thix.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }

    public void setX(double x) { thix.x = x; }
    public void setY(double y) { this.y = y; }

}
```

- public 클래스에서라면 확실히 이 방법이 맞다. 패키지 바깥에서 접근 할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현을 언제든지 바꿀 수 있는 유연성을 얻을 수 있다
- package-private 혹은 private 중첩 클래스라면 데이터 필드를 도출한다 해도 하등 문제가 없다. 그 클래스가 표현 하려는 추상 개념만 올바르게 표현해주면 된다. 클라이언트가 클래스 내부 표현에 묶이기는 하나,
클라이언트 역시 이클래스를 포함하는 패키지 안에서만 동작하는 코드이다. 따라서 패키지 바깥 코드는 전혀 손대지 않고 데이터 표현 방식을 바꿀 수 있다
- 가끔 자바 플랫폼 라이브러리에서도 public 클래스의 필드를 직접 노출하지 말라는 규칙을 어기는 사례가 종종 있는데, 대표적인 예가 java.awt.package 패키지의 Point와 Dimension 클래스이다. 절대 따라하지 마라
- public 클래스의 필드가 불변이라면 직접 노출할 때의 단점이 조금은 줄어 들지만, 이는 여전히 좋은 생각은 아니다. API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을때 부수적인 작업을 여전히 수행 할 수없다
- 단 불변식은 보장할 수 있게 되는데, 다음 클래스는 각 인스턴스가 유요한 시간을 표현함을 보장한다

```java
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if(hour < 0 || hour >= HOURS_PER_DAY) {
            throw new IlligalArguementException("시간: " + hour);
        }
        if(minute < 0 || minute >= MINUTES_PER_DAY ) {
            throw new IlligalArguementException("분: " + minute);
        }
        this.hour = hour;
        this.minute = minute;
    }
}
```
