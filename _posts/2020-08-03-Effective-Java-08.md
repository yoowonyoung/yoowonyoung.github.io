---
layout: post
title: "Effective Java - 아이템8: finalizer와 cleaner사용을 피하라"
description: finalizer와 cleaner사용을 피하라
date: 2020-08-03 21:49:00 +09:00
categories: EffectiveJava Study
---


# 객체의 생성과 파괴

## 아이템 8 : finalizer와 cleaner사용을 피하라

- 자바는 두가지 객체 소멸자를 제공 한다. 그중 finalizer는 예측 할수없고, 상황에따라 위험 할 수 있어 일반적으로 불필요 하다
- 또한 finalizer는 오동작, 낮은성능, 이식성의 문제가 있어 기본적으론 쓰지 말아야 한다
- 그래서 자바9에서는 finalizer를 대체할 cleaner를 제공 하지만, cleaner도 finalizer보단 덜 위험할 뿐 여전히 예측할 수 없고, 느리고, 일반적으로는 불필요 하다
- finalizer와 cleaner는 즉시 수행 된다는 보장이 없으므로, 제떄 실행 되어야 하는 작업은 절대 할 수 없다
- finalizer나 cleaner를 얼마나 신속히 수행 할 지는 전적으로 가비지 컬렉터 알고리즘에 달려 있으며, 클래스에 finalizer가 달려있으면 그 인스턴스의 자원 회수가 제멋대로 지연 될 수 있다
- finalizer와 cleaner는 수행 시점 뿐만 아니라 수행 여부조차 보장하지 않는다. 따라서 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존 해서는 안된다
- System.gc나 System.runFinalization도 실행될 가능성을 높여줄뿐 보장하지는 않는다
- 또한 finalizer에서 발생되는 에러는 무시되며, 처리할 작업이 남아있어도 그순간 종료된다
- finalizer와 cleaner는 심각한 성능 문제도 있고, finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수 있다
    * 생성자나 직렬화 과정에서 예외가 발생하면, 이 되다만 객체에서 악의적인 하위 클래스의 finailizer가 수행될 수 있다
    * 객체 생성을 막을려면 생성자에서 예외를 전지는 것만으로도 충분 하지만, finalizer가 있으면 그렇지만도 않은것이다
    * 이러한 공격을 막기 위해서는 아무일도 하지 않는 finalize 메소드를 만들고 final로 선언 해야 한다

- 종료해야 할 자 원을 담고있는 객체의 클래스에서는 finalizer나 cleaner를 쓰는 것 보다 AutoCloserable을 구현 해주는게 좋다
- finalizer나 cleaner는 자원의 소유자가 close메서드를 호출하지 않을것에 대한 안전망 역할을 할 수는 있다
    * 하지만 그럴만한 값어치가 있는지 심사숙고 해야한다. 자바 라이브러리의 일부클래스는 안전망 역할의 finalizer를 제공한다

- 또한 네이티브 피어(Native Peer)와 연결된 객체에서라면 사용해도 괜찮다
    * 네이티브 피어란 일반 자바 객체가 네이티브 베서드를 통해 기능을 위임한 네이티브 객체를 말한다
    * 네이티브 피어는 자바 객체가 아니니 가비지 컬렉터가 그 존재를 알지 못하기 떄문에, 자바 피어를 회수 할 떄, 네이티브 피어를 회수하지 못한다
    * 성능 저하를 감당 할 수 있고, 네이티브 피어가 심각한 자원을 들고 있지 않을때만 적절하다

- finalizer와 달리 cleaner는 사용하기 더 어렵다. 클래스의 public API에 나타나지 않기 떄문이다

```java
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    //청소가 필요한 자원. 절대 Room을 참조 해서는 안된다
    private static class State implements Runnable {
        int numJunkPiles;
        
        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        //close 메서드나 cleaner가 호출한다
        @Override public void run() {
            System.out.println("방청소");
            numJunkPiles = 0;
        }
    }

    //방의 상태 cleanable과 공유한다
    private final State state;

    // cleanable 객체, 수거 대상이 되면 동작
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this,state);
    }

    @Override public void close() {
        cleanable.clean();
    }
}
```

- static으로 선언된 중첩 클래스인 State는 cleaner가 수거할 자원들을 담고 있다(실제론 네이티브 피어를 가르키는 포인터가 있을것이다)
- run 메서드가 호출되는 상황은 둘 중 하나인데, Room이 Close메서드를 호출 하거나, cleaner가 State의 run 메서드를 호출 해 줄것이다
- State 인스턴스는 절대로 Room 인스턴스를 참조 해서는 안된다(순환 참조)
- Room의 cleaner는 단지 안전망으로 쓰이고, 클라이언트가 모든 Room 생성을 try-with-resource블록으로 감쌋다면 자동 청소는 전혀 필요하지 않다

```java
public class Audlt {
    public static void main(String[] args) {
        try(Room myRoom = new Room(7)) {
            System.out.println("안녕~");
        }
    }
}
```

- 위와 같은 코드가 잘 짜여진 클라이언트다