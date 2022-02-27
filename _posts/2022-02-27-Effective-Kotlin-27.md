---
layout: post
title: "Effective Kotlin - 아이템27: 변화로부터 코드를 보호 하려면 추상화를 사용하라"
description: 변화로부터 코드를 보호 하려면 추상화를 사용하라
date: 2022-02-27 20:20:00 +09:00
categories: EffectiveKotlin Study
---


# 추상화 설계

## 아이템 27 : 변화로부터 코드를 보호 하려면 추상화를 사용하라
- 물위를 겉는것과 명세서로 소프트웨어를 개발하는것은 쉽다. 둘 다 동결 되어 있다면

### 상수
- 리터럴을 상수 프로퍼티로 변경하면 해당 값에 의미 있는 이름을 붙일 수 있으며, 상수의 값을 변경 해야할 때 쉽게 변경 가능

### 함수
- 많이 사용되는 알고리즘은 확장함수로 만들어서 사용 가능

```kotlin
// 안드로이드에서 토스트 메시지를 띄우는 예시
Toast.makeText(this, message, Toast.LENGTH_LONG).show()

fun Context.toast(
    message: String,
    duration: Int = Toast.LENGTH_LONG
) {
    Toast.makeText(this, message, duration).show()
}

context.toast(message)
```

### 클래스

```kotlin
class MessageDisplay(val context: Context) {
    fun show(
        message: String,
        duration: MessageLength = MessageLength.LONG
    ) {
        val toastDuration = when(duration) {
            SHORT -> Length.SHORT
            LONG -> Length.LONG
        }
        Toast.makeText(context, message, toastDuration).show()
    }

    enum class MessageLength { SHORT, LONG }
}

val messageDisplay = MessageDisplay(context)
messageDisplay.show("Message")
```

- 클래스가 함수보다 더 강력한 이유는 상태를 가질 수 있으며, 많은 함수를 가질 수 있다는 점
- 의존성 주입 프레임워크를 사용하면 클래스 생성도 위임 할 수 있고, mock 객체를 활용해서 해당 클래스에 의존하는 다른 기능을 테스트할 수 있음

```kotlin
@Inject
lateinit var messageDisplay: MessageDisplay

val messageDisplay: MessageDisplay = mockk()
```

### 인터페이스
- 인터페이스를 사용하면 인터페이스 뒤에 객체를 숨김으로써 실질적인 구현을 추상화 하고, 사용자가 추상화 된것에만 의존하게 만들어서 결합을 줄일 수 있음. 그래서 코틀린의 표준 라이브러리를 보면 거의 모든것이 인터페이스로 표현 되고 있음

```kotlin
interface MessageDisplay {
    fun show(
        message: String,
        duration: MessageLength = LONG
    )
}

class ToastDisplay(val context: Context): MessageDisplay {
    override fun show(
        message: String,
        duration: MessageLength
    ) {
        val toastDuration = when(duration) {
            SHORT -> Length.SHORT
            LONG -> Length.LONG
        }
        Toast.makeText(context, message, toastDuration).show()
    }

        enum class MessageLength { SHORT, LONG }
}

// 인터페이스 페이킹을 이용한 테스트
val messageDisplay: MessageDisplay = TestMessageDisplay()
```

- 인터페이스 페이킹이 클래스 모킹보다 간단하므로 별도의 모킹 라이브러리 없이 테스트가 가능하단것도 장점


### 추상화가 주는 자유
- 추상화를 하는 방법
    * 상수로 추출한다
    * 동작을 함수로 래핑한다
    * 함수를 클래스로 래핑한다
    * 인터페이스 뒤에 클래스를 숨긴다
    * 보편적인 객체를 특수한 객체로 래핑한다

- 추상화를 할때 사용하는 도구
    * 제너릭 타입 파라미터
    * 내부 클래스 추출
    * 생성을 제한(팩토리 함수로만 객체를 생성하게 만드는 등)

### 추상화의 문제
- 어쨋거나 추상화도 비용이 발생하기 때문에 극단적으로 모든것을 추상화 해서는 안됨
- 생각할것을 어느정도 숨겨야 개발이 쉬워지는것도 사실이지만 너무 많은것을 숨기면 결과를 이해하는것 자체가 어려워짐

### 어떻게 균형을 맞춰야 할까
- 많은 개발자가 참여하는 프로젝트는 이후에 객체 생성과 사용 방법을 변경하기가 어려우므로 추상화를 해서 모듈과 부분을 분리하는게 좋음
- 의존성 주입 프레임워크를 사용 한다면 생성이 얼마나 복잡한지는 신경쓰지 않아도 됨
- 테스트를 할것이라면 추상화를 하는게 좋음
- 프로젝트가 작고 실험적이라면 추상화를 하지 않고도 직접 변경해도 괜찮음
