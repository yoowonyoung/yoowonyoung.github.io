---
layout: post
title: "Effective Java - 아이템13: clone 재정의는 주의해서 진행해라"
description: clone 재정의는 주의해서 진행해라
date: 2020-08-08 20:00:00 +09:00
categories: EffectiveJava Study
---


# 모든 객체의 공통 메서드

## 아이템 13 : clone 재정의는 주의해서 진행해라

- Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스 이지만, 아쉽게도 그 목적을 제대로 이루지 못햇다
- 가장 큰 문제는 clone메서드가 선언된 곳이 Cloneable이 아닌 Object이고, 그마저도 protected라는데 있다
- 그래서 Cloneable을 구현 하는것만으로는, 외부 객체에서 clone 메서드를 호출 할 수 없다
- 리플렉션을 이용하면 가능하지만 100% 성공 한다는 보장이 없고, 해단 객체가 접근이 허용된 clone 메서드를 제공한다는 보장이 없기 떄문이다
- 그래도 Cloneable 방식은 널리 쓰이고 있어서 잘 알아두는것이 좋고, 이번 아이템에서는 clone메서드를 잘 동작하게끔 구현하는 방법과 언제 그렇게 해야할지, 가능한 다른선택지를 알아보자
- 메서드 하나 없는 Cloneable 인터페이스는 Object의 protected 메서드인 clone의 동작 방식을 결정한다
- Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException을 던진다
- 이는 상당히 이례적으로 사용한 예이니 따라하면 안된다
- 명세에서는 이야기하지 않지만, 실무에서 Cloneable을 구현한 클래스는 clone 메서드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다
- 이 기대를 만족시키려면 그 클래스와 모든 상위클래스는 복잡하고, 강제할수도 없고, 허술하게 기술된 프로토콜을 지켜야 하는데, 그 결과로 꺠지기 쉽고, 위험하고, 모순적인 메커니즘이 탄생한다. 
생성자를 호출하지 않고도 객체를 생성하는것이다
- clone메서드의 일반 규약은 허술하다. Object에서 가져온 일반규약은 다음과 같다

```java
이 객체의 복사본을 생성해 반환한다. '복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다. 일반적인 의도는 다음과 같다. 
어떤 객체 x에 대해 다음 식은 참이다

x.clone() != x

또한 다음 식도 참이다

x.clone().getClass() == x.getClass()

하지만 이상의 요구를 반드시 만족해야 하는것은 아니다.
한면 다음 식도 일반적으로 참이지만, 역시 필수는 아니다

x.clone().equals(x)

관례상, 이 메서드가 반환하는 객체는 super.clone을 호출해 얻어야 한다.
이 클래스와(Object를 제외한) 모든 상위 클래스가 이 관례를 따른다면 다음 식은 참이다.

x.clone().getClass() == x.getClass()

관례상, 반환된 객체와 원본 객체는 독립적이어야한다. 이를 만족하려면 super.clone로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다
```

- 강제성이 없다는 점만 빼면 생성자 연쇄와 살짝 비슷한 매커니즘이다
- 즉 clone 메서드가 super.clone이 아닌, 생성자를 호출해 얻은 인스턴스를반환해도 컴파일러는 불평하지 않는다
- 하지만 이 클래스의 하위클래스에서 super.clone을 호출한다면 잘못된 클래스의 객체가 만들어져, 결국 하위 클래스의 clone메서드가 제대로 동작하지 않는다
- clone을 재정의한 클래스가 final이라면 걱정해야 할 하위클래스가 없으니 이 관례는 무시해도 안전하다
- 제대로 동작하는 clone 메서드를 가진 상위 클래스를 상속해 Cloneable을 구현하고 싶다고 해보자. 먼저 super.clone을 호출한다. 이렇게 얻은 객체는 원본의 완벽한 복제본으로, 
모든 필드가 기본타입이거나 불변이라면 더이상 손볼게 없다

```java
@Override
public PhonNumber clone() {
    try {
        return (PhonNumber) super.clone();
    } catch (CloneNotSupportException e) {
        throw new AssertionError(); //일어날수 없는일이다
    }
}
```

- 이 메서드가 동작하려면 PhonNumber의 클래스 선언에 Cloneable을 구현한다고 추가해야 한다
- Object의 clone 메서드는 Object를 반환하지만 PhonNumber의 clone 메서드는 PhonNumber를 반환하게 했다. 자바가 공변 반환 타이핑을 지원하니 이렇게 하는것이 가능하고, 이를 권장하기도 한다
- super.clone 호출을 try-catch블록으 감싼 이유는 Object의 clone 메서드가 검사 예외인 CloneNotSupportedException을 던지도록 선언되었기 떄문이다
    * 우리는 super.clone이 성공할것임을 안다. 이 거추장스러운 예외는 사실 CloneNotSupportedException이 비검사 예외였다는 신호다

- 이런 간단한 구현이 클래스가 가변객체를 참조하는 순간 재앙으로 변한다
- 앞서 아이템7에서 소개한 Stack 클래스를 떠올려 보면, clone 메서드가 단순히 super.clone의 인스턴스를 그대로 반환할 경우, Stack의 size는 올바른 값을 갖겟지만, 
elements필드는 원본 Stack인스턴스와 똑같은 배열을 참조할것이다. 원본이나 복제본 둘중 한곳을 수정하면 다른 하나도 수정되어버리는것이다
- Stack의 하나뿐인 생성자를 호출한다면 이러한 상황은 절대 일어나지 않는다
- clone메서드는 사실상 생성자와 같은 효과를 낸다. 즉, clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다
- 다음은 올바르게 만들어진 Stack의 clone이다

```java
@Override
public Stack clone() {
    try{
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch (CloneNotSupportException e) {
        throw new AssertionError(); 
    }
}
```

- elements.clone의 결과를 Object[]로 형변환 할 필요는 없다. 배열의 clone은 런타임 타입과 컴파일러타입 모두가 원본과 똑같은 배열을 반환한다. clone이 제대로 사용되는것이다
- 한편 element가 final이면 위 방식은 정상동작 하지 않는데, final에는 새로운 값을 할당할 수 없기 때문이다
- 직렬화와 마찬가지로 Cloneable 아키텍처는 '가변 객체를 참조로 하는 필드는 final로 선언하라' 라는 일반 용법과 충돌한다
- clone을 재귀적으로 호출하는 것만으로는 충분하지 않을떄가 있다. 해시테이블용 clone을 만든다고 생각해보자

```java
public class HashTable implements Clonable {
    private Entry[] buckets = ...;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }
}
```

- Stack에서처럼 단순히 버킷 배열의 clone을 재귀적으로 호출하면, 복제본은 자신만의 버킷 배열을 가지지만, 이 배열은 원본과 같은 연결리스트를 참조하기 떄문에 동작에 문제가 생길 수 있다
- 이를 해결하기 위해 각 버킷을 구성하는 연결리스트를 복사해야 한다

```java
public class HashTable implements Clonable {
    private Entry[] buckets = ...;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }

    Entry deepCopy() {
        return new Entry(key,value,next == null ? null : next.deepCopy());
    }
}

@Override
public HashTable clone() {
    try{
        HashTable result = (HashTable) super.clone();
        result.buckets = new Entry[buckets.length];
        for(int i = 0; i < buckets.lentgh; i ++) {
            if(buckets[i] != null) 
                result.buckets[i] = buckets[i].deepCopy();
        }
        return result;
    } catch (CloneNotSupportException e) {
        throw new AssertionError(); 
    }
}
```

- private 클래스인 HashTable.Entry는 깊은 복사를 지원하도록 보강되었고, HashTable의 clone 메서드는 적절한 크기의 새로운 버킷 배열을 할당한 다음, 배열을 순회하며 비지않은 각 버킷에 대해 깊은복사를 한다
- 이러한 방법은 연결리스트 복제로는 좋지 않은데, 재귀 호출 때문에 리스트의 원소 수만큼 스택 프레임을 소비하여 오버플로의 가능성이 있기 떄문이다. 이 문제를 해결 하려면 deepCopy를 재귀호출 대신 반복자를 써서 순회 해야한다

```java
Entry deepCopy() {
    Entry result = new Entry(key,value,next);
    for(Enrty p = result; p.next != null; p = p.next) 
        p.next = new Entry(p.next.key, p.next.value, p.nxet.next);
    return result;
}
```

- 이제 복잡한 가변 객체를 복제하는 마지막 방법이다
- 먼저 super.clone을 호출하여 얻은 객체의 모든 필드를 초기상태로 설정한 다음, 원본 객체의 상태를 다시 생성하는 고수준 메서드들을 호출한다
- HashTable의 예에서는, buckets 필드를 새로운 버킷 배열로 초기화 한 다음, 원본 테이블에 담긴 모든 키-값 쌍 각각에 대해 put을 통해 둘의 내용을 같게 만들어 주면 된다
- 이처럼 고수준 API를 사용하면 제범 우아한 코드를 얻게 되지만, 저수준에서 바로 처리하는것 보다는 느려지게 된다. 또한 Cloneable 아키텍처의기초가 되는 필드 단위 객체복사를 우회하기 떄문에 어울리지 않는다
- 생성자에서는 재정의 될 수 있는 메서드를 호출하지 말아야 하는데, clone도 마찬가지 이다
- 만약 clone이 하위 클래스에서 재정의한 메서드를 호출하게 되면, 하위 클래스는 복제 과정에서 자신의 상태를 교정할 기회를 잃어 원본과 복제본의 상태가 달라질 수 있다
- Object 의 clone 메서드는 CloneNotSupportException을 던진다고 선언 했지만, 재정의 메서드는 그렇지 않다
- public 인 clone 메서드에서는 thorws절을 없애야, 그 메서드를 사용하기 편하기 떄문이다
- 상속해서 쓰기 위한 클래스 설계 방식 2가지(아이템 19)에서는 어느쪽에서든 Cloneable을 구현하면 안된다
- Object의 방식을 모방해 제대로 작동하는 clone메서드를 구현해 protected로 두고, CloneNotSupportException도 던질수 있다고 선언하는것이다
- 이 방식은 마치 Object를 바로 상속할때처럼 Cloneable구현 여부를 하위 클래스에서 선택 하도록 해준다
- 다른 방법으로 clone을 동작하지 않게 구현하고 하위 클래스에서 재정의 하지 못하게 둘수도 있다. 다음과 같이 퇴화 시키면 되는것이다

```java
@Override
protected final Object clone() throws CloneNotSupportException {
    throw new CloneNotSupportException();
}
```

- 기억해야 할 것이 하나 더 있는데, Cloneable을 구현한 스레드 안전 클래스를 작성 할 떄에는, clone역시 적절히 동기화 해줘야 한다는점이다
- 요약하자면 Cloneable을 구현하는 모든 클래스들은 clone을 재정의 해야하며, 이떄 접근 제한자는 public으로, 반환 타입은 자기 자신으로 변경한다
- 이 메서드는 가장 먼저 super.clone을 호출 한 후 필요한 필드를 전부 적절히 수정한다. 객체 내부의 깊은 구조에 숨어있는 모든 가변 객체를 복사하고, 복제본이 가진 객체 참조를 모두 복사한 객체들을 가리키게 하는것이다
- 이러한 내부 구조는 주로 clone을 재귀적으로 호출해 구현하지만, 이 방식이 항상 최선인것은 아니다
- Cloneable을 이미 구현한 클래스를 확장한다면, 어쩔수 없이 clone이 잘 작동하도록 구현해야 한다
- 그렇지 않은 상황에서는 복사 생성자와 복사 팩터리라는 더 나은 객체 복사 방식을 제공 할 수 있다
- 복사 생성자와 복사 팩터리메서드는 언어모순적이며 위험천만한 객체 생성 매커니즘을 쓰지도 않고, 엉성하게 문서화된 규약에 기대지도 않고, 정상적인 final용법과도 충돌하지 않으며, 불필요한 검사예외나 형변환도 필요없다
- 뿐만 아니라 복사 생성자와 복사 팩터리메서드는 해당 클래스가 구현한 모든 인터페이스 타입의 인스턴스를 인수로 받을 수 있다