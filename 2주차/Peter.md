
## Ch. 5. 연관관계 매핑 기초

- 객체는 참조를 사용해서 관계를 맺고, 테이블은 외래 키를 사용해서 관계를 맺는다.
- 객체의 참조와 테이블의 외래 키를 매핑하는 것이 목표

### 1. **단방향 연관관계**
   - 다대일(N:1) 관계
   - 회원:팀 N:1 예시
     - 객체
       - Member.team 멤버변수로 팀 객체와 연관관계
       - 회원은 팀을 알 수 있지만 팀은 회원을 알 수 없다 (단방향)
     - 테이블
       - team_id 외래 키로 팀 테이블과 연관관계
       - 외래 키를 통해 조인 가능 (양방향)
     > 객체, 테이블 연관관계의 차이     
     > - 참조를 통한 관계는 단방향
     > - 양방향으로 하려면 반대 쪽에도 필드 추가해야 함
     > - 사실은 양방향이 아니라 단방향x2
   - 객체 연관관계
     - 객체 그래프 탐색을 통해 연관관계 탐색
   - 테이블 연관관계
     - 조인(외래키)를 사용한 연관관계 탐색
   - JPA를 사용한 관계 매핑
     ```java
     @Entity
     public class Member {
     
        @Id
        @Column(name = "MEMBER_ID")
        private String id;
        private String userName;
     
        // 연관관계 매핑
        @ManyToOne
        @JoinColumn(name="TEAM_ID")
        private Team team;
     
        public void setTeam(Team team) {
            this.team = team;
        }
     }
     ```
     - 멤버변수 team 과 외래키인 TEAM_ID를 매핑하는 것이 연관관계 매핑
   - @JoinColumn
     - name : 매핑할 외래키 이름. 기본 값은 {필드명} + _ + {참조하는 테이블의 기본키 컬럼명}
     - referencedColumnName : 외래키가 참조하는 대상 테이블의 컬럼명. 기본 값 {참조하는 테이블의 기본키 컬럼명}
     - foreignKey(DDL) : 제약 조건 설정. 테이블 생성 때만 사용 됨
   - @ManyToOne
     - optional : false로 설정하면 연관된 엔티티가 항상 있어야 함. 기본값 true
     - fetch : 글로벌 페치 전략. 기본 값 @ManyToOne=FetchType.EAGER
     - cascade : 영속성 전이 기능을 사용한다


### 2. **연관관계 사용**
   - 저장
     - 객체 간의 연관관계를 설정(참조) 하고, member.setTeam(team1)
     - 저장한다. em.persist()
     - 연관된 모든 엔티티는 영속 상태여야 한다.
   - 조회
     - 객체 그래프 탐색
       - member.getTeam()
     - 객체지향 쿼리 사용(JPQL)
     ```java
     private static void queryLogicJoin(EntityManager em) {
        
        String jpql = "select m from Member m join m.team t where " +
            "t.name=:teamName";
     
        List<Member> resultList = em.createQuery(jpql, Member.class)
                                    .setParameter("teamName", "팀1")
                                    .getResultList();
     
        System.out.println(resultList);
     }
     ```
     - JPQL은 객체를 대상으로 하고 SQL 보다 간결함.
   - 수정
     - 객체의 연관관계를 수정
     - 변경 감지 기능 작동
   - 연관관계 제거
     - 연관관계를 null로 설정
   - 연관된 엔티티 삭제
     - 외래키 제약조건이 있는 경우, 기존의 연관관계를 제거하고 엔티티를 삭제해야 한다. em.remove()

### 3. **양방향 연관관계**
- 양방향 연관관계 매핑
   ```java
   @OneToMany(mappedBy = "team")
   private List<Member> members = new ArrayList<Member>();
    ```
- 일대다 관계이기 때문에 컬렉션을 추가, @OneToMany 추가


### 4. **연관관계의 주인**
   - 객체의 관계에서 양방향은 없음. 단방향이 두 개인 것
   - 객체의 참조는 둘인데, 외래키는 하나이므로 외래키를 관리하는 주체가 필요하다
   - **연관관계의 주인**
     - 연관관계의 주인만이 데이터베이스 연관관계와 매핑되고, 외래 키를 관리할 수 있다.
     - 주인이 아닌 연관관계는 읽기만 할 수 있다. 외래키 변경 불가
     - mappedBy 속성으로 주인 지정
       - 주인은 mappedBy 속성을 사용하지 않는다
       - 주인이 아니면 mappedBy 속성을 사용해서 속성의 값으로 연관관계의 주인을 지정
     - 연관관계의 주인은 외래키가 있는 곳으로
       - 회원 테이블에 TEAM_ID 외래키가 있다면 회원 엔티티의 team 을 주인으로
       - 팀 엔티티에는 mappedBy="team" 속성 사용하여 주인 지정
   - @ManyToOne에는 mappedBy 속성이 없다. 다대일 관계에서 '다' 쪽이 외래키를 가지기 때문


### 5. **양방향 연관관계 저장**
   - 주인 연관관계를 사용해 연관 관계 설정.
   - 주인이 아닌 연관관계는 데이터베이스에 저장할 때 무시됨.


### 6. **양방향 연관관계의 주의점**
   - 연관관계의 주인에는 값을 입력하지 않고, 주인이 아닌 곳에만 값을 입력하는 실수 주의
   - 객체 관점에서 양쪽 방향에 모두 값을 입력해주는 것이 가장 안전하다.
     - JPA를 사용하지 않는 순수한 객체 상태에서 심각한 문제가 발생할 수 있음
     - 데이터베이스뿐만아니라 객체도 함께 고려해야 한다.
   - 연관관계 편의 메소드
     ```java
     public void setTeam(Team team) {
        this.team = team;
        team.getMembers().add(this);
     }
     ```
   - 위의 편의 메소드를 사용하면 연관관계 변경시 버그 발생
     ```java
     member1.setTeam(team1);
     member1.setTeam(team2);
     Member findMember = team1.getMember();
     // member1이 여전히 조회됨
     ```
     - 연관관계 변경 시 기존 관계 제거하는 코드 필요
     ```java
     public void setTeam(Team team) {
     
        if (this.team != null) {
            this.team.getMembers.remove(this);
        }
        this.team = team;
        team.getMembers().add(this);
     }
     ```
   - 단방향 연관관계 2개를 양방향으로 동작하게 하려면 로직을 견고하게 작성해야 함
   - 단방향과 차이점은 반대방향으로 객체 그래프 탐색이 가능하다는 것
     - 단방향 매핑을 사용하고 반대 방향 탐색(JPQL 등)이 필요한 경우 추가해도 됨.
   > 양방향 매핑 시에는 무한 루프 조심
   > Member.toString()에서 getTeam() 호출, Team.toString()에서 getMembers 호출하는 경우
   > 엔티티를 JSON으로 변환할 때 자주 발생함

---
## Ch. 6. 다양한 연관관계 매핑
### 1. **다대일**
   1. 다대일 단방향 [N:1]
   2. 다대일 양방향 [N:1, 1:N]


### 2. **일대다**
   1. 일대다 단방향 [1:N]
      ```java
      @Entity
      public class Team {
      
           @Id @GeneratedValue
           @Column(name = "TEAM_ID")
           private Long id;
           private String name;
      
           @OneToMany
           @JoinColumn(name = "TEAM_ID")
           private List<Member> members = new ArrayList<>();
      }
      
      @Entity
      public class Member {
      
           @Id @GeneratedValue
           @Column(name = "MEMBER_ID")
           private Long id;
           private String name;
      }
      ```
      - Team 엔티티의 members로 MEMBER 테이블의 TEAM_ID 외래키 관리함
      - 일대다 단방향 관계를 매핑할 때는 @JoinColumn을 명시해야 한다.
        - 명시하지 않을 경우 JPA는 연결 테이블로 연관관계를 관리하는 전략을 기본으로 사용한다.
      - 일대다 단방향 매핑의 단점
        - 객체가 관리하는 외래 키가 다른 테이블에 있음
        - 저장과 연관관계 처리를 위해 SQL을 추가로 실행해야 함. INSERT, UPDATE
        - 일대다 단방향 보다는 다대일 양방향이 낫다
   2. 일대다 양방향 [N:1, 1:N]
      - 일대다 단방향 매핑 반대편에 같은 외래 키를 사용하는 다대일 단방향 매핑을 읽기 전용으로 추가하면 된다.
        ```java
        // 읽기 전용 다대일 단방향 매핑
        @ManyToOne
        @JoinColumn(name = "TEAM_ID", insertable = false, updatable = false)
        private Team team;
        ```
      - 일대다 단방향 매핑의 단점이 그대로임. 다대일 양방향 매핑을 사용해야 한다.


### 3. **일대일 [1:1]**
   - 일대일 관계는 반대도 일대일 관계
   - 일대일 관계는 주 테이블, 대상 테이블 어느 쪽이나 외래 키를 가질 수 있음.
   - 누가 외래키를 가질지 선택해야 함
     - 주 테이블에 외래 키
       - 주 객체가 대상 객체를 참조하 듯 주 테이블에 외래 키 두고 대상 테이블 참조
       - 외래 키를 객체 참조와 비슷하게 사용할 수 있음
       - 주 테이블만 확인해도 대상 테이블과 연관관계가 있는지 알 수 있음
     - 대상 테이블에 외래 키
       - 테이블 관계를 1:1 에서 1:N으로 변경할 때 테이블 구조 유지 가능
   - **주 테이블에 외래 키**
     - 단방향
       ```java
       @Entity
       public class Member {
            @Id @GeneratedValue
            @Column(name = "MEMBER_ID")
            private Long id;
            private String userName;
       
            @OneToOne
            @JoinColumn(name = "LOCKER_ID")
            private Locker locker;
       }
       
       @Entity
       public class Locker {
            @Id @GeneratedValue
            @Column(name = "LOCKER_ID")
            private Long id;
            private String name;
       }
       ```
     - 양방향
       ```java
       @Entity
       public class Member {
            @Id @GeneratedValue
            @Column(name = "MEMBER_ID")
            private Long id;
            private String userName;
       
            @OneToOne
            @JoinColumn(name = "LOCKER_ID")
            private Locker locker;
       }
       
       @Entity
       public class Locker {
            @Id @GeneratedValue
            @Column(name = "LOCKER_ID")
            private Long id;
            private String name;
       
            @OneToOne(mappedBy = "locker") // 연관관계의 주인 지정
            private Member member;
       }
       ```
   - **대상 테이블에 외래 키**
     - 단방향
       - 일대일 관계에서 대상 테이블에 외래 키가 있는 단방향 관계는 JPA에서 지원하지 않음
     - 양방향
       ```java
       @Entity
       public class Member {
            @Id @GeneratedValue
            @Column(name = "MEMBER_ID")
            private Long id;
            private String userName;
       
            @OneToOne(mappedBy = "member") // 연관관계 주인 지정
            private Locker locker;
       }
       
       @Entity
       public class Locker {
            @Id @GeneratedValue
            @Column(name = "LOCKER_ID")
            private Long id;
            private String name;
       
            @OneToOne
            @JoinColumn(name = "MEMBER_ID")
            private Member member;
       }
       ```
       > 프록시를 사용할 때 외래 키를 직접 관리하지 않는 일대일 관계는 지연 로딩으로 설정해도 즉시 로딩된다.  
     Locker.member는 지연 로딩 가능하지만, Member.locker는 지연 로딩 불가  
     프록시 대신 bytecode instrumentation을 사용하면 해결할 수 있음
       >


### 4. **다대다 [N:N]**
   - 관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없음
     - 연결 테이블 사용. 다대다 -> 일대다, 다대일로
   - 객체 2개로 다대다 관계는 가능
     - @ManyToMany
   1. **다대다: 단방향**
      ```java
      @Entity
      public class Member {
        @Id @Column(name = "MEMBER_ID")
        private String id;
        private String userName;
        
        @ManyToMany
        @JoinTable(name = "MEMBER_PRODUCT",
                   joinColumns = @JoinColumn(name = "MEMBER_ID"),
                   inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID"))
        private List<Product> products = new ArrayList<>(); 
      }
      
      @Entity
      public class Product {
        @Id @Column(name = "PRODUCT_ID")
        private String id;
        private String name;
      }
      ```
      - 데이터베이스에는 연결테이블을 포함하여 테이블 3개이지만, 엔티티 2개로 매핑
      - @JoinTable
        - name : 연결 테이블 지정
        - joinColumns : 현재 방향 매핑할 조인 컬럼 정보
        - inverseJoinColumns : 반대 방향 매핑할 조인 컬럼 정보
      ```java
      public void find() {
        Member member = em.find(Member.class, "member1");
        List<Product> products = member.getProducts(); // 객체 그래프 탐색
        System.out.println();
      }
      ```
      ```sql
      SELECT * FROM MEMBER_PRODUCT MP
      INNER JOIN PRODUCT P ON MP.PRODUCT_ID=P.PRODUCT_ID
      WHERE MP.MEMBER_ID=?
      ```
   2. **다대다: 양방향**
      ```java
      @ManyToMany(mappedBy = "products") // 역방향, 연관관계 주인 지정
      private List<Member> members; 
      ```
      - 연관관계 설정 시 양쪽 모두 설정. 편의 메소드 활용
   3. **다대다: 매핑의 한계와 극복, 연결 엔티티 사용**
      - @ManyToMany를 실무에서 사용하기에는 한계가 있다.
      - 실제로는 연결테이블에 두 테이블의 키만 담는 것이 아니라 추가 정보를 저장함
      - 추가한 컬럼들은 @ManyToMany로 매핑할 수 없다.
      - **연결 엔티티를 만들고 일대다, 다대일 관계로 설정** 
        ```java
        @Entity
        public class Member {
            @Id @Column(name = "MEMBER_ID")
            private String id;
        
            @OneToMany(mappedBy = "member") // 연결 엔티티가 연관관계의 주인
            private List<MemberProduct> memberProducts;
        }
  
        @Entity
        public class Product {
            @Id @Column(name = "PRODUCT_ID")
            private String id;
            private String name;
        
            // MemberProduct 에 대한 객체 그래프 탐색이 필요하지 않음
        }
        
        @Entity
        @IdClass(MemberProductId.class) // 식별자 클래스
        public class MemberProduct {
            @Id
            @ManyToOne
            @JoinColumn(name = "MEMBER_ID")
            private Member member;
        
            @Id
            @ManyToOne
            @JoinColumn(name = "PRODUCT_ID")
            private Product product;
        
            private int oderAmount;
        }
        
        public class MemberProductId implements Serializable {
            private String member;
            private String product;
        
            @Override
            public boolean equals(Object obj) {}
            @Override
            public int hashCode() {}
        
        }
        ```
        - @IdClass를 사용해 복합 기본 키 매핑
          > JPA의 복합 기본 키
          >  - 복합 키는 별도의 식별자 클래스로 만들어야 한다.
          >  - Serializable을 구현해야 한다.
          >  - equals, hashCode 를 재정의해야 한다.
          >  - 기본 생성자가 있어야 한다.
          >  - 식별자 클래스는 public이어야 한다.
          >  - @IdClass 또는 @EmbeddedId 를 사용해서 지정
        - 식별관계 Identifying Relationship: 부모 테이블의 기본키를 받아서 자신의 기본키+외래키로 사용하는 것
        - 연결 엔티티를 활용한 다대다 관계 저장
          1. 다대다에 해당하는 두 개의 엔티티 저장
          2. 연결 엔티티에 두 개의 연관관계 설정
          3. 연결 엔티티 저장
        - 조회
          - MemberProduct memberProduct = em.find(MemberProduct.class, memberProductId);
   4. **다대다: 새로운 기본 키 사용**
      - 데이터베이스에서 자동으로 생성해주는 대리 키를 Long 으로 사용하는 방법
      - 복합 키를 만들지 않아도 된다.
      ```java
      @Entity
      public class Order {
        @Id @GeneratedValue
        @Column(name = "ORDER_ID")
        private Long id;
      
        @ManyToOne
        @JoinColumn(name = "MEMBER_ID")
        private Member member;

        @ManyToOne
        @JoinColumn(name = "PRODUCT_ID")
        private Product product;
        
        private int oderAmount;
      }
      ```
   5. **다대다 연관관계 정리**  
      다대다 관계를 풀어내기 위해 연결 테이블을 만들 때 식별자 구성 방법
      - 식별 관계 : 받아온 식별자를 기본키+외래키로 사용
      - 비식별 관계 : 받아온 식별자는 외래키로 사용, 새로운 식별자 추가   
      후자가 ORM 매핑시 좋음.

---
## Ch. 7. 고급 매핑

### 1. **상속 관계 매핑**  
   - 관계형 데이터베이스에는 상속 개념이 없다.  
   - 슈퍼타입 서브타입 관계가 상속과 유사함  
   - 상속관계 매핑 : 객체의 상속과 관계형 데이터베이스의 슈퍼타입 서브타입 관계를 매핑하는 것
   
   1. **조인 전략**
      - 엔티티 각각을 테이블로 만들고 자식 테이블이 부모의 기본키를 받아서 기본키+외래키로 사용
      - 조회시 조인 사용
      - 객체는 타입으로 구분하지만 테이블은 타입 개념이 없기 때문에 타입을 구분하는 컬럼 추가 필요
      ```java
      @Entity
      @Inheritance(strategy = InheritanceType.JOINED)
      @DiscriminatiorColumn(name = "DTYPE")
      public abstract class Item {
        @Id @GeneratedValue
        @Column(name = "ITEM_ID")
        private Long id;
        private String name;
        private int price;
      }
      
      @Entity
      @DiscriminationValue("A")
      public class Album extends Item {
        private String artist;
      }
      
      @Entity
      @DiscriminationValue("M")
      public class Movie extends Item {
        private String director;
        private String actor;
      }
      ```
      - @Inheritance(strategy = InheritanceType.JOINED)
        - 부모 클래스에 사용
      - @DiscriminatiorColumn(name = "DTYPE")
        - 부모 클래스에 구분 컬럼 지정. 기본값: DTYPE
      - @DiscriminationValue("M")
        - 엔티티를 저장할 때 구분 컬럼에 입력할 값 지정
      - 자식 테이블은 부모 테이블의 ID 컬럼명을 그대로 사용.
        - 변경하고 싶으면 @PrimaryKeyJoinColumn 사용
      - 장점
        - 테이블이 정규화된다.
        - 외래 키 참조 무결성 제약조건 활용 가능
        - 저장공간을 효율적으로 사용
      - 단점
        - 조회할 때 조인이 많이 사용되어 성능 저하될 수도
        - 조회 쿼리 복잡
        - 데이터 등록시 INSERT 두 번 실행
      - 특징
        - JPA 표준 명세는 구분 컬럼을 사용하도록 하지만 하이버네이트 등 몇몇 구현체는 구분 컬럼 없이도 동작함
   2. **단일 테이블 전략**
      - 하나의 테이블 사용. 구분 컬럼으로 구분
      - 자식 엔티티가 매핑한 컬럼은 모두 null 허용해야한다.
      ```java
      @Entity
      @Inheritance(strategy = InheritanceType.SINGLE_TABLE)
      @DiscriminatiorColumn(name = "DTYPE")
      public abstract class Item {
        @Id @GeneratedValue
        @Column(name = "ITEM_ID")
        private Long id;
        private String name;
        private int price;
      }
      
      @Entity
      @DiscriminationValue("A")
      public class Album extends Item {
        private String artist;
      }
      
      @Entity
      @DiscriminationValue("M")
      public class Movie extends Item {
        private String director;
        private String actor;
      }
      ```
      - 장점
        - 조인이 필요 없어 조회 성능이 빠르다
        - 조회 쿼리가 단순하다.
      - 단점
        - 자식 엔티티가 매핑한 컬럼은 모두 null 허용
        - 테이블이 너무 커질 수 있고, 조회 성능이 오히려 느려질 수 있다.
      - 특징
        - 구분 컬럼을 꼭 사용해야 한다.
        - @DiscriminatorValue를 지정하지 않으면 기본값으로 엔티티 이름을 사용
   3. 구현 클래스마다 테이블 전략
      - 자식 엔티티 마다 테이블을 만든다.
      - 자식 테이블 각각에 필요한 컬럼이 모두 있다.
      ```java
      @Entity
      @Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
      @DiscriminatiorColumn(name = "DTYPE")
      public abstract class Item {
        @Id @GeneratedValue
        @Column(name = "ITEM_ID")
        private Long id;
        private String name;
        private int price;
      }
      
      @Entity
      public class Album extends Item {}
      
      @Entity
      public class Movie extends Item {}
      ```
      - 장점
        - 서브 타입을 구분해서 처리할 때 효과적
        - not null 제약조건 사용 가능
      - 단점
        - 여러 자식 테이블을 함께 조회할 때 성능이 느리다. UNION
        - 자식테이블을 통합해서 쿼리하기 어렵다.
      - 특징
        - 구분 컬럼을 사용하지 않는다.
      - 추천하지 않는 전략


### 2. @MappedSuperclass
- 부모 클래스는 테이블과 매핑하지 않고, 자식 클래스에게 매핑 정보만 제공하고 싶을 때 사용
- 실제 테이블과는 매핑되지 않고, 매핑 정보를 상속
```java
// 서로 관계가 없는 테이블, 엔티티에서 공통 속성을 모아서 부모 클래스로 추출
@MappedSuperclass
public abstract class BaseEntity {
    @Id @GeneratedValue
    private Long id;
    private String name;
}

@Entity
public class Member extends BaseEntity {
    private String email;
}

@Entity
public class Seller extends BaseEntity {
    private String shopName;
}
```
- 부모로 부터 물려받은 매핑 정보를 재정의 하려면
  - @AttributeOverrides, @AttributeOverride
- 연관관계를 재정의 하려면
  - @AssociationOverrides, @AssociationOverride
```java
@Entity
@AttributeOverride(name = "id", column = @Column(name = "MEMBER_ID"))
public class Member extends BaseEntity {}
```
- 특징
  - 테이블과 매핑되지 않고 자식 클래스에 엔티티의 매핑 정보를 상속하기 위해 사용
  - @MappedSuperclass로 지정한 클래스는 엔티티가 아니므로 em.find(), JPQL 등에서 사용 불가
  - 직접 생성해서 사용할 일은 거의 없으므로 추상 클래스로 만드는 것을 권장

### 3. 복합 키와 식별 관계 매핑

1. **식별 관계 vs 비식별 관계**
   - 식별 관계는 부모의 기본키를 기본키+외래키로 사용
   - 비식별 관계는 외래키로만 사용
     - 필수적 비식별 관계 Mandatory: 외래키 NOT NULL
     - 선택적 비식별 관계 Optional: 외래키 NULL
2. **복합키:비식별 관계 매핑**
    - JPA에서 복합키를 사용하는 법
      - **@IdClass**
        ```java
        @Entity
        @IdClass(ParentId.class)
        public class Parent {
            @Id
            @Column(name = "PARENT_ID1")
            private String id1;
        
            @Id
            @Column(name = "PARENT_ID2")
            private String id2;
        }
        
        public class PrentId implements Serializable {
            private String id1;
            private String id2;
        
            public ParentId() {}
            public ParentId(String id1, String id2) {
                this.id1 = id1;
                this.id2 = id2;
            }
        
            @Override
            public boolean equals(Object obj) {}
        
            @Override
            public int hashCode() {}
        }
        ```
        @IdClass 사용시 식별자 클래스 조건
        - 식별자 클래스의 속성명과 엔티티에서 사용하는 식별자의 속성명이 같아야 함
        - Serializable을 구현해야 한다.
        - equals, hashCode 를 재정의해야 한다.
        - 기본 생성자가 있어야 한다.
        - 식별자 클래스는 public이어야 한다.
        ```java
        // 저장
        Parent parent = new Parent();
        parent.setId1("id1");
        parent.setId2("id2");
        em.persist(parent);
        
        // 조회
        ParentId parentId = new ParentId("id1", "id2");
        Parent parent = em.find(Parent.class, parentId);
        ```
        - persist 호출 시 식별자 클래스는 내부에서 생성하여 처리됨
        - 조회 시 식별자 클래스 사용
        ```java
        // 자식 클래스 작성
        @Entity
        public class Child {
            @Id
            private String id;
        
            @ManyToOne
            @JoinColumns({
                @JoinColumn(name = "PARENT_ID1", referencedColumnName = "PARENT_ID1"),
                @JoinColumn(name = "PARENT_ID2", referencedColumnName = "PARENT_ID2")
            })
            private Parent parent;
        }
        ```
      - **@EmbeddedId**
        - 좀 더 객체지향적인 방법
        ```java
        @Entity
        public class Parent {
            
            @EmbeddedId
            private ParentId id;
        }
        
        @Embeddable
        public class PrentId implements Serializable {
            @Column(name = "PARENT_ID1")
            private String id1;
            @Column(name = "PARENT_ID2")
            private String id2;
        
            @Override
            public boolean equals(Object obj) {}
        
            @Override
            public int hashCode() {}
        }
        ```
        - Parent 엔티티에서 식별자 클래스 직접 사용. @EmbeddedId
        - @EmbeddedId 는 식별자 클래스에 기본키를 직접 매핑
        - 조건
          - @Embeddable 어노테이션 사용
          - Serializable을 구현해야 한다.
          - equals, hashCode 를 재정의해야 한다.
          - 기본 생성자가 있어야 한다.
          - 식별자 클래스는 public이어야 한다.
        ```java
        // 저장
        Parent parent = new Parent();
        ParentId parentId = new ParentId("id1", "id2");
        parent.setId(parentId);
        em.persist(parent);
        
        // 조회
        ParentId parentId = new ParentId("id1", "id2");
        Parent parent = em.find(Parent.class, parentId);
        ```
        - 저장, 조회 시 식별자 클래스 사용
      - **복합 키 클래스에서 hashCode(), equals()**
        - 영속성 컨텍스트는 엔티티의 식별자를 키로 사용하여 엔티티를 관리함
        - 식별자 객체의 동등성(equals)이 지켜지지 않으면 영속성 컨텍스트가 동작하는 데 심각한 문제 발생
        - 재정의시 보통 모든 필드를 사용하여 구현
      - **@IdClass vs @EmbeddedId**
        - 각각 장단점이 있으므로 적절히 일관성 있게 사용
      > 복합키에는 @GenerateValue 사용 불가. 북합키 컬럼 중 하나에도 사용 불가

3. **복합키:식별 관계 매핑**
- 부모의 기본키를 포함하여 복합 키를 구성해야 함
- **@IdClass와 식별 관계**
    ```java
    // 부모
    @Entity
    public class Parent {
        @Id @Column(name = "PARENT_ID")
        private String id;
        private String name;
    }
    
    // 자식
    @Entity
    @IdClass(Child.class)
    public class Child {
        @Id
        @ManyToOne
        @JoinColumn(name = "PARENT_ID")
        private Parent parent;
        
        @Id @Column(name = "CHILD_ID")
        private String childId;
        private String name;
    }
    
    // 자식 ID
    public class ChildId implements Serializable {
        private String parent;
        private String childId;
        // equals, hashCode 재정의
    }
    
    // 손자
    @Entity
    @IdClass(GrandChildId.class)
    public class GrandChild {
        @Id
        @ManyToOne
        @JoinColumns({
                @JoinColumn(name = "PARENT_ID"),
                @JoinColumn(name = "CHILD_ID")
        })
        private Child child;
    
        @Id @Column(name = "GRANDCHILD_ID")
        private String id;
        private String name;
    }
    
    // 손자 ID
    public class GrandChildId implements Serializable {
        private String child;
        private String id;
        // equals, hashCode 재정의
    }
    ```
    - 식별 관계는 기본키와 외래키를 같이 매핑해야 한다.
    - @Id와 @ManyToOne 사용

- **@EmbeddedId와 식별 관계**
    ```java
    // 부모
    @Entity
    public class Parent {
        @Id @Column(name = "PARENT_ID")
        private String id;
        private String name;
    }
    
    // 자식
    @Entity
    public class Child {
        @EmbeddedId
        private ChildId id;
  
        @MapsId("parentId")
        @ManyToOne
        @JoinColumn(name = "PARENT_ID")
        private Parent parent;
        
        private String name;
    }
    
    // 자식 ID
    @Embeddable
    public class ChildId implements Serializable {
        private String parentId;
        @Column(name = "CHILD_ID")
        private String id;
        // equals, hashCode 재정의
    }
    
    // 손자
    @Entity
    public class GrandChild {
        @EmbeddedId
        private GrandChildId id;
  
        @MapsId("childId")
        @ManyToOne
        @JoinColumns({
                @JoinColumn(name = "PARENT_ID"),
                @JoinColumn(name = "CHILD_ID")
        })
        private Child child;
    
        private String name;
    }
    
    // 손자 ID
    @Embeddable
    public class GrandChildId implements Serializable {
        private ChildId childId;
  
        @Column(name = "GRANDCHILD_ID")
        private String id;
        // equals, hashCode 재정의
    }
    ```
    - 식별 관계로 사용할 연관관계의 속성에 @MapsId를 사용
    - @MapsId : 외래키와 매핑한 연관관계를 기본키에도 매핑하겠다는 뜻
      - 속성값 : @EmbeddedId 를 사용한 식별자 클래스의 기본키 필드 지정

4. **비식별 관계로 구현**
   ```java
   // 부모
    @Entity
    public class Parent {
        @Id @GeneratedValue 
        @Column(name = "PARENT_ID")
        private String id;
        private String name;
    }
    
    // 자식
    @Entity
    public class Child {
        @Id @GeneratedValue 
        @Column(name = "CHILD_ID")
        private String id;
        private String name;
   
        @ManyToOne
        @JoinColumn(name = "PARENT_ID")
        private Parent parent;
    }
    
    // 손자
    @Entity
    public class GrandChild {
        @Id @GeneratedValue 
        @Column(name = "GRANDCHILD_ID")
        private String id;
        private String name;
   
        @ManyToOne
        @JoinColumn(name = "CHILD_ID")
        private Child child;
    }
   ```
   - 복합키 사용하지 않아 코드가 단순해짐
5. **일대일 식별 관계**
   - 자식테이블의 기본키로 부모테이블의 기본키만 사용.
   - 부모 기본키가 복합키가 아니라면 자식도 아님
   ```java
   // 부모
    @Entity
    public class Board {
        @Id @GeneratedValue 
        @Column(name = "BOARD_ID")
        private Long id;
        private String title;
   
        @OneToOne(mappedBy = "board")
        private BoardDetail boardDetail;
    }
    
    // 자식
    @Entity
    public class BoardDetail {
        @Id
        private Long boardId;
   
        @MapsId // BoardDetail.boardId 매핑
        @OneToOne
        @JoinColumn(name = "BOARD_ID")
        private Board board;
        private String content;
    }
   ```
6. **식별, 비식별 관계의 장단점**  
   식별 관계보다 비식별 관계를 선호
   - DB 설계 관점 
     - 식별 관계는 자식 테이블의 기본 키 컬럼이 점점 늘어남. SQL이 복잡해지고 기본 키 인덱스가 불필요하게 커짐
     - 식별 관계는 복합 기본키를 만들어야 하는 경우가 많음
     - 식별 관계는 기본키로 비즈니스 의미가 있는 자연키 컬럼을 조합하는 경우가 많음
       - 비식별 관계는 대리키를 주로 사용. 비즈니스 요구사항은 영원하지 않다. 자식들까지 전파된 것들을 걷어내기 어렵다
     - 부모의 기본키를 자식의 기본키로 사용하므로 테이블 구조가 유연하지 못함  
   - 객체 관계 매핑의 관점
     - 일대일 관계를 제외하고 복합기본키를 사용하므로 별도의 클래스 작성 등 많은 노력이 필요
     - 비식별 관계는 대리키를 주로 사용하므로 @GeneratedValue 등 활용 가능
   - 식별 관계의 장점
     - 특정 상황에 조인 없이 하위 테이블만으로 검색 완료 가능
    - 비식별 > 식별, 필수적 비식별 > 선택적 비식별

### 4. 조인 테이블
- 외래키가 아니라 별도의 테이블로 연관관계 관리
- 기본은 조인 컬럼, 필요한 경우 조인 테이블 사용
1. **일대일 조인 테이블**
   - 조인 테이블의 외래키 컬럼 각각에 유니크 제약조건 필요
2. **일대다 조인 테이블**
   - '다'에 해당 하는 컬럼에 유니크 제약 조건
3. **다대일 조인 테이블**
   - 일대다와 테이블 형태 같음.
4. **다대다 조인 테이블**
   - 조인 테이블의 두 컬럼으로 복합 유니크 제약조건 걸어야 함.
- 조인 테이블 전략 사용시 다른 컬럼 추가할 수 없음


### 5. 엔티티 하나에 여러 테이블 매핑
- @SecondaryTable
```java
@Entity
@Table(name = "BOARD")
@SecondaryTable(name = "BOARD_DETAIL", pkJoinColumns = @PrimaryKeyJoinColumn(name = "BOARD_DERAIL_ID"))
public class Board {
    @Id @GeneratedValue
    @Column(name = "BOARD_ID")
    private Long id;
    private String title;

    @Column(table = "BOARD_DETAIL") // 테이블을 지정하지 않으면 기본 테이블에 매핑
    private String content; // BOARD_DETAIL 에 있는 컬럼
}
```
- @SecondaryTable사용보다는 각각 엔티티를 만들어서 일대일 매핑하는 것을 권장

