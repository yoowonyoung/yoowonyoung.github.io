---
layout: post
title: "Effective Java - 아이템6: 불필요한 객체 생성을 피하라"
description: 불필요한 객체 생성을 피하라
date: 2020-07-29 21:49:00 +09:00
categories: EffectiveJava Study
---


# 객체의 생성과 파괴

## 아이템 6 : 불필요한 객체 생성을 피하라

- 똑같은 기능의 객체를 매번 생성 하기보다는, 객체 하나를 재사용 하는 편이 나을떄가 많다. 재사용은 빠르고 세련되있다. 특히 불변 객체는 언제든 재사용 할 수 있다

```java
String s = new String("hello")
```

- 위와 같은 문장은 실행 될 때마다 String 인스턴스를 새로 만든다

```java
String s = "hello"
```

- 이 코드는 새로운 인스턴스를 매번 만드는 대신 하나의 String 인스턴스를 사용 한다. 이 방식을 사용하면 같은 JVM안에서 이와 똑같은 문자열 리터럴을 사용하는 모든 코드가 같은객체를 사용함이 보장된다
- 생성자 대신 정적 팩터리 메서드를 제공하는 불변 클래스에서는 정적 팩터리 메서드를 사용해 불필요한 객체 생성을 피할 수 있다
- 생성 비용이 아주 비싼 객체도 더러 있는데, 이런 비싼 객체가 반복해서 필요 하다면 캐싱하여 재사용 하기를 권장한다

```java
static boolean isRomanNumeral(String s) {
    return s.matches( ... );
}
```

- 위의 코드에서 사용하는 String.matches는 정규 표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이 지만, 성능이 중요한 상황에서는 반복해서 사용하기엔 적합하지않다
- 메서드가 내부에서 만드는 정규 표현식용 Pattern 인스턴스는, 한번 쓰고 버려져서 곧바로 가비지 컬렉션의 대상이 되기 떄문이다
- 성능을 개선하기 위해서는 필요한 정규식을 표현하는 불변 Pattern 인스턴스를 클래스 초기화 과정에서 직접 생성해 캐싱해두고, 나중에 isRomanNumeral이 호출 될 때마다 이 인스턴스를 재활용 하면 된다

```java
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.Compile( ... );

    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

- 이렇게 개선된 isRomanNumeral는, 초기화 된 후 이 메서드를 한번도 호출하지 않는다면 ROMAN필드는 쓸데없이 초기화 된 꼴인데, 이를 지연 초기화로 불필요한 초기화를 없앨 수 있지만 권하지는 않는다
- 객체가 불변이라면 재사용해도 안전함이 명백하다. 하지만 훨씬 덜 명확하거나, 심지어 직관에 반대되는 상황도 있다(Map의 keySet 메서드는 매번 같은 Set을 반환한다)
- 불필요한 객체를 만들어내는 또 다른 예로는 오토박싱을 들 수 있는데, 오토박싱은 기본 타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만, 완전히 없애주는 것은 아니다

```java
private static long sum() {
    Long sum = 0L;
    for(long i = 0; i < Integer.MAX_VALUE; i++) sum += i;
    return sum;
}
```

- 위의 프로그램은 정확한 답을 내기는 하지만, sum변수를 long이 아닌 Long으로 선언해서 불필요한 인스턴스가 231개나 만들어진다. 단순히 sum의 타입을 long으로만 바꿔주면 빨라진다
- 이번 아이템을 "객체 생성은 비싸니 피해야한다"로 오해하면 안된다. 특히나 요즘의 JVM은 별다른 일을 하지 않는 작은 객체를 생성하고 회수하는 일이 크게 부담이 되지 않는다
- 이번 아이템은 방어적 복사를 다루는 아이템 50 : 새로운 객체를 만들어야 한다면 기존 객체를 재사용 하지 마라와 대조적이다
    + 방어적 복사가 필요한 상황에서 객체를 재사용 했을때의 피해가, 필요 없는 객체를 반복 생성 했을때 보다 훨씬 더 크다