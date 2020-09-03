---
layout: post
title: "Effective Java - 아이템47: 반환 타입으로는 스트림보다 컬렉션이 낫다"
description: 반환 타입으로는 스트림보다 컬렉션이 낫다
date: 2020-09-03 22:52:00 +09:00
categories: EffectiveJava Study
---


# 람다와 스트림

## 아이템 47 : 반환 타입으로는 스트림보다 컬렉션이 낫다

- 원소 시퀀스, 즉 일련의 원소를 반환하는 메서드는 수없이 많다
- 자바7까지는 이런 메서드의 반환 타입으로 Collection, Set, List와 같은 컬렉션 인터페이스 혹흔 Itetable이나 배열을 썻다
- 이중 가장적합한 타입을 선택하기란 어렵지 않다. 기본적으로 컬렉션 인터페이스이기 때문이다
- for-each문에서만 쓰이거나, 반환된 원소 시퀀스가 일부 Collection 메서드를 구현할 수 없을때에는 Iterable 인터페이스를, 반환 원소들이 기본타입이거나 성능에 민감한 상황이라면 배열을 썻다
- 하지만 스트림이라는 개념이 생기면서 이 선택은 아주 복잡한일이 되어버렸다
- 원소 시퀀스를 반환할 때에는 당연히 스트림을 사용해야한다는 이야기를 들어봣을지도 모르지만, 스트림은 반복을 지원하지 않는다. 따라서 스트림과 반복을 알맞게 조합해야 좋은 코드가 나온다
- API를 스트림으로만 짜놓으면 반환된 스트림을 for-each로 반복하길 원하는 사용자는 당연히 불만이 생길것이다
    * 여기서 한가지 재미있는 사실이 있다. 사실 Stream인터페이스는 Iterable엔터페이스가 정의한 추상 메서드를 전부 포함할 뿐만 아니라, Iterable에서 정의한 방식대로 동작한다. 하지만 for-each로 스트림을 반복할 수 없는 이유는, Stream이 Iterable을 확장한것이 아니여서 이다

- 이 문제를 해결해줄 멋진 우회로는 없지만, Stream의 iterator메서드에 메서드 참조를 건네면 코드가 지저분하고 직관성이 떨어지지만 못쓸 정도는 아니게 되는 것 처럼 보이지만 다음과 같은 끔직한 코드가 되어버린다

```java
for(Processhandle ph : (Iterable<ProcessHandle>) ProcessHandle.allProcess()::iterator)
```

- 작동은 하지만, 실전에 쓰기에는 너무 난잡하고 직관성이 떨어진다. 다행이 어댑터 메서드를 쓰면 상황은 조금 나아진다

```java
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iterator;
}

for(ProcessHandle p : iterableOf(ProcessHandle.allProcess()))
```

- 다행이 이 경우엔 자바의 타입 추론이 문맥을 잘 파악하여 어댑터 메서드 안에서 따로 형변환 하지 않아도 된다
- 아이템 45의 아나그램 프로그렘에서 스트림 버전은 사전을 읽을때 Files.lines메서드를 이용했고, 반복버전은 스캐너를 이용했다. 파일을 읽는동안 발생하는 모든 예외를 알아서 처리해준다는 점에서 Files.lines를 반복버전에서도 썻어야 햇지만, 스트림만 반환하는 API가 반환한 값을 for-each로 반복하길 원하는 프로그래머가 감수해야할 부분이다
- 반대로 API가 Iterable만 반환하면 이를 스트림 파이프라인에서 처리하려는 프로그래머가 화를 낼것이다. 이 또한 어댑터를 구현해야 한다

```java
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
    return StreamSupport.stream(iterable.spliterator(), flase);
}
```

- 객체 시퀀스를 반환하는 메서드를 작성하는데, 이 메서드가 오직 스트림 파이프라인에서만 쓰일것을 안다면 마음놓고 스트림을 반환하게 해주자
- 반대로 반환된 객체들이 반복문 안에서만 쓰일것을 안다면 Iterable을 반환하게 해주자
- 하지만 공개 API를 작성하는 경우에는 스트림 파이프라인을 사용하려는 사람과 반복문에서쓰려는 사람을 모두 배려해야 한다
- Collection 인터페이스는 Iterable의 하위타입이고, stream 메서드도 제공하니 반복과 스트림을 동시에 지원한다. 따라서 원소 시퀀스를 반환하는 공개 API의 반환 타입에는 Collection이나 그 하위타입을 쓰는게 일반적으로 최선이다
- Arrays역시 Arrays.asList와 HashSet과 같은 표준 컬렉션 구현체를 반환하는게 최선 일 수 있다. 하지만 단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려스는 안된다
- 반환할 시퀀스가 크지만 표현을 간결하게 할수 있다면 전용 컬렉션을 구현하는 방안을 검토해보자
- 예컨데 주어진 집합의 멱집합을 반환하는 상황이라면, 원소의 개수가 n개이면 멱집합 원소의 개수는 2^n 이므로 이를 표준 컬렉션 구현체에 저장하는 생각은 위험하다. 하지만 AbstractList를 이용하며 각원소의 인덱스를 비트벡터로 사용하면 훌륭한 전용 컬렉션을 손쉽게 구현 할 수 있다

```java
public class PowerSet {
    public static final <E> Colelction<Set<E>> of (Set<E> s) {
        List<E> src = new Array<>(s);
        if(src.size > 30)
            throw new IllegalArgumentException("Too manay arguemnt(limit 30): " + s);
        return new AbstractList<Set<E>>() {
            @Override
            public int size() {
                return 1 << src.size();
            }

            @Override
            public boolean contains(Object o) {
                return o instanceof Set && src.containsAll((Set)o);
            }

            @Override
            public Set<E> get(int index) {
                Set<E> result = new HashSet<>();
                for(int i = 0; index != 0; index >>= 1) {
                    if((index & 1) == 1)
                        result.add(src.get(i));
                }
                return result;
            }
        };
    }
}
```

- AbstractCollection을 활용해서 Collection 구현체를 작성할 때에는 Iterable용 메서드 외에 2개만 더 구현하면 된다. contains와 size이다
- 이 메서드는 손쉽게 효율적으로 구현할 수 있지만, 여러가지 이유(반복이 시작되기 전에는 시퀀스의 내용을 확정할 수 없음 등)으로 contains와 size를 구현하는게 불가는 한 경우에는 스트림이나 Iterable을 반환하는편이 낫다
- 떄로는 단순히 구현하기 쉬운쪽을 택할 수도 있다. 입력이 리스트의 연속적인 부분 리스트를 모두 반환하는 메서드를 작성한다고 생각해보면, 필요한 부분 리스트를 만들어 표준 컬렉션에 담는것은 단 3줄이면 된다
    * 하지만 이 컬렉션은 입력 리스트 크기의 거듭제곱 만큼 메모리를 차지한다. 좋은 방법은 아닌게 명백하다

- 전용 컬렉션을 구현하기란 지루한일이다. 특히 자바는 이럴때 쓸만한 골격 Iterable을 제공하지 않으니 더 지루하다
- 하지만 입력 리스트의 모든 부분을 스트림으로 구현하기는 어렵지 않다. 약간의 통찰만 있으면 된다
    * 첫번째 원소를 포함하는 부분 리스트를 그 리스트의 프리픽스라 해보자. 같은 식으로 마지막 원소를 포함하는 부분 리스트를 서픽스라고 하면, 이제 어떤 리스트의 부분리스트는 단순히 그 리스트의 프리픽스의 서픽스에(혹은 서픽스의 프리픽스)에 빈 리스트 하나만 추가하면 된다

```java
public class SubLists {
    public static <E> Stream<List<E>> of(List<E> list) {
        return Stream.concat(Stream.of(Collections.emptyList)),
            prefixes(list).flatMap(SubLines::suffixes));
    }

    private static <E> Steam<List<E>> prefixes(List<E> list) {
        return IntStream.rangeClosed(1, list.size())
            .mapToObj(end -> list.subList(0,end));
    }

    private static <E> Stream<List<E>> suffixes(List<E> list) {
        return IntStream.range(0,list.size())
            .mapToObj(start -> list.subList(start,list.size()));
    }
}
```

- Stream.concat 메서드는 반환되는 스트림에 빈 리스트를 추가하며, flatMap메서드는 모든 프리픽스의 모든 서픽스로 구성된 하나의 스트림을 만든다. 마지막으로 프리픽스들과 서픽스들의 스트림은 IntStream.range와 IntStream.rangeClosed가 반환하는 연속된 정숫값들을 매핑해서 만들었다
- 쉽게 말해 이 관용구는 정수 인덱스를 사용한 표준 for 반복문의 스트림 버전이다. 따라서 for반복문을 중첩해 만든것과 취지가 비슷하지만, 이쪽이 읽기에 더 좋다

```java
for(int start = 0; start < src.size(); start++) {
    for(int end = start +1; end <= src.size(); end++) {
        System.out.println(src.subList(start,end));
    }
}
```

- 이상으로 스트림을 반환하는 두가지 구현을 모두 알아봤는데, 모두 쓸만은하다. 하지만 반복을 사용하는게 더 자연스러운 상황에서도 사용자는 그냥 스트림을 쓰거나 Steam을 Iterable로 반환하는 어댑터를 이용해야 한다
- 하지만 이러한 어댑터는 클라이언트 코드를 더 어수선하게 만들고 느리다. 직접 구현한 전용 Collection을 사용하는게 더 빠르다
