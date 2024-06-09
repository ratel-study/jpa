# 단방향 연관관계
member
- team_id(fk)
- member_id(pk)
- username

team
- team_id (pk)
- name
  

객체 연관관계
- 회원 객체와 팀 객체는 단방향 관계다.
테이블 연관관계
- 회원 테이블과 팀 테이블은 양방향 관계다.
- TEAM_ID 외래 키를 통해서 회원과 팀을 조인할 수도 있고 반대로 팀과 회원도 조인할 수 있다.


코드
```
public class Member {

  @ManyToOne
  @JoinColumn(name = "TEAM_ID")
  private Team team;
```
@JoinColumn(name = "TEAM_ID")   
name에는 매핑할 외래 키 이름을 지정한다.

@ManyToONe
fetch: fetch 전략을 설정한다.
cascade: 영속성 전이 기능을 사용한다.

## 저장
```
Team team1 = new Team("team1", "팀1");
em.persist(team1);

Meamber member1 - new Member("member1", "회원1");
member1.setTeam(team1);
em.persist(member1);
```

## 조회
```
String jpql = "select m from Member m join m.team t where t.name=:teamMane";
List<Member> resultList = em.createQuery(jpql, Member.class)
   .setParameter("teamName", "팀1");
   .getResultList();
```

## 수정
```
Member member = em.find(Member.class, "member1");
member.setTeam(team2);
```
update member set team_id='team2'

## 연관관계 제거
```
Member member1 = em.find(Member.class, "member1");
member1.setTeam(null);
```
UPDATE MEMBER SET TEAM_ID=null


# 양방향 연관관계
```
@ManyToOne
@JoinColumn(name = "TEAM_ID")
private Team team;
```
```
@OneToMany(mappedBY = "team")
private List<Member> members new ArrayList<Member>();
```
테이블은 외래 키 하나로 두 테이블의 연관관계를 관리한다.   
엔티티를 양방향 연관관계로 설정하면 객체의 참조는 둘인데 외래 키는 하나다. 따라서 둘 사이에 차이가 발생한다.   
두 객체 연관관계 중 하나를 정해서 테이블의 외래키를 관리해야 하는데 이것을 연관관계의 주인이라고 한다.   
연관관계의 주인만이 데이터베이스 연관관계와 매핑되고 외래 키를 관리할 수 있다?   
만약 회원 엔티티에 있는 Member.team을 주인으로 선택하면 자기 테이블에 있는 외래 키를 관리하면 된다. 하지만 팀 엔티티에 있는 Team.members를 주인으로 선택하면 물리적으로 전혀 다른 테이블의 외래 키를 관리해야 한다.

## 양방향 영속
```
Team team1 = new Team("team1", "팀1");
em.persist(team1);

Member member1 = new Member("member1", "회원1");
member1.setTeam(team1);
team1.getMembers().add(member1);
em.persist(member1);

Member member2 = new Member("member2", "회원2");

member2.setTeam(team1);
team1.getMembers().add(member2);
em.persist(member2);
```

# 다양한 연관관계 매핑
## 다대일 단방향
Member
- MEMBER_ID(PK)
- TEAM_ID(FK)
- USERNAME
TEAM
- TEAM_ID(PK)
- NAME
```
@ManyToOne
@JoinColumn(name="TEAM_ID")
private Team team;
```
## 다대일 양방향
```
@ManyToOne
@JoinColumn(name = "TEAM_ID")
private Team team;
```
```
@OneToMany(mappedBy = "team")
private List<Member> members = new ArrayList<Member>();
```

## 일대다 단방향
```
@OneToMany
@JoinColumn(name = "TEAM_ID")
private List<Member> members = new ArrayList<Member>();
```

단점
```
Member member1 = new Member();
Member member2 = new Member();

Team team1 = new Team();
team1.getMembers().add(member1);
team2.getMembers().add(member2);

em.persist(member1); // insert문 1개
em.persist(member2); // insert문 1개
em.persist(team1); // insert문 1개, update문 2개
transaction.commit();
```
update문 2개가 추가로 날아간다.

## 일대다 양방향
일대다 양방향이나 다대일 양방향은 똑같은 말이다.

## 일대일 단방향
MEMBER
- MEMBER_ID(PK)
- LOCKER_ID(FK, UNI)
- USERNAME
LOCKER
- LOCKER_ID(PK)
- NAME
```
@OneToOne
@JoinColumn(name = "LOCKER_ID")
private Locker locker;
```

## 일대일 양방향
```
@OneToOne
@JoinColumn(name = "LOCKER_ID")d
private Locker locker;

@OneToOne(mappedBy = "locker")
private Member member;
```

## 허용하지 않는 단방향
MEMBER
- MEMBER_ID(PK)
- USERNAME
LOCKER
- LOCKER_ID(PK)
- MEMBER_ID(FK, UNI)
- NAME
위 디비 구조에서 다음 엔티니 구조는 허용되지 않는다.
Member
- id
- Locker locker
- username
Locker
- id
- name

## 다대다 단방향
중간 테이블이 필요하다.
MEMBER
- MEMBER_ID(PK)
- USERNAME
MEMBER_PRODUCT
- MEMBER_ID(PK,FK)
- PRODUCT_ID(PK, FK)
PRODUCT
- PRODUCT_ID(PK)
- NAME

```
@ManyToMany
@JoinTable(name = "MEMBER_PRODUCT", joinCOLumns = @JoinColumn(name = "MEMBER_ID"),
        inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID"))
private List<Product> products = new ArrayList<Product>();
```
@JoinTable.name : 연결 테이블 지정
@JoinTable.joinColumns: 현재 방향인 회원과 매핑할 조인 컬럼 정보 지정
@JoinTable.inverseJoinColumns: 반대 방향인 상품과 매핑할 조인 컬럼 정보 지정

## 다대다 양방향
```
@ManyToMany
@JoinTable(name = "MEMBER_PRODUCT", joinCOLumns = @JoinColumn(name = "MEMBER_ID"),
        inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID"))
private List<Product> products = new ArrayList<Product>();
```
```
@ManyToMany(mappedBy = "products")
private List<Member> members;
```

## 다대다 매핑을 다대일로 풀자
MEMBER
- MEMBER_ID(PK)
- USERNAME
MEMBER_PRODUCT
- MEMBER_ID(PK,FK)
- PRODUCT_ID(PK, FK)
- ORDERAMOUNT
- ORDERDATE
PRODUCT
- PRODUCT_ID(PK)
- NAME

```
public class Member {
  @OneToMany(mappedBy = "member")
  private List<MemberProduct> memberProducts;
}

@IdClass(MemberProductId.class)
public class MemberProduct {
  @Id
  @ManyToOne
  @JoinColumn(name = "MEMBER_ID")
  private Member member;

  @Id
  @ManyToONe
  @JoinColumn(name = "PRODUCT_ID")
  private Product product;
}
```

# 고급 매핑
## 상속 관계 매핑
ITEM
- ITEM_ID(PK)
- NAME
- PRICE
- DTYPE
ALBUM
- ITEM_ID(PK,FK)
- ARTIST
MOVIE
- ITEM_ID(PK,FK)
- DIRECTOR
- ACTOR
BOOK
- ITEM_ID(PK,FK)
- ISBN
```
@Inheritace(strategy = InheritanceType.JOINED)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {
  @Id @GeneratedValue
  @Column (name = "ITEM_ID")
  private Long id;

  private String name;
  private int price;
}

@Entity
@DiscriminatorValue("A")
public class Album extends Item {
  private String artist;
}

@Entity
@DiscriminatorValue("M")
public class Movie extends Item {
  private String director;
  private String actor;
}
```

장점
- 테이블이 정규화된다.
- 저장공간을 효율적으로 사용한다.

단점
- 조회할 때 조인이 많이 사용된다.

ITEM
- ITEM_ID(PK)
- NAME
- PRICE
- ARTIST
- DIRECTOR
- ACTOR
- AUTHOR
- ISBN
- DTYPE
```
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {
}

@DiscriminatorValue("A")
public class Album extends Item {}

@DiscriminatorValue("B")
public class Movie extends Item {}
```
ALBUM
- ITEM_ID(PK)
- NAME
- PRICE
- ARTIST

MOVIE
- ITEM_ID(PK)
- NAME
- PRICE
- DIRECTOR
- ACTOR

BOOK
- ITEM_ID(PK)
- NAME
- PRICE
- AUTHOR
- ISBN
```
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract clas Item {
}

@Entity
public class Album extends Item {}
@Entity
public class Movie extends Item {}
@Entity
public clsas Book extends Item {}
```

## 복합 키와 식별 관계 매핑
식별 관계: 부모 테이블의 기본 키를 내려받아서 자식 테이블의 기본 키 + 외래 키로 사용하는 관계다.   
비식별 관계: 부모 테이블의 기본 키를 받아서 자식 테이블의 외래 키로만 사용하는 관계다.   
ORM 신규 프로젝트 진행시 추천하는 방법은 될 수 있으면 비식별 관계를 사용하고 기본키는 Long 타입의 대리 키를 사용하는 것이다.   

비식별 관계 매핑
```
@IdClass(ParentId.class)
public class Parent {
  @Id
  @Column(name = "PARENT_ID1")
  private String id1;

  @Id
  @Column(name = "PARENT_ID2")
  private String id2;
}

public class ParentId implements Serializable {
...
}

public class Child {
  @Id
  private String id;

  @ManyToOne
  @JoinColumns({
    @JoinColumn(name = "PARENT_ID1",
      referenceColumnName = "PARENT_ID1"),
    @JoinColumn(name = "PARENT_ID2",
      referenceColumnName = "PARENT_ID2")
  })
  private Parent parent;

}
```


## 조인 테이블
일대일
PARENT
- PARENT_ID(PK)
- NAME
PARENT_CHILD
- PARENT_ID(PK, FK)
- CHILD_ID(UNI, FK)
CHILD
- CHILD_ID(PK)
- NAME

```
@OneToOne
@JoinTable(name = "PARENT_CHILD",
           joinColumns = @JoinColumn(name = "PARENT_ID"),
           inverseJoinColumns = @JoinColumn(name = "CHILD_ID")
)
private Child child;

```



## 엔티티 하나에 여러 테이블 매핑하기
BOARD
- BOARD_ID(PK)
- title
BOARD_DETAIL
- BOARD_DETAIL_ID(PK,FK)
- content

```
@SecondaryTable(name = "BOARD_DETAIL", pkJoinColumns = @PrimaryKeyJoinColumn(name = "BOARD_DETAIL_ID"))
public class Board {

  @Column(table = "BOARD_DETAIL")
  private String content;
}

```
