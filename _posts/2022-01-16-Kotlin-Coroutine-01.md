---
layout: post
title: "Kotlin Coroutine - 1, Hello Coroutine World!"
description: Kotlin Coroutine 1장
date: 2022-01-16 22:00:00 +09:00
categories: Kotlin Coroutine Study
---

# Hello Coroutine World!

## 코루틴
- 코틀린 문서에서는 코루틴을 경량 스레드라고 함. 대부분의 스레드와 마찬가지로 코루틴이 프로세서가 실행할 명령어 집합의 실행을 정의하며, 스레드와 비슷한 라이프 사이클을 갖고 있기 때문
- 스레드와 코루틴이 다른점은 코루틴이 빠르고 적은 비용으로 생성 할 수 있다는 것

```kotlin
suspend fun createCorutines(amount: Int) {
    val jobs = ArrayList<Jobs>()
    for(i int 1..amount) {
        jobs += launch {
            delay(1000)
        }
    }
    jobs.forEach {
        it.join()
    }
}

fun main(args: Array<String>) = runBlocking {
    val time = measureTimeMillis {
        createCoroutines(10000)
    }
    println(time)
}
```

- amount를 증가 시켜도 코틀린은 고정된 크기의 스레드 풀을 사용하고, 코루틴을 스레드들에 배포하기 떄문에 실행 시간은 매우 적게 증가한다
- 코루틴이 일시 중단 되는 동안(이 예제에서는 delay) 실행중인 스레드는 다른 코루틴을 실행하는데 사용되며 코루틴은 시작 또는 재개될 준비 상태가 된다
- 코루틴이 특정 스레드 안에서 실행 되더라도 스레드와 묶이지 않는다는점을 이해해야 한다. 코루틴의 일부를 특정 스레드에서 실행하고, 실행을 중지한 다음 다른 스레드에서 계속 실행하는것이 가능
- 스레드는 한번에 하나의 코루틴만 실행할 수 있기 때문에 프레임워크가 필요에 따라 코루틴을 스레드들 사이에서 옮기는 역할을 함

## 동시성에 대한 이해
- 올바른 동시성 코드는 결정론적 결과를 갖지만 실행 순서에서는 약간의 가변성을 허용하는 코드
- 순차적 코드의 문제점
    * 동시성 코드에 비해 성능이 저하될 가능성
    * 코드가 실행되는 하드웨어를 제대로 활용하지 못할 가능성

```kotlin
//순차적 코드
fun getProfile(id: Int) : Profile {
    val basicUserInfo = getUserInfo(id)
    val contactInfo = getContactInfo(id)

    return createProfile(basicUserInfo, contactInfo)
}


//동시성 코드
suspend fun getProfile(id: Int) {
    val basicUserInfo = asyncGetUserInfo(id)
    val contactInfo = asyncGetContactInfo(id)

    createProfile(basicUserInfo.await(), contactUserInfo.await())
}
```

- asyncGetUserInfo 와 asyncGetContactInfo의 실행이 서로 다른 스레드에서 실행 되고, asyncGetContactInfo의 실행이 asyncGetUserInfo의 완료에 좌우되지 않으므로 각 요청은 동시에 이뤄질수 있기 때문에 동시성 코드
- createProfile을 호출하면서 2개의 await을 함으로써 asyncGetUserInfo와 asyncGetUserInfo가 모두 완료될떄까지 getProfile의 실행을 일시 중단하고, 둘다 완료 됫을때만 createProfile을 수행. 어떤 동시성 호출이 먼저 종료 되던지간에 관계없이 getProfile의 결과가 결정론적임

## 동시성은 병렬성이 아니다
- 동시성은 두개 이상의 알고리즘의 실행 시간이 겹칠때 발생한다. 중첩이 발생하려면 2개 이상의 스레드가 필요하며, 이런 스레드들이 단일 코어에서 실행되면 병렬이 아니라 동시에 실행 되는데, 단일 코어가 서로 다른 스레드의 인스트럭션을 교차 배치 해서 스레드들의 실행을 효율적으로 겹쳐서 실행한다
- 병렬성은 2개의 알고리즘이 정확히 같은 시점에 실행 될때 발생 한다. 이것이 가능 하려면 2개 이상의 코어와 2개 이상의 스레드가 있어야 각 코어가 동시에 스레드의 인스트럭션을 실행할 수 있다
- 병렬은 동시성을 의미 하지만, 동시성은 병렬성이 ㅇ벗이도 발생할 수 있다

## CPU바운드와 I/O 바운드
- 