# 8장
jpa에서 런타임에 새로운 클래스를 만들어 다이나믹 프록시를 생성한다.

## 프록시의 특징
- 프록시 객체는 처음 사용할 때 한 번만 초기화한다.
- 프록시 객체를 초기화한다고 프록시 객체가 실제 엔티티로 바뀌는 것은 아니다.
- 프록시 객체는 원본 엔티티를 상속받은 객체이므로 타입 체크 시에 주의해서 사용해야 한다.
- 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 데이터베이스를 조회할 필요가 없으므로 em.getReference()를 호출해도 프록시가 아닌 실제 엔티티를 반환한다.

## 즉시 로딩
- 엔티티를 조회할 때 연관된 엔티티도 함께 조회한다.
- @ManyToOne(fetch = FetchType.EAGER)
```
@Entity
public class Member {
  @ManyToOne(fetch = FetchType.EAGER)
  private Team team;
}
```

```
Member member = em.find(Member.class, "member1");
Team team = member.getTeam();
```

```
SELECT
  M.MEMBER_ID AS MEMBER_ID,
  M.TEAM_ID AS TEAM_ID,
  M.USERNAME AS USERNAME,
  T.TEAM_ID AS TEAM_ID,
  T.NAME AS NAME
FROM
  MEMBER M LEFT OUTER JOIN TEAM T
    ON M.TEAM_ID = T.TEAM_ID
WHERE
M.MEMBER_ID='member1'
```

## 지연 로딩
- 연관된 엔티티를 실제 사용할 때 조회한다.
- @ManyToOne(fetch = FetchType.LAZY)
```
@Entity
public class Member {
  @ManyToONE(fetch = FetchType.LAZY)
  @JoinColumn(name = "TEAM_ID")
  private Team team;
}
```

```
Member member = em.find(Member.class, "member1");
// select from MEMBER where MEMBER_ID = 'member1'
Team team = member.getTeam();
team.getName();
// select * from TEAM WHERE TEAM_ID = 'team1'
```

## 영속성 전이:저장 CascaseType.PERSIST
자식 엔티티까지 같이 영속화한다.
```
@Entity
public class Parent {
  @OneToMany(mappedBy = "parent", cascade =CascadeType.PERSIST)
  private List<Child> children = new ArrayList<Child>();
}

private static void saveWithCascade(EntityManager em) {
  Child child1 = new Child();
  Child child2 = new Child();

  Parent parent = new Parent();
  child1.setParent(parent);
  child2.setParent(parent);
  parent.getChildren().add(child1);
  parent.getChildren().add(child2);

  em.persist(parent);
}
```

## 영속성 전이: 삭제 CascadeType.REMOVE
```
Parent findParent = em.find(Parent.class, 1L);
em.remove (findParent);

// Child findChild1 = em.find(Child.class, 1L);
// Child findChild2 = em.find(Child.class, 2L);
```

## 고아 객체
```
@Entity
public class Parent {
  @OneToMany(mappedBy = "parent", orphanRemoval = true)
  private List<Child> children = new ArrayList<Child>();
```

```
Parent parent1 = em.find(Parent.class, id);
parent1.getChildren().remove(0); // -> DELETE FROM CHILD WHERE ID=?
```

# 9장
값 객체 타입
- 기본값
- 임베디드
- 컬렉션 값

## 임베디드
```
@Entity
public class Member {
  @Id @GeneratedValue
  private Long id;
  private String name;

  @Embedded Period workPeroid;
  @Embedded Address homeAddress;
```

```
@Embeddable
public class Period {
  @Temporal(TemporalType.DATE) java.util.Date startDate;
  @Temporal(TemporalType.DATE) java.util.Date endDate;
}
```

## 컬렉션 값
```
@ElementCollection
@CollectionTable(name = "FAVORITE_FOODS", joinColumns = @JoinColumn(name = "MEMBER_ID"))
@Column(name = "FOOD_NAME");
private Set<String> favoriteFoods = new HashSet<String>();

@ElementCollection
@CollectionTable(name = "ADDRESS", joinColumns = @JoinColumn(name = "MEMBER_ID"))
private List<Address> addressHistory = new ArrayList<Address>();
```


# 10장
## TypedQuery
```
TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m", Member.class);

Query query = em.createQuery("SELECT m.username, m.age from Member m");
```

## 이름 기준 파라미터
```
TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m where m.username = :username", Member.class);
```

## 프로젝션
엔티티 프로젝션
```
select m from Member m
```
조회한 엔티티는 영속성 컨텍스트에서 관리된다.

임베디드 타입 프로젝션
```
String query = "SELECT o.address FROM Order o"
List<Address> addresses = em.createQuery(query, Address.class).getResultList();
```
임베디드 타입은 엔티티 타입이 아닌 값 타입이다. 따라서 이렇게 조회한 임베디드 타입은 영속성 컨텍스트에서 관리되지 않는다.

## fetch join
```
select t from Team t join fetch t.members where t.name = '팀A'
```
sql
```
SELECT T.*, M.* FROM TEAM T
INNER JOIN MEMBER M ON T.ID = M.TEAM_id
WHERE T.NAME = '팀A"
```
최적화를 위해 글로벌 로딩 전략을 즉시 로딩으로 설정하면 애플리케이션 전체에서 항상 즉시 로딩이 일어난다.   
물론 일부는 빠를 수는 있지만 전체로 보면 사용하지 않는 엔티티를 자주 로딩하므로 오히려 성능에 악영향을 미칠 수 있다.    
따라서 글로벌 로딩 전략은 될 수 있으면 지연 로딩을 사용하고 최적화가 필요하면 페치 조인을 적용하는 것이 효과적이다.

##  그 외
###  엔티티를 파라미터로 받는 코드
```
String qlString = "select m from Member m where m = :member";
List resultList = em.createQuery(qlString).setParameter("member", member)
                    .getResultList();

```
-> select m.* from Member m where m.id=?

```
Team team = em.find(Team.class, 1L);

String qlString = "select m from Member m where m.team = :team";
List resultList = em.createQuery(qlString)
                    .setParameter("team", team)
                    .getResultList();
```
-> select m.* from Member m where m.team_id=?
