---
layout: post
title: "Effective Kotlin - 아이템9: use를 사용하여 리소스를 닫아라"
description: use를 사용하여 리소스를 닫아라
date: 2022-02-18 22:15:00 +09:00
categories: EffectiveKotlin Study
---


# 안정성

## 아이템 9 : use를 사용하여 리소스를 닫아라
- 더 이상 필요 하지 않을 때 close를 사용 해서 명시적으로 닫아야 하는 리소스 들이 있음 (AutoCloseable을 상속 받는 Closeable 인터페이스를 구현하는 것들)
    * InputStream / OutputStream
    * java.sql.Connection
    * java.io.Reader(FileReader, BufferedReader, CSSParser)
    * java.new.Socket 과 java.util.Sacnner

- 가비지 컬렉터가 이것들을 처리 하지만, 굉장히 느리며 쉽게 처리 되지 않으므로 명시적으로 close 메서드를 호출 해주는 것이 좋음
- 전통적으로 try-catch-finally를 이용 하지만 닫는 도중에 예외가 발생 한다면 이러한 예외는 따로 처리 되지 않으므로 표준 라이브러리에 있는 use를 사용하는게 좋음

```kotlin
fun countCharacterInFile(path: String): Int {
    val reader = BufferedReader(FileReader(path))
    reader.use {
        return reader.lineSequence().sumBy { it.length }
    }
}

// 이렇게도 가능
fun countCharacterInFile(path: String): Int {
    BufferedReader(FileReader(path)).use { reader ->
        return reader.lineSequence().sumBy { it.length }
    }
}

// 파일을 한줄씩 처리 할때는 이렇게도 가능
fun countCharacterInFile(path: String): Int {
    File(path).useLines{ lines ->
        return lines.sumBy { it.length }
    }
}
```



