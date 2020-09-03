---
layout: post
title: "Effective Java - 아이템46: 스트림에서는 부작용이 없는 함수를 사용하라"
description: 스트림에서는 부작용이 없는 함수를 사용하라
date: 2020-09-03 21:17:00 +09:00
categories: EffectiveJava Study
---


# 람다와 스트림

## 아이템 46 : 스트림에서는 부작용이 없는 함수를 사용하라

- 스트림을 처음 봐서는 이해하기 어려울 수 있다. 원하는 작업을 스트림 파이프라인으로 표현하는 것조차 어려울 수 있으며, 성공하여  프로그램이 동작하더라도 장점이 무엇인지 쉽게 와닿지 않을 수 도 있다
- 스트림은 그저 또하나의 API가 아닌 함수형 프로그래밍에 기초한 패러다임이기 떄문이다. 스트림이 제공하는 표현력, 속도, 병렬성을 얻으려면 API는 말할것도 없고 이 패러다임까지 함께 받아들여야 한다
- 스트림 패러다임의 핵심은 계산을 일련의 변환으로 재구성 하는 부분이다
- 이때 각 변환단계는 가능한 이전 단계의 결과를 받아 처리하는 순수한 함수여야한다. 순수 함수란 오직 입력만이 결과에 영향을 주는 함수를 마한다. 다른  가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않아야 한다. 이렇게 하려면(중간 단계든 종단 단계든) 스트림 연산에 건네는 함수 객체는 모두 부작용이 없어야 한다

```java
Map<String, Long> freqe = new HashMap<>();
try(Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
```

- 위의 코드는 종종 볼 수 있는 스트림 코드로, 텍스트 파일에서 단어별 수를 세어 빈도표를 만드는 일을 한다
- 위의 코드는 스트림, 람다, 메서드 참조를 사용했고 결과도 올바르지만 결코 스트림 코드라고 할 수 없다. 스트림 코드를 가장한 반복적 코드이다
- 스트림 API의 이점을 살리지 못하여 같은 기능의 반복적 코드보다 조금더 길고, 읽기 어렵고, 유지보수에도 좋지 않다
- 이 코드의 모든 작업이 종단연산인 forEach에서 일어나는데, 이때 외부 상태(빈도표)를 수정하는 람다를 실행하면서 문제가 생긴다. forEach가 그저 스트림이 수행한 연산을 보여주는 일 이상을 하는것을(이 코드에서는 람다가 상태를 수정하고 있다) 보니 나쁜 코드일 것 같은 냄새가 난다

```java
Map<String, Long> freqe = new HashMap<>();
try(Stream<String> words = new Scanner(file).tokens()) {
    freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

- 위 코드는 앞서와 같은일을 하지만 이번에는 스트림 API를 제대로 사용하였다. 그 뿐만 아니라 짧고 명확하다
- 자바 프로그래머라면 for-each 반복문을 사용할 줄 알텐데, for-each 반복문은 forEach 종단연산과 비슷하게 생겻다. 하지만 forEach 연산은 종단 연산중 가장 기능이 적고 가장 덜 스트림 스럽다. 대놓고 반복적이라서 병렬화 할 수도 없다
- forEach 연산은 스트림 계산 결과를 보고할때만 사용하고 계산하는데에는 쓰지 말자. 물론 가끔은 스트림 계산 결과를 기존 컬렉션에 추가하는등의 용도로는 쓸 수 있다
- 이 코드는 Collector를 사용하는데, 스트림을 사용하려면 꼭 배워야 하는 개념이다
- java.util.stream.Collectors 클래스는 메서드를 39개나 가지고 있고, 그중에는 타입 매개변수가 5개나 되는것도 있다. 다행이 이러한 복잡한 세부 내용을 몰라도 이 API의 장점을 대부분 활용할 수 있다
    * 익숙해지기 전까지는 Collectors 인터페이스를 잠시 잊고, 그저 축소 전략을 캡슐화한 블랙박스라고 생각하자
    * 여기서 축소는 스틑림의 원소들을 객체 하나에 취합한다는 뜻이며, Collector가 생성하는 객체는 일반적으로 컬렉션이여서 Collector이다

- Collector를 사용하면 스트림의 원소를 손쉽게 컬렉션으로 모을 수 있다. Collector는 총 3가지로 toList(), toSet() toCollection(collectionFactory)가 그 주인공이다
    * 차례로 리스트, 집합, 프로그래머가 지정한 컬렉션 타입을 반환한다

- 지금까지 배운 지식을 활용해 빈도표에서 가장 흔한 단어 10개를 뽑아내는 스트림 파이프라인을 작성하면 다음과 같다

```java
List<String> topTen = freq.keySet().stream()
    .sorted(comparing(freq::get).reversed())
    .limit(10)
    .collect(toList()); // Collector를 정적 임포트 해두면 이렇게 쓸 수 있다
```

- 이 코드에서 어려운 부분은 sorted에 넘긴 비교자, 즉 comparing(freq::get).reversed() 뿐이다
    * comparing 메서드는 키 추출 함수를 받는 비교자 생성 메서드이다
    * 키 추출 함수로 쓰인 freq::qet은 입력받은 단어(키)를 빈도표에서 찾아(추출) 그 빈도를 반환한다
    * 마지막으로 가장 흔한 단어가 위로 오도록 비교자(comparing)을 역순(reversed)으로 정렬(sorted)한다

- Collectors의 나머지 36개 메서드들도 알아보자. 이 중 대부분은 스트림을 맵으로 취합하는 기능으로, 진짜 컬렉션에 취합하는 것보다 훨씬 복잡하다
- 스트림의 각 우너소는 키 하나와 값 하나에 연관되어 있다. 그리고 다수의 스트림원소가 같은 키에 연관될 수 있다
- 가장 간단한 맵 수집기는 toMap(keyMapper, valueMapper)로 스트림원소를 키에 매핑하는 함수와 값에 매핑하는 함수를 인수로 받는다

```java
private static final Map<String, Operation> stringToEnum =
    Stream.of(values()).collect(toMap(Object::toString, e -> e));
```

- 이 간단한 toMa형태는 스트림의 각 원소가 고유한 키에 매핑되어 있을때 적합하다. 스트림 원소 다수가 같은 키를 사용한다면 파이프라인이 IllegalStateException을 던지며 종료 될것이다
- 더 복잡한 형태의 toMap이나 groupingBy는 이런 충돌을 다루는 다양한 전략을 제공한다. 예컨대 toMap이나 groupingBy는 이런 충돌을 다루는 다양한 전략을 제공한다
    * 예컨대 toMap에 키 매퍼와 값 매퍼는 물론 병합함수까지 제공 할 수 있다. 병합 함수의 형태는 BinaryOperator<U>이며 여기서 U는 해당 맵의 값타입 이다
    * 같은 키를 공유하는 값들은 이 병합 함수를 사용해 기존 값에 합쳐진다. 병합함수가 곱셈이라면 키가 같은 모든 값들을 곱한 결과를 얻는것이다

- 인수 3개를 받는 toMap은 어떤 키와 그 키에 연관된 원소들중 하나를 골라 연관짓는 맵을 만들때 유용하다
- 다음 예시는 음악가의 앨범들을 담은 스트림을 가지고, 음악가와 그 음악가의 베스트 엘범을 연관짓는 것이다

```java
Map<Artist,Album> topHits = albums.collect(
    toMap(Album::artiest, a -> a, maxBy(comparing(Album::sales)));
)
```

- 여기서 비교자로 BinaryOperator에서 정적 임포트한 maxBy라는 정적 팩터리 메서드를 사용했다
    * maxBy는 Comparator<T>를 입력받아 BinaryOperator<T>를 돌려준다
    * 위의 경우에는 비교자 생성 메서드인 comparing이 maxBy에 넘겨줄 비교자를 반환하는데, 자신의 키 추출함수로는 Album::sales를 받았다
    * 복잡해 보일 수 있지만, 매끄럽게 읽을 수 있다. 말로 풀어보면 "앨범 스트림을 맵으로 바꾸는데, 그 맵은 각 음악가와 그 음악가의 베스트 엘범을 짝지은 것이다" 이며 이는 우리의 의도와 완벽히 부합한다

- 인수가 3개인 toMap은 충돌이나면 마지막 값을 취하는(last-write-wins) 수집기를 만들때에도 유용하다
    * 많은 스트림의 결과가 비 결정적이지만, 매핑 함수가 키 하나에 연결해준 값들이 모두 같을떄, 혹은 값이 다르더라도 모두 허용되는 값일때 이런 수집기가 필요해진다

```java
toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
```

- 마지막 toMap은 네번째 인수로 맵 팩터리를 받는다. 이 인수로는 EnumMap이나 TreeMap처럼 원하는 특정 맵 구현체를 직접 지정 할 수 있다
- 이상 3가지 toMap에는 변종이 있는데, 그중 toConcurrentMap은 병렬 실행이 된 후 결과로 ConcurrentHashMap을 생성한다
- 이번에는 Collectors가 제공하는 또다른 메서드인 groupingBy이다. 이 메서드는 입력으로 분류함수를 받고 출력으로는 원소들을 카테고리별로 모아놓은 맵을 담은 수집기를 반환한다
- 분류 함수는 입력받은 원소가 속하는 카테고리를 반환한다. 그리고 이 카테고리가 해당 원소의 맵 키로 쓰인다
- 다중 정의된 groupingBy중 형태가 가장 간단한것은 분류함수 하나를인수로 받아 맵으로 반환한다. 반환된 맴베 담긴 각각의 값은 해당 카테고리에 속하는 원소들을 모두 담은 리스트이다

```java
words.collect(groupingBy(word -> alphaetize(word)))
```

- groupingBy가 반환하는 수집기가 리스트 외의 값을 갖는 맵을 생성하게 하려면, 분류 함수와함께 다운 스트림 수집기도 명시해야 한다
    * 다운 스트림 수집기의 역할은 해당 카테고리의 모든 원소를 담은 스트림으로부터 값을 생성하는 일이다
    * 이 매개변수를 사용하는 가장 간단한 방법은 toSet()을 넘기는 것이다. 그러면 groupingBy는 원소들의 리스트가 아닌 집합(set)을 값으로 갖는 맵을 만들어낸다
    * toSet 대신 toCollection(collectionFactory)를 건네는 방법도 있는데, 예상할 수 있듯이 이렇게 하면 리스트나 집합 대신 컬렉션을 값을 갖는 맵을 생성한다. 원하는 컬렉션 타입을 선택 할 수 있다는 유연성은 덤이다
    * counting()을 넘기는 방법도 있는데, 이렇게 하면 각 카테고리(키)를 원소를 담은 컬렉션이 아닌 해당 카테고리에 속하는 원소의 개수(값)과 매핑한 맵을 받는다

```java
Map<String, Long> freq = words
    .collect(groupingBy(String::toLowerCase, counting()));
```

- groupingBy의 세번째 버전은 다운 스트림 수집기에 더해 맵 팩터리도 지정할 수 있게 해준다. 참고로 이 메서드는 점층적 인수 목록 패턴에 어긋난다. 즉 mapFactory 매개변수가 downStream매개변수보다 앞에 놓인다
    * 이 버전의 groupingBy를 사용하면 맵과 그 안에 담긴 컬렉션의 타입을 모두 지정할 수 있다. 값이 TreeSet인 TreeMap을 반환하는 수집기를 만들 수 있는 것이다

- 이상 총 3가지의 groupingBy 각각에 대응하는 groupingByConcurrent 메서드들도 볼 수 있다. 이름에서 알 수 있듯이 대응하는 메서드의 동시 수행 버전으로 ConcurrentHashMap 인스턴스를 만들어준다
- 많이 쓰이지는 않지만 groupingBy의 사촌격인 partitioningBy도 있다. 분류 함수 자리에 predicate를 받고 키가 Boolean인 맵을 반환한다. predicate에 더해 다운스트림 수집기까지 입력받는 버전도 다중 정의 되어있다
- counting 메서드가 반환하는 수집기는 다운 스트림 수집기 전용이다. Stream의 count 메서드를 직접 사용하여 같은 기능을 수행할 수 있으니, collect(counting()) 형태로 사용할 일은 전혀 없다
    * Collections에는 이런 속성의 메서드가 16개나 더 있다
    * 그중 9개는 이름이 summing, averaging, summarizing으로 시작하며 각각 int, long, double 스트림용으로 하나씩 있다
    * 다중정의된 reducing 메서드들, filtering, mapping, flatMapping, collectingAndThen 메서드가 있는데 대부분의 프로그래머들은 이들의 존재를 모르고 있어도 상관없다. 설계 관점에서 보면, 이 수집기들은 스트림 기능의 일부를 복제하여 다운스트림 수집기를 작은 스트림처럼 동작하게 만든것이다

- 이제 3개만 더 살펴보면 Collectors의 메서드를 모두 훑게 된다. 남은 이 3개의 메서드들은 특이하게도 Collectors에 정의되어 있지만 수집과는 관련이 없다
- 그중 maxBy, minBy는 인수로 받은 비교자를 이용해 스트림에서 값이 가장 작은 혹은 가장 큰 원소를 찾아 반환한다. Stream 인터페이스의 min과 max를 살짝 일반화 한 것이자, java.util.function.BinaryOperator의 minBy와 maxBy메서드가 반환하는 이진 연산자의 수집기 버전이다
- 마지막 남은 메서드는 joining인데, 이 메서드는 문자열등의 CharSequence 인스턴스의 스트림에만 적용 할 수 있다
- 이중 매개변수가 없는 joining은 단순히 원소들을 연결하는 수집기를 반환한다
- 인수가 하나짜리 joining은 CharSequence 타입의 구분 문자를 매개변수로 받는다. 연결 부위에 이 구분문자를 삽입하는데, 구분문자로 쉼표를 입력하면 CSV형태의 문자열을 만들어 주는것이다
- 인수가 3개짜리 joining은 구분문자에 더해 접두문자(prefix)와 접미문자(suffix)도 받는다
    * 접두문자를 [, 접미문자를 ], 구분문자를 , 로 해두면 [came, saw, conquered]처럼 컬렉션을 출력하는듯한 문자열을 생성 해주는것이다

- 이처럼 스트림 파이프라인 프로그래밍의 핵심은 부작용이 없는 함수 객체에 있는데, 스트림 뿐만 아니라 스트림 관련 객체에 건네지는 모든 함수가 부작용이 없어야 한다
- 스트림을 올바르게 사용하려면 수집기를 잘 알아둬야 하는데, 가장 중요한 수집기 팩터리는 toList, toSet, toMap, groupingBy, joining이다
