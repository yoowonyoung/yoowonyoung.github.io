---
layout: post
title: "Effective Java - 아이템9: try-finally보다는 tiry-with-resource를 사용하라"
description: try-finally보다는 tiry-with-resource를 사용하라
date: 2020-08-04 21:15:00 +09:00
categories: EffectiveJava Study
---


# 객체의 생성과 파괴

## 아이템 9 : try-finally보다는 tiry-with-resource를 사용하라

- 자바 라이브러리에서는 close메서드를 호출해 직접 닫아줘야 하는 자원이 많은데, 이는 클라이언트가 놓치기 쉬워 예측 할 수 없는 성능 문제로 이어지기도 한다
- 이런 자원중 상당수는 안정망으로 finalizer를 활용 하고 있지만 이는 그리 믿을만 하지 못하다
- 전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 try-finally가 쓰였지만 자원이 둘 이상이라면 이는 너무 지저분해진다
- 이는 try-with-resource로 해결 가능하다. 이 구조를 사용 하려면 해당 자원이 AutoCloseable 인터페이스를 구현 해야 한다
- 자바 라이브러리와 서드파티 라이브러리들의 수많은 클래스와 인터페이스가 이미 AutoCloeable을 구현 하거나 확장 해두었다

```java
static String fistLineOfFile(String path) throws IOException {
    try(Bufferedreader Br = new Bufferedreader(new FileReader(path))) {
        return br.readLine();
    }
}
```

- 이는 복수의 자원을 활용할떄도 유용하다

```java
static void copy(String src, String dist) throws IOException {
    try(InputStream in = new FileInputStream(input);
        OutputStream out = new FileOutputStream(dist)) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while((n = in.read(buf)) >= 0)
                out.write(buf,0,n);
        }
}
```

- try-with-resource 버전이 짧고 읽기 수월 할 뿐 아니라 문제를 전달하기도 훨씬 좋다
- 보통의 try-finally에서 처럼 try-with-resource에서도 catch 절을 쓸 수 있다. catch절 덕분에 try를 더 중첩 하지 않고도 다수의 예뢰를 처리 할 수 있다

```java
static String fistLineOfFile(String path, String defaultVal) throws IOException {
    try(Bufferedreader Br = new Bufferedreader(new FileReader(path))) {
        return br.readLine();
    } catch (IOException e) {
        return defaultVal;
    }
}
```

- 꼭 회수 해야 하는 자원을 다룰때는 try-finally 말고, try-with-resource를 사용하자
- 예외는 없다. 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다
- try-finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도, try-with-resource로는 정확하고 쉽게 자원을 회수 할 수 있다
