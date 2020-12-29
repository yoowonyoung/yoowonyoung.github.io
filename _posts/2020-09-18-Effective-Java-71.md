---
layout: post
title: "Effective Java - 아이템71: 필요없는 검사 예외 사용은 피하라"
description: 필요없는 검사 예외 사용은 피하라
date: 2020-09-18 23:46:00 +09:00
categories: EffectiveJava Study
---


# 예외

## 아이템 71 : 필요없는 검사 예외 사용은 피하라

- 검사 예외를 실어하는 자바 프로그래머가 많지만 제대로 활용하면 API와 프로그램의 질을 높일 수 있다
- 결과를 코드로 반환하거나 비검사 예외를 던지는것과 달리, 검사 예외는 발생한 문제를 프로그래머가 처리하여 안정성을 높이게끔 해준다
- 물론 검사예외를 과하게 사용하면 오히려 쓰기 불편한 API가 된다
    * 어떤 메서드가 검사 예외를 던질 수 있다면, 이를 호출하는 코드에서는 catch블록을 두어 그 예외를 붙잡아 처리하고나 더 바깥으로 전파해야한다
    * 어느쪽이든 API 사용자에게 부담을 준다

- API를 제대로 사용해도 발생할 수있는 예외이거나, 프로그래머가 의미 있는 조치를 취할 수 있는 경우라면 이정도 부담쯤은 받아 들일 수 있을 것이다
    * 그러나 둘 중 어디에도 해당하지 않는다면 비검사 예외를 사용하는게 좋다

- 검사예외가 프로그래머에게 지우는 부담은 메서드가 단 하나의 검사 예외만 던질때가 특히 크다
    * 이미 다른 검사예외도 던지는 상황에서 또다른 예외를 추가하는 경우라면 기껏해야 catch문 하나 추가하는것이 끝이다
    * 하지만 검사 예외가 단 하나뿐이라면 오직 그 예외때문에 API사용자는 try블록을 추가해야하고 스트림에서 직접 사용하지 못하게 된다. 그러니 이런 상황이라면 검사 예외를 던지지 않는 방법을 찾아보는게 좋다

- 검사 예외를 회피하는 가장 쉬운 방법은 적절한 결과 타입을 담은 옵셔널을 반환하는것이다. 검사 예외를 던지는 대신 빈 옵셔널을 반환하면 된다
    * 이 방식의 단점이라면 예외가 발생한 이유를 알려주는 부가 정보를 담을 수 없다는 것이다
    * 반면 예외를 사용하면 구체적인 예외 타입과 그 타입이 제공하는 메서드들을 활용해 부가 정보를 제공할 수 있다

- 또다른 방법으로 검사 예외를 던지는 메서드를 2개로 쪼개 비검사 예외로 바꿀 수 있다. 다음의 예시를 한번 보자

```java
try {
    obj.action(args);
} catch (TheCheckedException e) {
    ...
}


if(obj.actionPermitted(args)) {
    obj.action(args);
} else {
    ...
}
```

- 이 리팩터링을 모든 상황에 적용할 수는 없다. 그래도 적용 할 수만 있다면, 더 쓰기 편한 API를 제공 할 수 있다. 리팩터링 후의 API가 딱히 더 아름답지는 않지만 더 유연한것은 사실이며, 프로그래머가 이 메서드가 성공하리란걸 안다거나, 스레드가 실패시 중단하길 원한다면 다음처럼 한줄로 작성해도 무방하다

```java
obj.action(args);
```

- 이 한줄짜리 호출 방식이 주로 쓰일 거로 판단되면 리팩터링 하는 편이 바람직하다. 한편 actionPermitted는 상태 검사 메서드에 해다하므로 아이템 69에서 설명한 단점도 그대로 적용되니 주의해야한다
    * 즉, 외부 동기화 없이 여러 스레드가 동시에 접근할 수 있거나 외부 요인에 의해 상태가 변할 수 있다면 이 리팩터링은 적절하지 않다
    * actionPermitted와 action호출 사이에 객체의 상태가 변할 수 있기 때문이다
    * 또한, actionPermitted가 action메서드의 일부 작업을 중복 수행한다면 성능에서 손해이니, 이 리팩터링이 적절하지 않을 수 있다