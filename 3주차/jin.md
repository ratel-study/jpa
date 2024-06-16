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
