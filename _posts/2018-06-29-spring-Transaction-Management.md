---
date: 2018-06-28T19:12:33+00:00
category: spring-boot

---

# Reference
> https://stackoverflow.com/questions/8490852/spring-transactional-isolation-propagation
> 
>https://docs.spring.io/spring/docs/current/spring-framework-reference/data-access.html#transaction-event
>
>https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/transaction/TransactionDefinition.html
>
> [Spring 5 Reciepes - Chapter 10. Spring Transaction Management](https://www.apress.com/in/book/9781484227893)
> 

## 트랜잭션 모델

### Global Transaction 

RDB, messege queue 등 다른 datasource의 트랜잭션을 동시에 관리하는 방법.

기본적으로 JTA로 관리하지만 JNDI가 필요하므로 재사용성에 제약이 생기며 서버 환경에서만 작동한다는 문제점이 있다.


Spring에서는 `JtaTransactionManager`로 JTA를 사용할 수 있다.

특정 datasource를 대상으로 작동하는것이 아니기 때문에 제공해주지 않아도 된다.

### Local Transaction

datasource 한개를 대상으로 트랜잭션 관리하는 방법. Spring data가 기본적으로 채택하는 트랜잭션 관리 방법이다.

```java
public interface PlatformTransactionManager {

    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;

    void commit(TransactionStatus status) throws TransactionException;

    void rollback(TransactionStatus status) throws TransactionException;
}
```

## @Transactional

`@Transactional` 어노테이션을 사용하려면 꼭  접근자가 public인 메소드에 선언해야한다. 
protected나, private, default면 에러는 안나지만 적용이 안된다. 

꼭 필요하면 AspectJ의 `@EnableTransactionManagement(mode = AdviceMode.ASPECTJ)`와
`@EnableLoadTimeWeaving` 참고

### read-only

읽기 전용으로 트랜잭션을 획득해 해당 트랜잭션을 열고있는 동안은 수정이 불가능하다.
 
### timeout

해당 트랜잭션의 실행 시간이 timeout을 초과할 시 

기본적으로 타임아웃이 존재하지 않아서 따로 설정해주지 않을시 데드락이 발생할 수도 있다.

 uncheckedException(보통 `RuntimeException`)이 발생할때만 자동으로 롤백하며
 
 그 외 checkedException(`SqlException`)등은 롤백하지 않는다.
 
### Propagation

각 트랜잭션 사이의 관계(공존 여부, 우선순위 등)를 정의

| CONSTANT                  	| Meaning                                                               	| Remarks                   	|
|---------------------------	|-----------------------------------------------------------------------	|---------------------------	|
| PROPAGATION_REQUIRED      	| 현재 트랜잭션을 사용하되, 없으면 생성한다                             	| default                   	|
| PROPAGATION_MANDATORY     	| 현재 있는 트랜잭션을 무조건 사용한다. 없으면 Exception을 던진다       	|                           	|
| PROPAGATION_REQUIRES_NEW  	| 기존 트랜잭션 존재 여부와 상관없이 무조건 새로 생성한다.              	| 기존 트랜잭션은 suspend됨 	|
| PROPAGATION_NOT_SUPPORTED 	| 트랜잭션을 사용하지 않는다.                                           	|                           	|
| PROPAGATION_NEVER         	| 트랜잭션을 사용하지 않는다. 기존 트랜잭션이 있으면 Exception을 던진다 	|                           	|
| PROPAGATION_NESTED 	    | 물리적 트랜잭션 1개 안에 여러개의 논리적(서브) 트랜잭션을 만든다 | 물리적 트랜잭션을 기준으로 한개로 처리, 없으면 `PROPAGATION_REQUIRED`와 같이  작동																												|
|

### Isolation
여러개의 트랜잭션이 동시에 같은 dataset을 다를때 어떻게 할것인지 설정해주는 옵션.

데이터를 읽을 때 아래와 같은 동시성 문제를 방지하려면 적절하게 설정해줘야 한다.

| Isolation Level Mode      |  Read             |   Insert    |   Update    |       Lock Scope       |
|---------------------------|-------------------|-------------|-------------|------------------------|
| READ_UNCOMMITTED          |  uncommitted data | Allowed     | Allowed     | No Lock                |
| READ_COMMITTED (Default)  |   committed data  | Allowed     | Allowed     | Lock on Committed data |
| REPEATABLE_READ           |   committed data  | Allowed     | Not Allowed | Lock on block of table |
| SERIALIZABLE              |   committed data  | Not Allowed | Not Allowed | Lock on full table     |



`a=2`라는 가정하에 설명

#### Dirty read

`T1 tx` 가 a=1로 업데이트 하고 커밋하기 전 `T2 tx`가 a 읽어왔는데, 

도중에 `T1 tx`에서 rollback이 일어나

 `T2 tx`는 그전에 읽은 1을 가져와 데이터가 불일치 하는 경우

#### non-repeatable read

 `T1 tx`가 a를 읽었는데 `T2 tx`가 a=1로 업데이트한 경우,

 `T1 tx a=2` != `T2 tx a=1` 즉, 반복해서 값을 읽었을때 다른 값이 나온다

#### Phantom Reads

`T1 Tx`가 50번째 row를 읽었는데,

`T2 Tx`가 `b=2`를 INSERT 하는 바람에 더이상 T1에서 읽은 `a=2` 데이터가 50번째 로우의 데이터가 아닌 상황 


| CONSTANT                   | MEANING                                           | REMARKS                                                          |
|----------------------------|---------------------------------------------------|------------------------------------------------------------------|
| ISOLATION_DEFAULT          | datastore의 기본 설정을 따른다                         | default                                                          |
| ISOLATION_READ_UNCOMMITTED | 아직 커밋되지 않은 데이터를 읽을 수 있다.         | DIRTY-READ, NON-REPEATABLE-READ, PHANTOM-READ는 해결 안됨        |
| ISOLATION_READ_COMMITTED   | 커밋된 데이터만 읽을 수 있다.                     | NON-REPEATABLE-READ, PHANTOM-READ는 해결 안됨                    |
| ISOLATION_REPEATABLE_READ  | 값을 읽을 때 해당 값의 데이트를 허용하지 않는다.(block lock) | PHANTOM-READ는 해결 안됨(로우를 삽입하고 있을때 생기는 문제라서) |
| ISOLATION_SERIALIZABLE     | 한 트랜잭션이 table lock을 걸어버린다.             | 위의 문제는 해결되지만 성능에 문제가 생긴다.                     |


### TransactionManager 여러개 사용하기

한 인스턴스에서 여러개의 트랜잭션 매니저를 사용해야 하는 경우 (데이터 소스 여러개를 관리하는 경우)

각 datasource의 DataSourceTransactionManager bean name을 명시해주면 된다.

> 두 트랜잭션을 동시에 관리하려면 `JtaTransactionManager`를 사용해야 한다

```java
public class TransactionalService {

    @Transactional("order")
    public void setSomething(String name) { ... }

    @Transactional("account")
    public void doSomething() { ... }
}
```
