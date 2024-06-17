# Finn
JPA 스터디

---

## 8장 - 프록시와 연관관계 관리



### 프록시

엔티티를 조회할 때 연관된 엔티티들이 항상 사용되는 것은 아니다.

```java
@Entity
public class Member {

  @Id
  private Long id;

  private String username;

  @ManyToOne
  private Team team;
}
```

```java
@Entity
public class Team {
  @Id
  private Long id;
  
  public String name;
}
```

```java
public void printUserAndTeam(String memberId) {
    Member member = em.find(Member.class, memberId);
    Team team = member.getTeam();
    System.out.println("회원 이름 : " + member.getUsername());
    System.out.println("소속 팀 : " + team.getName());
}
```

위의 `printUserAndTeam` 메서드는 회원 엔티티를 찾아 팀의 이름도 출력한다.

이 때는 회원 엔티티를 조회할 때 팀 엔티티까지 함께 조회하는 게 효율적이다.

반면에 회원 엔티티만 사용할 때는 팀 엔티티까지 DB에서 조회하는 것은 효율적이지 못하다.

`JPA`는 이런 문제를 해결하기 위해 **엔티티가 실제 사용될 때까지 데이터베이스 조회를 지연하는 방법을 제공**한다.

이것을 **지연 로딩**이라고 한다.

지연 로딩은 JPA 표준 명세에서 JPA 구현체에게 위임했기 때문에 앞으로의 내용은 `하이버네이트` 구현체에 대한 내용이다.


### 프록시 기초

JPA에서 식별자로 엔티티 하나를 조회할 때 `EntityManager.find()`를 사용한다.

이때 영속성 컨텍스트에 엔티티가 없으면 데이터베이스에서 조회한다.

엔티티를 실제 사용하는 시점까지 데이터베이스 조회를 미루고 싶으면

`EntityManager.getReference()`메서드를 사용하면 된다.

이 메서드를 호출할 때 데이터베이스를 조회하지 않고 실제 엔티티 객체도 생성하지 않는 대신 데이터베이스 접근을 위한 **프록시 객체**를 반환한다.

- 프록시의 특징

프록시 클래스는 실제 클래스를 상속 받아서 만들어지므로 실제 클래스와 겉 모양이 같다.

사용하는 입장에서는 진짜 객체인지 프록시 객체인지 구분하지 않아도 된다.

프록시 객체의 메서드를 호출하면 프록시 객체는 실제 객체의 메서드를 호출한다.

- 프록시 객체의 초기화

프록시 객체는 실제 사용될 때 데이터베이스를 조회해 실제 엔티티 객체를 생성한다.

이를 **프록시 객체의 초기화**라고 한다.

프록시 객체의 초기화 과정은 다음과 같다.

1. 프록시 객체에 엔티티를 사용하는 메서드를 호출한다.
2. 프록시 객체에서 실제 데이터를 조회한다.
3. 실제 엔티티가 생성되어 있지 않으면 영속성 컨텍스트에 실제 엔티티를 생성을 요청한다.
  1. 이를 초기화라 한다.
4. 영속성 컨텍스트는 데이터 베이스를 조회해 실제 엔티티 객체를 생성한다.
5. 프록시 객체는 생성된 실제 엔티티의 객체의 참조를 target 멤버변수에 보관한다.
6. 프록시 객체는 실제 엔티티 객체의 메서드를 호출해 결과를 반환한다.

- 프록시의 특징

프록시의 특징은 다음과 같다.

- 프록시 객체는 처음 사용할 때 한 번만 초기화된다.
- 프록시 객체를 초기화한다고 프록시 객체가 실제 엔티티로 바뀌는 것은 아니다. 프록시 객체가 초기화되면 프록시 객체를 통해 실제 엔티티에 접근할 수 있다.
- 프록시 객체는 원본 엔티티를 상속받은 객체이므로 타입 체크 시 주의해야 한다.
- 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 데이터베이스를 조뢰할 필요가 없기 때문에 프록시가 아닌 실제 엔티티를 반환한다.
- 초기화는 영속성 컨텍스트의 도움을 받아야 가능하다.
  - 준영속 상태의 프록시를 초기화하면 에러가 발생한다.

### 프록시와 식별자

엔티티를 프록시로 조회할 때 식별자 값을 파라미터로 전달하는데 프록시 객체는 이 식별자 값을 보관한다.

프록시 객체는 식별자 값을 가지고 있으므로 식별자 값을 조회하는 `getId()`를 호출해도 프록시를 초기화하지 않는다.

단, 엔티티 접근 방식을 프로퍼티로 설정한 경우에만 초기화하지 않는다.

프록시는 연관관계를 설정할 때 유용하게 사용할 수 있다.

연관관계를 설정할 때는 식별자 값만 사용하므로 프록시를 사용하면 데이터베이스 접근 횟수를 줄일 수 있다.

연관관계를 설정할 때 엔티티 접근 방식을 필드로 설정해도 프록시를 초기화하지 않는다.

### 프록시 확인

JPA가 제공하는 `PersistenceUnitUtil.isLoaded(Object entity)` 메서드를 사용하면 프록시 인스턴스의 초기화 여부를 확인할 수 있다.

아직 초기화되지 않은 프록시 인스턴스는 `false`를 반환한다.

이미 초기화되었거나 프록시 인스턴스가 아니면 `true`를 반환한다.

### 즉시 로딩과 지연 로딩

프록시 객체는 주로 연관된 엔티티를 지연 로딩할 때 사용한다.

JPA는 개발자가 연관된 엔티티의 조회 시점을 선택할 수 있도록 다음 두 가지 방법을 제공한다.

- 즉시 로딩: 엔티티를 조회할 때 연관된 엔티티도 함께 조회한다.
  - 설정 방법: `@ManyToOne(fetch = FetchType.EAGER)`
- 지연 로딩: 연관된 엔티티를 실제 사용할 때 조회한다.
  - 설정 방법: `@ManyToOne(fetch = FetchType.LAZY)`

### 즉시 로딩

즉시 로딩을 사용하려면 `@ManyToOne`의 `fetch`속성을 `FetchType.EAGER`로 지정한다.

```java
@Entity
public class Member {

  @Id
  private Long id;

  private String username;

  @ManyToOne(fetch = FetchType.EAGER)
  @JoinColumn(name = "TEAM_ID")
  private Team team;
}
```

회원을 조회하는 순간 팀도 함께 조회한다.

이 때 쿼리를 2번 실행할 것 같지만 대부분의 JPA 구현체는

**즉시 로딩을 최적화하기 위해 가능하면 조인 쿼리를 사용한다.**

회원 테이블에 `TEAM_ID`외래 키는 `NULL`값을 허용하고 있기 때문에 팀에 소속되지 않은 회원이 있을 가능성이 있다.

팀에 소석하지 않은 회원과 팀을 내부 조인하면 팀은 물론이고 회원 데이터도 조회할 수 없다.

JPA는 이런 상황을 고려해서 외부 조인을 사용한다.

외부 조인보다 내부 조인이 성능 최적화에 유리하다.

외래 키에 `NOT NULL`제약 조건을 설정하면 값이 있는 것을 보장하기 때문에 내부 조인을 사용해도 된다.

`@JoinColumn`에 `nullable = false`를 설정해서 해당 외래 키는 `NULL`값을 허용하지 않는다고 JPA에게 알려주어야한다.

이렇게하면 JPA는 외부 조인 대신 내부 조인을 사용한다.

**nullable 설정에 따른 조인 전략**
- `@JoinColumn(nullable = true)`: `NULL`허용, 외부 조인 사용
- `@JoinColumn(nullable = false)`: `NULL`허용하지 않음, 내부 조인 사용

`@ManyToOne(optional = false)` 처럼 `optional` 옵션을 `false`로 설정해도 내부 조인을 사용한다.

JPA는 선택적 관계면 외부 조인을 필수 관계면 내부 조인을 사용한다.

### 지연 로딩

지연 로딩을 사용하려면 `@ManyToOne`의 `fetch` 속성을 `FetchType.LAZY`로 지정한다.

```java
@Entity
public class Member {

  @Id
  private Long id;

  private String username;

  @ManyToOne(fetch = FetchType.LAZY)
  @JoinColumn(name = "TEAM_ID")
  private Team team;
}

```

회원과 팀을 지연 로딩으로 설정했다.

회원을 조회하면 회원만 조회하고 팀은 조회하지 않는다.

대신 조회한 회원의 `team` 멤버변수에 프록시 객체를 넣어둔다.

이 프록시 객체는 실제 사용될 때까지 데이터 로딩을 미룬다 실제 데이터가 필요한 순간이 되어서야 데이터베이스를 조회해서 프록시 객체를 초기화한다.

team 엔티티가 영속성 컨텍스트에 이미 로딩되어 있다면 프록시가 아닌 실제 엔티티를 반환하게 된다.

### 즉시 로딩, 지연 로딩 정리

연관된 엔티티에 대한 즉시로딩, 지연로딩은 상황에 따라 어떤 것이 좋은지가 다르다.

- 지연 로딩: 연관된 엔티티를 프록시로 조회한다. 프록시를 실제 사용할 때 초기화하면서 데이터베이스를 조회한다.
- 즉시 로딩: 연관된 엔티티를 즉시 조회한다. 하이버네이트는 가능하면 SQL 조인을 사용해서 한 번에 조회한다.

### 프록시와 컬렉션 래퍼

하이버네이트는 엔티티를 영속 상태로 만들 때 엔티티에 컬렉션이 있으면 컬렉션을 추적하고 관리할 목적으로 원본 컬렉션을 하이버네이트가 제공하는 내장 컬렉션으로 변경한다.

이것을 컬렉션 래퍼라 한다.

엔티티를 지연 로딩하면서 프록시 객체를 사용해 지연 로딩을 수행하지만 컬렉션은 컬렉션 래퍼가 지연 로딩을 처리해준다.

### JPA 기본 페치 전략

`fetch` 속성의 기본 설정값은 다음과 같다.

- `@ManyToOne`, `@OneToOne`: 즉시 로딩(fetchType.EAGER)
- `@OneToMany`, `@ManyToMany`: 지연 로딩(fetchType.LAZY)

JPA의 기본 페치 전략은 연관된 엔티티가 하나면 `즉시 로딩`을 컬렉션이면 `지연 로딩`을 사용한다.

컬렉션을 로딩하는 것은 비용이 많이 들고 너무 많은 데이터를 로딩할 수 있기 때문이다.

**추천하는 방법은 모든 연관관계에 지연 로딩을 사용하는 것이다.**

애플리케이션 개발이 어느 정도 완료단계에 왔을 때 실제 사용하는 상황을 보고 꼭 필요한 곳에만 즉시 로딩을 사용하도록 최적화하면 된다.

### 컬렉션에 FetchType.EAGER 사용 시 주의 점

컬렉션에 `FetchType.EAGER`를 사용할 경우에 주의할 점은 다음과 같다.

- 컬렉션을 하나 이상 즉시 로딩하는 것은 권장하지 않는다.
  - 서로 다른 컬렉션을 2개 이상 조인할 때 SQL 실행 결과가 N * M이 되서 너무 많은 데이터를 반환할 수 있고 애플리케이션 성능이 저하될 수 있다.
  - JPA는 이렇게 조회된 결과를 메모리에서 필터링해 반환한다.
  - 2개 이상의 컬렉션을 즉시 로딩으로 설정하는 것은 권장하지 않는다.
- 컬렉션 즉시 로딩은 항상 외부 조인을 사용한다.
  - JPA는 일대다 관계를 즉시 로딩할 때 항상 외부 조인을 사용한다.

### 영속성 전이: CASCADE

특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶으면 영속성 전이 기능을 사용하면 된다.

JPA는 `CASCADE` 옵션으로 영속성 전이를 제공한다.

영속성 전이를 사용하면 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장할 수 있다.

JPA에서 엔티티를 저장할 때 연관된 모든 엔티티는 영속 상태여야 한다.

### 영속성 전이: 저장

영속성 전이를 활성화하는 옵션을 적용한다.

```java
@Entity
public class Parent {
  // 생략
  
  @OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST)
  private List<Child> children = new ArrayList<>();
}
```

부모만 영속화하면 `CascadeType.PERSIST` 로 설정한 자식 엔티티까지 함께 영속화해서 저장한다.

### 영속성 전이: 삭제

`CascadeType.REMOVE`로 설정하면 부모 엔티티만 삭제하면 연관된 자식 엔티티도 함께 삭제된다.

```java
Parent findParent = em.find(Parent.class, 1L);
em.remove(findParent);
```
위의 코드를 실행하면 `DELETE` SQL을 실행한다.

부모는 물론 연관된 자식도 모두 삭제한다.

삭제 순서는 외래 키 제약조건을 고려해 자식을 먼저 삭제하고 부모를 삭제한다.

`CascadeType.REMOVE`를 설정하지 않으면 부모 엔티티만 삭제하게 되므로 외래 키 제약 조건으로 데이터베이스에서 외래 키 무결성 예외가 발생한다.

### 고아 객체

JPA는 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제하는 기능을 제공한다.

**부모 엔티티의 컬렉션에서 자식 엔티티의 참조만 제거하면 자식 엔티티가 자동으로 삭제된다.**

```java
@Entity
public class Parent {
  // 생략
  
  @OneToMany(mappedBy = "parent", orphanRemoval = true)
  private List<Child> children = new ArrayList<>();
}
```

`orphanRemoval = true`를 설정해 고아 객체 제거 기능을 활성화한다.

고아 객체 제거는 **참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제하는 기능**이다.

이 기능은 참조하는 곳이 **하나**일 때만 사용해야 한다.

특정 엔티티가 개인 소유하는 엔티티에만 이 기능을 적용해야 하기 때문에 `orphanRemoval`은 `@OneToOne`, `@OneToMany`에만 사용할 수 있다.

고아 객체 제거는 부모를 제거하면 자식 객체는 고아가 되기 때문에 부모를 제거하면 자식 객체도 같이 제거된다.

### 영속성 전이 + 고아 객체, 생명 주기

`CascadeType.ALL` + `orphanRemoval = true`를 동시에 사용하면 부모 엔티티를 통해 자식의 생명주기를 관리할 수 있다.

---

## 9장 값 타입


엔티티 타입은 식별자를 통해 지속해서 추적할 수 있지만 값 타입은 식별자가 없고 숫자나 문자같은 속성만 있으므로 추적할 수 없다.

값 타입은 다음 3가지로 나눈다.

- 기본값 타입
  - 자바 기본 타입
  - 래퍼 클래스
- 임베디드 타입
  - JPA에서 사용자가 직접 정의한 값
- 컬렉션 값 타입

### 기본값 타입

```java
@Entity
public class Member {

  @Id
  private Long id;

  private String username;
}
```

`Member`엔티티의 `username`이 값 타입이다.

식별자도 없고 생명주기도 엔티티에 의존한다.

### 임베디드 타입 (복합 값 타입)

새로운 값 타입을 정의해서 사용할 수 있다.

JPA에서는 **임베디드 타입**이라고 한다.

```java
@Entity
public class Member {
  @Id
  private Long id;
  private String username;

  @Embedded
  Period workPeriod;
}
```

```java
public class Period {

    @Temporal(TemporalType.DATE)
    Date startDate;

    @Temporal(TemporalType.DATE)
    Date endDate;

    public boolean isWork(Date date) {
        // ....
    }
}
```

임베디드 타입을 사용하면 엔티티가 더욱 의미있고 응집력 있게 변한다.

임베디드 타입을 사용하려면 다음 2가지 어노테이션이 필요하다.

둘 중 하나는 생략해도 된다.

- `@Embeddable`: 값 타입을 정의하는 곳에 표시
- `@Embedded`: 값 타입을 사용하는 곳에 표시

임베디드 타입은 엔티티의 생명주기에 의존하므로 컴포지션 관계가 된다.

**임베디드 타입은 값이 속한 엔티티의 테이블에 매팽된다.**

### 임베디드 타입과 연관관계

임베디드 타입은 값 타입을 포함하거나 엔티티를 참조할 수 잇다.

```java
@Entity
public class Member {
  @Id
  private Long id;
  private String username;

  @Embedded
  PhoneNumber phoneNumber;
}
```

```java
@Embeddable
public class PhoneNumber {
    String areaCode;
    String localNumber;
    @ManyToOne 
    PhoneServiceProvider provider;
}

```

### @AttributeOverride: 속성 재정의

임베디드 타입에 정의한 매핑정보를 재정의하려면 엔티티에 `@AttributeOverride` 를 사용하면 된다.

예를들어 회원에게 주소가 하나 더 필요할 때

```java
@Entity
public class Member {
  @Id
  private Long id;
  private String username;

  @Embedded
  Address homeAddress;

  @Embedded
  Address companyAddress;
}
```

이렇게 작성하면 테이블에 매핑할 때 컬럼명이 중복되게 된다.

```java
@Entity
public class Member {
  @Id
  private Long id;
  private String username;

  @Embedded
  Address homeAddress;

  @Embedded
  @AttributeOverrides({
    @AttributeOverride(name = "city", column = @Column(name = "COMPANY_CITY"))
  })
  Address companyAddress;
}

```

위의 코드처럼 `@AttributeOverrides`를 사용해 매핑할 컬럼명을 지정해주면 된다.

### 임베디드 타입과 null

임베디드 타입이 null이면 매핑한 컬럼 값은 모두 null이 된다.

```java
member.setAddress(null);
em.persist(member);
```

### 값 타입과 불변 객체

값 타입은 단순하고 안전하게 다룰 수 있어야 한다.

임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 위험하다.

```java
member1.setAddress(new Address("주소"));
Address address = member1.getAddress();

address.setCity("주소2");
member2.setAddress(address);
```

위의 코드대로 되면 회원 2의 주소만 변경되는 것이 아닌 **회원1의 주소도 같이 변경하게 된다.**

이렇게 뭔가를 수정했는데 예상치 못한 곳에서 문제가 발생하는 걸 **부작용**이라고 한다.

이런 부작용을 막으려면 **값을 복사해서 사용해야한다.**

```java
Address a = new Address("A");
Address b = a.clone();

b.setCity(b);
```
이렇게 매번 복사를 하면 **공유 참조**를 피할 수 있지만 **원본 참조 값을 직접 넘기는 것을 막을 방법이 없다**는 게 문제다.

객체의 공유 참조는 피할 수 없다. 따라서 **근본적으로 객체의 값을 수정하지 못하게 막으면 된다.**

### 불변 객체

객체를 불변하게 만들면 값을 수정할 수 없으므로 **부작용**을 원천 차단할 수 있다.

값 타입은 될 수 있으면 **불변 객체**로 설계해야 한다.

참조 값을 공유해도 인스턴스의 값을 수정할 수 없기때문에 부작용이 발생하지 않는다.

### 값 타입의 비교

자바가 제공하는 객체 비교는 2가지다.

- 동일성 비교: 인스턴스의 참조 값을 비교 `==`사용
- 동등성 비교: 인스턴스의 값을 비교 `equals()` 사용

값 타입을 비교할 때는 `equals()`를 사용하므로 `equals()`를 재정의해야 한다.

재정의할 때는 보통 모든 필드의 값을 비교하도록 구현한다.

`equals()` 재정의할 때는 `hashCode()` 메서드도 재정의하는 것이 안전하다.


### 값 타입 컬렉션

값 타입을 하나 이상 저장하려면 컬렉션에 보관하고 `@ElementCollection`, `@CollectionTable` 어노테이션을 사용하면 된다.

```java
@Entity
public class Member {
  @Id
  private Long id;
  private String username;

  @Embedded
  Address homeAddress;

  @ElementCollection
  @CollectionTable(name = "FAVORITE_FOODS", 
    joinColumns = @JoinColumn(name = "MEMBER_ID")
  )
  @Column(name = "FOOD_NAME")
  private set<String> favoriteFoods = new HashSet<String>();
}
```

관계형 데이터베이스의 테이블은 컬럼 안에 컬렉션을 포함할 수 없기 때문에 별도의 테이블을 추가하고 `@CollectionTable` 를 사용해서 추가한 테이블을 매핑해야 한다.

### 값 타입 컬렉션 사용

```java
Member member = new Member();

// 임베디드 값 타입
member.setHomeAddress(new Address("address"));

// 기본 값 타입 컬렉션
member.getFavoriteFoods().add("짬뽕");
member.getFavoriteFoods().add("짜장");

// 임베디드 값 타입 컬렉션
member.getAddressHistory().add(new Address("address1"));

em.persist(member);
```

JPA는 영속화할 때 값 타입도 함께 저장한다.

- member: INSERT SQL 1번
- member.homeAddress: 컬렉션이 아닌 임베디드 값 타입이므로 회원 테이블을 저장하는 SQL에 포함된다.
- member.favoriteFoods: INSERT SQL 3번
- member.addressHistory: INSERT SQL 2번

따라서 `em.persist`한번의 호출로 총 6번의 `ISNERT SQL`이 실행된다.

값 타입 컬렉션은 영속성 전이 + 고아 객체 제거 기능을 필수로 가진다고 볼 수 있다.

값 타입 컬렉션도 조회할 때 페치 전략을 선택할 수 있는데 `LAZY`가 기본이다.

값 타입 컬렉션을 수정할 때는

- 임베디드 값 타입 수정: 임베디드 값 타입은 매핑한 엔티티의 테이블만 UPDATE한다.
- 기본값 타입 컬렉션 수정: 컬렉션 내의 원소를 제거하고 새로운 원소를 추가해야 한다.
- 임베디드 값 타입 컬렉션 수정: 값 타입은 불변해야 하기 때문에 기존 컬렉션에서 원소를 제거하고 새로 추가해야 한다.

### 값 타입 컬렉션의 제약사항

엔티티는 식별자가 있으므로 값을 변경해도 식별자로 데이터베이스에 저장된 원본 데이터를 찾아 변경할 수 있다.

하지만 값 타입은 **식별자가 없기** 때문에 데이터베이스에 저장된 원본 데이터를 찾기는 어렵다.

특정 엔티티 하나에 소속된 값 타입은 소속된 엔티티를 찾아 값을 변경하면 되지만 컬렉션에 보관된 값들은 **별도의 테이블에 보관**된다.

컬렉션의 값 타입이 변경되면 데이터베이스에서 원본 데이터를 찾기 어렵다.

JPA 구현체들은 값 타입 컬렉션에 변경 사항이 발생하면, 값 타입 컬렉션이 매핑된 테이블의 모든 데이터를 **삭제**하고 현재 컬렉션에 있는 값 타입들의 모든 값을 데이터베이스에 **다시 저장한다.**

따라서 값 타입 컬렉션이 매핑된 테이블에 데이터가 많다면 값 타입 컬렉션 대신 **일대다 관계**를 고려해야 한다.

값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본 키를 구성해야하기 때문에 `null`을 입력할 수 없고, 중복해서 저장할 수 없다는 제약이 있다.

값 타입 컬렉션 대신 새로운 엔티티를 만들어 **일대다 관계**로 설정한 뒤 **영속성 전이 + 고아 객체 제거** 기능을 적용하면 값 타입 컬렉션처럼 사용할 수 있다.

---

## 10장 - JPQL



JPQL 의 특징은 다음과 같다.

- 객체지향 쿼리 언어이다. 테이블을 대상으로 쿼리를 하는게 아닌 엔티티 객체를 대상으로 쿼리한다.
- SQL을 추상화해서 특정 데이터 베이스 SQL에 의존하지 않는다.
- 결국 SQL로 변환된다.

### 기본 문법과 쿼리

JPQL도 `select`, `update`, `delete` 문을 사용할 수 있다.

엔티티 저장시에는 `persist()`를 사용하면 되므로 `insert`문은 없다.

### SELECT 문

```jpql
select m from Member as m where m.username = 'Hello'
```

- 엔티티와 속성은 대소문자를 **구분**한다.
  - select, from, as 같은 키워드는 대소문자를 구분하지 않는다.
- JPQL에서 사용한 `Member`는 클래스 명이 아니라 **엔티티 명**이다.
- JPQL에서 별칭을 필수로 사용한다. 별칭 없이 작성하면 잘못된 문법이라는 오류가 발생한다.

### TypeQuery, Query

작성한 JPQL을 실행하려면 쿼리 객체를 만들어야 한다.

쿼리 객체는 `TypeQuery`와 `Query` 가 있는데

반환할 타입을 명확히 지정할 수 있으면 `TypeQuery`, 그렇지 않으면 `Query` 객체를 사용하면 된다.

```java
TypedQuery<Member> query = em.createQuery("select m from Member m", Member.class);

List<Member> resultList = query.getResultList();
```

```java
Query query = em.createQuery("select m.name, m.age from Member m");

List resultList = query.getResultList();

for (Object o : resultList) {
    Object[] result = (Object[]) o;
    System.out.println(result[0]);
    System.out.println(result[1]);
}
```

`SELECT`절에서 여러 엔티티나 컬럼을 선택할 때는 `Query` 객체를 사용해야 한다.

타입을 변환할 필요가 없는 `TypeQuery`가 더 편리하다.

### 결과 조회

다음 메서드들을 호출하면 실제 쿼리를 실행해서 데이터베이스를 조회한다.

- `query.getResultList()`: 결과를 반환한다. 결과가 없으면 빈 컬렉션을 반환한다.
- `query.getSingleResult()`: 결과가 정확히 하나일 때 사용한다.
  - 결과가 없으면 예외가 발생한다.
  - 결과가 1개보다 많으면 예외가 발생한다.

### 파라미터 바인딩

JPQL은 이름 기준 파라미터 바인딩도 지원한다.

```java
String nameParam = "name1";
TypedQuery<Member> query = em.createQuery("select m from Member m where m.name = :name", Member.class);
query.setParameter("name", nameParam);

List<Member> resultList = query.getResultList();
```
이름 기준 파라미터는 앞에 `:`를 사용한다.


```java
String nameParam = "name1";
TypedQuery<Member> query = em.createQuery("select m from Member m where m.name = ?1", Member.class);
query.setParameter(1, nameParam);

List<Member> resultList = query.getResultList();
```

위치 기준 파라미터를 사용하려면 `?:` 다음에 위치 값을 주면 된다.

위치 값은 1부터 시작이다.

직접 문자열을 더해 JPQL을 작성하는 것은 SQL 인젝션 공격을 당할 수 있다.

또한 파라미터 바인딩을 사용하면 SQL로 파싱한 결과를 재사용할 수 있고, 데이터베이스 내부에서도 파싱한 값을 재사용하기 때문에

애플리케이션과 데이터베이스 모두 성능이 향상된다.

때문에 파라미터 바인딩은 **선택이 아닌 필수다**

### 프로젝션

SELECT절에 조회할 대상을 지정하는 것을 **프로젝션**이라 한다.

프로젝션 대상은 엔티티, 엠비디드 타입, 스칼라 타입이 있다.

- 엔티티 프로젝션
  - 조회한 엔티티는 영속성 컨텍스트에서 관리된다.
- 임베디드 타입 프로젝션
  - 임베디드 타입은 조회의 시작점이 될 수 없다는 제약이 있다.
  - 임베디드 타입은 엔티티 타입이 아닌 값 타입이기 때문에 영속성 컨텍스트에서 관리되지 않는다.
- 스칼라 타입 프로젝션
  - 숫자, 문자, 날짜와 같은 기본 데이터 타입을 스칼라 타입이라 한다.

### 여러 값 조회

꼭 필요한 데이터들만 선택해서 조회해야할 때도 있는데 프로젝션에 여러 값을 선택하면

`TypeQuery`를 사용할 수 없고 `Query`를 사용해야 한다.

스칼라 타입뿐만 아니라 엔티티 타입도 여러 값을 함께 조회할 수 있다.

이때 조회한 엔티티는 영속성 컨텍스트에서 관리한다.

### NEW 명령어

실제 여러 값을 조회할 때 `Object[]`를 사용하는 것보다 `DTO`를 만들어 의미 있는 객체로 변환해서 사용한다.

```java
public class UserDto {
    private String name;
    private int age;

    public UserDto() {
    }

    public UserDto(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

```java
TypedQuery<UserDto> query = em.createQuery("select new learn.jpa.model.UserDto(m.name, m.age) from Member m", UserDto.class);

List<UserDto> resultList = query.getResultList();
```

`NEW`명령어를 사용한 클래스로 `TypedQuery`를 사용할 수 있기 때문에 객체 변환 작업을 줄일 수 있다.

`NEW` 명령어를 사용할 때는 다음 2가지를 주의해야 한다.

- 패키지 명을 초함한 전체 클래스 명을 입력해야 한다.
- 순서와 타입이 일치하는 생성자가 필요하다.

### 페이징 API

JPA는 페이징을 다음 두 API로 추상화했다.

- `setFirstResult(int startPosition)`: 조회 시작 위치 (0부터 시작)
- `setMaxResults(int maxResult)`: 조회할 데이터 수

```java
TypedQuery<UserDto> query = em.createQuery("select new learn.jpa.model.UserDto(m.name, m.age) from Member m", UserDto.class);
query.setFirstResult(1);
query.setMaxResults(1);

List<UserDto> resultList = query.getResultList();
```

JPA에서는 데이터 베이스 방언을 사용해 데이터 베이스마다 다른 페이징처리를 API로 처리할 수 있다.

### 집합과 정렬

집합 함수 사용시 참고사항

- `null`값은 무시하므로 통계에 잡히지 않는다.
- 값이 없는데 `SUM`, `AVG` 같은 함수를 사용하면 `null`값이 된다.
- `DISTINCT`를 집합 함수 안에 사용해 중복된 값을 젝저하고 나서 집합을 구할 수 있다.
- `DISTINCT`를 `count`에서 사용할 때 임베디드 타입은 지원하지 않는다.

### group by, HAVING

`GROUP BY`는 통계 데이터를 구할 때 특정 그룹끼리 묶어준다.

`HAVING`은 `GROUP BY`와 함께 사용하는데 `GROUP BY`로 그룹화한 통계 데이터를 기준으로 필터링한다.

통계 쿼리는 보통 전체 데이터를 기준으로 처리하므로 실시간으로 사용하기엔 부담이 많기 때문에 결과가 많다면 결과만 저장하는 테이블을 별도로 만들고 사용자가 적은 시간에 통계 쿼리를 실행해 그 결과를 보관하는 것이 좋다.

### JPQL 조인

JPQL도 조인을 지원하는데 SQL 조인과 기능은 같고 문법만 약간 다르다.

#### 내부 조인

내부 조인은 `INNER JOIN` 을 사용한다. `INNER`는 생략할 수 있다.

```java
SELECT m
FORM Member m INNER JOIN m.team t
WHERE t.name = :teamName
```

JPQL 조인의 가장 큰 특징은 연관 필드를 사용한다는 것이다.

`m.team`이 연관 필드이다.

연관 필드는 다른 엔티티와 연관관계를 가지기 위해 사용하는 필드를 말한다.

JPQL은 JOIN 명령어 다음에 조인할 객체의 연관 필드를 사용한다.

#### 외부 조인

JPQL의 외부 조인은 다음과 같이 사용한다.

```java
SELECT m
FROM Member m LEFT JOIN m.team t
```

외부 조인은 기능상 SQL의 외부 조인과 같다. OUTER는 생략 가능하다.

#### 컬렉션 조인

일대다 관계나 다대다 관계처럼 컬렉션을 사용하는 곳에 조인하는 것을 **컬렉션 조인**이라 한다.

#### 세타 조인

`WHERE`절을 사용해 세타 조인을 할 수 있다.

**세타 조인은 내부 조인만 지원한다.**

```java
SELECT count(m)
FROM Member m, Team t
WHERE m.username = t.name
```

### JOIN ON 절

JPA 2.1부터 조인할 때 `ON`절을 지원한다.

`ON`절을 사용하면 조인 대상을 필터링하고 조인할 수 있다.

보통 `ON`절은 외부 조인에서만 사용한다.

모든 회원을 조회하면서 회원관 연관된 팀도 조회할 때 팀 이름이 A인 팀만 조회하는 쿼리이다.

```java
SELECT m, t FROM Member m
LEFT JOIN m.team t on t.name = 'A'
```

### 페치 조인

페치 조인은 JPQL에서 성능 최적화를 위해 제공하는 기능이다.

연관된 엔티티나 컬렉션을 한 번에 같이 조회하는 기능이다.

### 엔티티 페치 조인

페치 조인을 사용해서 회원 엔티티를 조회하면서 연관된 팀 엔티티도 함께 조회하는 JPQL이다.

```java
SELECT m
FROM Member m JOIN FETCH m.team
```

`join fetch`를 사용하면 연관된 엔티티나 컬렉션을 **함께 조회한다.**

**페치 조인은 별칭을 사용할 수 없다.**

페치 조인으로 조회한 연관된 엔티티는 **프록시가 아닌 실제 엔티티**이다.

따라서 지연 로딩이 일어나지 않는다.

또한 프록시가 아닌 실제 엔티티이므로 준영속 상태가 되어도 연관된 엔티티를 조회할 수 있다.

### 컬렉션 페치 조인

일대다 관계인 컬렉션을 페치 조인하는 JPQL이다.

```java
SELECT t
FROM Team t join fetch t.members
WHERE t.name = 'A'
```

이때 조인 결과 테이블을 보면 동일한 팀 엔티티가 여러개일 수 있다.

일대다 조인은 결과가 증가할 수 있지만 일대일, 다대일 조인은 결과가 증가하지 않는다.

### 페치 조인과 DISTINCT

JPQL의 `DISTINCT` 명령어는 SQL에 `DISTINCT`를 추가하고 애플리케이션에서 한 번 더 중복을 제거한다.

```java
SELECT distinct t
FROM Team t join fetch t.members
WHERE t.name = 'A'
```

`SELECT distinct t` 는 팀 엔티티의 중복을 제거하라는 것이다.

### 페치 조인과 일반 조인의 차이

JPQL은 결과를 반환할 때 연관관계까지 고려하지 않는다.

SELECT 절에서 지정한 엔티티만 조회할 뿐이다.

엔티티에 즉시 조인이 설정되었을 시 일반 조인시 프록시 객체나 아직 초기화하지 않은 컬렉션 래퍼를 반환한다.

반면 페치 조인을 사용하면 연관된 엔티티도 **함께 조회**하기 때문에 실제 엔티티가 반환된다.

### 페치 조인의 특징과 한계

페치 조인을 사용하면 SQL 한 번으로 연관된 엔티티들을 함께 조회할 수 있어 성능을 최적화할 수 있다.

엔티티에 직접 적용하는 로딩 전략은 애플리케이션 전체에 영향을 미치므로 **글로벌 로딩 전략**이라 부른다.

**페치 조인은 글로벌 로딩 전략보다 우선한다.**

페치 조인을 사용하면 일부는 빠를 수 있지만 전체로 보면 사용하지 않는 엔티티를 자주 로딩하므로 오히려 성능에 악영향을 미칠 수 있다.

글로벌 로딩 전략은 **지연 로딩**으로하고 최적화가 필요하면 페치 조인을 적용하는 것이 효과적이다.

페치 조인은 다음과 같은 한계가 있다.

- 페치 조인 대상에는 별칭을 줄 수 없다.
  - 따라서 SELECT, WHERE, 서브 쿼리에 페치 조인 대상을 사용할 수 없다.
- 둘 이상의 컬렉션을 페치할 수 없다.
- 컬렉션을 페치 조인하면 페이징 API를 사용할 수 없다.

### 경로 표현식

경로 표현식은 `.`을 찍어 객체 그래프를 탐색하는 것이다.

- 상태 필드: 단순히 값을 저장하기 위한 필드
- 연관 필드: 연관 관계를 위한 필드, 임베디드 타입 포함
  - 단일 값 연관 필드 : @ManyToOne, @OneToOne
  - 컬렉션 값 연관 필드: @OneToMany, @ManyToMany

상태 필드는 단순히 값을 저장하는 필드이고, 연관 필드는 객체 사이의 연관관계를 맺기 위해 사용하는 필드다.

### 경로 표현식과 특징

JPQL에서 경로 표현식을 사용하려면 각 경로에 따라 어떤 특징이 있는지 알아야 한다.

- 상태 필드 경로: 경로 탐색의 끝. 더는 탐색할 수 없다.
- 단일 값 연관 경로: 묵시적으로 내부 조인이 일어난다. 단일 값 연관 경로는 계속 탐색할 수 있다.
- 컬렉션 값 연관 경로: 묵시적으로 내부 조인이 일어난다. 더는 탐색할 수 없다. from 절에서 조인을 통해 별칭을 얻으면 별칭으로 탐색할 수 있다.

단일 값 연관 필드로 경로 탐색을 하면 SQL에서 내부 조인이 일어나는데 이를 **묵시적 조인**이라 한다.

### 경로 탐색을 사용한 묵시적 조인 시 주의사항

묵시적 조인이 발생했을 때 주의사항은 다음과 같다.

- 항상 내부 조인이다.
- 컬렉션에서 경로 탐색을 하려면 명시적으로 조인을 해서 별칭을 얻어야 한다.
- 묵시적 조인으로 인해 SQL의 FROM절에 영향을 준다.

묵시적 조인은 조인이 일어나는 상황을 파악하기 어렵기 때문에 묵시적 조인보다는 명시적 조인을 사용하자.

### 서브 쿼리

JPQL에서 서브 쿼리는 `SELECT`, `FROM`절에서는 사용할 수 없다.

서브 쿼리는 `EXISTS`, `ALL`, `ANY`, `SOME`, `IN` 과 함께 사용할 수 있다.

### 컬렉션 식

컬렉션 식은 컬렉션에만 사용하는 특별한 기능이다.

컬렉션은 컬렉션 식 이외에 다른 식은 사용할 수 없다.

`is null`처럼 컬렉션 식이 아닌 것은 사용할 수 없다.

- 빈 컬렉션 비교 식
  - `m.orders is not empty`
- 컬렉션의 멤버 식
  - 엔티티나 값이 컬렉션에 포함되어 있으면 참
  - `where :param member of t.members`

### 스칼라 식

스칼라는 숫자, 문자, 날짜, case, 엔티티 타입 같은 가장 기본적인 타입들을 말한다.

### 기타 정리

- enum은 `=` 비교 연산만 지원한다.
- 임베디드 타입은 비교를 지원하지 않는다.
- JPA 표준은 ``을 길이 0인 empty String으로 정했지만 데이터베이스마다 null로 사용하는 경우도 있으니 확인해야 한다.

### 엔티티 직접 사용

- 기본 키 값
  - JPQL에서 엔티티 객체를 직접 사용하면 SQL에서는 해당 엔티티의 기본 키 값을 사용한다.
- 외래 키 값

### Named 쿼리: 정적 쿼리

JPQL 쿼리는 크게 동적 쿼리와 정적 쿼리로 나눌 수 있다.

- 동적 쿼리: JPQL을 문자로 완성해 직접 넘기는 것
- 정적 쿼리: 미리 정의한 쿼리에 이름을 부여해 필요할 때 사용하는 것

Named 쿼리는 애플리케이션 로딩 시점에 JPQL 문법을 체크하고 미리 파싱해 둔다.

파싱된 결과를 재사용하므로 성능상 이점도 있다.

Named 쿼리는 `@NamedQuery`를 사용해 자바 코드나 XML 문서에 작성할 수 있다.

```java
@NamedQuery(
    name = "Member.findByName",
    query = "select m from Member m where m.name = :name
)
public class Member {

}
```

```java
TypedQuery<Member> query = em.createNamedQuery("Member.findByName", Member.class);

query.setParameter("name", "name1");

List<Member> resultList = query.getResultList();
```

`em.createNamedQuery()`를 사용해 Named 쿼리를 사용한다.

Named 쿼리는 영속성 유닛 단위로 관리되므로 충돌을 방지하기 위해 엔티티 이름을 앞에 주는 것이 좋다.

하나의 엔티티에 2개 이상의 Named 쿼리를 정의하려면 `@NamedQueries`를 사용하면 된다.

`@NamedQuery`에는 다음과 같은 속성이 있다.

- `lockMode`: 쿼리 실행 시 락을 건다.
- `hints` 2차 캐시를 다룰 때 사용한다.

만약 XML과 어노테이션에 같은 설정이 있을 경우 **XML**이 우선권을 가진다.
