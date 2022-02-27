---
layout: post
title: "Effective Kotlin - 아이템28: API 안정성을 확인하라"
description: API 안정성을 확인하라
date: 2022-02-27 20:58:00 +09:00
categories: EffectiveKotlin Study
---


# 추상화 설계

## 아이템 28 : API 안정성을 확인하라
- 프로그래밍에서 안정적이고 표준적인 API를 선호하는 이유
    * API가 변경되고 개발자가 이를 업데이트 했다면 여러 코드를 수동으로 업데이트 해야함
    * 사용자가 새로운 API를 배워야함

- 좋은 API는 한번에 설계할 수 없기 때문에 API 제작자는 개선을 위해서 변경을 원함. 이때 API 또는 API의 일부가 불안정 하다면 이를 명확하게 알려줘야 하는데 보통 버저닝을 통해서 많이 알려줌
    * 시멘틱 버저닝: MAJOR.MINOR.PATCH 의 구조. MAJOR 호환되지 않는 수준의 API 변경, MINOR는 이전 변경과 호환되는 기능을 추가, PATCH는 간단한 버그 수정
    * MAJOR 버전이 0이라면 안정적일것이라 생각하면 안됨

- 안정적인 API의 일부를 변경해야 한다면 전환하는데 시간을 두고 Deprecated 어노테이션을 활용해 사용자에게 미리 알려줘야 하고, 직접적인 대안이 있다면 IDE가 자동 전환을 할 수 있게 ReplaceWith를 붙여 줘야함

```kotlin
@Deprecated("Use suspending getUsers instead", ReplaceWith("getUsers()"))
fun getUsers(callBack: (List<Users>)-> Unit) {
    // ...
}
```

