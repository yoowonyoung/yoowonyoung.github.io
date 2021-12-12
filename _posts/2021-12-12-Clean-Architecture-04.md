---
layout: post
title: "Clean Architecture 4장 - 유스케이스 구현하기"
description: Clean Architecture 4장 
date: 2021-12-12 17:12:00 +09:00
categories: Clean Architecture Study
---

# 유스케이스 구현하기
- 육각형 아키텍처는 도메인 중심의 아키텍처에 적합하기때문에 도메인 엔티티를 만드는것으로 시작한후, 해당 도메인 엔티티를 중심으로 유스케이스를 구현한다

## 도메인 모델 구현하기

```java
package buckpak.domain;

public class Account {
    private AccountId id;
    private Money baselineBalance;
    private ActivityWindow activityWindow;

    // constructor & setter 생략

    public Money calculateBalance() {
        return Money.add(
            this.baselineBalance,
            this.activityWindow.calculateBalance(this.id);
        )
    }

    public boolean withdraw(Money money, AccountId targetAccountId) {
        if(!mayWithdraw(money)) {
            return false;
        }

        Activity withdrawal = new Activity(
            this.id,
            this.id,
            targetAccountId,
            LocalDateTime.now(),
            money
        );
        this.activityWindow.addActivity(withdrawal);
        return true;
    }

    private boolean mayWithdraw(Money money) {
        return Money.add(
            this.calcuateBalance(),
            money.negate()
        ).isPositive();
    }

    public boolean deposit(Money money, AccountId sourceAccountId) {
        Activity deposit = new Activity(
            this.id,
            sourceAccountId,
            this.id,
            LocalDateTime.now(),
            money
        );
        this.activityWindow.addActivity(deposit);
        return true;
    }
}
```

- Activity는 실제 계좌의 현재 스냅숏을 제공하며, 계좌에 대한 모든 입금과 출금은 Activity 엔티티에 포착된다
- 계좌의 현재 잔고를 계산하기 위해 Account 엔티티는 Activity Window의 첫번째 활동 바로전의 잔고를 표현하는 baselineBalance 속성을 가지고 있으며, 현재 총 잔고는 기준 잔고(baselineBalance)에 모든 Activity Widnow의 잔고들을 합한 값이 된다

## 유스케이스 둘러보기
- 일반적인 유스케이스는 다음과 같은 단계를 따른다
    * 입력을 받는다
    * 비즈니스 규칙을 검증한다
    * 모델 상태를 조작한다
    * 출력을 반환한다

- 유스케이스는 인커밍 어댑터로 입력을 받는데, 유스케이스 코드는 도메인 로직에만 신경써야 하고 입력 유효성 검증으로 오염되면 안되기 때문에 입력 유효성 검증은 다른곳에서 처리한다
- 유스케이스는 비즈니스 규칙을 검증할 책임이 있으며, 도메인 엔티티와 이 책임을 공유한다
- 비즈니스 규칙을 충족하면 유스케이스는 입력을 기반으로 어떤 방법으로든 모델의 상태를 변경하는데, 도메인 객체의 상태를 바꾸고 영속성 어댑터를 통해 구현된 포트로 이 상태를 전달해서 저장될 수 있게 한다. 유스케이스는 또다른 아웃고잉 어댑터를 호출할수도 있다
- 마지막 단계는 아웃고잉 어댑터에서 온 출력값을, 유스케이스를 호출한 어댑터로 반환할 출력 객체로 변환하는것이다

```java
package buckpal.application.service;

@RequiredArgsConstructor
@Transactional
public class SendMoneyService implements SendMoneyUseCase {
    private final LoadAccountPort loadAccountPort;
    private final AccountLock accountLock;
    private final UpdateAccountStatePort updateAccountStatePort;

    @Override
    public boolean sendMoney(SendMoneyCommand command) {
        // TODO 비즈니스 로직 검증
        // 모델 상태 조작
        // 출력값 반환
    }
}
```

- 서비스는 인커밍포트 인터페이스인 SendMoneyUseCase를 구현하고, 계좌를 불러오기 위해 아웃고잉 포트 인터페이스인 LoadAccountPort를 호출한다. 그리고 데이터베이스의 계좌 상태를 업데이트 하기위해 UpdateAccountStatePort를 호출한다

## 입력 유효성 검증
- 애플리케이션 계층에서 입력 유효성 검증을 해야하는 이유는 그렇게 하지 않을 경우 애플리케이션 코어의 바깥쪽으로부터 유효하지 않은 입력값을 받게되고, 모델의 상태를 해칠 수 있기 때문이다
- 입력 모델에서 이러한 입력 유효성을 검증하는게 좋다. 정확히는 SendMoneyCommand 클래스의 생성자 내에서 검사를 하는게 좋다

```java
package buckpal.application.port.in;

@Getter
public class SendMoneyCommand {
    private final AccountId sourceAccountId;
    private final AccountId targetAccountId;
    private final Money money;

    public SendMoneyCommand(
        AccountId sourceAccountId,
        AccountId targetAccountId,
        Money money) {
        this.sourceAccountId = sourceAccountId;
        this.targetAccountId = targetAccountId;
        this.money = money;
        requireNonNull(sourceAccountId);
        requireNonNull(targetAccountId);
        requireNonNull(money);
        requireGreaterThan(money, 0);
    }
}
```

- 송금을 위해서는 출금 계좌와 입금 계좌의 ID, 송금할 금액이 필요하며, 모든 파라미터는 Null이 아니여야하고 송금액은 0보다 커야한다. 이러한 조건중 하나라도 위반된다면 객체를 생성할때 예외를 던져 객체 생성을 막으면 된다
- SendMoneyCommand는 유스케이스 API의 일부이기 때문에 인커밍 포트 패키지에 위치하며, 유효성 검증이 애플리케이션의 코에어에 남아있지만 유스케이스 코드를 오염시키지 않는다
- 자바에는 Bean Validation API가 이미 존재하기 때문에, 이를 이용해서 필요한 유효성 규칙들을 필드에 어노테이션으로 표현할수있다

## 생성자의 힘
- SendMoneyCommand에 필드를 새로 추가해야한다고 생각해보자. Builder를 이용 할 경우 빌더를 호출하는 코드에 새로운 필드를 추가하는것을 잊을 수 있는데, 컴파일러는 이에 대해 경고를 해주진 못하지만 생성자를 직접 사용했다면 다를것이다

## 유스케이스마다 다른 입력모델
- 각기 다른 유스케이스에 동일한 입력 모델을 사용하고 싶을 수도 있다. 계좌 등록하기와 계좌 정보 업데이트 유스케이스에서 둘다 거의 비슷한 계좌 상세 정보가 필요할것이다. 하지만 두 유스케이스에서 같은 입력 모델을 공유 할경우 특정 필드에 null을 허용해야 할것이다
- 불변 커맨드 객체의 필드에 대해서 null을 유효한 상태로 받아들이는것은 그 자체로 좋지 않다
- 따라서 각 유스케이스 전용 입력 모델은 유스케이스를 훨씬 명확하게 만들고 다른 유스케이스와의 결합도 제거해서 불필요한 부수효과가 발생하지 않는다

## 비즈니스 규칙 검증하기
- 비즈니스 규칙을 검증하는것은 도메인 모델의 현재 상태에 접근해야 하는데, 비즈니스 규칙은 유스케이스의 맥락 속에서 의미적인 유효성을 검증하는 일이기 때문이다
- "출금 계좌는 초과 출금 되어서는 안된다" 라는 규칙을 보면, 이 규칙은 출금 계좌와 입금 계좌가 존재하는지 확인하기 위해 모델의 현재 상태에 접근해야 하기 때문에 비즈니스 규칙이다
- "송금되는 금액은 0보다 커야한다"라는 규칙은 모델에 접근하지 않고도 검증될 수 있으므로 유효성 검증으로 구현할 수 있다
- 비즈니스 규칙 검증은 도메인 엔티티 안에 넣는게 좋다. 이렇게 하면 지켜야하는 비즈니스 로칙 바로 옆에 규칙이 위치하기 떄문에 위치를 정하는것도 쉽고 추론하기도 쉽다
- 도메인 엔티티에서 비즈니스 규칙을 검증하기가 여의치 않다면 유스케이스 코드에서 도메인 엔티리를 사용하기 전에 해도 된다

```java
package buckpal.application.service;

@RequiredArgsConstructor
@Transactional
public class SendMoneyService implements SendMoneyUseCase {
    //...

    @Override
    public boolean sendMoney(SendMoneyCommand command) {
        requireAccountExists(command.getSourceAccountId());
        requireAccountExists(command.getSourceTargettId());
    }
}
```

## 풍부한 도메인 모델 vs 빈약한 도메인 모델
- 풍부한 도메인 모델에서는 애플리케이션의 코어에 있는 엔티티에서 가능한 많은 도메인 로직이 구현된다. 엔티티들은 상태를 변경하는 메서드를 제공하고, 비즈니스 규칙에 맞는 유효한 변경만 허용한다
- 이때 유스케이스는 도메인 모델의 진입점으로 동작하며 유스케이스는 사용자의 의도만을 표현하며 이 의도를 실제 작업을 수행하는 체계화된 도메인 엔티티 메서드 호출로 변환한다. 많은 비즈니스 규칙이 유스케이스 구현체 대신 엔티티에 위치하게 된다
- 빈약한 도메인 모델에서는 엔티티 자체가 굉장히 얇다. 일반적으로 엔티티는 상태를 표현하는 필드와 이 값을 읽고 바꾸기 위한 getter/setter 메서드만 포함하고 어떤 도메인 로직도 가지고 있지 않다
- 즉, 도메인 로직이 유스케이스 클래스에 구현돼 있다. 비즈니스 규칙을 검증하고, 엔티티의 상태를 바꾸고, 데이터 베이스 저장을 담당하는 아웃고잉 포트에 엔티티를 전달할 책임 역시 유스케이스 클래스에 있다

## 유스케이스마다 다른 출력 모델
- 유스케이스들간에 같은 출력 모델을 공유하게 되면 유스케이스들도 강하게 결합된다. 한 유스케이스에서 출력 모델에 새로운 필드가 필요해지면 이 값과 관련이 없는 다른 유스케이스에도 이 필드를 처리해야 한다
- 단일 책임의 원칙을 적용하고 모델을 분리해서 유지하는것은 유스케이스의 결합을 제거하는데에 도움이 된다. 같은 이유로 도메인 엔티티를 출력 모델로 사용하고싶은 유혹도 견뎌내야 한다

## 읽기 전용 유스케이스는 어떨까
- 읽기 전용 작업을 유스케이스라고 언급하는것은 조금 이상하다. 하지만 애플리케이션 코어의 관점에서 이 작업은 간단한 데이터 쿼리이며 프로젝트 맥락에서 유스케이스로 간주하지 않는다면 실제 유스케이스와 구분하기 위해 쿼리로 구현할 수 있다
- 한가지 방법으로 쿼리를 위한 인커밍 전용 포트를 만들고 이를 쿼리서비스에 구현하면 된다

```java
package buckpal.application.service;

@RequireArgsConstructor
class GetAccountBalanceService implements GetAccountBalanceQuery {
    private final LoadAccountPort loadAccountPort;

    @Override
    public Money getAccountBalance(AccountId accountId) {
        return loadAccountPort.loadAccount(accountId, LocalDateTime.now()).calculateBalance();
    }
}
```

- 쿼리서비스는 유스케이스 서비스와 동일한 방식으로 동작한다. GetAccountBalanceQuery라는 인커밍 포트를 구현하고, 데이터베이스로부터 실제로 데이터를 로드하기 위해 LoadAccountPort라는 아웃고잉 포트를 호출한다
- 이처럼 읽기 전용 쿼리는 쓰기가 가능한 유스케이스(커맨드)와 코드상에서 명확히 구분되는데, 이는 CQS(Command-Query Separation)나 CQRS(Command-Query Responsibility Segregation)과 아주 잘 맞는다

## 유지보수가 가능한 소프트웨어를 만드는데 어떻게 도움이 될까?
- 유스케이스별 모델을 만듬으로써 유스케이스를 명확히 이해 할 수 있고, 장기적으로 유지보수하기도 쉽다
- 여러명의 개발자가 다른사람이 작업중인 유스케이스를 건드리지 않은채로 여러개의 유스케이스를 동시에 작업할수있다
- 꼼꼼한 입력 유효성 검증, 유스케이스별 입출력 모델은 지속 가능한 코드를 만드는데 큰 도움이 된다