---
layout: post
title: "Effective Java - 아이템78: 공유중인 가변 데이터는 동기화해 사용하라"
description: 공유중인 가변 데이터는 동기화해 사용하라
date: 2020-09-20 20:17:00 +09:00
categories: EffectiveJava Study
---


# 동시성

## 아이템 78 : 공유중인 가변 데이터는 동기화해 사용하라

- synchronized키워드는 해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장한다
- 많은 프로그래머가 동기화를 배타적 실행, 즉 한 스레드가 변경하는중이라서 상태가 일관되지 않은 순간의 객체를 다른 스레드가 보지 못하게 막는 용도로만 생각한다
- 먼저 이 관점에서 이야기 하자면, 한 객체가 일관된 상태를 가지고 생성되고, 이 객체에 접근하는 메서드는 그 객체에 락을 건다. 락을건 메서드는 객체의 상태를 확인하고 필요하면 수정한다. 즉, 객체를 하나의 일관된 상태에서 다른 일관된 상태로 변화시킨다. 동기화를 제대로 사용하면 어떤 메서드도 이 객체의 상태가 일관되지 않은 순간을 볼 수 없을것이다
- 하지만 동기화에는 중요한 기능이 하나 더 있다. 동기화 없이는 한 스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수 있다. 동기화는 일관성이 깨진 상태를 볼 수없게 하는것은 물론, 동기화딘 메서드나 불록에 들어간 스레드가 같은 락으 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다
- 언어 명세상 long과 double을 제외한 변수를 읽고 쓰는 동작은 원자적이라, 여러 스레드가 같은 변수를 동기화 없이 수정하는 중이라도, 항상 어떤 스레드가 정상적으로 저장한 값을 온전히 읽어옴을 보장한다
- 이 말을 듣고 "성능을 높이려면 원자적 데이터를 읽고 쓸때는 동기화 하지 말아야 겟다"라고 생각하면 안된다. 스레드가 필드를 읽을때 항상 '수정이 완전히 반영된값'을 얻는다고 보장하지만, 한 스레드가 저장한값이 다른 스레드에 '보이는가'를 보장하지는 않는다
- 동기화는 배타적 실행 뿐만 아니라, 스레드 사이의 안정적인 통신에 꼭 필요하다. 이는 한 스레드가 만든 변화가 다른 스레드에게 언제 어떻게 보이는지를 규졍한 자바의 메모리 모델 때문이다
- 공유중인 가변 데이터를 비록 원자적으로 읽고 쓸 수 있더라도, 동기화에 실패하면 처참한 결과로 이어질 수 있다
- 다른 스레드를 멈추는 작업을 생각해보자(Thread.stop은 안전하지 않아 오래전에 deprecated되었으므로 사용하지 말자!). 다른 스레드를 멈추기 위해서, 첫번째 스레드는 자신의 boolean 필드를 폴링하면서 그 값이 true가 되면 멈추게 하고, 이 필드를 false로 초기화 한다. 다른 스레드에서 이 스레드를 멈추고자 할 때 true로 변경하는 식이다. boolean필드를 읽고 쓰는 작업은 원자적이라, 이런 필드에 접근할 때 동기화를 제거 하기도 한다

```java
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while(!stopRequested)
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

- 이 프로그램은 1초후에 종료 되리라 생각되지만 실제로 그렇지는 않다! 원인은 동기화에 있다. 동기화 하지 않으면 메인 스레드가 수정한 값을 백그라운드 스레드가 언제쯤에나 보게될지 보증 할 수 없다. 동기화가 빠지면 가상머신이 다음과 같은 최적화를 수행 해버릴수도 있다

```java
//기존 코드
while(!stopRequested)
    i++

//최적화된 코드
if(!stopRequested)
    while(true)
        i++;
```

- 이는 OpenJDK 서버 VM이 실제로 적용하는 끌어올리기(hoisting)라는 최적화 기법이다. 이 결과로 프로그램은 응답 불가 상태가 되어 더이상 진전이 없다
- stopRequested 필드를 동기화해 접근하면 이 문제를 해결 할 수 있다. 다음과 같이 바꾸면 기대한대로 1초 뒤에 종료된다

```java
public class StopThread {
    private static boolean stopRequested;

    private static synchronized void requesteStop() {
        stopRequested = true;
    }

    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args) throws InterrupedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while(!stopRequested())
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```

- 쓰기 메서드와 읽기 메서드 모두를 동기화 했음에 주목해야 한다. 쓰기 메서드만 동기화 해서는 충분하지 않다. 쓰기와 읽기 모두가 동기화 되지 않으면 동작을 보장하지 않는다. 어떤 기기에서는 둘 중 하나만 동기화해도 동작하는듯 보이지만, 겉모습에 속아서는 안된다. 사실 이 메서드는 단순해서 동기화 없이도 원자적으로 잘 동작한다. 앞서 이야기 했든 동기화는 배타적 수행과 스레드간의 통신이라는 두가지 기능을 수행하는데, 이 코드에서는 그중 통신 목적으로만 쓰이는것이다
- 반복문에서 매번 동기화 하는 비용이 크지는 않지만, 더 빠른 대안도 있다. stopRequested 필드를 volatile으로 선언하면 동기화를 생략 해도 된다. volatile한정자는 배타적 수행과는 관련이 없지만, 항상 가장 최근에 기록된 값을 읽게됨을 보장한다

```java
public class StopRequested {
    private static volatile boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while(!stopRequested)
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

- volatile은 주의해서 사용해야 한다. 예를들어 다음은 일렬번호를 생성할 의도로 작성한 메서드이다

```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```

- 이 메서드는 매번 고유한 값을 반환할 의도로 만들어졌다. 이 메서드의 상태는 nextSerialNumber라는 단 하나의 필드로 결정되는데, 원자적으로 접근 할 수 있고 어떤 값이든 허용한다. 따라서 굳이 동기화 하지 않더라도 불변식을 보호 할 수 있어보이지만, 이 역시 동기화 없이 올바로 동작하지 않는다
- 문제는 ++ 연산자이다. 이 연산자는 코드상으로는 하나지만, 실제로는 nextSerialNumber 필드에 두번 접근한다. 먼저 값을 읽고 그런 다음 증가한 새로운 값을 저장하는것이다. 만약 두번째 스레드가 이 두 접근사이를 비집고 들어와 값을 읽어가면 첫번째 스레드와 똑같은 값을 돌려받게 된다. 이러한 오류를 안전실패라고 한다
- generateSerialNumber 메서드에 synchronized 한정자를 붙이면 이 문제가 해결된다. 동시에 호출해도 서로 간섭하지 않으며 이전 호출이 변경한 값을 읽게 된다는뜻이다. 메서드에 synchronized를 붙였다면, nextSerialNumber 필드에서는 volatile을 제거해야 한다. 이 메서드를 더 견고하게 만들려면 int대신 long을 사용하거나 nextSerialNumber가 최댓값에 도달하면 예외를 던지게 하자
- 아이템59의 조언에 따라 java.util.concurrent.atomic 패키지의 AutomicLong을 사용해보자. 이 패키지에는 락 없이도 스레드 안전한 프로그래밍을 지원하는 클래스들이 담겨있다. volatile은 동기화의 두가지 효과중 통신쪽만 지원하지만, 이 패키디는 원자성까지 지원한다

```java
private static final AutomicLong nextSerialNum = new AutomicLong();

public static long generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
}
```

- 이번 아이템에서 언급한 묹들을 피하는 가장 좋은 방법은, 애초에 가변 데이터를 공유하지 않는것이다. 불변 데이터만 공유하거나 아무것도 공유하지 말자
- 가변 데이터는 단일 스레드에서만 쓰도록 하자. 이 정책을 받아들였다면 그 사실을 문서에 남겨 유지보수 과정에서도 정책이 계속 지켜지도록 하는게 중요하다
- 프레임워크와 라이브러리를 깊이 이해하는것도 중요하다. 이런 외부코드가 인지하지 못한 스레드를 수행하는 복병으로 작용 할 수 있기 때문이다
- 한 스레드가 데이터를 다 수정한 후 다른 스레드에 공유할 때는 해당 객체에서 공유하는 부분만 동기화 해도 된다. 그러면 그 객체를 다시 수정할 일이 생기기 전까지 다른 스레드들은 동기화 없이 자유롭개 값을 읽어갈 수 있다
    * 이런 객체를 사실상 불변이라고 하고, 다른 스레드에 이런 객체를 원하는 행위를 안전 발행 이라고 한다
    * 객체를 안전하게 발행하는 방법은 많다. 클래스 초기화 과정에서 객체를 정적 필드, volatile필드, final필드, 혹은 보통의 락을 통해 접근하는 필드에 저장해도 된다. 그리고 동시성 컬렉션에 저장하는 방법도 있다