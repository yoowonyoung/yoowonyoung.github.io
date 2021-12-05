---
layout: post
title: "Elegant Object - 1장: 출생"
description: Elegant Object 1장
date: 2021-10-31 22:45:00 +09:00
categories: ElegantObject Study
---


# 출생
- 객체는 자신의 가시성 범위안에서 살아간다

```java
if(price < 100) {
    Cash extra = new Cash(5);
    price.add(extra);
}
```

- 위의 예시에서 if블록 안에서만 extra라는 객체를 볼 수 있기 때문에, if블록 내부가 extra 객체의 가시성 범위

## -er로 끝나는 이름을 사용하지 마라
- 클래스는 객체의 팩토리로써, 객체를 생성하는데 이를 일반적으로 클래스가 객체를 인스턴스화 한다고 표현
- Java에서 제공하는 new 연산자로 할수있는 유일한 작업은 객체라고 불리는 클래스의 인스턴스를 생성하는 것 뿐. 유사한 객체가 이미 존재하거나 재사용 가능한지 확인하지도 않으며, new가 동작하는 방식을 변경할 수 있는 어떠한 매개변수도 제공하지 않음
- 널리 알려진 팩토리 패턴은 new 연산자를 대신해서 사용할 수 있는 더 강력한 옵션이지만, 개념적으로 팩토리 패턴과 new 연산자는 동일
- Java에서 팩토리 패턴은 new 연산자를 확장한것처럼 동작. new 연산자가 실행되기 전에 부가적인 로직을 더할 수 있기 때문에, new 연산자를 보다 유연하고 강력하게 만들 수 있음

```java
class Shapes {
    public Shapes make(String name) {
        if(name.equals("cicle")) {
            return new Cicle();
        }
        if(name.equals("rectangle")) {
            return new Rectangle();
        }
        throw new IllegalArgumentException("not found");
    }
}
```

- 팩토리에서도 객체를 생성하는 최종 단계에서는 여전히 new 연산자를 사용함으로써, 개념상 팩토리 패턴과 new 연산자는 크게 다르지 않음
- 종종 클래스를 객체의 템플릿으로 설명하는데, 이는 완전히 잘못된 표현으로 클래스는 객체의 팩토리이며 클래스를 객체의 능동적 관리자로 생각 해야함. 클래스는 객체를 꺼내거나 반환할 수 있는 위치이기에 클래스를 저장소 혹은 웨어하우스로 부르는게 맞음
- 이제부터 본론. 클래스 이름을 짓는 적절한 방법에 관한 이야기. 클래스 이름을 짓는 방법엔 두가지가 있는데, 올바른 방법과 잘못된 방법
- 잘못된 방법으로는 클래스의 객체들이 무엇을 하고 있는지를 살펴본후 기능에 기반해서 이름을 짓는 방법

```java
class CashFormatter {
    private int dollars;
    CashFormatter(int dlr) {
        this.dollars = dlr;
    }
    public String format() {
        return String.format("$ %d", this.dollars);
    }
}
```

- CashFormatter 클래스의 객체는 dollars에 저장된 금앨을 문자열로 포맷팅하는 일을 수행. 그래서 이 객체의 이름을 formatter로 짓기로 결정한것은 적절해보임
- 하지만 클래스의 이름은 무엇을 하는지가 아니라, 무엇인지에 기반 해야 함. CashFormatter라는 이름은 Cash, USDCash, CashInUSD등과 같이 바꾸어야 하며, ```format()```은 ```usd()```로 수정해야함

```java
class Cash {
    private int dollars;
    Cash(int dlr) {
        this.dollars = dlr;
    }
    public String usd() {
        return String.format("$ %d", this.dollars);
    }
}
```

- 다시말해 객체는 그의 역량으로 특정지어져야 함. 할 수 있는일로 설명 해야 한다는 말. 여기에 숨겨져있는 악마가 접미사 '-er'
- '-er'로 끝나는 이름을 가진 수많은 클래스들이 존재하는데(Manager, Controller, Helper, Handler, Writer, Reader, Validator - or도 역시 동일) 이들 모두 잘못 지어진 이름
- 상반된 방식으로 지어진 이름으로는 Target, EncodedText, DecodedData, Content 등이 있음
- 이 규칙에도 예외는 있지만 (computer, user 등) 이는 많지 않음
- 객체는 객체의 외부 세계와 내부 세계를 이어주는 연결장치가 아니라, 객체는 캡슐화된 데이터의 대표자
- 연결장치는 존중받지 못함. 정보를 수정하거나 스스로 어떤 일을 수행할 만큼 충분히 강력하지도 똑똑하지도 못하기 떄문에 단순히 정보만 전달함
- 대표자는 스스로 결정을 내리고 행동할 수 있는 자립적인 엔티티. 객체는 연결장치가 아니라 대표자여야 함
- 클래스의 이름이 '-er'로 끝난다면 이 클래스의 인스턴스는 실제로는 객체가 아니라 어떤 데이터를 다루는 절차들의 집합일 뿐임. 이는 절차적인 사고방식
- 올바른 클래스 이름을 짓기 위해서는 클래스의 객체들이 무엇을 캡슐화 할것인지를 관찰하고 이 요소들에 붙일 적합한 이름을 찾아야함
- 임의의 숫자 리스트가 존재할 때, 이 리스트의 구성요소중 소수를 찾는 알고리즘을 만든다고 가정. 오직 소수만으로 구성된 리스트를 얻는것이 목적이라면 Primers, PrimeFinder, PrimeChooser가 아니라 PrimeNumbers여야함
- 객체가 이런 기능을하는 프로시져와 유사해 보인다고 하더라도, 객체는 프로시저의 집합처럼 행동해서는 안됨. 즉 PrimeNumbers 클래스가 숫자 리스트를 캡슐화 하고 있는 동안에는, 외부에서 직접 객체 내부에 포함된 리스트를 처리하거나 조회하도록 허용해서는 안됨
- PrimeNumbers가 "지금은 내가 바로 그 목록이야!"라고 이야기 하도록 해야 하며, 외부에서 목록을 이용해서 어떤 일을 해야한다면, 객체에게 그 일을 하도록 요청하고, 수신한 요청을 처리하기 위해 객체 스스로 무엇을 해야 할지를 결정 해야함
- PrimeNumbers는 숫자의 리스트이며, 리스트를 처리하기 위한 메서드의 집합이 아님. 리스트 그 자체
- 요약하자면 새로운 클래스에 이름을 붙일 때에는 무엇을 하는지가 아니라 무엇인지를 생각 해야함

## 생성자 하나를 주 생성자로 만들어라
- 생성자는 새로운 객체에 대한 진입점. 생성자는 몇개의 인자들을 전달받아 어떤 일을 수행한 후, 임무를 수행할 수 있도록 객체를 준비 시킴

```java
class Cash {
    private int dollars;
    Cash(int dlr) {
        this.dollars = dlr;
    }
}
```

- 위의 예시에는 하나의 생성자가 존재하고, 이 생성자가 수행하는 하나의 작업은 인자로 전달된 달러를 dollars라는 이름의 private 정수 프로퍼티에 캡슐화 하는 일
- 이 책에서 권장하는 방식을 따르게 될 경우, 클래스는 많은 수의 생성자와 적은 수의 메서드를 포함할것
- 생성자의 개수가 더 많을수록 클래스는 더 개선되고, 사용자 입장에서 클래스를 더 편하게 사용할 수 있음

```java
new Cash(30);
new Cash("$29.95");
new Cash(29.95d);
new Cash(29.95f);
new Cash(29.95,"USD");
```

- 생성자가 다양할 경우, 위와 같이 다양한 방법으로 Cash 인스턴스를 생성할 수 있음
- 위의 예제는 모두 동일한 객체를 생성하는데, 이때 동일하다는 말은 생성된 객체들이 동일하게 행동 한다는 뜻
- 생성자가 많아질수록 클라이언트가 클래스를 더 유연하게 사용할 수 있게 되지만, 메서드가 많아진다면 클래스를 사용하기는 더 어려워짐. 클래스의 초점이 흐려지고 단일 책임의 원칙을 위반하기 떄문
- Cash 클래스의 사용자는 텍스트로 표현된 숫자를 다룰 경우에도 별도의 변환이나 파싱을 할 필요 없기 때문에 유연성을 얻을 수 있음. 이러한 유연성 덕분에 코드를 더 적게 작성해도 되고, 중복 코드 역시 줄어듬
- 생성자의 주된 작업은 제공된 인자를 사용해서 캡슐화 하고있는 주 프로퍼티를 초기화 하는일. 이런 초기화 로직을 단 하나의 생성자에만 위치 시키고 주 생성자라고 부르기를 권장하며, 다른 부 생성자들이 주 생성자를 호출하도록 만들기를 권장

```java
class Cash {
    private int dollars;
    Cash(float dlr) {
        this((int) dlr);
    }
    Cash(String dlr) {
        this(Cash.parse(dlr));
    }
    Cash(int dlr) {
        this.dollars = dlr;
    }
}
```

- 주 생성자를 모든 부 생성자 뒤에 위치시키는데, 이는 유지보수에 좋음. 가장 마지막에 정의된 생성자로 곧장 스크롤을 내리면 그곳에 항상 주 생성자가 있다면 유지보수가 더 편함
- 하나의 주 생성자와 다수의 부 생성자 원칙의 핵심은, 이 원칙이 중복 코드를 방지하고 설계를 더 간결하게 만들기 때문에 유지보수성이 향상된다는것

```java
class Cash {
    private int dollars;
    Cash(float dlr) {
        this.dollars = (int)dlr;
    }
    Cash(String dlr) {
        this.dollars = Cash.parse(dlr);
    }
    Cash(int dlr) {
        this.dollars = dlr;
    }
}
```

- 위의 잘못된 예시에서 dollars의 값이 항상 양수여야 한다고 가정하면, 이를 보장하기 위해서는 서로 다른 세곳의 생성자 안에서 일일히 유효성 검사 로직을 작성해야함. 원칙을 따랏더라면 한곳의 생성자에서만 유효성 검사 로직을 추가하면 됨

## 생성자에 코드를 넣지 마라
- 주 생성자를 통해 필요한 모든 인자들을 전달받는 클래스가 하나 있다고 가정
- 인자들은 새로운 객체의 상태를 초기화 하는데 필요한 충분한 정보를 담고 있고, 주 생성자는 객체 초기화 프로세르를 시작하는 유일한 장소이기 때문에 제공되는 인자들은 완전해야함. 즉 어떤것도 누락하지 않고 중복된 정보도 없어야함
- 이때 이러한 인자들을 이용해서 할 수 있는일은 무엇이고, 할 수 없는 일은 무엇인가. 즉, 인자들을 어떻게 다뤄야만 하는가에 대한 고민
- 경험적으로는 인자에 손대지 말아야함

```java
class Cash {
    private int dollars;
    Cash(String dlr) {
        this.dollars = Integer.parseInt(dlr);
    }
}
```

- 위의 예시에서는 객체를 초기화 하는 동안 생성자에 전달된 인자에 접근하고있음. 생성자에 선언된 인자의 타입은 문자열이지만, 내부에 캡슐화 하고있는것이 정수형이라 인자로 전달된 문자열을 정수로 변환할 필요가 있는데 예제에서는 이 작업을 생성자에서 처리하고있음. 이것이 잘못된 방법
- 객체 초기화에는 코드가 없어야 하고 인자를 건드려서는 안됨. 대신, 필요하다면 인자들을 다른 타입의 객체로 감싸거나 가공하지 않은 형식으로 캡슐화 해야함

```java
class Cash {
    private Number dollars;
    Cash(String dlr) {
        this(new StringAsInteger(dlr));
    }
    Cash(Number dlr) {
        this.dollars = dlr;
    }
}

class StringAsInteger implements Number {
    private String source;
    StringAsInteger(String src) {
        this.source = src;
    }
    int intValue() {
        return Integer.parseInt(this.source);
    }
}
```

- 위의 예시에서는 실제로 사용하는 지점까지 객체의 변환 작업을 연기
- 표면적으로 두 Cash클래스로부터 인스턴스를 생성하는 과정은 동일 해보이지만, 첫번째 예제의 객체는 숫자를 캡슐화 하지만 두번째 예제의 객체는 Number처럼 보이는 StringAsInteger인스턴스를 캡슐화 한다는 차이
- 진정한 객체지향에서 인스턴스화란 더 작은 객체들을 조합해서 더 큰 객체를 만드는것. 객체들을 조합해야하는 단 하나의 이유는 새로운 계약을 준수하는 엔티티가 필요하기 때문
- 위의 예시에서 텍스트 객체 "5"를 직접 사용할 수 없던 이유는, 텍스트 타입의 객체가 필요한 메서드를 제공하기 때문. 그래서 다른 타입의 새로운 객체를 생성 해야 했음
- 객체를 인스턴스화 하는 일과, 객체가 우리를 위해 작업을 하게 만드는것은 서로 겹처서는 안됨. 즉, 생성자는 어떤 일을 수행하는곳이 아니기 때문에 생성자 안에서 어떤 작업을 하도록 요청해서는 안됨. 이래서 생성자는 코드가 없어야 하고 오직 할당문만 있어야 하는것
- 이렇게 하는데에는 순수한 기술적인 이유가 몇가지 있는데, 첫번째 이유가 생성자에 코드가 없을 경우 성능 최적화가 더 쉽기 때문
- 위의 예시에서 ```intValue()```를 호출 할때마다 매번 텍스트를 정수로 파싱하기에, 생성자에 파싱 로직이 있는경우보다 빠르다는것에 의아해할것
- 여기서 요점은, 생성자에서 직접 하싱을 하는 경우 최적화가 불가능 하다는 것. 객체를 생성할 때마다 매번 파싱이 수행 되기 때문에 실행 여부를 제어할 수 없음. ```intValue()```를 호출할 필요가 없는 경우에도 CPU는 파싱을 위해 시간을 소모
- 인자를 전달된 상태 그대로 캡슐화 하고, 나중에 요청이 있을때 파싱을 하도록 하면 클래스의 사용자들이 파싱 지점을 자유롭게 결정 할 수 있음
- 파싱이 여러번 수행되고 싶지 않도록 한다면 데코레이터를 추가해서 최초의 파싱 결과를 캐싱하는것도 가능

```java
class CachedNumber implements Number {
    private Number origin;
    private Collection<Integer> cached = new ArrayList<>(1); // Null을 피하기 위해 ArrayList를 사용. 원시적인 캐싱
    public CachedNumber(Number num) {
        this.origin = num;
    }
    public int intValue() {
        if(this.cached.isEmpty()) {
            this.cached.add(this.origin.intValue());
        }
        return this.cached.get(0);
    }
}
```

- 이러한 해결방법이 지닌 아름다움은 제어하기 쉽고 투명하다는 점
- 객체를 인스턴스화 하는 동안에는 객체를 만드는일 이외에는 어떠한 일도 수행하지 않음
- 실제 작업은 객체의 메서드가 수행하며, 우리가 직접 이 과정을 제어 할 수 있음. 객제가 동작하는 동안 최적화도 할 수 있음
- 따라서 생성자에서 코드를 없애면 사용자가 쉽게 제어할 수 있는 투명한 객체를 만들 수 있으며, 객체를 이해하고 재사용 하기도 쉬워짐. 객체는 요청 받을 때만 동작하고, 그 전에는 어떠한 일도 수행하지 않는 좋은 방식으로 게으름
- 분명 파싱이 단 한번만 일어나는 경우도 있겠지만, 일관성이라는 측면에서 반대. 이 클래스가 미래에 어떤 일이 일어날지, 다음 리팩토링 시점엔 얼마나 많은 변경이 더해질지 모르기 때문