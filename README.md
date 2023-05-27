[Transaction 블로그 포스팅 주소](https://velog.io/@wish17/%EC%BD%94%EB%93%9C%EC%8A%A4%ED%85%8C%EC%9D%B4%EC%B8%A0-%EB%B0%B1%EC%97%94%EB%93%9C-%EB%B6%80%ED%8A%B8%EC%BA%A0%ED%94%84-52%EC%9D%BC%EC%B0%A8-Spring-MVC-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98Transaction)

# [Spring MVC] 트랜잭션(Transaction)

> ``트랜잭션(Transaction)``
- 전부 성공하든가 전부 실패하든가(All or Nothing)하게 만드는 것
-  여러개의 작업들을 하나의 그룹으로 묶어서 처리하는 처리 단위를 의미

>``ACID 원칙``
- 원자성(Atomicity)
    - 작업을 더이상 쪼갤 수 없음을 의미
- 일관성(Consistency)
    -   일관성있게 저장되거나 변경되는 것을 의미
- 격리성(Isolation)
    - 여러 개의 트랜잭션이 실행될 경우 각각 독립적으로 실행이 되어야 함을 의미
- 지속성(Durability)
    -   트랜잭션이 완료되면 그 결과는 지속되어야 한다는 의미

``커밋(commit)``
- 모든 작업을 최종적으로 DB에 반영하는 명령어
(commit 명령을 수행하면 DB에 영구적으로 저장)
- 만약 commit 명령을 수행하지 않으면 작업의 결과가 데이터베이스에 최종적으로 반영되지 않는다.
- commit 명령을 수행하면, 하나의 트랜젝션 과정은 종료하게 된다.



``롤백(rollback)``
- 작업 중 문제가 발생했을 때, 트랜잭션 내에서 수행된 작업들을 취소하는 것
(트랜잭션 시작 이 전의 상태로 되돌아감)

***

## 선언형 방식의 트랜잭션 적용

Spring에서 선언형 방식으로 트랜잭션을 적용하는 2가지 방법

1. 비즈니스 로직에 애너테이션을 추가하는 방식
2.  AOP 방식을 이용해서 비즈니스 로직에서 아예 트랜잭션 적용 코드 자체를 감추는 방식

### 애너테이션 방식의 트랜잭션 적용

``@Transactional`` 애너테이션을 클래스 레벨에 추가하면 기본적으로 해당 클래스에서 이용하는 모든 메서드에 트랜잭션이 적용된다.

또한 ``RuntimeException``이 발생하면 rollback도 해준다.
but Exception, SQLException, DataFormatException 같은 체크 예외(checked exeption)는 ``@Transactional`` 애너테이션만 추가해서는 rollback이 되지 않는다.

```java
@Transactional(rollbackFor = {SQLException.class, DataFormatException.class})
```
위와 같이 해당 체크 예외를 직접 지정해주거나 언체크 예외(unchecked exception)로 감싸서 rollback이 동작하도록 할 수 있다.

>``@Transactional(readOnly = true)``
- 말 그대로 읽기전용으로 만들어 commit 절차를 진행하기는 하지만 JPA 내부적으로 영속성 컨텍스트를 flush하지 않도록 하는 방법
- 읽기용으로만 쓰도록 성능 최적화 하는 용도로 사용
(불필요한 동작 안하게 만들어 줌)

>플러시(flush)란? 
- 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영하는 것을 의미

#### 클래스 레벨과 메서드 레벨의 트랜잭션 적용 순서

- 클래스 레벨에만 ``@Transactional``이 적용된 경우
    - 클래스 레벨의 ``@Transactional`` 애너테이션이 메서드에 일괄 적용

- 클래스 레벨과 메서드 레벨에 함께 적용된 경우
    - 메서드 레벨의 @Transactional 애너테이션이 적용
    - 만약 메서드 레벨에 @Transactional 애너테이션이 적용되지 않았을 경우, 클래스 레벨의 @Transactional 애너테이션이 적용

즉, 메서드마다 ``@Transactional(특성)`` 특성(Attribute)을 다르게 할 수 있는 것이다.


#### 트랜잭션 전파(Transaction Propagation)

- 트랜잭션의 경계에서 진행 중인 트랜잭션이 존재할 때 또는 존재하지 않을 때, 어떻게 동작할 것인지 결정하는 방식을 의미

- 트랜잭션 전파는 propagation 애트리뷰트를 통해서 설정할 수 있다.

- ``Propagation.REQUIRED`` 
    - 진행 중인 트랜잭션이 없으면 새로 시작하고, 진행 중인 트랜잭션이 있으면 해당 트랜잭션에 참여
    - 가장 많이 사용되는 propagation 유형의 디폴트 값
- ``Propagation.REQUIRES_NEW``
    - 이미 진행중인 트랜잭션과 무관하게 새로운 트랜잭션을 시작(생성)
    - 기존에 진행중이던 트랜잭션은 새로 시작된 트랜잭션이 종료할 때까지 중지된다.
- ``Propagation.MANDATORY``
    - 진행 중인 트랜잭션이 없으면 예외를 발생
- ``Propagation.NOT_SUPPORTED``
    - 트랜잭션을 필요로 하지 않음을 의미
    - 진행 중인 트랜잭션이 있으면 메서드 실행이 종료될 때 까지 진행중인 트랜잭션은 중지
- ``Propagation.NEVER``
    - 트랜잭션을 필요로 하지 않음을 의미
    - 진행 중인 트랜잭션이 존재할 경우에는 예외를 발생

#### 트랜잭션 격리 레벨(Isolation Level)

``@Transactional`` 애너테이션의 ``isolation`` 애트리뷰트
=> 독립적으로 실행되어야 하는 격리성을 조정할 수 있는 옵션

- ``Isolation.DEFAULT``
    - DB에서 제공하는 기본 값

- ``Isolation.READ_UNCOMMITTED``
    - 커밋하지 않은 데이터를 다른 트랜잭션에서 읽는 것을 허용

- ``Isolation.READ_COMMITTED``
    - 다른 트랜잭션에 의해 커밋된 데이터를 읽는 것을 허용

- ``Isolation.REPEATABLE_READ``
    - 트랜잭션 내에서 한 번 조회한 데이터를 (변경되더라도) 반복해서 조회하면 같은 데이터가 조회되도록 한다.

- ``Isolation.SERIALIZABLE``
    - 동일한 데이터에 대해서 동시에 두 개 이상의 트랜잭션이 수행되지 못하도록 한다.

트랜잭션의 격리 레벨은 일반적으로 데이터베이스나 데이터소스에 설정된 격리 레벨을 따르는 것을 권장한다.
