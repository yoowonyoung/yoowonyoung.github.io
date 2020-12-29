---
layout: post
title: "Effective Java - 아이템75: 예외의 상세 메시지에 실패 관련 정보를 담으라"
description: 예외의 상세 메시지에 실패 관련 정보를 담으라
date: 2020-09-19 19:54:00 +09:00
categories: EffectiveJava Study
---


# 예외

## 아이템 75 : 예외의 상세 메시지에 실패 관련 정보를 담으라

- 예외를 잡지 못해 프로그램이 실패하면 자바 시스템은 그 예외의 스택 추적정보를 자동으로 출력한다
- 스택 추적은 예외 객체의 toString메서드를 호출해 얻는 문자열로, 보통은 예외 클래스 이름 뒤에 상세 메시지가 붙는 형태이다. 이 정보가 실패 원인을 분석해야하는 프로그래머 혹은 사이트 신뢰성 엔지니어가 얻을 수 있는 유일한 정보인경우가 많다. 더구나 그 실패를 재현하기 어렵다면, 더 자세한 정보를 얻기 어렵거나 불가능 하다
- 예외의 toString 메서드에 실패 원인에 관한 정보를 가능한 많이 담아 반환하는일은 아주 중요하다. 달리 말하면, 사후 분석을 위해 실패 순간의 상황을 정확히 포착해 예외의 상세 메시지에 담아야 한다
- 실패 순간을 포착하려면 발생한 예외에 관여된 모든 매개변수와 필드값을 실패 메시지에 담아야 한다. 예컨데 IndexOutOfBoundsExeption의 상세 메시지는 범위의 최솟값과 최댓값, 그리고 범위를 벗어난 인덱스의 값을 담아야 한다
    * 이 정보는 실패에 관한 많은것을 알려준다. 셋중 한두개 혹은 셋 모두가 잘못 되있을 수 있고, 이 현상들의 원인이 모두 다르므로, 현상을 보면 무엇을 고쳐야 할지를 분석하는데 도움이 된다

- 관련 데이터를 모두 담아야 하지만 장황할 필요는 없다. 문제를 분석하는 사람은 스택 추적뿐만 아니라 관련 문서와 필요하다면 소스코드를 살펴본다. 스택 추적에는 예외가 발생한 파일 이름과 줄번호등의 정보들이 정확히 기록되있는게 보통이다. 그러니 문서와 소스코드에서 얻을 수 있는 정보는 길게 늘어놔봐야 군더더기가 될 뿐이다
- 예외의 상세 메시지와 최종 사용자에게 보여줄 오류 메시지를 혼동해서는 안된다. 최종 사용자에게는 친절한 안내 메시지를 보여줘야 하는 반면, 에외 메시지는 가독성보다는 담긴 내용이 후러씬 중요하다. 예외 메시지의 주 소비층이 문제를 분석해야할 프로그래머와 사이트 신뢰성 엔지니어이기 때문이다
- 실패를 적절히 포착하려면 필요한 정보를 예외 생성자에서 모두 받아서 상세 메시지까지 미리 생성해놓는 방법도 괜찮다. 예를들어 현재의 IndexOutOfBoundsException 생성자는 String을 받지만, 다음과 같이 구현했어도 좋았을것이다

```java
/**
* IndexOutOfBoundsException을 생성한다
*
* @param lowerBound 인덱스의 최솟값
* @param upperBound 인덱스의 최댓값 +1
* @param index 인덱스의 실젯값
*/
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index) {
    super(String.format("최솟값 : %d, 최댓값 : %d, 인덱스 : %d", lowerBound, upperBound, index));
    this.lowerBound = lowerBound;
    this.upperBound = upperBound;
    this.index = index;
}
```

- 자바9 에서는 IndexOutOfBoundsException에 드디어 정수 인덱스 값을 받는 생성자가 추가되었지만, 아쉽게 최솟값과 최댓값까지 받지는 않는다
- 아이템70에서 제안했듯, 예외는 실패와 관련한 정보를 얻을 수 있는 접근자 메서드를 적절히 제공하는것이 좋다. 포착한 실패 정보는 예외 상황을 복구하는데 유용할 수도 있으므로, 접근자 메서드는 비검사 예외보다는 검사 예외에서 더 빛을 발한다
- 하지만 'toString이 반환한 값에 포함된 정보를 얻어 올 수 있는 API를 제공하자' 라는 일반원칙을 따른다는 관점에서, 비검사 예외라도 상세 정보를 알려주는 접근자 메서드를 제공하라고 권하고 싶다