---
layout: post
title: "Effective Kotlin - 아이템36: 상속보다는 컴포지션을 사용하라"
description: 상속보다는 컴포지션을 사용하라
date: 2022-03-06 22:00:00 +09:00
categories: EffectiveKotlin Study
---


# 클래스 설계

## 아이템 36: 상속보다는 컴포지션을 사용하라

- 상속은 관계가 명확하지 않을때 사용하면 여러 문제가 발생할 수있음

### 간단한 행위 재 사용

```koltin
class ProfileLoader {
    fun load() {
        // 프로그레스바를 보여줌
        // 프로파일을 읽어 들임
        // 프로그래스 바를 숨김
    }
}

class ImageLoader {
    fun load() {
        // 프로그래스바를 보여줌
        // 이미지를 읽어 들임
        // 프로그래스 바를 숨김
    }
}
```

- 위와 같은 코드는 보통 아래와 같이 슈퍼 클래스를 만들어 공통된 행위를 추출함

```kotlin
abstract class LoaderWithProgress {
    fun load() {
        // 프로그래스바를 보여줌
        innerLoad()
        // 프로그래스바를 숨김
    }
}

class ProfileLoader: LoaderWithProgress() {
    override fun innerLoad() {
        // 프로파일을 읽어들임
    }
}

class ImageLoader: LoaderWithProgress() {
    override fun innerLoad() {
        // 이미지를 읽어 들임
    }
}
```

- 위와 같은 코드는 문제 없이 동작 하지만, 아래와 같은 단점이 있음
    * 상속은 하나의 클래스를 대상으로 하기 때문에, 상속을 사용해서 행위를 추출 하다보면 많은 함수를 갖는 거대한 클래스가 만들어지게 되고 깊고 복잡한 계층 구조가 만들어짐
    * 상속은 클래스의 모든것을 가져오기 때문에 불필요한 함수를 갖는 클래스가 만들어질 수 있음(인터페이스 분리의 원칙 위반)
    * 상속은 이해하기 어려움

- 위와 같은 이유 때문에 컴포지션을 사용 하는게 좋은데, 컴포지션은 객체를 프로퍼티로 갖고 함수를 호출 하는 형태로 재 사용하는것을 의미

```kotlin
class Progress{
    fun showProgress() { //... }
    fun hideProgress() { //... }
}

class ProfileLoader {
    val progress = Progress()

    fun load() {
        progress.showProgress()
        // 프로파일 읽기
        progress.hideProgress()
    }
}

class ImageLoader {
    val progress = Progress()

    fun load() {
        progress.showProgress()
        // 이미지 읽기
        progress.hideProgress()
    }
}
```

- 컴포지션은 추가 코드가 필요 하며, 추가 코드를 적절하게 처리하는게 조금은 어려울수도 있지만, 추가 코드로 인해서 코드를 읽는 사람들이 코드의 실행을 더 명확하게 예측 할 수 있다는 장점도 있음
- 컴포지션을 활용하면 하나의 클래스 내부에서 여러 기능을 재사용 할 수 있게 됨

### 모든것을 가져올 수밖에 없는 상속
- 상속은 슈퍼클래스의 메서드, 제약, 행위등 모든것을 가져오기 떄문에 계층 구조를 나타낼떄 굉장히 좋지만, 일부분을 재사용하기 위한 목적으로는 적합하지 않음
- 일부분만 재사용하고 싶다면 컴포지션을 사용하는게 좋은데, 우리가 원하는 행위만 가져올 수 있기 때문

### 캡슐화를 꺠는 상속
- 상속을 활용할떄는 내부적인 구현 방법에 의해서 클래스의 캡슐화가 쉽게 깨질 수 있기 때문에 주의 해야 함

```kotlin
class CounterSet<T>: HashSet<T> {
    var elemantsAdded: Int = 0
        private set

    override fun add(element: T): Boolean {
        elementsAdded++
        return super.add(element)
    }

    override fun addAll(elements: Collection<T>): Boolean {
        elementsAdded += elements.size
        return super.addAll(elements) 
    }
}

val counterList = CounterSet<String>()
counterList.addAll(listOf("A","B","C"))
print(counterList.elementsAdded) // 6, HashSet의 addAll 내부에서 add를 사용해서 2배로 나오게 됨
``` 

- 위 문제를 해결하기 위해 addAll 함수를 제거 할 수 있으나, 어느날 라이브러리 구현이 변경이 된다면 문젝 ㅏ발생함
- 상속대신 컴포지션으로 구현을 하면 되지만 이는 다형성이 사라지게 되어 CounterSet은 더이상 Set이 아니게 됨
    * 이를 유지 하고 싶다면 위임 패턴을 사용하면 되는데, 코틀린에서 위임 패턴은 다음과 같이 쉽게 작성 가능
    * 다형성이 필요한데 상속된 메서드를 직접 활용하는것이 위험할떄는 위임 패턴을 사용하는게 좋음
    * 일반적으로 다형성이 그렇게까지 필요한 경우는 없기 떄문에 단순하게 컴포지션을 활용하면 해결되는 경우가 많음

```kotlin
class CounterSet<T>(
    private val innserSet: MutableSet<T> = mutableSetOf()
): MutableSet<T> by innserSet {
    var elementsAdded: Int = 0
        private set

    override fun add(element: T): Boolean {
        elementsAdded++
        return innserSet.add(element)
    }

    override fun addAll(elements: Collection<T>): Boolean {
        elementsAdded += elements.size
        return innerSet.addAll(elements)
    }
}
```

- 상속으로 캡슐화를 꺨 수 있다는것은 사실은 보안 문제. 대부분의 경우 이러한 행위는 규약으로 지정되어 있거나 서브클래스에 의존할 필요가 없는 경우임

### 오버리이딩 제한 하기
- 어떤 이유로 상속은 허용하지만 메서드는 오버라이드 하지 못하게 만들고 싶은 경우가 있다면, 상속용으로 설계된 메서드에만 open을 붙이면 됨

### 정리
- 컴포지션과 상속의 차이
    * 컴포지션이 더 안전함. 다른 클래스의 내부적인 구현에 의존하지 않고 외부에서 관찰되는 동작에만 의존하므로
    * 컴포지션이 더 유연함. 상속은 한 클래스만을 대상으로 하지만, 컴포지션은 여러 클래스를 대상으로 할 수 있고, 컴포지션은 필요한것만 받을 수 있음
    * 컴포지션이 더 명시적. 이것은 장점이자 단점인데, 리시버를 명시적으로 활용할 수 밖에 없으므로 메서드가 어디것인지 확실하게 알 수 있음
    * 컴포지션이 더 번거로움. 객체를 명시적으로 사용해야 하므로, 대상 클래스에 일부 기능을 추가할 떄 이를 포함한느 객체의 코드를 변경 해야함
    * 상속은 다형성을 활용할 수 있음

- 일반적으로 OOP에서는 상속보다 컴포지션을 사용하는게 좋음
- 상속은 명확한 is-a 관계 일때 사용 하는것이 좋음
