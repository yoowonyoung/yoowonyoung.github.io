---
layout: post
title: "Effective Java - 아이템11: equals를 재정의 하려거든 hashCode도 재정의 하라"
description: equals를 재정의 하려거든 hashCode도 재정의 하라
date: 2020-08-08 17:27:00 +09:00
categories: EffectiveJava Study
---


# 모든 객체의 공통 메서드

## 아이템 11 : equals를 재정의 하려거든 hashCode도 재정의 하라

- equals를 재정의한 클래스 모두에서 hashCode도 재정의 해야 한다
- 그렇지 않으면 hashCode일반 규약도 어기게 되어 해당 클래스의 인스턴스를 HahsMap이나 HashSet의 원소로 사용할 떄 문제를 일으킬 것이다
- 다음은 Object명세의 일부이다
    * equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇번을 호출해도 일관되게 항상 같은 값을 반환해야 한다
    * equals가 두 객체를 같다고 판단 했다면, 두 객체의 hashCode는 똑같은 값을 반환 해야 한다
    * equals가 두 객체를 다르다고 판단 했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없지만 이는 해시 테이블의 성능에 영향이 있다

- hashCode 재정의를 잘못 했을때 크게 문제가 되는 조항은 두번쨰 이다. 즉 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다
- 앞선 아이템10의 PhonNumber 클래스의 인스턴스를 HashMap의 원소로 사용할때 다음과 같은 문제가 발생한다

```java
Map<PhonNumber, Strimg> m = new HashMap<>();
m.put(new PhonNumber(707,867,5309), "제니");
m.get(new PhonNumber(707,867,5309); // NULL반환
```

- PhonNumber 클래스는 equals를 재정의 했지만, hashCode는 재정의 하지 않았으므로 논리적 동치인 두 객체가 서로 다른 해시코드를 반환하여 두번째 규약이 어겨지게 되었다
- 이 문제는 hashCode를 적절하게 작성 해 주면 되는데, 절대 다음과 같은 hashCode는 사용하면 안된다

```java
@Override
public int hashCode() {
    return 42;
}
```

- 위 hashCode는 적법하긴 하지만, 모든 객체에 대해서 똑같은 해시코드를 반환하여 해시테이블의 성능이 극악으로 떨어지게 된다
- 좋은 해시함수라면 서로 다른 인스턴스에 대해 다른 해시코드를 반환해야 한다. 다음은 좋은 hashCode를 작성하는 간단한 요령이다
    * int 변수 result를 선언한 후 값 c로 초기화 한다. 이때 c는 해당 객체의 첫번째 핵심 필드(equals에 사용된)를 다음 단계의 방식으로 계산한 해시코드다
    * 해당 객체의 첫번째 핵심필드 c와 나머지 핵심 필드 f에 대해 다음 작업을 수행한다. c에 대해 먼저 수행한 후 f에 대해서 반복한다
        + 기본 타입 필드라면, [Type].hashCode(f) 를 수행한다
        + 참조 타입 필드이면서 이 클래스의 equals메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다.
        계산이 더 복잡해질것 같으면, 이 필드의 표준형을 만들어 그 표준형의 hashCode를 호출한다. 필드의 값이 null이면 0을 사용한다(이건 전통이다)
        + 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 배열의 핵심 원소가 하나도 없다면 단순히 상수(여기서도 0이 좋다)를 사용한다. 모든 원소가 핵심원소리면 Arrays.hashCode를 사용한다
        + 위에서 계산한 해시코드 c로 result를 result = 31 * result + c 로 갱신한다
    * result를 반환한다

- hashCode를 다 구현 했다면, 이 메서드가 동치인 인스턴스에 대해서 똑같은 해시코드를 반환할지 자문 해보고, 이 역시 단위 테스트를 작성하자
- 파생필드는 해시코드 계산에서 제외해도 된다. 즉 다른 필드로 부터 계산해낼 수 있는 필드는 모두 무시 되어야 한다
- equals 비교에 사용되지 않은 필드는 반드시 제외 되어야 hashCode 규약 2번째를 어기게 된다
- 이를 이용해 PhonNumber에 재정의 하면 다음과 같다

```java
@Override
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```

- 이렇게 구현된 해시함수는 충분히 쓸만하지만, 해시 충돌이 더욱 적은 방법을 써야하면 구아바(Guava)의 com.google.common.hash.HashString을 참고하자
- Objects 클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메서드인 hash를 제공 한다
- 이 메서드를 활용하면 앞서 요령대로 구현한 코드와 비슷한 수준의 hashCode 함수를 단 한줄로 작성 할 수 있지만, 속도는 조금 느려진다

```java
@Override
public int hashCode() {
    return Objects.hash(lineNum,prefix,areaCode);
}
```

- 클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보다는 캐싱하는 방식을 고려해야 한다
- 이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어 질 떄 미리 해시코드를 계산 해둬야 한다
- 해시의 키로 사용되지 않는다면 hashCode가 처음 불릴 때 계산하는 지연 초기화 전략도 좋지만, 스레드 안정성을 신경써야 한다

```java
private int hashCode;

@Override
public int hashCode() {
    int result = hashCode;
    if(result == 0) {
        result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
    }
    return result;
}
```

- 성능을 높인답시고 해시코드를 계산 할 떄 핵심 필드를 생략해서는 안된다. 속도야 빨라질 수 있지만, 해시품질이 나빠져 해시 테이블의 성능이 떨어진다
- hashCode가 반환하는 값의 생성 규칙을 API사용자에게 자세히 공표하지 말아야, 클라이언트가 이 값에 의지하지 않게되고, 추후에 계산 방식을 바꿀 수 있다
- equals를 재정의 할 떄에는 hashCode도 반드시 재정의 해야 한다. 그렇지 않으면 프로그램이 제대로 동작 하지 않을것이다
- 재정의한 hashCode는 Objetc의 API문서에 기술된 일반 규약을 따라야 한다
