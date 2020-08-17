---
layout: post
title: "Effective Java - 아이템23: 태그달린 클래스 보다는 클래스 계층구조를 활용하라"
description: 태그달린 클래스 보다는 클래스 계층구조를 활용하라
date: 2020-08-17 15:00:00 +09:00
categories: EffectiveJava Study
---


# 클래스와 인터페이스

## 아이템 23 : 태그달린 클래스 보다는 클래스 계층구조를 활용하라

```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };
    final Shape shape;

    double length;
    double width;

    double radius;

    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTAHGNE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionException(shape);
        }
    }
}
```

- 위처럼 두가지 이상의 의미를 표현할수 있으며, 그중 현재 표현하는 의미를 태그값으로 알려주는 클래스들이 있다
- 이처럼 태그 달린 클래스는 단점이 한가득 이다. 열거타입 선언, 태그필드, switch등 쓸데없는 코드가 많으며, 여러 구현이 한 클래스에 혼합되있어 가독성도 나쁘다
- 또한 다른 의미를 위한 코드도 언제나 함께하니 메모리도 낭비되며, 필드들을 final로 선언하려면 해당 의미에 쓰이지 않는 필드까지 생성자에서 초기화해야 하므로 불필요한 코드가 늘어난다
- 또 다른 의미를 추가하려면 모든 switch문을 찾아 새 의미를 처리하는 코드를 추가해야 하며, 인스턴스의 타입만으로는 현재 나타내는 의미를 알 수 없다
- 한마디로 태그달린 클래스는 정황하고, 오류내기가 쉽고, 비효율적이다
- 태그달린 클래스는 클래스 계층구조를 어설프게 흉내낸 아류일뿐이며, 클래스 계층구조를 서브타이핑 하는것이 훨씬낫다
- 태그달린 클래스에서 클래스 계층구조로 바꾸는 방법은 다음과 같다
    * 계층구조의 루트가 될 추상클래스를 정의하고, 태그값에 따라 동작이 달라지는 메서드들을 루트클래스의 추상메서드로 선언한다
    * 태그의 값에 상관없이 동작이 일정한 메서드들을 루트클래스의 일반메서드로 추가한다. 모든 하위 클래스에서 사용하는 공통 데이터필드도 루트 클래스로 올린다
    * 루트클래스를 화장한 구체 클래스를 의미별로 하나씩 정의한다

```java
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radiusl

    Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * (radius * radius);
    }
}

class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    double area() {
        return length * width;
    }
}
```

- 위와같은 클래스 계층구조는 태그달린 클래스의 단점이 모두 사라진다
- 간결하고 명확하며 쓸데없는 코드는 모두 사라졌다
- 각 필드들이 모두 final로 되었기 때문에, 초기화가 적절하게 이뤄지지 않으면 컴파일러가 잡아줄것이며, 실수로 빼먹은 case문때문에 런타임오류가 발생할 일도 없다
- 루트클래스의 코드를 건드리지 않고도 다른 프로그래머들이 독립적으로 계층구조를 확장하고 함께 사용할수있다
- 타입사이의 자연스러운 계층관계를 반영할 수 있어서 유연성은 물론 컴파일 타입 검사능력도 높여준다는 장점도 있다
- 예컨데 사각형의 특별한 형태인 정사각형은 다음과 같이 쉽게 나타낼 수 있다

```java
class Square extends Rectangle {
    Square(double side) {
        super(side,side);
    }
}
```

