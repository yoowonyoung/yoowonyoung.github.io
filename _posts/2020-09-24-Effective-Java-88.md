---
layout: post
title: "Effective Java - 아이템88: readObject는 방어적으로 작성하라"
description: readObject는 방어적으로 작성하라
date: 2020-09-24 21:50:00 +09:00
categories: EffectiveJava Study
---


# 직렬화

## 아이템 88 : readObject는 방어적으로 작성하라

```java
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if(this.start.compareTo(this.end) > 0)
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
    }

    public Date start() { return new Date(start.getTime());}
    public Date end() { return new Date(end.getTime());}
    public String toString() { return start + "-" + end};
}
```

- 위와같이 방어적 복사를 하는 클래스를 직렬화 하기로 했다고 해보자. Period 객체의 물리적 표현이 논리적 표현과 부합하므로 기본 직렬화를 사용해도 나쁘지 않다. 따라서 implements Serializable을 추가하는것으로 모든일을 끝낼수 있을것으로 보인다. 하지만 그렇게하면 이 클래스의 불변식이 보장되지 않는다
- readObject 메서드가 실질적으로는 또 다른 public 생성자 이기 때문이다. 다른 생성자와 똑같은 수준으로 주의를 기울여야 하기 때문에, readObject에서도 인수가 유효한지 검사해야하고, 필요하다면 매개변수를 방어적으로 복사해야 한다
- 쉽게말해 readObject는 매개변수로 바이트스트림을 받는 생성자라 할 수 있기 때문에, 불변식을 깨트릴 용도로 임의로 생성한 바이트스트림이 건네지면 문제가 발생한다
- 이러한 문제를 고치려면 Period의 readObject 메서드가 defaultReadObject를 호출한 다음 역직렬화된 객체가 유효한지 검사해야한다. 이 유효성 검사에 실패하면 InvalidObjectException을 던지게 하여 잘못된 역직렬화를 막을 수 있다

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    if(start.compareTo(end) > 0)
        throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
}
```

- 위의 작업으로 허용되지 않는 Period 인스턴스가 생성되는건 막았지만 아직 미묘한 문제 하나가 숨어있다. 정상 Period 인스턴스에서 시작된 바이트 스트림 끝에 private Date 필드로의 참조를 추가하면 가변 Period 인스턴스가 만들어지는것이다
- 이러한 문제의 근원은 Period의 readObject가 방어적 복사를 충분히 하지 않은데에 있다. 객체를 역직렬화 할때는 클라이언트가 소유해서는 안되는 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사해야 한다. 따라서 readObject에서 불변 클래스 안의 모든 private 가변 요소를 방어적으로 복사해야한다

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    start = new Date(start.getTime());
    end = new Date(end.getTime());

    if(start.compareTo(end) > 0)
        throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
}
```

- 방어적 복사를 유효성 검사보다 앞서 수행하며, Date의 clone은 사용하지 않았음에 주목해야한다. 두 조치 모두 Period를 공격으로부터 방어하는데 필요하다. 또한 final필드는 방어적 복사가 불가능 하니 주의해야한다
    * 따라서 이 readObject 메서드를 사용하려면 start와 end필드에서 final한정자를 제거해야한다

- 기본 readObject 메서드를 써도 좋을지 판단하는 간단한 방법이 있다. transient 필드를 제외한 모든 필드의 값을 배개변수로 받아 유효성 검사없이 필드에 대입하는 public 생성자를 추가해도 괜찮은지를 보는것이다
    * 답이 아니오라면 커스텀 readObject 메서드를 만들어 모든 유효성 검사와 방어적 복사를 수행해야 한다
    * 혹은 직렬화 프록시 패턴을 사용하는방법도 있다

- final이 아닌 직렬화 기능 클래스라면 readObject와 생성자의 공통점이 하나 더 있는데, 마치 생성자처럼 readObject 메서드도 재정의 가능 메서드를 호출해서는 안된다는것이다