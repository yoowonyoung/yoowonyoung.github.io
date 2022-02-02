---
layout: post
title: "Kotlin Cookbook - 12장: 스프링 프레임워크"
description: Kotlin Cookbook 12장
date: 2022-01-16 16:44:00 +09:00
categories: Kotlin Cookbook Study
---

# 스프링 프레임워크

## 확장을 위해 스프링 관리 빈 클래스 오픈하기

### 문제
- 스프링은 개발자가 작성한 클래스를 확장하는 프록시를 생성해야 하지만, 코틀린의 클래스들은 기본으로 final 이다

### 해법
- 스프링 관리 클래스를 열어주는 스프링 플러그인을 사용

### 설명
- 스프링은 시동 과정에서 프록시를 생성하는데, 실체가 클래스라면 해당 클래스를 확장해야 하는데 이 과정에서 코틀린이 문제가 된다
- 코틀린은 기본으로 정적으로 결합한다. 즉 클래스가 open 키워드를 사용해 확장을 위한 열림으로 표시되지 않으면 메소드 재정의 또는 클래스 확장이 불가능 하다
- 코틀린은 이런 문제를 all-open 플러그인으로 해결 하는데, 이 플러그인은 클래스와 클래스에 포함된 함수에 명시적으로 open 키워드를 추가하지 않고 명시적인 open 어노테이션으로 클래스를 생성 한다
- all-open 플러그인도 유용 하지만, 좀 더 좋은 kotlin-spring 플러그인도 있다. 이것을 사용해야 한다
- kotlin-spring 플러그인은 ```@Component, @Async, @Transactional, @Cacheable, @SpringBootTest``` 스프링 어노테이션이 붙은 클래스들을 자동으로 open 시켜 준다

## 코틀린 data 클래스로 퍼시스턴스 구현 하기

### 문제
- 코틀린 data 클래스로 JPA를 사용 하고 싶다

### 해법
- kotlin-jpa 플러그인을 빌드 파일에 추가 한다

### 설명
- JPA 관점에서 data클래스는 2가지 문제가 있다. 첫번째는 JPA는 모든 속성에 기본값을 제공하지 않는 이상 기본 생성자가 필수지만 data클래스는 기본 생성자가 없으며, val 속성과 함꼐 data클래스를 생성하면 불변 객체가 되는데 JPA는 불변 객체와 잘 동작하지 않도록 설계 되있다는 것
- 기본 생성자 문제는 no-arg 플러그인으로 해결 할 수 있다. 해당 플러그인은 코틀린 엔티티에 기본 생성자를 자동으로 구성한다
- no-arg 플러그인보다 kotlin-jpa 플러그인이 더 쓰기 쉬운데, kotlin-jpa 플러그인은 ```@Entity, @Embeddable, @MappedSuperClass``` 어노테이션이 달린 클래스에 자동으로 기본 생성자를 추가 해준다
- 불변 클래스 문제는 엔티티로 사용하고 싶은 코틀린 클래스에 data 클래스 대신 필드 값을 변경 할 수 있게 속성에 var 타입을 사용하는 단순 클래스를 사용하는것이 좋다
- 광범위한 var 사용과 자동으로 생성되는 toString, equals, hashCode의 부재는 불편하지만, 이 방식이 JPA가 요구하는 방식에 더 적합한 접근법이다

## 의존성 주입하기

### 문제
- 오토와이어링이 필요한 빈과 필요핮 ㅣ않는 빈을 선언 하고 의존성 주입을 사용해서 빈을 서로 오토와이어링 하고 싶다

### 해법
- 코틀린 스프링은 생성자 주입을 제공 하지만, 필드 주입에는 lateinit var를 사용 해야 한다. 선택적인 빈은 널 허용 타입으로 선언해야 한다

### 설명
- 스프링은 가능하다면 의존성을 생성자로 주입하는것을 선호하는데, 코틀린에서 역시 생성자 인자에 ```@Autowired``` 어노테이션을 사용 할 수 있고, 클래스에서 생성자가 하나뿐이라면 스프링이 자동으로 클래스의 유일한 생성자에 모든 인자를 오토와이어링 하기 떄문에 ```@Autowired``` 어노테이션을 쓸 필요가 없다
- val 속성은 선언시에 값이 반드시 있어야 하기 때문에 값을 나중에 제공해 초기화 할 수 없어 lateinit 은 항상 var와 함께 가팅 쓰여야 한다. var는 차후에 언제든지 값이 바뀔수 있기 때문에 생성자 주입이 더 선호된다
- 클래스의 속성이 필수가 나이라면 해당 속성을 널 허용으로 선언하면 된다