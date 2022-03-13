---
layout: post
title: "Effective Kotlin - 아이템52: mutable 컬렉션 사용을 고려하라"
description: mutable 컬렉션 사용을 고려하라
date: 2022-03-13 15:50:00 +09:00
categories: EffectiveKotlin Study
---


# 효율적인 컬렉션 처리

## 아이템 52: mutable 컬렉션 사용을 고려하라

- immutable 컬렉션보다 mutable 컬렉션이 좋은점은 성능적인 측면에서 더 빠르다는것
- immutable 컬렉션에 요소를 새로 추가하려면 새로운 컬렉션을 만들면서 여기에 요소를 추가해야 하는데, 이때 컬렉션을 복제 처리 하는 비용은 굉장히 많이 드므로 복제 처리를 하지 않는 mutable 컬렉션이 성능적 관점에서 좋음
- immutable 컬렉션은 안전하다는 측면에서 좋지만, 지역변수로 사용할때는 mutable 컬렉션이 더 합리적이라고 할 수 있음
- 표준 라이브러리도 내부적으로 어떤 처리를 할때는 mutable 컬렉션을 사용하도록 구현되어 있음
