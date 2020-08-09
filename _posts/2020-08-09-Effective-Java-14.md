---
layout: post
title: "Effective Java - 아이템14: Comparable을 구현할지 고려하라"
description: Comparable을 구현할지 고려하라
date: 2020-08-09 13:20:00 +09:00
categories: EffectiveJava Study
---


# 모든 객체의 공통 메서드

## 아이템 14 : Comparable을 구현할지 고려하라

- Comparable 인터페이스의 유일한 메서드인 compareTo는 Comparable은 Object의 메서드가 아니다
- compareTo는 Object의 equals와 비교하여 2가지가 다른데, compareTo는 단순 동치성비교에 더해 순서까지 비교 할 수 있으며, 제너릭하다
- 따라서 Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서가 있음을 뜻하며, Arrays.sort 등을 사용하여 쉽게 정렬이 가능하다
- 검색, 극단값 계산, 자동 정렬되는 컬렉션 관리도 역시 쉽게 할 수 있다
- 자바의 모든 값 클래스와 열거 타입이 Comparable을 구현하였다
- 알파벳, 숫자, 연대같이 순서가 명확한 클래스를 작성한다면 반드시 Comparable을 구현하자
- compareTo 메서드의 일반 규약은 equals의 규약과 비슷하다
    * 이 객체와 주어진 객체의순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환한다
    * 이 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastExcetpion을 던진다
    * 다음 설명에서 sgn(표현식) 표기는 수학에서 말하는 부호함수를 뜻하며, 표현식의 값이 음수,0,양수 일때 -1,0,1을 반환하도록 정의했다
    * Comparable을 구현한 클래스는 모든 x,y에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다(예외 역시동일하다)
    * Comparable을 구현한 클래스는 추이성을 보장해야 한다. 즉 x.compareTo(y) > 0 && y.compareTo(z) > 0 이면 x.compareTo(z) > 0 이다
    * Comparable을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0 이면 x.compareTo(z) == y.compareTo(z) 이다
    * 이번 권고는 필수는 아니지만 꼭 지키는게 좋다. (x.compareTo(y) == 0) == (x.equals(y)) 여야 한다

- 모든 객체에 대해 전여 동치관계를 부여하는 equals와는 달리, compareTo는 타입이 다른 객체를 신경쓰지 않아도 된다(ClassCaseException을 던지면 되기 때문이다)
- compareTo 규약을 지키지 못하면, TreeSet, TreeMap, Collections, Arrays등과 어울릴 수 없다
- 첫번째 규약은 두 객체의 참조 순서를 바꿔 비교해도 예상한 결과가 나와야 한다는 얘기이다
- 두번째 규약은 첫번째가 두번째보다 크고, 두번째가 세번쨰 보다 크다면, 첫번째가 세번째보다 커야 한다는 이야기다
- 마지막 규약은 크기가 같은 객체들끼리는 어떤 객체와 비교해도 항상 결과가 같아야한다는 것이다
- 이상 세 규약은 compareTo메서드로 수행하는 동치성 검사도 equals 규약과 똑같이 반사성, 대칭성, 추이성을 충족해야 함을 뜻한다. 그래서 주의사항도 똑같다. 우회방법도 똑같다
- 기촌 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가 했다면 compareTo 규약을 지킬 방법이 없다. 이를 해결 하기 위해서는 확장하는 대신 독립된 클래스를 만들고, 이 클래스 내부에 원래 클래스를 가르키는 필드를 두면된다
- compareTo의 마지막 규약은 필수는 아니지만 꼭 지키는것이 좋다. 간단히 말하면 compareTo 메서드로 수행한 동치성 테스트의 결과가 equals와 같아야 한다는것이다. 
이를 잘 지키면 compareTo로 줄지은 순서와 equals의 결과가 일치한다. 결과가 일관되지 않아도 동작은 잘 하지만, Collection, Set, Map등에 넣으면 정의된 동작과 엇박자가 될것이다.
정렬된 컬렉션들은 동치성을 비교할때 equals가 아닌 compareTo를 사용하기 떄문이다
- compareTo의 작성 요령도 역시 equals와 비슷한데, 몇가지 차이점만 주의하면 된다
- Comparable은 타입을 인수로 받는 제너릭 인터페이스 이므로, compareTo 메서드의 인수 타입은 컴파일 타입에 정해진다. 입력 인수의 타입을 확인하거나 형변환 할필요가 없다는것이다
- compareTo 메서드는 각 필드가 동치인지를 비교 하는게 아니라, 그 순서를 비교한다
- 객체 참조 필드를 비교 하려면 compareTo를 재귀적으로 호출한다
- Comparable을 구현하지 안흔ㄴ 필드나 표준이 아닌 순서로 비교 해야 한다면, 비교자를 대신 사용한다. 비교자는 직접 만들거나 자바가 제공하는것중 골라 쓰면 된다

```java
public final clas CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
    public int compareTo(CaseInsensitiveString cis){
        return String.CASE_INSENSITIVE_ORDER.compare(s,cis.s);
    }
}
```

- CaseInsensitiveString 이 Comparable<CaseInsensitiveString>을 구현한것에 주목해야 한다. CaseInsensitiveString의 참조는 CaseInsensitiveString 참조와만 비교할 수 있다는 뜻으로 일반적이다
- 클래스에 핵심 필드가 여러개라면 어느것을 먼저 비교하느냐가 중요해진다. 가장 핵심적인 필드부터 비교해야하며, 비교 결과가 0이 아니라면, 즉 순서가 결정되면 거기서 끝이다. 그 결과를 곧장 반환해야 한다
- 다음은 PhonNumber 클래스용 compareTo 메서드를 이 방식으로 구현한것이다

```java
public int compareTo(PhonNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode);
    if(result == 0) {
        result = Short.compare(prefix, pn.prefix);
        if( result == 0) {
            result = Short.compare(lineNum, pn.lienNum);
        }
    }
    return result;
}
```

- 자바8 에서는, Comparator 인터페이스가 일련의 비교자 생성 메서드와 팀을 꾸려 메서드 연쇄 방식으로 비교자를 생성 할 수 있게 되었다. 
그리고 이 비교자들을 Comparable 인터페이스가 원하는 compareTo 메서드를 구현하는데 사용 할 수 있다. 이 방법은 간결하지만 약간의 성능 저하가 뒤따른다
- 자바의 정적 임포트 기능을 이용하면, 정적 비교자 생성 메서드들을 그 이름만으로 사용 할 수 있어 코드가 훨씬 깔끔해진다

```java
private static final Comparator<PhonNumber> COMPARATOR =
    comparingInt((PhonNumber pn) -> pn.areaCode)
        .thenComparingInt(pn -> pn.prefix)
        .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhonNumber pn) {
    return COMPARATOR.compare(this,pn);
}
```

- 위 코드는 클래스를 초기화 할 때 비교자 생성 메서드 2개를 이용해 비교자를 생성한다
- 첫번째인 comparingInt은 객체 참조를 int 타입 키에 매핑하는 키 추출 함수를 인수로 받아, 그 키를 기준으로 순서를 정하는 비교자를 반환하는 정적 메서드이다
- 위의 예제에서 comparintgInt는 람다를 인수로 받으며, 이 람다는 PhonNumber에서 추출한 지역코드를 기준으로 전화번호의 순서를 정하는 Comparator<PhonNumber>를 반환한다
- 지역코드가 같을떄의 비교는 thenComparingInt가 수행한다. thenComparingInt는 원하는 만큼 연달아 호출 할 수있다
- Comparator는 수많은 보조 생성 메서드들이 있는데, long과 double 용으로 comparintInt와 thenComparingInt의 변형이 있으며, short은 그냥 int용을 쓰면 되고 float는 double용을 쓰면 된다
- 객체 참조용 비교자 생성 메서드도 준비되어 있는데, 우선 comparing이라는 정적 메서드 2개가 다중 정의 되어있다. 
첫번째는 키 추출자를 받아서 그 키의 자연적 순서를 쓰고, 두번째는 키 추출자 하나와 추출된 키를 비교할 비교자까지 2개를 키로 받는다
- thenComparing도 3개가 다중 정의 되어있는데, 첫번째는 비교자 하나만 인수로 받아서 부차 순서를 정하고, 두번째는 키 추출자를 그 키의 자연적 순서로 보조 순서를 정한다. 
마지막 세번째는 키 추출자 하나와 추출된 키를 비교할 비교자까지 2개를 인수로 받는다
- 이따금 '값의 차'를 기준으로 첫번째 값이 두번째 값보다 작으면 음수를, 두 값이 같으면 0을, 첫번째 값이 크면 양수를 반환하는 compareTo나 compare메서드가 있다

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode();
    }
};
```

- 이러한 방식은 사용하면 안된다. 정수 오버플로나 부동소수점 계산 방식에 따른 오류가 생길 수 있다. 따라서 이러한 방식은 다은 두 방식중 하나로 변경 하는게 좋다

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
};

static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```
