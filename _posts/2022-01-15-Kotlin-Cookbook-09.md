---
layout: post
title: "Kotlin Cookbook - 10장: 입력/출력"
description: Kotlin Cookbook 10장
date: 2022-01-15 16:33:00 +09:00
categories: Kotlin Cookbook Study
---

# 입력/출력

## use로 리소스 관리하기

### 문제
- 파일 같은 리소스를 처리하고 사용을 끝마쳤을떄 확실히 리소스를 닫고 싶다

### 해법
- kotlin.io 패키지의 use 또는 java.io.Reader의 useLines를 사용한다

### 설명
- java는 try-with-resource 구문을 활용해서 리소스 관리를 JVM에게 맡길수있다. 이때 리소스는 Closeable 인터페이스를 구현한 리소스이기만 하면 된다
- 코틀린은 try-with-resource 를 지원하지 않지만 대신 Closeable에 확장함수 use, Reader와 File에는 useLine을 추가했다

```kotlin
inline fun <T> File.useLines(
    charset: Charset = Charsets.UTF_8,
    block: (Sequence<String>) -> T
): T = bufferedReader(charset).use{ block(it.lineSequence()) }

inline fun <T: Closeable?, R> T.use(block: (T) -> R): R
```

- useLine의 첫번째 선택적인자는 문자 집합(기본값은 UTF-8), 두번쨰 인자는 파일의 줄을 타나내는 Sequence를 제너릭 인자 T로 매핑하는 람다. useLines의 함수 구현은 처리를 완료한 다음 리더를 자동으로 닫음
- 위의 구현에서 bufferedReader를 생성하고 BufferedReader의 use 함수에 처리를 위임함
- use 확장함수의 구현은 예외처리로 인해 복잡하지만 핵심적으로는 try-catch의 finally 부분에서 close()를 한다는것이다
- use 블록은 기반 코드가 라이브러리에 내장 되어있고, 제공한 람다가 실제 작업을 수행하는 Execute Around Method 디자인 패턴임

## 파일에 기록 하기

### 문제
- 파일에 기록을 하고싶다

### 해법
- File 확장 함수에는 일반적인 자바 입출력 메소드 뿐만 아니라 출력 스트림과 Writer를 리턴하는 확장 함수가 있다

### 설명
- java.io.File 클래스에 확장함수로 forEachLine을 통해 파일을 순회 하거나 readLines를 호출해 모든 줄이 담긴 컬렉션을 획득 할수 있다
- useLines를 사용해 파일의 줄마다 호출되는 함수를 제공 할수도 있고, readText나 readBytes를 사용해 전체 내용을 각각 문자열이나 바이트 배열로 읽어 올 수 있다
- 파일에 존재하는 내용 모두를 교체하려면 writeText를 써야 한다
- File클래스에는 파일에 데이터를 추가하는 appendText 확장함수가 있는데 writeText나 appendText 는 writeBytes와 appendBytes에 기록 작업을 위힘한다
