---
layout: post
title: "Effective Java - 아이템40: @Override 어노테이션을 일관되게 사용하라"
description: @Override 어노테이션을 일관되게 사용하라
date: 2020-08-30 19:18:00 +09:00
categories: EffectiveJava Study
---


# 열거타입과 어노테이션

## 아이템 40 : @Override 어노테이션을 일관되게 사용하라

- 자바가 기본적으로 제공하는 어노테이션중 보통의 프로그래머에게 가장 중요한것은 ```@Override``` 일것이다
- ```@Override```는 메서드 선언에만 달 수 있으며, 이 어노테이션이 달렸다는것은 상위 타입의 메서드를 재정의 했음을 뜻한다
- 이 어노테이션을 일관되게 사용하면 여러가지 악명높은 버그들을 예방해준다

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }

    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for(int i = 0; i < 10; i++) {
            for(char ch = 'a'; ch <= 'z'; ch++) {
                s.add(new Bigram(ch,ch));
            }
        }
        System.out.println(s.size());
    }
}
```

- 위 코드는 소문자 2개로 구성된 바이그램 26개를 10번 반복해 집합에 넣는다. Set은 중복을 허용하지 않으므로 26이 출력될것 같지만 실제론 260이 출력된다
- 그 이유는 equals가 재정의가 아니라 다중정의(overloading)되어버렸기 때문이다. Object의 equals를 재정의 하려면 매개변수 타입을 Object로 해야 하기 때문이다
    * Object의 equals는 ==와 똑같이 객체의 식별성만 확인하기 떄문에, 같은 소문자를 가지는 바이그램 10개가 각각이 서로 다른 객체로 인식되고 결국 260이 출력되는것이다

- 다행이 이러한 오류는 컴파일러에서 잡을 수 있지만, 그러려면 다음 코드와 같이 Object의 equals를 재정의한다는 의도를 명확히 명시해야한다 

```java
@Override
public boolean equals(Bigram b) {
    ...
}
```

- 그러니 상위 클래스의 메서드를 재정의하려는 모든 메서드에 ```@Override``` 어노테이션을 달아야 한다
- 단 한가지 예외가 있는데, 구체클래스에서 상위 클래스의 추상 메서드를 재정의 할떄에는 굳이 ```@Override```를 달지 않아도된다
    * 구체클래스인데 아직 구현하지 않은 추상메서드가 남아있다면, 컴파일러가 그 사실을 바로 알려주기 때문이다

- 한편 IDE는 ```@Override```를 일관되게 사용하도록 부추겨준다. 관련 옵션을 켜두면 실수로 재정의 했을때 경고해 줄것이다
- ```@Override```는 클래스 뿐만 아니라 인터페이스의 메서드를 재정의 할떄에도 유용하게 사용할 수 있는데, 디폴트 메서드를 지원하기 시작하면서 인터페이스 메서드를 구현한 메서드에도 ```@Override```를 다는 습관을 들이면 시그니처가 올바른지 재차 확인할 수 있다
- 추상클래스나 인터페이스에서는 상위클래스나 상위 인터페이스의 메서드를 재정의 하는 모든 메서드에서는 ```@Override```를 다는것이 좋은데, 이는 상위 클래스가 구체 클래스이던 추상 클래스이던 마찬가지 이다. 이는 실수로 추가한 메서드가 없음을 보장한다