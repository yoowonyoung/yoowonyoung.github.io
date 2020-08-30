---
layout: post
title: "Effective Java - 아이템39: 명명패턴 보다 어노테이션을 활용하라"
description: 명명패턴 보다 어노테이션을 활용하라
date: 2020-08-30 17:55:00 +09:00
categories: EffectiveJava Study
---


# 열거타입과 어노테이션

## 아이템 39 : 명명패턴 보다 어노테이션을 활용하라

- 전통적으로 도구나 프레임워크가 특별히 다뤄야할 프로그램 요소에는 구분되는 명명 패턴을 사용하였다
    * JUnit은 테스트 메서드 이름이 test로 시작해야 한다

- 이러한 방법은 효과적이지만 단점도 크다. 오타가 발생하면 안되며, 올바른 프로그램 요소에서만 사용되리란 보장이 없기 때문이다
- 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다는것도 문제인데, 예를들면 특정 예외를 던져야 성공하는 테스트일 경우, 기대하는 예외 타입을 매개변수로 전달해야 하는데 이때 적절한 방법이 없다
- 어노테이션은 이 모든 문제를 해결해주는 멋진 개념으로 JUnit 역시 이를 사용하며, 이 아이템에서는 어노테이션의 동작 방식을 보여준다

```java
/**
* 매개변수 없는 정적 메서드 전용이다
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```

- 위의 예시는 테스트용 어노테이션인 ```@Test```이다
- 보다시피 ```@Test``` 어노테이션 타입 자체에도 ```@Retention``` 과 ```@Target```의  두가지 다른 어노테이션이 달려있는데, 이를 메타 어노테이션이라고 한다
    * ```@Retention```은 ```@Test```가 런타임에도 유지되어야 한다는 표시이며, 이를 생략하면 테스트 도구는 ```@Test```를 인식할수없다
    * ```@Target```은 ```@Test```가 반드시 메서드 선언에서만 사용되야 한다고 알려주며, 클래스 선언이나 필드선언에는 달수가 없다

- javax.annotation.processiong API 문서를 참고하면, 위 주석에서 명시한것 처럼 여러 제약을 컴파일러가 강제할수 있는것을 알 수 있다

```java
public class Sample {
    @Test
    public static void m1{}
    public static void m2{}
    @Test
    public static void m3() {
        throw new RuntimeException("Error");
    }
    public static void m4() {
        throw new RuntimeException("Error");
    }
    @Test
    public void m5(){}
}
```

- 위의 코드는 ```@Test``` 어노테이션을 실제로 적용한 모습으로써, 이와같이 아무 매개변수 없이 대상에 마킹한다는 뜻에서 마커 어노테이션이라 부른다
- 위 코드에서 어노테이션이 붙지않은 m2는 테스트 도구가 무시할것이며, m5는 정적 메서드가 아닌데 사용한것으로 잘못 사용한것이다
- ```@Test``` 어노테이션이 Sample 클래스의 의미에 직접적인 영향을 주지는 않지만, 추가 정보를 제공해주는 것으로, 대상의 코드의 의미는 그대로 둔채 그 어노테이션에 관심있는 도구에서 특별히 처리할 기회를 준다

```java
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for(Method m : testClass.getDeclaredMethods()) {
            if(m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch(InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println("failed");
                } catch(Exception exc) {
                    e.printStackTrace();
                    System.out.println("Invalid use");
                }
            }
        }
        System.out.printf("passed %d fail %d",passed, tests-passed);
    }
}
```

- 위의 테스트 러너는 정규화된 클래스 이름을 받아 그 클래스에서 ```@Test``` 어노테이션이 달린 메서드를 차례로 호출한다
    * isAnnodationPresent가 실행할 메서드를 찾아준다
    * 리플렉션 메커니즘이 발생한 예외를 InvocationTargetException으로 감싸서 던져주기 때문에, 프로그램은 InvocationTargetException을 잡아 원래 예외에 담신 실패 정보를 추출해야 한다
    * InvocationTargetException 이외에 다른 예외를 던질경우 ```@Test```를 잘못 사용했다는 뜻이다

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```

- 특정 예외를 던져야만 성공하는 테스트를 지원하기 위해 새로이 ```@ExceptionTest```를 선언하였다
- 이 어노테이션의 매개변수 타입은 Class<? extends Throwable>인데, 여기서의 와일드카드의 의미는 "Thworable을 확장한 클래스의 Class 객체"라는 의미로, 모든 예외와 오류 타입을 수용한다

```java
public class Sample2 {
    @ExceptionTest(ArithmeticException.class)
    public static void m1() { // 성공
        int i = 0;
        i = i / i;
    }

    @ExceptionTest(ArithmeticException.class)
    public static void m2() { // 실패 - 다른 Excpeion
        int[] a = new int[0];
        int i = a[1];
    }

    @ExceptionTest(ArithmeticException.class)
    public static void m3() { // 실패 - Exception이 안던져짐
    }
}
```

- 이제 새로운 어노테이션을 다룰수있도록 다음과 같이 테스트 도구를 수정해야 한다

```java
if(m.isAnnotationPresent(Test.class)) {
    tests++;
    try {
        m.invoke(null);
        System.out.println("fail - no exception throwed");
    } catch(InvocationTargetException wrappedExc) {
        Throwable exc = wrappedExc.getCause();
        Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.clas).value();
        if(excType.isInstance(exc)) passed++;
        else System.out.println("fail exception not match");
    } catch(Exception exc) {
        e.printStackTrace();
        System.out.println("Invalid use");
    }
}
```

- ```@Test``` 어노테이션용 에러코드와 비슷한데, 한가지 차이는 이 코드는 어노테이션 매개변수의 값을 추출해 테스트 메서드가 올바른 예외를 던지는지 확인한다
    * 형변환 코드는 없으니 ClassCastExcption은 걱정하지 않아도 된다

- 여기서 한걸음 더들어가서, 예외를 여러개 명시하고 그중 하나가 발생하면 성공하게 만들수 있다
- 이때는 매개변수 타입을 Class객체의 배열로 만들고 아래와 같이 수정하면 된다

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable>[] value();
}

@ExceptionTest({IndexOutOfBoundException.class, NullPointerException.class})
public static void doubleyBad() {
    List<String> list = new ArrayList<>();
    list.add(5,null);
}
```

```java
if(m.isAnnotationPresent(Test.class)) {
    tests++;
    try {
        m.invoke(null);
        System.out.println("fail - no exception throwed");
    } catch(InvocationTargetException wrappedExc) {
        Throwable exc = wrappedExc.getCause();
        int oldPassed = passed;
        Class<? extends Throwable>[] excTypes = m.getAnnotation(ExceptionTest.clas).value();
        for(Class<? extends Throwable> excType : excTypes) {
            if(excType.issInstance(exc)) {
                passed++;
                break;
            }
        }
        if(passed = oldPassed) System.out.printf("Test %s fail %s %n",m,exec);
    } catch(Exception exc) {
        e.printStackTrace();
        System.out.println("Invalid use");
    }
}
```

- 배열 매개변수를 받는 어노테이션용 문법은 아주 유연하여서, 단일 원소 배열에 최적화 되어있지만 기존의 코드를 수정없이 수용한다
- 자바8에서는 여러개의 값을 받는 어노테이션을 다른 방식으로도 만들 수 있는데, 배열 매개변수를 사용하는 대신 어노테이션에 ```@Repeatable``` 메타 어노테이션을 추가하는 방식이다
- ```@Repeatable```을 단 어노테이션은 하나의 프로그램 요소에 여러번 달 수 있지만, 다음과 같은 주의사항이 있다
    * ```@Repeatable``` 어노테이션을 단 어노테이션을 반환하는 컨테이너 어노테이션을 하나 더 정의하고, ```@Repeatable```에 이 컨테이너 어노테이션의 class객체를 매개변수로 전달해야 한다
    * 컨테이너 어노테이션은 내부에서 어노테이션 타입의 배열을 반환하는 value 메서드를 정의해야한다
    * 컨테이너 어노테이션 타입에는 적절한 ```@Retention``` 과 ```@Target```을 정의해야 한다

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable>[] value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] values();
}

@ExceptionTest(IndexOutOfBoundsException.class)
@ExcpetionTest(NullPointerException.class)
public static void doublyBad() { ... }
```

- 반복 가능 어노테이션은 처리 항때에도 주의를 요하는데, 반복 가능 어노테이션을 여러개 달면 하나만 달았을때와 구분하기 위해 해당 컨테이너 어노테이션이 사용되게 된다
- getAnnotationsByType 메서드는 이 둘을 구분하지 않아서 반복 가능 어노테이션과 그 컨테이너 어노테이션을 모두 가져오지만 isAnnotationPresent는 이 둘을 명확히 구분하기 때문에, 어노테이션을 여러개를 달고 isAnnotationPresent로 검사하게 되면 문제가 발생하므로 컨테이너 어노테이션과, 어노테이션을 구분해서 검사해야 한다

```java
if(m.isAnnotationPresent(Exception.class) || m.isAnnotationPresent(ExceptionTestContainer.class))
```

- 반복 가능 어노테이션을 이용해 하나의 프로그램 요소에 같은 어노테이션을 여러개 사용할때의 가독성을 올렸지만, 어노테이션을 선언하고 처리하는 부분의 코드량이 늘어나고 오류의 가능성이 높으므로 주의해야한다
- 어노테이션으로 할수 있는 일을 명명 패턴으로 처리할 이유는 없다
- 도구 제작자가 아닌 일반 프로그래머가 어노테이션을 직접 작성할일은 거의없지만, 자바 개발자라면 예외없이 자바가 제공하는 어노테이션 타입들을 사용해야 한다