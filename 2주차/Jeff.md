# Jeff
JPA 스터디

---
# 05. 연관관계 매핑

ORM 에서 가장 어려운 부분

→ 객체의 참조와 외래키를 매핑하는 것이 핵심

- 방향.Direction
    - 단방향, 양방향
    - 객체관계에서만 방향이 존재 테이블 관계는 항상 양방향
- 다중성. Multiplicity
    - N:1 , 1:N, 1:1, N:N
- 연관관계의 주인
    - 양방향으로 만드는 경우 주인을 정해야 함

## 5.1 단방향 연관관계

- 회원과 팀
- 회원은 하나의 팀에만 소속
- 회원과 팀은 다대일 관계

객체 연관관계

- Member 에서 Team 객체를 필드로 연관관계
- 단방향 관계

테이블 연관관계

- 회원 테이블은 TEAM_ID 외래키로 테이블과 연관관계
- 양방향 관계

### 객체 vs 테이블

가장 큰 차이 : 단방향 vs 양방향

객체에도 양방향을 하려면 참조를 추가해야함

→ 단, 이것은 사실은 각 각이 단방향인 것

### 5.1.1 순순 객체에서의 연관 관계

Team을 setting 해서 사용 하는 예제

→ 객체 참조를 통해 연관관계 탐색

*⇒ 객체 그래프 탐색*

### 5.1.2 테이블 연관관계

외래키 설정 후 INSERT 시 외래키 매핑

→ 외래키를 통해 연관관계 탐색

⇒ JOIN

### 5.1.3 JPA를 통한 객체 관계 매핑

- @ManyToOne
    - N:1 다대일
    - Optional 연관된 엔티티 여부
    - fetch → 패치 전략 사용 → 8장
    - cascade → 영속성 전이 기능 사용 → 8장
- @JoinColumn(name=“TEAM_ID”)
    - 생략 가능
- @OneToMany
    - targetEntity=.class 로 설정 가능(그냥 제네릭 쓰자)

@ManyToOne , @OneToOne

## 5.2. 연관관계 사용

### 5.2.1 저장

→ 연관된 모든 엔티티는 영속상태여야함

→ 연관 관계의 id 를 적절히 가져가서 저장함

### 5.2.2 조회

조회 방법 크게 2가지

1. 객체 그래프 탐색(객체 연관관계를 사용한 조회)

   → 객체 참조

2. 객체지향 쿼리 JPQL 사용

   → 조인


### 5.2.3 수정

→ 기본적인 수정과 똑같다. 매핑되는 id 를 변경감지를 통해 변경

### 5.2.4 연관관계 제거

→ null로 셋팅하면 됨

### 5.2.5 연관된 엔티티 삭제

→ 연관관계를 제거하고 삭제하고 팀 삭제 해야함

→ FK 문제

## 5.3 양방향 연관관계

팀에서 회원 가도록 연관관계 추가

→ 양방향 되려면 컬렉션 구조로 가져야함

ex) Collection, Set, Map 등

디비는 이미 양방향이므로 추가할 내용이 없음

객체에만 상대 관계의 객체에

OneToMany 혹은 ManyToOne 추가하면 됨

MappedBy → 어떤 변수에 매핑하는건지 설정

## 5.4 연관관계의 주인

mappedBy 속성 왜 필요한가 ?

→ 객체의 연관관계는 양방향이라는 것이 없음

서로 단방향 2개이기 때문에 이를 논리적으로 묶어줘야 하기 때문

연관관계는 2개인데 외래키는 하나이기 때문에

누가 관계의 주인인지를 정해야 함

### 5.4.1 양방향 매핑의 규칙 : 연관관계의 주인

주인인 쪽에서만 외래키 관리(등록,수정,삭제) 가능

주인이 아닌쪽은 읽기만 가능

주인이 아닌쪽에서 mappedBy로 주인쪽 매핑

연관관계의 주인

→ 외래키를 가지고있는 쪽으로 정해야함

→ DB에서 다대일 일대다 에서는 항상 다 쪽에서 외래키를 가짐

→ @ManyToOne 에는 MappedBy 속성이 없음

## 5.5 양방향 연관관계 저장

단방향 저장하는 것과 완전히 동일

주인이 아닌쪽에서 설정하는 연관관계에 대한 정보는 반영되지 않음

## 5.6 양방향 연관관계의 주의점

가장 많은 실수 → 주인에 값을 입력하지 않고 주인이 아닌쪽에 값을 입력

⇒ 반영되지 않는다.(읽기 전용임)

### 5.6.1 순수한 객체까지 고려한 양방향 연관관계

한쪽만 입력하게 되면 JPA 를 사용하지 않는 순수 객체 상태에서는 문제 발생

→ 양쪽 다 셋팅해주는 것이 안전함

### 5.6.2 연관관계 편의 메서드

setTeam 같은 메서드 내부에 상대 연관관계에 요소를 추가하는 방법 등을 활용 할 수 있음

### 5.6.3 연관관계 편의 메서드 주의사항

setTeam 내부 상대 연관관계 매핑 할 때

Team 을 변경하는 경우에는 기존 Team 쪽의 Member 연관관계를 삭제해야함

→ 양방향 연관관계 관리하기 빡세다

- 단뱡향 매핑만으로 테이블과 객체의 연관관계 매핑은 이미 완료
- 단방향을 양방향으로 만드는 경우 반대방향 객체 그래프 탐색 가능
- 양방향 연관관계 매핑 시 객체에서 양방향 모두 관리

단방향을 우선적으로 사용하고 → 꼭 필요한 경우 양방향을 검토해보자

- 연관관계의 주인
    - 용어로 인한 오해로 비즈니스상 더 중요한 부분을 선택하면 안됨
    - 외래키 등록된을 주인으로

양방향에서 toString() 오버라이딩 하거나 무한루프에 빠질 수 있음 주의

→ 특히 JSON 변환 시에 자주 발생함 (어노테이션 사용해서 무시하거나 하는 방법 사용)

# 06. 다양한 연관관계 매핑

연관관계 매핑 시 고려 사항

1. 다중성
    1. 1:1 관계인지 1:N 관계인지
2. 단방향, 양방향
    1. 참조하는 방향
3. 연관관계의 주인
    1. 양방향인 경우 주인 정하기

## 다중성

- N:1
    - @ManyToOne
- 1:N
    - @OneToMany
- 1:1
    - @OneToOne
- N:N
    - @ManyToMany → 실무에서 사용 x

## 단방향, 양방향

→ 객체는 방향 설정(양방향도 결국 단방향 2개)

## 6.1 다대일

- 1:N 의 항상 반대

다대일 단방향, 다대일 양방향

→ 5장에서 다 했던 내용

## 6.2 일대다

- 1:N 단방향

  → 특이하게도 1쪽에서 외래키 관리

  → JoinColumn 명시해야함

    - 하지않으면 JPA에서 조인관계를 JoinTable 을 통해 관리함 → 7장
    - 단점
        - 매핑한 객체가 관리하는 외래키가 다른 테이블에 존재함
        - INSERT SQL 한번으로 안되고
            - INSERT 1회(연관관계 주인)
            - INSERT 1회(연관관계 대상) → 이 때 FK 반영안됨(모르기때문)
            - UPDATE 1회(PK 보관하는 N 쪽 테이블)

      → UPDATE가 추가로 생김


    ⇒ 일대다 단방향 보다는 다대일 양방향을 사용하자
    
    - 엔티티를 매핑한 테이블이 아닌 다른 테이블의 외래키를 관리해야하는 구조적인 문제가 있음
    - 성능 및 관리의 어려움
    - 일반적인 경우 다대일 양방향이 관리하기 더 나음

### 6.2.2 일대다 양방향

→ 다대일 양방향과 같은 의미임

일대다(연관관계 주인 기준으로) 양방향이란건 없음

강제로 N 쪽에 insertable, updatable 을 false 로 설정해서 할 수도 있음

→ 어거지이므로 이럴거면 그냥 다대일 양방향 쓰자

## 6.3. 일대일

특징

- 일대일 관계의 반대도 일대일 관계
- 테이블 관계에서 일대다, 다대일과 다르게
    - 일대일은 어느곳에서든 외래키 가질 수 있음

→ 일반적으로 주 테이블에서 관리되는 것을 선호

### 6.3.1 주테이블에 외래키

단방향

양방향

→ 특별할 건 없음

### 6.3.1 대상 테이블에 외래키

→ 일대일에서 대상테이블의 외래키 연관관계 매핑은

JPA 에서 지원하지 않음

→ 양방향으로 매핑해야함

- 프록시를 쓰는 경우 지연로딩이 안됨

  → 프록시의 한계 때문에 발생


<aside>
❓ 왜 ? 일대일 인 경우에만 이것이 작동을 하지 않을까?

→ 존재 여부의 불확실성
다대일의 경우는 참조의 존재가 보장 됨 ?

일대일인 경우는 왜 확인을 해야하는거지 ?

</aside>

## 6.4 다대다

테이블 구조에서는

→ 다대다 관계는 중간 테이블을 추가해서 매핑해서 관리해야함

객체는 다대다 구조가 가능함

### 6.4.1. 다대다 : 단방향

회원, 상품 관계

@ManyToMany , @JoinTable 사용하여 테이블을 바로 매핑

→ 중간 테이블을 설정함

@JoinTable.joinColumns : 현재 방향인 회원과 매핑할 조인 테이블 정보 (MEMBER_ID)

@JoinTable.inverseJoinColumns : 반대 방향인 상품과 매핑할 조인 정보(PRODUCT_ID)

### 6.4.2. 다대다 : 양방향

똑같음. ManyToMany 하고 mappedBy로 master 정하자(아닌 쪽이 mappedBy)

### 6.4.3 다대다 : 매핑의 한계와 극복, 연결 엔티티 사용

→ 실무에서는 중간 매핑 테이블에 담아야 하는 정보가 더 많음

→ 추가된 컬럼이 있으면 @ManyToMany 사용 불가

⇒ 테이블 구조와 동일하게 중간 객체 둬야함

그리고 중간 객체를 각 객체에서 OneToMany로 매핑

(한쪽은 안만들 수도 있음)

중간객체에서는 ManyToOne 으로 각각의 객체 매핑

@IdClass 로 복합 기본키 매핑 위한 식별자 클래스 생성 후 설정

(MemberId 와 ProductId 를 가지는 MemberProductId 클래스)

→ JPA에서 복합키 사용하려면 이렇게 해야함

- 별도의 식별자 클래스로 생성
- Serializable 구현
- equals & hashCode 재정의
- 기본 생성자
- public
- @IdClass 혹은 @Embeddable 사용 가능

부모 테이블의 기본 키를 받아서

자신의 기본 키 + 외래키로 사용 하는 것

→ 식별 관게 identifying Relationship

### 6.4.4 다대다 : 새로운 기본 키 사용

→ 복합키를 기본키로 사용하지 않고 대리키 값을 기본 키로 가져가고

→ 나머지는 참조로 사용하는 것

간편하고 비즈니스에 의존하지 않음

→ 비즈니스 관련 Key 를 PK로 하지 않는 것과 비슷한 논리

⇒ 비 식별 관계로 설계하는 것이 좋음

- 받아온 식별자는 외래키로만 사용하고 새로운 식별자 추가

→ 관리하기 편하고 단순하고 간편함
  getId에 있으면 → 접근자


# 07. 고급매핑

- 상속 관계 매핑
- @MappedSuperclass
    - 등록, 수정일 같이 여러 엔티티에서 공통으로 사용하는 매핑 정보 상속
- 복합키, 식별 관계 매핑
    - DB 식별자가 하나 이상인 경우 매핑
- 조인 테이블
    - 연결 테이블 매핑하는 방법
- 엔티티 하나에 여러 테이블 매핑

## 7.1. 상속 관계 매핑

- DB에서는 상속 개념 X

  → 가장 유사한 Super-Tye & Sub-Type RelationShip


![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8da4657-58ed-4483-8d1b-3efce9010f0f/e5676aba-9edc-4d40-9e9d-d230244e2b52/Untitled.png)

슈퍼- 서브 타입 논리 모델을 테이블로 구현 하는 방법 3가지

1. 각각의 테이블로 변환

   ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8da4657-58ed-4483-8d1b-3efce9010f0f/5697647d-f94c-4cb2-b571-a795184115be/Untitled.png)

2. 통합 테이블로 변환

   ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8da4657-58ed-4483-8d1b-3efce9010f0f/658c4c5d-506a-49b8-96a6-0be551a88dc8/Untitled.png)

3. 서브타입 테이블로 변환

   ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8da4657-58ed-4483-8d1b-3efce9010f0f/43eb3d17-db54-46b3-9345-a0534e050461/Untitled.png)


### 7.1.1 조인 전략

> 엔티티 각각을 모두 테이블로 생성
→ 자식 테이블이 부모 테이블의 기본키를 받아서 기본키 + 외래키로 사용
>

# 부모 테이블에 타입을 구분 할 수 있는 구분자가 있어야함

- @Inheritance(strategy = InheritanceType.JOINED)
    - 상속 매핑 전략 설정
- @DiscriminatorColumn
    - 구분자 컬럼 지정
- @DiscriminatorValue
    - 구분자 값 지정
- @PrimaryKeyJoinColumn
    - 자식 테이블의 기본 키 컬럼 명 변경 하고 싶은 경우 매핑

### 장점

- 테이블이 정규화 됨
- 외래 키 참조 무결성 제약조건 활용 가능
- 저장공간 효율적으로 사용

### 단점

- 조회할 때 조인이 많이 사용되어 성능 저하
- 조회 쿼리 복잡
- INSERT 각 각 해야함

### 7.1.2 단일 테이블 전략

> 테이블 하나만 사용하고 구분 컬럼으로 구분
>

# 주의 할 점 : 자식 엔티티가 매핑한 컬럼 모두 null 허용해야함

→ 필요없는 컬럼이 많음

- @Inheritance(strategy = InheritanceType.SINGLE_TABLE)

### 장점

- 조인 필요 없으므로 조회 성능 빠름
- 조회 쿼리 단순함

### 단점

- 자식 엔티티 매핑 컬럼은 전부 null을 허용해야함
- 단일 테이블에 다 저장하면 오히려 너무 커져서 더 느릴 수도 이음

### 특징

- 구분 컬럼 꼭 사용 @DiscriminatorColumn 필수
    - 안쓰면 엔티티 이름을 사용함

### 7.1.3 구현 클래스마다 테이블 전략

> 자식 엔티티 마다 별도 테이블 생성
>

- @Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)

→ 추천하지 않는 전략

### 장점

- 서브 타입을 구분해서 처리할 때 효과적
- not null 제약조건 사용 가능

### 단점

- 여러 자식 테이블 조회할 때 UNION 사용해야 해서 느림
- 자식 테이블 통합해서 쿼리 하기 어려움

### 특징

- 구분 컬럼 사용하지 않음

→ DBA, ORM 전문가 전부 추천하지 않는 전략

## 7.2. @MappedSuperclass

> 부모 클래스는 테이블과 매핑하지 않고 상속받는 자식 클래스에 매핑 정보만 제공
>

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8da4657-58ed-4483-8d1b-3efce9010f0f/97ca5d23-e1f1-4d90-ad3a-d15f35e1faea/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8da4657-58ed-4483-8d1b-3efce9010f0f/8ab67044-f5e6-449e-93d9-e44ece5117d3/Untitled.png)

부모로부터 물려받은 매핑정보 재정의

→ @AttributeOverrides // @AttributeOverride

연관관계 재정의

→ @AssociationOverrides // @AssociationOverride

- 공통 정보를 사용하는 클래스들의 매핑 정보 모으는 역할만 함
    - 추상화 클래스 생성 권장

## 7.3 복합 키와 식별 관계 매핑

### 7.3.1 식별 관계 vs 비식별 관계

> 외래키가 기본키에 포함되는지 여부에 따라 구분
>

### 식별 관계

- 부모 테이블의 기본 키를 내려받아 자식 테이블의 기본 키 + 외래키로 사용

  ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8da4657-58ed-4483-8d1b-3efce9010f0f/0d9e794b-b9dd-449a-9bfd-aadb0e9561ab/Untitled.png)


### 비식별 관계

- 부모 테이블의 기본 키를 받아서 자식테이블의 외래키로만 사용

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8da4657-58ed-4483-8d1b-3efce9010f0f/47b92b98-18b4-4259-aa5c-5ea712fdc73f/Untitled.png)

1. 필수적 비식별 관계
    1. NULL 허용 X
    2. 연관관계 필수
2. 선택적 비식별 관계
    1. NULL 허용 O
    2. 연관관계 선택

→ 비식별 관계를 주로 사용하고 꼭 필요한 경우만 식별 관계 사용 권장

### 7.3.2 복합 키 : 비식별 관계 매핑

1. @IdClass : RDB에 가까운 방법
    1. 식별자 클래스의 속성명, 엔티티에서 사용하는 식별자의 속성명 동일
    2. Serializable 인터페이스 구현
    3. equals, hashCode 재정의
    4. ㅍ기본 생성자 필수
    5. 식별자 클래스는 public
2. @EmbeddedId : 객체지향에 가까운 방법
    1. @Embeddable 어노테이션 붙여야함
    2. Serializable 인터페이스 구현
    3. equals, hashCode 재정의
    4. 기본 생성자 필수
    5. 식별자 클래스는 public

   → @IdClass와의 차이점

    1. 식별자 클래스에 기본 키 직접 매핑함

        ```java
        @Embeddable
        public class ParentId implements Serializable{
        	
        	@Column(name = "PARENT_ID1")
        	private String id1;
        
        	@Column(name = "PARENT_ID2")
        	private String id2;	
        	
        	//equals , hashCode 재정의
        }
        ```

    2. 키로 존재하는게 아니라 클래스 자체로 인스턴스 변수 가지고 있음

**둘 다 equals(), hashCode() 재정의 반드시 할 것 주의**

어떤 것을 선택 ?

→ 취향 차이

- EmbeddedId 는 JPQL 좀 더 길어질 수 있음

### 7.3.3 복합 키 : 식별 관계 매핑

→ 비식별 관계와 비슷하지만

@IdClass 의 경우

```java
@Id 
@ManyToOne
@JoinColumn(name = "PARENT_ID")
public Parent parent;

@Id @Column(name = "CHILD_ID")
private String childId;
```

```java
public class ChildId implements Serializable {
	private String parent; //Child.parent 매핑
	private String childId; //Child.childId 매핑
}
```

이런식으로 부모 클래스 인스턴스 변수로 가지고 추가로 IdClass 로 매핑

@EmbeddedId 의 경우

→ @MapsId 사용해야함

```java
@EmbeddedId
private ChildId id;

@MapsId("parentId")
@ManyToOne
@JoinColumn(name = "PARENT_ID")
public Parent parent;

private String name;
```

```java
@Embeddable
public class ChildId implements Serializable {
	private String parentId; //@MapsId("parentId")로 매핑
	
	@Column(name = "CHILD_ID")
	private String id;
}
```

- @MapsId
    - 외래키와 매핑한 연관관계를 기본키에도 매핑하는 어노테이션

### 7.3.4 비식별 관계로 구현

→ 비식별 관계로 구현하면 훨씬 간결하고 유지보수 하기 쉬워짐

### 7.3.5 일대일 식별 관계

- 부모테이블의 기본 키 값만 사용



```java
@Entity
public class Board{
	@Id @GenratedValue
	@Column(name = "BOARD_ID")
	private Long id;
	
	private String title;
	
	@OneToOne(mappedBy = "board")
	private BoardDetail boardDetail;
}
```

```java
@Entity
public class BoardDetail{

  @Id
	private Long boardId;
	
	@MapsId
	@OneToOne
	@JoinColumn(name = "BOARD_ID")
	private Board board;
	
	private String content;
}
```

### 7.3.6 식별, 비식별 관계의 장단점

비 식별 관계 더 선호

- 식별 관계는 자식 테이블의 기본 키 컬럼이 점점 늘어남

  → 기본 키 인덱스 불필요하게 커질 수 있음

- 복합 기본 키 만들어야 하는 경우 많음
- 식별 관계에서 비즈니스와 관계 있는 키로 만드는 경우 많음

  → 비즈니스에 의존적임

- 테이블 구조가 유연하지 못함 (기본키 제약)

## 7.4. 조인 테이블

연관관계 설계 방법 두가지

1. 조인 컬럼 사용(외래 키)

   ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8da4657-58ed-4483-8d1b-3efce9010f0f/ab8b90f3-4a71-48ca-b6a2-59163a674300/Untitled.png)

    - 관계를 가끔 맺는 경우에는 NULL 값이 너무 많아짐
2. 조인 테이블 사용

   ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8da4657-58ed-4483-8d1b-3efce9010f0f/bbc8a864-f213-42d6-adb5-697586c46423/Untitled.png)

    - 테이블을 따로 생성해야하는 번거로움
    - 주로 다대다 관계를 풀어내기 위해 사용

## 7.5. 엔티티 하나에 여러 테이블 매핑

@SecondaryTable 을 사용하여 한 엔티티에 여러 테이블 매핑 가능

→ 써야 할 이유 찾기 힘들 듯 일대일 매핑 하는 것이 더 나은 방향