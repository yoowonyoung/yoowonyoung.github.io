---
layout: post
title: "Effective Java - 아이템89: 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용해라"
description: 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용해라
date: 2020-09-24 22:09:00 +09:00
categories: EffectiveJava Study
---


# 직렬화

## 아이템 89 : 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용해라

- readResolve기능을 이용하면 readObject가 만들어낸 인스턴스를 다른것으로 대체 할 수 있다. 역직렬화한 객체의 클래스가 readResolve 메서드를 적절히 정의 해뒀다면, 역직렬화 후 새로 생성된 객체를 인수로 이 메서드가 호출되고, 이 메서드가 반환한 객체 참조가 새로 생성된 객체를 대신해 반환된다

```java
public class Elvis implements Serializable {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }

    public void leaveTheBuilding() { ... }

    private Object readResolve() {
        return INSTANCE;
    }
}
```

- 싱글턴 클래스들은 implements Serializable 을 추가하는 순간 더이상 싱글턴이 아니게 된다. 어떤 readObject를 사용하던 이 클래스가 초기화될 때 만들어진 인스턴스와는 별개인 인스턴스를 반환하게 된다
- 위와 같이 readResolve 메서드를 추가해 싱글턴 속성을 유지할 수 있다. 이 메서드는 역직렬화 한 객체를 무시하고 클래스 초기화때 만들어진 Elvis인스턴스를 반환한다. 따라서 Elvis 인스턴스의 직렬화 형태는 아무런 실 데이터를 가질 이유가 없으니, 모든 인스턴스 필드를 transient로 선언해야 한다
    * 사실 readResolve를 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴스 필드는 모두 transient로 선언해야 한다

- 싱글턴이 transient가 아닌 참조 필드를 가지고 있다면, 그 필드의 내용은 readResolve 메서드가 실행되기전에 역직렬화 된다. 그렇다면 잘 조작된 스트림을 써서 해당 참조 필드의 내용이 역직렬화 되는 시점에 그 역직렬화된 인스턴스의 참조를 훔쳐올 수 있다
- 더 자세히 알아보자. 먼저 readResolve 메서드와 인스턴스 필드 하나를 포함한 도둑 클래스를 작성한다. 이 인스턴스 필드는 도둑이 숨길 직렬화된 싱글턴을 참조하는 역할을 하며, 직렬화된 스트림에서 싱글턴의 비 휘발성 필드를 이 도둑의 인스턴스로 교체한다. 그렇게 되면 싱글턴은 조직을 참조하고 도둑은 싱글턴을 참조하는 순환 고리가 만들어진다
- 싱글턴이 도둑을 포함하므로 싱글턴이 역직렬화 될 때 도둑의 readResolve 메서드가 먼저 호출된다. 그 결과, 도둑의 readResolve 메서드가 수행될때 도둑의 인스턴스 필드에는 역직렬화 도중인 싱글턴의 참조가 들어가게 된다
- 도둑의 readResolve 메서드는 이 인스턴스 필드가 참조한 값을 정적 필드로 복사하여 readResolve가 끝난 후에도 참조 할수 있도록 한다. 그런 다음 이 메서드는 도둑이 숨긴 transient가 아닌 필드의 원래 타입에 맞는 값을 반환한다

```java
public class Elvis implements Serializable {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {}

    private String[] fatoriteSongs = { "Hound Dog" , "Heartbreak Hotel" };
    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }

    private Object readResolve() {
        return INSTANCE;
    }
}


public class ElvisStealer implements Serializable {
    static Elvis impersonator;
    private Elvis payload;

    private Object readReslove() {
        impersonator = payload;

        return new String[] { "A Fool Such as I" };
    }

    private static final long serialVerisonUID = 0;
}
```

- 위의 코드들은 잘못된 싱글턴의 예시와 그를 훔치는 도둑 클래스이다. 그리고 다음이 직렬화의 허점을 이용해 다른 싱글턴을 만드는 방법이다

```java
public class ElvisImpersonator {
    private static final byte[] serializedFrom = { .... };

    public static void main(String[] args) {
        Elvis elvis = (Elvis) deserialize(serializedFrom);
        Elvis impersonator = ElvisStealer.impersonator;
        
        elvis.printFavorites();
        impersonator.printFavorites();
    }
}

실행 결과
[Hound Dogs, HeartBreak Hotel]
[A Fool Such as I]
```

- 위 코드를 통해 서로 다른 2개의 Elvis 인스턴스가 생성됨을 확인 할 수 있다
- favoriteSongs 필드를 transient로 선언하여 이 문제를 고칠 수 있지만, Elvis를 원소 하나짜리 열거타입으로 만드는것이 더 안전한 선택이다
- ElvisStealer 공격으로 보여줫듰이 readResolve 메서드를 사용해 순간적으로 만들어진 역직렬화된 인스턴스에 접근하지 못하게 하는 방법은 깨지기 쉽고 신경을 많이써야 하는 작업이다
- 직렬화 가능한 인스턴스 통제 클래스를 열거 타입을 이용해 구현하면 선언한 상수외의 다른 객체는 존재하지 않음을 자바가 보장해준다
    * 물론 공격자가 AccessibleObject.setAccessible같은 특권 메서드를 악용한다면 이야기가 달라진다
    * 임의의 네이티브 코드를 수행할 수 있는 특권을 가로챈 공격자에게는 모든 방어가 무력화 된다

```java
public enum Elvis {
    INSTANCE;
    private String[] fatoriteSongs = { "Hound Dog" , "Heartbreak Hotel" };
    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
}
```

- 위와 같이 열거타입의 싱글턴으로 만들면 된다
- 인스턴스 통제를 위해 readResolve를 사용하는 방식이 완전히 쓸모 없는것은 아니다. 직렬화 가능 인스턴스 통제 클래스를 작성해야 하는데, 컴파일 타임에는 어떤 클래스들이 있는지 알 수 없는 상황이라면 열거 타입으로 표현하는것이 불가능 하기 때문이다
- readResolve 메서드의 접근성은 매우 중요하다. final클래스에서라면 readResolve 메서드는 private 여야 한다. final이 아닌 클래스라면 다음의 몇가지를 고려해야 한다
    * private로 선언하면 하위 클래스에서 사용할 수없다
    * package-private로 선언하면 같은 패키지에 속한 하위 클래스에서만 사용 할 수 있다
    * protected나 public으로 선언하면 이를재정의 하지 않은 모든 하위 클래스에서 사용할 수 있다
    * protected나 public이면사 하위 클래스에서 재정의 하지 않았다면, 하위 클래스의 인스턴스를 역직렬화 하면서 상위 클래스의 인스턴스를 생성하여 ClassCastException이 발생 할 수 있다
