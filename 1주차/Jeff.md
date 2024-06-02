# Jeff
JPA 스터디

---
# 03. 영속성 관리

## JPA가 제공하는 기능

1. 엔티티와 테이블을 매핑하는 설계 부분
2. 매핑한 엔티티를 실제 사용하는 부분

EntityManager 를 통해 어떻게 사용하는지 알아 볼 것

(Hibernate 기준)

### EntityManager

> 엔티티 저장, 수정, 삭제, 조회 등 모든 일 처리
>

→ 엔티티를 저장하는 가상의 데이터베이스로 생각

## 3.1. 엔티티 매니저 팩토리와 엔티티 매니저

DB 하나를 사용하는 어플리케이션 → 일반적으로 EntityManagerFactory 하나만 생성함

```java
//비용 많이 듦
EntityManagerFactory emf = 
Persistence.createEntityManagerFactory("jpabook")

// META-INF/persistence.xml에 있는 정보를 바탕으로 EntutyMangerFacotry 생성

//비용 거의 안듦
EntityManager em = enf.createEntityManger();
```

- EntityManagerFactory
    - 엔티티 매니저 만드는 공장
    - 만드는 비용이 상당히 큼
        - 어플리케이션 전체에서 공유하도록 설계
    - Thread-safety
    - 생성 시점에 보통 DB connection pool 도 생성
        - J2SE 환경에서 사용하는 방법
- EntityManager
    - 비용 거의 들지 않음
    - Thread-unsafety 함
    - DB 연결이 필요한 시점에서 Connection 획득

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8da4657-58ed-4483-8d1b-3efce9010f0f/107085b8-54f6-4340-a0c5-304c820f847b/Untitled.jpeg)

## ***3.2. 영속성 컨텍스트***

> 엔티티를 영구 저장하는 환경
>

EntityManager 를 통해 엔티티를 저장하거나 조회 하는 경우

→ EntityManager는 영속성 컨텍스트에 엔티티 보관하고 관리

```java
em.persist(member);
```

→ 엔티티 매니저를 사용해 회원 엔티티를 **영속성 컨텍스트에 저장**

- EntityManager를 생성 할 때 생성
  (여러 EntityManager가 같은 영속성 컨텍스트에 접근도 가능 → 11장)

## 3.3. 엔티티의 생명주기

Entity 에는 4가지 상태 존재

1. 비영속(new/trasient) : 영속성 컨텍스트와 전혀 관계 없는 상태
2. 영속(managed) : 영속성 컨텍스트에 저장된 상태
3. 준영속(detached) : 영속성 컨텍스트에 저장되었다가 분리된 상태
4. 삭제(removed) : 삭제된 상태

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8da4657-58ed-4483-8d1b-3efce9010f0f/8b3fc39c-3534-4b2e-8683-0b1a557b01fe/Untitled.jpeg)

### 비영속

> 엔티티 객체 생성. (순수한 객체 상태)
>

→ 객체 자체인 상태

### 영속

> 엔티티매니저를 통해 엔티티를 영속성 컨텍스트에 저장
>

→ 영속성 컨텍스트가 관리하는 엔티티

(JPQL을 사용해서 조회한 엔티티 또한 바로 영속 됨)

### 준영속

> 영속성 컨텍스트가 관리하지 않는 영속 상태의 엔티티
>

### 삭제

> 엔티티를 영속성 컨텍스트와 데이터베이스에서 삭제
>

## 3.4. 영속성 컨텍스트의 특징

### 영속성 컨텍스트와 식별자 값

- 영속성 컨텍스트는 식별자 값(@Id로 기본키와 매핑한 값)으로 엔티티를 구분

⇒ 반드시 식별자가 있어야 함

### 영속성 컨텍스트와 데이터베이스 저장

- 일반적으로 Trasnaction Commit 시점에 DB에 반영

⇒ Flush라고 함

### 영속성 컨텍스트가 엔티티 관리하는 경우 장점

- 1차 캐시
- 동일성 보장
- 트랜잭션을 지원하는 쓰기지연
- 변경 감지
- 지연 로딩

### 3.4.1. 엔티티조회

1차 캐시

- 영속성 컨텍스트 내부의 캐시
- 영속 엔티티는 모두 1차 캐시에 저장

( 영속성 컨텍스트 내부에 Map이 있고 키는 @Id로 매핑한 식별자, 값은 엔티티의 인스턴스)

→ find 로 찾는 경우

1. 1차캐시부터 확인 후 반환
2. 없는 경우 DB에서 조회
3. DB에서 조회한 객체 1차 캐시에 저장, 영속화
4. 영속 상태의 엔티티 반환

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8da4657-58ed-4483-8d1b-3efce9010f0f/4163db9a-5a2f-4793-ada0-d5c7c526724d/Untitled.jpeg)

영속 엔티티의 동일성 보장

```java
Member a = em.find(Member.class, "member1");
Member b = em.find(Member.class, "member1");

// a==b -> true
```

영속성 컨텍스트는 1차 캐시에 있는 엔티티 인스턴스 반환하기에 항상 같은 객체 반환함.

- 동일성(identity)
    - 물리적 동일성(실제 인스턴스 동일)
- 동등성(equality)
    - 논리적 동일성(인스턴스의 식별 값 동일 → equals 재정의 해야 동일함 보장 가능)

→ 1차 캐시를 통해 REPEATABLE READ 등급의 트랜잭션 격리 수준을 DB가 아닌 어플리케이션 레벨에서 제공하는 장점

### 3.4.2 엔티티 등록

Transaction Commit 하기 전까지 엔티티 저장하지 않고

→ 내부 쿼리 저장소에 INSERT SQL 모아둠.

→ Commit 시점에 한번에 DB에 INSERT

⇒ 트랜잭션을 지원하는 쓰지지연

이것이 1차캐시가 있음으로써 지원이 가능함.

트랜잭션을 보장해야 하는 것은 결국 commit 직전에만 쿼리를 실행하면 되기 때문

→ with(nolock) 의 경우 READ UNCOMMITED 인데 이것에 대한 duty read도 줄어들 수 있지 않을까?

### 3.4.3. 엔티티 수정

SQL에 의존적으로 엔티티의 수정에 대해서 설계해야함

(수정이 일어나는 항목 케이스마다 따로 SQL문을 작성하거나

한번에 다 하면 → 원하지 않는 수정이 일어날 수 있음에 주의하거나)

### 변경감지

→ 영속성 컨텍시트에 최초 상태를 스냅샷 형태로 저장함

트랜잭션 커밋

1.  엔티티 매ㅔ니저 내부에서 플러시 호출
2. 엔티티와 스냅샷을 비교해 변경된 엔티티 식별
3. 변경된 엔티티의 수정 쿼리 생성해 쓰기 지연 SQL 저장소에 전송
4. 쓰기 지연 저장소의 SQL을 DB에 전송
5. DB에서 트랜잭션 커밋

변경감지는 모든 필드를 업데이트 함

왜?

- 수정 쿼리가 항상 같으므로 애플리케이션 로딩 시점에 미리 생성해두고 재사용 가능
- 데이터베이스에 동일한 쿼리를 보내면 파싱된 쿼리 재사용 가능

→ 필드가 너무 많거나 저장되는 내용이 큰 경우에는 수정된 데이터만 사용해서 동적으로 UPDATE SQL 생성 전략 사용

⇒ @DynamicUpdate 어노테이션 활용

(참고 : @DynamicInsert 도 있음)

### 3.4.4. 엔티티 삭제

1. 영속성 컨텍스트에서 제거하고
2. 쓰기지연 SQL 저장소에 삭제 쿼리 저장

→ 삭제된 엔티티는 재사용하지 말고 GC 대상이 되도록 방치

## 3.5. 플러시

> 영속성 컨텍스트의 변경 내용을 DB에 반영
>

1. 변경 감지 동작
    1. 영속성 컨텍스트의 모든 엔티티 스냅샷과 비교
2. 쓰기지연 SQL 저장소의 쿼리를 DB에 전송

영속성 컨택스트 flush 하는 방법 3가지

1. 직접 em.flush() 호출
    1. 테스트 or 다른 프레임워크를 JPA와 함께 사용할 때 제외하고 거의 사용x
2. 트랜잭션 커밋시 자동 호출
    1. 반드시 영속성 컨텍스트의 변경 내용을 DB에 반영해야하므로 JPA 는 플러시 자동으로 호출하도록 설계됨
3. JPQL 쿼리 실행 시 자동 호출
    1. 객체지향 쿼리 호출 시에도 자동 실행

   왜?

   find와 다르게 직접 DB 값을 읽어오기 때문에

   → 근데 이러면 Transaction이 기존 Transactional 처럼 동작하나 ?


### 3.5.1. 플러시 모드 옵션

FlushModeType 사용

- FlushModeTypeAUTO : 커밋이나 쿼리 실행 시 플러시(기본값)
- FlushModeTypeCOMMIT : 커밋시에만 플러시

COMMIT 모드로 최적화 가능 → 10장에서 다룸

→ 영속성 컨텍스트에 보관된 엔티티를 지우고 DB에 넣는 방식이 아니라 변경 내용을 동기화 하는 개념

→ DB와의 동기화를 최대한 늦추는 것이 가능 한 이유는 Transaction이라는 작업단위가 존재하기 때문

## 3.6. 준영속

> 영속성 컨텍스트 제공하는 기능 사용 x (당연)
>

준영속 상태 만드는 법

1. em.detach(entity) : 특정 엔티티만 준영속 상태 전환
2. em.clear() : 영속성 컨텍스트 완전 초기화
3. em.close() : 영속성 컨텍스트 종료

### 3.6.1. detach()

→ 실행하는 순간

- 1차캐시
- 쓰기지연 SQL

해당 엔티티 관리하기 위한 모든 정보 제거

### 3.6.2. clear()

→ 해당 영속성 컨텍스트의 모든 엔티티를 준영속 상태로 만듦

### 3.6.3. close()

→ 영속성 컨텍스트 종료

### 3.6.4. 준영속 상태의 특징

- 거의 비영속 상태에 가깝다.
- 식별자 값을 가짐

  → 한번 영속상태였으므로 식별자 값을 가짐

- 지연 로딩 불가능
    - 실제 객체 대신 프록시 객체 로딩 후 실제 사용 시점에서 영속성 컨텍스트 통해서 불러오는 기법

### 3.6.5. merge()

→ 준영속 상태의 엔티티를 다시 영속 상태로 변경

사실은 새로운 영속 상태의 엔티티를 반환하는 것임

(다른 인스턴스임)

예시)

1. 영속 컨텍스트에 등록 커밋
2. 영속 해제(준영속) 전환
3. 값 변경(변경감지x)
4. 다시 영속 상태로 merge
5. 1차 캐시 조회 → 없음
6. DB에서 조회 후 영속
7. commit 시점에 변경 감지
8. 변경된 값 저장

새로운 인스턴스를 기존 인스턴스에서 참조하도록 하는 것이 안전

(merge 처리 된 새로운 인스턴스)

### 비영속 병합

→ 병합은 비영속, 준영속을 신경쓰지 않음

식별자 값으로 엔티티 조회 할 수 있으면

→ 불러서 병합

→ 조회할 수 없으면 새로 생성하여 병합

병합은 → save or update 기능 수행

# 04. 엔티티 매핑

**JPA에서 엔티티와 테이블 매핑하는 것이 가장 중요**

→ XML을 활용한 방식도 있지만 다루지 않음

- 객체와 테이블 매핑
    - @Entity , @Table
- 기본 키와 매핑
    - @Id
- 필드와 컬럼 매핑
    - @Column
- 연관관계 매핑
    - @ManyToOne
    - @JoinColumn

## 4.1. @Entity

> 테이블과 매핑할 클래스에 반드시 붙여야 하는 어노테이션
>

속성

| 속성 | 기능 | 기본값 |
| --- | --- | --- |
| name | JPA에서 사용할 엔티티 이름 지정. 보통은 기본 값인 클래스 이름 사용 → 다른 패키지 클래스와 충돌 주의 | 클래스 이름 |

주의사항

- 기본생성자 필수 (파라미터가 없는 public or protected)
- final 클래스, enum, interface, inner 클래스에는 사용 불가
- 저장할 필드에 final 사용하면 안됨

  → 기본생성자가 필수인데 final 을 사용할수가 없긴함


## 4.2. @Table

> 엔티티와 매핑할 테이블 지정
>

| 속성 | 기능 | 기본값 |
| --- | --- | --- |
| name | 매핑할 테이블 이름 | 엔티티 이름 |
| catalog | catalog 기능이 있는 데이터베이스에서 catalog 매핑 |  |
| schema | schema 기능이 있는 데이터베이스에서 schema 매핑 |  |
|  | DDL 생성 시 유니크 제약조건 만듦, 2개 이상 복합 유니크 제약 조건도 가능 
→ 스미카 자동생성 기능을 사용해 DDL 만드는 경우에만 사용됨 |  |

## 4.3. 다양한 매핑 사용

회원관리에서 추가되는 사항들

- 회원Type 구분
    - @Enumerated(EnumType.STRING) : 자바의 enum 을 활용해 타입 정의
- 가입일, 수정일 필요
    - @Temporal(TemproalType.TIMESTAMP) 로 날짜타입 매핑
- 회원 설명 필드(길이제한x)
    - @Lob을 사용하여 CLOB,BLOB 타입 매핑

## 4.4. 데이터베이스 스키마 자동 생성

JPA는 데이터베이스 스키마를 자동으로 생성하는 기능 지원

클래스의 매핑정보를 토대로 데이터 베이스 방언을 사용하여 스키마 생성

기존테이블을 삭제하고 다시 생성함 → 주의..

운영 환경에서 사용할만큼 완벽하지는 않음

→ 개발환경에서 매핑을 어떻게 해야하는지 참고하는 정도로만 사용

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8da4657-58ed-4483-8d1b-3efce9010f0f/05f9344f-f0b0-47e3-ad6d-e10dc7d4521a/Untitled.jpeg)

이름 매핑 전략 변경

→ DB는 관례상 스네이크

→ Java는 관례상 카멜

ImprovedNamingStrategy 클래스 제공(두가지를 컨버팅)

## 4.5. DDL 생성 기능

제약조건 추가 가능

ex) nullable = flase, legnth =10

@Table 의 uniqueConstraints 속성 등

DDL 자동 생성시에만 사용되고 JPA 실행 로직에는 영향 x

직접 DDL 만드는 경우 필요 없음

## 4.6. 기본 키 매핑

- 직접 할당 : 기본 키를 애플리케이션에서 직접 할당
- 자동 생성 : 대리 키 사용 방식
    - IDENTITY : 기본 키 생성을 DB에 위임
    - SEQUENCE : 데이터베이스 시퀀스를 사용해 기본키 할당
    - TABLE : 키 생성 테이블 사용

  → DB 벤더마다 지원하는 방식 다르기 때문

  ex) MySQL 은 시퀀스 지원x → AUTO_INCREMENT 기능 제공


자동생성 전략 사용 위해서는 @GeneratedValue 추가하고 생성 전략 정하면 됨

키 생성 전략 사용 시 persistence.xml 에 hibernate.id.new_generator_mappings=true 반드시 추가해야함

→ 새로운 키 생성 전략 개발 했는데 기존 호환성 때문에 기본 값을false로 둠

### 4.6.1. 기본 키 직접 할당 전략

직접 @Id로 매핑

사용 가능 타입

- 자바 기본형
- 자바 래퍼(Wrapper) 형
- String
- java.util.Date
- java.sql.Date
- java.math.BigDecimal
- java.math.BigInteger



기본 키 직접 할당하는 경우에는 키 없이 저장하면 예외 발생

### 4.6.2. IDENTITY 전략

DB에 기본키 생성 위임

ex) MySQL 의 AUTO_INCREMENT 기능 사용

→ 저장 하고 난 다음에 기본 키 값을 구할 수 있을 때 사용

@GeneratedValue 를 통해 지정

<aside>
💡 IDENTITY 전략과 최적화

데이터 저장 후에만 키 가져올 수 있으니 저장과 동시에 키 반환하도록 할 수 있음
Statement.getGenratedKeys()
→ MyBatis 의 selectKey 와 같은 역할

</aside>

⚡주의

- 엔티티가 영속화 되려면 식별자가 반드시 필요함
- IDENTITY 는 저장해야 식별자 구할 수 있음
- 무조건 INSERT SQL이 전달
- 트랜잭션을 지원하는 쓰기 지연이 동작하지 않음

### 4.6.3. SEQUENCE 전략

SEQUENCE 매핑하여 해당 시퀀스를 사용하는 전략

→ IDENETITY 와 비슷해보이지만 내부 동작이 다름

- SEQEUNCE 부터 조회하고 시퀀스를 획득 후
- ID에 매핑
- 영속성 컨텍스트에 저장
- Transaction Commit 시점에 Flush 해서 저장

⚡주의

allocationSize 가 기본값이 50임 ( 시퀀스 한번 호출에 증가하는 수 → 최적화 때문에)

DB상에서 하나씩 증가하도록 되어있다면 반드시 1로 설정해야함

50개의 시퀀스를 획득 한 후에

메모리에서 50개까지 할당하는 방식

→ INSERT 할 때마다 시퀀스를 조회하지 않으므로 최적화 가능



### 4.6.4. TABLE 전략

> 키 생성 전용 테이블 생성, 이름과 값으로 사용할 컬럼을 만들어DB 시퀀스를 흉내내는 전략
>

→ 시퀀스 없는 DB에서 이렇게 사용 가능 할 듯

### 4.6.5. AUTO 전략

데이터 베이스 방언에 따라

IDENTITY, SEQUENCE, TABLE 중 하나 자동으로 선택

→ 개발단계나 프로토타입에서 유용함(DB 의존적이지 않기에)

### 4.6.4. 기본 키 매핑 정리

- 직접 할당
    - 영속화 하기 전에 직접 Id 할당해야함
- SEQUENCE
    - 시퀀스에서 식별자 획득 후 영속화
- TABLE
    - 시퀀스와 동일
- IDENETITY
    - DB에 저장 후 식별자 획득 후 영속화

권장하는 식별자 선택 전략

- null 허용x
- 유일
- 불변

기본 키 선택 전략

- 자연 키
    - 비즈니스에 의미있는 키

  ex) 주민등록번호, 전화번호, 이메일

- 대리 키
    - 비즈니스와 관련 없는 임의로 만들어진 키

  ex) 시퀀스, auto_increment


### **대리 키를 권장**

왜 ?

→ 핸드폰 번호처럼 유일 할 수는 있어도

존재하지 않을 수 있음

→ 비즈니스 환경은 언젠가 변함

정부 정책이 변경된다거나..(주민번호를 key 로 헀는데 저장할 수 없게 된다면)

이런 자연키 대신 대리키 사용하고

유니크 인덱스를 활용하여 사용하는 것을 권장

## 4.7. 필드와 컬럼 매핑 : 레퍼런스

| 분류 | 매핑어노테이션 | 설명 |
| --- | --- | --- |
| 필드와 컬럼 매핑 | @Column | 컬럼 매핑 |
|  | @Enumerated | enum 매핑 |
|  | @Temporal | 날짜 매핑 |
|  | @Lob | BLOB, CLOB 매핑 |
|  | @Transient | 특정 필드 DB에 매핑하지 않음 |
| 기타 | @Access | JPA가 엔티티에 접근하는 방식 지정 |

### 4.7.1. @Column

name, nullable 주로 사용

insertable, updatable → 읽기만 하는 경우 사용 가능

<aside>
✨ 참고

@Column 생략하는 경우

→ @Column 속성의 기본 값이 적용 됨

이 때, 기본 타입인 경우 nullable 이므로

Column nullable을 false 로 기본 매핑

단, 기본 타입을 @Column으로 매핑하는 경우에는 nullable = false 를 추가하는 것이 안전함

</aside>

### 4.7.2. @Enumerated

Java의 ENUM 타입 매핑

- EnumType.ORDINAL
    - enum에 정의된 순서대로 0 , 1 이런 식으로 저장
    - 순서 바뀌면 변경되므로 웬만하면 쓰지말자
- EnumType.STRING
    - enum이름 그대로 ADMIN은 ‘ADMIN’ USER는 ‘USER’ 로 저장

### 4.7.3. @Temporal

날짜 타입을 매핑할 때 사용

- TemporalType.DATE
    - 날짜 (2023-01-01)
- TemporalType.TIME
    - 시간(11:10:10)
- TemporalType.TIMESTAMP
    - timestamp(2023-10-12 11:10:10)

### 4.7.4. @Lob

문자면 CLOB

이외는 BLOB 매핑

### 4.7.5. @Transient

매핑 안하는 임시 값 보관 하고 싶을 때 사용

### 4.7.6. @Access

JPA가 엔티티데이터에 접근하는 방식을 지정

- 필드 접근
    - AccessType.FIELD
        - private 이어도 접근 가능
    - AccessType.PROPERTY
        - 접근자(getter) 사용

  기본적으로는 @Id 가 어디에 있냐에 따라서 접근 달라짐

  필드에 있으면 → 필드

  getId에 있으면 → 접근자