---
layout: post
title: "Kotlin Cookbook - 13장: 코루틴과 구조적 동시성"
description: Kotlin Cookbook 13장
date: 2022-01-16 17:19:00 +09:00
categories: Kotlin Cookbook Study
---

# 코루틴과 구조적 동시성

## 코루틴 빌더 선택하기

### 문제
- 코루틴을 생성하는 올바른 함수를 선택해야 한다

### 해법
- 사용 가능한 빌더 함수중에 하나를 선택 한다

### 설명
- 새 코루틴을 생성 하려면 runBlocking, launch, async중 하나를 사용 해야 하는데, runBlocking은 최상위 이지만, launch와 async는 CoroutineScope의 확장 함수 이다
- 코루틴의 GlobalScope에도 launch와 async가 있는데 이는 시작하는 코루틴이 특정 코루틴 잡에도 할당되지 않고 영구적으로 취소되지 않으면 애플리케이션 전체 라이프사이클에 거렻 실행되므로 반드시 써야할 이유가 없다면 쓰지 말아야 한다
- runBlocking
    * 명령줄 검증, 테스트에 유용
    * 현재 스레드를 블록하고 모든 내부 코루틴이 종료 될때까지 블록
    * ```fun <T> runBlocking(block: suspend CoroutineScope.() -> T): T```
    * runBlocking 자체는 suspend 함수가 아니라서 보통 함수에서 호출 가능
    * runBlocking 은 인자로써 CoroutineScope에 확장 함수로 추가될 suspend 함수를 받고, 이 인자로 받은 함수를 실행하고, 실행한 함수가 리턴하는 값을 리턴

```kotlin
fun main() {
    println("Before Coroutine")
    runBlocking {
        print("Hello")
        delay(200L)
        print("workd!")
    }
    println("After Coroutine")
}
```

- launch
    * 독립된 프로세스를 실행하는 코루틴을 시작하고, 해당 코루틴에서 리턴 받을 필요가 없는 경우
    * launch함수는 CoroutineScpoe의 확장함수 이므로 CoroutineScope사용이 가능한경우에만 사용 가능
    * launch 함수는 코루틴 취소가 필요하면 사용할 수 있는 Job 인스턴스를 리턴
    
    ```kotlin
    fun CoroutineScpoe.launch(
        context: CoroutineContext = EmptyCoroutineContext,
        start: CoroutineStart = CoroutineStart.DEFAULT,
        block: suspend CoroutineScope.() -> Unit
    ): Job
    ```

    * CoroutineContext는 다른 코루틴과 상태를 공유하기 위해서 사용
    * start는 오직 CoroutineStart.DEFAULT/LAZY/AUTOMATIC/UNDISPATCHED만 가능
    * 마지막 파라미터인 람다는 반드시 인자가 없는 suspend 함수여야하고, 아무것도 리턴하면 안된다

```kotlin
fun main() {
    println("Before runBlock")
    runBlocking {
        print("Beofore launch")
        launch {
            print("Hello")
            delay(200L)
            print("workd!")
        }
        print("After launch")
    }
    println("After runBlock")
}
```

- async
    * 값을 리턴 해야 하는 경우 일반적으로 사용하는 async 빌더. async빌더도 CoroutineScope의 확장함수

    ```kotlin
    fun CoroutineScpoe.async(
        context: CoroutineContext = EmptyCoroutineContext,
        start: CoroutineStart = CoroutineStart.DEFAULT,
        block: suspend CoroutineScope.() -> T
    ): Defferred<T>
    ```

    * 파라미터로 제공한 일시 중단 함수가 값을 리턴하면 async 함수가 Deferred(지연된) 인스턴스로 해당 값을 감쌈 - Deferred는 JS의 Promis 또는 자바의 Feature와 비슷한 느낌
    * Deferred에서 알아야할 중요한 함수가 생산된 값을 리턴하기 전에 코루틴이 완료될때까지 기다리는 await

    ```kotlin
    suspend fun add(x: Int, y: Int): Int {
        deplay(Random.nextLong(1000L))
        return x + y
    }

    suspend fun main() = coroutineScope {
        val firstSum = async {
            println(Thread.currentThread().name)
            add(2,2)
        }
        val secondSum = async {
            println(Thread.currentThread().name)
            add(3,4)
        }
        println("Awating...")
        val total = firstSum.await() + secondSum.await()
        println(total)
    }
    ```

    * delay는 코루틴을 실행하고 있는 스레드를 블록하지 않고 코루틴을 대기상태로 만드는 일시중단 함수

- coroutineScope 빌더
    * 함수 종료 전에 포함된 모둔 코루틴이 완료될떄까지 기다리는 일시 중단 함수
    * 메인스레드를 블록하지 않지만, 반드시 suspend 함수의 일부로써 호출 되야함
    * 정의된 영역 안에 코루틴을 사용해야한다는 코루틴의 기본 원칙이 있는데 coroutineScope의 이점은 이런 코루딘 완료 여부를 확인하기 위해 코루틴을 조사해야할 필요가 없다는것

    ```kotlin
    suspend fun <R> coroutineScope(
        block: suspend CoroutineScope.() -> R
    ): R
    ```

    * coroutineScope 함수는 인자가 업속 제너릭 값을 리턴하는 람다를 받음
    * coroutineScope 함수는 suspend 함수기 때문에 반드시 suspend 함수 또는 다른 코루틴에서 호출 되어야 함
    * 일반적으로 coroutineScope로 시작해서 코루틴이 모두 포함된 영역을 설정하고, 결과 블록 안에서 개별 작업을 다루기 위해 launch 또는 async 를 사용 하며, 이후 이 영역은 프로그램 종료 전에 모든 코루틴이 완료 될떄까지 기다리고, 만약 코루틴이 하나라도 실패한다면 나머지 코루틴을 취소 한다


```kotlin
suspend fun main() = coroutineScope {
    for(i in 0 until 10) {
        launch {
            delay(1000L - i * 10)
            print("$i")
        }
    }
}
```

    
## async/await 을 withContext로 변경하기

### 문제
- async로 코루틴을 시작하고 바로 다음에 코루틴이 완료될동안 기다리는 awit 코드를 간소화 하고 싶다

### 해법
- async/awit 조합을 withContext로 변경

### 설명

```kotlin
suspend fun <T> withContext(
    context: CoroutineContext,
    block: suspend CoroutineScope.() -> T
): T
```

- withContext는 주어진 코루틴 컨텍스트와 함께 명시한 suspend 블록을 호출하고, 완료 될때까지 일시정지 한 후에 그 결과를 리턴한다

```kotlin
suspend fun retrivel1(url: String) = coroutineScope {
    async(Dispatcher.IO) {
        println("Retrievel")
        delay(100L)
        "asyncResult"
    }.await()
}

suspend fun retrivel2(url: String) = 
    withContext(Dispatchers.IO) {
        print("Retriving")
        delay(100L)
        "withContextResult"
    }

fun main() = runBlocking<Unit> {
    val result1 = retrivel1("www.mysite.com")
    val result2 = retrivel2("www.mysite.com")
    println(result1)
    println(result2)
}
```

- retrivel1, retrivel2 모두 Dispatchers.IO 디스패처를 사용하기 때문에 두 함수간에 차이점은 async/awit 이냐 withContext 이냐의 차이 뿐이다
- 실제로 intelliJ에서 async다음에 바로 await하는 코드를 보면 withContext로 리팩토링 해준다

## 디스패처 사용하기

### 문제
- I/O 작업 또는 다른 작업을 위한 전용 스레드풀을 사용해야 한다

### 해법
- Dispatchers 클래스에서 적당한 디스패처를 사용 한다

### 설명
- 코루틴은 CoroutineContext타입의 컨택스트 내에서 실행 되는데, 코루틴 컨택스트에는 CoroutineDispatcher 클래스의 인스턴스에 해당하는 코루틴 디스패처가 포함되어 있다
- 이 디스패처는 코루틴이 어떤 스레드 또는 스레드 풀에서 코루틴을 실행 할지를 결정 하며, launch 또는 async 같은 빌더를 사용할 떄 CoroutineContext 선택 파라미터를 통해 디스패처를 명시 할 수 있다
- 코틀린 표준 라이브러리의 내장 디스패쳐는 ```Dispatcher.Default```, ```Dispatchers.IO```, ```Dispatchers.Unconfined``` 이다
- ```Dispatchers.Unconfined``` 는 일반적으로 어플리케이션 코드에서 사용하면 안된다
- 기본 디스패처 사용은 코루틴이 대규모의 계산 리소스를 소모하는 경우에 적합하며, IO 디스패처는 파일 I/O 또는 블로킹 네트워크 I/O 같은 I/O 집약적인 블록킹 작업을 제거하기 위해 성성된 on-demand 공유 풀을 사용 한다
- 두 디스패처 모두 사용하기 쉽다. 필요에 따라 launch, async, withContext의 인자로 넣기만 하면 된다

```kotlin
fun main() = runBlocking<Unit> {
    launchWithIO()
    launchWithDefault()
} 

suspend fun launchWithIO() {
    withContext(Dispatchers.IO) {
        delay(100L)
        println("using dispatchers.IO")
    }
}

suspend fun launchWithDefault() {
    withContext(Dispatchers.Default) {
        delay(100L)
        println("using dispatchers.Default")
    }
}
```

## 자바 스레드 풀에서 코루틴 실행 하기

### 문제
- 코루틴을 사용하는 사용자 정의 스레드 풀을 제공 하고 싶다

### 해법
- 자바 ExcutorService의 asCoroutineDispatcher 함수를 사용

### 설명
- 코틀린 라이브러리는 java.util.concurrent.ExcutorService에 asCoroutineDispathcer 확장 메소드를 추가 했다. 이는 ExcutorService의 인스턴스를 ExcutorCoroutineDispathcer의 구현으로 변환 한다

```kotlin
fun main() = runBlocking<Unit> {
    val dispatcher = Excutors.newFixedThreadPool(10).asCoroutineDispatcher()

    withContext(dispatcher) {
        delay(100L)
        print(Thread.currentThread().name)
    }

    dispatcher.close()
}
```

- ExecutorService는 close를 호출하지 않으면 계속 실행 되므로, 즉 main이 절대 종료 되지 않으므로 항상 close를 호출 해야 한다
- 코틀린 라이브러리 개발자들은 ExcutorCoroutineDispatcher 클래스가 Closeable 인터페이스를 구현하도록 다음과 같이 리팩토링 하였다

```kotlin
abstract class ExecutorCoroutineDispatcher: CoroutineDispatcher(), Closeable {
    abstract override fun close()
    abstract val executor: Executor
}

//하위 클래스(CloserableCoroutineDispatcher) 안에서
override fun close() {
    (executor as? ExecutorService)?.shutdown()
}
```

- 이제 use를 사용해서 다음과 같이 호출 하면 된다

```kotlin
Executors.newFixedThreadPool(10).asCoroutineDispatcher().use {
    withContext(it) {
        delay(100L)
        println(Thread.currentThread().name)
    }
}
```


## 코루틴 취소 하기

### 문제
- 코루틴 내의 비동기 처리를 취소 하고 싶다

### 해법
- launch 또는 withTimeout이나 withTimeoutOrNull같은 함수가 리턴하는 Job 레퍼런스를 사용 한다

### 설명
- launch는 코루틴을 취소하기 위해 사용할 수 있는 Job 타입의 인스턴스를 리턴 한다

```kotlin
fun main() = runBlocking {
    val job = launch {
        repeat(100) { i ->
            print("$i")
            dealy(100L)
        }
    }
    delay(500L)
    job.cancel()
    job.join()// cancel과 join을 결합한 cancelAndJoin도 있음
}
```

- 잡을 취소 하려는 이유가 시간이 너무 오래 걸려서라면 withTimeout함수를 사용할수도 있다

```kotlin
suspend fun<T> withTimeout(
    timeMillis: Long,
    block: suspend CoroutineScope.() -> T
): T
```

- withTimeout은 코루틴안의 suspend 블록을 한번 실행 하고 타임 아웃을 초과하면 TimeoutCalcellationException을 던진다

```kotlin
fun main() = runBlocking {
    withTimeout(1000L) {
        repeat(50) { i ->
            println("$i")
            delay(100L)
        }
    }
}
```

- 원한다면 TimeoutCalcellationException을 캐치 하거나 타임아웃시 예외 대신 null을 리턴하는 withTimeoutOrNull을 사용할수있다

## 코루틴 디버깅

### 문제
- 코루틴의 실행 정보가 더 많이 필요하다

### 해법
- JVM에서 -Dkotlinx.coroutines.debug 플래그를 사용해서 실행

### 설명
- JVM에서 코루틴을 디버그 모드로 실행 하려면 kotlinx.coroutines.debug 속성을 사용 하면 된다
- 디버그 모드는 실행된 모든 코루틴에 고유한 이름을 부여 한다. 각 코루틴에는 스레드 이름의 일부로 보이는 고유한 이름이 있으며, CoroutineName 클래스를 사용해 직접 이름을 부여 할수도 있다

```kotlin
suspend fun rertieve1(url: String) = coroutineScope {
    async(Dispatcher.IO + CoroutineName("async")) {
        // ...
    }.await()
}
```

- 위의 예제 에서는 ```Thread.currentThread().name```을 출력하면 코루틴의 이름으로 ```async```가 출력 됨을 볼 수 있다