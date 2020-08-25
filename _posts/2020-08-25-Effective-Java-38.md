---
layout: post
title: "Effective Java - 아이템38: 확장할 수 있는 열거타입이 필요하면 인터페이스를 사용하라"
description: 확장할 수 있는 열거타입이 필요하면 인터페이스를 사용하라
date: 2020-08-25 22:24:00 +09:00
categories: EffectiveJava Study
---


# 열거타입과 어노테이션

## 아이템 38 : 확장할 수 있는 열거타입이 필요하면 인터페이스를 사용하라

- 열거타입은 확장할 수 없다는 단점이 한가지 있다. 사실 대부분의 상황에서 열거타입을 확장하는것은 좋지 않은 생각이다. 확장한 타입의 원소는 기반 타입의 원소로 취급하지만, 그 반대는 성립하지 않는다면 이상하지 않은가?
- 하지만 확장할 수 있는 열거 타입이 어울리는 쓰임이 최소한 하나는 있는데, 바로 연산코드(operation code 혹은 opcode)이다
- 연산코드의 각 원소는 특정 기계가 수행하는 연산을 뜻하며, 이따금 API가 제공하는 기본 연신 외에 사용자 확장 연산을 추가할 수 있도록 열어줘야 할 때가 있다
- 열거 타입으로 이 효과를 내는 방법이 있는데, 열거타입도 클래스인것을 이용하여 임의의 인터페이스를 구현할 수 있다는것이다
- 연산 코드용 인터페이스를 정의하고, 열거 타입이 이 인터페이스를 구현하게 하면 된다. 이때 열거 타입이 그 인터페이스의 표준 구현체 역할을 한다

```java
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}
```

- 열거타입인 BasicOperation은 확장할 수 없지만, 인터페이스인 Operation은 확장할 수 있고 이 인터페이스를 연산의 타입으로 사용하면 된다
- 이렇게 하면 Operation을 구현한 또 다른 열거타입을 정의해 기본 타입인 BasicOperation을 대체 할 수 있다

```java
public enum ExtendedOperation implement Operation {
    EXP("^") {
        public double apply(double x, double y) { return Math.pow(x,y); }
    },
    REMAINDER("%") {
        public double apply(double x, double y) { return return x % y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}
```

- 새로 작성한 연산은 기존 연산을 쓰던곳이면 어디든 쓸 수 있다. BasicOperation이 아니라 Operation 인터페이스를 사용하도록 작성되있기만 하면된다
- 개별 인스턴스 수준에서뿐 아니라 타입 수준에서도, 기본 열거타입 대신 확장된 열거타입을 넘겨 확장된 열거타입의 모든 원소를 사용하게 할 수도 있다

```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(ExtendOperation.class, x, y);
}

private static <T extends Enum<T> & Operation> void test (Class<T> opEnumType, double x, double y) {
    for(Operation op : opEnumType.getEnumConstants())
        System.out.printf("%f %s %f = %f %n",x, op, y, op.apply(x,y));
}
```

- main 메서드는 test메서드에 ExtendedOperation의 class리터럴을 넘겨 확장된 연산들이 무엇인지 알려준다
- 여기서 class리터럴은 한정적 타입 토큰 역할을 한다
- opEnumType 매개변수의 선언(```<T extends Enum<T> & Operation> Class<T>```)는 솔직히 복잡한데, Class 객체가 열거타입인 동시에 Operation의 하위 타입이여야 한다는 뜻이다. 열거타입이어야 원소를 순회할 수 있고, Operation이여야 원소가 뜻하는 연산을 수행 할 수 있기 때문이다

```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test (Collection<? extends Operation> opSet) {
    for(Operation op : opSet)
        System.out.printf("%f %s %f = %f %n",x, op, y, op.apply(x,y));
}
```

- Class 객체 대신 한정적 와일드카드 타입이 Collection<? extends Operation>을 넘겨서, 그나마 덜 복잡하고 test 메서드가 살짝 더 유연해졌다
- 여러 구현 타입의 연산을 조합해 호출하게 되었지만, 특정 연산에서는 EnumSet과 EnumMap을 사용하지 못한다
- 인터페이스를 이용해 확장 가능한 열거 타입을 흉내내는 방식에도 한가지 사소한 문제가 있는데, 바로 열거타입끼리 구현을 상속할수 없다는 점이다
- 아무 상태에도 의존하지 않는 경우에는 디폴트 구현을 이용해 인터페이스에 추가하는 방법도 있다
- 반면 Operation에는 연산기호를 저장하고 있는 저장하고 찾는 로직이 BasicOperation과 ExtendedOperation 모두에 들어가야만 한다
- 공유하는 기능이 많다면 그 부분을 별도의 도우미 클래스나 정적 도우미 메서드로 분리하는 방식으로 코드 중복을 없앨 수 있다
- 자바 라이브러리에서도 이 패턴을 쓰는데 java.nio.file.LinkOption 열거타입은 CopyOption, OpenOption 인터페이스를 구현했다