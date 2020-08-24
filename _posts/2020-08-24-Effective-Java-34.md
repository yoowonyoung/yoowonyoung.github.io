---
layout: post
title: "Effective Java - 아이템34: int 상수대신 열거 타입을 사용하라"
description: int 상수대신 열거 타입을 사용하라라
date: 2020-08-24 00:00:00 +09:00
categories: EffectiveJava Study
---


# 열거타입과 어노테이션

## 아이템 34 : int 상수대신 열거 타입을 사용하라

- 열거타입은 일정 개수의 상수값을 정의한 다음 그 외의 값은 허용하지 않는 타입이다. 자바에서 열거타입을 지원하기 전까지는 정수 상수를 여러개 선언해서 사용하곤 했다
- 정수 열거 패턴에는 단점이 많다. 타입 안전이 보장되지 않으며, 표현력도 좋지 않다
- 자바가 정수 열거 패턴을 위한 별도 이름공간을 지원하지 않기 때문에 접두어를 사용해서 이름 충돌을 막고는 했다
- 정수 열거 패턴을 사용한 프로그램은 깨지기도 쉬운데, 평범한 상수를 나열한것 뿐이라 컴파일 하면 그 값이 클라이언트 파일에 그대로 새겨지며, 상수의 값이 바뀌면 클라이언트도 다시 컴파일 해야한다
- 또, 정수 상수는 문자열로 출력하기가 다소 까다로운데, 그 값을 출력하거나 디버거로 살펴보면 그 의미가 아닌 단순한 숫자로 보여 썩 도움이 되지 않는다
- 정수 대신 문자열 상수를 이용하는 변형된 물자열 열거 패턴이 있는데 이는 더욱 나쁘다. 상수의 의미는 출력이 되지만, 문자열 상수의 이름대신 문자열 값 그대로 하드코딩하게 만들수 있기 때문이다
- 이러한 열거 패턴의 단점을 말끔히 없애주는것이 바로 열거 타입이다

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLATE, BLOOD }
```

- 겉보기에는 C,C++,C# 등과 같은 다른 언어의 열거타입과 비슷해보이지만, 보이는것이 다는 아니다
- 자바의 열거 타입은 완전한 형태의 클래스여서 단순한 정수값일 뿐인 다른 열거타입보다 훨씬 강력하다
- 자바의 열거 타입을 뒷받침 하는 아이디어는 단순한데, 열거타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final필드로 공개하는것이다
- 열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final이며, 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없으니 열거타입으로 만들어진 인스턴스들은 딱 하나씩만 존재함이 보장된다
    * 싱글턴은 원소가 하나뿐인 열거타입이라고 볼 수 있고, 열거타입은 싱글턴을 일반화한 형태라 볼 수 있다

- 열거 타입은 컴파일타임 타입 안정성이 제고 되는데, 위의 예시에서 Apple열거 타입을 매개변수로 받는 메서드를 선언했다면, 건네받은 참조는 null이 아닐경우 Apple의 세가지 값중 하나임이 확실하다
- 열거 타입에는 각자의 이름 공간이 있어서 이름이 같은 상수도 평화롭게 공존하며, 열거타입에 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일 하지 않아도 된다. 공개되는것이 오직 필드의 이름뿐이라, 상수값이 클라이언트로 컴파일되어 각인되지 않기 때문이다
- 또한 열거타입의 toString 메서드는 출력하기에 적합한 문자열도 제공해준다
- 열거타입은 이처럼 정수 열거 패턴의 단점들을 해소해주며, 열거타입에는 임의의 메서드나 필드를 추가할 수 있고, 임의의 인터페이스도 구현 가능하다
- 단순히 상수 모음일 뿐인 열거타입이지만 실제로는 클래스이므로 추상 개념하나를 완벽히 표현할 수 있다

```java
public enum Planet {
    MECURY(3.302e+23, 2.439e6),//생성자가 호출
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6),
    ...;

    private final double mass;
    private final double radius;
    private final double surfaceGravity;

    private static final double G = 6.67300E-11;

    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass() { return mass; }
    public double radius() { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;
    }
}
```

- 위의 예시는 태양계를 나타낸 열거타입인데, 각 행성에는 질량과 반지름이 있고, 이 두속성을 이용해 표면 중력을 계산할 수 있다. 따라서 어떤 객체의 질량이 주어지면 그 객체가 행성 표면에 있을때의 무게도 계산 할 수 있다
- 위처럼 열거타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다
- 열거타입은 근본적으로 불변이라 모든 필드는 final이여야 하고, 필드를 public으로 선언해도 되지만, private로 두고 별도 public 접근자를 두는게 낫다
- 이 Planet 열거타입은 단순하지만 놀랍도록 강력한데, 어떤 객체의 지구에서의 무게를 입력받아 여덟 행성에서의 무게를 출력하는일도 다음처럼 짧게 구현 가능하다

```java
public class WeightTable {
    public static void main(String[] args) {
        double earthWeight = Double.parseDouble(args[0]);
        double mass = earthWeight / Planet.EARTH.surfaceGravity();
        for(Planet p : Planet.values()){
            System.out.printf("%s에서의 무게는 %f 이다 %n",p,p.surfaceWeight(mass));
        }
    }
}
```

- 열거 타입은 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드인 values를 제공한다. 값은 선언된 순서로 저장된다
- toString 메서드는 상수 이름을 문자열로 반환하므로 println과 printf로 출력하기에 적합하다
- 열거 타입에서 상수하나를 제거할경우, 해당 제거한 상수를 참조하지 않는 클라이언트에는 아무런 영향이 없다. 참조하는 클라이언트는 컴파일 오류가 발생할것이다
- 열거타입을 선언한 클래스 혹은 그 패키지에서만 유용한 기능은 다른 클래스와 마찬가지로 private나 package-private로 구현하면 된다
- 널리 쓰이는 열거 타입은 톱레벨 클래스로 만들고, 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 멤버 클래스로 만들면 된다
    * 예를 들면 소수 자릿수의 반올림 모드를 뜻하는 열거타입인 RoundingMode는 BigDecimal이 사용하는데, 반올림 모드는 BigDecimal과 관련없는 영역에서도 유용한 개념이라 RoudingMode은 톱레벨로 선언 되어있다

- 한걸음 더 나아가 상수마다 동작이 달라져야 하는 상황도 있을 수 있다

```java
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE;

    public double apply(double x, double y) {
        switch(this) {
            case PLUS: return x + y;
            case MINUS: return x - y;
            case TIMES: return x * y;
            case DIVIDE: return x / y;
        }
        throw new AssertionError("알 수 없는 연산 : " + this);
    }
}
```

- 위 코드는 사칙연산 계산기의 연산 종류를 열거 타입으로 선언하고, 실제 연산까지 열거 타입 상수가 직접 수행하는 코드이다. 하지만 이 코드는 동작은 하지만 그리 예쁘지는 않다. throw문은 실제로 도달할 수 없으나 기술적은 도달할 수 있기때문에 throw가 없으면 컴파일이 되지 않고, 깨지기 쉬운 코드라는 것이다
- 다행이 열거 타입은 상수별로 다르게 동작하는 코드를 구현하는 수단을 제공한다. 열거 타입에 apply라는 추상메서드를 선언하고 각 상수별 클래스 몸체에서 재정의 하는 것이다. 이것이 상수별 메서드 구현이다

```java
public enum Operation {
    PLUS {public double apply(double x, double y){return x + y;}},
    MINUS {public double apply(double x, double y){return x - y;}},
    TIMES {public double apply(double x, double y){return x * y};},
    DIVIDE {public double apply(double x, double y,){return x / y};};

    public abstract double apply(double x, double y);
}
```

- 보다시피 apply는 상수 선언 바로 옆에 붙어있으니 새로운 상수를 추가할 때 apply도 재정의 해야한다는 사실을 깜빡하기 어려울 뿐더러, apply가 추상메서드 이므로 재정의 하지 않았다면 컴파일 오류로 알려준다
- 상수별 메서드 구현을 상수별 데이터와 결합할 수도 있다

```java
public enum Operation {
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

    Operation(String symbol) { this.symbol = symbol; }

    @Override
    public String toString() { return symbol; }
    public abstract double apply(double x, double y);
}
```

- 다음은 이 toString이 계산식 출력을 얼마나 편하게 해주는지를 보여준다

```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    for(Operation op : Operation.values()) {
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x,y));
    }
}
```

- 열거 타입에는 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 valueOf(String)이 자동으로 생성된다
- 열거 타입의 toString을 재정의 하려거든, 다음과 같이 toString이 반환하는 문자열을 해당 열거 타입 상수로변환해주는 fromString도 함께 제공 하는것을 고려해보자

```java
private static final Map<String, Operation> stringToEnum = 
    Stream.of(values()).collect(toMap(Object::toString, e -> e));//Stream 활용

public static Optional<Operation> fromString(String symbol) {//주어진 문자열이 가르키는 연산이 존재하지 않을수 있으므로 Optional
    return Operation.ofNullable(stringToEnum.get(symbol));
}
```

- Operation 상수가 stringToEnum 맵에 추가되는 시점은 열거 타입 상수 생성후 정적 필드가 초기화 될 때이다
- 열거 타입의 정적 필드중 열거 타입의 생성자에서 접근 할 수 있는것은 상수 변수 뿐이다
- 한편 상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다

```java
enum PayrollDay {
    MONDAY, THESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;

    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;

        int overtimePay;
        switch(this) {
            case SATURDAY: case SUNDAY:
                overtimePay = basePay/2;
                break;
            default:
                overtimePay = minutesWorked <= MINS_PER_SHIFT ? 0 : 
                    (minutesWorked - MINS_PER_SHIFT) * payRate /2;
        }

        return basePay + ovetimePay;
    }
}
```

- 급여명세서에 쓸 요일을 표현하는 열거 타입의 예시이다. 이 열거 타입은 시간당 기본 임금과 그날 일한 시간이 주어지면 계산하는 메서드를 가지고 있다
- 분명 간결하지만, 관리 관점에서는 위험한 코드이다. 휴가와 같은 새로운 값을 열거 타입에 추가하려면 그 값을 처리하는 case문을 잊지 말고 쌍으로 넣줘야 한다
- 상수별 메서드 구현으로 급여를 정확히 계산하는 방법으로는 두가지 인데, 첫번째로는 잔업수당을 계산하는 코드를 모든 상수에 중복해서 넣는것이고, 두번째는 계산 코드를 평일용과 주말용ㅇ로 나눠 각각 도우미 메서드로 작성한 다음 각 상수가 자신에게 필요한 메서드를 적절히 호출해주는 것이다
    * 당연하게도 두 방법 모두 코드가 장황해지므로 가독성이 크게 떨어지고 오류 발생 가능성이 높아진다

- PayrollDay에 평일 잔업 수당 계산용 메서드인 overtimePay를 구현해놓고, 주말 상수에서만 재정의 해서 쓰면 장황한 부분은 줄어들지만 switch문을 썻을때와 똑같은 단점이 나타난다. 즉 새로운 상수를 추가하며 overtimePay를 재정의 하지 않으면 평일용 코드가 그대로 물려받아진다
- 가장 깔끔한 방법은 새로운 상수를 추가할때마다 잔업수당 '전략'을 선택 하도록 하는것이다

```java
enum PayrollDay {
    MONDAY(WEEKDAY), THESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY), SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType;}

    int pay(int minutesWorkded, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    //전략 열거타입
    enum PayType {
        WEEKDAY {
            int overtimePay(int minutesWorkded, int payRate) {
                return overtimePay = minutesWorked <= MINS_PER_SHIFT ? 0 : 
                    (minutesWorked - MINS_PER_SHIFT) * payRate /2;
            }
        },
        WEEKEND {
            int overtimePay(int minutesWorkded, int payRate) {
                return overtimePay = basePay/2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minutesWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
```

- 기존의 switch문은 열거 타입의 상수별 동작을 구현하는데 적합하지 않지만, 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch문이 좋은 선택이 될 수 있다

```java
public static Operation inverse(Operation op) {
    switch(op) {
        case PLUS: return Operation.MINUS;
        case MINUS: return Operation.PLUS;
        case TIMES: return Operation.DIVIDES;
        case DIVIDES: return Operation.TIMES;

        default: throw new AssertionError("알수없는 연산 " + op);
    }
}
```

- 서드파티에서 가져온 Operation이있고 각각의 연산의 반대 연산을 하는 메서드가 필요하다고 하면 위처럼 하면 된다
- 추가하려는 메서드가 의미상 열거 타입에 속하지 않는다면 직접 만든 열거타입이라도 이 방식을 적용하는게 좋다
- 대부분의 경우에는 열거 타입의 성능은 정수 상수와 별반 다르지 않다
- 필요한 원소를 컴파일 타임에 다 알수 있는 상수 집합이라면 항상 열거타입을 사용하는게 좋다
- 열거타입에 정의된 상수 개수가 영원히 고정일 불변일 필요는 없다. 열거타입읜 나중에 상수가 추가되도 바이너리 수준에서 호환되도록 설계되어있다
