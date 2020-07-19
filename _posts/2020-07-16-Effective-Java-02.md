---
layout: post
title: "Effective Java - 아이템2: 생성자에 매개변수가 많다면 빌더를 고려하라"
description: 생성자 대신 빌더
date: 2020-07-19 19:45:00 +09:00
categories: EffectiveJava Study
---


# 객체의 생성과 파괴

## 아이템 2 : 생성자에 매개변수가 많다면 빌더를 고려하라

- 기존 프로그래머들은 생성자 혹은 정적 팩터리 메서드에 션택적 매개변수가 많을 경우 점층적 생성자 패턴을 즐겨 사용하였다
- 하지만 점층적 생성자 패턴도 매개변수가 많아지면 클라이언트 코드를 작성하거나 읽기 어려워지는 문제가 있다
- 이것을 자바 빈즈에서는 매개변수가 없는 생성자를 통해 객체를 만든 후 각각 setter메서드를 활용해 값을 지정한다
- 자바 빈즈 역시 객체를 하나 만들려면 메서드를 여러개 호출해야 하고, 객체가 완전히 생성되기 전에는 일관성이 무너진 상태에 놓이게 된다는 단점이 있다
- 일관성이 무너지는 문제 때문에, 자바 빈즈 패턴에서는 클래스를 불변으로 만들 수 없다
- 이러한 점층적 생성자와 자바 빈즈의 단점을 보완하여 빌더를 사용하면 된다. 빌더의 예시는 다음과 같다

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        //필수 매개변수
        private final int servingSize;
        private final int servings;

        //선택 매개변수 - 기본값으로 초기화
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calrories(int val) { calories = val; return this }
        public Builder fat(int val) { fat = val; return this }
        public Builder sodium(int val) { sodium = val; return this }
        public Builder carbohydrate(int val) { carbohydrate = val; return this }

        public NutritionFacts(Builder builder) {
            return new NutritionFacts(this);
        }

        private NutritionFacts(Builder builder) {
            servingSize = builder.servingSize;
            servings = builder.servings;
            calories = builder.calories;
            fat = builder.fat;
            sodium = builder.sodium;
            carbohydrate = builder.carbohydrate;
        }
    }
}

NutritionFacts cocaCola = new NutritionFacts.Builder(240,8).calories(100).sodium(35).carbohydrate(27).build();
```

- NutritionFacts 클래스는 불변이며, 모든 매개변수의 기본값들을 한곳에 모아두었다
- 빌더의 setter 메서드들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출 할 수 있다. 이렇게 흐르듯 연결되는 메서드 호출은 플루언트API 혹은 메서드 연쇄라 부른다
- 이러한 코드는 쓰기 쉽고, 무엇보다도 읽기 쉽다
- 잘못된 매개변수를 최대한 일찍 발견 하려면 빌더의 생성자와 메서드에서 입력 매개변수를 검사하고, builc 메서드가 호출하는 생성자에서 여러 매개변수에 걸친 불변식을 검사하면 된다
- 이러한 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다. 각 계층의 클래스에 관련 빌더를 멤버로 정의하고, 추상 클래스는 추상 빌더를 구체 클래스는 구체 빌더를 갖게 하면 된다

```java
public abstract class Pizza {
    public enum Topping { HAN, MUSHROOM, ONION, PEPER, SAUSAGE }
    final Set<Topping> topping;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNoNull(topping));
            return self();
        }

        abstract Pizza build();

        //하위 클래스는 이 메서드를 재정의 하여 this를 반환 하도록 해야 한다
        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone();
    }
}
```

- Pizza.Builder 클래스는 재귀적 타입 한정을 사용하는 제너릭 타입으로, 여기에 추상 메서드인 self를 더해 하위 클래스에서는 형변환 하지 않고도 메서드 연쇄를 지원 할 수 있다

```java
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNoNull(size);
        }

        @Override
        public NyPizza build() {
            return new NyPizza(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}
```

- 각 하위 클래스의 빌더가 정의한 build메서드는 해당하는 구체 하위 클래스를 반환 하도록 선언한다
- 하위 클래스의 메서드가 상위클래스의 메서드가 정의한 반환타입이 아닌, 그 하위 타입을 반환하는 기능을 공변 반환 타이핑이라 한다. 이 기능을 이용하면 클라이언트가 형변환에 신경쓰지 않고 빌더를 사용 할 수 있다
- 빌더 패턴에 장점만 있는것은 아니다. 객체를 만들려면 그에 앞서 빌더 부터 만들어야 하며, 빌더 생성비용이 크지는 않지만 성능에 민감한 상황에는 문제가 될 수 있다
- 또한 코드가 장황해지기 떄문에, 매개변수가 4개 이상은 되어야 값어치를 하지만, API는 시간이 지날수록 매개변수가 늘어나는 경향이 있기 떄문에 미리 빌더로 만드는게 나을 수 도 있다
- 생성자나 정적 팩터리가 처리해야할 매개변수가 많다면, 빌더 패턴을 선택하는게 낫다