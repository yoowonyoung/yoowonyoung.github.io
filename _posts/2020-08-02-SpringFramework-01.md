---
layout: post
title: "Spring Framework"
description: Spring Framework
date: 2020-08-02 16:21:00 +09:00
categories: Spring SpringFramework Study
---


## Spring Framework
- 자바 기반의 오픈소스 애플리케이션 프레임워크
- 엔터프라이즈급 애플리케이션을 개발하기 위한 기능들이 다 있음(보안, 대규모 데이터 처리 등등)

## Spring Framework 특징
- 경량 컨테이너로 자바 객체를 직접 관리
- POJO 방식의 프레임워크
- 제어 반전(IoC : Inversion of Control) 지원
- 의존성 주입(DI : Dependency Injection) 지원
- 관점 지향 프로그래밍(AOP : Aspect Orinted Programming) 지원
- 영속성과 관련된 다양한 서비스 지원, 확장성이 높음

## IoC, DI, DL, AOP
- 기존 애플리케이션 개발에서는 프로그램의 흐름을 제어하는 주체가 애플리케이션 코드
- 프레임워크기반의 개발에서는 프레임워크가 자신의 흐름을 주체하는 주체가 되어 필요할때마다 애플리케이션 코드를 호출해서 사용하는 방식 -> IoC
- 따라서 실행 시점에 클래스 사이의 관계가 형성되기 떄문에, 어떠한것에도 의존하지 않음
- 의존성이 삽입된다는 의미로 의존성 주입(DI)라고 하기도 하는데, IoC는 DI와 DL에 의해 이뤄짐
- DL(Dependency Lookup) : 의존성 검색, 컨테이너에서는 객체들을 관리하기 위해 별도의 저장소에 빈을 저장하는데, 개발자들이 컨테이너에서 제공하는 API를 이용해 사용하고자 하는 빈을 검색
- DI(Dependency Injection) : 각 클래스 사이에 필요로 하는 의존관계 빈 설정 정보를 바탕으로 컨테이너가 자동 연결
- AOP는 핵심 기능과 공통 기능을분리시켜 핵심 로직에 영향을 끼치지 않게 공통 기능을 개발하는 형태. 무분별하게 중복되는 코드를 한곳에 모아 중복되는 코드를 제거함으로 효율적인 유지보수와 재사용성이 극대화

## MVC, MVP, MVVM
- MVC - Model, View, Controller
    * Model - 어플리케이션에서 사용되는 데이터 처리
    * View - UI
    * Controller - 사용자의 입력을 받아 처리하는 부분
    * 사용자 액션이 컨트롤러로 들어오고, 컨트롤러가 액션을 확인해 모델을 업데이트. 그 후 컨트롤러가 모델을 보여줄 뷰를 선택하면, 뷰는 해당 모델을 이용해 화면을 업데이트 하는 시퀀스
    * 가장 단순하여 널리쓰이지만, 모델 - 뷰 사이의 결합도가 높음

- MVP - Model, View, Presenter
    * Presenter - View에서 요청한 정보로 모델을 가공해 다시 View에 전달 해주는 부분
    * 사용자의 액션이 뷰를 통해 들어옴. 뷰는 자신이 보여줄 데이터를 Presenter에 요청하게 되고, Presenter는 다시 모델에 데이터를 요청, 그렇게 모델이 반환한 데이터를 Presenter가 받아 뷰로 돌려줌
    * Presenter와 뷰는 1대1 관계이고, 뷰와 모델 사이의 의존성도 끊어졌지만, 뷰와 Presenter가 높은 의존성을 갖게 됨

- MVVM - Model, View, View Model
    * View Model - 뷰를 표현하기 위해 만든 뷰를 위한 모댈, 뷰를 나타내주기 위한 모델이자 뷰를 나타내기 위해 데이터를 처리하는 부분
    * 사용자의 액션이 뷰를 통해 들어오고, 이는 Command패턴을 통해 View Model에 전달, View Model은 모델에게 데이터를 요청하고, 모델이 반환한 데이터를 가공해서 저장한다. 뷰는 View Model과 데이터 바인딩 되있다
    * Command 패턴과 데이터 바인딩을 통해 뷰와 View Model 사이의 의존성을 없앴고, View Model과 뷰는 1대 n 관계

## Spring boot
- 스프링을 사용하기 쉽게 대부분의 설정이 미리 되어있음
- 단독으로 실행이 가능한 어플리케이션을 내장
- Tomcat, Jetty 등을 내장
- 설정을 위한 XML이 필요 없음
- 기본 설정이 되어있는 starter 컴포넌트들이 제공되며 대부분 자동으로 되어있음



## Ref
- [Spring Framework](https://khj93.tistory.com/entry/Spring-Spring-Framework%EB%9E%80-%EA%B8%B0%EB%B3%B8-%EA%B0%9C%EB%85%90-%ED%95%B5%EC%8B%AC-%EC%A0%95%EB%A6%AC)
- [MVC, MVP, MVVM](https://beomy.tistory.com/43)
  

    