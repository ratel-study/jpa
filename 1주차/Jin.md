# Jin
JPA 스터디

## 3장
entity의 상태
![image](https://github.com/ratel-study/jpa/assets/31182783/2e804316-4fca-4297-a127-9f1df2f8c09b)

- 비영속: 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태
- 영속: 영속성 컨텍스트에 관리되는 상태
- 준영속: 영속성 컨텍스트에 저장되었다가 분리된 상태
- 삭제: 삭제된 상태

hibernate
![image](https://github.com/ratel-study/jpa/assets/31182783/8c181faf-3f62-4902-a1bd-cddb9b0e6315)

jpa의 장점
- 1차 캐시
- 동일성 보장
- 트랜잭션을 지원하는 쓰기 지연
- 변경 감지
- 지연 로딩

1차 캐시
- db를 조회하기 전에 entityManager 속 1차 캐시를 한 번 더 조회한다.

동일성 보장
- 1차 캐시로 반복 가능한 읽기 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공

플러시
- 영속성 컨텍스트의 변경내용을 데이터베이스에 반영
- ex: 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송
- 플러시는 영속성 컨텍스트를 비우지 않는다.


# 4장
@Entity
- 기본 생성자는 필수
- final 클래스, enum, interface, inner 클래스에는 사용할 수 없다.
- 저장할 필드에 final을 사용하면 안 된다.

@Table
- name: 테이블 이름

hibernate.hbm2ddl.auto
- create
- create-drop
- update
- validate
- none

@Column
- name
- insertable, updatable

@Temporal, @Lob, @Transient

@GeneratedValue(starategy = GenerationType.AUTO)
@Id
- identity
- sequence
- table
- auto

---
