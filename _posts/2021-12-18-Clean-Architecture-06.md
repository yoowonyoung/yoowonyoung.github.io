---
layout: post
title: "Clean Architecture 6장 - 영속성 어댑터 구현하기"
description: Clean Architecture 6장 
date: 2021-12-18 13:57:00 +09:00
categories: Clean Architecture Study
---

# 영속성 어댑터 구현하기
- 의존성을 역전시키기 위해 영속성 계층을 애플리케이션 계층의 플러그인으로 만들어야함

## 의존성 역전
- 영속성 계층 대신 애플리케이션 서비스에 영속성 기능을 제공하는 영속성 어댑터
- 애플리케이션 서비스에서는 영속성 기능을 사용하기 위해 포트 인터페이스를 호출, 포트는 실제로 영속성 작업을 수행하고 데이터베이스와 통신할 책임을 가진 영속성 어댑터 클래으에 의해 구현
- 영속성 어댑터는 주도되는 혹은 아웃고잉 어댑터로, 애플리케이션에 의해 호출될뿐 애플리케이션을 호출하지는 않음
- 포트는 사실상 애플리케이션 서비스와 영속성 코드 사이의 간접적인 계층. 영속성 코드를 리팩터링 하더라도 코어 코드를 변경하는 결과로 이어지지는 않음
- 런타임에도 의존성은 애플리케이션 코에에서 영속성 어댑터로 향함

## 영속성 어댑터의 책임
- 영속성 어댑터가 하는 일반적인 일들은 다음과 같음
    * 입력을 받음
    * 입력을 데이터베이스 포맷으로 매핑
    * 입력을 데이터베이스로 보냄
    * 데이터베이스 출력을 애플리케이션 포맷으로 매핑
    * 출력을 반환

- 영속성 어댑터는 포트 인터페이스를 통해 입력을 받는데, 입력 모델은 인터페이스가 지정한 도메인 엔티티이거나 특정 데이터베이스 연산 전용 객체
- 영속성 어댑터는 데이터베이스를 쿼리하거나 변경하는데 사용할 수 있는 포맷으로 입력 모델을 매핑하는데, JPA 엔티티 객체로 매핑
- 핵심은 영속성 어댑터의 입력 모델이 영속성 어댑터 내부에 있는것이 아니라 애플리케이션 코어에 있어서, 영속성 어댑터 내부를 변경하는게 코어에 영향을 미치지 않는것
- 데이터베이스 응답을 포트에 정의된 출력 모델로 매핑해서 반환하는데, 출력 모델 역시 영속성 어댑터가 아닌 애플리케이션 코어에 위치 해야함

## 포트 인터페이스 나누기
- 보통은 특정 엔티티가 필요로하는 모든 데이터베이스 연산을 하나의 레파지토리 인터페이스에 넣어두는게 일반적. 하지만 데이터베이스 연산에 의존하는 각 서비스는 인터페이스에서 단 하나의 메서드만 사용하더라도 하나의 넓은 포트 인터페이스에 대해 의존성을 갖게 됨
- 인터페이스 분리의 원칙에 의해, 클라이언트는 오로지 자신이 필요로하는 메서드만 알면 되도록 넓은 인터페이스를 특화된 인터페이스로 분리해야함
- 각 서비스는 실제로 필요한 메서드에만 의존하고, 포트의 이름이 포트의 역할을 명확하게 잘 표현하도록 함(ex> LoadAccountPort, UpdateAccountPort 등)

## 영속성 어댑터 나누기
- 영속성 연산이 필요한 도메인 클래스(혹은 DDD에서 애그리게이트) 하나당 하나의 영속성 어댑터를 구현하는 방식도 선택할 수 있음. 이렇게 하면 영속성 어댑터들은 각 영속성 기능을 이용하는 도메인 경계를 따라 자동으로 나뉘어짐
- 영속성 어댑터를 훨씬 더 많은 클래스로 나눌수도 있는데, JPA나 OR매퍼를 이용한 영속성 포트를 구현하면서도 성능을 개선하기위해 평범한 SQL을 사용하는 포트도 함께 구현하는등의 경우
- 도메인 코드는 영속성 포트에 의해 정의된 명세를 어떤 클래스가 충족시키는지에 대해 관심이 없으므로, 모든 포트가 구현되있기만 한다면 영속성 계층에서 하고싶은 어떤 작업이든 해도 됨
- 애그리게이트 하나당 하나의 영속성 어댑터 접근 방식 또한 나중에 여러개의 바운디드 컨텍스트의 영속성 요구사항을 분리하기 위한 좋은 토대가 됨

## 스프링 데이터 JPA 예제

```java
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class Account {
    @Getter private final AccountId id;
    @Getter private final ActivityWindow activityWindow;
    private final Money baselineBalance;

    // ...
}
```

- Account 클래스는 getter와 setter만 가진 간단한 데이터 클래스가 아니며, 최대한 불변성을 유지하려고 하고있음
- 데이터베이스와의 통신에 스프링 데이터 JPA를 사용할것이므로 Entity 어노테이션이 추가된 클래스를 만들어야함

```java
@Entity
@Table(name = "account")
@Data
@AllArgsConstructor
@NoArgsController
class AccountJpaEntity {

    @Id
    @GeneratedValue
    private Long id;
}
```

- activity 테이블도 있어야함

```java
@Entity
@Table(name = "activity")
@Data
@AllArgsConstructor
@NoArgsConstructor
class ActivityJapEntity {

    @Id
    @GeneratedValue
    private Long id;

    @Coluumn
    private LocalDateTime timestamp;

    @Column
    private Long ownerAccountId;

    @Column
    private Long sourceAccountId;
    
    @Column
    private Long targetAccountId;

    @Column
    private Long amount;
}
```

- JPA의 ```@ManyToOne```이나 ```@OneToMany```에너테이션을 이용해 ActivityJpaEntity 와 AccountJpaEntity를 연결해서 관계를 표현할수도 있지만, 데이터베이스 쿼리에 부수효과가 생길수있기 때문에 이부분은 일단 제외

```java
interface AccountRepository extends JpaRepository<AccountJpaEntity, Long> {

}
```

```java
interface ActivityRepository extends JpaRepository<ActivityJpaEntity, Long> {
    // ...
}
```

- 스프링 부트는 이 레포지토리들을 자동으로 찾고, 스프링 데이터는 실제로 데이터베이스와 통신하는 인터페이스 구현체를 제공해준다
- 이제 JPA엔티티와 레포지토리를 만들었으니 영속성 기능을 제공하는 영속성 어댑터를 만들어야함

```java
@RequiredArgsConsturctor
@Component
class AccountPersistenceAdapter implements LoadAccountPort, UpdateAccountStatePort {
    private final AccountRepository accountRepository;
    private final ActivityRepository activityRepository;
    private final AccountMapper accountMapper;

    @Override
    private Account loadAccount(AccountId accountId, LocalDateTime baselineDate) {
        AccountJpaEntity account = accountRepository.findById(accountId.getValue()).orElseThorw(EntityNotFoundException::new);
        List<ActivityJapEntity> activities = activityRepository.findByOwnerSince(accountId.getValue(), baseLineDate);
        ...
    }

    @Override
    public void updateActivities(Account account) {
        //...
    }
}
```

- 영속성 어댑터는 애플리케이션에 필요한 LoadAccountPort와 UpdateAccountStatePort 라는 2개의 포트를 구현
- 위의 코드에서는 Account와 Activity 도메인 모델, AccountJpaEntity와 ActivityJpaEntity 데이터베이스 모델간에 양방향 매핑이 존재하는데, JPA 어노테이션을 Account와 Activity로 옮길수도 있겟지만 이런 매핑하지 않기도 유효한 전략일수가 있다
- JPA로 인해 도메인 모델을 타협할수밖에 없기도 한데JPA는 기본 생성자를 필요로 하고, 또 영속성 계층 측면에서는 성능 측면에서 ```@ManyToOne```관계를 설정하는것이 적절할수있겟지만, 데이터의 일부만 가져오기를 원하는경우 가 발생할수있다. 그래서 영속성 측면과 타협 없이 풍부한 도메인 모델을 생성하고 싶다면 도메인 모델과 영속성 모델을 매핑하는게 좋다

## 데이터베이스 트랜잭션은 어떻게 해야 할까
- 트랜잭션은 하나의 특정한 유스케이스에 대해서 일어나는 모든 쓰기 작어베 걸쳐 있어야 한다. 그래야 그중 하나라도 실패할경우 다같이 롤백 되기 때문
- 영속성 어댑터는 어떤 데이터베이스 연산이 같은 유스케이스에 포함되는지 알지 못하기 때문에, 트랜잭션을 열고 닫을지 결정할 수 없으므로, 이 책임은 영속성 어댑터 호출을 관장하는 서비스에 위임해야 함
- 가장 쉬운 방법은 ```@Transactional``` 어노테이션을 어플리케이션 서비스 클래스에 붙여서 모든 public 메서드를 트랜잭션으로 감싸는것
- 만약 서비스가 트랜잭션 어노테이션으로 오염되지 않고 깔끔하게 유지되길 원한다면 AspectJ같은 도구로 관점 지향 프로그래밍으로 트랜잭션 경계를 코드에 위빙할수있음

## 유지보수 가능한 소프트웨어를 만드는데 어떻게 도움이 될까
- 영속성 어댑터를 만들면 도메인 코드가 영속성과 관련된것들로부터 분리되어 풍부한 도메인 모델을 만들수 있음
- 좁은 포트 인터페이스를 사용하면 포트마다 다른 방식으로 구현할수있는 유연함이 생기고, 포트뒤에서 어플리케이션이 모르게 다른 영속성 기술을 사용할수도 있음

