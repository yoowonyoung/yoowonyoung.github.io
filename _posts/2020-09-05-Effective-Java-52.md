---
layout: post
title: "Effective Java - 아이템52: 다중정의는 신중히 사용하라"
description: 다중정의는 신중히 사용하라
date: 2020-09-05 23:12:00 +09:00
categories: EffectiveJava Study
---


# 메서드

## 아이템 52 : 다중정의는 신중히 사용하라

- 다음은 컬렉션을 집합, 리스트, 그외로 구분하고자 만든 프로그램이다

```java
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "집합";
    }

    public static String classify(List<?> lst) {
        return "리스트";
    }

    public static String classify(Collection<?> c) {
        return "그외";
    }

    public static void main(String[] args) {
        Collection<?> collections = {
            new HashSet<String>(),
            new ArrayList<Integer>(),
            new HashMap<String, String>().values();
        };

        for(Collection<?> c : collections)
            System.out.println(classify(c));
    }
}
```

- 이 코드를 실제로 수행해보면 "그외" 만 계속 출력될것이다. 그 이유는 다중정의된 세 classify중 어느 메서드를 호출할지가 컴파일 타임에 정해지기 때문이다
- 컴파일 타임에 for문 안의 c는 항상 Collection<?> 타입이다. 런타임에는 매번 타입이 달라지지만 호출할 메서드에 영향을 주지는 못한다. 따라서 컴파일 타임의 매개변수 타입을 기준으로 항상 세번째 classify만 불리게 된다
- 이처럼 직관과 어긋나는 이유는 재정의한 메서드는 동적으로 선택되고, 다중정의한 메서드는 정적으로 선택되기 때문이다
- 메서드를 재정의 했다면 해당 객체의 런타임 타입이 어떤 메서드를 호출할지의 기준이 된다. 메서드를 재정의 한 다음 하위클래스의 인스턴스에서 그 메서드를 호출하면 재정의한 메서드가 실행된다. 컴파일 타임에 그 인스턴스의 타입이 무었이였는지는 상관이 없다

```java
Class Wine {
    String name() { return "포도주"; }
}

Class SparklingWine extends Wine {
    @Override
    String name() { return "발포성 와인"; }
}

Class Champange extends SparklingWine {
    @Override
    String name() { return "샴페인"; }
}

public class Overriding {
    public static void main(String args[]) {
        List<Wine> wineList = List.of(
            new Wine(), new SparklingWine(), new Champange());

        for(Wine wine : wineList) {
            System.out.printn(wine.name());
        }
    }
}
```

- Wine 클래스에 정의된 name메서드는 하위 클래스인 SparklingWine와 Champange에서 재정의 된다. 그리고 예상한것처럼 이 프로그램은 차례로 포도주, 발포성 와인, 샴페인을 출력한다. for문에서 컴파일 타임의 타입이 모두 Wine인것에 무관하게 항상 가장 하위에서 정의한 재정의 메서가 실행되는것이다
- 다중정의된 메서드 사이에서 객체의 런타임 타입은 전혀 중요하지 않다. 선택은 오직 컴파일 타임에, 오직 매개변수의 컴파일 타임 타입에 의해 이뤄진다
- 위의 CollectionClassifier예에서 프로그램의 원래 의도대로 동작하게 만들려면, 모든 classify메서드를 하나로 합친후 instanceof로 명시적으로 검사하면 말끔히 해결된다

```java
public static String classify(Collection<?> c) {
    return c instanceof Set ? "집합" : c instanceof List ? "리스트" : "그외";
}
```

- 프로그래머에게는 재정의가 가장 정상적인 동작방식이고, 다중정의가 예외적인 동작방식으로 보일것이다. 이처럼 헷갈릴수 있는, 다중정의가 혼동을 일으키는 코드는 작성하지 않는게 좋다
- 정확히 어떻게 사용했을때 다중정의가 혼란을 주느냐는 논란의 여지가 있다. 안전하고 보수적으로 가려면 매개변수 수가 같은 다중정의는 만들지 말아야 한다
- 가변인수를 사용하는 메서드라면 다중정의를 아예 하지 말아야 한다
- 이런 규칙만 잘 따르기만 하면 어떤 다중 정의 메서드가 호출될지 헷갈일 일은 없을것이다. 다중정의하는 대신 메서드 이름을 다르게 지어주는 길도 항상 열려있으니 말이다
- 이번에는 ObjectOutputStream 클래스를 살펴보자. 이 클래스의 write메서드는 모두 기본타입과 일부 참조 타입용 변형을 가지고 있다. 그런데 다중정의가 아닌, 모든 메서드에 다른 이름을 지어주는 길을 선택했다(writeBooleam, writeInt, writeLong)
- 이 방식이 다중정의보다 나은 또다른점은, read메서드의 이름과 짝을 맞추기 좋다는것이다(readBoolean, readInt, readLong). 실제로 ObjectInputStream 클래스의 read메서드는 이렇게 되어있다
- 한편, 생성자는 이름을 다르게 지을수 없으니 두번째 생성자부터 무조건 다중정의가 된다. 하지만 정적 팩터리라는 대안을 활요할 수 있는 경우가 많다. 또한 생성자는 재정의 될 수 없으니 다중정의와 재정의가 혼용될 걱정은 하지 않아도 된다
- 매개변수 수가 같은 다중정의 메서드가 많더라도, 그중 어느것이 주어진 매개변수 집합을 처리할지가 명확히 구분된다면 헷갈일 일은 없을것이다
- 즉, 매개변수중 하나 이상이 근본적으로 다르다면 헷갈일 일이 없다. 근본적으로 다르다는건 두 타입의 null이 아닌 값이 서로 어느쪽으로든 형변활 될 수 없다는 뜻이다. 이 조건만 충족한다면 어떤 다중정의 메서드를 호출할지가 매개변수들의 런타임 타입만으로 정해진다
- 따라서 컴파일타임 타입에는 영향을 받지 않게 되고, 혼란을 주는 주된 원인이 사라진다. 예컨대 ArrayList에는 int를 받는 생성자와 Collection을 받는 생성자가 있는데, 어떤 상황에서든 두 생성자중 어느것이 호출될지 헷갈일 일은 없을것이다
- 자바4 까지는 모든 타입이 모든 참조 타입과 근본적으로 달랐지만, 자바5에서 오토박싱이 도입되면서 평화롭던 시대가 막을 내렷다

```java
public class SetList {
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();
        List<Integer> list = new ArrayList<>();

        for(int i = -3; i < 3; i ++) {
            set.add(i);
            list.add(i);
        }

        for(int i = 0; i < 3; i ++) {
            set.remove(i);
            list.remove(i);
        }

        System.out.println(set + " " + list);
    }
}
```

- 이 프로그램은 -3부터 2까지 정수를 정렬된 집합과 리스트에 각각 추가한다음 remove을 3번 호출한다. 그러면 이 프로그램은 음이 아닌 값을 제거하고 출력 할것이라 예상되지만, 실제로는 집합에서는 음이 아닌 값이 제거되지만, 리스트에서는 홀수가 제거된다
- set.remove(i)의 시그니처는 remove(object)이다. 다중정의된 다른 메서드가 없으니 기대한대로 동작하여 집합에서 0 이상의 수들을 제거한다
- list.remove(i)는 다중정의된 remove(int index)를 선택한다. 이 remove는 지정한 위치의 원소를 제거하는 기능을 수행한다. 그래서 0,1,2번째 원소가 제거 되는것이다
- 이 문제는 list.remove인수를 Integer로 형변환 하여 올바른 다중저으이 메서드를 선택하게 하면 된다. 혹은 Integer.valueOf를 사용해 i를 Integer로 변환하여 list.remove로 전달해도 된다
- 이 예가 혼란스러웠던 이유는 List<E> 인터페이스가 remove(Object)와 remove(int) 를 다중정의했기 때문이다. 제너릭이 도입되기전인 자바4까지의 List에서는 Object와 int가 근본적으로 달라서 문제가 없다. 그런데 제너릭과 오토박싱이 등장하면서 두 메서드의 매개변수 타입이 더는 근본적으로 다르지 않게 되었다
- 자바 언어에 제너릭과 오토박싱을 더한 결과 List인터페이스가 취약해졌다. 다행이 같은 피해를 입은 API는 거의 없지만 다중정의시 주의를 기울여야할 근거로는 충분하다

```java
new Thread(System.out::println).start();
ExcutorService exec = new Excutors.newCachedThreadPool();
exec.submit(System.out::println);
```

- 자바8에서 도입된 람다와 메서드 참조 역시 다중정의시의 혼란을 키웠다. 위의 코드는 모습이 비슷하지만 두번째 부분만 컴파일 오류가 난다. 넘겨진 인수는 모두 System.out::println으로 똑같고 양쪽 모두 Runnable을 받는 형제 메서드를 다중정의하고있는데도 말이다!
- 원인은 submit 다중정의 메서드 중에는Callable<T>를 받는 메서드도 있다는데에 있다. 하지만 모든 println이 void를 반환한, 반환값이 있는 Collable과 헷갈릴 일은 없다고 생각할수도 있지만, 다중정의 메서드를 찾는 알고리즘은 이렇게 동작하지 않는다
- 만약 println이 다중정의 없이 단 하나만 존재했다면 이 submit 메서드 호출은 제대로 컴파일 됫을것이다. 지금은 참조된 메서드(println)과 호출한 메서드(submit) 모두 다중정의 되어있어, 다중정의 해소 알고리즘이 우리의 기대처럼 동작하지 않기 때문이다
- 기술적으로 말하면 System.out::println은 부정확한 메서드 참조이다. 또한 암시적 람다타입이나 부정확한 메서드 참조같은 인수 표현식은 목표 타입이 선택되기 전까지는 그 의미가 정해지지 않기 때문에 적용성 테스트때 무시된다. 이것이 이 문제의 원인이다
    * 사실 이 설명은 컴파일러 제작자를 위한 설명이니 무시해도 된다

- 핵심은 다중정의된 메서드들이 함수형 인터페이스를 인수로 받을때, 비록 서로 다른 함수형 인터페이스라 하더라도 인수 위치가 같으면 혼란이 생긴다는것이다. 따라서 메서드를 다중정의할 때, 서로 다른 함수명 인터페이스라도 같은 위치의 인수로 받아서는 안된다
- Object외의 클래스 타입과 배열 타입은 근본적으로 다르다. Serializable과 Cloneable외의 인터페이스 타입과 배열 타입도 근본적으로 다르다. 한편 String과 Throwable처럼 상위/하위 관계가 아닌 두 클래스는 관련이 없다고 한다
- 이 외에도 어떤 방향으로도 형변환할 수 없는 타입쌍이 있지만, 어쨋든 이것이 복잡해지면 대부분의 프로그래머는 어떤 다중정의 메서드가 선택될지를 구분하기 어려워 할것이다
- 다중정의된 메서드중 하나를 선택하는 규칙은 매우 복잡하며, 자바가 버전업될수록 더 복잡해지고 있어, 이 모두를 이해하고 사용하는 프로그래머는 극히 드물것이다
- 이번 아이템에서 설명한 지침들을 어기고 싶을때도 있을것이다. 이미 만들어진 클래스가 끼어들면 특히 더 그렇다
    * String의 경우 contetEquals를 자바4 시절부터 가지고 있었다
    * 자바5가 되면서 StringBuffer, StringBuilder, String, CharBuffer등 비슷한 부류의 타입을 묶기위한 CharSequence가 등장하였고 String에도 CahrSequence를 받는 contentEquals가 다중정의 되어있다
    * 그 결과 이번 아이템의 지침을 대놓고 어기는 모습이 되었지만, 이 두 메서드는 같은 객체를 입력하면 완전히 같은 작업을 수행해주니 해로울것은 전혀 없다. 이처럼 어떤 다중정의 메서드가 불리는지는 몰라도 기능이 똑같다면 전혀 신경쓸 필요가 없다. 이렇게 하는 가장 일반적인 방법은 더 특수한 다중정의 메서드에서 덜 특수한 다중정이 메서드로 포워드 하는것이다

```java
public boolean contentEquals(StringBuffer sb) {
    return contentEquals((CharSequence) sb);
}
```

- 자바 라이브러리는 이번 아이템의 정신을 지켜내려 애쓰고 있지만, 실패한 클래스도 있다
    * String의 valueOf(char[])와 valueOf(Object)는 같은 객체를 건네더라도 전혀 다른일을 수행한다. 이것이 잘못된 사례이다

- 프로그래밍언어가 다중정의를 허용한다고 해서 다중정의를 꼭 활용하란뜻은 아니다. 일반적으로 매개변수 수가 같을떄에는 다중정의를 피하는게 좋다
- 생성자라면 이 조언을 따르기가 불가능할 수도 있지만, 그럴때는 헷갈릴만한 매개변수는 형변환하여 정확한 다중정의 메서드가 선택되도록 해야한다. 이것이 불가능하다면 다중정의된 메서드들이 모두 동일하게 동작하도록 만들어야 한다