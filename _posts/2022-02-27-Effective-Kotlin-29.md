---
layout: post
title: "Effective Kotlin - 아이템29: 외부 API를 Wrap 해서 사용하라"
description: 외부 API를 Wrap 해서 사용하라
date: 2022-02-27 21:05:00 +09:00
categories: EffectiveKotlin Study
---


# 추상화 설계

## 아이템 29: 외부 API를 Wrap 해서 사용하라
- 불안정한 API를 과도하게 사용하는것은 굉장히 위험함. 잠재적으로 불안정하다고 판단되는 외부 라이브러리 API는 Warp 해서 사용 해야 함
- Warp 사용시 이점
    * 문제가 있다면 Warpper만 변경하면 되므로 API 변경에 쉽게 대응
    * 프로젝트의 스타일에 맞춰서 API의 형태 조절 가능
    * 특정 라이브러리에서 문제가 발생하면 Warpper를 수정해서 다른 라이브러리를 사용 하도록 코드를 쉽게 변경 가능
    * 필요한 경우 동작을 추가하거나 수정 가능

- Warp 사용시 단점
    * Warpper를 따로 정의 해야함
    * 다른 개발자가 프로젝트를 다룰 때 어떤 Warpper들이 있는지 따로 확인 해야 함
    * Warpper들은 프로젝트 내부에서만 유효하므로 문제가 생겨도 질문을 할 수 없음

- 장점과 단점을 모두 이해하고 Warp할 API를 결정 해야함

