---
layout: post
title: "Effective Java - 아이템50: 적시에 방어적 복사본을 만들어라"
description: 적시에 방어적 복사본을 만들어라
date: 2020-09-05 01:12:00 +09:00
categories: EffectiveJava Study
---


# 메서드

## 아이템 50 : 적시에 방어적 복사본을 만들어라

- 자바는 안전한 언어이다. 이것이 자바를 쓰는 즐거움중 하나이며, 네이티브 메서드를 사용하지 않으니 C,C++ 같이 안전하지 않은 언어에서 흔히 보는 버퍼 오버런, 배열 오버런, 와일드 포인터 같은 메모리 충돌 오류에서 안전하다
- 자바로 작성한 클래스는 시스템의 다른부분에서 무슨짓을 하든 그 불변식이 지켜진다. 메모리 전체를 하나의 거대한 배열로 다루는 언어에서는 누릴수 없는 장점이다
- 하지만 아무리 자바라 하더라도 다른 클래스의 침범으로부터 아무런 노력없이 다 막을수 있는것은 아니다. 그러니 클라이언트가 불변식을 깨트리려 혈안이 되어있다고 가정하고, 방어적으로 프로그래밍 해야 한다
- 어떤 객체든 그 객체의 허락 없이는 외부에서 내부를 수정하는 일은 불가능 하다. 하지만 주의를 기울이지 않으면 자기도 모르개 내부를 수정할 수 있도록 허락하는 경우가 생긴다. 흔히 발생하는 문제이다

```java
public final class Period {
    private final Date start;
    private final Date end;

    /**
    * @param start 시작 시각
    * @param end 종료시각; 시작 시간보다 뒤여야 한다
    * @throws IllegalArgumentException 시작시간이 종료시간보다 늦을떄 발생한다
    * @throws NullPointerException start나 end가 null이면 발생한다
    */

    public Period(Date start, Date end) {
        if(start.compareTo(end) > 0) 
            throw new IllegalArgumentException(start + "가 " + end + " 보다 늦다");
        this.start = start;
        this.end = end;
    }

    public Date start() {
        return start;
    }

    public Date end() {
        return end;
    }

    //생략
}
```

- 기간을 표현하는 위 클래스는 한번 값이 정해지면 변하지 않도록 할 생각이였다. 얼핏보면 이 클래스는 불변처럼 보이고 시작 시간이 종료 시간보다 늦을수 없나는 불편식이 무리없이 지켜질것 같다. 하지만 Date가 가변이라는 사실을 이용하면 어렵지 않게 그 불변식을 깨트릴 수 있다

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78);//p의 내부가 수정됨
```

- 다행이 자바8 이후로는 이 문제는 쉽게 해결할수 있다. Date 대신 불변인 Instant를 사용하면 된다(LocalDateTime, ZondedDateTime도 괜찮다)
    * Date는 낡은 API이니 새로운 코드를 작성할 때는 더이상 사용하면 안된다
    * 하지만 앞으로 쓰지 않는다고 해서 이런 문제에서 해방되는것은 아니다. Date 처럼 가변인 낡은 값 타입을 사용하던 시절이 워낙 길었던 탓에, 여전히 많은 API와 내부 구현 안에 그 잔재가 남아있다. 이번 아이템은 예전에 작성된 낡은 코드들을 대처하기 위함이다

- 외부 공격으로부터 Period 인스턴스의 내부를 보호하려면 생성자에서 받은 가변 매개변수 각각을 방어적으로 복사(defensive copy)해야한다. 그런다음 Period 인스턴스 안에서는 원본이 아닌 복사본을 사용하면 된다

```java
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    if(this.start.compareTo(this.end) > 0) 
            throw new IllegalArgumentException(this.start + "가 " + this.end + " 보다 늦다");
    //이하 동일
}
```

- 새로 작성한 생성자를 사용하면 앞서 공격은 더이상 Period에 위협이 되지 않는다
- 매개변수의 유효성을 검사하기 전에, 방어적 복사본을 만들고, 이 복사본으로 유효성을 검색한점에 주목하자. 순서가 부자연스러워 보일수 있으나, 반드시 이렇게 해야만 한다
- 멀티 스레딩 환경이라면 원본 객체의 유효성을 검사한 후 복사본을 만드는 그 찰나의 순간에 다른 스레드가 원본 객체를 변경할 수 있기 때문이다. 방어적 복사를 매개변수 검사 이전에 한다면 이런 위험에서해방될 수 있다
    * 이는 검사시점(time-of-check)/사용시점(time-of-use) 공격 혹은 영문 표기를 줄여 TOCTOU라고 한다

- 방어적 복사에 Date의 clone 메서드를 사용하지 않은점에도 주목해야 한다. Date는 final이 아니므로, clone이 Date가 정의한게 아닐 수 있기 때문이다. 즉 clone이 악의를 가진 하위 클래스의 인스턴스를 반환 할 수도 있다
- 매개변수가 제 3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone을 사용해서는 안된다
- 생성자를 수정하면 앞서의 공격은 막아낼 수 있지만 Period 인스턴스는 아직도 변경 가능하다. 접근자 메서드가 내부의 가변 정보를 직접 드러내기 때문이다

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start,end);
p.end().setYear(78);//p의 내부가 변경되었다!
```

- 두번째 공격을 막아내려면, 단순히 접근자가 가변 필드의 방어적 복사본을 반환하면 된다

```java
public Date start() {
    return new Date(start.getTime());
}

public Date end() {
    return new Date(end.getTime());
}
```

- 새로운 접근자까지 갖추면 Period는 완벽한불변으로 거듭난다. 아무릐 악의적인 혹은 부주의한 프로그래머라도 시작 시간이 종료 시간보다 나중일 수 없다는 불변식을 위배할 방법은 없다
- Period 자신 말고는 가변 필드접근할 방법이 없으니 확실하다. 모든 필드가 객체 안에 완벽하게 캡슐화 되어있다
- 생성자와 달리 접근자 메서드에서는 방어적 복사에 clone을 사용해도 된다. Period가 가지고 있는 Date객체는 java.util.Date임이 확실하기 때문이다. 그래도 인스턴스를 복사하는데에는 일반적으로 생성자나 정적 팩터리 메서드를 쓰는게 좋다
- 매개변수를 방어적으로 복사하는 목적이 불변 객체를 만들기 위해서만은 아니다
- 메서드든 생성자든 클라이언트가 제공한 객체의 참조를 내부의 자료구조에 보관해야한다면, 항시 그 객체가 잠재적으로 변경될 수 있는지를 생각 해보아야 한다. 변경 될 수 있는 객체랄면 그 객체가 클래스에 넘겨진 뒤 임의로 변경되어도 그 클래스가 문제없이 동작할지를 따져봐야 한다. 확신할 수 없다면 복사본을 만들어 저장해야 한다
    * 예컨대 클라이언트가 건네준 객체를 내부의 Set 인스턴스에 저장하거나 Map 인스턴스의 키로 사용딘다면, 추후 그 객체가 변경될경우 객체를 담고 있는 Set 혹은 Map의 불변식이 깨질것이다

- 내부 객체를 클라이언트에 건네주기전에 방어적 복사본을 만드는 이유도 마찬가지 이다. 여러분의 클래스가 불변이든 가변이든, 가변인 내부 객체를 클라이언트에 반환할 때에는 심사숙고 해야한다. 안심할 수 없다면 원본을 노출하지 말고 방어적 복사본을 반환해야한다
- 길이가 1이상인 배열은 무조건 가변임을 잊지 말아야 한다. 그러니 배무에서 사용하는 배열을 클라이언트에 반환할 때는 항상 방어적 복사를 수행해야 한다. 혹은 배열의 불변 뷰를 반환하는 방법도 있다
- 이상의 모든 작업에서 우리는 "되도록 불변 객체들을 조합해 객체를 구성해야 방어적 복사를 할 일이 줄어든다" 라는 교훈을 얻을 수 있다
    * Period의 경우 자바8 이상이라면 Instant(혹은 LocalDateTime이나 ZoneDateTime)를 사용하라

- 방어적 복사에는 성능 저하가 따르고, 또 항상 쓸 수 있는것도 아니다(같은 패키지에 속하는 등의 이유로). 호출자가 컴포넌트 내부를 수정하지 않으리라 확신하면 방어적 복사를 생략할수 있다. 이러한 상황이라도 호출자에서 해당 매개변수가 반환값을 수정하지 말아야함을 명확히 문서화 하는게 좋다
- 다른 패키지에서 사용한다고 해서 넘겨받은 가변 매개변수를 항상 방어적으로 복사해서 저장해야 하는것은 아니다
- 때로는 메서드나 생성자의 매개변수로 넘기는 행위가 그 객체의 통제권을 완전히 이전함을 뜻한다. 이처럼 통제권을 이전하는 메서드를 호출하는 클라이언트는 해당 객체를 더이상 직접 수정하는 일이 없다고 약속해야 한다. 이런것도 역시 문서화 해둬야 한다
- 통제권을 넘겨 받기로 한 메서드나 생성자를 가진 클래스들은 악의적인 클라이언트 공격에 취약하다. 따라서 방어적 복사를 생략해도 되는 상황은 해당 클래스와 그 클라이언트가 상호 신뢰할 수 있을때, 혹은 불변식이 깨지더라도 그 영향이 오직 호출한 클라이언트로 국한될 때로 한정해야 한다
    * 후자의 예가 래퍼 클래스 패턴이다. 래퍼 클래스의 특성상 클라이언트는 래퍼에 넘긴 객체에 여전히 접근 할 수 있다. 따라서 래퍼의 불변식은 쉽게 파괴되지만, 영향은 오직 그 클라이언트 자신만이 받는다

