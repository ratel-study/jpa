## Ch. 8. 프록시와 연관관계 관리

- 프록시, 즉시로딩, 지연로딩
- 영속성 전이, 고아 객체

### 프록시

- 엔티티를 조회할 때 연관된 엔티티들이 항상 사용되지는 않는다.
- 지연 로딩 : 엔티티가 실제 사용될 때까지 DB 조회를 지연한다.
  - 지연 로딩을 하려면 실제 엔티티 객체 대신에 가짜 객체가 필요함 > 프록시 객체 
1. **프록시 기초**
   - em.find() 메서드를 사용하면 영속성 컨텍스트에 엔티티가 없을 때 DB를 조회한다.
   - em.getReference()를 사용하면 실제 사용하는 시점까지 DB조회를 미룬다.
     - 엔티티 객체를 생성하지 않고, DB접근을 위임한 프록시 객체 반환
   - 프록시 객체는 실제 클래스를 상속 받아 만들어진다.
   - 프록시 객체는 실제 객체에 대한 참조를 보관한다.
   - 프록시 객체의 메소드를 호출하면 프록시 객체는 실제 객체의 메소드를 호출한다.
   - 실제 사용될 때 DB를 조회해 실제 엔티티 객체를 생성한다. > 프록시 객체의 초기화
   - **프록시의 특징**
     - 프록시 객체는 처음 사용할 때 한 번만 초기화 된다.
     - 초기화 되어도 프록시 객체가 실제 엔티티로 바뀌는 것은 아니다.
     - 프록시 객체는 원본 엔티티를 상속받은 객체이므로 타입 체크 시에 주의해서 사용해야 한다.
     - 영속성 컨텍스트에 찾는 엔티티가 있다면 em.getReference()를 호출해도 프록시가 아닌 실제 엔티티를 반환한다.
     - 초기화는 영속성 컨텍스트의 도움을 받아야 가능하다. 준영속 상태의 프록시를 초기화하면 문제가 발생한다.
2. **프록시와 식별자**
   - 엔티티를 프록시로 조회할 때 PK 값을 파라미터로 전달하는데 프록시 객체는 이 식별자 값을 보관한다.
   - team.getId()로 식별자 값을 조회할 때는 프록시를 초기화 하지 않음.(@Access(AccessType.PROPERTY))인 경우만
   - 프록시는 연관관계 설정 시 유용하게 사용할 수 있다.
     ```java
     Member member = em.find(Member.class, "member1");
     Team team = em.getReference(Team.class, "team1"); // SQL 실행하지 않음
     member.setTeam();
     ```
     - DB를 조회하지 않고 연관관계 설정 가능.(AccessType.FIELD 도 프록시 초기화 하지 않음)
3. **프록시 확인**
   - PersistenceUnitUtil.isLoaded(Object entity) 메소드를 사용하면 프록시의 초기화 여부를 알 수 있다.
     - 초기화 되었거나 프록시 인스턴스가 아니라면 return true
   - 진짜 엔티티인지 프록시로 조회한 것인지
     - 클래스명으로 확인 가능. ..javassist.. 가 클래스명 뒤에 있다면 프록시
   - 하이버네이트의 initialize() 프록시 강제 초기화

### 즉시 로딩과 지연 로딩
1. **즉시 로딩** Eager Loading
   - @ManyToOne의 fetch 속성을 FetchType.EAGER로 지정
   - 대부분의 JPA 구현체는 즉시 로딩을 최적화하기 위해 조인쿼리로 한번에 조회
   > **NULL 제약조건과 JPA 조인 전략**   
   > 외래 키에 NOT NULL 제약조건을 설정하여 값이 있는 것을 보장하고, @JoinColumn 에 nullable = false 설정하면  
   > JPA가 내부 조인을 사용한다.   
   > @ManyToOne.optional = false로 설정해도 내부 조인을 사용한다.
   > 
2. **지연 로딩** Lazy Loading
   - @ManyToOne의 fetch 속성을 FetchType.LAZY로 지정
   - 조회 대상이 영속성 컨텍스트에 이미 있으면 프록시가 아닌 실제 엔티티를 사용함
### 지연 로딩 활용
1. **프록시와 컬렉션 래퍼**
   - 하이버네이트는 엔티티를 영속상태로 만들 때 엔티티에 있는 컬렉션을 하이버네이트 내장 컬렉션으로 변경 > 컬렉션 래퍼
     - 컬렉션 추적, 관리 목정
   - 엔티티를 지연로딩할 때의 프록시 객체처럼 컬렉션은 컬렉션 래퍼가 지연 로딩을 처리한다.
     - member.getOrders()를 호출해도 컬렉션은 초기화되지 않음. member.getOrders().get(0) 초기화 됨
2. **JPA 기본 페치 전략**
   - @ManyToOne, @OneToOne: 즉시 로딩
   - @OntToMany, @ManyToMany: 지연 로딩
   - 연관된 엔티티가 하나면 즉시 로딩, 컬렉션이면 지연 로딩이 기본 전략.
   - 모든 연관관계에 지연 로딩을 사용하는 것을 추천.
     - 실제 사용하는 상황을 보고 꼭 필요한 곳에만 즉시 로딩을 사용하도록 최적화
     - SQL을 직접 다루면 이런 유연한 최적화가 어렵다.
3. **컬렉션에 FetchType.EAGER 사용 시 주의점**
   - DB로 봤을 때는 일대다 조인. 만약 서로 다른 컬렉션을 2개 이상 조인한다면 실행 결과가 너무 많아 질 것.
   - JPA는 조회 결과를 메모리에서 필터링해 반환한다. 성능 저하 우려
   - 컬렉션 즉시 로딩은 항상 OUTER JOIN을 사용한다.

### 영속성 전이: CASCADE
- 특정 엔티티를 연속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶으면 영속성 전이 기능을 사용
1. **영속성 전이: 저장**
   - cascade = CascadeType.PERSIST : 부모를 영속화할 때 연관된 자식들도 함께 영속화
   - 연관관계 매핑과는 관련 없음.
2. **영속성 전이: 삭제**
   - CascadeType.REMOVE : 부모 엔티티만 삭제하면 연관된 자식 엔티티도 함께 삭제
   - DB 자식테이블에 외래 키 제약조건이 있다면 부모 엔티티만 삭제했을 때 외래키 무결성 예외 발생
3. **CASCADE 종류**
   - 여러 속성이 있고, 여러 속성을 함께 사용 가능
   - PERSIST, REMOVE 는 바로 전이가 발생하지 않고 플러시를 호출할 때 전이가 발생

### 고아 객체 Orphan
- 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제하는 기능 : 고아 객체 제거
- 부모 엔티티의 컬렉션에서 자식 엔티티의 참조만 제거하면 자식 엔티티가 자동으로 삭제
- orphanRemoval = true 속성 사용
- 플러시 때 적용 됨
- 참조하는 곳이 하나일 때만 사용해야 함. 삭제한 엔티티를 다른 곳에서도 참조한다면 문제 발생
- @OneToOne, @OneToMany에만 사용 가능
- 부모를 제거하면 자식도 제거됨(CascadeType.REMOVE 와 같음)

### 영속성 전이 + 고아 객체, 생명주기
- CascadeType.ALL, orphanRemoval = true 를 동시에 사용한다면?
  - 엔티티 스스로 생명주기를 관리하는 것이 아니라 부모 엔티티를 통해서 자식의 생명주기 관리하게 된다.
> 영속성 전이는 DDD의 Aggregate Root 개념을 구현할 때 사용하면 편리하다

---
## Ch. 9. 값 타입
- JPA의 데이터 타입
  - 엔티티 타입
    - 식별자를 통해 지속적으로 추적 가능
  - 값 타입
    - 기본값 타입
      - 기본 타입
      - 래퍼 클래스
      - String
    - 임베디드 타입
    - 컬렉션 값 타입


### 기본값 타입
- 엔티티 인스턴스를 제거하면 값타입은 제거된다. 생명주기를 엔티티에 의존함
- 값타입은 공유해서는 안된다.

### 임베디드 타입(복합 값 타입)
- 임베디드 타입 : 새로운 값 타입을 직접 정의해서 사용
```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    private String name;
    
    @Embedded Period workPeriod;
    @Embedded Address homeAddress;
}

@Embeddable
public class Period {
    @Temporal (TemporalType.DATE) Date startDate;
    @Temporal (TemporalType.DATE) Date endDate;
    
    public boolean isWork(Date date) {}
}
```
- @Embeddable: 값 타입 정의 시
- @Embedded: 값 타입 사용 시
1. **임베디드 타입과 테이블 매핑**
   - 객체와 테이블을 더 세밀하게 매핑 가능 (하나의 테이블에 여러 클래스)
   - 더 세밀한 객체지향 모델 설계 가능
2. **임베디드 타입과 연관관계**
   - 임베디드 타입은 값 타입을 포함하거나 엔티티를 참조할 수 있다.
3. **@AttributeOverride: 속성 재정의**
   - 회원에게 주소가 하나 더 필요한 경우. 컬럼명이 중복되는 문제. 매핑정보를 재정의 해야한다.
   ```java
   @Embedded
   @AttributeOverrides({
    @AttributeOverride(name="city", column=@Column(name="COMPANY_CITY")),
    @AttributeOverride(name="street", column=@Column(name="COMPANY_STREET"))
   })
   Address companyAddress;
   ```
   > @AttributeOverrides는 엔티티에 설정해야 함. 임베디드타입이 임베디드타입을 가지고 있어도 엔티티에 설정해야 함
   > 
4. **임베디드 타입과 null**
   - 임베디드 타입이 null이면 매핑한 컬럼은 모두 null

### 값 타입과 불변 객체
1. 값 타입 공유 참조
   - 임베디드 타입 등 값 타입을 공유했을 때 예상하지 못한 곳에서 문제가 생길 수 있다.
2. 값 타입 복사
   - 값 타입을 공유하는 것은 위험하므로 복사해서 사용해야 한다.
   - 객체의 공유 참조는 피할 수 없다. 하지만 객체가 수정되지 않도록할 수는 있다.
3. 불변 객체
   - 값 타입은 부작용이 없어야 한다.
   - 값 타입은 될 수 있으면 불변 객체로 설계해야 한다.
   - 생성자로만 값을 설정하고 수정자를 만들지 않으면 된다.

### 값 타입의 비교
- 값 타입은 인스턴스가 달라도 그 안의 값이 같으면 같다고 봐야한다.(동등성 비교. equals())
- 값 타입의 equals() 를 재정의할 때는 보통 모든 필드의 값을 비교하도록 구현

### 값 타입 컬렉션
- 값 타입을 하나 이상 저장하려면 컬렉션에 보관하고 @ElementCollection, @CollectionTable 어노테이션 사용
- 별도의 테이블을 추가하고 매핑함
```java
@ElementCollection
@CollectionTable(name = "FAVORITE_FOODS", joinColumns = @JoinColumn(name = "MEMBER_ID"))
private Set<String> favoriteFoods = new HashSet<>();
```
1. 값 타입 컬렉션 사용
   - 값 타입 컬렉션을 가진 엔티티만 영속화하면, 값 타입도 함께 저장된다.
   - 영속성 전이 + 고아 객체 제거 기능을 가지고 있는 것과 같다.
   - 값 타입 컬렉션 조회 시 페치 전략 선택 가능. 기본값: LAZY
2. 값 타입 컬렉션의 제약사항
   - ?? 값 타입 컬렉션에 보관된 값 타입들은 별도의 테이블에 보관되기 때문에
   - ?? 여기에 보관된 값 타입의 값이 변경되면 데이터베이스에 있는 원본 데이터를 찾기 어렵다.
   - JPA 구현체들은 값 타입 컬렉션이 변경되면 값 타입 컬렉션이 매핑된 테이블의 연관된 모든 데이터를 삭제하고 다시 저장
   - 실무에서는 일대다 관계를 고려해야 함
   - 모든 컬럼을 묶어서 기본 키를 구성해야 한다. (not null, 중복 안됨)
   - 제약들 해결하려면 
     - **값 타입 컬렉션 대신 새로운 엔티티 생성하여 일대다 관계로 설정하고 영속성 전이 + 고아 객체 제거 적용**
---
## 객체지향 쿼리 언어
### 객체지향 쿼리 소개
1. **JPQL 소개**
   - JPQL은 엔티티 객체를 조회하는 객체지향 쿼리.
   - SQL을 추상화하여 특정 데이터베이스에 의존하지 않는다.
   ```java
    String jpql = "select m from Member as m where m.username = 'kim";
    List<Member> resultList = em.createQuery(jpql, Member.class).getResultList();
   ```
   - Member: 엔티티 이름
   - m.username: 엔티티 객체의 필드명
2. **Criteria 소개**
   - JPQL을 생성하는 빌더 클래스
   - query.select(m).where()
   - 장점
     - 컴파일 시점에 오류 발견
     - IDE의 자동완성 지원
     - 동적 쿼리 작성이 편하다
   - 단점
     - 복잡하고 장황하다.
     - 불편하고 가독성이 좋지않음.
3. **QueryDSL 소개**
   - JPQL 빌더
   - 코드 기반. 단순. 사용 쉽다.
   ```java
   JPAQuery query = new JPAQuery(em);
   Qmember member = Qmember.member;
   
   List<Member> members =
    query.from(member)
    .where(member.username.eq("kim"))
    .list(member);    
   ```
   - QMember: 쿼리 전용 클래스
4. **네이티브 SQL 소개**
   - SQL 직접 사용
   - 특정 데이터베이스에 의존하는 기능을 사용해야 할 경우
   - SQL은 지원하지만 JPQL은 지원하지 않는 기능
   - 특정 DB에 의존하는 단점
5. **JDBC 직접, 마이바티스 등**
   - 영속성 컨텍스트를 적절할 때에 강제로 플러시해야 함.
   - JPA를 우회하여 DB에 접근하기 때문.
   - 스프링 프레임워크를 사용하면 JPA, Mybatis 통합 가능. AOP활용 하여 플러시 적절하게 사용 가능

### JPQL
1. **기본 문법과 쿼리 API**
   - SELECT 문
     - 대소문자 구분: 엔티티와 속성은 대소문자 구분, 키워드는 대소문자 구분하지 않음
     - 엔티티명: @Entity(name="이름")으로 지정. 기본값은 클래스명
     - 별칭 필수
   - TypeQuery, Query
     - 반환 타입이 명확하면 TypeQuery, 아니면 Query
     - Query 객체는 Object[] 또는 Object 반환
   - 결과 조회
     - query.getResultList(), query.getSingleResult() 실제 쿼리 실행.


2. **파라미터 바인딩**
   - 이름 기준 파라미터 Named parameters
     - 파라미터 앞에 : 사용. (where m.username = :username)
   - 위치 기준 파라미터 Positional parameters
     - ? 다음에 위치 값 (where m.username = ?1)
   - 이름 기준이 더 좋다
   > 직접 문자를 더해 JPQL을 만들면 위험하다. SQL 인젝션. JPA,DB의 성능에도 안좋다.
   > 
3. **프로젝션**
   - 프로젝션: SELECT 절에 조회할 대상을 지정하는 것
     - 프로젝션 대상: 엔티티, 임베디드 타입, 스칼라 타입(기본 데이터 타입)
     - 엔티티 프로젝션
       - 조회한 엔티티는 영속성 컨텍스트에서 관리됨
     - 임베디드 타입 프로젝션
       - 임베디드 타입은 조회의 시작점이 될 수 없다
         - SELECT a FROM Address a (잘못된 쿼리)
         - SELECT o.address FROM Order o 
       - 영속성 컨텍스트에서 관리되지 않음
     - 스칼라 타입 프로젝션
     - 여러 값 조회
       - TypeQuery 사용 불가
       - 엔티티 타입도 여러 값 조회 사용 가능
         - 영속성 컨텍스트에서 관리됨
     - NEW 명령어
       - "SELECT NEW jpabook.jpql.UserDTO(m.username, m.age) FROM Member m"
       - 반환받을 클래스를 지정 가능. 생성자에 조회 결과를 넘겨준다.
       - TypeQuery 사용 가능
       - 패키지명을 포함하여 클래스명을 입력
       - 순서, 타입이 일치하는 생성자 필요


4. **페이징 API**
   - setFirstResult(int startPosition): 조회 시작 위치(0부터 시작)
   - setMacResults(int maxResult): 조회할 데이터 수
   - 추상화하여 DB마다 적절하게 변환함


5. **집합과 정렬**
   - 집함 함수 사용 시 참고사항
     - NULL값은 무시. 통계에 잡히지 않음
     - 값이 없을 경우 집함함수의 결과는 NULL. (COUNT는 0)
     - DISTINCT를 집합 함수 안에 사용하여 중복 값 제거하여 집합을 구할 수 있음
     - DISTINCT를 COUNT 안에 사용할 때 임베디드 타입은 지원하지 않음


6. **JPQL 조인**
   - 내부 조인
     - INNER JOIN. INNER 생략 가능
     - "SELECT m FROM Member m INNER JOIN m.team t WHERE t.name = :teamname"
     - 연관필드 사용. (m.team): 다른 엔티티와 연관관계를 가지기 위해 사용하는 필드
     - "SELECT m, t FROM Member m JOIN m.team t" : 두 엔티티 조회
   - 외부 조인
     - "SELECT m FROM Member m LEFT [OUTER] JOIN m.team t"
   - 컬렉션 조인
     - 일대다, 다대다 등
     - "SELECT t, m FROM Team t LEFT JOIN t.members m"
   - 세타 조인
     - 내부 조인만 지원
     - "SELECT COUNT(m) FROM Member m, Team t WHERE m.username = t.name"
   - JOIN ON 절
     - 내부조인의 ON 절은 WHERE 절을 사용할 때와 결과가 같다. 주로 외부조인에 사용

    
7. **페치 조인**
   - JPQL에서 성능 최적화를 위해 제공하는 기능
   - JOIN FETCH 명령어로 사용
   - 엔티티 페치 조인
     - "SELECT m FROM Member m JOIN FETCH m.team"
     - SELECT M.*, T.* FROM MEMBER M INNER JOIN TEAM T ON M.TEAM_ID=T.ID
     - 페치 조인은 별칭을 사용할 수 없음 (하이버네이트는 허용)
     - m만 조회하여도 실행 SQL에서는 T.* 도 조회.
     - 지연 로딩으로 설정하여도 페치 조인을 사용하면 함께 조회되어 프록시가 아니라 실제 엔티티
     - 준영속 상태에서도 객체그래프 탐색 가능
   - 컬렉션 페치 조인
     - 일대다 조인으로 조회 결과가 늘어남
     - 팀-회원 일대다 조회시 팀에 속한 회원수만큼 같은 팀 엔티티를 여러개 반환
   - 페치 조인과 DISTINCT
     - JQPL의 DISTINCT는 SQL에 키워드를 추가하고 애플리케이션에서 한 번 더 중복을 제거한다.
     - 엔티티 중복 제거
   - 페치 조인과 일반 조인의 차이
     - JPQL은 결과를 반환할 때 연관관계까지 고려하지 않는다. SELECT 절에 지정한 엔티티만 조회한다.
   - 페치 조인의 특징과 한계
     - 글로벌 로딩 전략(fetch = FetchType.LAZY) 보다 페치 조인이 우선
     - 글로벌은 지연 로딩으로, 최적화가 필요한 경우 페치조인을 사용하는 것이 효과적이다.
     - 페치 조인의 한계
       - 페치 조인 대상에는 별칭을 줄 수 없다.
         - SELECT, WHERE 절, 서브 쿼리에 페치 조인 대상 사용 불가
         - 몇몇 구현체에서 사용가능 하지만 연관 데이터 수가 달라져 데이터 무결성 깨질 수 있다.
       - 둘 이상의 컬렉션을 페치할 수 없다.
       - 컬렉션을 페치 조인하면 페이징 API 사용 불가
         - 하이버네이트는 경고 로그와 함께 메모리에서 페이징처리함
       - 페치조인은 객체 그래프를 유지할 때 효과적
       - 여러 테이블 조회하여 엔티티가 아닌 전혀 다른 결과를 내야한다면 페치 조인보다는 필요한 필드들만 조회하여 DTO로 반환이 효과적

    
8. **경로 표현식**
   - .을 찍어 객체 그래프를 탐색하는 것
   - **경로 표현식의 용어 정리**
     - 상태 필드: 단순히 값을 저장하기 위한 필드
     - 연관 필드: 연관관계를 위한 필드, 임베디드 타입 포함
       - 단일 값 연관 필드: @ManyToOne, @OneToOne, 대상이 엔티티
       - 컬렉션 값 연관 필드: @OneToMany, @ManyToMany, 대상이 컬렉션
   - **경로 표현식과 특징**
     - 상태 필드 경로: 경로 탐색의 끝. 더이상 탐색 불가
     - 단일 값 연관 경로: 묵시적으로 내부 조인. 계속 탐색 가능
     - 컬렉션 값 연관 경로: 묵시적으로 내부 조인. 더이상 탐색 불가. FROM절에서 조인을 통해 별칭을 얻으면 별칭으로 탐색 가능
       - size기능 사용하면 컬렉션의 크기를 구할 수 있다
   - **경로 탐색을 사용한 묵시적 조인 시 주의사항**
     - 내부 조인
     - 컬렉션은 경로 탐색의 끝. 더 탐색하려면 명시적 조인
     - SELECT, WHERE 절에서 경로 탐색을 사용하지만 FROM 절에 영향을 준다 (쿼리 성능)


9. **서브 쿼리**
   - WHERE, HAVING 절에서만 사용 가능
   - 서브 쿼리 함수
     - [NOT] EXISTS
       - 서브쿼리에 결과가 존재하면 참
     - {ALL | ANY | SOME}
       - 비교 연산자와 함께 사용
       - ALL: 조건을 모두 만족하면 참
       - ANY, SOME: 조건을 하나라도 만족하면 참
     - [NOT] IN
       - 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참


10. **조건식**
    - 타입 표현
      - 문자: ''
      - 숫자: L, D, F
        - 10L, 10D, 10F
      - 날짜
        - {d'yyyy-mm-dd'}, {t'hh-mm-ss'}, {ts'yyyy-mm-dd hh:mm:ss.f'}
      - Boolean: TRUE, FALSE
      - Enum: 패키지명 포함한 전체 이름 사용
      - 엔티티 타입: TYPE(m) = Member
        - 주로 상속과 관련해 사용
    - 연산자 우선 순위
      1. 경로 탐색 연산
      2. 수학 연한
      3. 비교 연산
      4. 논리 연산: NOT, AND, OR
    - 컬렉션 식
      - 컬렉션은 컬렉션 식만 사용 가능 (IS NULL 등 사용 불가)
      - 빈 컬렉션 비교 식: {컬렉션 값 연관 경로} IS EMPTY
      - 컬렉션의 멤버 식: {엔티티나 값} MEMBER [OF] {컬렉션 값 연관 경로}


11. **다형성 쿼리**
    - 부모 엔티티를 조회하면 자식 엔티티도 조회됨
    - TYPE
      - 조회 대상을 특정 자식 타입으로 한정할 때 주로 사용
    - TREAT
      - 부모 타입을 특정 자식 타입으로 다룰 때 사용 (타입캐스팅)
      - FROM, WHERE 절에서 사용
      - 하이버네이트는 SELECT도 가능


12. **사용자 정의 함수 호출**
    - FUNCTION('function_name' {, function_arg}*)
    - 하이버네이트 사용시
      - 방언 클래스 상속하여 사용할 데이터베이스 함수 미리 등록
      - hibernate.dialect에 해당 방언 등록


13. **기타 정리**
    - enum은 = 비교 연산만 지원
    - 임베디드 타입은 비교를 지원하지 않는다.
    - EMPTY STRING: ''
    - NULL 정의
      - 조건을 만족하는 데이터가 하나도 없으면 NULL
      - NULL은 알 수 없는 값. NULL과 수학적 계산 결과는 모두 NULL
      - NULL==NULL은 알 수 없는 값.
      - NULL IS NULL은 참


14. **엔티티 직접 사용**
    - 기본 키 값
      - JPQL에서 엔티티 객체를 직접사용 하면 SQL에서는 기본 키 값을 사용
      - COUNT(m) > COUNT(m.id)
    - 외래 키 값
      - WHERE m.team = :team
      - WHERE m.team_id = ?


15. **Named 쿼리: 정적 쿼리**
    - 동적 쿼리: em.createQuery() 문자로 완성해서 직접 넘기는 것
    - 정적 쿼리: 미리 정의한 쿼리에 이름을 부여해서 필요할 때 사용
    - Named 쿼리는 애플리케이션 로딩 시점에 JPQL 문법을 체그하고 미리 파싱해둔다.
      - 오류를 빨리 확인할 수 있고, 성능에 유리
    - @NamedQuery 사용하거나 XML 문서에 작성
    - @NamedQuery
      ```java
      @Entity
      @NamedQuery(name="Member.findByUsername", query="select...")
      public class Member {}
      ```
    - XML에 정의
      - XML이 더 편리함
      - &, <, > 가 예약 문자이기 때문에 주의 필요
    - XML, 어노테이션에 같은 설정이 있으면 XML 우선
