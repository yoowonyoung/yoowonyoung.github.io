---
layout: post
title: "Effective Java - 아이템70: 복구할 수 있는 상황에서는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라"
description: 복구할 수 있는 상황에서는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라
date: 2020-09-17 21:58:00 +09:00
categories: EffectiveJava Study
---


# 예외

## 아이템 70 : 복구할 수 있는 상황에서는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라

- 자바는 문제 상황을 알리는 타입으로 검사 예외, 런타임 예외, 에러 이렇게 3가지를 제공하는데, 언제 무엇을 사용해야 하는지 헷갈려 할 수 있다
- 여기에 그럴때 사용할 수 있는 멋진 지침들이 있다
- 호출하는 쪽에서 복구하리라 여겨지는 상황에서는 검사 예외를 사용하라. 이것이 검사와 비검사 예외를 구분하는 기본 규칙이다
- 검사 예외를 던지면 호출자가 그 예외를 catch로 잡아 처리하거나 더 바깥으로 전파하도록 강제하게 된다. 따라서 메서드 선언에 포함된 검사 예외 각각은 그 메서드를 호출 했을때 발생할 수 있는 유력한 결과임을 API사용자에게 알려주는것이다
- 달리 말하면 API설계자는 API 사용자에게 검사예외를 던져주어 그 상황에서 회복해내라고 요구한것이다
- 비검사 throwable은 두가지로, 런타임 예외와 에러이다. 둘다 동작 측면에서는 다르지 않다. 이 둘은 프로그램에서 잡을 필요가 없거나 혹은 통상적으로는 잡지 말아야 한다
- 프로그램에서 비검사 예외나 에러를 던졋다는것은 복구가 불가능 하거나, 더 실행해봐야 득보다는 실이 많다는 뜻이다. 이런 throwable을 잡지 않는 스레드는 적절한 오류 메세지를 내뱉으며 중단된다
- 프로그래밍 오류를 나타낼 때는 런타임 예외를 사용하자
- 런타임 예외의 대부분은 전제 조건을 만족하지 못했을때 발생한다. 전제 조건 위배란 단순히 클라이언트가 해당 API의 명세에 기록된 제약을 지키지 못했다는 뜻이다
    * 예컨대 배열의 인덱스는 0에서 '배열크기 -1'사이 이여야 한다. ArrayIndexOutOfBoundsException은 이 전제조건이 지켜지지 않았다는 뜻이다

- 이상의 조건에서 문제가 한가지 있다면, 복구할 수 있는 상황인지 프로그래밍 오류인지 항상 명확히 구분되는것은 아니라는 사실이다
    * 예를들어 자원 고갈은 말도 안되는 크기의 배열을 할당해 생긴 프로그래밍 오류 일수도 있고, 진짜로 자원이 부족한것일수도 있다
    * 자원이 일시적으로만 부족하거나 수요가 순간적으로만 몰린것이라면 충분히 복구 할 수 있지만, 이러한 판단은 API의 설계자에 달렸다
    * 복구가 가능하다고 믿는다면 검사 예외를, 복구가 불가능 하다고 생각하면 비검사 예외를 선택하는 편이 낫다

- 에러는 보통 JVM이 자원 부족, 불변식 깨짐 등 더이상 수행을 계속할 수 없는 상황을 나타낼때 사용한다. 자바 언어 명사가 요구하는것은 아니지만 업계에 널리 퍼진 규약이니 Error클래스를 상속해 하위 클래스를 만드는일은 자제해야한다. 즉 우리가 구현하는 비검사 throwable은 모두 RuntimeException의 하위 클래스여야 한다
    * 또한 Error는 throw문으로 직접 던지는 일도 없어야 한다(AssertionError는 예외이다)

- Exception, Error, RuntimeException을 상속하지 않는 throwable을 만들 수도 있는데, 자바 언어 명세에서 이런 throwable을 직접 다루지는 않지만, 암묵적으로 일반적인 검사예외(Exception의 하위중 RuntimeException을 상속하지 않은)처럼 다룬다
    * 하지만 이런 throwable은 정상적인 검사예외보다 나을게 하나도 없으면서 API사용자를 헷갈리게 만드므로, 절대 사용하지 말자

- API설계자들도 예외 역시 어떤 메서드라도 정의할수 있는 완벽한 객체 라는 사실을 잊곤 한다. 예외의 메서드는 주로 그 예외를 일으킨 상황에 대한 정보를 코드 형태로 전달하는데 쓰이며, 이런 메서드가 없다면 직접 오류 메시지를 파싱해 정보를 빼내야 하는데, 이는 대단히 나쁜습관이다
    * throwable 클래스들은 대부분 오류 메시지 포맷을 상세히 기술하지 않는데, 이는 JVM이나 릴리즈에 따라 포맷이 달라 질 수 있다는 뜻이다

- 검사 예외는 일반적으로 복구할 수 있는 조건일때 발생한다. 따라서 호출자가 예외 상황에서 벗어나는데 필요한 정보를 알려주는 메서드를 함께 제공하는것이 중요하다
- 결론은 간단하다. 복구할 수 있는 상황이라면 검사 예외를, 프로그래밍 오류라면 비검사 예외, 확실하지 않아도 비검사 예외를 던지며, 검사 예외라면 복구에 필요한 정보를 알려주는 메서드도 함께 제공해야 한다