---
layout: post
title: "Effective Java - 아이템43: 람다보다는 메서드 참조를 이용하라"
description: 람다보다는 메서드 참조를 이용하라
date: 2020-08-30 21:31:00 +09:00
categories: EffectiveJava Study
---


# 람다와 스트림

## 아이템 43 : 람다보다는 메서드 참조를 이용하라

- 자바의 이전버전에서는 함수 타입을 표현할 때 추상 메서드를 하나만 담은 인터페이스를 사용 했다
    * 이런 인터페이스의 인스턴스를 함수 객체라고 하며, 특정 함수나 동작을 나타내는데 썻다

- JDK 1.1 까지만 해도 함수 객체를 만드는 주요 수단은 익명 클래스 였다

```java
Collections.sort(words, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
})
```

- 전략 패턴처럼, 함수 객체를 사용하는 과거 객체지향 디자인 패턴에는 익명 클래스면 충분 했다
- 하지만 익명 클래스 방식은 코드가 너무 길기 때문에 자바는 함수형 프로그래밍에 적합하지 않았다
- 자바8부터는 추상메서드 하나짜리 인터페이스는 특별한 의미를 인정받아 특별한 대우를 받게 되었는데, 함수형 인터페이스라 부르는 이 인터페이스들의 인스턴스를 람다식을 이용해 만들 수 있게 된것이다
- 람다는 함수나 익명 클래스와 개념은 같지만 코드는 훨씬 간결하다

```java
Collections.sort(words, (s1,s2) -> Integer.compare(s1.length(), s2.length()));
```

- 여기서 람다, 매개변수(s1,s2), 반환값의 타입은 각각 Comparator<String>, String, int지만 코드에서는 언급이 없다. 컴파일러가 타입 추론을 해준것이다
    * 컴파일러의 타입 추론 규칙은 매우 복잡하다. 타입을 명시해야 코드가 더 명확할때를 제외하고는, 람다의 모든 매개변수 타입은 생략하자
    * 컴파일러가 타입을 알 수 없다는 오류를 낼때만 해당 타입을 명시하면 된다

- 람다 자리에 비교 생성자 메서드를 사용하면 이 코드는 더 간결해진다

```java
Collections.sort(words, comparingInt(String::length));
```

- 자바8에 List 인터페이스에 추가된 sort 메서드를 이용하면 이는 더욱 짧아진다

```java
words.sort(comparingInt(String::length));
```

- 람다를 언어 차원에서 지원하면서 기존에는 적합하지 않았던 곳에서도 함수 객체를 실용적으로 사용할 수 있게 되었는데, 아이템 34의 Operation 열거 타입의 예시를 들 수 있다

```java
public enum Operaion {
    PLUS("+", (x,y) -> x+y),
    MINOUS("-", (x,y) -> x-y),
    TIMES("*", (x,y) -> x*y),
    DEVIDE("/", (x,y) -> x/y);

    private final String symbol;
    private final String DoubleBinaryOperator op;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override
    public String toString() {
        return symbole;
    }

    public double apply(double x, double y) {
        return op.applyAsDouble(x,y);
    }
}
```

- 이 코드에서 열거 타입 상수의 동작을 표현한 람다를 DoubleBinaryOperator 인터페이스 변수에 할당했다. 이는 java.util.function 패키지가 제공하는 다양한 함수 인터페이스중 하나로 Double타입 인수를 2개 받아 Double 타입 결과를 돌려준다
- 이 코드는 각 열거 타입 상수의 동작을 람다로 구현해 생성자에 넘가고, 생성자는 이 람다를 인스턴스 필드로 저장해둔다. 그 다음 apply 메서드에서 필드에 저장된 람다를 호출하고 있다
- 람다 기반의 Operation 열거 타입을 보면, 상수별 클래스 몸체는 더이상 사용할 이유가 없다고 보이지만 실제로는 그렇지 않다. 람다는 이름이 없고, 문서화도 못하기때문에, 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄수가 많아진다면 람다를 사용하지 말아야 한다
- 람다는 한줄일때 가장 좋으며, 세줄 안에는 끝내는게 좋다
- 열거타입 생성자에게 넘겨지는 인수들의 타입도 컴파일 타임에 추론되기 때문에, 열거 타입 생성자 안의 람다는 열거 타입의 인스턴스 멤버에 접근 할 수 없다
    * 따라서 인스턴스 필드나 메서드를 사용해야만 하는 상황이라면 상수별 클래스 몸체를 사용해야 한다

- 람다는 함수형 인터페이스에서만 쓰이기 때문에, 추상클래스의 인스턴스를 만들때 람다를 쓸 수 없으니 익명 클래스를 써야 한다
- 비슷하게 추상 메서드가 여러개인 인터페이스의 인스턴스를 만들때에도 익명 클래스를 쓸 수 있다
- 마지막으로 람다는 자기 자신을 참조할수가 없다. 람다에서의 this는 바깥 인스턴스를 가리키기 떄문에, 함수 객체가 자기 자신을 참조해야한다면 익명 클래스를 사용해야 한다
- 람다도 익명 클래스처럼 직렬화 형태가 구현별로 다를 수 있기 때문에, 람다를 직렬화 하는 일은 극히 삼가해야 한다. 직렬화 해야한 하는 함수 객체가 있다면, private 정적 중첩 클래스의 인스턴스를 사용해야 한다
