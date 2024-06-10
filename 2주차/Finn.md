# Finn
JPA 스터디

---

## 5장 - 연관관계 매핑 기초

---

객체는 참조를 사용해서 관계를 맺고 테이블은 외래 키를 사용해서 관계를 맺는다.

연관관계 매핑을 이해하기 위해서는 다음 핵심 키워드를 알아야한다.

- 방향: `단방향`, `양방향`이 있다.
  - 회원과 팀이 있을 때 `회원 -> 팀`, `팀 -> 회원` 한 쪽만 참조하는 것을 **단방향 관계**
  -  `회원 -> 팀`, `팀 -> 회원` 양쪽 모두 서로 참조하는 것을 **양방향 관계**
- 다중성: `다대일`, `일대다`, `일대일`, `다대다` 가 있다.
  - 여러 회원은 한 팀에 속하므로 회원과 팀은 `다대일` 관계다.
  - 한 팀에 여러 회원이 속할 수 있으므로 팀과 회원은 `일대다` 관계이다.
- 연관관계의 주인: 객체를 양방향 연관관계로 만들면 연관관계의 주인을 정해야 한다.

### 단방향 연관관계

회원과 팀의 다대일 단방향 관계를 알아본다.

- 회원과 팀이 있다.
- 회원은 하나의 팀에만 소속될 수 있다.
- 회원과 팀은 다대일 관계다.

#### 객체 연관관계

- 회원 객체는 `Member.team` 필드(멤버변수)로 팀 객체와 연관관계를 맺는다.
- 회원 객체와 팀 객체는 **단방향 관계**이다.
- 회원은 `Member.team` 필드를 통해서 팀을 알 수 있지만 **팀은 회원을 알 수 없다.**

#### 테이블 연관관계

- 회원 테이블은 `TEAM_ID` 외래 키로 팀 테이블과 연관관계를 맺는다.
- 회원 테이블과 팀 테이블은 **양방향 관계**이다.
- 회원 테이블의 `TEAM_ID` 외래 키를 통해 회원과 팀을 조인하고 반대로 팀과 회원도 조인할 수 있다.

#### 객체 연관관계와 테이블 연관관계의 가장 큰 차이

- 참조를 통한 연관관계는 언제나 단방향이다.
  - 양방향으로 만ㄷ늘고 싶으면 반대쪽에도 필드를 추가해 참조를 보관해야 한다.
- 양쪽에서 서로 참조하는 것을 양방향 연관관계라 한다.
  - 정확히는 **서로 다른 단방향 관계 2개이다.**
- 테이블은 외래 키 하나로 양방향으로 조인할 수 있다.

```java
// 단방향 연관관계
class A {
  B b;
}

class B {

}
```

```java
// 양방향 연관관계
class A {
  B b;
}

class B {
  A a;
}
```

#### 객체 연관관계 VS 테이블 연관관계 정리

- 객체는 참조로 연관관계를 맺는다.
- 테이블은 외래 키로 연관관계를 맺는다.
- 참조를 사용하는 객체의 연관관계는 단방향이다.
  - `A -> B(a.b)`
- 외래 키를 사용하는 테이블의 연관관계는 양방향이다.
  - `A JOIN B` 가 가능하면 `B JOIN A` 도 가능하다.
- 객체를 양방향으로 참조하려면 단방향 연관관계를 2개 만들어야 한다.
  - `A -> B(a.b)`
  - `B -> A(b.a)`

### 순수한 객체 연관관계

순수한 객체를 사용한 연관관계를 알아본다.

JPA를 사용하지 않은 순수한 회원과 팀 클래스 코드이다.

```java
public class Team {
  private String id;
  private String name;
}

public class Member {

  public Member(final String id, final String username) {
    this.id = id;
    this.username = username;
  }

  private String id;
  private String username;

  private Team team;

  public void setTeam(final Team team) {
    this.team = team;
  }

  public Team getTeam() {
    return team;
  }
}

```

```java
// 실행 코드
    Member member1 = new Member("member1", "회원1");
    Member member2 = new Member("member2", "회원2");
    Team team = new Team("team1", "팀1");

    member1.setTeam(team);
    member2.setTeam(team);

    Team findTeam = member1.getTeam();
```

코드에서 회원1과 회원2는 팀1에 소속했다.

`Team findTeam = member1.getTeam();` 코드를 통해 회원1이 소속한 팀1을 조회할 수 있다.

이렇게 객체는 참조를 이용해 연관관계를 탐색할 수 있는데 이를 **객체 그래프 탐색**이라고 한다.

### 테이블 연관관계

```sql
CREATE TABLE MEMBER (
  MEMBER_ID VARCHAR(255) NOT NULL,
  TEAM_ID VARCHAR(255),
  USERNAME VARCHAR(255),
  PRIMARY KEY (MEMBER_ID)
)

CREATE TABLE TEAM (
  TEAM_ID VARCHAR(255) NOT NULL,
  NAME VARCHAR(255),
  PRIMARY KEY (TEAM_ID)
)

ALTER TABLE MEMBER ADD CONSTRAINT FK_MEMBER_TEAM
FOREGN KEY (TEAM_ID)
REFERENCES TEAM
```

회원 테이블과 팀 테이블을 생성한다.

추가로 회원 테이블의 `TEAM_ID`에 외래키 제약 조건을 설정했다.

```sql
INSERT INTO TEAM(TEAM_ID, NAME) VALUES('team1', '팀1');
INSERT INTO MEMBER(MEMBER_ID, TEAM_ID, USERNAME) VALUES('member1', 'team1', '회원1');
INSERT INTO MEMBER(MEMBER_ID, TEAM_ID, USERNAME) VALUES('member2', 'team1', '회원2');
```

```sql
SELECT T.*
FROM MEMBER M
JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
WHERE M.MEMBER_ID = 'member1'
```

위처럼 데이터베이스는 외래 키를 사용해 연관관계를 탐색할 수 잇는데 이를 **조인**이라 한다.

### 객체 관계 매핑

이제 JPA를 사용해 둘을 매핑해본다.

```java
@Entity
public class Team {
  
  @Id
  @Column(name = "TEAM_ID")
  private String id;
  
  private String name;
}

@Entity
public class Member {

  @Id
  @Column(name = "MEMBER_ID")
  private String id;

  private String username;

  // 연관관계 매핑
  @ManyToOne
  @JoinColumn(name = "TEAM_ID")
  private Team team;

  // 연관관계 설정
  public void setTeam(Team team) {
    this.team = team;
  }
}

```

- 객체 연관관계: 회원 객체의 `Member.team` 필드 사용
- 테이블 연관관계: 회원 테이블의 `MEMBER.TEAM_ID` 외래 키 컬럼을 사용

```java

  // 연관관계 매핑
  @ManyToOne
  @JoinColumn(name = "TEAM_ID")
  private Team team;
```

- `@ManyToOne`: 다대일 관계 매핑 어노테이션이다.
  - 연관관계를 매핑할 때 다중성을 나타내는 어노테이션을 필수로 사용한다.
- `@JoinColumn(name = "TEAM_ID")`: 조인 컬럼은 외래 키를 매핑할 때 사용한다.
  - name 속성에는 매핑할 외래 키 이름을 지정한다.
  - 이 어노테이션은 생략할 수 있다.

### @JoinColumn

`@JoinColumn`은 외래 키를 매핑할 때 사용한다. 속성은 다음과 같다.

- name: 매핑할 외래 키 이름
  - 기본값: `필드명 + _ + 참조하는 테이블의 기본 키 컬럼 명`
- referncedColumnName: 외래 키가 참조하는 대상 테이블의 컬럼 명
  - 참조하는 테이블의 기본 키 컬럼명
- foreignkey(DDL): 외래 키 제약 조건을 직접 지정할 수 있다.
  - 테이블을 생성할 때만 사용한다.

`@JoinColumn`을 생략하면 외래 키를 찾을 때 기본 전략을 사용한다.

### @ManyToOne

`@ManyToOne`은 다대일 관계에서 사용한다. 속성은 다음과 같다.

- optional: `false`로 설정하면 연관된 엔티티가 항상 있어야한다.
- fetch: 글로벌 패치 전략을 설정한다.
- cascade: 영속성 전이 기능을 사용한다.
- targetEntity: 연관된 엔티티의 타입 정보를 설정한다.
  - 이 기능은 거의 사용하지 않는다.
  - 컬렉션을 사용해도 제네릭으로 타입 정보를 알 수 있다.

`@ManyToOne`과 비슷한 `@OneToOne` 관계도 있다.

반대편이 일대다 관계면 다대일을 일대일 관계이면 일대일을 사용하면 된다.

### 연관관계 사용

#### 저장

```java
    Team team = new Team("team1", "팀1");
    em.persist(team);

    Member member1 = new Member("member1", "회원1");
    member1.setTeam(team);
    em.persist(member1);

    Member member2 = new Member("member2", "회원2");
    member2.setTeam(team);
    em.persist(member2);
```

JPA에서 엔티티를 저장할 때 연관된 모든 엔티티는 영속 상태여야한다는 걸 주의해야한다.

위의 코드에서는 회원 엔티티는 팀 엔티티를 참조하고 저장했다.

JPA는 참조한 팀의 식별자(team.id)를 외래 키로 사용해 적절한 등록 쿼리를 생성한다.

#### 조회

연관관계가 있는 엔티티를 조회하는 방법은 크게 2가지이다.

- 객체 그래프 탐색(객체 연관관계를 사용한 조회)
- 객체지향 쿼리 사용(JPQL)

`member.getTeam()` 을 사용해 `member`와 연관된 `team` 엔티티를 조회할 수 있다.

객체지향 쿼리인 JPQL을 사용해 연관관를 조회할 수 있다.

```jpql
select m from Member m join m.team t
where t.name = :teamName
```

#### 수정

```java
    Team team2 = new Team("team2", "팀2");
    em.persist(team2);
    
    Member member = em.find(Member.class, "member1");
    member.setTeam(team2);
```

불러온 엔티티의 값만 변경해두면 트랜잭션을 커밋할 때 플러시가 일어나면서 변경 감지 기능이 작동한다.

이는 연관관계 수정시에도 마찬가지인데 참조하는 대상만 변경하면 나머지는 JPA가 자동으로 처리한다.

#### 연관관계 제거

```java
    Member member1 = em.find(Member.class, "member1");
    member1.setTeam(null); // 연관관계 제거
```

연관관계를 `null`로 설정하여 연관관계를 제거했다.

#### 연관된 엔티티 삭제

연관된 엔티티를 삭제하려면 기존에 있던 연관관계를 먼저 제거하고 삭제해야 한다.

외래 키 제약 조건으로 인해 데이터베이스에서 오류가 발생하는 걸 방지하기 위함이다.

```java
member1.setTeam(null);  // 회원 1 연관관계 제거
member2.setTeam(null);  // 회원 2 연관관계 제거
em.remove(team) // 팀 삭제
```

### 양방향 연관관계

팀에서 회원으로 접근할 수 있게 양방향 연관관계로 매핑해본다.

회원과 팀은 `다대일`관계이다.

반대로 팀과 회원은 `일대다` 관계이다.

`일대다`관계에서는 여러 건과 연관관계를 맺을 수 있기 때문에 컬렉션을 사용한다.

JPA에서는 `List`를 포함한 `Collection`, `Set`, `Map` 같은 다양한 컬렉션을 지원한다.

데이터베이스 테이블은 외래 키 하나로 양방향으로 조회할 수 있다.

외래 키 하나만으로 양방향 조회가 가능하므로 처음부터 **양방향 관계이다.**

### 양방향 연관관계 매핑

```java
@Entity
public class Member {
  @Id
  @Column(name = "MEMBER_ID")
  private String id;

  private String username;

  // 연관관계 매핑
  @ManyToOne
  @JoinColumn(name = "TEAM_ID")
  private Team team;

  // 연관관계 설정
  public void setTeam(Team team) {
    this.team = team;
  }

  public Team getTeam() {
    return team;
  }

  public Member(final String id, final String username) {
    this.id = id;
    this.username = username;
  }
}
```
회원 엔티티는 변경한 부분이 없다.

```java
@Entity
public class Team {

  @Id
  @Column(name = "TEAM_ID")
  private String id;

  private String name;
  
  // 추가
  @OneToMany(mappedBy = "team")
  private List<Member> members = new ArrayList<>();

  public Team(final String id, final String name) {
    this.id = id;
    this.name = name;
  }

  public Team() {
    
  }
}
```

회원과 팀은 일대다 관계이기 때문에 `@OneToMany`를 사용해 양방향 관계를 매핑했다.

`mappedBy` 속성은 양방향 매핑일 때 **반대쪽 매핑의 필드 이름**을 넣어주면 된다.

이제부터 팀에서 회원 컬렉션으로 객체 그래프를 탐색할 수 있다.

```java
Team team = em.find(Team.class, "team1");
List<Member> members = team.getMembers();

for (Member member : members) {
  System.out.println(member.getUsername());
}
```

### 연관관계의 주인

엄밀히 말해 객체에는 양방향 연관관계라는 것이 없다.

서로 다른 단방향 연관관계 2개를 양방향인 것처럼 보이게 할 뿐이다.

객체 연관관계는 다음과 같다.

- 회원 -> 팀 (단방향)
- 팀 -> 회원 (단방향)

엔티티를 양방향 연관관계로 설정하면 객체의 참조는 둘인데 **외래 키는 하나다.**

두 객체의 연관관계 중 하나를 정해 테이블의 외래 키를 관리해야 하는데 이걸 **연 관관계의 주인**이라고 한다.

### 양방향 매핑의 규칙: 연관관계의 주인

양방향 연관관계 매팽시에는 두 연관관계 중 하나를 **연관관계의 주인**으로 정해야한 한다.

**연관관계의 주인만이 외래 키를 등록, 수정, 삭제할 수 있다.**

**주인이 아닌 쪽은 읽기만 할 수 있다.**

연관관계의 주인은 `mappedBy` 속성을 사용해 정한다.

- 주인은 `mappedBy` 속성을 사용하지 않는다.
- 주인이 아니면 `mappedBy` 속성을 사용해 연관관계의 주인을 지정한다.

연관관계의 주인을 정한다는 것은 **외래 키 관리자를 선택하는 것**이다.

### 연관관계의 주인은 외래 키가 있는 곳

연관관계의 주인은 테이블에 외래 키가 있는 곳을 정해야 한다.

예제에서는 회원 테이블이 외래 키를 가지고 있기 때문에 `Member.team`이 주인이 된다.

데이터베이스 테이블의 다대일, 일대 다 관계에서는 항상 **다** 쪽이 외래 키를 가진다.

따라서 `@ManyToOne`에는 `mappedBy` 속성이 없다.

### 양방향 연관관계 저장

```java
Team team1 = new Team("team1", "팀1");
em.persist(team1);

Member member1 = new Member("member1", "회원1");
member1.setTeam(member1);
em.persist(member1);

Member member2 = new Member("member2", "회원2");
member2.setTeam(member2)
em.persist(member2);
```

연관관계 주인을 통해 팀과 회원을 저장했다.

양방향 연관관계에서는 연관관계의 주인이 외래 키를 관리하기 때문에 **주인이 아닌 방향은 값을 설정하지 않아도 데이터베이스에 외래 키 값이 정상 입력된다.**

**주인이 아닌 곳에 입력된 값은 외래 키에 영향을 주지 않는다.**

엔티티 매니저는 이곳에 입력된 값을 사용해서 외래 키를 관리한다.

### 양방향 연관관계의 주의점

양방향 연관관계에서 주의할 점은 연관관계의 주인에는 값을 입력하지 않고 주인이 아닌 곳에만 값을 입력하면 정상적으로 저장이되지 않는다는 것이다.

```java
Member member1 = new Member("member1", "회원1");
em.persist(member1);

Member member2 = new Member("member2", "회원2");
em.persist(member2);

Team team1 = new Team("team1", "팀1");
// 주인이 아닌 곳만 연관관계 설정
team1.getMembers().add(member1);
team1.getMembers().add(member2);
em.persist(team1);
```

연관관계의 주인만이 외래 키의 값을 변경할 수 있다.

### 순수한 객체까지 고려한 양방향 연관관계

**객체 관점에서 양쪽 방향에 모두 값을 입력해주는 것이 가장 안전하다.**

그렇지 않으면 JPA를 사용하지 않는 순수한 객체 상태에서 심각한 문제가 발생할 수 있다.

ORM은 객체와 관계형 데이터베이스 둘 다 중요하기 때문에 데이터베이스와 객체를 함께 고려해야 한다.

객체까지 고려하면 양쪽 다 관계를 맺어야 한다.

```java
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

양쪽에 연관관계를 설정했기 때문에 순수한 객체 상태에서도 동작한다.

테이블의 외래 키도 정상 입력된다.

이때는 연관관계의 주인인 `Member.team` 값을 사용한다.

### 연관관계 편의 메서드

양방향 연관관계는 양쪽 다 신경써야 한다.

양쪽에 연관관계를 맺다보면 실수로 둘 중 하나만 호출해 양방향이 깨질 수 있다.

```java
  public void setTeam(Team team) {
    this.team = team;
    team.getMembers().add(this);
  }
```

위의 코드처럼 메소드 하나로 양방향 관계를 모두 설정할 수 있도록한다.

한 번에 양방향 관계를 설정하는 메서드를 **연관관계 편의 메서드**라 한다.

### 연관관계 편의 메서드 작성 시 주의사하아

`setTeam()` 메서드에는 버그가 있다.

`setTeam` 호출 후 다른 팀으로 변경할 때 기존 팀과 회원의 연관관계를 삭제하는 코드를 추가해야 한다.

```java
  public void setTeam(Team team) {
    // 기존 팀과 관계를 제거
    if (this.team != null) {
      this.team.getMembers().remove(this);
    }
    this.team = team;
    team.getMembers().add(this);
  }
```

객체에서 양방향 연관관계를 사용하려면 로직을 견고하게 작성해야 한다.

### 정리

단방향 매핑과 비교해 양방향 매핑은 복잡하다.

**양방향의 장점은 반대방향으로 객체 그래프 탐색 기능이 추가된 것 뿐이다.**

주인의 반대편읜 `mappedBy`를 사용해 주인을 지정해야 한다.

주인의 반대편은 단순히 객체 그래프 탐색만 할 수 있다.

- 단방향 매핑만으로 테이블과 객체의 연관관계 매핑은 이미 완료되었다.
- 단방향을 양방향으로 만들면 반대방향으로 객체 그래프 탐색 기능이 추가된다.
- 양방향 연관관계를 매핑하려면 객체에서 양쪽 방향을 모두 관리해야 한다.

#### 연관관계의 주인을 정하는 기준

연관관계의 주인은 외래 키의 위치와 관련해서 정해야지 비즈니스 중요도로 접근하면 안 된다.

양방향 매핑시에는 무한루프에 빠지지 않게 조심해야 한다.

`일대다`를 연관관계의 주인으로 선택하는 것이 불가능한 것은 아니지만 성능과 관리 측면에서 권장하지 않는다.

---

## 6장 - 다양한 연관관계 매핑

---

엔티티의 연관관계를 매핑할 때는 다음 3가지를 고려해야 한다.

- 다중성
- 단방향, 양방향
- 연관관계의 주인

### 다대일

다대일 관계의 반대 방향은 항상 `일대다` 관계이다.

일대다 관계의 반대 방향은 항상 `다대일` 관계이다.

데이터베이스에서는 항상 `다`쪽이 외래 키를 가지ㅏ고 있다.

객체의 양방향 관계에서 연관관계의 주인은 항상 `다`쪽이다.

회원과 팀이 `다대일` 관계라면 회원 쪽이 연관관계의 주인이다.

### 다대일 단방향 [N:1]

회원과 팀의 `다대일` 단방향 연관관계 예제이다.

```java
@Entity
public class Member {
  
  @Id @GeneratedValue
  @Column(name = "MEMBER_ID")
  private Long id;
  
  private String  username;
  
  @ManyToOne
  @JoinColumn(name = "TEAM_ID")
  private Team team;
}

```

```java
@Entity
public class Team {
  
  @Id @GeneratedValue
  @Column(name = "TEAM_ID")
  private Long id;
  
  private String name;
}
```

회원은 `Member.team` 으로 팀 엔티티를 참조할 수 있다.

반대로 팀에서는 회원을 참조하는 필드가 없다.

회원과 팀은 `다대일 단방향` 연관관계이다.

`@JoinColumn(name = "TEAM_ID")` 을 사용해 `Memeber.team` 필드를 `TEAM_ID` 외래 키와 매핑했다.

`Member.team` 필드로 회원 테이블의 `TEAM_ID` 외래 키를 관리한다.

### 다대일 양방향 [N:1, 1:N]

회원과 팀의 다대일 양방향 관계이다.

```java
@Entity
public class Team {

  @Id
  @GeneratedValue
  @Column(name = "TEAM_ID")
  private Long id;

  private String name;

  @OneToMany(mappedBy = "team")
  private List<Member> members = new ArrayList<>();

  public void addMember(Member member) {
    this.members.add(member);
    if (member.getTeam() != this) {
      member.setTeam(this);
    }
  }

  public List<Member> getMembers() {
    return members;
  }
}
```

```java
@Entity
public class Member {

  @Id
  @GeneratedValue
  @Column(name = "MEMBER_ID")
  private Long id;

  private String  username;

  @ManyToOne
  @JoinColumn(name = "TEAM_ID")
  private Team team;

  public void setTeam(Team team) {
    this.team = team;

    //무한 루프에 빠지지 않도록 체크
    if (!team.getMembers().contains(this)) {
      team.getMembers().add(this);
    }
  }

  public Team getTeam() {
    return team;
  }
}
```

- 양방향은 외래 키가 있는 쪽이 연관관계의 주인이다
  - 일대다와 다대일 연관관계는 항상 `다`쪽에 외래 키가 있다.
  - `Member.team`이 연관관계의 주인이다.
  - `JPA`는 외래 키를 관리할 때 연관관계의 주인만 사용한다.
  - 주인이 아닌 쪽은 조회를 위한 `JPQL`이나 객체 그래프 탐색 때 사용한다.
- 양방향 연관관계는 항상 서로를 참조해야 한다.
  - 양방향 연관관계는 항상 서로 참조해야 한다.
  - 양방향 연관관계에서는 연관관계 편의 메서드를 작성하는 것이 좋다.
    - 양방향 연관관계를 양쪽 모두 작성하게 되면 무한루프에 빠지게 되므로 무한 루프에 빠지지 않도록 검사하는 로직이 필요하다.


### 일대다

일대다 관계는 다대일 관계의 **반대 방향이다.**

엔티티를 하나 이상 참조할 수 있으므로 자바 컬렉션인

`Collection`, `List`, `Set`, `List` 중 하나를 사용한다.

### 일대다 단방향[1:N]

일대다 단방향 관계는 반대쪽 테이블에 있는 외래 키를 관리하게 된다.

일대다 단방향으로 매핑한 팀과 회원을 살펴본다.

```java
@Entity
public class Member {
  
  @Id @GeneratedValue
  @Column(name = "MEMBER_ID")
  private Long id;
  
  private String username;
}
```

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
```

일대다 단방향 관계를 매핑할 때는 `@JoinColumn`을 명시해야 한다.

그렇지 않으면 JPA는 조인 테이블 전략을 기본으로 사용해 매핑하게 된다. (조인 테이블 전략은 7.4절에서)

**일대다 단뱡향 매핑의 단점**

일대다 단방향 매핑은 매핑한 객체가 관리하는 외래 키가 다른 테이블에 있다는 점이다.

본인 테이블에 외래 키가 있으면 저장과 연관관계 처리를 `INSERT SQL` 한 번으로 끝낼 수 있지만

다른 테이블에 외래 키가 있으면 연관관계 처리를 위한 `UPDATE SQL`을 추가로 실행해야 한다.

```java
public void testSave() {
  Member member1 = new Member("1")
  Member member2 = new Member("1")

  Team team1 = new Team("team1");
  team1.getMembers().add(member1);
  team1.getMembers().add(member2);

  em.persist(member1);
  em.persist(member2);
  em.persist(team1);

  transaction.commit();
}
```

`Member`엔티티를 저장할 때는 `Member`테이블의 외래 키에 아무 값도 저장되지 않는다.

대신 `Team` 엔티티를 저장할 대 `Team.members`의 참조 값을 확인해 회원 테이블의 외래 키를 업데이트 한다.

이러한 점은 성능 문제도 있지만 관리도 부담스럽다.

때문에 **일대다 단방향 보다는 다대일 양방향 매핑**을 권장한다.

### 일대다 양방향[1:N, N:1]

일대다 양방향 매핑은 **존재하지 않는다.**

대신 다대일 양방향 매핑을 사용해야 한다.

정확히는 양방향 매핑에서 `@OneToMany`는 연관관계의 주인이 될 수 없다.

때문에 `@ManyToOne`에는 `mappedBy` **속성이 없다.**

일대다 양방향 매핑은 일대다 단방향 매핑 반대편에 같은 외래 키를 사용하는 `다대일` 단방향 매핑을 **읽기 전용**으로 하나 추가하면 된다.

일대다 양방향 관계를 살펴본다.

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
```

```java
@Entity
public class Member {

  @Id @GeneratedValue
  @Column(name = "MEMBER_ID")
  private Long id;

  private String username;
  
  @ManyToOne
  @JoinColumn(name = "TEAM_ID", insertable = false, updatable = false)
  private Team team;
}
```

일대다 단방향 매핑 반대편에 다대일 단방향 매핑을 추가했다.

서로 같은 외래 키를 관리하므로 문제가 생길 수 있기 때문에

반대편인 다대일 쪽은 `insertable = false`, `updatable = false` 으로 설정해 읽기만 가능하게 한다.

이 방법은 일대다 양방향 매핑이라기 보단 단방향 매핑 반대편에

다대일 단방향 매핑을 **읽기 전용**으로 추가해 일대다 양방향처럼 보이도록 하는 방법이다.

일대다 단방향 매핑이 가지는 단점을 그대로 가지기 때문에 **될 수 있으면 다대일 양방향 매핑을 사용하자.**

### 일대일[1:1]

일대일 관계는 양쪽이 서로 하나의 관계만 가진다.

일대일 관계는 다음과 같은 특징이 있다.

- 일대일 관계는 그 반대도 일대일 관계다.
- 테이블 관계에서는 항상 `다`쪽이 외래 키를 가진다.
  - 일대일 관게는 주 테이블이나 대상 테이블 둘 중 어느 곳이나 외래 키를 가질 수 있다.

### 주 테이블에 외래 키

주 객체가 대상 객체를 참조하는 것처럼 주 테이블에 외래 키를 두고 대상 테이블을 참조한다. 객체지향 개발자들이 선호한다.

주 테이블만 확인해도 대상 테이블과 연관관계가 있는지 알 수 있다는 장점이 있다.

#### 단방향

회원과 사물함의 일대일 단방향 관계를 살펴본다.

```java
@Entity
public class Locker {
  @Id
  @GeneratedValue
  @Column(name = "LOCKER_ID")
  private Long id;
  
  private String name;
}

```

```java
@Entity
public class Member {
  @Id @GeneratedValue
  @Column(name = "MEMBER_ID")
  private Long id;

  private String username;

  @OneToOne
  @JoinColumn(name = "LOCKER_ID")
  private Locker locker;
}
```

일대일 관계이기 때문에 `@OneToOne`을 사용한다.

이 관계는 다대일 단방향 `@ManyToOne`과 거의 비슷하다.

#### 양방향

```java
@Entity
public class Locker {
  @Id
  @GeneratedValue
  @Column(name = "LOCKER_ID")
  private Long id;
  
  @OneToOne(mappedBy = "locker")
  private Member member;
  
  private String name;
}

```

```java
@Entity
public class Member {
  @Id @GeneratedValue
  @Column(name = "MEMBER_ID")
  private Long id;

  private String username;

  @OneToOne
  @JoinColumn(name = "LOCKER_ID")
  private Locker locker;
}
```

양방향이기 때문에 `mappedBy`를 사용해 주인을 정했다.




### 대상 테이블에 외래 키

데이터베이스 개발자들이 선호한다. 테입르 관계를 일대일에서 일대다로 변경할 때 테이블 구조를 그대로 유지할 수 있다는 장점이 있다.

일대일 관계 중 대상 테이블에 외래 키가 있는 `단방향`관계는 JPA에서 지원하지 **않는다.**

비슷하게 할 수 있는 방법이 존재하지만 이런 방법을 사용할 정도면 설계를 다시 고민해봐야 한다.

#### 양방향

대상 테이블에 외래 키가 있는 양방향 관계를 알아본다.

```java
@Entity
public class Locker {
  @Id
  @GeneratedValue
  @Column(name = "LOCKER_ID")
  private Long id;

  @OneToOne
  @JoinColumn(name = "MEMBER_ID")
  private Member member;

  private String name;
}

```

```java
@Entity
public class Member {
  @Id @GeneratedValue
  @Column(name = "MEMBER_ID")
  private Long id;

  private String username;

  @OneToOne(mappedBy = "member")
  private Locker locker;
}
```

프록시를 사용할 때 외래 키를 직접 관리하지 않는 일대일 관계는 **지연 로딩으로 설정해도 즉시 로딩된다.**

위의 코드에서 `Member.locker`는 즉시 로딩된다.

이는 프록시의 한계 떄문에 발생하는 문제인데 프록시 대신 `bytecode instrumentation`을 사용하면 해결할 수 있다.

### 다대다[N:N]

관계형 데이터베이스는 정구화된 테이블 2개로 다대다 관계를 표현할 수 **없다.**

다대다 관계를 일대다, 다대일 관계로 풀어내는 연결 테이블을 사용한다.

객체는 객체 2개로 다대다 관계를 만들 수 있다.

`@ManyToMany`를 사용하면 다대다 관계를 편리하게 매핑할 수 있다.

### 다대다: 단방향

다대다 단방향 관계인 회원과 상품 엔티티를 살펴본다.

```java
@Entity
public class Member {
  @Id @Column(name = "MEMBER_ID")
  private String id;
  
  private String username;
  
  @ManyToMany
  @JoinTable(name = "MEMBER_PRODUCT", 
      joinColumns = @JoinColumn(name = "MEMBER_ID"), 
      inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID")
  )
  private List<Product> products = new ArrayList<Product>();
  
}
```

```java
@Entity
public class Product {
  @Id @Column(name = "PRODUCT_ID")
  private String id;

  private String name;
}
```

회원 엔티티와 상품 엔티티를 `@ManyToMany`로 매핑했다.

`@JoinTable` 을 사용해 연결 테이블을 바로 매핑했기 때문에 `회원_상품` 엔티티 없이 매핑을 완료할 수 있다.

`@JoinTable`의 속성은 다음과 같다.

- `@JoinTable.name`: 연결 테이블을 지정한다.
- `@JoinTable.joinColumns`: 현재 방향인 회원과 매핑할 조인 컬럼 정보를 지정한다.
- `@JoinTable.inverseJoinColumns`: 반대 방향인 상품과 매핑할 조인 컬럼 정보를 지정한다.

`MEMBER_PRODUCT`테이블은 대다대 관계를 `일대다`, `다대일` 관계로 풀어내기 위해 필요한 연결 테이블일 뿐이다.

실행된 SQL을 살펴보면 `MEMBER_PRODUCT`와 상품 테이블을 조인해서 연관된 상품을 조회한다.

### 다대다: 양방향

다대다 매핑이므로 역방향도 `@ManyToMany`를 사용한다.

그리고 양쪽 중 원하는 곳에 `mappedBy`로 연관관계의 주인을 지정한다.

```java
@Entity
public class Product {
  @Id @Column(name = "PRODUCT_ID")
  private String id;

  
  @ManyToMany(mappedBy = "products")
  private List<Member> members;
  
  private String name;
}
```

양방향 연관관계는 연관관계 편의 메서드를 추가해 관리하는 것이 편하다.

### 다대다: 매핑의 한계와 극복, 연결 엔티티 사용

`@ManyToMany`를 사용하면 연결 테이블을 자동으로 처리해주므로 도메인 모델이 단순해지고 여러가지로 편리하다.

하지만 실무에 사용하기에는 한계가 있는데 연결 테이블에 다른 컬럼을 추가하게 되면 `@ManyToMany`를 사용할 수 없기 때문이다.

결국 연결 테이블을 매핑하는 엔티티를 만들고 엔티티의 관계도 `다대다`에서 `일대다`, `다대일` 관계로 풀어야 한다.

```java
public class MemberProductId implements Serializable {
  private String member;
  private String product;

  @Override
  public boolean equals(final Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    final MemberProductId that = (MemberProductId) o;
    return Objects.equals(member, that.member) && Objects.equals(product, that.product);
  }

  @Override
  public int hashCode() {
    return Objects.hash(member, product);
  }
}
```

```java
@Entity
@IdClass(MemberProductId.class)
public class MemberProduct {

  @Id
  @ManyToOne
  @JoinColumn(name = "MEMBER_ID")
  private Member member;

  @Id
  @ManyToOne
  @JoinColumn(name = "PRODUCT_ID")
  private Product product;
}
```

```java
@Entity
public class Member {
  @Id @Column(name = "MEMBER_ID")
  private String id;

  private String username;

  @OneToMany(mappedBy = "member")
  private List<MemberProduct> memberProducts;

}
```

```java
@Entity
public class Product {
  @Id @Column(name = "PRODUCT_ID")
  private String id;
}
```

회원과 회원상품을 양방향 관계로 만들었다.

상품 엔티티에서 회원상품 엔티티로 객체 그래프 탐색 기능이 필요하지 않다고 판단해 연관관계를 만들지 않았다.

회원상품 엔티티는 기본 키를 매핑하는 `@Id`와 외래 키를 매핑하는 `@JoinColumn`을 동시해 사용해 `기본 키 + 외래 키`를 한번에 매핑했다.

`@IdClass`를 사용해 복합 기본 키를 매핑했다.

### 복합 기본 키

JPA에서는 복합 키를 사용하기 위해 별도의 식별자 클래스를 만들어야 한다.

그리고 엔티티에 `@IdClass`를 사용해 식별자 클래스를 지정한다.

복합 키를 위한 식별자 클래스는 다음과 같은 특징이 있다.

- 복합 키는 별도의 식별자 클래스로 만들어야 한다.
- `Serializable`을 구현해야 한다.
- `equals와 hashCode`메서드를 구현해야 한다.
- 기본 생성자가 있어야 한다.
- 식별자 클래스는 `public`이어야 한다.
- `@IdClass`외에 `@EmbeddedId`를 사용하는 방법도 있다.

### 식별 관계

부모 테이블의 기본 키를 받아 자신의 기본 키 + 외래 키로 사용하는 것을 **식별 관계**라 한다.

회원 상품 엔티티는 데이터베이스에 저장될 때 연관된 회원과 상품의 식별자를 가져와 자신의 기본 키 값으로 사용한다.

복합 키는 항상 식별자 클래스를 만들어야 한다.

### 다대다: 새로운 기본 키 사용

기본 키 생성 전략은 데이터베이스에서 자동으로 생성해주는 대리 키를 `Long`값으로 사용하는 것을 추천한다.

이 방법이 ORM 매핑 시에 복합 키를 만들지 않아도 되므로 간단히 매핑을 완성할 수 있다.

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
}
```

```java
@Entity
public class Member {
  @Id @Column(name = "MEMBER_ID")
  private String id;

  private String username;

  @OneToMany(mappedBy = "member")
  private List<Order> orders = new ArrayList<>(); 
}
```

```java
@Entity
public class Member {
  @Id @Column(name = "MEMBER_ID")
  private String id;

  private String username;

  @OneToMany(mappedBy = "member")
  private List<Order> orders = new ArrayList<>(); 
}
```

```java
@Entity
public class Product {
  @Id @Column(name = "PRODUCT_ID")
  private String id;
  private String name;
}

```

대리 키를 사용함으로써 식별 관계에 복합 키를 사용하는 것보다 매핑이 단순하고 이해하기 쉬워졌다.

이렇게 새로운 기본 키를 사용해 다대다 관계를 풀어내는 것도 좋은 방법이다.

### 다대다 연관관계 정리

다대다 관계를 일대다 다대일 관계로 풀어내기 위해 연결 테이블을 만들 때 식별자를 어떻게 구성할지 선택해야 한다.

- 식별 관계: 받아온 식별자를 기본 키 + 외래 키로 사용한다.
- 비식별 관계: 받아온 식별자는 외래 키로만 사용하고 새로운 식별자를 추가한다.

데이터베이스 설계에서는 기본 키 + 외래 키로 사용하는 것을 식별 관계라 하고, 외래 키로만 사용하는 것을 비식별 관계라고 한다.

객체 입장에서는 비식별 관계를 사용하는 것이 단순하고 편리하게 ORM 매핑을 할 수 있다.

이러한 이유로 식별 관계보다는 비식별 관계를 추천한다.

---

## 7장 - 고급 매핑

---

### 상속 관계 매핑

관계형 데이터베이스에는 객체지향 언어에서 다루는 상속이라는 개념이 없다.

대신 `슈퍼타입 서브타입`관계라는 모델링 기법이 가장 유사하다.

ORM에서 말하는 상속 관계 매핑은 객체의 상속 구조와 데이터베이스의 슈퍼타입 서브타입 관계를 매핑하는 것이다.

`슈퍼타입 서브타입`논리 모델을 구현할 때는 3가지 방법을 선택할 수 있다.

- 각각의 테이블로 변환: 각각을 모두 테이블로 만들고 조회할 때 조인을 사용한다.(JPA 에서는 조인 전략)
- 통합 테이블로 변환: 테이블을 하나만 사용해서 통합한다. (JPA에서는 단일 테이블 전략)
- 서브타입 테이블로 변환: 서브 타입마다 하나의 테이블을 만든다. (JPA에서는 구현 클래스마다 테이블 전략)

### 조인 전략

조인 전략은 엔티티 각각을 모두 **테이블**로 만들고 자식 테이블이 부모 테이블의 기본 키를 받아서 `기본 키 + 외래 키`로 사용하는 전략이다.

조회할 떄 조인을 자주 사용한다.

객체는 타입으로 구분할 수 있지만 테이블은 타입의 개념이 없기 때문에 타입을 구분하는 컬럼을 추가해야 한다.

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {
  @Id @GeneratedValue
  @Column(name = "ITEM_ID")
  private Long id;

  private String name;
  private int price;
}

@Entity
@DiscriminatorValue("A")
public class Album extends Item{
  private String artist;
}

@Entity
@DiscriminatorValue("M")
public class Movie extends Item{
  private String director;
  private String actor;
}
```
- `@Inheritance(strategy = InheritanceType.JOINED)`: 상속 매핑은 부모 클래스에 `@Inheritance`를 사용해야 한다. 속성 값으로 매핑 전략을 지정한다.
- `@DiscriminatorColumn(name = "DTYPE")`: 부모 클래스에 구분 컬럼을 지정한다. 이 컬럼으로 지정된 자식 테이블을 구분한다.
- `@DiscriminatorValue("A")`: 엔티티를 저장할 때 구분 컬럼에 입력할 값을 지정한다.

기본 값으로 자식 테이블은 부모 테이블의 ID 컬럼명을 그대로 사용하는데 자식 테이블의 기본 키 컬럼명을 변경하고 싶으면 `@PrimaryKeyJoinColumn`을 사용하여 변경한다.

- 장점:
  - 테이블이 정규화된다.
  - 외래 키 참조 무결성 제약조건을 활용할 수 있다.
  - 저장공간을 효율적으로 사용한다.
- 단점:
  - 조회할 때 조인이 많이 사용되므로 성능이 저하될 수 있다.
  - 조회 쿼리가 복잡하다.
  - 데이터를 등록할 때 INSERT SQL을 두 번 실행한다.
- 특징:
  - 하이버네이트를 포함한 몇몇 구현체는 구분 컬럼 없이도 동작한다.


### 단일 테이블 전략

단일 테이블 전략은 테이블을 하나만 사용한다.

구분 컬럼으로 어떤 자식 데이터가 저장되었는지 구분한다.

조회할 때 조인을 사용하지 않으므로 일반적으로 가장 빠르다.

주의점은 자식 엔티티가 매핑한 컬럼 모두 `null`을 허용해야 한다.

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {
  @Id @GeneratedValue
  @Column(name = "ITEM_ID")
  private Long id;

  private String name;
  private int price;
}

@Entity
@DiscriminatorValue("M")
public class Movie extends Item {
  private String director;
  private String actor;
}

@Entity
@DiscriminatorValue("A")
public class Album extends Item{
  private String artist;
}
```

`@Inheritance(strategy = InheritanceType.SINGLE_TABLE)`로 설정하면 단일 테이블 전략을 사용한다.

- 장점:
  - 조인이 필요 없으므로 일반적으로 조회 성능이 빠르다.
  - 조회 쿼리가 단순하다.
- 단점:
  - 자식 엔티티가 매핑한 컬럼은 모두 `null`을 허용해야 한다.
  - 단일 테이블에 모든 것을 저장하기 때문에 테이블이 커질 수 있다. 상황에 따라서는 조회 성능이 오히려 느릴 수 있다.
- 특징:
  - 구분 컬럼을 꼭 사용해야 한다.
  - `@DiscriminatorValue`를 지정하지 않으면 기본으로 엔티티 이름을 사용한다.

### 구현 클래스마다 테이블 전략

자식 엔티티마다 테이블을 만드는 전략이다.

```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Item {
  @Id @GeneratedValue
  @Column(name = "ITEM_ID")
  private Long id;

  private String name;
  private int price;
}

@Entity
public class Movie extends Item {
  private String director;
  private String actor;
}

@Entity
public class Album extends Item{
  private String artist;
}
```

일반적으로 추천하지 않는 전략이다.

- 장점:
  - 서브 타입을 구분해서 처리할 때 효과적이다.
  - `not null` 제약조건을 사용할 수 있다.
- 단점:
  - 여러 자식 테이블을 함께 조회할 때 성능이 느리다.
  - 자식 테이블을 통합해서 쿼리하기 어렵다.
- 특징:
  - 구분 컬럼을 사용하지 않는다.

이 전략은 데이터베이스 설계자와 ORM 전문가 둘 다 추천하지 않으므로 조인이나 단일 테이블 전략을 고려하자.

### @MappedSuperclass

부모 클래스는 테이블과 매핑하지 **않고** 부모 클래스를 상속 받는 자식 클래스에게 매핑 정보만 제공하고 싶으면 `@MappedSuperclass`를 사용하면 된다.

`@MappedSuperclass`는 추상 클래스와 비슷한데 `@Entity`는 실제 테이블과 매핑되지만 `@MappedSuperclass`는 실제 테이블과 매핑되지 않는다.

단순히 매핑 정보를 상속할 목적으로만 사용된다.

```java
@MappedSuperclass
public abstract class BaseEntity {
  @Id @GeneratedValue
  private Long id;
  private String name;
}

@Entity
public class Member extends BaseEntity{
  
  private String email;
}
```

`BaseEntity`에는 객체들이 주로 사용하는 공통 매핑 정보를 정의한다.

부모로부터 물려받은 매핑 정보를 재정의하려면 `@AttributeOverrides`나 `@AttributeOverride`를 사용하면 된다.

연관관계를 재정의하려면 `@AssociationOverrides`나 `@AssociationOverride`를 사용하면 된다.

`@MappedSuperclass` 특징은 다음과 같다.

- 테이블과 매핑되지 않고 자식 클래스에 엔티티 매핑 정보를 상속하기 위해 사용한다.
- `@MappedSuperclass`로 지정한 클래스는 엔티티가 아니다.
  - `em.find()`나 `JPQL`에서 사용할 수 없다.
- 이 클래스를 직접 생성해 사용할 일은 거의 없으므로 **추상 클래스**로 만드는 걸 권장한다.

엔티티는 엔티티이거나 `@MappedSuperclass`로 지정한 클래스만 상속받을 수 있다.

### 복합 키와 식별 관계 매핑

데이터베이스 테이블 사이에 관계는 외래 키가 기본 키에 포함되는지 여부에 따라 **식별 관계**와 **비식별 관계**로 구분한다.

### 식별 관계

식별 관계는 부모 테이블의 기본 키를 내려받아서 자식 테이블의 기본 키 + 외래 키로 사용하는 관계이다.

### 비식별 관계

비식별 관계는 부모 테이블의 기본 키를 받아서 자식 테이블의 외래 키로만 사용하는 관계이다.

비식별 관계는 외래 키에 `NULL`을 허용하지는지에 따라 필수적 비식별 관계와 선택적 비식별 관계로 나눈다.

- 필수적 비식별 관계 : 외래 키에 `NULL`을 허용하지 않는다. 연관관계를 필수적으로 맺어야 한다.
- 선택적 비식별 관계 : 외래 키에 `NULL`을 허용한다. 연관관계를 맺을지 말지 선택할 수 있다.

최근에는 비식별 관계를 주로 사용하고 꼭 필요한 곳에만 식별 관계를 사용하는 추세다.

### 복합 키: 비식별 관계 매핑

둘 이상의 컬럼으로 구성된 복합 기본 키는 별도의 식별자 클래스를 만들어야 한다.

```java
@Entity
public class Hello {
  @Id 
  private String id1;

  @Id
  private String id2;
}
```

JPA는 식별자를 구분하기 위해 `equals`와 `hashCode`를 사용해서 동등성 비교를 한다.

JPA는 복합키를 지원하기 위해 `@IdClass`와 `EmbeddedId` 2가지 방법을 제공한다.

`@IdClass`

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
```

```java
public class ParentId implements Serializable {
  private String id;
  private String id2;

  @Override
  public boolean equals(final Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;

    final ParentId parentId = (ParentId) o;

    if (!Objects.equals(id, parentId.id)) return false;
    return Objects.equals(id2, parentId.id2);
  }

  @Override
  public int hashCode() {
    int result = id != null ? id.hashCode() : 0;
    result = 31 * result + (id2 != null ? id2.hashCode() : 0);
    return result;
  }
}
```

`@IdClass`를 사용해 `ParentId`클래스를 식별자 클래스로 지정하였다.


`@IdClass`를 사용할 때 식별자 클래스는 다음 조건을 만족해야 한다.

- 식별자 클래스의 속성명과 엔티티에서 사용하는 식별자의 속성명이 같아야 한다.
  - Parent.id1과 ParentId.id1이 같다.
- `Serializable`인터페이스를 구현해야 한다.
- `equals`,  `hashCode`를 구현해야 한다.
- 기본 생성자가 있어야 한다.
- public 이어야 한다.

복합키로 조회하는 예제 코드를 살펴본다.

```java
@Entity
public class Child {
  
  @Id
  private String id;
  
  @ManyToOne
  @JoinColumns({
          @JoinColumn(name = "PARENT_ID1", referencedColumnName = "PARENT_ID1"),
          @JoinColumn(name = "PARENT_ID2", referencedColumnName = "PARENT_ID2"),
  })
  private Parent parent;
}
```

부모 테이블의 기본 키 컬럼이 복합 키이므로 자식 테이블의 외래 키도 복합 키다.

`@EmbeddedId`

`@EmbeddedId`는 좀 더 객체지향적인 방법이다.

```java
@Entity
public class Parent {
  
  @EmbeddedId
  private ParentId id;

  private String name;
}
```

```java
@Embeddable
public class ParentId implements Serializable {
  
  @Column(name = "PARENT_ID1")
  private String id;

  @Column(name = "PARENT_ID2")
  private String id2;

  @Override
  public boolean equals(final Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;

    final ParentId parentId = (ParentId) o;

    if (!Objects.equals(id, parentId.id)) return false;
    return Objects.equals(id2, parentId.id2);
  }

  @Override
  public int hashCode() {
    int result = id != null ? id.hashCode() : 0;
    result = 31 * result + (id2 != null ? id2.hashCode() : 0);
    return result;
  }
}
```

식별자 클래스에는 `@Embeddable`을 붙인다.

`@EmbeddedId`를 적용한 식별자 클래스는 식별자 클래스에 기본 키를 직접 매핑한다.

`@EmbeddedId`를 적용한 식별자 클래스는 다음 조건을 만족해야 한다.

- `@Embeddable` 어노테이션을 붙여주어야 한다.
- `Serializable`인터페이스를 구현해야 한다.
- `equals`,  `hashCode`를 구현해야 한다.
- 기본 생성자가 있어야 한다.
- public 이어야 한다.


### 복합 키와 equals(), hashCode()

복합 키는 `equals`,  `hashCode`를 필수로 구현해야 한다.

영속성 컨텍스트는 엔티티의 식별자를 키로 사용해 엔티티를 관리한다.

식별자를 비교할 때 equals(), hashCode()를 사용하기 때문에 식별자 객체의 동등성이 지켜지지 않으면 예상과 다른 엔티티가 조회되거나 엔티티를 찾을 수 없는 등 영속성 컨텍스트가 엔티티를 관리하는데 심각한 문제가 발생한다.

식별자 클래스는 보통 equals(), hashCode() 구현할 떄 **모든 필드를 사용한다.**

### @IdClass VS @EmbeddedId

`@IdClass` 와 `@EmbeddedId`는 각각의 장단점이 있으므로 본인의 취향에 맞는 것을 일관성 있게 사용하면 된다.

`@EmbeddedId` 가 더 객체지향적이고 중복도 없어서 좋아보이지만 특정 상황에 `JPQL`이 조금 더 길어질 수 있다.

### 복합 키 : 식별 관계 매핑

부모, 자식, 손자까지 계속 기본 키를 전달하는 식별 관계다.

자식 테이블은 부모 테이블의 기본 키를 포함해서 복합 키를 구성해야 하므로 `@IdClass` 와 `@EmbeddedId`를 사용해 식별자를 매핑해야 한다.

### `@IdClass`와 식별 관계

```java
@Entity
public class Parent {
  
  @Id @Column(name = "PARENT_ID")
  private String id;
  private String name;
}
```

```java
@Entity
@IdClass(ChildId.class)
public class Child {
  @Id
  @ManyToOne
  @JoinColumn(name = "PARENT_ID")
  public Parent parent;

  @Id
  @Column(name = "CHILD_ID")
  private String childId;
}

```

```java
public class ChildId implements Serializable {
  private String parent;
  private String childId;


  @Override
  public boolean equals(final Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;

    final ChildId childId1 = (ChildId) o;

    if (!Objects.equals(parent, childId1.parent)) return false;
    return Objects.equals(childId, childId1.childId);
  }

  @Override
  public int hashCode() {
    int result = parent != null ? parent.hashCode() : 0;
    result = 31 * result + (childId != null ? childId.hashCode() : 0);
    return result;
  }
}
```


```java
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
}
```


```java
public class GrandChildId implements Serializable {
  private ChildId child;
  private String id;

  @Override
  public boolean equals(final Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;

    final GrandChildId that = (GrandChildId) o;

    if (!Objects.equals(child, that.child)) return false;
    return Objects.equals(id, that.id);
  }

  @Override
  public int hashCode() {
    int result = child != null ? child.hashCode() : 0;
    result = 31 * result + (id != null ? id.hashCode() : 0);
    return result;
  }
}
```

### @EmbeddedId와 식별 관계

`@EmbeddedId`로 식별 관계를 구성할 때는 `@MapsId`를 사용해야 한다.

```java
@Entity
public class Parent {

  @Id @Column(name = "PARENT_ID")
  private String id;
  private String name;
}
```

```java
@Entity
public class Parent {

  @Id @Column(name = "PARENT_ID")
  private String id;
  private String name;
}
```

```java
@Entity
@IdClass(ChildId.class)
public class Child {
  
  @EmbeddedId
  public ChildId id;
  
  @MapsId("parentId")
  @ManyToOne
  @JoinColumn(name = "PARENT_ID")
  public Parent parent;

  @Id
  @Column(name = "CHILD_ID")
  private String childId;
}
```

```java
@Embeddable
public class ChildId implements Serializable {
  private String parentId;
  private String childId;

  // equals, hashCode 구현 생략

}
```

```java

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
  public Child child;
}
```

```java
@Embeddable
public class GrandChildId implements Serializable {
  private ChildId child;

  @Column(name = "GRANDCHILD_ID")
  private String id;


  @Override
  public boolean equals(final Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;

    final GrandChildId that = (GrandChildId) o;

    if (!Objects.equals(child, that.child)) return false;
    return Objects.equals(id, that.id);
  }

  @Override
  public int hashCode() {
    int result = child != null ? child.hashCode() : 0;
    result = 31 * result + (id != null ? id.hashCode() : 0);
    return result;
  }
}
```

`@MapsId`는 외래 키와 매핑한 연관관계를 기본 키에도 매핑하겠다는 뜻이다.


### 비식별 관계로 구현

그 동안의 예제를 비식별 관계로 구현하는 코드이다.

```java
@Entity
public class Parent {

  @Id @GeneratedValue
  @Column(name = "PARENT_ID")
  private Long id;
  private String name;
}
```
```java
@Entity
public class Child {

  @Id @GeneratedValue
  @Column(name = "CHILD_ID")
  private Long id;
  private String name;

  @ManyToOne
  @JoinColumn(name = "PARENT_ID")
  private Parent parent;
}
```
```java
@Entity
public class GrandChild {

  @Id @GeneratedValue
  @Column(name = "GRANDCHILD_ID")
  private Long id;
  private String name;

  @ManyToOne
  @JoinColumn(name = "CHILD_ID")
  private Child child;
}
```

복합 키를 사용한 코드보다 매핑도 쉽고 코드도 단순하다.

또한 복합 키가 없으므로 복합 키 클래스를 만들지 않아도 된다.

### 일대일 식별 관계

일대일 식별 관계는 자식 테이블의 기본 키 값으로 부모 테이블의 기본 키 값만 사용한다.

부모 테이블의 기본 키가 복합 키가 아니면 자식 테이블의 기본 키는 복합 키로 구성하지 않아도 된다.

```java
@Entity
public class BoardDetail {
  @Id
  private Long boardId;

  @MapsId
  @OneToOne
  @JoinColumn(name = "BOARD_ID")
  private Board board;
}
```

식별자가 단순히 컬럼 하나면 `@MapsId`를 사용하고 속성 값은 비워두면 된다.

### 식별, 비식별 단계의 장단점

- 식별 관계는 부모 테이블의 기본 키를 자식 테이블로 전파하면서 자식 테이블의 기본 키 컬럼이 점점 늘어난다.

- 식별 관계는 2개 이상의 컬럼을 합해서 복합 기본 키를 만들어야 하는 경우가 많다.

- 식별 관계를 사용할 때 기본 키로 비즈니스 의미가 있는 자연 키 컬럼을 조합하는 경우가 많다.

- 비식별 관계의 기본 키는 비즈니스와 전혀 관계없는 대리 키를 주로 사용한다.

- 식별 관계는 부모 테이블의 기본 키를 자식 테이블의 기본 키로 사용하므로 비식별 관계보다 테이블 구조가 유연하지 못하다.

객체 관계 매핑의 관점에서 다음과 같은 이유로 비식별 관계를 선호한다.

- 일대일 관계를 제외하고 식별 관계는 2개 이상의 컬럼을 묶은 복합 기본 키를 사용한다.
- 비식별 관계의 기본 키는 주로 대리 키를 사용한다.

식별 관계가 가지는 장점도 있는데 다음과 같다.

- 기본 키 인덱스를 활용하기 좋다.
- 특정 상황에 조인 없이 하위 테이블만으로 검색을 할 수 있다.

`ORM` 신규 프로젝트 진행시에는 될 수 있으면 **비식별 관계를 사용하고 기본 키는 Long 타입의 대리 키를 사용**하는 것을 추천한다.

선택적 비식별 관계보다는 필수적 비식별 관계를 사용하는 것이 내부 조인만 사용해도 되기 때문에 좋다.

### 조인 테이블

데이터베이스 테이블의 연관관계를 설계하는 방법은 크게 2가지다.

- 조인 컬럼 사용
- 조인 테이블 사용

조인 테이블은 주로 다대다 관계를 일대다, 다대일 관계로 풀어내기 위해 사용한다.

### 일대일 조인 테이블

일대일 관계를 만들려면 조인 테이블의 외래 키 컬럼 각각에 유니크 제약조건을 걸어야 한다.

```java
@Entity
public class Parent {

  @Id
  @GeneratedValue
  private Long id;

  @OneToOne
  @JoinTable(name = "PARENT_CHILD",
          joinColumns = @JoinColumn(name = "PARNET_ID"),
          inverseJoinColumns = @JoinColumn(name = "CHILD_ID")
  )
  private Child child;
}
```

```java
@Entity
public class Child {

  @Id
  @GeneratedValue
  @Column(name = "CHILD_ID")
  private Long id;

}
```

`@JoinTable`을 사용한다.

속성은 다음과 같다.

- name: 매핑할 조인 테이블 이름
- joinColumns: 현재 엔티티를 참조하는 외래 키
- inverseJoinColumns: 반대방향 엔티티를 참조하는 외래 키

### 일대다 조인 테이블

일대다 관계를 만들려면 조인 테이블의 컬럼 중 다(N)와 관련된 컬럼에 유니크 조건을 걸어야 한다.


### 다대일 조인 테이블

다대일은 일대다에서 방향만 반대이다.

### 다대다 조인 테이블

다대다 관계를 만들려면 종니 테이블의 두 컬럼을 합해서 하나의 복합 유니크 제약조건을 걸어야 한다.

```java
@Entity
public class Parent {

  @Id
  @GeneratedValue
  private Long id;

  @ManyToMany
  @JoinTable(name = "PARENT_CHILD",
          joinColumns = @JoinColumn(name = "PARNET_ID"),
          inverseJoinColumns = @JoinColumn(name = "CHILD_ID")
  )
  private List<Child> child = new ArrayList<Child>;
  
}
```

```java
@Entity
public class Child {

  @Id
  @GeneratedValue
  @Column(name = "CHILD_ID")
  private Long id;

```

### 엔티티 하나에 여러 테이블 매핑

잘 사용하지는 않지만 `@SecondaryTable`을 사용하면 한 엔티티에 여러 테이블을 매핑할 수 있다.

이 방법은 항상 두 테이블을 조회하므로 최적화하기 어렵다.


