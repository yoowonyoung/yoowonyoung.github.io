---
layout: post
title: "Effective Kotlin - 아이템24: 제너릭 타입과 variance 한정자를 활용하라"
description: 제너릭 타입과 variance 한정자를 활용하라
date: 2022-02-23 22:14:00 +09:00
categories: EffectiveKotlin Study
---


# 재사용성

## 아이템 24 : 제너릭 타입과 variance 한정자를 활용하라
- variance 한정자 (out 또는 in)이 없이 제너릭을 사용하면 invariant(불공변성)으로 제너릭 타입으로 만들어지는 타입들이 서로 관련성을 갖지 않는다는것을 의미
- out은 타입 파라미터를 covariant(공변성)으로 만듬

```kotlin
class Cup<out T>
open class Dog
class Puppy: Dog()

fun main(args: Array<String>) {
    val b: Cup<Dog> = Cup<Puppy>() // out으로 명시 되어 있어서 Cup<Puppy> 는 Cup<Dog>의 서브 타입
    val a: Cup<Puppy> = Cup<Dog>() // 오류
}
```

- in 한정자는 타입 파라미터를 contravariant(반변성)으로 만듬

```kotlin
class Cup<in T>
open class Dog
class Puppy: Dog()

fun main(args: Array<String>) {
    val b: Cup<Dog> = Cup<Puppy>() // 오류
    val a: Cup<Puppy> = Cup<Dog>() // in으로 명시되어 Cup<Puppy>가 Cup<Dog>의 슈퍼 타입
}
```

### 함수 타입
- 함수 타입은 파라미터 유형과 리턴 타이벵 따라서 서로 어떤 관계를 가짐

```kotlin
fun printProcessedNumber(transition: (Int) -> Any) {
    print(transition(42))
}

// (Int) -> Any, (Int) -> Number, (Number) -> Any, (Number) -> Any,(Number) -> Number 등으로 가능. Int -> Number -> Any 의 슈퍼타입 이기 때문
val intToDouble: (Int) -> Number = { it.toDouble() }
val numberAsText: (Number) -> Any = { it.toShort() }

printProcessedNumber(intToDouble)
printProcessedNumber(numberAsText)
```

- 코틀린 함수 타입의 모든 파라미터 타입은 contravariant 이고, 모든 리턴 타입은 contravariant 이기 때문
    * (T in, T in) -> T out

### variance 한정자의 안정성
- 자바의 배열은 covariant이기 때문에 런타임에 문제가 발생 할 수 있음( ex> Integer 배열을 런타임에 Object배열로 캐스팅 하고, String 값을 할당 하는 경우. 컴파일 과정에선 문제가 발생하지 않으나 런타임에 오류 발생)
- 타입 파라미터를 예측할 수 있다면 어떤 서브 타입이라도 전달 할 수 있고, 아규먼트를 전달할때 암묵적인 업캐스팅도 가능. 이것은 covariant하지 않음. 코틀린에서는 이것을 public in 한정자 위치에 out 한정자가 오는것을 금지하는것으로 막고 있음

```kotlin
// covariant 하지 못함
open class Dog
class Puppy: Dog()
class Hound: Dog()

fun takeDog(dog: Dog) {}

takeDog(Dog())
takeDog(Puppy())
takeDog(Hound())

// covariant
class Box<out T> {
    private var value: T? = null // private가 없다면 오류
    
    private set(value: T) { // private가 없다면 오류
        this.value = value
    }
}
```

- out 한정자는 public out 한정자 위치에서도 안전하므로 따로 제한되지는 않음
- 예시로 ```List<T>```는 covariant로 함수의 파라미터가 ```List<Any?>```로 예측 된다면 별도의 변환 없이 모든 종류를 파라미터로 전달 할 수 있지만, ```MutableList<T>```에서는 T는 in 한정자 위치에서 사용되며 안전하지 않으므로 invariant
- contravaraint 타입 파라미터에서도 비슷한 문제가 발생 하는데, out은 암묵적인 업캐스팅을 하용 하는데 contravariant에 맞는 동작은 아니므로, 코틀린은 covariant 타입 파라미터(in 한정자)를 public out 한정자 위치에 사용하지 못하게 막음

```kotlin
class Box<in T> {
    private var value: T? = null // private가 없다면 오류
    
    private set(value: T) { // private가 없다면 오류
        this.value = value
    }
}
```

### variance 한정자의 위치
- variance 한정자는 선언 부분에 사용 하거나 클래스와 인터페이스를 활용 하는 2가지 방법이 있음

```kotlin
// 선언 부분. 클래스와 인터페이스가 사용되는 모든 곳에 영향
class Box<out T>(val value: T)
val boxStr: Box<String> = Box("Str")
val boxAny: Box<Any> = boxStr

// 클래스와 인터페이스를 활용. 특정 변수에만 variance 한정자 적용
class Box<T>(val value: T)
val boxStr: Box<String> = Box("Str")
// 사용하는 쪽의 varaiance
val boxAny: Box<out Any> = boxStr
// MutableList는 in 한정자를 포함하면 요소를 리턴하지 못하므로 in을 붙이지 않지만, 단일 파라미터 타입에는 붙일수 있다
fun fillWithPuppies(list: MutableList<in Puppy>) {
    list.add(Puppy("Beam"))
    list.add(Puppy("Jim"))
}
```

- variance 한정자를 사용하면 위치는 제한됨
    * ```MutableList<out T>```가 있다면 get으로 요소를 추출하면 T 타입이 예상 되지만 set은 Nothing 타입이 전달될꺼라 예상 되므로 사용할 수 없음(모든 타입의 서브 타입을 갖는 리스트인 Noting 리스트가 존재할 가능성이 있기 떄문)
    * ```MutableList<in T>```가 있다면 get/set을 모두 쓸 수 있지만 get을 통해 전달되는 자료형은 Any?(모든 타입의 슈퍼 타입을 가진 리스트인 Any리스트가 존재할 가능성이 있기 때문)

### 정리
- 타입 파라미터의 기본적인 variance 동작은 invariant. 아무런 관계가 없음
- out 한정자는 타입 파라미터를 covariant하게 만들기 때문에 A가 B의 서브타입이면 ```Cup<A>``` 는 ```Cup<B>```의 서브타입. 이런 covariant 타입만 out 위치에 사용 가능
- in 한정자는 타입 파라미터를 contravariant하게 만들기 때문에 A가 B의 서브타입이면 ```Cup<B>``` 는 ```Cup<A>```의 슈퍼타입. 이런 contravariant 타입만 in 위치에 사용 가능
- 코틀린에서 List와 Set의 타입 파라미터는 covaraint(out 한정자), MutableList와 MutableSet의 타입 파라미터는 covaraint(out 한정자) 의 타입 파라미터는 invariant(한정자 없음)
- 함수 타입의 파라미터 타입은 contravariant(in 한정자), 리턴 타입은 contravariant(out 한정자)
- 리턴만 되는 타입에는 covariant(out 한정자), 허용만 되는 타입에는 contravariant(in 한정자)