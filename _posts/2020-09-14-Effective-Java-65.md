---
layout: post
title: "Effective Java - 아이템65: 리플렉션 보다는 인터페이스를 사용하라"
description: 리플렉션 보다는 인터페이스를 사용하라
date: 2020-09-14 21:56:00 +09:00
categories: EffectiveJava Study
---


# 일반적인 프로그래밍 원칙

## 아이템 65 : 리플렉션 보다는 인터페이스를 사용하라

- 리플렉션 기능을 사용하면 프로그램에서 임의의 클래스에 접근 할 수 있다. Class 객체가 주어지면 그 클래스의 생성자, 메서드, 필드에 해당하는 Constructor, Method, Field 인스턴스를 가져올 수 있고, 이어서 이 인스턴스들로는 그 클래스의 멤버 이름, 필드타입, 메서드 시그니쳐등을 가져 올 수 있다
- 나아가 Constructor, Method, Field 인스턴스를 이용해 각각에 연결된 실제 생성자, 메서드 필드를 조작 할 수도 있다. 이 인스턴스들을 통해 해당 클래스의 인스턴스를 생성하거나, 메서드를 호출하거나, 필드에 접근할 수 있다는 뜻이다
    * 예를들어 Method.invoke는 어떤 클래스의 어떤 객체가 가진 어떤 메서드라도 호출 해줄수 있게 해준다
    * 리플렉션을 이용하면 컴파일 당시에 존재하지 않던 클래스도 이용할 수 있다

- 물론 리플렉션도 단점이 있다
    * 컴파일 타임 타입 검사가 주는 이점을 하나도 누릴 수 없다. 예외검사도 마찬가지이다. 프로그램이 리플렉션 기능을 써서 존재하지 않는 혹은 접근할수 없는 메서드 호출을 시도하려 한다면 런타임 오류가 발생한다
    * 리플렉션을 이용하면 코드가 지저분해지고 장황해진다
    * 성능이 떨어진다. 리플렉션을 이용한 메서드 호출은 일반 메서드 호출보다 훨씬 느리다. 고려해야하는 요소가 많아 정확한 차이는 이야기 하기 어렵지만 느리다

- 코드 분석 도구나, 의존관계 주입 프레임워크처럼 리플렉션을 써야 하는 복잡한 애플리케이션들도 있지만, 이 애플리케이션들 마저도 리플렉션 사용을 점차 줄이고 있다. 단점이 명료하기 때문이다
- 리플렉션은 아주 제한된 형태로만 사용해야 그 단점을 피하고 이점만 취할 수 있다
- 컴파일 타임에 이용할수 없는 클래스를 사용해야만 하는 프로그램은, 비록 컴파일 타임이라도 적절한 인터페이스나 상위 클래스를 이용할 수는 있을 것이다
- 이런 경우 리플렉션은 인스턴스 생성에만 쓰고, 이렇게 만든 인스턴스는 인터페이스나 상위 클래스로 참조해서 쓰자

```java
public static void main(String[] args) {
    Class<? extends Set<String>> cl = null;
    try {
        cl = (Class<? extends Set<String>>)Class.forName(args[0]);
    } catch(ClassNotFoundException e) {
        fatalError("Class Not found");
    }

    Constructor<? extends Set<String>> cons = null;
    try{
        cons = cl.getDeclaredConstructor();
    } catch(NoSuchMethodException e) {
        fatalError("No args Constructor Not found");
    }

    Set<String> s = null;
    try {
        s = cons.newInstance();
    } catch(IllagalAccessException e) {
        fatalError("Constructor not found");
    } catch(InstantiationException e) {
        fatalError("Instantiation Exception");
    } catch(InvocationTargetException e) {
        fatalError("Constructor Throw exception");
    } catch(ClassCastException e) {
        fatalError("Not a set class");
    }

    s.addAll(Arrays.asList(args).subList(1, args.length));
    System.out.println(s);
}

private static void fatalError(String msg) {
    System.err.println(msg);
    System.exit(1);
}
```

- 위 프로그램은 Set<String> 인터페이스의 인스턴스를 생성하는데, 정확한 클래스는 명령줄의 첫번째 인수로 확정한다. 그리고 생성한 Set에 두번째 이후의 인수들을 추가한다음 출력한다
- 첫번쨰 인수와 상관없이 두번째 인수 이후에서의 중복은 제거된다
- 반면 이 인수들이 출력되는 순서는 첫번째 인수로 지정한 클래스가 무엇이냐에 따라 달라지는데, HashSet은 무작위 일것이고, TreeSet은 알파벳 순일것이다
- 이 예는 리플렉션의 단점 2가지를 보여준다
    * 런타임에 총 여섯가지나 되는 예외를 던질 수 있다. 모두 인스턴스를 리플렉션 없이 생성했다면 컴파일 타임에 잡아낼 수 있을것이다
    * 클래스 이름만으로 인스턴스를 생성해내기 위해 무려 25줄이나 되는 코드를 사용했다는점이다. 리플렉션이 아니였더라면 생성자 호출 단 한줄이면 끝났을것이다
    * 리플렉션 예외를 각각 잡는 대신에 모든 리플렉션 예외의 상위클래스인 RefletiveOperationException을 잡도록 하여 코드 길이를 줄일수도 있다

- 리플렉션의 단점은 모두 생성하는부분에만 국한되기 때문에, 객체가 일단 만들어지면 그 이후의 코드는 여타 Set 인스턴스를 사용할때와 똑같다
- 이 프로그램은 컴파일 하며 비검사 형변환 경고가 뜨는데, ```Class<? extends Set<String>>``` 으로의 형변환은 명시한 클래스가 Set을 구현하지 않았더라도 성공하기 때문에 실제 문제로 이어지지는 않는다. 단지 인스턴스를 생성하려고 할때 ClassCastException이 뜰 뿐이다
- 드물긴 하지만, 리플렉션은 런타임에 존재하지 않을수도 있는 다른 클래스, 메서드, 필드와의 의존성을 관리할때 유용하다. 가동할 수 있는 최소한의 환경, 즉 주로 가장 오래된 저번만을 지원하도록 컴파일 한 후, 이후 버전의 클래스와 메서드 등은 리플렉션으로 접근하도록 하는 방식이다
    * 이렇게 하려면 접근하려는 새로운 클래스나 메서드가 런타임에 존재하지 않을 수 있다는 사실을 반드시 감안해야 한다