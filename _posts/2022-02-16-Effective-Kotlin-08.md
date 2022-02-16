---
layout: post
title: "Effective Kotlin - 아이템8: 적절하게 null을 처리 하라"
description: 적절하게 null을 처리 하라
date: 2022-02-16 22:10:00 +09:00
categories: EffectiveKotlin Study
---


# 안정성

## 아이템 8 : 적절하게 null을 처리 하라
- null은 값이 부족하다(lack of value)를 말하는것. 프로퍼티가 null이라는것은 값이 설정되지 않았거나 제거 되었다는것
- null은 최대한 명확한 의미를 가져야 nullable 값을 처리 하기가 쉬움
- nullable 타입을 처리하는 3가지 방법
    * ?./스마트 캐스팅/엘비스 연산자등을 활용해서 언전하게 처리
    * 오류를 throw
    * 함수 또는 프로퍼티를 리팩터링 해서 nullable 타입이 나오지 않게 바꿈


### null을 안전하게 처리

```kotlin
printer?.print() // 안전 호출
if(printer != null) printer.print() // 스마트 캐스팅

// 엘비스 연산자 사용
val printerName1 = printer?.name ?: "Unnamed"
val printerName2 = printer?.name ?: return
val printerName3 = printer?.name ?: throw Error("Printer must be named")
```

- 많은 객체가 nullable과 관련된 처리를 지원 하는데, 예를 들어 컬렉션 처리를 할 때 무언가 없다는것을 나타낼때는 null이 아닌 빈 컬렉션을 사용 하는것이 일반적
- 스마트 캐스팅은 코틀린의 규약 기능(constrants feature)를 지원 하는데, 이를 이용 하면 다음과 같이 스마트캐스팅 가능

```kotlin
println("what is yoir name?")
val name = readLine()
if(!name.isNullOrBlank()) {
    println("Hello ${name.toUpperCase()}")
}
```

### 오류 throw 하기
- 오류를 강제로 발생 시킬때에는, throw, !!, requireNotNull, checkNotNull등을 활용 

```kotlin
fun process(user: User) {
    requireNotNull(user.name)
    val context = checkNotNull(context)
    val networkService =
        getNetworkService(context) ?:
        throw NoInternetConnection()
    networkService.getData { data, userData -> 
        show(data!!, userData!!)
    }
}
```

### not-null assertion(!!) 과 관련된 문제
- !! 을 사용하면 자바에서 nullable을 처리 할 떄 발생 할 수 있는 문제가 똑같이 발생. 어떤 대상이 Null이 아니라고 생각 하고 다루면 NPE 발생. 변수를 null로 설정하고 이후에 !!을 사용하는 방법은 좋은 방법이 아님
- !! 을 사용하는것 보다는 명시적으로 예외를 발생시켜 제너릭 NPE보다 많은 정보를 주는게 나음
- !! 연산자가 의미를 가지는 경우는 굉장히 드물고, 일반적으로 nullabliity 가 제대로 표현되지 않는 라이브러리를 사용할 때 정도에만 사용 해야함
- 일반적으로 !! 연산자 사용은 피해야함. !!를 보면 반드시 조심하고 무언가 잘못되있을 가능성을 생각 해야 함

### 의미 없는 nullability 피하기
- 필요한 경우가 아니라면 nullability 자체를 피하는것이 좋음
- nullability를 피하는 방법들
    * 클래스에서 nullability에 따라 여러 함수를 만들어 제공 (ex> get / getOrNull)
    * 어떤 값이 클래스 생성 이후에 확실하게 설정된다는 보장이 있다면 lateinit 프로퍼티와 notNull 델리게이트를 사용
    * 빈 컬렉션 대신 null을 리턴하지 마라. 컬렉션을 빈 컬렉션으로 둘때와 Null로 둘떄는 의미가 완전히 다르다
    * nullable enum과 None enum은 다른 의미. null enum은 별도로 처리 해야 하지만 None enum은 정의에 없으므로 필요한 경우 사용하는 쪽에서 추가해서 활용 할 수 있다는 의미

### lateinit 프로퍼티와 notNull 델리게이트
- lateinit 한정자는 프로퍼티가 이후에 설정될것임을 명시하는 한정자. 처음 사용하기 전에 반드시 초기화 되어 있을 경우에만 lateinit을 붙이면 됨
- lateinit와 nullable의 차이
    * !! 연산자로 Unpack 하지 않아도 됨
    * 이후에 어떤 의미를 나타내기 위해서 null을 사용 하고 싶을때 nullable로 만들 수 있음
    * 프로퍼티가 초기화 된 이후에는 초기화 되지 않은 상태로 돌아갈 수 없음

```kotlin
class UserControllerTet {
    private lateinit var dao: UserDao
    private lateinit var controller: UserController

    @BeforeEach
    fun init() {
        dao = mockk()
        controller = userController(dao)
    }
}
```

- lateinit를 사용할 수 없는 경우도 있음. JVM에서 Int, Long, Double, Boolean과 같은 기본 타입과 연결된 타입으로 프로퍼티를 초기화 해야 하는 경우. 이런 경우는 Delegates.notNull을 사용 해야 함

```kotlin
class DoctorActivity: Activity() {
    private var doctorId: Int by Delegates.notNull()
    private var fromNotification: Boolean By Delegates.notNull()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        doctorId = intent.extras.getInt(DOCTOR_ID_ARG)
        fromNotification = intent.extras.getBoolean(FROM_NOTIFICATION_ARG)
    }
}
```

- onCreate때 초기화 하는 프로퍼티는 지연 초기화 하는 형태로 다음과 같이 프로퍼티 위임도 가능

```kotlin
class DoctorActivity: Activity() {
    private var doctorId: Int by arg(DOCTOR_ID_ARG)
    private var fromNotification: Boolean By arg(FROM_NOTIFICATION_ARG)
}
```