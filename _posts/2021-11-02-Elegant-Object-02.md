---
layout: post
title: "Elegant Object - 2장: 학습"
description: Elegant Object 2장
date: 2021-11-02 22:38:00 +09:00
categories: ElegantObject Study
---


# 학습

## 가능하면 적게 캡슐화 하라
- 유지보수성을 생각하다면 4개 또는 그 이하의 객체를 캡슐화 할것을 권장. 더 많은 객체를 캡슐화 해야 한다면 클래스에 문제가 있는것. 예외는 없음

```java
class Cash {
    private Integer digits;
    private Integer cents;
    private Integer currency;
}
```

- Cash 클래스는 digit, cents, currency 3개의 객체를 캡슐화하는데, 이 객체들 전체를 가리켜 상태 또는 식별자라 부름
- 이러한 식별자가 같은 객체들은 서로 동일한것. 하지만 Java언어의 특성상 식별자가 같더라도 다른 객체로 보기 떄문에 ```equals```를 오버라이드 하고 ```==```대신에 ```equals```를 사용 해야함
- 객체의 식별자는 기본적으로 세계 안에서 객체가 위치하는 좌표인데, 4개 이상의 좌표는 직관에 위배됨

## 최소한 뭔가는 캡슐화 하라
- 어떠한것도 캡슐화 하지 않는 객체가 존재하기도 함

```java
class Year {
    int read() {
        return System.currentTimeMillies() 
            / (1000 * 60 * 60 * 24 * 30 * 12) - 1970; // 알고리즘 오류는 무시하라
    }
}
```

- 어떤것도 캡슐화 하지 않기 때문에 이 클래스의 객체들은 모두 동일하나, 이 설계 역시 잘못됨. 너무 많은 캡슐화도 바람직 하지 않지만 아무것도 캡슐화 하지 않는것도 마찬가지
- 프로퍼티가 없는 클래스는 정적 메서드와 유사한데, 정적 메서드가 없고 인스턴스 생성과 실행을 엄격히 분리하는 순수OOP 에서는 기술적으로 프로퍼티가 없는 클래스는 만들 수 없음
- 실행으로부터 인스턴스 생성을 고립시켜야 하는데, 이는 생성자에서만 new연산자를 허용한다는 말
- Year에서 ```read()```메서드는 System의 정적 메서드를 사용하는데, 순수OOP에서는 정적 메서드가 존재하지 않기 떄문에, 어떤 클래스의 인스턴스를 생성한 후 이 인스턴스를 통해 시스템 클럭을 얻어야함

```java
class Year {
    private Millis millis;
    Year(Millis msec) {
        this.millis = msec;
    }
    int read() {
        return this.millis.read() 
            / (1000 * 60 * 60 * 24 * 30 * 12) - 1970; // 알고리즘 오류는 무시하라
    }
}
```

- 객체가 무(無)와 비슷한 무언가가 아니라면, 다른 무언가를 캡슐화 해야함. 이런 특징을 가진 엔티티만이 아무것도 캡슐화 하지 않을 수 있는데, 오직 하나만 존재하고 생존이나 자신의 좌표를 표현하기 위해 다른 엔티티를 필요로 하지 않기 때문
- 어떤 일을 수행하는 객체라면 다른 객체들과 공존하면서 이를 사용해야함. 자기 자신을 식별 할 수 있도록 다른 객체들을 캡슐화 해야함
- 다른 관점에서 살펴 본다면, 캡슐화된 상태는 세계안에서 객체의 위치를 지정하는 고유한 식별자로, 객체가 어떤것도 캡슐화 하지 않는단 말은 객체 자신이 세계 전체가 된다는 말
- 최종적으로 Year 클래스를 순수OOP의 형태로 만들면 다음과 같을것. 캡슐화에 관한 이야기는 이정도에서 마무리

```java
class Year {
    private Number num;
    Year(final Millis msec) {
        this.num = new Min(
            new Div(
                msec,
                new Mul(1000,60,60,24,30,12)
            ),
            1970
        );
    }
    int read() {
        return this.num.intValue();
    }
}
```

## 항상 인터페이스를 사용하라
- 객체는 다른 객체와 의사소통을 하면서 다른 객체의 작업을 지원하고, 다른 객체들 역시 이 객체에게 도움을 줌. 즉 서로를 필요로 하기 때문에 결합됨
- 애플리케이션이 성장하기 시작하고 객체들의 수가 수십개를 넘어간다면 객체들 사이의 강한 결합도가 심각한 문제로 떠오르는데, 이는 유지보수성에 영향을 미치기 때문
- 애플리케이션 전체를 유지보수가 가능하게 만들기 위해서는 최선을 다해서 객체를 분리해야 함. 이를 가능하게 해주는 가장 훌륭한 도구가 바로 인터페이스

```java
interface Cash {
    Cash multiply(float factor);
}
```

- Cash는 인터페이스로써, 다른 객체와 의사소통하기 위해 따라햐 하는 계약임

```java
class DefaultCash implements Cash {
    private int dollars;
    DefaultCash(int dlr) {
        this.dollars = dlr;
    }

    @Override
    Cash multiply(float factor) {
        return new DefaultCash(this.dollars * factor);
    }
}

class Employee {
    private Cash salary;
}
```

- 위와 같이 객체가 계약을 준수 할 수 있도록 만들 수 있으며, 금액이 필요하다면 실제 구현 대신 계약에 의존하면 됨
- Employee 클래스는 Cash 인터페이스의 구현 방법에 아무런 관심이 없고, ```multiply()```메서드가 어떻게 동작하는지 관심도 없고 알수도 없음. 그래서 Employee 클래스와 DefaultCash 클래스 사이를 느슨하게 분리가 가능
- 추가적으로 클래스 안의 모든 퍼블릭 메서드가 인터페이스를 구현하도록 만드는것을 권장. 올바르게 설계된 클래스라면 최소한 하나의 인터페이스라도 구현하지 않는 퍼블릭 메서드를 포함해서는 안됨. 클래스의 사용자로 하여금 이 클래스에 강하게 결합되도록 조장하기 때문
- 좀 더 철학적으로 보자면, 클래스가 존재하는 이유는 다른 누군가가 클래스의 서비스를 필요로 하기 떄문인데, 이 서비스는 계약이자 인터페이스이기 때문에 어딘가에는 문서화 되어야 하며, 서비스 제공자들은 서로 경쟁하기 때문에 각가의 경쟁자는 서로 다른 경쟁자로 쉽게 대체 할수 있어야함. 이것이 느슨한 결합도


## 메서드 이름을 신중하게 선택하라
- 빌더의 이름은 명사로, 조정자의 이름은 동사로
- 빌더는 무언가를 만들고 새로운 객체를 반환하는 메서드, 항상 뭔가를 반환하며 이름은 항상 명사여야함

```java
//빌더 예시
int pow(int base, int power);
float speed();
Employee employee(int id);
String parsedCell(int x, int y);//parsed 라는 형용사를 이용해 명사인 cell을 꾸밈으로, 이 메서드가 어떤 형태로든 cell을 파싱하고 반환하리란걸 기대할수있음
``` 

- 객체로 추상화한 실세계 엔티티를 수정하는 메서드를 조정자라고 부르며, 조정자의 반환타입은 항상 void여야 하고, 이름은 항상 동사여야함

```java
void save(String content);
void put(String key, Float value);
void remove(Employee emp);
void quicklyPrint(int id); //quickly라는 부사를 이용해 print를 꾸밈. 중심요소는 print
```

- 빌더와 조정자에 자신만의 이름을 붙일 때에도, 원칙을 지켜야 함
- 개념적으로 빌더와 조정자 사이에는 어떤 메서드도 존재해서는 안됨. 뭔가를 조작한후에 반환하거나, 만드는 사이에 조작하면 안됨

```java
int save(String content);
boolean put(String key, Float value);
float speed(float val);
```

- 위의 예시에서 save메서드는 조정자이기 때문에 무언가를 저장(save)하지만 int값을 빌더처럼 반환하고 있으므로 잘못됨. void로 바꾸거나 byteSaved 등으로 바꿔야함
- put 역시 조정자처럼 동작하나 빌더처럼 boolean을 반환하기에 문제. 아무것도 반환하지 않게 수정한다면 전달된 key 값이 정상적으로 변경됬는지 알 수 없기 때문에, PutOperation 클래스를 만들고 조정자인 save와 성공/실패 여부를 반환하는 빌더인 success 메서드를 만들어야함
- getter/setter는 이러한 관점에서 보면 문제가 있음

### 빌더가 명사여야 하는 이유
- 어떤것을 반환하는 메서드의 이름을 동사로 짓는것은 잘못. 일반적으로 제과점에 들려서 "브라우니를 요리해주세요"라고 하지 한고 "브라우니 주세요" 라고 하기 때문
- 이러ㄴ 브라우니를 요리하는 방법은 내 관심사가 아님. 단지 객체에게 브라우니를 요구하고 객체들은 그 요구를 만족시켜줌

```java
class Bakery {
    Food cookBrownie();
}
```

- 위의 cookBrownie는 객체의 메서드가 아님. 프로시저임. Bakery를 자립적이고 자율적인 객체로 존중하고 있지 않기 때문
- 올바르게 지은 메서드 이름은 사용자들이 객체를 설계한 목적, 객체가 수행해야하는 임무, 객체의 존재 목적과 살아가는 의미를 더 잘 이해할 수 있도록 해줌
- 부적절하게 지은 메서드 이름은 객체의 전체적인 개념을 망가트리고 사용자들이 이 객체를 데이터 집합이나 프로시저의 모음으로 다루도록 종용함
- 이러한 이유로 메서드의 이름을 동사로 지을때에는 객체에게 무엇을할지를 알려주어야함. 무엇을 만들라고 요청하는것은 예의에 어긋남. 만들어야 할지만 요청하고 만드는 방법은 객체 스스로 결정하도록 해야함

```java
InputStream load(URL url) -> InputStream stream(URL url)
String read(File file) -> String content(File file)
int add(int x, int y) -> int sum(int x, int y)
```

- 여기서 add대신 sum으로 바꾼것에 주목해야함. 우리는 객체에게 x와 y를 더하라고 요청하지 않고 대신, 두 수의 합을 계산하고 새로운 객체를 반환해달라고 요청할 뿐임

### 조정자가 동사여야 하는 이유
- 객체는 실세계의 엔티티를 대표함. 실세계의 엔티티를 조작해야 하는 경우에는 다음과 같이가객체가 그 작업을 수행하도록 요청해야함

```java
class Pixel {
    void paint(Color color);
}
Pixel center = new Pixel(50,50);
center.paint(new Color("red"));
```

- 이 코드는 center객체에게 스크린상의 (50,50)좌표에 위치한 픽셀을 칠하도록 요청하는데, 이 과정에서 뭔가 만들어질것이라고는 기대하지 않음
- paint는 값을 반환하지 않는 동사임. 예시를 들자면 바텐더에게 음악을 틀어달라고 요청하는것과 같음. 처음부터 뭔가를 돌려받을것이라고 기대하고 이 행동을 하지 않음. 음악을 틀고 현재의 볼륨 상태를 말해주세요 하지 않기 때문
- 오직 빌더만이 값을 반환할수있고 명사이며, 객체가 뭔가를 조정해야한다면 이름은 동사이고 반환값이 없음
- 핵심적인 원칙만 준수한다면 규칙을 완화할수도 있음. Builder패턴을 사용하면서 with로 시작하는 메서드 이름을 사용하는것과 같이

### 빌더와 조정자 혼합하기

```java
class Document {
    int write(Inputstream content)
}
```

- write는 파일 내용을 저장하고 저장된 바이트 수를 반환
- 얼핏 보면 문제가 없어보이지만 앞서 말한 원칙들을 위반하고 있음
- write메서드 안에서 데이터를 쓰는 동시에 쓰여진 바이트의 수를 카운트 함. 하나의 메서드 안에서 너무 복잡한 일을 처리하고 있음. 메서드의 목적이 명확하지 않기에 깔끔하게 명사나 동사 둘중 하나로 이름을 지을수가 없음

```java
class Document {
    OutputPipe output();
}

class OutputPipe {
    void write(InputStream content);
    int bytes();
    long time();
}
```

- 코드에서 볼수있는것처럼 output 메서드는 빌더임. 이 메서드를 통해 문서에 내용을 쓸 준비를 하는 OutputPipe타입의 객체를 생성. 연산을 수행할 객체를 준비한것이므로 이제 OutputPipe 인스턴스에 write()를 호출하면 객체는 관련된 데이터를 모으기 시작할것

### Boolean값을 결과로 반환하는 경우
- Boolean 값을 반환하는 메서드는 규칙에 있어서 예외적인 경우. 값을 반환하기 때문에 빌더에 속하지만, 가독성 측면에서 형용사로 지어야함

```java
boolean empty();
boolean readable();
boolean negative();
```

- 접두사 is는 중복이기에 메서드의 이름에 포함시키지는 않지만, 메서드를 읽을때에는 일시적으로 앞에 붙여 자연스럽게 들리게 해야함
- boolean을 반환하는 메서드를 특별하게 다뤄야 하는 이유는 대부분의 언어들이 논리구성자를 특별한 방식으로 다루기 때문
- 문자열이 비어있는지 여부를 반환하는 emptiness 메서드가 있다고 할때 ```if(name.emptiness() == true)``` 이렇게 사용하지 않고 `==true` 부분을 생략해서 쓰는데, 그렇다면 형용사로 지어야 더 자연스럽게 읽히게 됨

## 퍼블릭 상수를 사용하지 마라
- 상수(constant)라고 불리는 public static final 프로퍼티는 객체 사이의 데이터를 공유하기 위해 사용하는 매우 유명한 매커니즘
- 역설적으로 객체들은 그 어떤것도 공유해서는 안됨. 대신 독립적이여야하고 닫혀있어야함. 상수를 이용한 공유 매커니즘은 캡슐화와 객체지향적인 사고 전체를 부정하는 일임
