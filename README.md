# 자바 ORM 표준 JPA 프로그래밍
- jdbc 쿼리문을 자바 코드에서 직접 작성해야함
- JPA 는 객체를 저장하고 조회하는 것처럼 jpa 이용해서 db 조회, 저장
- 개발 생산성, 유지보수가 쉬워짐

### 실무에서 중요한 것
- 객체와 테이블 설계를 잘하고 매핑하는 것이 중요.
- JPA 내부 동작 방식 이해

## JPA를 사용해야하는 이유
> 지금 시대는 객체를 관계형 DB에 관리해야함  

### SQL 중심 개발의 문제점
- 무한 반복, 지루한 코드
- *SQL에 의존적 개발을 피하기 힘듬*
- 패러다임의 불일치(객체 vs 관계형DB)
- 엔티티 신뢰 문제가 발생(DAO를 가져와도 뭐를 가져왔는지 확인해야함)
- 계층형 아키텍쳐에서 진정한 의미의 계층 분할이 힘듬.
- 객체지향적으로 모델링하려면 매핑 작업이 늘어남.

### 객체와 관계형 DB의 차이
1. 상속
- db에서 상속관계를 가진 테이블을 적용할 경우
- insert 하려면 2번해야하고, 테이블 조인해서 객체 생성하고, 저장해서 복잡하게 가져와야한다. 그래서 DB 저장할 객체는 상속관계 사용하지 않음
2. 연관관계
- 객체는 참조를 사용, 테이블은 외래 키를 사용
- 객체는 단방향으로 접근, 테이블은 양방향으로 접근가능
3. 데이터 타입
4. 데이터 식별 방법

## JPA
- java persistence API
- 자바 진영의 ORM 기술 표준
- 애플리케이션과 JDBC 사이에서 동작
### ORM
- object relational mapping
- 객체는 객체대로 설계, 관계형 데이터 베이스는 관계형 데이터 베이스대로 설계하고 중간에서 매핑

### JPA 동작
*저장*
1. DAO -> entity object persist -> JPA
2. Entity 분석
3. INSERT SQL 생성
4. JDBC API 사용
5. 패러다임 불일치 해결
*조회*
1. DAO -> find id -> JPA
2. SELECT SQL 생성
3. JDBC API 사용
4. Result Set 매핑
5. 패러다임 불일치 해결

- JPA는 인터페이스의 모음

### JPA 왜 사용해야할까?
1. 생산성
- CRUD가 쉬움 (특히 수정)
2. 유지보수
- 기존은 필드 변경시 모든 SQL 수정, JPA는 필드만 추가하면되고 SQL은 JPA가 해결
3. 패러다임의 불일치 해결
- 개발자는 객체지향적으로 개발하고, JPA가 알아서 해결해줌
4. 객체 그래프를 자유롭게 탐색할 수 있음.
5. 동일한 트랜잭션에서 조회한 엔티티 같음을 보장
6. JPA의 성능 최적화 기능
- 1차 캐시와 동일성 보장 (같은 트랜잭션에서 같은 엔티티 반환)
- 트랜잭션을 지원하는 쓰기 지연 (트랜잭션을 커밋할때까지 INSER SQL 모음, JDBC BATCH SQL 기능 사용해서 한번에 SQL 전송)
- 지연 로딩(지연로딩:객체가 실제사용될 떄 로딩, 즉시로딩:JOIN SQL로 한번에 연관된 객체까지 미리 조회)


## JPA 설정
- maven 프로젝트 사용할 경우 resources/META-INF/persistence.xml 필요
### 데이터베이스 방언
- JPA는 특정 DB 종속 X
- SQL 표준을 지키지 않는 특정 데이터베이스만의 고유한 기능 dialect 를 사용하면 됨.


## JPA 구동 방식
persistence class 설정 정보 조회 -> EntityManagerFactory class 만듬 -> 필요할때마다 entitymanager 빼서 씀

EntityManagerFactory는 프로그램 실행시점에 한번 만들고,
EntityManager를 트랜잭션 일어날때마다 만들어서 사용하면됨.

em으로 transaction을 선언하고,
begin()해서 시작하고, 할 일하고 commit or rollback 
마지막에 em.close()로 닫아줘야 transcation 끝남

> 사용할때는 java collections 사용하는 것 처럼 사용하면 됨.  

등록
``` 
Member member = new Member();
member.setId(1L);
member.setName("Hello");
em.persist(member);
```

조회
```
//1L -> PK
Member findMember = em.find(Member.class, 1L);
```

삭제
```
Member findMember = em.find(Member.class, 1L);
em.remove(findMember);
```

수정
```
Member findMember = em.find(Member.class, 1L);
findMember.setName("HelloJPA");
```
em.persist 같은거 특별히 안해도 됨.

* 엔티티 매니저 팩토리는 하나만 생성해서 애플리케이션 전체 공유
* 엔티티 매니저는 쓰레드 간에 공유 X (사용하고 버려야함)
* JPA의 모든 데이터 변경은 트랜잭션 안에서 실행

### JPQL
- 가장 단순한 조회 -> em.find()
- 조건이 있는 검색을 하려면 JPQL을 사용하면 됨.
- JPQL은 엔티티 객체를 대상으로 쿼리, SQL은 DB TABLE을 대상으로 쿼리


## 영속성 관리
### 영속성 콘텍스트 - 엔티티를 영구 저장하는 환경
Entity Manager Factory 에 의해 요청에 따라 Entity Manager 생성하고
Entity Manager 가 db 커넥션을 사용해서 DB를 사용.
- EntityManager.persist(entity);
- 영속성 콘텍스트는 논리적인 개념
- Entity Manager를 통해 영속성 콘텍스트에 접근

EntityManager -> 1:1로 영속성 콘텍스트 생성

### 엔티티의 생명주기
- 비영속 (new/transient) - 영속성 콘텍스트와 전혀 관계가 없는 새로운 상태
```
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");
```
- 영속 (managed) - 영속성 콘텍스트에 관리되는 상태
```
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");

EntityManager em = emf.createEntityManager();
em.getTransaction().begin();

em.persist(member);
```
- 준영속 (detached) - 영속성 콘텍스트에 저장되었다가 분리된 상태
```
//회원 엔티티를 영속성 콘텍스트에서 분리
em.detach(member)
```
- 삭제 (removed) - 삭제된 상태
```
//객체를 삭제한 상태
em.remove(member);
```

### 영속성 콘텍스트의 이점 - 영속성 콘텍스트를 통해 중간에 하나의 계층이 더 생겨서 여러 이점들을 누릴 수 있음.
- 1차 캐시
- 동일성 보장
1차 캐시로 반복 가능한 읽기 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공
- 트랜잭션을 지원하는 쓰기 지연
persist해도 Insert SQL DB에 보내지 않고, 쓰기지연SQL저장소에 저장함.
commit 하면 보냄
- 변경 감지
Java Collections 다루듯이 쓸수있음 변경후 persist 같은 거 안해도됨.
- 지연 로딩

em.find로 조회할 경우 영속성 콘텍스트의 1차캐시에서 먼저 조회하고,
없을 경우 DB에서 찾아서 1차캐시에 저장하고, 결과 반환.
애플리케이션 전체에서 교환하는 건아니고 트랜잭션이 끝나면 사라짐.


JPA는 tx commit 할 때 flush()가 호출 됨.
-> entity와 스냅샷(값을 읽어온 최초 시점을 저장해놓음)을 비교
-> 다르면 update

### 플러시
- 영속성 콘텍스트의 변경내용을 데이터베이스에 반영
- 플러시 발생시 - 변경감지, 수정된 엔티티 쓰기 지연 SQL 저장소 등록, 저장소 쿼리를 데이터 베이스 전송
- *영속성 콘텍스트를 비우지 않음*
- 트랜잭션이라는 작업 단위가 중요 -> 커밋 직전에만 동기화.
- 영속성 콘텍스트 플러시 방법
	- em.flush()  직접
	- 트랜잭션 커밋 - 자동
	- JPQL 쿼리 실행 - 자동
- 플러시 모드 옵션
```
em.setFlushMode(FlushModeType.COMMIT)
FlushModeType.AUTO - 커밋이나 쿼리 실행할 때 플러시 (기본값)
FlushModeType.COMMIT - 커밋할 때만 플러시
```

### 준영속
- 영속 상태의 엔티티가 영속성 콘텍스트에서 분리
- 영속성 콘텍스트 제공하는 기능 사용 못함.
*준영속 상태로 만드는 방법*
- em.detach(entity) - 특정 엔티티만 준영속 상태로 전환
- em.clear() - 영속성 콘텍스트 완전 초기화
- em.close() - 영속성 콘텍스트 종료


## 엔티티 매핑
- 객체와 테이블 매핑 : @Entity, @Table
- 필드와 컬럼 매핑 : @Column
- 기본 키 매핑 : @Id
- 연관관계 매핑 : @ManyToOne, @JoinColumn
(ex 멤버와 팀, 다대일, 다대다 등)

### 객체와 테이블 매핑
@Entity
- JPA가 관리하는 엔티티
- 기본생성자 필수(파라미터가 없는 public 또는 protected 생성자)
- final 클래스, enum, interface, inner 클래스 사용 X
- 저장할 필드에 final X
- 속성
	- name : 매핑할 테이블 이름
	- catalog : 데이터베이스 catalog 매핑
	- schema : 데이터베이스 shema 매핑
	- uniqueConstraints : DDL 생성시에 유니크 제약조건 생성

### 데이터베이스 스키마 자동 생성
- DDL을 애플리케이션 실행 시점에 자동 생성
- 테이블 중심 -> 객체 중심
- 데이터베이스 방언 활용 데이터베이스 맞는 적절한 DDL 생성
- 이렇게 생성한 DDL 은 *개발*에서만 사용
```  
hibernate.hbm2ddl.auto value=create 
```
- 옵션
	- create : 기존 테이블 삭제 후 생성
	- create-drop : create와 같으나 종료시점에 테이블 DROP
	- update : 변경분만 반영 (운영DB 사용 X)
	- validate : 엔티티와 테이블이 정상 매핑 되었는지만 확인
	- none : 사용하지 않음
- *운영에서는 절대 create, create-drop, update* 사용하면 안됨.
- DDL 생성 기능
	- DDL 생성 기능은 DDL을 자동 생성할 떄만 사용되고, JPA의 실행 로직에는 영향을 주지 않음.

### 필드와 컬럼 매핑
*@Column*
- name : 필드와 매핑할 테이블 컬럼 이름
- insertable, updatable : 등록, 변경 가능 여부
- nullable : null 값 허용 여부
- unique : 유니크 제약조건
- columnDefinition : 컬럼 정의 직접 ex) varchar(100)
- length : 문자 길이 제약조건, String에서만 사용
- precision, scale : BigDecimal 타입에서 사용
- enumerated - *EnumType*은 절대 ordinal 쓰면 안됨 -> String으로

*@Lob*
- 데이터베이스 BLOB, CLOB 타입과 매핑
- @Lob에는 지정할 수 있는 속성 없음.
- String 이면 CLOB, 나머지는 BLOB

*@Transient*
- 매핑 안하고 싶을 때

## 기본 키 매핑
- 직접 세팅할 때는 그냥 @ID 사용하면 됨.
- 자동으로 할당 할때는 @GeneratedValue
	- IDENTITY : 기본키 생성을 데이터 베이스에 위임 ex) mysql auto increment
				DB에 넣어봐야 알기때문에, em.persist를 호출하면 쿼리 날림.
	- SEQUENCE :  데이터베이스의 시퀀스 사용
				DB 시퀀스에서 값을 가져와야해서 em.persist 할 때 시퀀스에서
				값 가져옴. allocation size 옵션으로 성능 높일 수 있음.
	- TABLE : 테이블 따로 만들어서 사용. 모든 DB 적용가능한 장점이 있지만 성능이 안좋아짐.
	- 권장 : null 아님, 유일, 변하면 안됨.- *Long형 + 대체키 + 키 생성 전략 사용*


## 연관관계 매핑
### 객체를 테이블에 맞추어 데이터 중심으로 모델링하면 협력관계를 만들 수 없다.
- 테이블은 외래 키로 조인을 사용해서 연관된 테이블을 찾는다.
- 객체는 참조를 사용해서 연관된 객체를 찾는다.
- 테이블과 객체 사이에는 이런 큰 간격이 있다.

### 단방향 연관관계
```
@ManyToOne
@JoinColumn(name = "TEAM_ID")
private Team team;
```
를 통해 간단하게 다대일 매핑 할 수 있음.
매핑 후에는
```
Member findMember = em.find(Member.class, member.getId());

Team findTeam = findMember.getTeam();
```
등 편리하게 사용할 수 있음.

### 양방향 연관관계와 연관관계의 주인

연관관계의 주인과 mappedBy
- 객체 연관관계 = 2개
	- 회원 -> 팀 연관관계 1개 (단방향)
	- 팀 -> 회원 연관관계 1개 (단방향)
- 테이블 연관관계 = 1개
	- 회원 <-> 팀의 연관관계 1개 (양방향)

*객체의 양방향 관계*
- 객체의 양방향 관계는 사실 양방향 관계가 아니라 서로 다른 단방향 관계 2개이다.
- 객체를 양방향으로 참조하려면 단방향 연관관계를 2개 만들어야 한다.

*테이블의 양방향 연관관계*
- 테이블은 외래키 하나로 두 테이블의 연관관계를 관리
- MEMBER.TEAM_ID 외래 키 하나로 양방향 연관관계 가짐(양쪽 조인할 수 있음)

> 둘 중 하나로 외래키를 관리해야 한다. - 연관관계의 주인  

*양방향 매핑 규칙*
- 객체의 두 관계중 하나를 연관관계의 주인으로 지정
- 연관관계의 주인만이 외래 키를 관리(등록, 수정)
- 주인이 아닌쪽은 읽기만 가능
- 주인은 mappedBy 속성 사용 x
- 주인이 아니면 mappedBy 속성으로 주인 지정

> *누구를 주인으로?*  
> - 외래키가 있는 곳을 주인으로 정해라.  
> - 다(Many) 쪽이 연관관계 주인.  

*양방향 매핑 시 가장 많이 하는 실수*
- 연관관계의 주인에 값을 입력하지 않음.

> 순수한 객체 관계를 고려하면 항상 양쪽 다 값을 입력해야한다.  
> -> 연관관계 편의 메소드를 생성하자   
```
> public void setTeam(Team team) {
    this.team = team;
    team.getMembers().add(this);
}
```
> 양방향 매핑 시에 무한루프를 조심하자.  

*양방향 매핑 정리*
- 단방향 매핑만으로도 이미 연관관계 매핑 완료
- 양방향 매핑은 반대 방향으로 조회(객체 그래프 탐색) 기능이 추가된 것 뿐
- JPQL에서 역방향으로 탐색할 일이 많음
- 단방향 매핑을 잘하고 양방향은 필요할 때 추가(테이블에 영향을 주지 않음)
- 연관관계의 주인은 비즈니스 로직이 아니라 외래 키의 위치를 기준으로 정해야함.


### 단방향, 양방향
- 테이블
	- 외래 키 하나로 양쪽 조인 가능
	- 사실 방향이라는 개념이 없음
- 객체
	- 참조용 필드가 있는 쪽으로만 참조 가능
	- 한쪽만 참조하면 단방향
	- 양쪽이 서로 참조하면 양방향

### 연관관계의 주인
- 테이블은 외래 키 하나로 두 테이블이 연관관계를 맺음
- 객체 양방향 관계는 A -> B, B -> A 처럼 참조가 2군데
- 연관관계의 주인 : 외래 키를 관리하는 참조
- 주인의 반대편 : 외래 키 영향주지 않고, 단순 조회만

### 다대일
- 다 쪽이 , 외래키 있는 쪽이 연관관계 주인
- @ManyToOne 반대는 놔둠
- 양방향 하고 싶을때는 반대만 추가하고 DB 변경은 필요없음 mappedBy만 붙이면됨.

## 일대다
- 일이 연관관계의 주인
- 다 쪽에 외래키가 있음
- 객체와 테이블의 차이 떄문에 반대편 테이블의 외래키를 관리하는 특이한 구조
- @JoinColumn을 꼭 사용해야함. 그렇지않으면 조인 테이블 방식을 사용함.(중간에 테이블 하나 추가)
*단점*
- 엔티티가 관리하는 외래키가 다른 테이블에 있음
- 연관관계 관리를 위해 추가로 UPDATE SQL 실행

> 일대다 단방향 매핑보다는 다대일 양방향 매핑 사용  

### 일대다 양방향
- 공식적으로는 존재 X
- @JoinColumn(insertable = false, updatable = false)
- 읽기 전용 필드를 사용해서 양방향 처럼 사용하는 방법
- *다대일 양방향을 사용하자*

## 일대일
- 일대일 관계는 그 반대도 일대일
- 주 테이블이나 대상 테이블 중에 외래 키 선택 가능
- 외래 키에 데이터베이스 유니크 제약조건 추### 주 테이블에 외래키
- 주 객체가 대상 객체의 참조 가지는 것처 주 테이블에 외래 키를 두고 대상 테이블을 찾음
- 객체지향 개발자 선호
- JPA 매핑 편리
- 장점 : 주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인 가능
- 단점 : 값이 없으면 외래키에 null 허용
### 대상 테이블에 외래키
- 대상 테이블에 외래 키가 존재
- 전통적인 데이터베이스 개발자 선호
- 장점 : 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경할 때 테이블 구조 유지
- 단점 : 프록시 기능의 한계로 지연 로딩으로 설정해도 항상 즉시 로딩됨

## 다대다
- 관계형 DB는 정규화된 테이블 2개로 다대다 관계를 표현할수없음
- 연결 테이블을 추가해서 일대일, 다대일 관계로 풀어내야함.
- 객체는 컬렉션을 사용해서 다대다 가능
- @ManyToMany 사용
- @JoinTable로 연결 테이블 지정
- 다대다 매핑 : 단방향, 양방향 가능
### 다대다 매핑 한계
- 편리해 보이지만 실무 사용 X
- 연결 테이블이 단순히 연결만 하고 끝나지 않음
- 주문시간, 수량같은 데이터가 들어올 수 있음 
### 다대다 매핑 한계 극복
- 연결 테이블용 엔티티 추가
- @ManyToMany -> @OneToMany, @ManyToOne


## N:M 관계는 1:N, N:1 로
- 테이블의 N:M 관계는 중간 테이블을 이용해서 1:N, N:1
- 실전에서는 중간테이블이 단순하지 않다.
- @ManyToMany는 제약 : 필드 추가 X, 엔티티 테이블 불일치
- 실전에서는 @ManyToMany 사용 X

### @JoinColumn
- 외래키를 매핑할때 사용
- name : 매핑할 외래키 이름
- referencedColumnName : 외래키가 참조하는 대상 테이블의 컬럼명

### @ManyToOne
- optional : false로 설정하면 연관된 엔티티가 항상 있어야 한다.
- fetch 글로벌 패치 전략 설정
- cascade : 영속성 전이 기능 사용

### @OneToMany
- mappedBy : 연관관계의 주인 필드를 선택한다.


## 상속관계 매핑
- 관계형 데이터베이스는 상속 관계 X
- 슈퍼타입 서브타입 관계라는 모델링 기법이 객체 상속과 유사
- 상속관계 매핑 : 객체의 상속과 구조와 DB의 슈퍼타입 서브타입 관계를 매핑
- 슈퍼타입 서브타입 논리 모델을 실제 물리 모델로 구현하는 방법
	- 각각 테이블로 변환 -> 조인 전략
	- 통합 테이블로 변환 -> 단일 테이블 전략
	- 서브타입 테이블로 변환 -> 구현 클래스마다 테이블 전략

- 조인전략
*장점*
	- 테이블 정규화
	- 외래 키 참조 무결성 제약조건 활용가능
	- 저장공간 효율화
*단점*
	- 조회 시 조인을 많이 사용, 성능 저하
	- 조회 쿼리가 복잡함
	- 데이터 저장시 INSERT SQL 2번 호출
- 단일 테이블 전략
*장점*
	- 조인이 필요없으므로 일반적으로 조회 성능이 빠름
	- 조회 쿼리가 단순함
*단점*
	- 자식 엔티티가 매핑한 컬럼은 모두 null 허용
	- 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있어서,
   	상황에 따라 조회 성능이 오히려 느려질 수 있다.
- 구현 클래스마다 테이블 전략
	- 쓰면 안되는 전략
*장점*
	- 서브 타입을 명확하게 구분해서 처리할때 효과적
	- not null 제약조건 사용가능
*단점*
	- 여러 자식 테이블을 함께 조회할 때 성능이 느림
	- 자식 테이블을 통합해서 쿼리하기 어려움

## @MappedSuperclass
- 공통 매핑 정보가 필요할 때 사용 (id, name) 객체 입장에서 속성만 상속
- 상속관계 매핑 X
- 엔티티 X , 테이블과 매핑 X
- 부모 클래스를 상속 받는 자식 클래스에 매핑 정보만 제공
- 조회, 검색 불가
- 직접 사용할 일 없으므로 추상 클래스 권장
- 참고로 @Entity로 지정한 클래스는 @Entity나 @MappedSuperclass로 지정한 클래스만 상속 가능


## 프록시
### em.find() vs em.getReference()
- em.find() : 데이터베이스를 통해 실제 엔티티 객체 조회
- em.getReference() : 데이터베이스 조회를 미루는 가짜(프록시) 엔티티 객체 조회
### 프록시 특징
- 실제 클래스를 상속받아서 만들어짐
- 실제 클래스와 겉 모양이 같다.
- 사용하는 입장에서는 진짜 객체인지 프록시 객체인지 구분하지 않고 사용하면 됨.
- 프록시 객체는 실제 객체의 참조를 보관
- 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드 호출
### 프록시 객체 초기화 과정
1. member 프록시 객체 생성후 getName() 호출하면
2. 영속성 컨텍스트에 초기화 요청
3. DB조회 해서 실제 Entity 생성
4. member target과 실제 객체 연결
### *프록시 특징*
- 프록시 객체는 처음 사용할 때 한번만 초기화
- 프록시 객체를 초기화 할때 프록시 객체로 실제 엔티티로 바뀌는게 아니라 초기화되면 프록시 객체를 통해 실제 엔티티 접근 가능
- 프록시 객체는 원본 엔티티를 상속받음 따라서 타입 체크시 주의해야함(== 비교 대신 instance of 사용)
- 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 em.getReference()를 호출해도 실제 엔티티 반환
- 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때, 프록시를 초기화하면 문제 발생
### 프록시 확인
- 프록시 인스턴스 초기화 여부 확인
PersistenceUnitUtil.isLoaded(Object entity)
- 프록시 클래스 확인 방법
entity.getClass().getName() 출력 (..javasist.. or HibernateProxy…)
- 프록시 강제 초기화
org.hibernate.Hibernate.initialize(entity);
- 참고 : JPA 표준은 강제 초기화 없음
강제 호출 : member.getName()

### 프록시와 즉시로딩 주의
- 가급적 지연 로딩만 사용 (특히 실무에서)
- 즉시 로딩을 적용하면 예상하지 못한 SQL 발생
- 즉시 로딩은 JPQL에서 N+1 문제 일으킨다.
- @ManyToOne, @OneToOne은 기본이 즉시 로딩 -> LAZY로 설정
- @OneToMany, @ManyToMany는 기본이 지연 로딩

### 지연로딩 활용
- Member와 Team은 자주 함께 사용 -> 즉시 로딩
- Member와 Order는 가끔 사용 -> 지연 로딩
- Order와 Product는 자주 함께 사용 -> 즉시 로딩

### 지연로딩 활용 - 실무
- 모든 연관관계에 지연 로딩을 사용해라!
- 실무에서 즉시 로딩을 사용하지 마라!
- JPQL fetch 조인이나, 엔티티 그래프 기능을 사용해라!
- 즉시 로딩은 상상하지 못한 쿼리가 나간다.

### 영속성 전이 : CASCASDE
- 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을때
- 예 : 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장.
**주의**
- 영속성 전이는 연관관계를 매핑하는 건과 아무관련이 없음
- 엔티티를 영속화 할 때 연관된 엔티티도 함께 영속화하는 편리함을 제공할 뿐
**종류**
- ALL : 모두 적용
- PERSIST : 영속
- REMOVE : 삭제
- MERGE : 병합
- REFRESH 
- DETACH

### 고아객체
- 고아객체 제거 : 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제
- orphanRemoval = true
**주의**
- 참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제하는 기능
- 참조하는 곳이 하나일때 사용해야함
- 특정 엔티티가 개인 소유할 때 사용
- @OneToOne, @OneToMany 만 가능

### 영속성 전이 + 고아 객체, 생명주기
- CascadeType.ALL + orphanRemoval = true
- 스스로 생명주기를 관리하는 엔티티는 em.persist()로 영속화, em.remove()로 제거
- 두 옵션을 모두 활성화 하면 부모 엔티티를 통해서 자신의 생명주기를 관리할 수 있음
- 도메인 주도 설계(DDD)의 Aggregate Root 개념을 구현할 때 유용

## JPA의 데이터 타입 분류
### 엔티티 타입
- @Entity로 정의하는 객체
- 데이터가 변해도 식별자로 지속해서 추적가능
- 예) 회원 엔티티의 키나 나이 값을 변경해도 식별자로 인식 가능
### 값 타입
- int, Integer, String 처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체
- 식별자가 없고 값만 있으므로 변경시 추적 불가
- 예) 숫자 100을 200으로 변경하면 완전히 다른 값으로 대체
- 값타입 분류
	- 기본 값 타입 - 자바 기본타입, 래퍼클래스, String
	- 임베디드 타입 - 복합값타입
	- 컬렉션 값 타입
#### 기본 값타입
- 예) String name, int age
- 생명주기를 엔티티에 의존 - 예) 회원을 삭제하면 이름, 나이 필드도 함께 삭제
- 값 타입은 공유하면 X - 예) 회원 이름 변경시 다른 회원의 이름도 함께 변경되면 안됨
> 참고 ) 자바의 기본 타입은 절대 공유 X  
> int, double 같은 기본 타입은 절대 공유 X  
> 기본 타입은 항상 값을 복사함  
> Integer 같은 래퍼 클래스나 String 같은 특수한 클래스는 공유 가능한 객체지만 변경 X  
#### 임베디드 타입 (복합 값 타입)
- 새로운 값 타입을 직접 정의할 수 있음
- JPA는 임베디드 타입이라 함
- 주로 기본 값 타입을 모아서 만들어서 복합값 타입이라고도함
- int, String 과 같은 값 타입
#### 임베디드 타입 사용법
- @Embeddable : 값 타입을 정의하는 곳에 표시
- @Embedded : 값 타입을 사용하는 곳에 표시
- 기본 생성자 필수
#### 임베디드 타입의 장점
- 재사용
- 높은 응집도
- Period.isWork() 처럼 해당 값 타입만 사용하는 의미있는 메소드를 만들수 있음
- 임베디드 타입을 포함한 모든 값 타입은, 값 타입을 소유한 엔티티의 생명주기를 의존함
#### 임베디드 타입과 테이블 매핑
- 임베디드 타입은 엔티티의 값일 뿐이다.
- 임베디드 타입을 사용하기 전과 후에 매핑하는 테이블은 같다.
- 객체와 테이블을 아주 세밀하게 매핑하는 것이 가능
- 잘 설계한 ORM 애플리케이션은 매핑한 테이블의 수보다 클래스 수가 많음.
#### @AttributeOverride : 속성 재정의
- 한 엔티티에서 같은 값 타입을 사용하면 컬럼명이 중복됨
- @AttributeOverrides, @AttributeOverride 를 사용해서 컬럼 명 속성을 재정의
#### 임베디드타입과 null
- 임베디드 타입의 값이 null이면 매핑한 컬럼 값은 모두 null
#### 값 타입과 기본 객체
- 값티입은 복잡한 객체 세상을 조금이라도 단순화하려고 만든 개념이다. 따라서 값 타입은 단순하고, 안전하게 다룰 수 있어야 한다.
#### 값 타입 공유 참조
- 임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 위험함, 부작용 발생#### 값 타입 복사
- 값 타입의 실제 인스턴스인 값을 공유하는 것은 위험하므로 값을 복사해서 사용
#### 객체 타입의 한계
- 객체 타입은 참조 값을 직접 대입하는 것을 막을 방법이 없다.
- 객체의 공유 참조는 피할 수 없다.
#### 불변객체
- 객체 타입을 수정할 수 없게 만들면 부작용을 원천 차단
- 값 타입은 불변 객체로 설계해야함
- 불변 객체 : 생성 시점 이후 절대 값을 변경할 수 없는 객체
- 생성자로만 값을 설정하고 수정자를 만들지 않으면 됨
- 참고 : Integer, String은 자바가 제공하는 대표적인 불변 객체
#### 값 타입 비교
- 값타입은 인스턴스가 달라도 그 안에 값이 같으면 같은 것으로 봐야함
- 동일성 비교 : 인스턴스의 참조 값을 비교, == 사용
- 동등성 비교 : 인스턴스의 값을 비교, equals() 사용
- 값 타입은 a.equals(b)를 사용해서 동등성 비교를 해야함
- 값 타입은 equals() 메소드를 적절하게 재정의(주로 모든 필드 사용)
#### 값 타입 컬렉션
- 값 타입을 하나 이상 저장할 때 사용
- @ElementCollection, @CollectionTable 사용
- 데이터베이스는 컬렉션을 같은 테이블에 저장할 수 없다.
- 컬렉션을 저장하기 위한 별도의 테이블이 필요함
#### 값 타입 컬렉션의 제약사항
- 값 타입은 엔티티와 다르게 식별자 개념이 없다.
- 값은 변경하면 추적이 어렵다
- __값 타입 컬렉션에 변경 사항이 발생하면, 주인 엔티티와 연관된 모든 데이터를 삭제하고, 값 타입 컬렉션에 있는 현재 값을 모두 다시 저장한다.__
- 값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본키를 구성해야함: null 입력 X, 중복 저장X
#### 값 타입 컬렉션 대안
- 실무에서는 상황에 따라 값 타입 컬렉션 대신에 일대다 관계를 고려
- 일대다 관계를 위한 엔티티를 만들고, 여기에서 값 타입을 사용
- 영속성 전이(Cascade) + 고아 객체 제거를 사용해서 값 타입 컬렉션 처럼 사용
- EX) Address Entity
> 값 타입은 정말 값 타입이라 판단될 때 사용  
> 엔티티와 값 타입을 혼동해서 엔티티를 값 타입으로 만들면 안됨  
> 식별자가 필요하고, 지속해서 값을 추적, 변경해야한다면 그것은 값 타입이 아닌 엔티티  


## 객체지향 쿼리 언어
### JPA는 다양한 쿼리 방법을 지원
- JPQL
- JPA Criteria
- Query DSL
- 네이티브 SQL
- JDBC API 직접 사용, MyBatis, SpringJdbcTemplate 함께 사용

### JPQL
- 가장 단순한 조회 방법
- JPA를 사용하면 엔티티 객체를 중심으로 개발 검색을 할때도 테이블이 아닌 엔티티 객체 대상 검색, 모든 DB 데이터를 객체로 변환해서 검색하는 것 불가능
- 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 SQL 필요
- JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어 제공
- JPQL은 엔티티 객체를 대상으로 쿼리
- SQL은 데이터베이스 테이블을 대상으로 쿼리

### Criteria
- 문자가 아닌 자바코드로 JPQL 작성 가능
- JPQL 빌더 역할
- JPA 공식 기능
- 단점 : 너무 복잡하고 실용성이 없음
- Criteria 대신 QueryDSL 사용 권장

### QueryDSL
- 문자가 아닌 자바코드로 JPQL을 작성할 수 있음
- JPQL 빌더 역할
- 컴파일 시점에 문법 오류를 찾을 수 있음
- 동적쿼리 작성 편리함
- 단순하고 쉬움
- 실무 사용 권장

### 네이티브 SQL
- JPA 가 제공하는 SQL을 직접 사용하는 기능
- JPQL로 해결할 수 없는 특정 데이터베이스에 의존적인 기능
- 예) 오라클 CONNECT BY, 특정 DB만 사용하는 SQL 힌트

### JDBC 직접 사용, SpringJdbcTemplate 등
- JPA를 사용하면서 JDBC 커넥션을 직접 사용하거나, 스프링 JdbcTemplate, 마이바티스등을 함께 사용가능
- 단 영속성 컨텍스트를 적잘한 시점에 강제로 플러시 필요
- 예) JPA를 우회해서 SQL을 실행하기 직전에 영속성 콘텍스트 수동 플러시


## JPQL
- JPQL은 객체지향 쿼리 언어, 테이블을 대상으로 쿼하는 것이 아니라 엔티티 객체를 대상으로 쿼리
- JPQL은 SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않음
- JPQL은 결국 SQL로 변환됨.

### JPQL 문법
- select m from Member as m where m.age > 18
- 엔티티와 속성은 대소문자 구분 O
- JPQL 키워드는 대소문자 구분 X
- 엔티티 이름 사용, 테이블 이름이 아님
- 별칭은 필수(m)

### TypeQuery, Query
- TypeQuery :  반환 타입이 명확할 때 사용 (TypedQuery<Member> query = em…)
- Query : 반환 타입이 명확하지 않을 때 사용 (Query query = em…)

### 결과조회 API
- query.getResultList() : 결과가 하나 이상일 때, 리스트 반환, 결과가 없으면 빈 리스트 반환
- query.getSingleResult() : 결과가 정확히 하나, 단일 객체 반환, 없거나 둘이상이어도 Exception

### 파라미터 바인딩 - 이름기준, 위치기준
```
SELECT m FROM Member m where m.username=:username
query.setParameter("username", usernameParam);
```

```
SELECT m FROM Member m where m.username=?1
query.setParameter(1, usernameParam);
```

### 프로젝션
- select 절에 조회할 대상을 지정하는 것
- 프로젝션 대상 : 엔티티, 임베디드 타입, 스칼라 타입
- select m from member m -> 엔티티 프로젝션
- select m.team from member m -> 엔티티 프로젝션
- select m.address from member m -> 임베디드 타입 프로젝션
- select m.username, m.age from member m -> 스칼라 타입 프로젝션
- DISTINCT로 중복 제거

#### 여러 값 조회
- select m.username, m.age from member m
- 1. Query 타입으로 조회
- 2. Object[] 타입으로 조회
- 3. new 명령어로 조회
	- 단순 값을 DTO로 바로 조회
	```
       select new jpabook.jpql.UserDTO(m.username, m.age)
	from member m
	```
	- 패키지 명을 포함한 전체 클래스 명 입력
	- 순서와 타입이 일치하는 생성자 필요

### 페이징
#### 페이징 API
- JPA는 페이징을 다음 두 API로 추상화
- setFirstResult : 조회 시작 위치
- setMaxResults : 조회할 데이터 수

### 조인
- 내부 조인
select m from member m [inner] join m.team t
- 외부 조인
select m from member m left [outer] join m.team t
- 세타 조인
select count(m) from member m, team t where m.username = t.name
- ON 절(JPA 2.1부터 지원)
	- 조인 대상 필터링
		- 예) 회원과 팀을 조인하면서, 팀 이름이 A인 팀만 조인
	- 연관관계가 없는 엔티티 외부 조인
		- 예) 회원의 이름과 팀의 이름이 같은 대상 외부 조인

### 서브 쿼리
- 나이가 평균보다 많은 회원
select m from member m
where m.age > (select avg(m2.age) from member m2)
- 한건이라도 주문한 고객
select m from member m
where (select count(o) from order o where m = o.member) > 0
- 서브쿼리 지원 함수
	- [NOT] EXISTS (subquery) : 서브쿼리에 결과가 존재하면 참
	- [NOT] IN (subquery) : 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참
#### JPA 서브쿼리 한계
- JPA는 WHERE, HAVING 절에서만 서브 쿼리 사용 가능
- SELECT 절도 가능(하이버네이트에서 지원)
- FROM 절의 서브 쿼리는 현재 JPQL에서 불가능
	- 조인으로 풀 수 있으면 풀어서 해결

### JPQL 타입 표현
- 문자 : ‘HELLO’, ‘She’’s’
- 숫자 : 10L, 10D, 10F
- Boolean : TRUE, FALSE
- ENUM : jpabook.MemberType.Admin (패키지명 포함)
- 엔티티 타입 : TYPE(m) = Member (상속 관계에서 사용)

### JPQL 기타
- SQL과 문법이 같은 식
- EXISTS, IN
- AND, OR, NOT
- =, >, >=, <, <=, <>
- BETWEEN, LIKE, IS NULL

### 조건식
- CASE 식
	- 기본 CASE 식
	- 단순 CASE 식
	- COALESCE : 하나씩 조회해서 null 이 아니면 반환
	- NULLIF : 두 값이 같으면 null 반환, 다르면 첫번째 값 반환

### JPQL 기본 함수
- 표준함수
	- CONCAT, SUBSTRING, TRIM, LOWER, UPPER, LENGTH, LOCATE, ABS, SQRT, MOD, SIZE, INDEX
- 사용자 정의 함수
	- 하이버네이트는 사용전 방언에 추가해야 한다. 
	- 사용하는 DB 방언을 상속받고, 사용자 정의 함수를 등록한다.

### 경로 표현식
- 점을 찍어서 객체 그래프를 탐색
- 예)
```
select m.username -> 상태 필드
  from Member m
		join m.team t -> 단일 값 연관 필드
		join m.orders o -> 컬렉션 값 연관 필드
where t.name = '팀A'
```
- 경로 표현식 용어 정리
	- 상태필드 : 단순히 값을 저장하기 위한 필드 (m.username)
	- 연관필드 : 연관관계를 위한 필드
		- 단일값 연관 필드 : @ManyToOne, @OneToOne,  대상이 엔티티
		- 컬렉션 값 연관 필드 : @OneToMany, @ManyToMany, 대상이 컬렉션
- 경로 표현식 특징
	- 상태 필드 : 경로 탐색의 끝, 탐색 X
	- 단일 값 연관 경로 : 묵시적 내부 조인 발생, 탐색 O
	- 컬렉션 값 연관 경로 : 묵시적 내부 조인 발생, 탐색 X
		- FROM 절에서 명시적 조인을 통해 별칭을 얻으면 별칭을 통해 탐색 가능
- 명시적 조인, 묵시적 조인
	- 명시적 조인 : Join 키워드 직접 사용
	select m from Member m join m.team t
	- 묵시적 조인 : 경로 표현식에 의해 묵시적으로 SQL 조인 발생(내부조인만 가능)
	select m.team from Member m
- 경로 탐색을 사용한 묵시적 조인 시 주의사항
	- 항상 내부 조인
	- 컬렉션은 경로 탐색의 끝, 명시적 조인을 통해 별칭을 얻어야함
	- 경로탐색은 주로 SELECT, WHERE 절에서 사용하지만 묵시적 조인으로 인해 SQL의 FROM(JOIN) 절에 영향을 줌
- 실무 조언
	- 가급적 묵시적 조인 대신에 명시적 조인 사용
	- 조인은 SQL 튜닝에 중요한 포인트
	- 묵시적 조인은 조인이 일어나는 상황을 한눈에 파악하기 어려움

### 페치 조인(fetch join)
- SQL 조인 종류 X
- JPQL에서 성능 최적화를 위해 제공하는 기능
- 연관된 엔티티나 컬렉션을 SQL 한번에 함께 조회하는 기능
- join fetch 명령어 사용
- 페치 조인 ::==[LEFT [OUTER] | INNER ] JOIN FETCH 조인경로

#### 엔티티 페치 조인
- 회원을 조회하면서 연관된 팀도 함께 조회 (SQL 한번에)
- SQL을 보면 회원 뿐만 아니라 팀(T.*)도 함께 SELECT
- [JPQL]
select m from Member m join fetch m.team
- [SQL]
select m.*, t.* from member m
inner join team t on m.team_id = t.id

#### 페치 조인과 DISTINCT
- SQL의 DISTINCT는 중복된 결과를 제거하는 명령
- JPQL의 DISTINCT 2가지 기능 제공
- 1. SQL에 DISTINCT를 추가
- 2. 애플리케이션에서 엔티티 중복제거
- select distinct t
from Team t join fetch t.members
where t.name = ‘팀A’
	- SQL에 DISTINCT를 추가하지만 데이터가 다르므로 SQL 결과에서 중복제거 실패
- DISTINCT가 추가로 애플리케이션에서 중복제거시도
- 같은 실벽자를 가진 Team엔티티 제거

#### 페치 조인과 일반 조인의 차이
- 일반 조인 실행시 연관된 엔티티를 함께 조회하지 않음
- JPQL은 결과를 반환할 때 연관관계 고려 X
- 단지 SELECT 절에 지정한 엔티티만 조회할 뿐
- 여기서는 팀 엔티티만 조회하고, 회원 엔티티는 조회 X
- 페치 조인을 사용할 때만 연관된 엔티티도 함께 조회 (즉시 로딩)
- 페치 조인은 객체 그래프를 SQL 한번에 조회하는 개념

### 페치 조인의 특징과 한계
- 페치 조인 대상에는 별칭을 줄 수 없다.
	- 하이버네이트는 가능, 가급적 사용 X
- 둘 이상의 컬렉션은 페치 조인 할 수 없다.
- 컬렉션을 페치조인하면 페이징API(setFirstResult, setMaxResults)를 사용할 수 없다.
- 연관된 엔티티들을 SQL 한 번으로 조회 - 성능 최적화
- 엔티티에 직접 적용하는 글로벌 로딩 전략보다 우선함
	- @OneToMany(fetch = FetchType.LAZY) //글로벌 로딩 전략
- 실무에서 글로벌 로딩 전략은 모두 지연 로딩
- 최적화가 필요한 곳은 페치 조인 적용

### 페치 조인 - 정리
- 모든 것을 페치 조인으로 해결할 수 는 없음
- 페치 조인은 객체 그래프를 유지할 때 사용하면 효과적
- 여러 테이블을 조인해서 엔티티가 가진 모양이 아닌 전혀 다른 결과를 내야하면,
페치 조인보다는 일반 조인을 사용하고 필요한 데이터들만 조회해서 DTO로 반환하는 것이 효과적

### 다형성 퀴리
TYPE
- 조회 대상을 특정자식으로 한정
- 예) Item 중에 Book, Movie를 조회하라
TREAT
- 자바의 타입 캐스팅과 유사
- 상속구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용
- FROM, WHERE, SELECT 사용

### 엔티티 직접사용 - 기본 키 값
- JPQL에서 엔티티를 직접 사용하면 SQL에서 해당 엔티티의 기본 키 값을 사용

### Named쿼리 - 정적 쿼리
- 미리 정의해서 이름을 부여해두고 사용하는 JPQL
- 정적쿼리
- 어노테이션, XML에 정의
- 애플리케이션 로딩시점에 초기화 후 재사용
- 애플리케이션 로딩시점에 쿼리를 검증

#### Named쿼리 환경에 따른 설정
- xml이 항상 우선권을 가진다.
- 애플리케이션 운영환경에 따라 다른 XML을 배포 할 수 있다. 

### JPQL 벌크연산
- 일반적 sql update, delete 문
- 예) 재고가 10개 미만인 모든 제품의 가격 10% 상승
- 쿼리 한번으로 여러 테이블 로우변경(엔티티)
- excuteUpdate()의 결과는 영향받은 엔티티 수 반환
- UPDATE, DELTE 지원
- INSERT(insert into … select, 하이버네이트 지원)

#### 벌크 연산 주의
- 벌크 연산은 영속성 컨텍스트를 무시하고, 데이터베이스에 직접 쿼리
	- 벌크연산을 먼저 수행
	- **벌크연산 후 영속성 컨텍스트 초기화 **