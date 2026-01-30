#Java 
#Spring 

---
https://cafe.naver.com/xxxjjhhh/229
개발자 유미 강의 참조


# 1+N 문제란?
자바에서 DB에 접근할 때, JPA를 자주 사용하게 됨

JPA는 추상화와 데이터를 객체 지향적으로 다룰 수 있다는 장점이 존재하지만, 
JPA 표준 스펙에 의해 상황에 따라 "1+N"이라는 문제 존재

1+N 문제는, 
2개 이상의 JOIN 된 테이블에서 연관된 데이터를 조회할 때, 
하나의 쿼리로 조회가 가능한 상황이지만, 
JPA 구현체는 1개 + N개의 쿼리로 조회하는 문제

# 실습 환경

**연관 관계 구조**
연습용 연관 관계는 “국가” : “도시” = 1 : N 형태의 JOIN
따라서 “korea”라는 하나의 CountryEntity에
“seoul”, “busan”과 같은 CityEntity가 연관됨

CityEntity
```java
@ManyToOne  
private CountryEntity countryEntity;
```
​
CountryEntity
```java
@OneToMany(mappedBy = "countryEntity")  
private List<CityEntity> cityEntities = new ArrayList<>();
```

# 1+N 발생 상황
**연관 관계 데이터 확인 case**
![[Pasted image 20260129214520.png]]
OneToMany 연관 관계예서 데이터를 확인하는 케이스
(CountryEntity : CityEntity)

**- One쪽 조회**
- 연관된 Many 접근 안함
- 연관된 Many 접근 함

**- Many쪽 조회 (주인)**
- 연관된 One 접근 안함
- 연관된 One 접근 함

### Case1 : One쪽 조회
**- One 조회 후 연관된 Many 접근 안함**
단, 한 번의 쿼리

**- One 조회 후 연관된 Many 접근 함**
One용 쿼리 한 번 + List< CountryEntity >를 순회하며 각각의 연관된 CityEntity들을 확인할 쿼리 N번

### Case2 : Many쪽 조회(주인쪽)
**- Many 조회 후 연관된 One 접근 안함**
CityEntity 쿼리 한 번 및 mappedBy용 쿼리 한 번 (총 2번)

**- Many 조회 후 연관된 One 접근 함**
CityEntity 쿼리 한 번 및 mappedBy용 쿼리 한 번 (총 2번)


**1+N**
위 케이스들을 통해 확인한 결과,
연관 관계의 두 Entity에 대해 두 Entity의 정보가 보고 싶은 경우 RDB SQL에서는 한 번의 구문으로 쿼리가 가능하나 JPA는 각각의 Entity를 각각 조회

주인이 되는 Many쪽에서 연관된 One을 조회하는 건 그나마 2번의 쿼리로 큰 문제가 없지만,
List< One >쪽에서 각각의 One에 대한 Many를 조회할 경우 
List<>의 size 만큼 쿼리가 발생하는 것이 확인되며 size가 커질수록 문제가 IO적 문제가 발생함이 예상됨.


**위험 예시 케이스**
- 프로젝트별 유저목록
	깃허브와 같은 프로젝트 리포지토리 List< One >이 존재하고, 각각의 리포지토리별 유저가 Many로 JOIN된 상태에서, 전체 리포지토리 목록과 목록별 참여 인원을 확인 케이스.
	 
- 관리자 대시보드 유저별 데이터
	관리자 대시보드에서 전체 유저에 List< One > 대한 각각의 게시글, 주문목록과 같은 유저의 데이터를 확인하는 케이스.


# Lazy, Eager
**연관 관계 접근에 따른 쿼리**
JPA의 find 관련 메소드들은 해당 Entity 테이블 데이터만 가져오도록 설계되어 있음

따라서 find로 찾은 특정 Entity의 연관된 다른 테이블 Entity는 접근시 무조건 추가 조회를 통해 가져옴
(연관 데이터를 사용할지 모르는 상황에서 가져오는 것은 자원적 소모가 있음)

**의문점은**
3강에서 각 case별 쿼리 개수 테스트를 수행할 때,
가져온 Entity에서 연관된 Entity에 접근하지 않는 상황에 대해 아래와 같은 의문이 있음

- ManyToOne : 접근하지 않아도 연관쪽 쿼리가 날라감
- OneToMany : 접근하지 않으면 연관쪽은 안날라감

**Lazy, Eager**
동일하게 접근하지 않았지만, 추가 쿼리의 발생 여부는 JPA Lazy, Eager 로딩 설정 때문

- Lazy 로딩 : 연관에 접근하지 않는다면 가져오지 않음
- Eager 로딩 : 연관에 접근하지 않아도 추가 쿼리로 먼저 가져옴

**JPA JOIN별 default 값**

- OneToOne : Eager
- ManyToOne : Eager
- OneToMany : Lazy

**ManyToOne Lazy 명시**
기존 Eager가 default인 ManyToOne Lazy 설정

CityEntity
```java
@ManyToOne(fetch = FetchType.LAZY) 
private CountryEntity countryEntity;
```

**기타**
Lazy, Eager 설정 모두 연관에 대한 추가 쿼리를 나중에 할 지, 지금 할 지 결정하는 것으로 
1+N 쿼리를 1 쿼리로 대체할 수 있는 것은 아님

# 연관 관계 쿼리 단일화
**JPA 쿼리 단일화**
JPA에서 Entity 조회 후 연관된 Entity 접근시 발생하는 추가 쿼리를 단건의 쿼리로 단일화
단일화 방법은 여러가지가 존재하지만, 장단점이 있음

1. JPQL @Query로 JOIN FETCH 작성
2. QueryDSL로 fetchJoin()
3. @EntityGraph
4. Lazy 쪽 조회시 Batch 설정을 통한 IN절

이 중 가장 많이 사용되는 JPQL @Query 작성에 대해 알아봄

**JPQL**
JPQL은 JPA에서 Entity(객체) 기반으로 SQL 쿼리를 사용할 수 있는 방법
Spring Data JPA 의존성에서 `@Query("")` 어노테이션을 통해 Repository에서 JPQL을 쉽게 다룰 수 있음

