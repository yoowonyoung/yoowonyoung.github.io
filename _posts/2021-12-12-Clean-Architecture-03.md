---
layout: post
title: "Clean Architecture 3장 - 코드 구성하기"
description: Clean Architecture 3장 
date: 2021-12-12 16:42:00 +09:00
categories: Clean Architecture Study
---

# 코드 구성하기

## 계층으로 구성하기

```
buckpal
|- domain
|   |- Account
|   |- Activity
|   |- AccountRepository
|   |- AccountService
|- persistence
|   |- AccountRepositoryImpl
|- web
    |- AccountController
```

- 코드를 구조화 하는 첫번째 방법으로는 계층을 이용하는것으로, 웹계층, 도메인 계층, 영속성 계층 각각에 대해 전용 패키지인 web, domain, persistence를 두는 형식이 있다
- 하지만 계층으로 구성하는 방식은 적어도 3가지 이유로 최적의 구조가 아니다
- 애플리케이션의 기능 조각이나 특성을 구분짓는 패키지 경계가 없다. 추가적인 구조가 없다면 아주 빠르게 서로 연관되지 않은 기능들끼리 예상하지 못한 부수효과를 일으킬 수 있는 클래스들의 엉망진창 묶음으로 변모 할 수 있다
- 애플리케이션이 어떤 유스케이스들을 제공하는지 파악할 수 없다. 특정 기능을 찾기 위해서는 어떤 서비스가 이를 구현했는지 추측해야 하고, 해당 서비스 내의 어떤 메서드가 그에 대한 책임을 수행하는지 찾아야 한다
- 패키지 구조를 통해서는 우리가 목표로 하는 아키텍처를 파악할 수 없다

## 기능으로 구성하기

```
buckpal
|- account
    |- Account
    |- AccountController
    |- AccountRepository
    |- AccountRepositoryImpl
    |- SendMoneyService
```

- 계좌와 관련된 모든 코드를 최상위의 account 패키지에 넣었는데, 각 기능을 묶은 그룹들은 account와 같은 레벨의 새로운 패키지로 들어가고, 패키지 외부에서 접근하면 안되는 클래스들에 대해 package-private 접근 수준을 통해 패키지간의 경계를 강화한다
- 기능에 의한 패키징 방식은 계층에 의한 패키징 방식보다 아키텍처의 가시성을 훨씬 더 떨어트린다. 어댑터를 나타내는 패키지 명이 없고, 인커밍 포트, 아웃고잉 포트를 확인할 수 없다

## 아키텍처적으로 표현력 있는 패키징 구조

```
buckpal
|- account
    |- adapter
    |   |- in
    |   |   |- web
    |   |       |- AccountController
    |   |- out
    |   |   |- persistence
    |   |       |- AccountPersistenceAdapter
    |   |       |- SpringDataAccountRepository
    |- domain
    |   |- Account
    |   |- Activity
    |- Application
        |- SendMoneyService
        |- port
           |- in
           |   |- SendMoneyUseCase
           |- out
                |- LoadAccountPort
                |- UpdateAccountStatePort
```

- 구조의 각 요소들은 패키지에 하나씩 직접 매핑된다. 최상위에는 Account와 관련된 유스케이스를 구현한 모듈임을 나타내는 account패키지가 있다
- 그다음 레벨에는 도메인 모델이 속한 domain 패키지, 도메인 모델을 둘러싼 서비스 계층을 포함하는 application패키지 등이 있다
- SendMoneyService는 인커민 포트 인터페이스인 SendMoneyUseCase를 구현하고, 아웃고잉 포트이자 영속성 어댑터에 의해 구현된 LoadAccountPort와 UpdateAccountStatePort를 사용한다
- adapter 패키지는 어플리케이션 계층의 인커밍 포트를 호출하는 인커밍 어댑터와, 어플리케이션 계층의 아웃고잉 포트에 대한 구현을 제공하는 아웃고잉 어댑터를 포함한다
- 이런 패키지 구조는 이른바 아키텍처 코드 갭(모델 코드 갭)을 효과적으로 다룰 수 있는데, 아키텍처가 코드에 직접적으로 매핑될 수 없는 추상적 개념이며, 패키지 구조가 아키텍터를 반영할 수 없다면 시간이 지남에 따라 코든느 점점 목표하던 아키텍처로부터 더 멀어지게 된다
- 패키지가 아주 많기 때문에 모든것을 public으로 만들어 패키지간의 접근을 허용해야 한다고 생각할수도 있지만, application 패키지와 domain 패키지내 일부 클래스들(어댑터에서 접근해야하는 포트들과, 도메인 클래스들은 서비스, 어댑터에서 접근이 가능해야하기 떄문)을 제외하고는 아니다
- 이 패키지 구조의 또다른 장점은 DDD 개념에 직접적으로 대응할 수 있다는 것인데, account와 같은 상위레벨 패키지가 다른 바운디드 컨텍스트(하나의 도메인 모델이 적용되는 범위)와 통신할 전용 진입점과 출구를 포함하는 바운디드 컨텍스트이고, domain 패키지 내에서는 DDD가 제공하는 모든 도구를 이용해 우리가원하는 어떤 도메인 모델이든지 만들 수 있다

## 의존성 주입의 역할
- 클린아키텍처의 가장 본질적인 요건은 애플리케이션 계층이 인커밍, 아웃고잉 어댑터에 의존을 갖지 않아야 하는데, 예제의 인커밍 어댑터는 제어의 흐름 방향이 어댑터와 도메인 코드간의 의존성 방향과 같은방향이라 어렵다. 이때 의존성 역전의 원칙을 사용해야한다
- 어플리케이션 계층에 인터페이스를 만들고, 어댑터에 해당 인터페이스를 구현한 클래스를 두면 되는데, 포트 인터페이스를 구현한 실제 객체를 직접 어플리케이션 계층 안에서 초기화 하고싶지 않으므로 의존성 주입을 하면 된다

## 유지보수 가능한 소프트웨어를 만드는데 어떻게 도움이 될까
- 실제 코드 구조를 우리가 목표로하는 헥사고날 아키텍처에 가깝게 만들어주었으므로, 코드에서 아키텍처의 특정 요소를 찾으려면 아키텍처 다이어그램의 박스 이름을 따라 패키지 구조를 탐색하면 된다. 이로써 의사소통, 개발, 유지보수가 모두 수월해진다