---
layout: post
title: "Effective Java - 아이템5: 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라"
description: 자원을 직접 명시하지 말고 의존 객체 주입
date: 2020-07-27 21:49:00 +09:00
categories: EffectiveJava Study
---


# 객체의 생성과 파괴

## 아이템 5 : 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

- 많은 클래스가 하나 이상의 자원에 의존한다, 가령 맞춤법 검사기는 사전에 의존하는데, 이런 클래스를 정적 유틸리티 클래스로 구현한 모습을 드물지 않게 볼 수 있다. 또 싱글턴으로 구현하는 경우도 흔하다

```java
// 정적 유틸리티 클래스
public class SpellChecker {
    private static final Lexicon dictionary = ...;
    private SpellChecker() {} // 객체 생성 방지
    public static boolean isValid(Strig word) { ... }
    public static List<String> suggestions(String typo) { ... }
}

// 싱글턴
public class SpellChecker {
    private static final Lexicon dictionary = ...;
    private SpellChecker() { ... }
    public static SpellChecker INSTANCE = new SpellChecker(...);
    public static boolean isValid(Strig word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```

- 두 방식 모두 사전을 단 하나만 사용한다고 가정하는 점에서 그리 훌륭해 보이지 않을 뿐더러, 테스트 하기도 어렵다
- 또한 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다
- 클래스가 여러 자원 인스턴스를 지원해야 하며, 클라이언트가 원하는 자원을 사용해야 한다
- 이러한 조건을 만족하는 간단한 패턴이, 인스턴스를 생성할 떄 생성자에 필요한 자원을 넘겨주는 방식이다
- 이는 의존 객체 주입의 한 형태로, 맞춤법 검사기를 생성할 때 의존객체인 사전을 주입 해주면 된다

```java
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = dictionary;
    }

    public static boolean isValid(Strig word) { ... }
    public static List<String> suggestions(String typo) { ... }    
}
```

- 의존 객체 주입 패턴은 아주 단순하여 수많은 프로그래머가 이 방식에 이름이 있다는 사실도 모른채 사용한다
- 의존 객체 주입 패턴은 자원이 몇개든 의존 관계가 어떻든 상관없이 잘 작동 하며, 불변을 보장하며, 여러 클라이언트가 의존 객체들을 안심하고 공유 할 수 있다
- 의존 객체 주입은 생성자, 정적 팩터리, 빌더에 모두 똑같이 적용 할 수 있다
- 이 패턴의 쓸만한 변형으로, 생성자에 자원 팩터리를 넘겨주는 방식이 있다
- 의존 객체 주입이 유연성과 테스트 용이성을 개선 해주긴 하지만, 의존성이 수천개나 되는 큰 프로젝트에서는 코드를 어지럽게 만든다
    * Dagger, Guice, Spring과 같은 의존 객체 주입 프레임워크를 사용하여 이 어질러짐을 해결 할 수있다