---
layout: post
title: "Effective Java - 아이템53: 가변인수는 신중히 사용하라"
description: 가변인수는 신중히 사용하라
date: 2020-09-06 16:17:00 +09:00
categories: EffectiveJava Study
---


# 메서드

## 아이템 53 : 가변인수는 신중히 사용하라

- 가변인수는 명시한 타입의 인수를 0개 이상 받을 수 있다
- 가변인수 메서드를 호출하면, 가장 먼저 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메서드에 건네준다

```java
static int sum(int... args){
    int sum = 0;
    for(int arg : args) {
        sum += arg;
    }
    return sum;
}
```

- 위 코드는 입력받은 int 인수들의 합을 계산해주는 가변인수 메서드 이다
- 인수가 1개 이상이여야 할때도 있는데, 인수 개수는 런타임에 자동생성된 배열의 길이로 알 수 있다

```java
static int min(int... args){
    if(args.length == 0)
        throw new IllegralArgumentException("인수가 1개 이상 필요합니다");
    int min = args[0];
    for(int i = 1; i < args.length; i++) {
        if(args[i] < min)
            min = args[i];
    }
    return min;
}
```

- 이 방식에는 문제가 몇개 있다. 가장 심각한 문제는 인수를 0개만 넣어 호출하면 컴파일 타임이 아닌 런타임에 실패한다는 점이며, 코드도 지저분하다
    * min의 초기값을 Integer.MAX_VALUE로 설정하지 않고는 더 명료한 for-each문도 사용할 수 없다

- 다행이 훨씬 나은 방법이 있다. 다음 코드처럼 매개변수 2개를 받도록 하면 된다. 즉 첫번째로는 평범한 매개변수를 받고, 가변인수는 두번쨰로 받으면 된다

```java
static int min(int firstArg, int... remainingArgs){
    int min = firstArg;
    for(int arg : remainingArgs) {
        if(arg < min) min = arg;
    }
    return min;
}
```

- 이상 예에서 보듯, 가변인수는 인수 개수가 정해지지 않았을떄에 아주 유용하다. printf는 가변인수와 한묶음으로 자바에 도입 되었고, 이때 핵심 리플렉션 기능도 재정비 되었다
- 하지만 성능에 민감한 상황이라면 가변인수가 걸림돌이 될 수도 있다. 가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화 하기 때문이다
    * 다행이 이 비용을 감당할수는 없지만 가변인수의 유연성이 필요할때 사용할 수 있는 패턴이 있다
    * 예를들어 해당 메서드 호출의 95%가 인수를 3개 이하로 사용한다고 할때, 인수가 0~4개인것까지 총 5개를 다중정의하고, 4개인것의 마지막 인수를 가변인수로 하는것이다
    * 그러면 메서드 호출중 단 5%만 배열을 생성한다
    * EnumSet도 이 기법을 사용한다

