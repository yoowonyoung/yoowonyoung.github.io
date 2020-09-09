---
layout: post
title: "Effective Java - 아이템60: 정확한 답이 필요하면 float와 double은 피해라"
description: 정확한 답이 필요하면 float와 double은 피해라
date: 2020-09-09 20:17:00 +09:00
categories: EffectiveJava Study
---


# 일반적인 프로그래밍 원칙

## 아이템 60 : 정확한 답이 필요하면 float와 double은 피해라

- float와 double은 과학과 공학 계산용으로 설계되엇다. 이진 부동소숫점 연산에 쓰이며, 넓은 범위의 수를 빠르게 정밀한 '근사치'로 계산 할 수 있도록 세심하게 설계되었다
- '근사치' 이기 때문에 정확한 결과가 필요할때는 사용하면 안된다. 특히 금융 관련 계산에 float와 double은 적합하지 않다. 0.1 혹은 10의 음수 거듭제곱수를 표현할 수 없기 때문이다
- 예를들어 주머니에 1.03달러가 있고 그중 42센트를 썻다고 해보자. 이때 남은돈을 구하기 위한 어설픈 코드는 다음과 같다

```java
System.out.println(1.03 - 0.42);
```

- 안타깝게도 이 코드는 0.610000000001을 출력할것이다. 또 다른 예시로 주머니에 1달ㄹ가 있다고 할때 10센트짜리 사탕 9개를 삿다고 하면 얼마가 남을까?

```java
System.out.println(1.00 - 9 * 0.10);
```

- 이 코드 또한 역시 0.09999999999998을 출력한다. 결과값을 반올림하면 되지 않을까 하는 생각을 가질수도 있지만 이는 정확하지 않다
- 예를들어 1달러를 가지고 있을때, 10센트, 20센트, 30센트... 1달러 짜리 사탕을 10센트부터 하나씩 산다고 하면 몇개나 살 수 있을까에 대한 코드이다

```java
public static void main(String[] args) {
    double funds = 1.00;
    int itemsBought = 0;
    for(double price = 0.10; funds >= price; price += 0.10) {
        funds -= price;
        itemBought++;
    }
    System.out.println(itemBought + "개 구입");
    System.out.println("잔돈 :" + funds);
}
```

- 이 코드는 사탕 3개를 구입하고 잔돈을 0.3999999999999 달러가 남앗다고 알려줄것이다. 물론 잘못되었다!
- 이런 금융 관련 계산에는 BigDecimal 혹은 int, long을 사용해야 한다
- 다음 코드는 위 코드에서 double을 BigDecimal로 변경한 코드이다. BigDecimal의 생성자로 문자열을 받았음에 주목해야한다. 이것이 부정확한 값을 사용되는걸 막기위한 조치이다

```java
public static void main(String[] args) {
    final BigDecimal TNE_CENTS = new BigDecimal(".10");

    BigDecimal funds = new BigDecimal("1.00");
    int itemsBought = 0;
    for(BigDecimal price = TEN_CENTS; funds.compareTo(price) >= 0; price.add(TEN_CENTS)) {
        funds.subtract(TEN_CENTS);
        itemBought++;
    }
    System.out.println(itemBought + "개 구입");
    System.out.println("잔돈 :" + funds);
}
```

- 이 코드는 실행하면 사탕을 4개 구입하고 잔돈은 0원이 남았다고 정확히 알려줄것이다. 드디어 올바른 답인것이다
- 하지만 BigDecimal에는 2가지 단점이 있는데, 기본 타입보다 쓰기가 훨씬 불편하며, 훨씬 느리다
- BigDecimal 대신 int 혹은 long을 사용할 수 있는데, 그럴경우 다룰 수 있는 값의 크기가 제한되고 소수점을 직접 관리 해야한다
- 이처럼 정확한 답이 필요한 계산에는 float나 double은 피해야 하며, 코딩시의 불편함이나 성능이 문제되지 않는다면 BigDecimal을 쓰자
- BigDecimal이 제공하는 8가지 반올림모드는 반올림을 완벽히 제어해주며, 법으로 정해진 반올림 계산을 수행해야하는 비즈니스 로직에서는 아주 편리하다
- 반면 성능이 중요하다면 int나 long을 쓰면서 소수점을 직접 추적 하자. 열 여덟자리가 넘는 수에는 어쩔수없이 BigDecimal을 써야한다