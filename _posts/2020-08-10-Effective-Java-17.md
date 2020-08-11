---
layout: post
title: "Effective Java - 아이템17: 변경 가능성을 최소화 하라"
description: 변경 가능성을 최소화 하라
date: 2020-08-10 20:53:00 +09:00
categories: EffectiveJava Study
---


# 클래스와 인터페이스

## 아이템 17 : 변경 가능성을 최소화 하라

- 불변 클래스란 간단히 말해 그 인스턴스의 내부 값을 수정 할 수 없는 클래스 이다
- 불변 인스턴스에 간직된 정보는 고정되어 객체가 파괴되는 순간까지 절대 달라지지 않는다
- 자바 플랫폼 라이브러리에도 다양한 불변 클래스들이 존재하는데 String, 박싱 클래스, BigInteger, BigDecimal등이 이에 속한다
- 불변 클래스는 가변 클래스보다 설계하고 구현하고 사용하기 쉬우며, 오류가 생길 여지도 적고 훨씬 안전하다
- 클래스를 불변으로 만들려면 다음 5가지 규칙을 따르면 된다
    * 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다
    * 클래스를 확장 할 수 없도록 한다. 상속을 막는 대표적인 방법은 final로 만드는 것이지만, 다른 방법 또한 존재한다
    * 모든 필드를 final로 선언한다. 시스템이 강제하는 수단을 이용해 설계자의 의도를 명확히 드러내는 방법이다. 새로 생성된 인스턴스를 동기화 없이 다른 스레드로 건네도 문제없이 동작하게끔 보장하는데도 필요하다
    * 모든 필드를 private로 선언한다. 필드가 참조하는 가변 객체를 클라이언트에서 직접 접근해 수정하는일을 막아준다. 기술적으로는 기본 타입필드나 불변 객체를 참조하는 필드를 public final로 선언해도 되지만,
    그럴 경우 다음 릴리즈에서 내부 표현을 바꾸지 못하므로 권하지 않는다
    * 자신 외에는 내부의 가편 컴포넌트에 접근 할 수 없도록 한다. 클래스에 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 그 객체의 참조를 얻을 수 없도록 해야한다

```java
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart() { return re; }
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im, re * c.im + im * c.re);
    }

    public Complex divideBy(Complex c) {
        double tmp = c.re * c.re + c.im + c.im;
        return new Complex((re * c.re + im * c.im) / tmp, (im * c.re - re * c.im)/tmp);
    }

    @Override
    public boolean equals(Object o) {
        if( o == this) return true;
        if(!( o instanceof Complex)) return false;
        Complex c = (Complex) o;
        return Double.compare(c.re, re) == 0 && Double.compare(c.im,im) == 0;
    }

    @Override
    public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override
    public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```

- 이 클래스는 복소수를 표현하며, Object의 메서드 몇개를 재정의 했고 실수부와 허수부 값을 반환하는 접근자 메서드, 사칙연산 메서드들을 정의 했다
- 사칙연산 메서드들이 인스턴스 자신은 수정하지 않고 새로운 Complex 인스턴스를 만들어 반환하는것에 주목해야 한다. 이처럼 피연산자에 함수를 적용해 그 결과를 반환하지만 피연산자 자체는 그대로인것이 함수형 프로그래밍이다
- 불변객체는 생선된 시점의 상태를 파괴될 때까지 그대로 간직하므로 단순하며, 근본적으로 스레드안전하기 떄문에 따로 동기화 할 필요가 없다. 그러므로 언심하고 공유 할 수 있다
- 불변클래스는 자주 사용되는 인스턴스를 캐싱하여 인스턴스를 중복 생성하지 않게 해주는 정적 팩터리를 제공 할 수 있다. 이런 정적 팩터리를 통해 메모리 사용량과 가비지 컬렉션을 줄일 수 있다
- 불변 객체를 자유롭게 공유 할 수 있기 떄문에, 방어적 복사도 필요 없다. 아무리 복사해봐야 원본과 똑같기 때문에 복사 자체가 의미가 없다
- 불뱐 객체는 자유롭게 공유 할 수 있음은 물론, 불변 객체 끼리는 내부 데이터를 공유 할 수 있다
- 객체를 만들 떄 다른 불변 객체들을 구성요소로 사용하면 이점이 많다. 값이 바뀌지 않는 구성요소로 이뤄져있다면, 그 구조가 아무리 복잡하더라도 불변식을 유지하기 훨씬 수월하기 떄문이다
- 불변 객체는 그 자체로 실패 원자성을 제공한다. 메서드에서 예외가 발생해도 그 객체는 여전히 메서드 호출 전과 똑같은 유효한 상태이다
- 불변 클래스에도 단점은 있는데, 값이 다르면 반드시 독립된 객체로 만들어야 한다는 것이다. 값의 가짓수가 많다면 이들을 모두 만드는데 큰 비용을 치뤄야 하기 떄문이다.  이는 원하는 객체를 완성하기까지의 단계가 많고, 그 중간 단계에서 만들어진 객체들이 모두 버려진다면 성능 문제가 더 불거진다
    * 다단계 연산들을 예측하여 기본 기능으로 제공하는 방법으로, 각 단계마다 객체를 생성하는것을 피할 수 있다

- 불변 클래스를 만드는 또다른 방법으로, 모든 생성자를 private 혹은 package-private로 만들고, public 정적 팩터리 메서드를 제공하는 방법도 있다. 사실 이 방식이 최선일떄가 많다

```java
public class Comples {
    private final double re;
    private final double im;

    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public static Complex valueOf(double re, double im) {
        return new Complex(re,im);
    }
}
``` 

- 바깥에서 볼 수 없는 pacakge-private 구현 클래스를 원하는 만큼 만들어 활용 할 수 있으니 훨씬 유연하다. 패키지 바깥의 클라이언트에서 바라본 이 불변객체는 사실상 final이기 때문이다. 하지만 내부 구현은 바꿀 수 있다
- BigInteger와 BigDecimal을 설계할 당시엔 이런 생각이 널리 퍼져있지 않아, 이 두 클래스의 메서드들은 모두 재정의 할 수 있게 되었고, 하위호환성 때문에 아직까지 고쳐지지 못하고 있다.
따라서 신뢰할수 없는 클라이언트로부터 이 클래스들의 인스턴스들의 인수로 받는다면 주의 해야 한다. 이떄 방어적 복사를 사용 해야 안전하다
- 이번 아이템의 초입에서 나열한 규칙중 모든 필드가 final이고 어떤 메서드도 그 객체를 수정할 수 없어야 한다 라는 규칙이 있는데, 사실 이는 성능을 위해 "어떤 메서드도 객체의 상태중 외부에 비치는 값을 변경 할 수 없다" 정도로 낮춰도 된다
- 직렬화를 할 떄는 추가로 주의 할 점이 있는데, Serializable을 구현하는 불변 클래스의 내부에 가변 객체를 참조하는 필드가 있다면, readObject나 readResolve 메서드를 반드시 제공하거나, 
ObjectOutputStream.writeUnshared와 ObjectInputStream.readUnshared를 사용해야 한다. 그렇지 않으면 공격자가 이 클래스로부터 가변 인스턴스를 만들어 낼 수 있다
- 정리 해보면 getter가 있다고 무조건 setter를 만들면 안된다. 클래스는 꼭 필요한 경우가 아니라면 불변이여야 하며, 불변 클래스는 장점이 많고 단점이라면 특정 상황에서의 성능저하 뿐이다
- 단순 값 객체는 항상 불변으로 만들어야 한다, 하지만 모든 클래스가 불변일 수는 없으므로 뿔변으로 만들수 없는 클래스라도 변경 할 수 있는 부분을 최소한으로 줄여야 한다
- 다른 합당한 이유가 없다면 모든 필드는 private final이여야 하며, 생성자는 불변식 설정이 완료된 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다
- 확실한 이유가 없다면 생성자와 정적 팩터리 외에는 그 어떤 초기화 메서드도 public으로 제공 되면 안된다

