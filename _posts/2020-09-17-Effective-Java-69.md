---
layout: post
title: "Effective Java - 아이템69: 예외는 진짜 예외 상황에만 사용하라"
description: 예외는 진짜 예외 상황에만 사용하라
date: 2020-09-17 21:30:00 +09:00
categories: EffectiveJava Study
---


# 예외

## 아이템 69 : 예외는 진짜 예외 상황에만 사용하라

- 운이 없다면 언젠가 다음과 같은 코드와 마주칠지도 모른다

```java
try { 
    int i = 0;
    while(true)
        range[i++].climb();
} catch(ArrayIndexOutOfBoundsException e) {
}
```

- 무슨일을 하는 코드인지 알겠는가? 전혀 직관적이지 않다는 사실 하나만으로도 코드를 이렇게 작성하면 안되는 이유는 충분하다
    * 이 코드는 배열의 원소를 순해하는데, 아주 끔찍한 방법으로 하고있다. 무한루프를 돌다가 배열의 끝에 도달해 ArrayIndexOutOfBoundsException이 발생하면 끝을 내는것이다

- 이 코드를 다음과 같이 표준적인 관용구대로 작성했다면 모든 자바 프로그래머가 곧바로 이해했을것이다

```java
for(Mountain m : range)
    m.climb();
```

- 그런데, 예외를 써서 루프를 종료한 이유는 무엇일까? 잘못된 추론을 근거로 해보면 성능을 높이려 하였던 것일수 있다. JVM은 배열에 접근할때마다 경계를 넘지 않는지 검사하는데, 이는 배열의 경계에도 마찬가지이므로, 위의 반복을 통하면 마지막 반복 하나를 안할수있다
- 하지만 이는 3가지면에서 잘못된 추론이다
    * 예외는 예외 상황에 쓸 용도로 설계되었으므로, JVM 구현자 입장에서는 명확한 검사만큼이나 빠르게 만들어야 할 동기가 약하다
    * 코드를 try-catch 블록안에 넣으면 JVM이 적용할 수 있는 최적화가 제안된다
    * 배열을 순회하는 표준 관용구는 앞서 걱정한 중복 검사를 수행하지 않는다. JVM이 알아서 최적화해 없애준다

- 이런 이야기의 교훈은 간단하다. 예외는 그 이름이 말해주듯 오직 예외 상황에서만 써야한다. 절대로 일상적인 제어 흐름용으로 쓰여선 안된다
- 더 일반화해 이야기하면 표준적이고 쉽게 이해되는 관용구를 사용하고, 성능 개선을 목적으로 과하게 머리를 쓴 기법은 자제하라
- 이 원칙은 API설계에도 적용된다. 잘 설계된 API라면 클라이언트가 정상적인 흐름에서 예외를 사용할 일이 없게 해야한다
- 특정 상태에서만 호출 할 수 있는 '상태 의존적' 메서드를 제공하는 클래스는 '상태 검사' 메서드도 함께 제공해야 한다
    * Iterator 인터페이스의 next와 hasNext가 각각 상태 의존적 메서드와 상태 검사 메서드에 해당한다

- 상태 검사메서드를 이용하면 다음과 같은 for관용구를 사용할 수 있다(for-each도 내부적으로 hasNext를 사용한다)

```java
for(Iterator<Foo> i = collection.iterator(); i.hasNext;) {
    Foo foo = i.next();
}
```

- 상태 검사 메서드 대신 사용할 수 있는 선택지도 있다. 올바르지 않은 상태일때 빈 옵셔널 혹은 null같은 특수한 값을 반환하는 방법이다
- 상태 검사 메서드, 옵셔널, 특정 값중 하나를 선택하는 지침은 다음과 같다
    * 외부 동기화 없이 여러 스레드가 동시에 접근할 수 있거나, 외부 요인으로 상태가 변할 수 있다면 옵셔널이나 특정 값을 사용한다. 상태검사메서드와 상태 의존적 메서드 호출 사이에 객체의 상태가 변할 수 있기 때문이다
    * 성능이 중요한 상황에서 상태 검사 메서드가 상태 의존적 메서드의 작업 일부를 중복 수행한다면 옵셔널이나 특정 값을 선택한다
    * 다른 모든 경우엔 상태 검사 메서드 방식이 조금 더 낫다고 할 수 있다. 가독성이 살짝 더 좋고, 잘못 사용 했을때 발견 하기가 쉽다. 상태 검사 메서드 호출을 깜빡 잊었다면 상태 의존적 메서드가 예외를 던져 버그를 확실히 드러낼것이다