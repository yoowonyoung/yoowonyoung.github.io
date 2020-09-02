---
layout: post
title: "Effective Java - 아이템45: 스트림은 주의해서 사용하라"
description: 스트림은 주의해서 사용하라
date: 2020-09-02 23:05:00 +09:00
categories: EffectiveJava Study
---


# 람다와 스트림

## 아이템 45 : 스트림은 주의해서 사용하라

- 스트림 API는 다량의 데이터 처리 작업을 돕고자 자바8에 추가되었다
- 이 API가 제공하는 추상개념중 핵심은 두가지 이다
    * 스트림은 데이터 원소의 유한 또는 무한 시퀀스를 뜻한다
    * 스트림 파이프라인은 이 원소들로 수행할 수 있는 연산 단계를 표현하는 개념이다

- 스트림의 원소들은 어디로부터든 올 수 있다. 대표적으로는 컬렉션, 배열, 파일, 정규표현식 패턴매처, 난수생성기, 혹은 다른 스트림이 그 예시이다
- 스트림 안의 데이터 원소들은 객체 참조나 기본 타입값이며, 기본 타입값으로는 int, long, double을 지원한다
- 스트림 파이프라인은 소스 스트림에서 시작해 종단 연산으로 끝나며, 그사이에 하나 이상의 중간 연산이 있을 수 있다
- 각 중간연산은 스트림을 어떠한 방식으로 변환한다. 예컨데 각 원소에 함수를 적용하거나 특정 조건을 만족하지 못하는 원소를 걸러낼 수 있다
- 중간 연산들은 모두 한 스트림을 다른 스트림으로 변환 하는대, 변환된 스트림의 원소 타입은 변환 전 스트림의 원소 타입과 같을 수 있고, 다를 수도 있다
- 종단 연산은 마지막 중간연산이 내놓은 스트림에 최후의 연산을 가한다. 원소를 정렬해 컬렉션에 담거나, 특정 원소하나를 선택하거나, 모든 원소를 출력하는 형식이다
- 스트림 파이프라인은 지연평가 된다. 평가는 종단 연산이 호출될 때 이뤄지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다
    * 이러한 지연 평가가 무한 스트림을 다룰 수 있게 해주는 열쇠이다. 종단 연산이 없는 스트림 파이프라인은 아무런 일도 하지 않으니 종단 연산을 빼먹으면 안된다

- 스트림 API는 메서드 연쇄를 지원하는 플루언트 API이다. 즉, 파이프라인 하나를 구성하는 모든 호출을 연결하여 단 하나의 표현식으로 완성할 수 있다. 파이프라인 연결해 표현식 하나로 만들 수 있다
- 기본적으로 스트림 파이프라인은 순차적으로 수행된다. 파이프라인을 병렬로 실행 하려면 파이프라인을 구성하는 스트림중 하나에서 parallel 메서드를 호출해주기만 하면 되나, 효과를 볼수있는 상황은 많지 않다
- 스트림 API는 다재다능하여 사실상 어떠한 연산이라도 해낼수 있지만, 해야한다는 뜻은 아니다
- 스트림을 제대로 사용하면 프로그램이 짧고 깔끔해지지만, 잘못 사용하면 읽기 어렵고 유지보수도 힘들어진다
- 스트림을 언제 사용해야 하는지에 대한 확고부동한 규칙은 없지만, 참고할만한 노하우들은 있다

```java
public class Anagrams {
    public static void main(String args[]) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        Map<String, Set<String>> groups = new HashMap<>();
        try(Scanner s = new Scanner(dictionary)) {
            while(s.hasNext()) {
                String word = s.next();
                groups.computeIfAbsent(alphabetize(word), (unused) -> new TreeSet<>()).add(word);
            }
        }

        for(Set<String> group : groups.values()) {
            if(group.size() >= minGroupSize)
                System.out.println(group.size() + ": " + group);
        }
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return String(a);
    }
}
```

- 위 코드는 사전파일에서 단어을 읽어 사용자가 지정한 문턱값보다 원소수가 많은 아나그램 그룹을 출력하는 코드이다
- 여기서 먼저 주목해야할 부분은 ```groups.computeIfAbsent``` 부분인데, 각 단어를 삽입할때 자바8에서 추가된 computeIfAbsent 메서드를 사용했다. 이 메서드는 맵 안에 키가 있는지 찾은다음, 있으면 그 값을 없으면 건네진 함수 객체를 키에 적용하여 값을 계산해낸 다음 그 키와 매핑해두고 그 값을 반환한다

```java
public class Anagrams {
    public static void main(String args[]) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        Map<String, Set<String>> groups = new HashMap<>();
        try(Stream<String> word = new Scanner(dictionary)) {
            word.collect(
                groupingBy(word -> word.chars().sorted().
                    collect(StringBuilder::new, (sb.c) -> sb.append((char)c), StringBuilder::append).toString)))
                .values().stream()
                .filter(group -> group.size() >= minGroupSize)
                .map(group -> group.size() + ": " + group)
                .forEach(Syste.out::println);
        }
    }
}
```

- 위의 코드는 앞선 코드와 같은일을 하지만 스트림을 과하게 활용한 코드이다. 사전 파일을 여는 부분만 활용하면 프로그램 전체가 단 하나의 표현식으로 처리된다. 사전을 여는 작업을 분리한 이유는 단지 try-with-resources를 사용하기 위해서이다
- 위의 코드는 이해하기 상당히 어렵다. 확실히 짧지만 이해하기가 어렵다. 이처럼 스트림을 이용하면 프로그램이 읽거나 유지보수하기 어려워진다

```java
public class Anagrams {
    public static void main(String args[]) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        Map<String, Set<String>> groups = new HashMap<>();
        try(Stream<String> word = new Scanner(dictionary)) {
            word.collect(groupingBy(word -> alphabetize(word))
            .values().stream()
            .filter(group -> group.size() >= minGroupSize)
            .forEach(g -> Syste.out.println(g.size() + ": " + g));
        }
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return String(a);
    }
}
```

- 위 코드는 앞서의 두 코드와 같은 기능을 하지만 스트림을 적당히 사용하여, 원본 코드보다 짧지만 명확하기까지 하다
- 스트림을 전에 본적이 없더라도 이 코드를 이해하는데는 어렵지 않을것이다. try-with-resource 블록에서 사전 파일을 열고, 파일의 모든 라인으로 구성된 스트림을 얻는다. 스트림 변수의 이름을 word로 지어 스트림 안의 각 원소가 단어임을 명확히 했으며, 중간연산 없이 종단연산에서 모든 단어를 수집해 맵으로 모은다. 그후 맵의 values의 반환값으로 새 스트림을 열어 필터링 한 후 살아남은 리스트를 forEach로 출력한다
    * 이처럼 람다 매개변수의 이름은 주의해서 정해야 한다. 람다에서는 타입 이름을 자주 생략하므로 매개변수 이름을 잘 지어야 파이프라인에서의 가독성이 유지된다. 또한 alphabetize와 같은 도우미 메서드를 적절히 활용하는 일의 중요성은 일반 반복코드에서보다 훨씬 중요하다

- alphabetize도 스트림으로 구현할 수 있지만, 그렇게 하면 명확성이 떨어지고 잘못 구현할 가능성이 커진다. 심지어 자바가 기본타입인 char용 스트림을 지원하지 않기 때문에 느려질 수도 있다
    * 그렇다고 해서 자바가 char 스트림을 지원해야 했었다는것은 아니다. 불가능 하기 때문이다
    * char값들을 스트림으로 처리 할경우, char가 아닌 int로 처리되기 때문에, int에서 다시 char로 변환하는 과정을 거쳐야 하며, 이때문에 char값들을 처리할 때에는 스트림을 삼가하는것이 낫다

- 스트림을 처음 쓰기 시작하면 모든 반복문을 스트림으로 대체하고 싶은 유혹이 있지만, 서두르지 않는것이 좋다. 스트림으로 바꾸는게 가능할지라도 코드 가독성과 유지보수 측면에서 손해를 볼 수 있기 때문이다
- 그러니 기존 코드는 스트림을 사용하도록 리팩토링 하되, 새 코드가 나아보일때만 반영 하도록 하자
- 이번 아이템에서 보여준것처럼 스트림 파이프라인은 되풀이되는 계산을 함수 객체(주로 람다나 메서드참조)로 표현한다. 그런데 함수 객체로는 할 수 없지만, 코드 블록으로는 할 수 있는 일들도 있다
    * 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있다. 람다는 final이거나 사실상 final인 변수들만 읽을 수 있고 지역변수 수정은 불가능 하다
    * 코드 블록에서는 return을 메서드에서 빠져나가거나, break나, continue문으로 블록 바깥의 반복문을 종료하거나, 반복을 한번 건너뛰고 또 메서드 선언에 명시된 검사 예외를 던질수 있지만 람다는 이중 어떤것도 할 수 없다
    * 따라서 계산 로직에서 이상의 일들을 수행해야 한다면 스트림과는 맞지 않는다

- 물론, 스트림에 아주 안성 맞춤인 연산들도 있다
    * 원소들의 시퀀스를 일관되게 변환한다
    * 원소들의 시퀀스를 필터링한다
    * 원소들의 시퀀스를 하나의 연산을 사용해 결합한다(더하기, 연결하기, 최솟값 구하기 등)
    * 원소들의 시퀀스를 컬렉션에 모은다(아마도 공통된 속성을 기준으로 묶어가며)
    * 원소들의 시퀀스에서 특정 조건에 만족하는 원소를 찾는다

- 스트림으로 처리하기 어려운 일도 있는데, 대표적인 예로 한 데이터가 파이프라인의 여러 단계(stage)를 통과할 때 이 데이터의 각 단계에서의 값들에 동시에 접근하기는 어려운 경우이다
- 스트림 파이프라인은 일단 한 값을 다른 값에 매핑하고 나면 원래의 값은 잃는 구조이기 때문이다. 원래 값과 새로운 값의 쌍을 저장하는 매핑 객체를 사용해 우회하는 방법도 있긴 하지만, 그리 만족스러운 해법은 아니다
- 매핑 객체가 필요한 단계가 여러곳이라면 특히 더 그렇다. 이런 방식은 코드양도 많고 지저분하여 스트림을 쓰는 주 목적에서 완전히 벗어나며, 가능한 경우라면 앞 단계의 값이 필요할 때 매핑을 거꾸로 수행하는 방법이 나을것이다
- 예를들어 처음 20개의 메르센 소수(2^p -1 , p가 소수일때 해당 메르센 수도 소수일수 있는데 이때를 메르센 소수라고 한다)를 출력하는 프로그램이 있다고 하자. 아마도 이 파이프라인의 첫 스트림으로는 모든 소수를 사용할것이다

```java
static Steam<BigInteger> primes() {
    return Stream.interate(TWO, BigInteger::nextPorbablePrime);
}
```

- 메서드의 이름 primes는 스트림의 원소가 소수임을 말해준다. 스트림을 반환하는 메서드 이름은 이처럼 원소의 정체를 알려주는 복수 명사로 쓰기를 강력히 추천한다
- 이 메서드가 이용하는 Stream.iterate라는 정적 팩터리는 매개변수를 2개 받는데, 첫번째 매개변수는 스트림의 첫 원소이고, 두번째 매개변수는 스트림에서 다음 원소를 생성해주는 함수이다

```java
public static void main(String[] args) {
    primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
        .filter(mresenne -> mresenne.isProbablePrime(50))
        .limit(20)
        .forEach(Syste.out::println);
}
```

- 이 코드는 앞서의 설명을 정직하게 구현했다. 소수들을 사용해 메르센 수를 계산하고, 결괏값이 소수인 경우만 남긴다음(매직넘버 50은 소수성 검사가 true를 반환할 확률을 제어한다), 결과 스트림의 원소수를 20개로 제한하고 작업이끝나면 결과를 출력한다
- 이제 우리가 메르센 소수 앞에 지수(p)를 출력하기를 원한다고 해보자. 이 값은 초기 스트림에만 나타나므로 결과를 출력하는 종단 연산에서는 접근 할 수 없다. 하지만 다행이 첫번째 중간연산에서 수행한 매핑을 거꾸로 수행해 메르센 수의 지수를 쉽게 계산 할 수 있다. 지수는 단순히 숫자를 2진수로 표현한 다음 몇비트인지를 세면 나오므로 종단 연산을 다음과 같이 바꾸면 된다

```java
.forEach(mp -> System.out.println(mp.bitLength() + ": " + mp));
```

- 스트림과 반복중 어느쪽을 써야 할지 바로 알기 어려운 작업도 많다. 카드 덱을 초기화 하는 작업을 생각해보면, 카드는 숫자(rank)와 무늬(suit)를 묶은 불변값 클래스이고, 숫자와 무늬는 모두 열거타입 이라고 할 수 있다
- 이 작업은 두 집합의 원소들로 만들수 있는 가능한 모든 조합을 계산하는 문제로, 수학자들은 이를 두 집합의 데카르트곱 이라고 부른다. 다음 코드는 이 데카르트곱을 for-each 반복문을 중첩해서 구현한 코드로 스트림에 익숙하지 않은 사람에게 친숙한 방법일 것이다

```java
private static List<Card> newDeck() {
    List<Card> result = new ArrayList<>();
    for(Suit suit : Suit.values()) {
        for(Rank rank : Rank.values())
            result.add(new Card(suit,rank));
    }
    return result;
}
```

- 다음은 위 코드를 스트림으로 구현한 코드이며, 중간 연산으로 사용한 flatMap은 스트림의 원소 각각을 하나의 스트림으로 매핑 한 다음, 그 스트림들을 다시 하나의 스트림이로 합친다. 이를 평탄화 라고 한다

```java
private static List<Card> newDeck() {
    return Steam.of(Suit.values())
        .flatMap(suit ->
            Stream.of(Rank.values())
                .map(rank -> new Card(suit,rank)))//중첩된 람다
        .collect(toList());
}
```

- 어느 newDeck이 좋아보일지는 개인의 취향과 프로그래밍 환경의 문제이다
- 처음 방식은 더 단순하고 자연스러워보여 이해하고 유지보수하기에 처음 코드가 더 편한 프로그래머가 많겟지만, 스트림 방식을 편하게 생각하는 프로그래머도 있다
- 스트림과 함수형 프로그래밍에 익숙한 프로그래머라면 스트림 방식이 조금 더 명확하고 그리 어렵지도 않을것이다
- 즉 스트림과 반복중 어느것이 더 나은지 확신하기 어렵다면 둘다 해보고 더 나은쪽을 택해야 한다
