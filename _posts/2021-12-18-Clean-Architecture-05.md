---
layout: post
title: "Clean Architecture 5장 - 웹 어댑터 구현하기"
description: Clean Architecture 5장 
date: 2021-12-18 13:25:00 +09:00
categories: Clean Architecture Study
---

# 웹 어댑터 구현하기
- 육각형 아키텍처는 도메인 중심의 아키텍처에 적합하기때문에 도메인 엔티티를 만드는것으로 시작한후, 해당 도메인 엔티티를 중심으로 유스케이스를 구현한다

## 의존성 역전
- 웹 어댑터(컨트롤러)는 주도하는 혹은 인커밍 어댑터. 외부로부터 요청을 받아 애플리케이션 코어를 호출하고 무슨일을 해야 할지를 알려줌
- 애플리케이션 계층은 웹 어댑터가 통신할 수 있는 특정 포트를 제공. 서비스는 이 포트를 구현하고 웹 어댑터는 이 포트를 호출
- 이 과정에서 의존성 역전이 적용되어 웹 어댑터가 유스케이스를 직접 호출할 수 있음
- 어댑터와 유스케이스 사이에 또다른 간접 계층이 필요한 이유는, 어플리케이션 코어가 외부 세계와 통신할수 있는 곳에 대한 명세가 포트이기 때문. 포트를 적절한 곳에 위치시키면 외부와 어떤 통신이 일어나고 있는지를 정확히 알 수 있음
- 웹 소켓을 통해 실시간 데이터를 사용자의 브라우저로 보낸다고 한다면, 반드시 의존성을 올바른 방향(서비스 -> 웹 소켓 컨트롤러)로 유지하기 위해서 아웃고잉 포트가 필요하며, 웹 어댑터는 인커밍 어댑터인 동시에 아웃고잉 어댑터가 된다

## 웹 어댑터의 책임
- 웹 어댑터는 일반적으로 다음과 같은 일을 함
    * HTTP 요청을 자바 객체로 매핑
    * 권한 검사
    * 입력 유효성 검증
    * 입력을 유스케이스의 입력 모델로 매핑
    * 유스케이스 호출
    * 유스케이스의 출력을 HTTP로 매핑
    * HTTP 응답을 반환

- 웹 어댑터는 URL, 경로, HTTP 메서드, 콘텐츠 타입과 같이 특정 기준을 만족하는 HTTP 요청을 수신해야 하며, HTTP 요청의 파라미터와 콘텐츠를 객체로 역직렬화 해야 함
- 보통은 웹 어댑터가 인증과 권한 부여를 수행하고 실패할 경우 에러를 반환
- 유스케이스의 유효성 검증(맥락에 유효한 입력인지 아닌지)과 달리 웹 어댑터의 입력 모델을 유스케이스의 입력 모델로 변환할수 있다는것을 검증
- 웹 어댑터는 유스케이스의 출력을 반환받고 HTTP 응답으로 직렬화 해서 호출자에게 전달
- 웹 어댑터의 책임이 많긴 하지만, 이 책임들은 애플리케이션 계층이 신경쓰면 안되는것이기도 함
- 웹 어댑터와 애플리케이션 계층간의 경계는 도메인과 애플리케이션 계층에서 개발하기 시작하면 자연스럽게 생김

## 컨트롤러 나누기
- 컨트롤러는 너무 적은것보다는 너무 많은게 나으며, 각 컨트롤러가 가능한 좁고 다른 컨트롤러와 가능한 적게 공유하는게 좋음

```java
@RestController
@RequiredArgsController
public class SendMoneyController {
    private final SendMoneyUseCase;

    @PostMapping("/accounts/send/{sourceAccountId}/{targetAccountId}/{amount}")
    void sendMoney(
        @PathVariable("sourceAccountId") Long sourceAccountId,
        @PathVariable("targetAccountId") Long targetAccountId,
        @PathVariable("amount") Long amount) {
            
            SendMoneyCommand command = new SendMoneyCommand(
                new AccountId(sourceAccouintId),
                new AccountId(targetAccountId),
                Money.of(amount));
            sendMoneyUseCase.sendMoney(command);
        }
}
```

- 각 연선에 대해서 가급적이면 별도의 패키지 안에서 별도의 컨트롤러를 만드는 방식이 좋음. 가급적 메서드 명과 클래스 명도 유스케이스를 최대한 반영해서 지어야 함
- 각 컨트롤러가 자체의 모델을 가지고 있거나 위의 예시처럼 원시 값을 받아도됨. 이러한 전용 모델 클래스는 컨트롤러 패키지에 private로 선언해 다른곳에서 실수로 재사용되지 않도록 하는게 중요
- 컨트롤러와 서비스명에 대해서도 잘 생각해봐야함. CreateAccount보다는 RegisterAccount같은 이름이 더 나을것임

## 유지보수 가능한 소프트웨어를 만드는데에는 어떻게 도움이 될까
- 웹 어댑터는 HTTP요청을 애플리케이션의 유스케이스에 대한 메서드 호출로 변환하고, 결과를 다시 HTTP로 변환할뿐 어떤 도메인 로직도 수행하지 않아야함
- 애플리케이션 계층은 HTTP에 대한 상세 정보를 노출하지 않음으로써 HTTP와 관련된 작업을 해선 안됨. 이렇게 할 경우 필요하면 웹 어댑터를 다른 어댑터로 쉽게 교체가 가능