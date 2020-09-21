---
layout: post
title: "Effective Java - 아이템81: wait와 notify보다는 동시성 유틸리티를 애용해라"
description: wait와 notify보다는 동시성 유틸리티를 애용해라
date: 2020-09-21 22:59:00 +09:00
categories: EffectiveJava Study
---


# 예외

## 아이템 81 : wait와 notify보다는 동시성 유틸리티를 애용해라

- 자바5에서 도입된 고수준의 동시성 유틸리티 덕분에, 이전이라면 wait와 notify로 하드코딩해야 했던 전형적인 일들을 쉽게 처리 할 수 있게되었다
    * wait와 notify는 올바르게 사용하기가 아주 까다로우니 고수준의 동시성 유틸리티를 활용하는게 좋다

- java.util.concurrent의 고수준 유틸리티는 세 범주로 나눌 수 있는데, 실행자 프레임워크, 동시성 컬렉션, 동기화 장치이며, 이번 아이템에서 살펴볼것이 동시성 컬렉션과 동기화 장치이다
- 동시성 컬렉션은 List, Queue, Map과 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션으로, 높은 동시성에 도달하기 위해 동기화를 각자의 내부에서 수행한다
    * 따라서 동시성 컬렉션에서의 동시성을 무력화 하는것은 불가능하며, 외부에서 락을 추가로 사용하면 오히려 성능에 저하가 온다

- 동시성 컬렉션에서 동시성을 무력화 하지 못하므로, 여러 메서드를 원자적으로 묶어 호출하는일 역시 불가능하다. 그래서 여러 기본 동작을 하나의 원자적 동작으로 묶는 '상태 의존적 수정' 메서드들이 추가되었다. 이 메서드들은 아주 유용해서 자바8에서는 일반 컬렉션 메서드에서도 디폴트로 추가되었다
- 예를들어 Map의 putIfAbsent(key, value)메서드는 주어진 키에 매핑된 값이 아직 없을때만 새 값을 집어넣고, 기존값이 있다면 그 값을 반환하고 없었다면 null을 반환 한다. 이 메서드 덕에 스레드 안전한 정규화 맵을 쉽게 구현할 수 있다

```java
private static final ConcurrentMap<String, String> map = new ConcurrentMap<>();

public static String intern(String s) {
    String prevoiusValue = map.putIfAbsent(s, s);
    return prevoiusValue == null ? s : prevoiusValue;
}
```

- 위의 코드는 String.intern의 동작을 흉내낸것으로, 아직 개선의 여지는 남아있다. ConcurrentMap은 get같은 검색 기능에 최적화 되어 있으므로, get을 먼저 호출하여 필요할때만 putIfAbsent를 호출하면 더 효과적이다

```java
public static String intern(String s) {
    String result = map.get(s);
    if(result == null) {
        result = map.putIfAbsent(s,s);
        if(result == null)
            result = s;
    }
    return result;
}
```

- ConcurrenrtHashMap은 동시성이 뛰어나며 속도도 무척 빠르다. 이처럼 동시성 컬렉션은 동기화한 컬렉션을 낡은 유산으로 만들어버렸다. 대표적인 예로 Collections.synchronizedMap보다는 ConcurrentHashMap을사용하는게 훨씬 좋다
- 컬렉션 인터페이스중 일부는 작업이 성공적으로 완료될 때 까지 기다리도록(즉, 차단되도록) 확장되었다. Queue를 확장한 BlockingQueue에 추가된 메서드중 take는 큐의 첫 원소를 꺼내지만, 큐가 비어있다면 새로운 원소가 추가될 때까지 기다린다
    * 이러한 특성덕에 BlockingQueue는 작업큐(생산자-소비자큐)로 쓰기에 적합하다
    * 작업큐는 하나 이상의 생산자 스레드가 작업을 큐에 추가하고, 하나 이상의 소비자 스레드가 큐에 있는 작업을 꺼내 처리하는 형태이다
    * ThreadPoolExecutor를 비롯한 대부분의 실행자 서비스 구현체에서 이 BlockungQueue를 사용한다

- 동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게하며, 서로 작업을 조율할 수 있게 해준다. 가장 자주 쓰이는 동기화 장치는 CountDownLatch와 Semaphore이다. CyclicBarrier와 Excahnger는 그보다 덜 쓰인다. 그리고 가장 강력한 동기화 장치는 Phaser이다
- 카운트다운 래치는 일회성 장벽으로, 하나 이상의 스레드가 또다른 하나 이상의 스레드 작업이 작업이 끝날때까지 기다리게 한다. CountDownLatch의 유일한 생성자는 int값을 받으며, 이 값이 래치의 countDown 메서드를 몇번 호출해야 대기중인 스레드들을 깨우는지를 결정한다
- 이 간단한 장치를 활용하면 유용한 기능들을 놀랍도록 쉽게 구현 할 수 있다
    * 예를들어 어떤 동작들을 동시에 시작해 모두 완료하기까지의 시간을 재는 간단한 프레임워크를 구축한다고 해보자
    * 이 프레임워크는 메서드 하나로 구성되며, 이 메서드는 동작들을 실행할 실행자와 동작을 몇개나 동시에 수행할 수 있는지를 뜻하는 동시성 수준을 매개변수로 받는다
    * 타이머 스레드가 시계를 시작하기전에 모든 작업자 스레드는 동작을 수행할 준비를 마친다
    * 마지막 작업자 스레드가 준비를 마치면 타이머 스레드가 '시작 방아쇠'를 당겨 작업자 스레드들이 일을 시작하게 된다
    * 마지막 작업자 스레드가 동작을 마치자마자 타이머 스레드는 시계를 멈춘다
    * 이상의 기능을 notify와 wait을 이용해 구현하려고 하면 매우 난해하고 지저분한 코드가 탄생하지만, CountDownLatch를 쓰면 직관적으로 구현 할 수 있다

```java
public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
    CountDownLatch ready = new CountDownLatch(concurrency);
    CountDownLatch start = new CountDownLatch(1);
    CountDownLatch done = new CountDownLatch(concurrency);

    for(int i = 0; i < concurrency; i ++) {
        executor.execute(() -> {
            ready.countDown();
            try {
                start.await();
                action.run();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                done.countDown();
            }
        });
    }

    ready.await();
    long startNanos = System.nanoTime();
    start.countDown();
    done.await();
    return System.nanoTime() - startNanos;
}
```

- 이 코드는 카운트다운 래치를 3개 사용한다
    * ready 래치는 작업자 스레드들이 준비가 완료됬음을 타이머 스레드에 통지할 때 사용한다
    * 통지를 끝낸 작업자 스레드들은 두번째 래치인 start가 열리기를 기다린다
    * 마지막 작업자 스레드가 ready.countDown을 호출하면 타이머 스레드가 시작 시간을 기록하고 start.countDown을 호출하여 기다리던 작업자 스레드들을 깨운다
    * 그 직후 다이머 스레드는 세번째 래치인 done이 열리기를 기다리며, done래치는 마지막 남은 작업자 스레드가 동작을 마치고 done.countDown을 호출하면 열린다
    * 타이머 스레드는 done래치가 열리자마자 깨어나 종료 시각을 기록한다

- 세부사항으로 time 메서드에 넘겨진 실행자는 concurrency 매개변수로 지정한 동시성 수준만큼의 스레드를 생성 할 수 있어야 한다. 그렇지 못하면 이 메서드는 결코 끝나지 않을것이다. 이런 상태를 기아 교착상태라고 한다
- InterruptedException을 캐치한 작업자 스레드는 Thread.currentThead().interrupt() 관용구를 사용해 인터럽트를 되살리고 자신은 run 메서드에서 빠져나온다. 이렇게 해야 실행자가 인터럽트를 적절하게 처리 할 수 있다
- 또 한가지 주목해야 할것은, System.nanoTime을 사용해 시간을 잰것이다. 시간 간격을 잴때는 항상 System.currentTimeMillis가 아닌 System.nanoTime을 사용해야 한다
- 이번 아이템은 동시성 유틸리티를 맛만 살짝 보여준다. 앞에서 사용한 카운타운 래치 3개는, CyclicBarrier(혹은 Phaser) 인스턴스 하나로 대체된다. 이렇게하면 코드는 더 명료하지만 이해하기는 더 어려울 수 있다
- 새로운 코드라면 언제나 wait와 notify가 아닌 동시성 유틸리티를 써야한다. 하지만 어쩔수 없이 레거시 코드를 다뤄야 한다면 wait은 스레드가 어떤 조건이 충족되기를 기다리게 할 때 사용하며, 락 객체의 wait 메서드는 반드시 그 객체를 잠근 동기화 영역 안에서 호출해야 한다

```java
synchronized(obj) {
    while(<조건이 충족되지 않음>)
        obj.wait();
}
```

- wait을 사용하는 표준 양식은 위와 같다. wait 메서드를 사용할 떄는 반드시 대기 반복문(wait loop)관용구를 사용하라. 반복문 밖에서는 절대로 호출 하지말자
- 대기전에 조건을 검사하여 조건이 이미 충족되었다면, wait을 건너 뛰게 하는것은 응답 불가 상태를 예방하는 조치이다. 만약 조건이 이미 충족되었는데 스레드가 notify(혹은 notifyAll) 메서드를 먼저 호출한 후 대기상태로 빠지면, 스레드를 다시 깨울 수 있다고 보장할 수 없다
- 대기 후에 조건을 검사하여 조건이 충족되지 않았다면 다시 대기하게 하는것은 안전 실패를 막는 조치이다. 만약 조건이 충족되지 않았는데 스레드가 동작을 이어나가면 락이 보호하는 불변식을 깨트릴 위험이 있다
- 조건이 만족되지 않아도 스레드가 깨어날 수 있는 상황이 몇가지 있는데, 다음이 그 예이다
    * 스레드가 notify를 호출 한다음 대기중이던 스레드가 깨어나는 사이에 다른 스레드가 락을 얻어 그 락이 보호하는 상태를 변경한다
    * 조건이 만족되지 않았음에도, 다른 스레드가 실수로 혹은 악의적으로 notify를 호출한다. 공개 된 객체를 락으로 사용해 대기하는 클래스는 이런 위험에 노출된다. 외부에 노출된 객체의 동기화된 메서드 안에서 호출하는 wait는 모두 이 문제에 영향을 받는다
    * 깨우는 스레드는 지나치게 관대해서, 대기중인 스레드중 일부만 조건이 충족되어도 notifyAll을 호출해 모든 스레드를 깨울 수도 있다
    * 대기중인 스레드가 (드물게) notify 없이도 깨어나는 경우가 더러 있다. 허위 각성이라는 현상이다

- notify와 notifyAll중 어느것은 선택하느냐에 대한 문제도 있는데, 일반적으로는 언제나 notifyAll을 사용하는게 합리적이고 안전한 조언이 될것이다. 깨어나야 하는 모든 스레드가 깨어남을 보장하니 항상 정확한 결과를 얻을 수 있을것이다. 다른 스레드가 깨어날수도 있긴 하지만, 그것이 정확성에는 영향을 주지 않을것이다. 깨어난 스레드들은 기다리던 조건이 충족되지 않았다면 다시 대기할것이다
- 모든 스레드가 같은 조건을 기다리고, 조건이 충족될때마다 단 하나의 스레드만 혜택을 받을 수 있다면 notifyAll대신 notify를 사용해 최적화 할 수 있다
- 하지만 notify대신 notifyAll을 사용해야할 이유가 있는데, 외부로 공개된 객체에 대해 실수로 혹은 악의적으로 notify를 호출하는 상황에 대비하기 위해 wait을 반복문 안에서 호출 했듯, notifyAll을 사용하면 사용하면 관련없는 스레드가 실수로 혹은 악의적으로 wait을 호출하는 공격으로 부터 보호할 수 있다. 그런 스레드가 중요한 notify를 삼켜버린다면 꼭 깨어낫어야할 스레드들이 영원히 대기 할 수 있다