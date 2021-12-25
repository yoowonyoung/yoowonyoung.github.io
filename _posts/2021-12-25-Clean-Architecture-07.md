---
layout: post
title: "Clean Architecture 7장 - 아키텍처 요소 테스트 하기"
description: Clean Architecture 7장 
date: 2021-12-25 15:08:00 +09:00
categories: Clean Architecture Study
---

# 아키텍처 요소 테스트 하기

## 테스트 피라미드
- 테스트 피라미드에 최 하층, 가장 넓은 범위를 가지고 있는것은 만드는 비용이 적고, 유지보수하기가 쉽고, 빨리 실행되고, 안정적인 작은 크기의 테스트들에 대해서 높은 커버리지를 유지하는 단위 테스트
- 여러개의 단위와 단위를 넘는 경계, 아키텍처 경계, 시스템 경계를 결합하는 테스트는 만드는 비용이 더 비싸고, 실행도 느리며, 꺠지기 더 쉬우므로 테스트가 비싸질수록 커버리지 목표는 낮게 잡아야 함
- 단위 테스트는 피라미드의 토대에 해당하므로, 일반적으로 하나의 클래스를 인스턴스화 하고 해당 클래스의 인터페이스를 통해 기능들을 테스트. 만약 테스트중인 클래스가 다른 클래스에 의존한다면 mock을 사용
- 통합테스트는 연결된 여러 유닛들을 인스턴스화 하고, 시작점이 되는 클래스의 인터페이스로 데이터를 보낸 후 유닛들의 네트워크가 기대한 대로 잘 동작하는지 검증. 객체 네트워크가 완전하지 않거나 어떤 시점에서는 목을 대상으로 수행
- 시스템 테스트는 애플리케이션을 구성하는 모든 객체 네트워크를 가동시켜 특정 유스케이스가 전 계층에서 잘 동작하는지 검증

## 단위 테스트로 도메인 엔티티 테스트 하기

```java
class AccountTest {

    @Test
    void withdrawalSucceeds() {
        AccountId accountId = new AccountId(1L);
        Account account = defaultAccount()
            .withAccountId(accountId)
            .withBaselineBalance(new ActivityWindow(
                defaultActivity()
                    .withTargetAccount(accountId)
                    .withMoney(Money.of(999L)).build(),
                defauntActivity()
                    .withTargetAccount(accountId)
                    .withMoney(Money.of(1L)).build()
            )).build();

        boolean success = account.withdraw(Money.of(555L), new AccountId(99L));

        assertThat(success).isTrue();
        assertThat(account.getActivityWindow().getActivity()).hasSize(3);
        assertThat(account.calculateBalance()).isEqualTo(Money.of(1000L));
    }
}
```

- 위 코드는 특정 상태의 Accont를 인스턴스화 하고 withdraw() 메서드를 호출해서 출금을 성공했는지 검증하고, Account 객체의 상태에 기대되는 부수효과들이 잘 일어났는지 확인하는 단순한 단위 테스트
- 이 테스트는 만들고 이해하는것도 쉽고, 아주 빠르게실행됨. 이런식의 단위 테스트가 도메인 엔티티에 녹아있는 비즈니스 규칙을 검정하기에 가장 적절한 방법. 도메인 엔티티의 행동은 다른 클래스에 거의 의존하지 않기 떄문에 다른 종류의 테스트는 필요하지 않음

## 단위 테스트로 유스케이스 테스트하기

```java
class SendMoneyServiceTest {
    //필드 선언 생략

    @Test
    void transactionSucceeds() {
        Account sourceAccount = givenSourceAccount();
        Account targetAccount = givenTargetAccount();

        givenWithdrawalWillSucceed(sourceAccount);
        givenDepositWillSucceed(targetAccount);

        Money money = Money.of(500L);

        SendMoneyCommand command = new SendMoneyCommand(
            sourceAccount.getId(),
            targetAccount.getId(),
            money
        );

        boolean success = sendMoneyService.sendMoney(command);
        assertThat(success).isTrue();

        AccountId sourceAccountId = sourceAccount.getId();
        AccountId targetAccountId = targetAccount.getId();

        then(accountLock).should().lockAccount(eq(sourceAccountId));
        then(sourceAccount).should().withdraw(eq(money), eq(targetAccountId));
        then(accountLock).should().releaseAccount(eq(sourceAccountId));
        //..
    }
}
```

- 테스트의 가독성을 높이기 위해서 Behavior driven development 에서 일반적으로 사용되는 방식대로 given/when/then으로 섹션을 나눔
- given 섹션에서 출금 및 임금 Account의 인스턴스를 각각 생성하고 적절한 상태로 만들어서 given으로 시작하는 메서드에 인자로 넣음
- when 섹션에서는 유스케이스를 실행하기 위해서 sendMoney() 메서드를 호출
- then 섹션에서는 트랜잭션이 성공적이었는지 확인하고, 출금 및 임금 Account 그리고 계좌에 락을 걸고 해제하는 책임을 가진 AccountLock에 대해 특정 메서드가 호출되었는지 검증
- Mock을 이용해여 given으로 시작하는 메서드의 목 객체를 생성
- 테스트중인 유스케이스 서비스는 상태가 없기 때문에 then 섹션에서 특정 상태를 검증할수없음. 대신 테스트는 서비스가 (모킹된) 의존 대상의 특정 메서드와 상호작용 했는지 검증. 이로 인해 테스트가 코드의 행동 변경 뿐만 아니라 구조 변경에도 취약해짐
- 테스트에서 어떤 상호작용을 검증 하고 싶은지 신중하게 생각해야함. 앞의 예제처럼 모든 동작을 검증하는 대신 중요한 핵심만 골라 집중해서 테스트 하는것이 좋음. 모든 동작을 검증하려고 했다가 클래스가 조금이라도 비뀌면 그때마다 테스트가 바뀌어야 하기 때문에 테스트의 가치가 떨어짐

## 통합 테스트로 웹 어댑터 테스트하기

```java
@WebMvcTest(controllers = SendMoneyController.class)
class SendMoneyControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private SendMoneyUseCase sendMoneyUseCase;

    @Test
    void testSendMoney() throws Exception {
        mockMvc.perform(
            post("/accounts/send/{sourceAccountId}/{targetAccountId}/{amount}",
                41L, 42L, 500)
            .header("Content-Type","application/json")
            .andExpect(status().isOk());
        )

        then(sendMoneyUseCase).shound()
            .sendMoney(eq(new SendMoneyCommand(
                new AccountId(41L),
                new AccountId(42L),
                Money.of(500L)
            )));
    }
}
```

- 스프링 부트 프레임워크에서 SendMoneyController 라는 웹 컨트롤러를 테스트하는 표준적인 통합 테스트 방법
- testSendMoney() 메서드에서 입력 객체를 만들고 Mock Http 요청을 웹 컨트롤러에 보냄
- isOk() 메서드로 HTTP 응답의 상태가 200임을 검증하고 모킹한 유스케이스가 잘 호출됫는지 검증. 웹 어댑터의 책임 대부분은 이 테스트로 커버
- MockMvc 객체를 이용해 모킹 했기 때문에 실제로 HTTP 프로토콜을 통해 테스트 한것은 아니지만, 프레임워크가 HTTP 프로토콜에 맞게 모든것을 적절히 잘 변환한다고 믿어야함
- 입력을 JSON에서 SendMoneyCommand 객체로 매핑하는 전과정은 다루고 있음. SendMoneyCommand가 자체 검증 커맨드로 되어 있다면 이 매핑이 유스케이스에 구문적으로 유효한 입력을 생성했는지도 확인할것
- 유스케이스가 실제로 호출 됬는지도 검증했고, HTTP 응답이 기대한 상태를 반환했는지도 검증
- 이 테스트는 하나의 웹 컨트롤러 클래스만 테스트한것처럼 보이지만, @WebMvcTest 어노테이션으로 인해 스프링이 특정 요청 경로, 자바와 JSON간의 매핑, HTTP 입력 검증등에 필요한 전체 객체 네트워크를 인스턴스화 하도록 만들었고, 웹 컨트롤러가 이 네트워크의 일부로써 잘 동작했는지 검증하기 때문에 통합테스트
- 웹 컨트롤러가 스프링 프레임워크에 강하게 묶여있기 때문에 격리된 상태로 테스트 하기 보다는 이 프레임워크와 통합된 상태로 테스트하는게 합리적

## 통합 테스트로 영속성 어댑터 테스트하기

```java
@DataJpaTest
@Import({AccountPersistenceAdapter.class, AccountMapper.class})
class AccountPersistenceAdapterTet {

    @Autowired
    private AccountPersistenceAdapter adapterUnderTest;

    @Autowired
    private ActivityRepository activityRepository;

    @Test
    @Sql("AccountPersistenceAdapterTest.sql")
    void loadsAccount() {
        Account account = adapter.loadAccount(
            new AccountId(1L),
            LocalDateTime.of(2018,8,10,0,0)
        );

        assertThat(account.getActivityWindow().getActivities()).hasSize(2);
        assertThat(account.calculateBalance()).isEqualTo(Money.of(500));
    }
}
```

- @DataJpaTest 어노테이션으로 스프링 데이터 리포지토리들을 포함해서 데이터 베이스 접근에 필요한 객체 네트워크를 인스턴스화 해야 한다고 스프링에 알려줌
- @Import 어노테이션으로 특정 객체가 이 네트워크에 추가되었다는것을 명확하게 표현
- loadAccount() 메서드에 대한 테스트에서는 SQL 스크립트를 이용, 데이터베이스를 특정 상태로 만들고, 어댑터 API를 통해 계좌를 가져온후 SQL 스크립트에서 설정한 상태값을 가지고 있는지 검증
- 이 테스트에서 데이터베이스는 모킹하지 않음. 테스트가 실제로 데이터베이스에 접근. 스프링에서는 인 메모리 데이터베이스를 테스트에서 사용하기 때문에 실용적이지만, 프로덕션 환경에서는 인 메모리 데이버테이스가 아니기 때문에 테스트가 완벽하게 통과했다 하더라도 데이터베이스 고유의 SQL 문법등으로 인해 문제가 발생할수 있음
- 영속성 어댑터는 Testcontainers 같은 라이브로리를 이용해 실제 데이터베이스를 이용해서 테스트 하는게 좋음

## 시스템 테스트로 주요 경로 테스트 하기

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_ROOT)
class SendMoneySystemTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    @Sql("SendMoneySystemTest.sql")
    void sendMoney() {
        Money initialSourceBalance = sourceAccount().calculateBalance();
        Money initialTargetBalance = targetAccount().calculateBalance();

        ResponseEntity response = whenSendMoney(
            sourceAccountId(),
            targetAccountId(),
            transferredAmmount()
        );

        then(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        then(sourceAccount().calculateBalance()).isEqualTo(intitalSourceBaalance.minus(transferredAmmount()));
        then(targetAccount().calculateBalance()).isEqualTo(initialTargetBalance.minus(transferredAmmount()));
    }

    private ResponseEntity whenSendMoney(AccountId sourceAccountId, AccountId targetAccountId, Money amount) {
        HttpHeaders headers = new HttpHeaders();
        headers.add("Content-Type", "application/json");
        HttpEntity<Void> request = new HttpEntity<>(null, headers);

        return restTemplate.exchange(
            "/accounts/sendMoney/{sourceAccountId}/{targetAccountId}/{amount}",
            HttpMethod.POST,
            request,
            Object.class,
            sourceAccountId.getValue(),
            targetAccountId.getValue(),
            amount.getAmount()
        );
    }
}
```

- @SpringBootTest 어노테이션으로 스프링이 어플리케이션을 구성하는 모든 객체 네트워크를 띄우게 함. 랜덤 포트로 애플리케이션을 띄우도록 설정
- 웹 어댑터에서처럼 MockMvc를 이용한것이 아닌 TestRestTemplate를 사용해서 프로덕션 환경에 가깝게 테스트
- 실제 HTTP 통신을 하는것처럼 실제 출력 어댑터도 이용. 헥사고날 아키텍처에서는 몇개의 출력 포트 인터페이스만 모킹하면 되서 편리
- 시스템 테스트는 단위 테스트와 통합 테스트가 발견하는 버그와는 또다른 종류의 버그를 발견할수 있음(계층간 매핑 버그 등)
- 시스템 테스트는 여러개의 유스케이스를 결합해서 시나리오를 만들떄 더 유용

## 얼만큼의 테스트가 충분할까
- 라인커버리지는 테스트 성공을 측정하는데 있어서 잘못된 지표
- 얼마나 마음 편하게 소프트웨어를 배포할수있느냐를 테스트 성공의 기준으로 삼으면 됨. 테스트를 실행한후에 소프트웨어를 배포해도 될만큼 테스트를 신뢰한다면 그것으로 된것
- 처음 몇번의 배포에는 믿음의 도약이 필요한데, 프로덕션의 버그를 수정하고 이로부터 배우는것을 우선순위로 삼고 있다면 제대로 가고 있는것
- 헥사고날 아키텍처에서는 다음과 같은 테스트 전략을 사용
    * 도메인 엔티티는 단위 테스트로 커버
    * 유스케이스는 단위 테스트로 커버
    * 어댑터는 통합 테스트로 커버
    * 사용자가 취할수 있는 중요 애플리케이션 경로는 시스템 테스트로 커버

## 유지보수 가능한 소프트웨어를 만드는데 어떻게 도움이 될까
- 핵심 도메인 로젝은 단위 테스트로, 어댑터는 통합 테스트로 처리하는 명확한 테스트 전략 정의 가능
- 입출력 포트라는 뚜렷한 모킹 지점을 가짐
- 모킹하는게 너무 버거워지거나 코드의 특정 부분을 커버하기위해 어떤 테스트를 써야 할지 모르겟다면 이는 경고 신호
