---
layout: post
title: "Effective Kotlin - 아이템6: 사용자 정의 오류 보다는 표준 오류를 사용 하라"
description: 사용자 정의 오류 보다는 표준 오류를 사용 하라
date: 2022-02-14 21:35:00 +09:00
categories: EffectiveKotlin Study
---


# 안정성

## 아이템 6 : 사용자 정의 오류 보다는 표준 오류를 사용 하라
- 가능한 직접 오류를 정의 하는 것보다 최대한 표준 오류를 사용하는것이 좋음
- 자주 사용 되는 예외 목록
    * IllegalArgumentExeption / IllegalStateException: require / check 등을 통해 throw
    * IndexOutOfBoundsException: 인덱스 파라미터 값이 범위를 벗어남. 컬렉션 또는 배열과 함께 사용
    * ConcurrentModificationException: 동시 수정(Concurrent Modification)을 금지 했는데 발생
    * UnsupportedOperationException: 사용자가 사용 하려고 했던 메서드가 현재 객체에서는 사용 할 수 없음 (인터페이스 분리 원칙을 위반)
    * NoSuchElementException: 사용자가 사용 하려고 했던 요소가 존재하지 않음