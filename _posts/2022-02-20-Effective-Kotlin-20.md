---
layout: post
title: "Effective Kotlin - 아이템20: 일반적인 알고리즘을 반복해서 구현하지 말라"
description: 일반적인 알고리즘을 반복해서 구현하지 말라
date: 2022-02-20 15:55:00 +09:00
categories: EffectiveKotlin Study
---


# 재사용성

## 아이템 20 : 일반적인 알고리즘을 반복해서 구현하지 말라
- 특정 프로젝트에 국한되는 알고리즘이 아닌 수학적인 연산, 수집 처리 처럼 별도의 모듈 또는 라이브러리고 분리 할 수 있는것들이 있음
- 이미 구현된 라이브러리등을 사용하면 다음과 같은 이점이 있음
    * 코드를 작성하는 속도가 빨라짐
    * 구현을 따로 읽지 않아도, 함수의 이름 등만 보고도 무엇을 하는지 확실 하게 알 수 있음
    * 직접 구현할 때 발생할 수 있는 실수를 줄일 수 있음
    * 제작자들이 한번만 최적화 해주면 이러한 함수를 활용하는 모든 곳이 최적화 됨

### 표준 라이브러리
- 일반적인 알고리즘은 stdlib에 모두 구현 되어 있음
- 표준라이브러리에 없는 알고리즘이 필요 하다면 직접 구현 해야 하는데, 범용 유틸리티 함수로 구현 하는게 좋음
- 코틀린 stdlib에 정의된 대부분의 함수는 확장 함수로, 확장 함수는 톱레벨 함수, 프로퍼티 위임, 클래스 등에 비해 다음과 같은 이점이 있음
    * 함수는 상태를 유지 하지 않으므로 행위를 나타내기 좋음. Side Effect가 없는 경우 더 좋음
    * 톱 레벨 함수와 비교해서 확장 함수는 구체적인 타입이 있는 객체에만 사용을 제한 할 수 있으므로 좋음
    * 수정할 객체를 아규먼트로 전달 받아 사용하는 것 보다 확장 리시버로 사용 하는것이 가독성이 좋음
    * 확장 함수는 객체에 정의한 함수보다 객체를 사용할 때 자동 완성 기능 등의 제안이 이뤄지므로 쉽게 찾을 수 있음

