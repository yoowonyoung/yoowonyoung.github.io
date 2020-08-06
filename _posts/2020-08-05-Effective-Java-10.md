---
layout: post
title: "Effective Java - 아이템10: equals는 일반 규약을 지켜 재정의 하라"
description: equals는 일반 규약을 지켜 재정의 하라
date: 2020-08-05 20:00:00 +09:00
categories: EffectiveJava Study
---


# 모든 객체의 공통 메서드

## 아이템 10 : equals는 일반 규약을 지켜 재정의 하라

- equals는 재정의 하기 쉬워 보이지만 다음과 같은 경우에는 재정의 하지 않는것이 최선이다
    * 각 인스턴스가 본질적으로 고유하다. 값을 표현하는게 아니라 Thread와 같이 동작하는 개체를 표현하는 클래스가 여기에 해당한다
    * 인스턴스의 논리적 동치성(logical equality)를 검사할 일이 없다. java.util.regex.Pattern의 equals와 같이 같은 정규표현식을 검사하는 경우가 아닌 경우
    * 상위 클래스에서 재정의한 equals가 하위 클래스에서도 딱 들어 맞는다
    * 클래스가 private이거나 package-private이고 equals메서드를 호출 할 일이 없다
        + 이와 같은 경우는 equals를 재정의해 예외를 던짐으로써 막아버릴 수 있다

- equals는 객체 식별성(두 객체가 물리적으로 같은지)이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스가 논리적 동치성을 비교하도록 재정의 하지 않았을때 재정의 해야 한다
- 주로 값 클래스(Integer, String 등)과 같은 클래스들이 여기에 해당하고, equals를 재정의 해두면 그 인스턴스는 값을 비교하길 원하는 프로그래머의 기대에 부흥 할 뿐만 아니라, Map의 키와 Set의 원소로 쓸 수 있다
- 값 클래스라 해도, 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스라면 equals를 재정의 하지 않아도 된다. Enum도 이 경우에 해당한다
- equals를 재정의 할 때에는 반드시 일반 규약에 따라야 하며, 다음은 object에 적힌 명세 규약이다
    * equals 메서드는 동치관계(equivalence releation)를 구현하며 다음을 만족 한다
    * 반사성(reflexivity): null이 아닌 참조값 x에 대해, x.equals(x)는 true이다
    * 대칭성(symmetery): null이 아닌 모든 참조값 x,y에 대해 x.equals(y)가 ture이면 y.equals(x)도 true이다
    * 추이성(transitivity): null이 아닌 모든 참조값 x,y,z에 대해 x.equals(y)가 ture이고 y.equals(z)도 true이면 x.equals(z)도 true이다
    * 일관성(consistency): null이 아닌 모든 참조값 x,y,에 대해 x.equals(y) 반복해서 호출하면 항상 true거나 항상 false를 반환해야 한다
    * null 아님: null이 아닌 모든 참조값 x에 대해, x.equals(null)은 false이다

- 동치 관계란 쉽게 말해 집합을 서로 같은 원소들로 이뤄진 부분 집합으로 나누는 연산이고, equals 메서드가 쓸모 있으려면 모든 원소가 같은 동치류에 속한 어떤 원소와도 교환 할 수 있어야 한다
- 반사성은 단순히 말하면 객체는 자기 자신과 같아야 한다는 뜻이고, 이는 일부러 어기는 경우가 아니라면 만족시키지 못하기가 더욱 어렵다
- 대칭성은 두 객체는 서로에 대한 동치 여부에 대해 똑같이 답해야 한다는 뜻이다. 대칭성 요건은 다음과 같은 경우 자칫 잘못하면 어길 수 있다

```java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requiredNonNull(s);
    }
    //대칭성 위배!!
    @Override
    public boolean equals(Object o) {
        if(o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(((CaseInsensitiveString)0).s);
        if(o instanceof String) 
            return s.equalsIgnoreCase((String) o);
        return false;
    }
}
```

- 위와 같은 경우 CaseInsensitiveString는 String을 알고 있지만, String은 CaseInsensitiveString을 알고 있지 못해 다음과 같은 경우 대칭성이 성립하지 않는다

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Hello");
String s = "Hello";
cis.equals(s) //true
s.equals(cis) //false
```

- 위와 같이 equals 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할지 알 수 없다
- 추이성은 첫번째 객체와 두번째 객체가 같고, 두번째 객체와 세번째 객체가 같다면, 첫번째 객체와 세번째 객체도 같아야 한다는 뜻이다
- 이 요건도 간단하지만 자칫하면 어기기 쉽다

```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if(!(o instanceof point)) 
            return false;
        Point p = (Point)o;
        return p.x == x && p.y == y;
    }
}
```

- 위와 같은 클래스를 확장해서 점에 색상을 더해보자

```java
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x,y);
        this.color = color;
    }
}
```

- 이것을 그대로 두면 Point의 구현이 상속되어 색상 정보는 무시한채 비교가 수행된다. 이를 또다른 ColorPoint와 위치와 색상이 같을떄만 true를 반환하는 equals를 생각 해보자

```java
//대칭성 위배
@Override
public boolean equals(Object o) {
    if(!(o instanceof ColorPoint))
        return flase;
    return super.equals(o) && ((ColorPoint)o).color == color;
}
```

- 이 메서드는 일반 Point를 ColorPoint에 비교한 결과와 그 둘을 바꿔 비교한 결과와 다를 수 있다
- 그렇다면 ColorPoint와 Point를 비교할때 색상을 무시 하도록 하면 어떨까? 그러면 대칭성이 지켜지지만 추이성이 위배된다

```java
//추이성 위배
@Override
public boolean equals(Object o) {
    if(!(o instanceof Point))
        return flase;
    if(!(o instanceof ColorPoint))
        return o.equals(this);
    return super.equals(o) && ((ColorPoint)o).color == color;
}
```

- 안타깝게도 이 문제는 모든 객체지향 언어의 동치관계에서 나타나는 근본적인 문제며, 구체 클래스를 확장해서 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다
- getClass를 이용해서 해결 하는 듯 보이나, 이는 리스코프 치환 원칙에 위배된다
- 하지만 괜찮은 우회방법이 하나 있는데, 상속 대신 컴포지션을 이용하면 된다. Point를 상속하는 대신 Point를 Color의 private 필드로 두고 ColorPoint와 같은 위치의 일반 Point를 반한하는 view 메서드를 public으로 추가하는 식이다

```java
public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x,y);
        this.color = Objects.requireNotNull(color);
    }

    public Point asPoint() {
        return point;
    }

    @Override
    public boolean equals(Object o) {
        if(!(o instanceof ColorPoint))
            return flase;
        ColorPoint cp = (ColorPoint) o;
        return cp.pount.equals(point) && cp.color.equals(color);
    }   
}
```

- 자바 라이브러리에도 구체 클래스를 확장해 값을 추가한 클래스가 종종 있는데, Timestamp는 Date를 확장해 nanoseconds 필드를 추가했다. 그 결과로 Timestamp의 equals는 대칭성을 위배한다
- 일관성은 두 객체가 같다면 앞으로도 영원이 같아야 한다는뜻인데, 가변 객체는 클래스 비교 시점에 따라 서로 다를수도 혹은 같을수도 있는 반면, 불변 객체는 한번 다르면 끝까지 달라야 한다
- 클래스가 불변이든 가변이든 equals의 판단에 신뢰 할 수 없는 자원이 끼어들게 해서는 안되며, 이 제약을 어기면 일관성 조건을 만족시키기가 아주 어렵다
    * URL의 equals는 주어진 URL과 매핑된 호스트의 IP주소를 이용해 비교 하는데, 호스트 이름을 IP주소로 바꾸려면 네트워크를 통해야 하므로 그 결과가 항상 같다고 보장 할 수는 없다. 이는 문제이고 따라하면 안된다

- null아님은 이름처럼 모든 객체가 null과 같지 않아야 한다는 뜻인데, 이러한 상황은 상상하기 어렵지만 실수로 Null Pointer Exception을 던지는 코드에서도 이 경우를 허용하지 않는다
- 따라서 equals는 다음과 같은 단계로 구현 되어야 한다

1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다. 이는 성능 최적화에 좋다
2. instanceof 연산자로 입력이 올바른 타입인지 검사한다
3. 입력을 올바른 타입으로 형변환 한다
4. 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다. 하나라도 다르면 false를 반환한다

- float와 double을 제외한 기본 타입 필드는 == 으로 검사하고, float와 double은 부동소수점을 다뤄야 하므로 Float.compate, Double.compare로 비교한다
- 앞서 CaseInsensitiveString 예처럼 비교하기가 아주 복잡한 필드를 가진 클래스는, 그 필드의 표준형을 저장 해둔 후 표준형 끼리 비교하면 훨씬 수월하다
- 어떤 필드를 먼저 비교하느냐가 equals의 성능을 좌우하기도 하는데, 최상의 성능을 원한다면 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교하자
- equals를 구현 했다면, 대칭적인지, 추이성이 있는지, 일관성이 있는지에 대한 자문을 해보고 단위 테스트를 작성해 확인하자
- 다음은 모범적으로 작성된 PhonNumber 클래스용 equals 이다

```java
public final class PhonNumber {
    private final short areaCode, prefix, lineNum;

    public PhonNumber(int areaCode, int prefix, int lineNul) {
        this.areaCode = rangeCheck(areaCode,999,"지역코드");
        this.prefix = rangeCheck(prefix,999,"프리픽스");
        this.lineNum = rangeCheck(lineNum,999,"가입자 번호");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if(val < 0 || val > max)
            throw new IlligalArgumentException(arg + ": " + val);
        return (short)val;
    }

    @Override
    public boolean equals(Object o) {
        if(o == this)
            return true;
        if(!(o instanceof PhonNumber)) 
            return false;
        PhonNumber pn = (PhonNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode;
    }
}
```

- 마지막으로 다음과 같은 주의사항이 있다
    * equals를 재정의 할 떄 hashCode도 반드시 재정의 해야 한다
    * 너무 복잡하게 해결하려 들지 말자. 필드의 동치성만 검사해도 equals 규약을 어렵지않게 지킬 수있다
    * Object 타입외의 타입을 매개변수로 받는 euqals 메서드는 선언하지 말자. 이는 Object.equals의 재정의가 아닌 다중정의가 되며, 이는 문제를 야기할 수 있다

- 꼭 필요한 경우가 아니면 equals를 재정의 하지말자. 많은 경우에 Object의 equals가 원하는 비교를 정확히 수행 해주며, 재정의 해야 할때는 규약을 확실히 지켜가며 해야 한다

