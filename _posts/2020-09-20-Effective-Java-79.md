---
layout: post
title: "Effective Java - 아이템79: 과도한 동기화는 피하라"
description: 과도한 동기화는 피하라
date: 2020-09-20 21:17:00 +09:00
categories: EffectiveJava Study
---


# 예외

## 아이템 79 : 과도한 동기화는 피하라

- 과도한 동기화는 성능을 떨어트리고, 교착상태에 빠트리고, 심지어 예측할 수 없는 동작을 낳기도 한다
- 응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안된다
    * 예를들어 동기화된 영역 안에서는 재정의 할 수 있는 메서드를 호출하면 안되며, 클라이언트가 넘겨준 함수 객체를 호출해서도 안된다
    * 동기화된 영역을 포함한 클래스 관점에서는 이런 메서드는 모두 바깥 세상에서 온것이며, 그 메서드가 무슨일을 하는지 알지 못하며 통제할 수 없다
    * 이런 에일리언 메서드가 하는 일에 따라 동기화된 영역은 예외를 일으키거나, 교착상태에 빠지거나, 데이터를 훼손 할수도 있다

- 구체적인 예로, 다음은 어떤 Set을 감싼 래퍼 클래스이고, 이 클래스의 클라이언트는 집합에 원소가 추가되면 알림을 받을 수 있다. 옵저버 패턴인것이다. 또한 아이템 18의 ForwardingSet을 재사용해 구현한것이다

```java
public class ObserbableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) { super(set); }

    private final List<SetObserver<E>> observer = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) {
        synchronized(observers) {
            for(SetObserver<E> observer : observers ){
                observer.added(this, element);
            }
        }
    }

    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if(added)
            nofigyElementAdded(element);
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for(E element : c)
            result |= add(element);
        return result;
    }
}
```

- 관찰자들은 addObserver와 removeObserver 메서드를 호출해 구독을 신청하거나 해지한다. 두경우 모두 다음 콜백 인터페에스의 인스턴스를 메서드에게 건넨다

```java
@FunctionalInterface
public interface SetObserver<E> {
    void added(ObservableSe<E> set, E element);
}
```

- 이 인터페이스는 구조적으로 ```BiConsumer<ObservableSet<E>,E>```와 동일하다. 그럼에도 커스텀함수형 인터페이스를 정의한 이유는 이름이 더 직관적이고 다중 콜백을 지원 하도록 확장 할 수 있어서 이다
- 눈으로 보기에 ObservableSet은 잘 동작할것 같다. 예컨대 다음 프로그램은 0부터 99까지를 출력한다

```java
public static void main(String[] args) {
    ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());

    set.addObserver((s,e) -> System.out.println(e));

    for(int i = 0; i < 100; i++)
        set.add(i);
}
```

- 이제 조금 흥미진진한 시도를 해보자. 평상시에는 앞서와 같이 집합에 추가된 정숫값을 출력하다, 그 값이 23이면 자기 자신을 제거(구독해지) 하는 관찰자를 추가 해보자

```java
set.addObserver(new SetObserver<>() {
    public void added(ObservableSet<Integer> s, Integer e) {
        System.out.println(e);
        if(e == 23)
            s.removeObserver(this);
    }
});
```

- 이 프로그램은 0부터 23까지를 출력한 후 관찰자 자신을 구독해지 한다음 조용히 종료할것이라 예상하지만, 실제로 보면 그렇게 진행되지 않는다! 이 프로그램은 23까지 출력한다음 ConcurrentModificationException을 던진다. 관찰자의 added메서드 호출이 일어난 시점이 notifyElementAdded가 관찰자들의 리스트를 순회하는 도중이기 때문이다
    * added메서드는 ObservableSet의 removeObserver 메서드를 호출하고, 이 메서드는 다시 observers.remove 메서드를 호출한다. 여기서 문제가 발생하는 것이다
    * 리스트에서 원소를 제거하려는데, 마침 지금은 이 리스트를 순회하는 도중이니 허용되지 않은 동작인것이다. notifyElementAdded메서드에서 수행하는 순회는 동기화 블록 안에 있으므로 동시수정이 일어나지 않도록 보장하지만, 정작 자신이 콜백을 거쳐 되돌아와 수정하는것까지는 막지 못한다

- 이번에는 조금 이상한 시도로, 구독 해지를하는 관찰자를 작성하는데, removeObserver를 직접 호출하지 않고, 실행자 서비스를 사용해 다른 스레드한테 부탁 하는것이다

```java
set.addObserver(new SetObserver<>() {
    public void added(ObservableSet<Integer> s, Integer e) {
        System.out.println(e);
        if(e == 23) {
            ExecutorService exec = Excutors.newSingleThreadExecutor();
            try {
                exec.submit(() -> s.removeObserver(this)).get();
            } catch(ExecutionException | InturrupedException ex) {
                throw new AssertionError(ex);
            } finally {
                exec.shutdown();
            }
        }
    }
});
```

- 이 프고르램을 실행하면 예외는 나지 않지만 교착상태에 빠진다. 백그라운드 스레드가 s.removeObserver를 호출하면 관찰자를 잠그려 시도하지만, 락을 얻을 수 없다. 메인 스레드가 이미 락을 쥐고 있기 때문이다. 그와 동시에 메인 스레드는 백그라운드 스레드가 관찰자를 제거하기만을 기다리는 중이다. 교착상태인것이다
- 사실 관찰자가 자신을 구독해지 하는데 굳이 백그라운드 스레드를 이용할 이유가 없으니 좀 억지스러운 예시이지만, 보안문제자체는 진짜이다. 실제 시스템에서도 동기화된 영역 안에서 에일리언 메서드를 호출하여 교착상태에 빠지는 사례는 자주 있다
- 앞서의 두 예에서는 운이 좋았다. 동기화 영역이 보호하는 자원(관찰자)은 에일리언 메서드가 호출될 때 일관된 상태였으니 말이다
- 그렇다면 똑같은 상화이지만 불변식이 임시로 깨진 경우라면 어떻게 될까? 자바언어의 락은 재진입을 허용하므로 교착상태에 빠지지는 않는다
- 예외를 발생시킨 첫번째 예에서라면 에일리언 메서드를 호출하는 스레드는 이미 락을 쥐고 있으므로, 다음번 락 획득도 성공한다. 그 락이 보호하는 데이터에 대해 개념적으로 관련이 없는 다른 작업이 진행중인데도 말이다. 문제의 주 원인은 락이 제 구실을 하지 못하였기 때문이다
- 재진입 가능 락은 객체지향 멀티스레드 프로그램을 쉽게 구현할 수 있도록 해주지만, 응답불가(교착상태)가 될 상황을 안전실패로 변모 시킬 수 있다
- 다행이 이런 문제는 대부분 어렵지 않게 해결 할 수있다. 에일리언 메서드 호출을 동기화 블록 바깥으로 옮기면 된다. notifyElementAdded 메서드에서라면 관찰자 리스트를 복사해 쓰면 락 없이도 안전하게 순회할 수 있다. 예외 발생과 교착상태 증상이 모두 사라진다

```java
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized(observers) {
        snapshot = new ArrayList<>(observers);
    }
    for(SetObserver<E> observer : snapshot)
        observer.added(this, element);    
}
```

- 사실 에일리언 메서드 호출을 동기화 블록 바깥으로 옮기는 더 나은 방법이 있다. 자바의 동시성 컬렉션 라이브러리의 CopyOnWriteArrayList가 정확히 이 목적으로 특별히 설계된것이다
    * 이름에서 말해주듯 ArrayList를 구현한 클래스로, 내부를 변경하는 작업은 항상 깨끗한 복사본을 만들어 수행하도록 구현했다
    * 내부의 배열은 절대 수정되지 않으니 순회할 때 락이 필요없어 매우 빠르다
    * 다른 용도로 쓰인다면 CopyOnWriteArrayList는 끔직히 느리겠지만, 수정할일은 드물고 순회만 빈번히 일어나는 관찰자 리스트 용도로는 최적이다

- ObservableSer을 CopyOnWriteArrayList를 사용해 다시 구현하면 다음과 같이 바꾸면 된다(add와 addAll은 바꿀게 없다). 명시적으로 동기화를 한곳이 없어졌다는것에 주목해야 한다

```java
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

private void notifyElementAdded(E element) {
    for(SerObserver<E> observer : observers)
        observer.added(this, element);
}
```

- 이전의 예시처럼 동기화 영역 바깥에서 호출되는 에일리언 메서드를  열린 호출이라고 한다. 에일리언 메서드는 얼마나 오래실행 될 지 알 수 없는데, 동기화 영역 안에서 호출 된다면 그동안 다른 스레드는 보호된 자원을 사용하지 못하고 대기해야만 한다. 따라서 열린 호출은 실패 방지 효과외에도 동시성 효율을 크게 개선해준다
- 기본 규칙은 동기화 영역에서는 가능한 일은 적게 하는것이다. 락을 얻고, 공유데이터를 검사하고, 필요하면 수정하고, 락을 놓는다. 오래걸리는 작업이라면 아이템78의 지침을 어기지 않으며 동기화 영역 바깥으로 옮기는 방법을 찾아보자
- 성능 측면도 간단히 살펴보자. 자바의 동기화 비용은 빠르게 낮아져왔지만, 과도한 동기화를 피하는 일은 오히려 더 중요해졌다. 멀티코어가 일반화된 오늘날, 과도한 동기화가 초래하는 진짜 비용은 락을 얻는데 드는 CPU비용이 아닌, 경쟁하느라 낭비하는 시간, 즉 병렬로 실행할 기회를 잃고 모든 코어가 메모리를 일관되게 보기 위한 지연시간이 그 진짜 비용이다. 가상 머신의 코드 최적화를 제한한다는 점도 과도한 동기화의 숨은 비용이다
- 가변 클래스를 작성하려거든 다음 두 선택지중 하나를 따르자
    * 동기화를 전혀 하지 말고, 그 클래스를 동시에 사용해야하는 클래스가 외부에서 알아서 동기화 하게 하자
    * 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들자. 클라이언트가 외부에서 객체 전체에 락을 거는것보다 동시성을 월등히 개선 할 수 있을때만 이 방법을 선택해야 한다
    * java.util은 Vector와 HashTable을 제외하고는 첫번째 방식을 취했고, java.util.concurrent는 두번째 방식을 취했다

- 클래스 내부에서 동기화 하기로 했다면, 락 분할과 락 스트라이핑, 비차단 동시성 제어등 다양한 기법을 동원해 동시성을 높여 줄 수있다
- 여러 스레드가 호출할 가능성이 있는 메서드가 정적 필드를 수정한다면 그 필드를 사용하기전에 반드시 동기화 해야한다(비 결정적 행동도 용인하는 클래스라면 상관없다)
    * 하지만 클라이언트가 여러 스레드로 복제돼 구동하는 상황이라면, 다른 클라이언트에서 이 메서드를 호출하는것을 막을 수 없으니 외부에서 동기화 할방법이 없다
    * 이 정적 필드가 private라도 서로 관련이 없는 스레드들이 동시에 읽고 수정할 수 있게되어, 사실상 전역변수와 같아진다
    * 이전 아이템에서 generateSerialNumber 메서드에서 쓰인 nextSerialNumber가 바로 이러한 사례이다