---
layout: post
title: "Effective Kotlin - 아이템31: 문서로 규약을 정의하라"
description: 문서로 규약을 정의하라
date: 2022-02-27 21:30:00 +09:00
categories: EffectiveKotlin Study
---


# 추상화 설계

## 아이템 31: 문서로 규약을 정의하라
- 일반적인 문제는 행위가 문서화 되지 않고 요소의 이름이 명확하지 않다면, 이를 사용하는 사용자는 우리가 만들려고 했던 추상화 목표가 아닌 현재 구현에만 의존 하게 됨. 이러한 문제는 예상되는 행위를 문서로 설명함으로써 해결

### 규약
- 어떤 행위를 설명하면 사용자는 이를 일종의 약속으로 취급하며 이를 기반으로 스스로 자유롭게 생각하던 예측을 조정함. 이러한 예측되는 행위를 요소의 규약 이라고 부름
- 규약이 적절하게 정의되어 있다면 클래스를 만든 사람은 클래스가 어떻게 사용될지 걱정하지 않아도 됨

### 규약 정의하기
- 이름: 일반적인 개념과 관련된 메서드는 이름 만으로 동작을 예측할 수있음
- 주석과 문서: 필요한 모든 규약을 적을 수 있는 강력한 방법
- 타입: 타입은 객체에 대한 많은것을 알려줌. 어떤 함수의 선언에 있는 리턴 타입과 아규먼트 타입은 굉장히 큰 의미가 있음

### 주석을 써야할까?
- 주석을 사용하면 함수 또는 클래스에 더 많은 내용의 규약을 설명할 수 있음
- 현대의 주석은 문서를 자동생성하는데에 많이 도움이 됨
- 물론 주석을 다는것 보다는 함수로써 추출하는것이 훨씬 좋음

### 타입 시스템과 예측
- 클래스가 어떤 동작을 할것이라 예측 되면 그 서브클래스도 이를 보장해야함(리스코프 치환의 원칙)
- 사용자가 클래스의 동작을 확실하게 예측할 수 있게 하려면 공개 함수에 대한 규약을 잘 지정 해야하는데, 이때 이러한 규약은 KDoc등으로 적어주면 좋음

### 조금씩 달라지는 세부사항
- 구현의 세부사항은 항상 달라질수 있지만 최대한 많이 보호하는게 좋고, 일반적으로 캡슐화를 통해서 이를 보호함
- 캡슐화는 허용되는 범위를 지정하는데 도움을 주기 때문에, 캡슐화가 많이 적용될수록 사용자가 구현에 신경을 많이 쓸 필요가 없어지므로 더 많은 자유를 가지게 됨
