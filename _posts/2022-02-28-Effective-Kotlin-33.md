---
layout: post
title: "Effective Kotlin - 아이템33: 생성자 대신 팩토리 함수를 사용하라"
description: 생성자 대신 팩토리 함수를 사용하라
date: 2022-02-28 22:41:00 +09:00
categories: EffectiveKotlin Study
---


# 객체 생성

## 아이템 33: 생성자 대신 팩토리 함수를 사용하라
- 디자인패턴으로 굉장히 다양한 생성 패턴들이 만들어져있음. 그중 생성자의 역할을 대신해주는 함수를 팩토리 함수라 하며 다음과 같은 장점을 가지고 있음
    * 생성자와는 다르게 함수에 이름을 붙일 수 있어서, 이름을 통해 객체가 생성되는 방법과 아규먼트로 무엇이 필요하는지를 설명 가능하며, 동일한 파라미터 타입을 갖는 생성자의 충돌을 줄일 수 있음
    * 생성자와는 다르게 함수가 원하는 형태의 타입을 리턴할 수 있음. 인터페이스 뒤에 실제 객체의 구현을 숨기기에 유용해짐
    * 생성자와는 다르게 호출될 때마다 새 객체를 만들 필요가 없음. 싱글턴처럼 하나만 만들게 하거나 캐싱 매커니즘도 활용 가능
    * 아직 존재하지 않는객체를 리턴할 수있음
    * 객체 외부에 팩토리 함수를 만들면 그 가시성을 원하는대로 제어 할 수 있음
    * 팩토리 함수는 인라인으로 만들수있으며, 그 파라미터들을 reified로 만들 수 있음
    * 생성자로 만들기 복잡한 객체도 만들어낼 수있음
    * 생성자는 즉시 슈퍼클래스 또는 기본 생성자를 호출해야 하지만, 팩토리 함수는 원하는 떄에 호출 가능

- 서브클래스 생성에는 슈퍼클래스의 생성자가 필요하기 때문에 서브 클래스를 만들어 낼 수없으므로, 팩토리 함수로 슈퍼클래스를 만들기로 했다면 서브 클래스에도 팩토리 함수를 만들어야함
- 기본 생성자를 사용하지 말란말은 아님. 팩토리 함수 내부에서는 생성자를 사용해야 하기 떄문. 팩토리 함수는 기본 생성자가 아닌 추가 생성자와 경쟁관계인 것

### Companion 객체 팩토리 함수
- 팩토리 함수를 정의하는 가장 일반적인 방법은 companion객체를 사용하는것

```kotlin
class MyLinkedList<T>(
    val head: T,
    val tail: MyLinkedList<T>?
) {
    companion object {
        fun <T> of(vararg element: T): MyLinkedList<T>? {
            /*...*/
        }
    }
}

val list = MyLinkedList.of(1,2)

// 코틀린에서는 인터페이스에서도 됨
class MyLinkedList<T>(
    val head: T,
    val tail: MyLinkedList<T>?
): MyList<T> {
    // ...
}

interface MyList<T> {
    // ...
    companion object {
        fun <T> of(vararg element: T): MyList<T>? {
            /*...*/
        }
    }
}

val list = MyLinkedList.of(1,2)
```

- 팩토리 함수 이름 규칙
    * from: 파라미터를 하나 받고, 같은 타입의 인스턴스를 하나 리턴
    * of: 파라미터를 여러개 받고, 이를 통합해서 인스턴스를 만들어줌
    * valueOf: from 또는 of 와 비슷하지만, 의미를 조금 더 쉽게 읽을 수 있게 이름을 붙임
    * instance 또는 getInstance: 싱글턴으로 인스턴스를 하나 리턴
    * createInstance 또는 newInstance: getInstance처럼 동작하지만 싱글턴이 적용되지 않아 호출 할때마다 새로운 인스턴스
    * getType: getInstance 처럼 동작하지만 팩토리 함수가 다른 클래스에 있을 때 사용하는 이름. 타입은 팩토리 함수에서 리턴하는 타입
    * newType: newInstance 처럼 동작하지만 팩토리 함수가 다른 클래스에 있을 때 사용하는 이름. 타입은 팩토리 함수에서 리턴하는 타입

- Companion 객체는 더 많은 기능을 가지고 있음. 예를 들어 companion 객체는 인터페이스를 구현할 수있으며 클래스를 상속 받을 수도 있고, 추상 companion 객체 팩토리는 값을 가질 수 있음. 이를 이용해서 캐싱이나 테스트용 가짜 객체 생성도 가능함

```kotlin
abstract class ActivityFactory {
    abstract fun getInstance(context: Context): Intent

    fun start(context: Context) {
        // ...
    }

    fun startForResult(activity: Activity, requestCode: Int) {
        // ...
    }
}


class MainActivity: AppCompactActivity {
    //...

    companion object: ActivityFactory() {
        override fun getInstance(context: Context): Intent = 
            Intent(context, MainActivity::class.java)
    }
}


val intent = MainActivity.getIntent(context)
MainActivity.start(context)
MainActivity.startForResult(activity,requestCode)
```

### 확장 팩토리 함수
- companion 객체를 직접 수정할 수 없고, 다른 파일에 함수를 만들어야 한다면 확장 함수를 활용 하면됨

```kotlin
interface Tool {
    companion object { /*...*/ }
}

fun Tool.Companion.createBigTool(/*...*/) : BigTool {
    // ...
}

Tool.createBigTool()
```

- 이런식으로 외부 라이브러리 확장이 가능하지만, companion 객체를 확장 하려면 적어도 비어있는 컴패니언 객체는 있어야 함

### 톱레벨 팩토리 함수
- listOf, setOf, mapOf 같은 톱레벨 팩토리 함수도 존재함
- 톱레벨 함수는 신중하게 사용해야 하는데, public 톱레벨 함수는 모든 곳에서 사용할 수 있으므로 IDE가 제공하는 팁을 복잡하게 만드는 단점이 있고, 톱레벨 함수의 이름을 클래스 메서드 이름처럼 만들면 혼란은 가중됨

### 가짜 생성자
- 코틀린의 생성자는 톱레벨 함수와 같은 형태로 사용되기 때문에 톱레벨 함수처럼 참조 될 수 있음

```kotlin
class A
val a = A()

val reference: () ->A = ::A
```

- List와 MutableList는 인터페이스라 생성자를 가질 수 없지만 다음과 같은 함수가 존재하므로 생성자처럼 사용 가능

```kotlin
public inline fun <T> List(
    size: Int,
    init: (index: Int) -> T
): List<T> = MutableList(size, init)

public inine fun <T> MutableList(
    size: Int,
    init: (index: Int) -> T
): MutableList<T> {
    val list = ArrayList<T>(size)
    repeat(size) { index -> list.add(init(index)) }
    return list
}
```

- 이런 톱레벨 함수는 생성자처럼 보이고, 생성자처럼 동작하지만 팩토리 함수와 같은 모든 장점을 가짐. 이것을 가짜 생성자라고 부름
- 개발자가 진짜 생성자 대신 가짜 생성자를 만드는 이유는 인터페이스를 위한 생성자를 만들고 싶거나, reified 타입 아규먼트를 갖게 하고 싶을떄임
- invoke를 통해서 가짜 생성자를 만들수도 있지만 이는 추천하지 않는 방법. 톱레벨 함수를 사용하는게 좋음

### 팩토리 클래스의 메서드
- 팩토리 클래스는 클래스의 상태를 가질 수 있다는 특징 때문에 팩토리 함수보다 다양한 기능을 가짐
- 팩토리 클래스는 프로퍼티를 가질 수 있기 때문에 이를 활용해 최적화나 캐싱 등 다양한 기능을 도입할 수있음
