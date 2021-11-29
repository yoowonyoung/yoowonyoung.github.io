---
layout: post
title: "Elegant Object - 4장: 은퇴"
description: Elegant Object 4장
date: 2021-11-29 20:40:00 +09:00
categories: ElegantObject Study
---


# 은퇴

## 절대 NULL을 반환하지 마라
- 객체가 Null을 반환하는 순간, 객체를 장애를 가진 존재로 취급해서는 안된다는 조언에 반대된다

```java
public String title() {
    if(/* 타이틀이 없는 경우 */) {
        return null;
    }
    return "Elegant Object";
}

String title = x.title();
print(title.length());
```

- 위 코드에서 ```title.length()```를 호출할 때마다 항상 NPE가 던져질지 모른다는 사실에 불안할 수 밖에 없음
- 문제는 예외가 아니라, 객체에 대한 신뢰가 무너졌다는것

```java
String title = x.title();
if(title == null) {
    print("Can't print");
    return;
}
print(title.length());
```

- 위와 같이 반환된 결과를 확인하는 코드는 객체지향 원칙에 위배되는 끔찍한 코드임
- 객체라는 사상에는 우리가 신뢰하는 엔티티라는 개념이 담겨있음. 객체는 우리의 의도에 관해 전혀 알지 못하는 데이터 조각이 아님
- 객체를 신뢰한다는 말은 객체가 자신의 행동을 전적으로 책임지고 우리가 어떤식으로든 간섭하지 않는다는 의미가 담겨있음
- Null이 반환됨으로 인해 반환값을 검사하는 방식은 애플리케이션에 대한 신뢰가 부족하다는 명백한 신호. 이 신뢰가 깨짐으로써 코드를 읽을때 이 메서드 호출이 신뢰할만한가 판단해야 하기 떄문에 유지보수성도 떨어짐
- 그럼에도 불구하고 JDK에서도 Null을 반환하는것을 여럿 볼 수 있는데, 이는 JDK 설계 시점에 빠르게 실패하기 원칙을 몰랏던것이 아닐까 함
- 소프트웨어의 견고성과 실패 탄력 회복성에 관련해서 상반되는 두가지 철학이 존재하는데, 빠르게 실패하기와 안전하게 실패하기 임
- 안전하게 실패하기는 버그, 입출력 문제, 메모리 오버플로우등이 발생한 상황에서도 소프트웨어가 계속 실행 될 수 있도록 최대한 많은 노력을 기울일것을 권장. 따라서 Exception을 던지는것 대신, Null을 반환함으로써 누군가 이 상황을 처리해줄것을 요구하는것
- 빠르게 실패하기는 문제가 발생하면 곧바로 실행을 중단하고 최대한 빨리 예외를 던짐으로써, 문제를 빠르게 수정하는것을 권장. 상황을 구조하는것이 아니라 실패를 분명하게 만드는것
- Null의 대안 으로는 메서드를 두개로 나누어 첫번째 메서드에서는 객체의 존재를 확인하고, 두번쨰 메서드에서 객체를 반환하는 방법(아무것도 찾지 못한경우 두번째 메서드는 예외를 던짐)과, Null을 반환하거나 예외를 던지는 대신 객체 컬렉션을 반환하는 방법이 있음
- Java8 이후의 Optional을 사용하는 방법도 있지만, 의미론적으로 부정확하기 때문에(실제로 반환하는게 그 객체가 아닌, 그 객체가 들어있을 수 도 있는 봉투를 반환하는 것이니깐) OOP와 대립한다고 생각하며 사용을 권하지 않음
- Null 객체 디자인 패턴도 방법이 될 수 있음. Null 객체는 일부 작업은 정상적으로 처리 하지만, 나머지 작업은 처리하지 않음. 이 방법은 제한된 상황에서만 사용 가능 하고, 반환된 객체의 타입을 동일하게 유지해야 하는것도 있음

## Checked Exception만 던져라
- Unchecked Exception을 사용하는것은 실수이며 모든 Exception은 Checked Exception이여야함. 다양한 Exception 타입을 만드는것도 좋지 않음

```java
public byte[] content(File file) throws IOException {
    byte[] array = new byte[1000];
    new FileInputStream(file).read(array);
    return array;
}
```

- 메소드의 시그니쳐가 ```throws IOException```으로 종료되는 사실에 주목 해야함. 이것은 무슨일이 있어도 content를 호출하는 쪽에서 IOException을 잡아야 한다는것을 의미. 호출하는 쪽에서는 content 메서드를 실행하는 중에 발생할 수 있는 문제를 완전히 책임지지 않고서는 메서드 호출도 할 수 없음
- 이처럼 IOException은 catch 구문을 사용해서 반드시 잡아야 하기 때문에, Checked Exception에 속하며 ```throws IOException```을 메서드 시그니처에 선언 하므로 항상 가시적이라는 Checked Exception의 특징을 보여줌
- Unchecked Exception은 무시할 수 있으며 예외를 잡지 않아도 무방함. 메서드 시그니처에서 Unchecked Exception이 던져진다는 사실에 관해서 일절 언급하지 않기 때문에 가시적이지 못함
- 사실 꼭 필요한 경우가 아니라면 예외를 잡지 않는게 좋음. 이상적인 설계에서는 애플리케이션의 각 진입점별로 오직 하나의 catch문만 존재 해야 함
- 예외를 잡을것이라면 항상 예외를 체이닝 해야함

```java
public int length(File file) throws Exception {
    try {
        return content(file).length();
    } catch (IOException e) {
        throw new Exception("길이를 계산할 수 없음",e);
    }
}
```

- 위와 같은 예외 체이닝은 훌륭한 프랙티스임. 원래의 문제를 새로운 문제로 대체 함으로써 문제가 발생했다는 사실을 무시하지도 않으며, 그 대신 원래의 문제를 새로운 문제로 감싸서 함께 상위로 던짐
- 예외 체이닝의 핵심은 문제를 발생 시켰던 낮은 수준의 군본 원인을 소프트웨어의 더 높은 수준으로 이동시켰다는것임. 그대신 주의 해야 할것이 절대로 문제를 발생시킨 근본 원인을 무시하고 새로운 예외를 만들어서 던지면 안된다는것임
- 예외 체이닝은 의미론적으로 문제와 관련된 문맥을 풍부하게 만들기 위해서 필요함. 근본 원인만 가지고는 정보를 전달하는데 충분하지 않음
- 또한 예외는 단 한번만 복구 해야함. 무조건 예외를 잡아서는 안된다는 주장은 옳지 않음. 어떤 소프트웨어에서든 복구에 적합한 몇개의 장소가 존재 하며, 그 이외의 장소에서는 예외를 잡아서 체이닝해서 다시 던지는게 좋음. 보통 복구에 적합한 장소는 최상위임
- 관점 지향 프로그래밍도 한가지 방법. AOP는 단순하면서도 강력한 프로그래밍 패러다임으로 OOP와 궁합이 좋음. 
- 단 한번만 복구 할것이라면 단 하나의 예외타입만으로도 충분함. 올바르게 체이닝 했다면 예외 타입을 알 필요가 없음. 어떤 예외라도 담을수 있는 예외 객체 하나만 있으면 됨


## final이거나 abstract 이거나
- 상속에 반대하는 가장 강력한 주장은 상속이 객체들의 관계를 너무 복잡하게 만든다는 것인데, 이 문제를 일으키는 주범은 상속 그 자체가 아닌 가상 메서드 임
- 직관적으로 생각하면 상속은 자식 클래스가 부모 클래스의 코드를 계승받는 하향식 프로세스인데, 메서드 오버라이딩은 부모가 자식의 코드에 접근하는것을 가능하게 하기 때문에 상식에 어긋나는것
- 클래스와 메서드를 final이나 abstract 둘 중 하나로만 제한 한다면 이러한 문제가 발생할 수 있는 가능성을 없앨 수 있음
- final 클래스는 사용자 관점에서는 블랙박스임. final 클래스는 상속들 통해 수정할 수 없으며, 불투명이고 독립적이며 자신이 어떻게 행동해야하는지 알고 있고 어떤 도움도 필요하지 않음
- 반면 abstract는 글래스박스이고 불완전함. 스스로 행동할 수 없기 때문에 누군가의 도움이 필요하며 일부 요소가 누락되어있음
- 이런 final/abstract 이외의 세번째 신분에 대해서 강력하게 반대하는것. 블랙박스와 글래스박스 둘중 어느쪽도 될 수 있기 때문에 매우 혼란스러울수밖에 없음. 클래스는 자신이 불투명하고 자립적이며 견고하다고 가정하지만, 누군가가 이 가정을 무시하고 가상메서드를 이용해 특정 요소를 교체하는것도 가능함
- 만약 이 원칙을 준수해서 모든 클래스를 final이나 abstract로 만든다면 상속을 사용할일이 거의 없어질텐데, 이때 상속이 적절한 경우는 클래스의 행동을 확장하지 않고 정제할때임. 대신 이런일을 예상해서 설계한 추상 클래스만을 개선 해야함

```java
abstract class Document {
    public abstract byte[] content();
    public final int length() {
        return this.content().length;
    }
}

final class DefaultDocument extends Document {
    @Override
    public byte[] content() {
        //디스크에서 내용을 로드
    }
}

final class EncryptedDocument extends Document {
    @Override
    public byte[] content() {
        //디스크에서 내용을 로드하고 암호화 한 후 반환
    }
}
```

- 디스크로부터 콘텐츠를 로드하는 방법을 알고있는 DefaultDocument를 이용해서 Document를 정제 하거나, 다른 방식으로 정제하는 EncryptedDocument의 예시
- 두 클래스 모두 length() 메서드가 자신들의 메서드를 사용하는 방법을 명확하게 알고있다는 가정하에 메서드를 개선하고 있음. 확장보다 개선이 훨씬 더 깔끔한 이유가 이것

## RAII을 사용하라
- 리소스 획득이 초기화(Resource Acuistion Is Initialization, RAII)라는 개념은 C++에 존재하는 매우 강력한 기법이지만, Java에서는 가비지 컬렉션을 사용하기 때문에 사라진 개념. Java에 Destructor가 존재하지 않는 이유도 동일
- Java에서도 RAII를 모방할 수 있지만 C++의 처리 방식이 훨씬 깔끔함

```C++
#include<stdio.h>
#include<string>

class Text {
    public:
        Text(const char* name) {
            this->f = fopen(name, "r");
        }
        ~Text() {
            fclose(this ->f);
        }
        const std::string& content() {
            //파일의 내용을 읽어서 반환
        }
    private:
        File* f;
};

int main() {
    Text t("test.txt");
    t->content();
}
```

- 먼저 Text() 생성자를 호출해서 Text 클래스의 객체 t를 생성하고, 파일을 읽기 위해 content() 메서드를 호출하고, 객체 t가 가시성의 범위를 벗어나면 파괴자인 ~Text()가 호출되고 파괴자는 파일을닫음
- 여기서의 요령은 객체가 살아있는 동안에만 리소스를 확보 하는것. 예제에서는 파괴자가 호출되기 전까지만 파일 핸들 f를 확보하고 있음. 이렇게 객체를 초기화 할떄 리소스를 확보하는 방식에서 RAII이 유래된것임
- Java는 가비지컬렉션이 이를 대신하기 때문에 자바에 파괴자가 없어, Java에서 비슷하게 하려면 try-with-resource 기법을 사용 해야함

```java

main() {
    try(Text t = new Text("test.txt")) {
        t.content();
    }
}
```

- try 블록이 끝날때 객체 t를 파괴하는 대신, 객체 t의 close() 메서드를 호출함. 이 방식을 사용하기 위해서는 Text 클래스가 Closable 인터페이스를 구현하도록만 하면 됨
- 파일, 스트림, 데이터베이스 커넥션등 실제 리소스를 사용하는곳에서 모두 RAII를 사용할것을 적극 추천. Java에서는 AutoCloseable을 사용하기 때문