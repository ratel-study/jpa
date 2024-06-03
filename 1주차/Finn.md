# Finn
JPA 스터디

---

## 3장 영속성 관리


JPA가 제공하는 기능은 크게 두가지이다.

- 엔티티와 테이블을 매핑하는 설계 부분
- 매핑한 엔티티를 실제 사용하는 부분

매핑한 엔티티는 **엔티티 매니저**를 통해 사용한다.

- 엔티티 매니저: 엔티티를 저장, 수정, 삭제, 조회 등 엔티티와 관련된 모든 일을 처리한다.

### 엔티티 매니저 팩토리와 엔티티 매니저

데이터 베이스를 하나만 사용하는 애플리케이션은 일반적으로 `EntityManagerFactory`를 하나만 생성한다.

```java
// 비용이 아주 많이든다.
final EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
```

`Persistence.createEntityManagerFactory("jpabook")`를 호출하면 설정 정보를 바탕으로 `EntityManagerFactory` 를 생성한다.

그 후 엔티티 매니저 팩토리에서 엔티티 매니저를 생성하면 된다.

```java
// 팩토리에서 매니저 생성, 비용이 거의 안듬
final EntityManager em = emf.createEntityManager();
```

엔티티 매니저 팩토리는 **엔티티 매니저를 생성하는 공장** 이다.

팩토리를 생성할 때 드는 비용은 상당히 크기 때문에 한 개만 만들어 애플리케이션 전체에서 공유하도록 설계되어있다.

팩토리에서 매니저를 생성하는 비용은 **거의 들지 않는다.**

**팩토리는 여러 스레드가 동시에 접근해도 안전하므로 스레드 간에 공유가 가능하다.**

**매니저는 여러 스레드가 동시에 접근하면 동시성 문제가 발생하기 때문에 스레드 간에 절대 공유하면 안된다.**

엔티티 매니저는 데이터 베이스 연결이 꼭 필요한 시점까지 커넥션을 얻지 않는다.

보통 트랜잭션을 시작할 때 커넥션을 획득한다.

JPA 구현체들은 팩토리를 생성할 때 **커넥션 풀** 도 만든다.


### 영속성 컨텍스트란?

영속성 컨텍스트는 **엔티티를 영구 저장하는 환경**이다.

엔티티 매니저로 엔티티를 저장하거나 조회하면 엔티티 매니저는 영속성 컨텍스트에 엔티티를 보관하고 관리한다.

```java
em.persist(member);
```

`persist` 메소드는 엔티티 매니저를 사용해 회원 엔티티를 **영속성 컨텍스트에 저장**한다.

영속성 컨텍스트는 엔티티 매니저를 생성할 때 하나 만들어진다.

매니저를 통해 영속성 컨텍스트에 접근할 수 있고, 영속성 컨텍스트를 관리할 수 있다.

### 엔티티의 생명주기

엔티티에는 4가지 상태가 존재한다.

- 비영속 : 영속성 컨텍스트와 전혀 관계가 없는 상태
- 영속 : 영속성 컨텍스트에 저장된 상태
- 준영속 : 영속성 컨텍스트에 저장되었다가 분리된 상태
- 삭제 : 삭제된 상태

### 비영속

엔티티 객체를 생성한 상태.

아직은 순수한 객체 상태이며 영속성 컨텍스트나 데이터 베이스와는 전혀 관련이 없다.

```java
    // 비영속 상태 - 객체를 생성한 상태
    Member member = new Member();
    member.setName("test");
```

### 영속

엔티티 매니저를 통해 엔티티를 영속성 컨텍스트에 저장한 상태

**영속성 컨텍스트가 관리하는 엔티티를 영속 상태**라고 한다.

영속 상태는 영속성 컨텍스트에 의해 관리된다는 뜻이다.

`em.find`나 `JPQL`을 사용해서 조회한 엔티티도 영속성 컨텍스트가 관리하는 영속 상태다.

```java
    // 비영속 상태 - 객체를 생성한 상태
    Member member = new Member();
    member.setName("test");

    // 영속 상태 - 엔티티 매니저를 통해 영속성 컨텍스트에 저장한 상태
    em.persist(member);
```

### 준영속

영속성 컨텍스트가 관리하던 영속 상태의 엔티티를 영속성 컨텍스트가 관리하지 않으면 **준영속 상태**가 된다.

특정 엔티티를 준영속 상태로 만들려면 `em.detach()`를 호출하면 된다.

`em.close()`를 호출해서 영속성 컨텍스트를 닫거나 `em.clear()`를 호출해서 영속성 컨텍스트를 초기화해도 영속 상태의 엔티티는 준영속 상태가 된다.

```java
    // 비영속 상태 - 객체를 생성한 상태
    Member member = new Member();
    member.setName("test");

    // 영속 상태 - 엔티티 매니저를 통해 영속성 컨텍스트에 저장한 상태
    em.persist(member);
    
    // 준영속 상태 - 엔티티를 영속성 컨텍스트에서 분리한 상태
    em.detach(member);
    em.close();
    em.clear();
```

### 삭제

엔티티를 영속성 컨텍스트와 데이터 베이스에서 삭제한다.

```java
    // 비영속 상태 - 객체를 생성한 상태
    Member member = new Member();
    member.setName("test");

    // 영속 상태 - 엔티티 매니저를 통해 영속성 컨텍스트에 저장한 상태
    em.persist(member);

    // 준영속 상태 - 엔티티를 영속성 컨텍스트에서 분리한 상태
    em.detach(member);
    em.close();
    em.clear();
    
    // 삭제 - 객체를 삭제한 상태
    em.remove(member);
```

### 영속성 컨텍스트의 특징

영속성 컨텍스트의 특징은 다음과 같다.

- 영속성 컨텍스트와 식별자 값
    - 영속성 컨텍스트는 엔티티를 식별자 값(@Id)으로 구분한다.
    - 영속 상태는 반드시 식별자 값이 있어야 한다.
- 영속성 컨텍스트와 데이터베이스 저장
    - 영속성 컨텍스트에 엔티티를 저장하면 트랜잭션을 커밋하는 순간 새로 저장된 엔티티를 데이터베이스에 반영한다.
    - 이것을 Flush라고 한다.
- 영속성 컨텍스트가 엔티티를 관리하면 다음과 같은 장점이 있다.
    - 1차 캐시
    - 동일성 보장
    - 트랜잭션을 지원하는 쓰기 지연
    - 변경 감지
    - 지연 로딩

### 엔티티 조회

영속성 컨텍스트는 내부에 캐시를 가지고 있는데 이를 **1차 캐시**라 한다.

엔티티는 모두 이곳에 저장된다.

1차 캐시의 키는 식별자 값(@Id)이다.

영속성 컨텍스트에 데이터를 저장하고 조회하는 모든 기준은 데이터베이스 기본 키 값이다.

```java
Member member = new Memeber();
member.setId("member1");

// find(entityClass, primaryKey)
Member member = em.find(Member.class, "member1");
```

`find()` 메소드를 호출하면 먼저 1차 캐시에서 엔티티를 찾고 만약 엔티티가 1차 캐시에 없으면 데이터 베이스에서 조회한다.

이때 데이터 베이스에서 조회한 후 1차 캐시에 저장한 후에 영속 상태의 엔티티를 반환하게 된다.

그 후 같은 엔티티를 조회하면 메모리에 있는 1차 캐시에서 불러오게 되어 성능상 이점을 누릴 수 있다.

### 영속 엔티티의 동일성 보장

```java
Member a = em.find(Member.class, "member1");
Member b = em.find(Member.class, "member1");

System.out.println(a == b); // 동일성 비교
```

`a == b`는 참이다.

영속성 컨텍스트는 성능상 이점과 엔티티의 동일성을 보장한다.

JPA는 1차 캐시를 통해 **반복 가능한 읽기(REPEATABLE READ) 등급의 트랜잭션 격리 수준을 애플리케이션 차원에서 제공**한다는 장점이 있다.

- 동일성 : 실제 인스턴스가 같다. `==` 의 값이 같다.
- 동등성 : 실제 인스턴스는 다를 수 있지만 가지고 있는 값이 같다.
    - `equals()` 메소드를 통해 비교한다.

### 엔티티 등록

```java
    Member member = new Member();
    member.setName("test");
    EntityTransaction transaction = em.getTransaction();
    // 엔티티 매니저는 데이터 변경 시 트랜잭션을 시작해야한다.
    transaction.begin();
    
    em.persist(member);
    
    // 여기까지 INSERT SQL 을 데이터 베이스에 보내지 않는다.
    
    // 커밋하는 순간 데이터베이스에 INSERT SQL을 보낸다.
    transaction.commit();
```

엔티티 매니저는 트랜잭션을 커밋하기 직전까지 데이터베이스에 엔티티를 저장하지 않는다.

내부 쿼리 저장소에 INSERT SQL을 차곡차곡 모아둔 뒤 트랜잭션을 커밋할 때 모아둔 쿼리를 데이터베이스에 보낸다.

이를 **트랜잭션을 지원하는 쓰기 지연**이라 한다.

INSERT SQL은 1차 캐시에 저장하는 순간 만들어져 **쓰기 지연 SQL 저장소**에 보관된다.

트랜잭션을 커밋하면 엔티티 매니저는 영속성 컨텍스트를 플러시한다.

플러시는 영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화하는 작업이다.

플러시 작업은 쓰기 지연 SQL 저장소에 모인 쿼리를 데이터베이스에 보낸다.

영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화한 후에 실제 데이터베이스 트랜잭션을 커밋한다.

트랜잭션 범위 내에서 실행되므로 어느 시점에 쿼리를 보내도 결과는 같다.

어떻게든 커밋 직전에만 데이터베이스에 SQL을 전달하면 되기 때문에 트랜잭션을 지원하는 쓰기 지연이 가능하다.

이 기능을 잘 활용하면 성능 최적화를 할 수 있다.

### 엔티티 수정

SQL을 사용하면 수정 쿼리를 직접 작성해야 한다.

프로젝트가 커지고 요구사항이 늘어나면서 수정 쿼리도 점점 추가되는데

예를들어 회원의 이름과 나이를 변경하는 기능을 개발했는데 회원의 등급을 변경하는 기능이 추가된다면 이에 해당하는 수정 쿼리를 추가로 작성해야한다.

유저의 정보를 모두 변경하는 하나의 쿼리를 만들어 사용하는 방법도 있지만 실수의 여지가 많다.

때문에 요구 사항에 맞는 수정 쿼리를 상황에 따라 계속 추가하게 되는데 이런 방식은

수정 쿼리가 많아지는 것은 물론이고 비즈니스 로직을 분석하기 위해 SQL을 계속 확인해야 한다.

직접적이든 간접적이든 비즈니스 로직이 SQL에 의존하게 된다.

JPA에서는 다음과 같이 엔티티를 수정한다.

```java
    EntityTransaction transaction = em.getTransaction();
    transaction.begin();
    
    // 영속 엔티티 조회 및 수정
    Member member = em.find(Member.class, 1L);
    member.setName("test");
    
    transaction.commit();
```

JPA로 엔티티를 수정할 때는 단순히 엔티티를 조회한 후 데이터만 변경하면 된다.

이렇게 엔티티의 변경사항을 데이터베이스에 자동으로 반영하는 기능을 **변경감지**라 한다.

JPA는 엔티티를 영속성 컨텍스트에 보관할 때 최초 상태를 복사해서 저장해두는데 이걸 **스냅샷**이라고 한다.

플러시 시점에 스냅샷과 엔티티를 비교해 변경된 엔티티를 찾는다.

순서는 다음과 같다.

1. 트랜잭션 커밋시 엔티티 매니저 내부에서 먼저 Flush가 호출된다.
2. 엔티티와 스냅샷을 비교해 변경된 엔티티를 찾는다.
3. 변경된 엔티티가 있으면 수정 쿼리를 생성해 쓰기 지연 SQL 저장소에 보낸다.
4. 쓰기 지연 저장소의 SQL을 데이터베이스에 보낸다.
5. 데이터베이스 트랜잭션을 커밋한다.

변경 감지는 영속성 컨텍스트가 관리하는 **영속 상태의 엔티티에만 적용**된다.

JPA의 기본 전략은 엔티티의 모든 필드를 업데이트하는 것이다.

데이터베이스에 보내는 데이터 전송량이 증가하는 단점이 있지만, 다음과 같은 장점이 있다.

- 모든 필드를 사용하면 수정 쿼리가 항상 같다.
    - 애플리케이션 로딩 시점에 수정 쿼리를 미리 생성해두고 재사용할 수 있다.
- 데이터베이스에 동일한 쿼리를 보내면 데이터베이스는 이전에 한 번 파싱된 쿼리를 재사용할 수 있다.

필드가 많거나 저장되는 내용이 너무 크면 수정된 데이터만 사용해서 동적으로 UPDATE SQL을 생성하도록 전략을 선택하면 된다.

`@DynamicUpdate`를 사용하면 수정된 데이터만 사용해 동적으로 UPDATE SQL을 생성한다.

데이터를 저장할 때 데이터가 존재하는 필드만으로 INSERT SQL을 동적으로 생성하는 `@DynamicInsert`도 있다.

### 엔티티 삭제

엔티티를 삭제하려면 먼저 삭제 대상 엔티티를 조회해야 한다.

```java
    Member member = em.find(Member.class, 1L);
    // 엔티티 삭제
    em.remove(member);
```

`remove()`메소드의 인자로 엔티티를 넘겨주면 대상 엔티티를 삭제한다.

즉시 삭제하는 것이 아니라 삭제 쿼리를 쓰기 지연 SQL 저장소에 등록한다.

이후 트랜잭션 커밋시 플러시를 호출하면 실제 데이터베이스에 삭제 쿼리를 전달한다.

`remove()`를 호출하는 순간에 `member`는 영속성 컨텍스트에서 젲거된다.

이렇게 삭제된 엔티티는 자연스럽게 가비지 컬렉션의 대상이 되도록 두는 것이 좋다.

### 플러시

플러시(Flush)는 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영한다.

플러시 실행시 다음과 같은 과정으로 작업이 처리된다.

1. 변경 감지가 동작해 영속성 컨텍스트에 있는 모든 엔티티를 스냅샷과 비교해서 수정된 엔티티를 찾는다.
    1. 수정된 엔티티는 수정쿼리를 만들어 쓰기 지연 SQL 저장소에 등록한다.
2. 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송한다.

영속성 컨텍스트를 플러시하는 방법은 다음 3가지이다.

- `em.flush`를 직접 호출한다.
    - 메소드를 직접 호출해 영속성 컨텍스트를 강제로 플러시한다.
- 트랜잭션 커밋 시 플러시가 자동 호출된다.
    - JPA는 트랜잭션을 커밋할 때 플러시를 자동으로 호출한다.
- JPQL 쿼리 실행 시 플러시가 자동 호출된다.
    - JPQL 호출 시 플러시가 실행된다.

식별자를 기준으로 조회하는 `find`메소드를 호출할 때는 플러시가 실행되지 않는다.

### 플러시 모드 옵션

플러시에는 두가지 옵션이 있다.

- FlushModeType.AUTO: 커밋이나 쿼리를 실행할 때 플러시 (기본값)
- FlushModeType.COMMIT: 커밋할 때만 플러시

플러시 모드를 별도로 설정하지 않으면 `AUTO`로 동작한다.

`COMMIT`모드는 성능 최적화를 위해 사용할 수 있다.

플러시는 영속성 컨텍스트에 보관된 엔티티를 지우는게 아닌 **영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화하는 것이다.**

### 준영속

영속성 컨텍스트가 관리하는 영속 상태의 엔티티가 영속성 컨텍스트에서 **분리된 것**을 **준영속 상태**라 한다.

**준영속 상태의 엔티티는 영속성 컨텍스트가 제공하는 기능을 사용할 수 없다.**

영속 상태의 엔티티를 준영속 상태로 만드는 방법은 크게 3가지이다.

1. `em.detach(entity)` : 특정 엔티티만 준영속 상태로 전환한다.
2. `em.clear()` : 영속성 컨텍스트를 완전히 초기화한다.
3. `em.close()` : 영속성 컨텍스트를 종료한다.

영속 상태의 엔티티는 주로 영속성 컨텍스트가 종료되면서 준영속 상태가 된다.

개발자가 직접 준영속 상태로 만드는 일은 드물다.

### 준영속 상태의 특징

- 영속성 컨텍스트가 관리하지 않으므로 1차 캐시, 쓰기 지연, 변경 감지, 지연 로딩을 포함한 영속성 컨텍스트가 제공하는 어떠한 기능도 동작하지 않는다.
- 비영속 상태는 식별자 값이 없을 수도 있지만 준영속 상태는 이미 한 번 영속 상태였으므로 반드시 식별자 값을 가지고 있다.
- 지연 로딩 시 문제가 발생한다.

### 병합: merge()

준영속 상태의 엔티티를 다시 영속 상태로 변경하려면 병합을 사용하면 된다.

`merge(entity)`메소드는 준영속 상태의 엔티티를 받아 그 정보로 **새로운 영속 상태의 엔티티를 반환**한다.

`merge(entity)`의 동작 방식은 다음과 같다.

1. `merge(entity)`를 실행한다.
2. 파라미터로 넘어온 준영속 엔티티의 식별자 값으로 1차 캐시에서 엔티티를 조회한다.
    1. 1차 캐시에 엔티티가 없으면 데이터베이스에서 엔티티를 조회하고 1차 캐시에 저장한다.
3. 조회한 영속 엔티티에 파라미터로 넘어온 준영속 엔티티의 값을 채워 넣는다.
4. 조회한 영속 엔티티를 반환한다.

### 비영속 병합

`merge()`메소드는 파라미터로 넘어온 엔티티의 식별자 값으로 영속성 컨텍스트를 조회하고 찾는 엔티티가 없으면 데이터베이스에서 조회하는데 만약 데이터베이스에서도 발견하지 못하면 새로운 엔티티를 생성해서 병합한다.

병합은 `SAVE or UPDATE` 기능을 수행한다.

---

## 4장 엔티티 매핑


JPA를 사용하는 데 가장 중요한 일은 엔티티와 테이블을 정확히 매핑하는 것이다.

JPA는 다양한 매핑 어노테이션을 지원하는데 크게 4가지로 분류할 수 있다.

- 객체와 테이블 매핑 : `@Entity`, `@Table`
- 기본 키 매핑 : `@Id`
- 필드와 컬럼 매핑 : `@Column`
- 연관관계 매핑 : `@ManyToOne`, `@JoinColumn`

### @Entity

JPA를 사용해서 테이블과 매핑할 클래스는 `@Entity`를 **필수로 붙여야한다.**

`@Entity`가 붙은 클래스는 JPA가 관리하는 것으로 엔티티라 부른다.

`@Entity` 적용시 주의사항은 다음과 같다.

- 기본 생성자는 필수 (파라미터가 없는 `public` 또는 `protected` 생성자)
- `final` 클래스, `enum`, `interface`, `inner` 클래스에는 사용할 수 없다.
- 저장할 필드에 `final`을 사용하면 안 된다.

### @Table

`@Table`은 엔티티와 매핑할 테이블을 지정한다.

기본값은 엔티티 이름을 테이블 이름으로 사용한다.

`@Table`의 속성은 다음과 같다.

- name : 매핑할 테이블 이름
- catelog: catelog 기능이 있는 데이터베이스에서 catelog를 매핑한다.
- schema: schema 기능이 있는 데이터베이스에서 schema를 매핑한다.
- uniqueConstraints: DDL 생성 시에 유니크 제약 조건을 만든다.
    - 2개 이상의 복합 유니크 제약 조건도 만들 수 있다.
    - 스키마 자동 생성 기능을 사용해서 DDL을 만들 때만 사용된다.

### 데이터베이스 스키마 자동 생성

JPA는 데이터베이스 스키마를 자동으로 생성하는 기능을 지원한다.

```java
hibernate.hbm2ddl.auto
```

JPA에 스키마 자동 생성 기능을 설정하면 애플리케이션 실행 시점에 데이터베이스 테이블을 자동으로 생성한다.

자동 생성되는 DDL은 지정한 데이터베이스 방언에 따라 달라진다.

스키마 자동 생성 기능이 만든 DDL은 운영 환경에서 사용할 만큼 완벽하지 않기 때문에 개발 환경에서 참고하는 정도로만 사용하는 것이 좋다.

`hibernate.hbm2ddl.auto` 의 속성은 다음과 같다.

- create: 기존 테이블을 삭제하고 새로 생성한다.
- create-drop: create 속성에 추가로 애플리케이션을 종료할 때 생성한 DDL을 제거한다.
- update: 데이터베이스 테이블과 엔티티 매핑정보를 비교해 변경 사항만 수정한다.
- validate: 데이터베이스 테이블과 엔티티 매핑정보를 비교해 차이가 있으면 경고를 남기고 애플리케이션을 실행하지 않는다. (이 설정은 DDL을 수정하지 않는다.)
- none: 자동 생성 기능을 사용하지 않으려면 속성 자체를 삭제하거나 유효하지 않은 옵션 값을 주면 된다. (none은 유효하지 않은 옵션 값이다.)

### DDL 생성 기능

회원 이름은 필수로 입력되어야 하고, 10자를 초과하면 안된다는 조건을 DDL 자동 생성으로 적용하는 코드는 다음과 같다.

```java
@Entity
@Table(name = "members")
public class Member {
    @Id
    private String id;

    @Column(name = "name", nullable = false, length = 10)
    private String name;
}    
```

`nullable`을 `false`로 지정하면 `not null` 제약 조건을 추가할 수 있다.

```java
@Entity
@Table(name = "members",
uniqueConstraints = {
        @UniqueConstraint(
                name = "NAME_UNIQUE",
                columnNames = {"name"}
        )
})
public class Member {
  public Member() {
  }

  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  @Column(name = "name", nullable = false, length = 10)
  private String name;
}  
```

`@Table`의 `uniqueConstraints` 속성을 사용해 유니크 제약 조건을 추가할 수 있다.

이러한 기능들은 단지 DDL을 자동 생성할 때만 사용되고 **JPA의 실행 로직에는 영향을 주지 않는다.**

JPA 애플리케이션의 실행 동작에는 영향을 주지 않지만 개발자가 엔티티만 보고도 쉽게 지약 조건을 파악할 수 있는 장점이 있다.

### 기본 키 매핑

JPA가 제공하는 데이터베이스 기본 키(primary key) 생성 전략은 다음과 같다.

- 직접 할당: 기본 키를 애플리케이션에서 직접 할당한다.
- 자동 생성: 대리 키 사용 방식
    - `IDENTITY`: 기본 키 생성을 데이터베이스에 위임한다.
    - `SEQUENCE`: 데이터베이스 시퀀스를 사용해서 기본 키를 할당한다.
    - `TABLE`: 키 생성 테이블을 사용한다.

`SEQUENCE`나 `IDENTITY` 전략은 사용하는 데이터베이스에 의존한다.

`TABLE` 전략은 키 생성용 테이블을 하나 만들어두고 시퀀스처럼 사용하는 방법이기 때문에 모든 데이터베이스에서 사용할 수 있다.

기본 키를 직접 할당하려며 `@Id`만 사용하면 되고 자동 생성 전략을 사용하려면

`@Id`에 `@GeneratedValue`를 추가하고 원하는 키 생성 전략을 선택하면 된다.

키 생성 전략을 사용하려면 다음 속성을 반드시 추가해야 한다.

```java
hibernate.id.new_generator_mapping=true
```

### 기본 키 직접 할당 전략

기본 키를 직접 할당하려면 `@Id`로 매핑하면 된다.

```java
@Id
@Column(name = "id")
private String id;
```

@Id 적용 가능 자바 타입은 다음과 같다.

- 자바 기본형
- 자바 래퍼 형
- Stirng
- `java.util.Date`
- `java.sql.Date`
- `java.math.BigDecimal`
- `java.math.BigInteger`

기본 키 직접 할당 전략은 `em.persist()`로 엔티티를 저장하기 전에 애플리케이션에서 기본 키를 직접 할당해야한다.

```java
Board board = new Board();
board.setId("id1");
em.persist(board);
```

기본 키 직접 할당 전략에서는 식별자 값 없이 저장하면 **예외가 발생한다.**

### IDENTITY 전략

`IDENTITY` 전략은 기본 키 생성을 데이터베이스에 위임하는 전략이다.

주로 `MySQL`, `PostgreSQL`, `DB2` 에서 사용한다.

`MySQL`의 `AUTO_INCREMENT` 기능은 데이터베이스가 기본 키를 자동으로 생성해준다.

`IDENTITY` 전략은 **데이터베이스에 값을 저장하고 나서야 기본 키 값을 구할 수 있을 때 사용한다.**

`IDENTITY` 전략을 사용하려면 `@GeneratedValue`의 `strategy` 속성 값을 `GenerationType.IDENTITY`로 지정하면 된다.

이 전략을 사용하면 JPA는 기본 키 값을 얻어오기 위해 데이터베이스를 추가로 조회한다.

`IDENTITY` 전략은 데이터를 데이터베이스에 `INSERT`한 후에 기본 키 값을 조회할 수 있다.

`JDBC3`에 추가된 `Statement.getGeneratedKeys()`를 사용하면 데이터를 저장하면서 동시에 생성된 기본 키 값도 얻어 올 수 있다.

엔티티가 영속 상태가 되려면 식별자가 반드시 필요하기 때문에 `em.persist()`를 호출하는 즉시 `INSERT SQL`이 데이터베이스에 전달된다.

`IDENTITY` 전략은 **트랜잭션을 지원하는 쓰기 지연이 동작하지 않는다.**

### SEQUENCE 전략

데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트다.

`SEQUENCE` 전략은 이 시퀀스를 사용해서 기본 키를 생성한다.

이 전략은 시퀀스를 지원하는 오라클, PostgreSQL, H2 데이터베이스에서 사용할 수 있다.

```java
@Entity
@SequenceGenerator(
        name = "BOARD_SEQ_GENERATOR",
        sequenceName = "BOARD_SEQ",
        initialValue = 1, allocationSize = 1
)
public class Board {

  @Id
  @GeneratedValue(strategy = GenerationType.SEQUENCE,
  generator = "BOARD_SEQ_GENERATOR")
  private Long id;
  
  public Board() {
  }
}
```

`@SequenceGenerator`를 사용해 시퀀스 생성기를 등록한다.

`sequenceName` 속성의 이름으로 JPA는 시퀀스 생성기를 실제 데이터베이스의 시퀀스와 매핑한다.

`SEQUENCE` 전략은 `em.persist()`를 호출할 때 먼저 데이터베이스 시퀀스를 사용해서 식별자를 조회한다.

조회한 식별자를 엔티티에 할당한 후에 엔티티를 영속성 컨텍스트에 저장하고 트랜잭션을 커밋해서 플러시가 일어나면 엔티티를 데이터베이스에 저장한다.


### @SequenceGenerator

@SequenceGenerator의 속성은 다음과 같다.

- name: 식별자 생성기 이름
- sequenceName: 데이터베이스에 등록되어 있는 시퀀스 이름
- initialValue: DDL 생성시에만 사용됨, 시퀀스 DDL을 생성할 때 처음 시작하는 수
- allocationSize: 시퀀스 한 번 호출에 증가하는 수(기본값 50)
- catelog, schema: 데이터베이스 catelog, schema 이름

`SEQUENCE` 전략은 데이터베이스 시퀀스를 통해 식별자를 조회하는 추가 작업이 필요하다.

때문에 데이터베이스와 2번 통신하게 된다.

1. 식별자를 구하려고 데이터베이스 시퀀스를 조회한다.
2. 조회한 시퀀스를 기본 키 값으로 사용해 데이터베이스에 저장한다.

JPA에서는 시퀀스 접근을 최소화하기 위해 `allocationSize`를 사용한다.

`allocationSize`값이 50이면 시퀀스를 한 번에 50 증가 시키고 1 ~ 50까지는 메모리에서 식별자를 할당한다.

이 방법은 JVM이 여러 대 동작해도 기본 키 값이 충돌하지 않는다는 장점이 있다.

하지만 데이터베이스에 직접 접근해 데이터를 등록할 때 시퀀스 값이 한 번에 많이 증가한다는 점에 주의해야한다.


현재 까지의 최적화 방법은 `hibernate.id.new_generator_mapping=true` 으로 설정되어 있어야 동작한다.

`@SequenceGenerator`는 `@GeneratedValue` 옆에 사용해도 된다.

### TABLE 전략

`TABLE` 전략은 키 생성 전용 테이블을 하나 만들고 이름과 값으로 사용할 컬럼을 만들어 데이터베이스 시퀀스를 **흉내내는 전략**이다.

이 전략은 모든 데이터베이스에 적용할 수 있다.

`TABLE` 전략을 사용하려면 키 생성 용도로 사용할 테이블을 만들어야 한다.

```java
@Entity
@TableGenerator(
        name = "BOARD_SEQ_GENERATOR",
        table = "MY_SEQUENCES",
        pkColumnValue = "BOARD_SEQ", allocationSize = 1
)
public class Board {

  @Id
  @GeneratedValue(strategy = GenerationType.TABLE,
          generator = "BOARD_SEQ_GENERATOR")
  private Long id;

  public Board() {
  }
}
```

위처럼 작성하면 `Id` 식별자 값은 `BOARD_SEQ_GENERATOR` 테이블 키 생성기가 할당된다.

`TABLE` 전략은 시퀀스 대신에 테이블을 사용한다는 것만 제외하면 `SEQUENCE` 전략과 내부 동작방식이 같다.

시퀀스 용 테이블에 값이 없으면 JPA 가 `INSERT`하면서 초기화하므로 값을 미리 넣어둘 필요는 없다.

### @TableGenerator

`@TableGenerator`의 속성은 다음과 같다.

- name: 식별자 생성기 이름
- table: 키 생성 테이블명
- pkColumnName: 시퀀스 컬럼 명
- valueColumnName: 시퀀스 값 컬럼명
- pkColumnValue: 키로 사용할 값 이름
- initialValue: 초기 값, 마지막으로 생성된 값이 기준이다.
- allocationSize: 시퀀스 한 번 호출에 증가하는 수(기본값 50)
- catelog, schema: 데이터베이스 catelog, schema 이름
- uniqueConstraints(DDL): 유니크 제약 조건을 지정할 수 있다.

`TABLE` 전략은 값을 조회하면서 `SELECT` 쿼리르 사용하고 다음으로 값을 증가시키기 위해 `UPDATE` 쿼리를 사용한다.

최적화하는 방법은 `SEQUENCE` 전략과 동일하다.

### AUTO 전략

`AUTO`는 선택한 데이터베이스 방언에 따라 `IDENTITY`, `SEQUENCE`, `TABLE` 전략 중 하나를 자동으로 선택한다.

오라클이라면 `SEQUENCE`를 MySQL이라면 `IDENTITY`를 사용한다.

`AUTO` 전략의 장점은 데이터베이스를 변경해도 코드를 수정할 필요가 없다는 것이다.

`AUTO`를 사용할 때 `SEQUENCE`, `TABLE` 전략이 선택되면 시퀀스나 키 생성용 테이블을 미리 만들어두어야 한다.

스키마 자동 생성 기능을 사용한다면 하이버네이트가 기본값을 사용해 적절한 테이블을 만들어 준다.

### 기본 키 매핑 정리

데이터베이스의 기본 키는 다음 3가지 조건을 모두 만족해야 한다.

- `null`값은 허용하지 않는다.
- 유일해야 한다.
- 변해선 안 된다.

테이블의 기본 키를 선택하는 전략은 크게 2가지가 있다.

- 자연 키
    - 비즈니스에 의미가 있는키
    - 주민등록번호, 이메일, 전화번호
- 대리 키
    - 비즈니스와 관련 없는 임의로 만들어진 키(대체 키)
    - 오라클 시퀀스, auto_increment, 키 생성 테이블

비즈니스 환경은 언젠가 변하기 때문에 **자연 키보다는 대리 키를 권장**한다.

기본 키는 변하면 안 된다는 기본 원칙으로 인해 기본 키 값을 변경하려고 하면 JPA에서 예외를 발생시키거나 정상 동작하지 않는다.

### 필드와 컬럼 매핑: 레퍼런스

### @Column

`@Column`은 객체 필드를 테이블 컬럼에 매핑한다.

속성 중 `name`과 `nullable`이 주로 사용된다.

`@Column`을 생략하게 되면 기본 값이 적용되는데 자바 기본 타입일 때는 `nullable` 속성에 예외가 있다.

```java
int data1 // @Column 생략, 자바 기본 타입
data1 integer not null // 생성된 DDL

Integer data2 // @Column 생략, 객체 타입
data2 integer // 생성된 DDL

@Column
int data3 // @Column 사용, 자바 기본 타입
data3 integer // 생성된 DDL
```

JPA에서는 DDL 생성 기능을 사용할 때 `int data1` 같은 기본 타입에는 `not null` 제약조건을 추가한다.

`Integer data2` 처럼 객체 타입이면 `null`이 입력될 수 있으므로 `not null` 제약조건을 설정하지 않는다.

`int data3` 처럼 `@Column`을 사용하면 `@Column`은 `nullable = true`가 기본 값이므로 `not null` 제약조건을 설정하지 않는다.

자바 기본 타입에 `@Column`을 사용하면 `nullable = false`로 지정하는 것이 안전하다.

### @Enumerated

자바의 `enum` 타입을 매핑할 때 사용한다.

`@Enumerated`의 속성은 다음과 같다.

- value:
    - `EnumType.ORDINAL`: `enum` 순서를 데이터베이스에 저장
    - `EnumType.STRING`: `enum` 이름을 데이터베이스에 저장


`EnumType.ORDINAL`은 `enum`에 정의된 순서대로 데이터베이스에 저장한다.
- 장점: 데이터베이스에 저장되는 크기가 작다.
- 단점: 이미 저장된 `enum`의 순서를 변경할 수 없다.

`EnumType.STRING`은 `enum`이름 그대로 문자를 데이터베이스에 저장한다.
- 장점: 저장된 `enum`의 순서가 바뀌거나 `enum`이 추가되어도 안전하다.
- 단점: 데이터베이스에 저장되는 데이터 크기가 `ORDINAL`에 비해서 크다.

`EnumType.STRING`을 사용하는 것을 권장한다.

### @Temporal

날짜 타입을 매핑할 때 사용한다.

속성은 다음과 같다.

- value:
    - `TemporalType.DATE`: 날짜, 데이터베이스 `data`타입과 매핑 (예ㅣ 2013-10-11)
    - `TemporalType.TIME`: 시간, 데이터베이스 `time`타입과 매핑 (예ㅣ 11:11:11)
    - `TemporalType.TIMESTAMP`: 날짜와 시간, 데이터베이스 `timestamp`타입과 매핑 (예ㅣ 2013-10-11 11:11:11)

`@Temporal`을 생략하면 자바의 `Date`와 가장 유사한 `timestamp`로 정의된다.

`timestamp` 대신 `datetime`을 예약어로 사용하는 데이터베이스도 있는데 방언 덕분에 애플리케이션 코드는 변경하지 않아도 된다.

### @Lob

데이터베이스 `BLOB`과 `CLOB` 타입과 매핑한다.

`@Lob` 에는 지정할 수 있는 속성이 없다.

대신 매핑하는 필드 타입이 문자면 `CLOB` 으로 매핑하고 나머지는 `BLOB` 으로 매핑한다.

- CLOB: `String`, `char[]`, `java.sql.CLOB`
- BLOB: `byte[]`, `java.sql.BLOB`

### @Transient

`@Transient`이 붙은 필드는 매핑하지 않는다.

### @Access

`@Access`은 JPA가 엔티티 데이터에 접근하는 방식을 지정한다.

- 필드 접근: `AccessType.FIELD`로 지정한다.
    - 필드에 직접 접근한다.
    - `privatge`권한이어도 접근할 수 있다.
- 프로퍼티 접근: `AccessType.PROPERTY`로 지정한다.
    - 접근자 `getter`를 사용한다.

`@Access`를 설정하지 않으면 `@Id`의 위치를 기준으로 접근 방식이 설정된다.

```java
@Entity
@Access(AccessType.FIELD)
public class Member {
    @Id
    private Stirng id;
}
```

`@Id`가 필드에 있으므로 `@Access(AccessType.FIELD)`을 설정한 것과 같다.

따라서 `@Access(AccessType.FIELD)`을 생략해도 된다.

```java
@Entity
@Access(AccessType.PROPERTY)
public class Member {
    private Stirng id;

    @Id
    public String getId() {

    }
}
```

`@Id`가 `getter`에 있으므로 `@Access(AccessType.PROPERTY)` 설정한 것과 같다.

따라서 `@Access(AccessType.PROPERTY)`을 생략해도 된다.


```java
@Entity
public class Member {
    @Id
    private Stirng id;

    @Transient
    private String firstName;

    @Transient
    private String secondName;

    @Access(AccessType.PROPERTY)
    public String getFullName() {
        return firstName + secondName;
    }

    public String getId() {

    }
}
```

`@Id`가 필드에 있으므로 기본은 필드 접근 방식을 사용하고 `getFullName()`만 프로퍼티 접근 방식을 사용한다.

위의 코드에서는 회원 엔티티를 저장하면 회원 테이블의 `FULLNAME` 컬럼에 `firstName + secondName`의 결과가 저장된다.
