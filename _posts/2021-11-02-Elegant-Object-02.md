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

```java
class Records {
    private static final String EOL = "\r\n";

    void write(Writer out) {
        for(Record rec : this.all) {
            out.write(rec.toString());
            out.write(Records.EOL);
        }
    }
}
```

- static final 프로퍼티인 EOL은 private이고, Records 클래스의 내부에서만 사용된다. 이것은 완전히 올바른 상황이다

```java
class Rows {
    private static final String EOL = "\r\n";

    void print(PrintStream pnt) {
        for(Row row : this.fetch()) {
            pnt.printf("{ %s }%s",row, Rows.EOL);
        };
    }
}
```

- Rows의 로직은 Records의 로직과다르며 협력하는 객체 집합도 완전히 다르다. 두 클래스는 공통점이 전혀 없다

```java
public class Constants {
    public static final String EOL = "\r\n";
}
```

- 위와 같이 객체를 사용해서 중복 문제를 해결 할 수있다. 하지만 이렇게 해결하는것은 코드 중복이라는 하나의 문제를 해결하기 위해 결합도가 높아지는 문제와 응집도가 낮아진 문제가 발생해버렸다

- 결합도 증가
    - 두 클래스는 모두 같은 객체에 의존하고 있으며, 이 의존성은 하드코딩 되어 있다. 이 경우에 의존성을 쉽게 분리할 방법은 없다
    - Constants.EOL의 내용을 수정하면 어디에 어떻게 영향을 미칠지 모른다. Constant.EOL을 변경하는 입장에서는 이 값이 어떻게 사용되고 있는지 알 방법이 없기 때문이다
    - Constant.EOL 객체는 사용 방법과 관련된 어떤 정보도 제공하지 않은채 모든 곳에서 접근 가능한 전역 가시성 안에 방치되어 있다. 객체가 어떤 문맥 안에서 어떻게 사용되어야 하고, 이 객체의 변경으로 사용자가 어떤 영향을 받는지에서도 알 수 없다. 사용자들이 Constant.EOL에 결합된것이다

- 응집도 저하
    - 낮은 응집도는 객체가 자신의 문제를 해결 하는데 덜 집중한다는 의미
    - 객체들은 상수를 다루는 방법을 알고 있어야 하며, 아주 멍청한 상수 위에 자신만의 의미론을 덧붙여야 함
    - 상수는 자신의 존재 이유를 이해하지 못하는 텍스트 덩어리이기 때문
    - 의미를 추가하기 위해서 Records와 Rows 클래스 안에 더 많은 코드를 작성해야 하는데, 목적을 명확하게 만들어줄 코드를 추가해서 정적 상수를 감싸야함. 이런 코드는 Records와 Rows가 의도했던 원래 목적과는 동떨어져 있을 수 밖에 없음
    
- Records와 Rows의 목적은 한줄의 마지막을 처리하는게 아니라 레코드나 로우 자체를 처리하는것. 따라서 한 줄을 종료하는 작업을 다른 객체에 위임한다면 각 객체의 응집도를 향상 시킬 수 있음
- 객체사이의 데이터를 중복하는게 아닌 기능을 공유할 수 있도록 새로운 클래스를 만들어야 함

```java
class EOLString {
    private final String origin;
    EOLString(String src) {
        this.origin = src;
    }

    @Override
    String toString() {
        return String.format("%s\r\n",origin);
    }
}


class Records {
    void write(Writer out) {
        for(Record rec : this.all) {
            out.write(new EOLString(rec.toString()));
        }
    }
}
```

- 접미사를 줄 마지막에 정확하게 추가하는 방법에 관해서는 EOLString이 책임 질것. Records와 Rows는 더이상 해당 로직을 포함하지 않으며, EOLString이 그 작업을 책임진다는 사실만 알고 있음
- EOLString에 대한 결합은 계약을 통해 추가된 것이며, 계약을 통한 결합은 언제든지 분리가 가능 하기 때문에, 유지보수성이 저하되지 않음
- 만약 플랫폼에 의존하고있는 동작이 있어, windows에서 실행 될 경우, ```\r\n```을 추가하는것 대신 예외를 던져야 한다면 EOLString을 다음과 같이 바꾸면 됨

```java
class EOLString {
    private final Strin origin;
    EOLString(String src) {
        this.origin = src;
    }
    String toString() {
        if(/* 윈도우의 경우*/){
           throw new IllegalStateException("Window에서는 EOL이 불가능합니다");
        }
        return String.format("%s\r\n",origin);
    }
}
```

- public static 리터럴을 사용했던 이전 예제는 이런 변경이 불가능 함
- 퍼블릭 상수마다 계약의 의미를 캡슐화 하는 새로운 클래스를 만들어야 하며, 수백개의 단순한 상수 문자열 리터럴 대신 수백개의 마이크로 클래스를 만들어야 함
- 그렇다고 해서 중복코드를 가진 마이크로 클래스들에 의해 코드가 더 장황해지고 오염되지않음. 클래스사이에 중복 코드가 없다면 클래스가 작아질수록 코드는 더 깔끔해짐
- OOP에서는 퍼블릭 상수를 절대로 사용해서는 안된다. 변명의 여지가 없다. 현대적인 언어의 라이브러리에 퍼블릭 상수가 넘쳐난다는것은 불행한일이다
- Enum역시 동일하다 Enum과 퍼블릭 상수 사이에는 아무런 차이가 없기 때문에 이 또한 사용해서는 안된다


## 불변 객체로 만들어라
- 모든 클래스를 상태 변경이 불가능한 클래스(immutable class)로 만들면 유지보수성을 크게 향상 시킬 수 있다
- 불변성 역시 크기가 작고 응집력이 높으며 느슨하게 결합되고 유지보수하기 쉬은 클래스를 만들 수 있도록 한다
- 가변클래스보다는 불변클래스를 이해하기가 더 쉽다. 이해하기 쉬운 코드가 유지보수도 더 쉽다
- 인스턴스를 생성한 후에 상태를 변경 할 수 없는 객체를 불변 객체라고 부른다

```java
class Cash { //가변 객체 생성
    private int dollars;

    public void setDollars(int val) {
        this.dollars = val;
    }
}

class Cash { //불변 객체 생성
    private final int dollars;
    Cash(int val) {
        this.dollars = val;
    }
}
```

- 불변 객체는 필요한 모든 것을 내부에 캡슐화 하고 변경 할 수 없도록 통제
- 불변 객체를 수정해야 한다면 프로퍼티를 수정하는 대신 새로운 객체를 생성해야 함

```java
class Cash {
    private final int dollars;
    public Cash mul(int factor) {
        return new Cash(this.dollars * factor);
    }
}
```

- 불변 객체는 어떤 방식으로든 자기 자신을 수정할 수 없기 때문에, 항상 원하는 상태를 가지는 새로운 객체를 생성해서 반환해야 함

```java
//가변 객체
Cash five = new Cash(5);
five.mul(10);
System.out.println(five);

//불변 객체
Cash five = new Cash(5);
Cash fifty = five.mul(10);
System.out.println(fifty);
```

- 핵심은 절대로 변경 가능한 객체를 만들지 말라는것. 항상 불변 객체를 사용 해야함. 가변 객체는 객체 패러다임의 오용임
- 가변 객체의 예시를 보면 모든 사람들은 five 객체가 5달러처럼 행동 할것이라고 기대 하지만 예상과 달리 50달러로 행동 하고 있음
- 불변 클래스가 가변 클래스보다 더 좋고, 더 효과적이며, 일부 문제들을 더 우아하게 해결 할 수 있기때문에 더 자주 사용해야 한다는게 아니라, 가변 클래스를 만들지 말라는 이야기임
- 불변 객체로는 지연로딩을 할 수 없는데, 이는 언어 차원에서 제공 되어야 할 문제. 객체를 불변으로 유지 하면서도 지연 로딩을 구현 할 수 있는 다양한 해결 방법은 있음

### 식별자 가변성
- 불변 객체에는 식별자 가변성 문제가 없음
- 비슷한 두 객체를 비교한 후 객체의 상태를 변경 할 때, 두 객체는 이제 동일하지 않지만 우리는 여전히 두 객체가 동일하다고 생각 할 수 있음

```java
Map<Cash, String> map = new HashMap<>();
Cash five = new Cash("5");
Cash ten = new Cash("10");
map.put(five,"five");
map.put(ten,"ten");
five.mul(2);
System.out.println(map) // { 10 => "five", 10 => "ten" }
```

- five.mul(2)를 수행해서 five의 상태를 변경하면 map은 더이상 올바르지 않음. 처음에는 동일하지 않은 두 객체를 생성하고, 두 객체를 map에 추가해서 독립적인 엔트리가 생성 되었지만, mul을 호출해 상태가 5에서 10으로 바뀌었지만 map에게 알려주지 않아 map은 변경이 일어난 사실을 인지하지 못했음
- 이러한 문제가 식별자 가변성으로 매우 심각하고 찾기 어려운 버그로 이어질 가능성이 있음. 불변 객체를 사용하면 map에 추가한 이후 상태 변경이 불가능 하기 때문에 식별자 가변성 문제가 발생하지 않음

### 실패 원자성
- 실패 원자성이란 완전하고 견고한 상태의 객체를 가지거나 아니면 실패하거나 둘중 하나만 가능한 특성을 말함

```java
// 가변 
class Cash {
    private int dollars;
    private int cents;
    public void mul(int factor) {
        this.dollars *= factor;
        if(/*뭔가 잘못됨*/) {
            throw new RuntimeException();
        }
        this.cents *= factor;
    }
}

// 불변
class Cash {
    private final int dollars;
    private final int cents;
    public Cash mul(int factor) {
        if(/*뭔가 잘못됨*/) {
            throw new RuntimeException();
        }
        return new Cash(
            this.dollars * factor;
            this.cents * factor;
        )
    }
}
```

- 가변 객체의 경우 실행 도중 예외가 던져진다면 객체의 절반만 수정되고 나머지 절반은 원래 값을 유지한다. 이로 인해 매우 심각하고 발견하기 어려운 버그가 발생할 수 있다
- 불변 객체는 내부의 어떤 것도 수정 할 수 없기 때문에 이런 결함이 발생하지 않음. 대신 새로운 상태를 가진 새로운 객체를 인스턴스화
- 가변 객체를 사용하더라도 실패 원자성이라는 목표를 달성할수는 있지만, 이를 위해선 특별히 주의를 기울여야 하지만 불변 객체에서는 별도의 처리 없이도 원자성을 얻음

### 시간적 결합

```java
Cash price = new Cash();
price.setDollars(29);
price.setCents(95);
System.out.println(price); // 29.95
```

- 객체를 인스턴스화 하고 초기화 하는 일반적인 방법에 따라 짜여진 코드
- 위 4줄의 코드는 특정한 순서로 정렬되어 있다. 시간적인 순서에 따라 서로 결합되어 있다

```java
Cash price = new Cash();
price.setDollars(29);
System.out.println(price); // 29
price.setCents(95);
```

- 위와 같이 코드가 재정렬 되더라도 컴파일은 여전히 성공한다. 코드를 재정렬 하기로 마음 먹었다면 코드 줄 사이의 시간적인 결합을 이해해야 하는 것이다
- 이 상황에서 컴파일러는 아무 도움이 되지 않으며, 이런 시간적 결합을 이해하고 기억하는것은 전적으로 프로그래머의 몫이 되기 때문에 유지보수에 어려움이 따를 것

```java
Cash price = new Cash(29,95);
System.out.println(price);
```

- 하나의 문장 만으로도 객체를 인스턴스화 할 수 있고, 이 경우에는 인스턴스화와 초기화가 분리되지 않고 항상 함께 실행되기 때문에 시간적인 결합이 제거됨

### Side Effect 제거
- 객체가 가변일때는 누구나 손쉽게 객체를 수정할 수 있음
- 클래스가 불변이라면 누구도 객체를 수정할 수 없기 때문에 객체의 상태가 변하지 않았다고 확신 할 수 있음. 코드가 제대로 동작하지 않는 경우에도 Side Effect가 발생한 위치를 찾을 필요가 없음

### Null 참조 제거

```java
class User {
    private final int id;
    private String name = null;
    public User(int num) {
        this.id = num;
    }
    public void setName(String txt) {
        this.name = txt;
    }
}
```

- 이 클래스의 인스턴스가 생성될 때 name 프로퍼티의 초기값으로 null이 할당 됨. 나중에 setName메서드를 호출할 때 초기화 되며 그 전까지는 값이 null인 상태를 유지
- name에 접근하기 전에 값이 null인지 아닌지 확인해야 하기 때문에 코드가 ```if name != null```이라는 문장으로 가득할것이며, 잘못해서 null 체크를 잊는다면 NPE가 발생할 가능성이 있음
- 실제 값이 아닌 null을 참조하는 객체는 유지보수성도 떨어지는데, 언제 객체가 유효한 상태이고 언제 객체가 아닌 다른 형태로 바뀌는지를 이해하기 어렵기 때문
- 다른 클래스가 필요하지만 새로운 클래스를 생성하는 작업이 귀찮게 여겨질 때 이런 문제가 발생 하거나, 그 클래스를 어떻게 만들어야 할지 몰랏거나, OOP에서 클래스가 뭘 의미하는지를 몰랏거나 원인은 다양하지만 결과는 매우 동일. 매우 큰 클래스가 되는것
- 커다란 클래스를 만드는 이유는 우리가 문제를 더 작은 부분으로 분해하기 위해 상속과 캡슐화를 어떻게 사용해야 하는지 모르기 때문
- 불변 객체를 만들면 객체 안에 null을 포함 시키는 것이 애초에 불가능 해짐. 다시 말해 작고, 견고하고, 응집도 높은 객체를 생성할 수 밖에 없도록 강제되기 때문에 유지보수하기 더 쉬움

### 스레드 안정성
- 스레드 안정성이란 객체가 여러 스레드에서 동시에 사용 될 수 있으며, 그 결과를 항상 예측 가능하도록 유지할 수 있는 객체의 품질
- 병렬 스레드 환경에서 가변 객체 내부의 상태를 변경하다 발생하는 병렬성 이슈는 디버깅과 해결이 매우 어려운데, 재현이 매우 어렵거나 혹은 아예 불가능 하기 떄문
- 불변 객체는 실행 시점에 상태를 수정할 수 없게 금지함으로써 이 문제를 해결 하는데, 어떤 스레드도 객체의 상태를 수정할 수 없기 때문에 아무리 많은 스레드가 객체에 접근해도 문제가 없음
- synchronized 키워드를 이용해서 명시적으로 동기화를 함으로써 가변 클래스 역시 스레드 안정성을 확보 할수는 있지만, 이는 생각보다 쉽지 않으며, 성능상의 비용이 초래되고, 배타락/데드락의 문제가 발생할 가능성이 있음

### 작고 더 단순한 객체
- 객체가 더 단순해질수록 응집도는 더 높아지고 유지보수는 더 쉬워짐
- 소프트웨어가 복잡해질수록 소프트웨어를 만들어내는 프로그래머의 품질은 더 낮아짐. 최고의 소프트웨어는 단순하고, 이해하기 쉽고, 수정하기 쉽고, 문서화하기 쉽고, 지원하기 쉽고, 리팩토링 하기가 쉬움
- 대부분의 경우 단순하다는 것은 더 적은 코드 줄 수를 의미 하는데, 클래스가 짧을수록 하는 일이 무엇이고, 어디에서 실패하고, 어떻게 리팩토링 해야 하는지를 더 쉽게 이해 할 수있음
- Java 기준으로 클래스의 최대 크기는 공백을 포함해 250줄 정도라고 생각
- 클래스가 작다면 정확한 줄 수는 중요하지 않음. 애플리케이션에 포함되어있는 모든 클래스의 길이를 250줄 이하로 유지할 수 만 있다면 좋은 소프트웨어 개발자이자 아키텍트라고 생각해도 무방
- 불변 객체를 아주 크게 만드는일은 불가능 하기 때문에, 일반적으로 불변객체는 가변 객체보다 더 작음
- 불변 객체가 작은 이유는 생성자 안에서만 초기화 할 수 있기 때문. 인자를 10개씩 받는 생성자를 만들려는 사람은 없을것
- 불변성은 클래스를 더 깔끔하고 짧게 만듬. 이것이 불변 클래스를 이용할 때 얻을 수 있는 가장 중요한 장점

## 문서를 작성하는 대신 테스트를 만들어라
- 문서화는 유지보수에 있어 중요한 구성요소 이지만, 문서를 만드는일이 중요한게 아니라 클래스나 메서드에 관한 추가 정보가 얼마나 쉽게 접근할 수 있는지가 중요
- 더 읽기 쉬운 코드를 만들기 위해서는 코드를 읽게 될 사람이, 비즈니스 도메인, 프로그래밍 언어, 디자인 패턴, 알고리즘을 거의 이해하지 못하는 주니어 프로그래머라고 가정해야 함. 나쁜 프로그래머가 복잡한 코드를 짜고, 좋은 프로그래머는 단순한 코드를 짬
- 이상적인 코드는 스스로를 설명하기 때문에 어떤 추가 문서도 필요하지 않음. 나쁜 설계는 문서를 작성하도록 강요
- 좋은 클래스는 목적이 명확하고, 작고, 설계가 우아함. 코드를 문서화 하는대신 코드를 깔끔하게 만들어야함
- 깔끔하게 만든다는 말에는 단위 테스트도 함께 만든다는 말이 포함되어 있는데, 단위 테스트 역시 클래스의 일부로 봐야함
- 개념적 측면에서 단위 테스트는 클래스의 일부이지 독립적인 개체가 아님. 깔끔하고 유지보수 가능한 단위 테스트를 추가 한다면, 클래스를 더 깔끔하게 만들 수 있고 유지보수성을 향상 시킬 수 있음
- 단위테스트를 작성 할수록 더 적은 문서화가 요구되는 이유가 단위테스트가 바로 문서화 이기 때문
- Java코드로 작성된 단위 테스트를 이해하기 위해서는 영어에 유창할 필요가 없지만, Javadoc을 이해하기 위해서는 어느정도의 영어 독해 능력이 요구됨
- 단위 테스트는 클래스의 사용 방법을 보여주는데 반해, 문서는 이해하고 해석하기 어려운 이야기들을 들려줌

## Mock 객체 대신 Fake를 사용 하라
```java
class Cash {
    private final Exchange exchange;
    private final int cents;
    public Cash(Exchange exch, int cnts) {
        this.exchange = exch;
        this.cents = cnts;
    }
    public Cash in(String currency) {
        return new Cash(
            this.exchange,
            this.cent * this.exchange.rate(
                "USD",currency
            )
        );
    }
}
```

- Cash 클래스는 Exchange 클래스에 의존하고 있어 Cash클래스를 사용하기 위해 Exchange의 인스턴스를 Cash의 생성자에 전달해야함

```java
Cash dollar = new Cash(new NYSE("secret"),100);
Cash euro = dollar.in("EUR");
```

- 이때 뉴욕 증권 거래소에 접속하기 위해 secret이라는 패스워드를 사용하고 있음. 단위 테스트를 실행 할 때마다 매번 NYSE 서버에 요청하고 싶지도 않고, secret이라는 패스워드가 모든 프로그래머에게 노출 되는 것도 원치 않음. 이때 전통적인 접근 방식은 모킹을 이용하는것

```java
Exchange exchange = Mockito.mock(Exchange.class);
Mockito.doReturn(1.15)
    .when(exchange)
    .rate("USD","EUR")
Cash dollar = new Cash(exchange, 500);
Cash euro = dollar.in("EUR");
assert "5.75".equals(euro.toString())
```

- 위와 같이 모킹을 활용 할 수 있지만, 모킹은 나쁜 프랙티스이며 최후의 수단으로만 사용해야함. 모킹 대신 페이크객체를 사용할것을 제안

```java
interface Exchange {
    float rate(String origin, String target);
    final class Fake implements Exchange {
        @Override
        float rate(String origin, String target) {
            return 1.2345;
        }
    }
}

//테스트 코드
Exchange exchange = new Exchange.Fake();
Cash dollar = new Cash(exchange,500);
Cash euro = dollar.in("EUR");
assert "6.17".equals(euro.toString());
```

- 위와 같이 중첩된 페이크 클래스는 인터페이스의 일부이며 인터페이스와 함께 제공되어 좀 더 짧은 단위 테스트를 만들 수 있음
- 단위 테스트가 덜 명확해졌다고 생각 할 수도 있음. 6.17이란 값이 어디서 나왔는지 의문이 들 수 있지만, 좀 더 강력한 페이크 클래스를 만들면 되는일. 경우에 따라 페이크 클래스가 실제 클래스보다 더 복잡할 때도 있으며, 실제 클래스와는 조금 다른 방식으로 구현할 수도 있음
- 클래스의 행동이 변경되면 단위 테스트가 실패 하기 떄문에 단위 테스트는 코드 리팩토링에 큰 도움이 됨(true positive), 동시에 행동이 변경되지 않은경우 실패해서는 안됨(false potitive). 모킹을 하게되는 경우에는 이러한 수정이 발생하면 테스트가 실패하게 됨
- 페이크 클래스의 경우에는 인터페이스가 변경되면 자연스럽게 페이크 클래스의 구현도 변경되기 때문에, 단위 테스트를 변경 할 필요도, 단위 테스트가 깨지지도 않는다
- 요점은 모킹이 나쁜 프랙티스라는 사실. 단위 테스트를 지원하기 위해 만들어졌지만, 실제로는 단위 테스트에 우호적이지 않으며, 모킹은 클래스 구현과 관련된 내부의 세부 사항을 테스트에 결합시킴
- 또한 페이크 클래스는 인터페이스의 설계에 관해 더 깊이 고민 하도록 해줌. 인터페이스를 설계 하면서 페이크 클래스를 만들다보면 인터페이스의 설계에 관해 더 깊이 고민하도록 해줌. 인터페이스의 사용자 관점에서 고민할 수 있게 되며, 테스트 리소스를 사용해서 사용자와 동일한 기능을 구현하게 해줌

## 인터페이스를 짧게 유지하고 스마트를 사용하라
- 클래스를 작게 만드는게 중요하기 때문에 인터페이스를 작게 만드는것은 훨씬 더 중요함. 클래스가 다수의 인터페이스를 구현하기 때문
- 인터페이스는 구현 클래스가 준수해야 하는 계약으로, 인터페이스가 커질경우 단일 책임의 원칙을 위반하는 클래스를 만들도록 부추김
- 이를 해결하기 위해서 또다른 인터페이스를 만드는것이 아닌 인터페이스 안에 스마트 클래스를 추가해서 해결 할 수 있음

```java
interface Exchange {
    float rate(String source, String target);
    final class Smart {
        private final Exchange origin;
        public float toUsd(String source) {
            return this.origin.rate(source,"USD");
        }
    }
}
```

- 이 스마트 클래스는 명확하고 공통적인 작업을 수행하는 많은 메서드를 포함 할 수 있음. Exchange 인터페이스가 어떻게 구현되고 환율이 어떻게 계산되는지는 모르지만, 인터페이스 위에 특별한 기능을 적용하며 이 기능은 Exchange의 서로 다른 구현 사이에 공유
- 스마트 클래스를 인터페이스와 같이 제공해야 하는 이유는 인터페이스를 구현하는 서로 다른 클래스 안에 동일한 기능을 반복해서 구현하고 싶지 않기 때문

```java
float rate = new Exchange.Smart(new NYSE()).toUsd("EUR");
```

- NYSE말고 모든 Exchange 구현체에 동시에 더 많은 기능을 추가 해야 한다고 하면, 이를 스마트 클래스에 추가 하는게 나음. 스마트 클래스의 크기는 점점 더 커지겟지만 Exchange 인터페이스는 작고, 높은 응집도를 유지 할 수 있음
- 인터페이스는 계약이므로 인터페이스가 커질수록 더 많은것들이 요구되고 더 많은 문제가 발생함. 클래스의 응집도와 견고함이 심각하게 손상될수도 있음