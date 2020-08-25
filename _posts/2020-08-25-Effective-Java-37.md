---
layout: post
title: "Effective Java - 아이템37: ordinal 인덱싱 대신 EnumMap을 사용하라"
description: ordinal 인덱싱 대신 EnumMap을 사용하라
date: 2020-08-25 21:24:00 +09:00
categories: EffectiveJava Study
---


# 열거타입과 어노테이션

## 아이템 37 : ordinal 인덱싱 대신 EnumMap을 사용하라

```java
class Plant {
    enum LifyCycle { ANNUAL, PERENNIAL, BIENIAL }

    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }
}

Set<Plant>[] paintsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for(int i = 0; i < plantsByLifeCycle.length; i++)
    plantsByLifeCycle[i] = new HashSet<>();

for(Plant p : garden)
    plantByLifeCycle[p.lifeCycle.ordinal()].add(p);

for(int i = 0; i < plantsByLifeCycle.length; i ++){
    System.out.printf("%s: %s%n",Plant.LifeCycle.values()[i],plantsByLifeCycle[i]);
}
```

- 이따금 위 코드처럼 배열이나 리스트에서 원소를 꺼낼때, ordinal 메서드로 인덱스를 얻는 코드가 있다
- 동작은 하지만 문제는 한가득이다. 배열은 제너릭과 호환되지 않으므로, 비검사 형변환을 수행해야 하고, 깔끔히 컴파일 되지 않는다
- 또한, 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야만 하며, 가장 심각한 문제는 정확한 정숫값을 사용한다는걸 개발자가 직접 보증해야 한다는 점이다. 정수는 열거타입과 달리 안전하지 않기 때문이다
- 여기서 배열은 실질적으로 열거 타입 상수를 값으로 매핑하는 일을 한다. 그러니 Map을 사용할수도 있다
- 열거 타입을 키로 사용하도록 설계한 Map 구현체가 있는데 그것이 바로 EnumMap이다

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plants.LifeCycle.values()); 
for(Plant.LifeCycle lc : Plant.LifeCycle.values()) 
    plantsByLifeCycle.put(lc, new HashSet<>());
for(Plant p : garden)
    plantByLifeCycle.get(p.lifeCycle).add(p);
```

- 위 코드는 원래의 코드에 비해 더 짧고 명료하며 성능도 동등하다
- 안전하지 않은 형변환은 쓰지 않고, 맵의 키인 열거 타입이 그 자체로 줄력용 문자열을 제공하니 출력 결과에 직접 레이블을 달 일도 없다
- 나아가 배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 원천봉쇄 된다
- EnumMap의 성능이 ordinal을 쓴 배열 버전에 비견되는 이유는, 그 내부에서 배열을 사용하기 때문이다
- EnumMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로, 런타임 제너릭 타입 정보를 제공한다

```java
System.out.println(Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle)));
```

- 위와 같이 스트림을 활용해서 맵을 관리하면 코드를 더 줄일수 있는데, 이럴 경우 EnumMap이 아닌 고유 Map구현체를 쓰므로 EnumMap에서 얻은 공간과 성능 이점이 사라지는 문제가 있다

```java
System.out.println(Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle,
    () -> new EnumMap<>(LifeCycle.class), toSet())));
```

- 매개변수 3개짜리 Collectors.groupingBy 메서드는 mapFactory 매개변슈에 원하는 맵 구현체를 위와 같이 명시해 호출할 수 있다
- 스트림을 사용하면 EnumMap만 사용했을때와는 살짝다르게 동작한다. EnumMap 버전은 언제나 LifeCycle당 하나씩의 중첩 맵을 만들지만, 스트림 버전에서는 해당 LifeCycle에 속하는것이 있을때만 만든다

```java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

        private static final Transition[][] TRANSITIONS = {
            { null, MELT, SUBLIME },
            { FREEZE, null, BOIL },
            { DEPOSIT, CONDENSE, null}
        };

        public static Transition from(Phase from, Phase to) {
            return Transition[from.ordinal()][to.ordinal()];
        }
    }
}
```

- 위 코드처럼 두 열거 타입 값들을 매핑하느라 ordinal을 두번 쓴 배열들의 배열을 본적도 있을수 있다
- 위 코드는 멋져보이지만 겉모습에 속으면 안된다. 앞서 보여준 간단한 정원 예제와 마찬가지로 컴파일러는 ordinal과 배열 인덱스와의 관계를 알 수 없다.
즉 Phase나 Transition을 수정하면서 TRANSITIONS을 수정하지 않는다면 런타임 오류가 날것이다
- 위와 같은 코드도 맵 2개를 중첩한다면 쉽게 해결할 수 있다

```java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phash from, Phase to) {
            this.from = from;
            this.to = to;
        }

        private static final Map<Phase, Map<Phase, Transition>> m =
            Stream.of(values()).collect(groupingBy(t -> t.from, 
                () -> new EnumMap<>(Phase.class),
                toMap(t -> t.to, t -> t, (x,y) -> y, () -> EnumMap<>(Phase.class))));
        
        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
```

- 위 코드에서 맵을 초기화 하는 코드는 제법 복잡한데, 이 맵을 초기화 하기위한 수집기(Collector) 2개를 차례로 사용한다
    * 첫번쨰 수집기는 groupingBy에서는 이전 Phase를 기준으로 묶는다
    * 두번째 수집기인 toMap에서는 이후 Phase를 Transition에 대응시키는 EnumMap을 생성한다
    * 두번째 수집기의 병합 함수인 (x,y) -> y는 선언만 하고 실제로 쓰이지는 않는데, 이는 단지 EnumMap을 얻으려면 맵 팩터리가 필요하고, 수집기들은 점층적 팩토리를 제공하기 때문이다

- 이제 위 코드에서 새로운 Phase인 PLASMA가 추가되고 그에 해당하는 Transition인 IONIZE, DEIONSIZE가 추가되면 어떻게 할까?

```java
public enum Phase {
    SOLID, LIQUID, GAS, PLASMA;

    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),
        IONDIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);
    }
    //이하 동일
}
```

- 위와 같이 추가만 해주면 나머지 코드는 완벽히 동작한다