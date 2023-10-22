## 1. int로 할 수 있는데 bigint로 해서 그렇다.

Spring Boot에서 Long으로 id를 선언하면서 bigint로 타입이 지정되었다.

bigint는 8바이트

int는 4바이트이다.

[블로그](https://dogleg.co.kr/?p=163)에 따르면 id값이 43억이 넘어가는 아주 큰 DB를 사용한다면 고민없이 bigint형을 사용해야 한다고 한다.

그렇지 않으면 두 가지 요소를 고려해야 한다는 데,

#### 첫째는 mysql의 메모리 용량, 서버 저장 공간, 쿼리 속도 등을 고려해야 한다.

int형이 bigint형에 비해 10% 이상의 디스크 용량을 절약한다고 한다.

수 십억개의 데이터를 가지지 않을 것이라면 속도 등 효율성 측면에서 bigint형 대신 int형을 사용하는 것을 권장하고 있다.

DB의 양이 작으면 어떤 것이든 상관없고

43억개까지는 아니더라도 중규모의 DB라면 int형이 서버 운영에 훨씬 효율적인 것이다.

#### 둘째는 향후 업그레이드 용이성을 고려해야 한다.

DB 규모가 큰 어플리케이션 개발 시 서버 운영 효율이 낮아지더라도 bigint형으로 설정하는 것이 나을 수 있는 데, 그 이유는 나중에 int형을 bigint형으로 바꾸기가 너무 힘들기 때문이다.

id의 경우 다른 테이블에서 외래키로 사용하고 있을 경우가 더러 존재하고 이미 int형을 bigint형으로의 변경을 고려할 정도라면 가지고 있는 데이터의 양이 꽤 많다는 이야기로 수정이 엄청 오래 걸릴 것이다.

[블로그](https://dogleg.co.kr/?p=163)의 결론을 보면 작은 DB라면 bigint든 int든 뭐를 사용해도 상관없다고 한다.

다만 향후 대량의 데이터가 생길 것이라면 대량의 데이터 수정으로 고생하지 말고 서버 운영 효율을 좀 포기하더라도 bigint로 하라는 것이다.

(이거는 나중에 알게 된 건데 몇 억건의 데이터를 옮기려면 꽤 많은 시간이 걸린다고 한다..ㅇ 나중에 슬퍼하기 전에.. bigint로 하는 것이..)

## 2. 분산환경이면 어쩌려고 auto increment를 한거냐

JPA를 이용해서 문제가 된 id는 다음과 같은 코드로부터 탄생했다.

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

책을 보고 그대로 짰던 코드였기 때문에 딱히 의문이 없었는데 의문이 없는 게 의문이 되어버렸다..

```java
@GeneratedValue(strategy = GenerationType.IDENTITY)
```

위의 애노테이션의 의미는 다음과 같다.

기본키 생성을 데이터베이스에 위임한다. (= auto_increment)

영속성 컨텍스트는 엔티티를 영구 저장하는 환경인데 영속성 컨텍스트로 관리하기 위해서는 pk값이 무조건 필요하다.

원래 entityManager.persist()가 호출되면 바로 DB에 쿼리를 날리는 게 아니라 영속성 컨텍스트에 저장한 후에 트랜잭션의 커밋이 일어나면 그 때 DB에 저장되었다.
(영속성 컨텍스트의 쓰기 지연 SQL 저장소에 쿼리를 저장했다.)

그런데 pk값이 무조건 필요한데 auto_increment의 경우에는 entityManager.persist()가 호출될 때 바로 DB에 insert 쿼리를 날린다.

이후에 JPA 내부에서 insert 쿼리 실행 후 바로 생성된 id값을 리턴받아서 id를 pk로 영속성 컨텍스트에 저장한다.

위와 같은 방식으로 기본키 영속성을 관리하고 있다.

다시 원래 주제로 돌아와서 그래서 왜 분산환경이면 어쩌려고 auto increment했냐에서

여러 데이터베이스가 있는 상황에서 별다른 동기화가 되어있지 않다면 Duplicate Key가 발생할 확률이 높고 데이터 일관성에 문제가 생길 것이다.

또한, auto increment는 키를 예측하기 쉽다. SQL Injection과 같은 공격에 취약해진다.

### 분산환경에서는 UUID(Universally Unique ID)를 추천한다고 한다.

128비트 데이터로 100% unique하다고 보장은 못하는데 충돌 가능성이 굉장히 낮은 키다.

UUID는 자바 5에서부터 지원하고 있고 java.util.UUID 패키지에서 제공한다고 한다.

```java
import java.util.UUID

...

@Id
@GeneratedValue(generator = "uuids")
@GenericGenerator(name= "uuid2", strategy = "uuid")
private UUID id;
```

위와 같이 쓸 경우 BINARY(16) 혹은 VARCHAR 타입을 가지게 된다.

기존 int타입으로 auto increment하는 것보다 메모리도 많이 쓰고 uuid 생성때문에 insert할 때 시간이 더 많이 걸린다.

## 참고 사이트

1. [MySQL id컬럼 데이터타입 INT? BIGINT?](https://dogleg.co.kr/?p=163)
2. [@GeneratedValue(strategy = GenerationType.IDENTITY) : 기본키 영속성 관리](https://songii00.github.io/2020/03/25/2020-03-25-@GeneratedValue(strategy = GenerationType.IDENTITY)기본키 영속성 관리/)
3. [MySQL AUTO_INCREMENT의 문제점](https://wjdtn7823.tistory.com/59)