---
layout: post
title: "Effective Java - 아이템49: 매개변수가 유효한지 검사해라"
description: 매개변수가 유효한지 검사해라
date: 2020-09-04 23:36:00 +09:00
categories: EffectiveJava Study
---


# 메서드

## 아이템 49 : 매개변수가 유효한지 검사해라

- 메서드와 생성자 대부분은 입력 매개변수의 특정 조건을 만족하기를 바란다
    * 예컨대 인덱스 값은 음수이면 안되며, 객체 참조는 null이 아니여야 하는식이다

- 이런 제약은 반드시 문서화 해야하며, 메서드 몸체가 시작되기전에 검사해야 한다. 이는 "오류는 가능한 빨리 (발생한 곳에서) 잡아야 한다" 라는 일반 원칙의 한 사례이기도 한다. 오류를 잘생한 즉시 잡지못한다면 해당 오류를 감지하기 어려워지고, 감지하더라도 오류의 발생 지점을 찾기 어려워 진다
- 메서드 몸체가 실행되기 전에 매개변수를 확인한다면, 잘못된 값이 넘어왓을때 즉각적이고 깔끔한 방식으로 예외를 던질 수 있다
- 매개변수 검사를 제대로 하지 못하면 몇가지 문제가 생길 수 있다
    * 메서드가 수행되는 중간에 모호한 예외를 던지며 실패 할 수 있다. 더 나쁜 상황은 메서드가 잘 수행되지만 잘못된 결과를 반환할 때 이다. 한층 더 나쁜 상황은 메서드는 문제없이 수행 되지만, 어떤 객체를 이상한 상태로 만들어놓아서 미래의 알수 없는 시점에 이 메서드와는 관련없는 오류를 낼 때 이다

- public과 protected 메서드는 매개변수 값이 잘못되었을때 던지는 예외는 문서화 해야한다(@throws 자바독 태그를 이용하면 된다). 보통은 IllegalArgumentException, IndexOutOfBoundsException, NullPointerException중 하나가 될것이다
- 매개변수의 제약을 문서화한다면 그 제약을 어겼을때 발생하는 예외도 함꼐 기술해야 한다. 이러한 간단한 방법으로 API 사용자가 제약을 지킬 가능성을 크게 높일 수 있다

```java
/**
* (현재값 mod m) 값을 반환한다. 이 메서드는 항상 음이아닌 BigInteger를 반환한다는 점에서 remainder메서드와는 다르다
* @param m 계수(양수여야 한다)
* @return 현재 값 mod m
* @throws ArthmeticException m이 0보다 작거나 같으면 발생한다
*/
public BigInteger mod(BigInteger m) {
    if (m.signum() <= 0) 
        throw new ArthmeticException("계수m은 양수여야 합니다. " + m);
}
```

- 이 메서드는 m이 null이면 m.signum()을 호출할 때 NullPointerException을 던진다. 그렇지만 "m이 null일때 NullPointerException을 던진다"라는 설명은 어디에도 없는데, 이는 이 설명을 개별 메서드가 아닌 BigInteger 클래스 수준에서 기술했기 때문이다. 클래스 수준 주석은 그 클래스의 모든 public메서드에서 적용되므로 각각의 메서드에 일일히 기술하는것 보다 훨씬 깔끔한 방법이다
- @Nullable이나 이와 비슷한 어노테이션을 사용해 특정 매개변수는 null이 될 수 있다고 알려줄 수 있지만, 표준적인 방법은 아니며 같은 목적으로 사용할 수 있는 어노테이션도 여러가지이다
- 자바7에서 추가된 java.util.Objects.requiredNonNull 메서드는 유연하고 사용하기도 편하니, 더이상 null검사를 수동으로 하지 않아도 된다. 원하는 예외 메시지도 지정할수 있는데, 입력을 그대로 반환하므로 값을 사용하는 동시에 null검사를 수행할 수 있다

```java
this.strategy = Objects.requiredNonNull(strategy,"전략");
```

- 반환값은 그냥 무시하고 필요한곳 어디든 순수한 null검사목적으로 사용해도 된다
- 자바9에서는 Objects에 범위 검사 기능도 더해졌다.checkFromIndexSize, checkFromToIndex, checkIndex라는 메서드 들인데, null검사 메서드 만큼 유연하지는 않다. 예외 메시지를 지정할 수 없고, 리스트와 배열 전용으로 설계되어 있다. 또한 닫힌 범위(양 끝단 값을 포함하는)는 다루지 못한다. 그래도 이러한 제약이 걸림돌이 되지 않는 상황에서는 아주 유용하다
- 공개되지 않은 메서드라면 패키지 제작자인 여러분이 메서드가 호출되는 상황을 통제 할 수 있다. 따라서 오직 유효한 값이 메서드에 넘겨지리라는것을 여러분이 보증 할 수 있고, 그렇게 해야한다
    * 즉, public이 아닌 메서드라면 assert를 사용해 매개변수의 유효성을 검증할 수 있다

```java
private static void sort(long[] a, int offset, int length) {
    assert a != null;
    assert offest >= 0 && offeset <= a.length;
    assert length >= 0 && length <= a.length - offset;
    //작업 수행
}
```

- 여기서 핵심은, 이 단언문들은 자신이 단언한 조건이 무조건 참이라고 선언하는것이다. 이 메서드가 포함된 패키지를 클라이언트가 어떻게 사용하던 상관이 없다
- 단언문은 몇가지면에서 일반적인 유효성 검사와 다르다
    * 실패하면 AssertionError를 던진다
    * 런타임에 아무런 효과도, 성능저하도 없다(단 java실행시 명렬줄에서 -ea, --enableassetions를 설정하면 런타임에 영향이 있다)

- 메서드가 직접 사용하지는 않으나 나중에 쓰기 위해 저장하는 매개변수는 특히 신경써서 더 검사해야 한다

```java
static List<Integer> intArrayAsList(int[] a) {
    Objects.requireNonNull(a);
    return new AbstractList<>() {
        @Override
        public Integer get(int i) {
            return a[i];
        }

        @Override
        public Integer set(int i, Integer val) {
            int oldVal = a[i];
            a[i] = val;
            return oldVal;
        }

        @Override
        public int size() {
            return a.length;
        }
    };
}
```

- 위 코드는 int 배열의 List뷰를 반환하는 메서드이다. 내부에서 Objects.requiredNonNull을 이용해 null검사를 수행하므로 클라이언트가 null을 건네면 NullPointerException을 던진다
    * 만약 이 검사를 생략하면 새로 생성한 List인스턴스를 반환하는데, 클라이언트가 돌려받은 List를 사용할 때 비로소 NullPointerException이 발생한다

- 생성자는 "나중에 쓰려고 저장하는 매개변수의 유효성을 검사하라"는 원칙의 특수 사례이다. 생성자 매개변수의 유효성검사는 클래스 불변식을 어기는 객체가 만들어지지 않게 하는데 꼭 필요하다
- 메서드 몸체 실행전에 매개변수 유효성을 검사해야 한다는 규칙에도 예외는 있다. 유효성 검사 비용이 지나치게 높거나 실용적이지 않을때, 혹은 계산과정에서 암묵적으로 검사가 수행될 때 이다
    * 예를들어 Collections.sort(List)처럼 객체 리스트를 정렬하는 메서드를 생각해보자. 리스트 안의 객체들은 모두 상호 비교될 수 있어야 하며, 정렬 과정이서 비교가 이뤄진다. 만약 상호 비교 될 수 없는 객체가 들어있다면, 그 객체와 비교할 때 ClassCaseException을 던질것이다

- 때로는 계산 과정에서 필요한 유효성 검사가 이뤄지지만 실패했을때 잘못된 예외을 던지기도 한다
    * 계산중 잘못된 매개변수값을 사용해 발생한 예외와 API문서에서 던지기로한 예외가 다를 수 있다는 뜻이다
    * 이러한 경우에는 예외 번역 관용구를 사용해 API문서에 기재된 예외로 번역해줘야 한다

- 이번 아이템을 "매개변수에 제약을 두는게 좋다" 라고 해석해서는 안된다. 사실은 그 반대이기 때문이다
- 메서드는 최대한 범용적으로 설계해야 한다. 메서드가 건네받은 값으로 무언가 제대로 된 일을 할 수 있다면, 매개변수 제약은 적을수록 좋다. 하지만 구현하려는 개념 자체가 특정한 제약을 내재한 경우도 드물지 않다
- 메서드나 생성자를 작성할 때면, 그 매개변수들에 어떤 제약이 있을지 생각해야 하며, 그 제약들을 문서화 하고 메서드 코드 시작부분에 명시적으로 검사해야 한다