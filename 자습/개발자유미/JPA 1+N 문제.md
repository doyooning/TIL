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
	SQL문을 직접 작성하기 때문에 런타임 에러 등이 발생할 수 있음
2. QueryDSL로 fetchJoin()
	JPQL을 고도화시킴
3. @EntityGraph
4. Lazy 쪽 조회시 Batch 설정을 통한 IN절
	N개 쪽에서 N개 전부 가져오지 않고 적당히 나눠서 IN절로 처리

이 중 실무에서 가장 많이 사용되는 JPQL @Query 작성에 대해 알아봄

**JPQL**
JPQL은 JPA에서 Entity(객체) 기반으로 SQL 쿼리를 사용할 수 있는 방법
Spring Data JPA 의존성에서 `@Query("")` 어노테이션을 통해 Repository에서 JPQL을 쉽게 다룰 수 있음

**JPQL JOIN FETCH 쿼리 작성**

- 쿼리 작성
```java
@Query("SELECT co FROM CountryEntity co " +
		"JOIN FETCH co.cityEntities ci")
List<CountryEntity> findAllFetch();
```
보통은 쿼리 작성시 객체 이름은 객체 앞글자를 따서 소문자로 작성
여기서는 겹치니까 co, ci로 작성

- 서비스단 사용
```java
public List<CountryEntity> readCountryFetch() {
    return countryRepository.findAllFetch();
}
```

- 컨트롤러단 수정
```java
@GetMapping("/read/one/child")
public String readOneChild(Model model) {
    model.addAttribute("COUNTRYLIST", countryService.readCountryFetch());
    return "readOneChild";
}
```

# JOIN FETCH, LEFT JOIN FETCH

**JOIN FETCH 데이터 누락**
JOIN FETCH 구문 작성시 누락되는 데이터가 존재

OneToMany와 같은 JOIN 상황에서 
One은 존재하지만, 연관된 Many가 0개인 경우
이 경우 One쪽도 함께 조회되지 않는 데이터 누락 문제가 발생

**- 예시**
예제에서 CountryEntity 데이터에 물린 CityEntity가 없는 경우 
findAll (JPQL로 작성)로 찾은 List< CountryEntity >에 해당 CountryEntity가 포함되지 않음

- 상황 만들기
	CountryEntity에 "germany" 국가 데이터를 추가하고 연관된 City는 추가하지 않은 경우

**해결 방법**
- 기존 SQL 구문
```java
@Query("SELECT co FROM CountryEntity co " +
"JOIN FETCH co.cityEntities ci")
List<CountryEntity> findAllFetch();
```

- 변경: LEFT 추가
```java
@Query("SELECT co FROM CountryEntity co " +
"LEFT JOIN FETCH co.cityEntities ci")
List<CountryEntity> findAllFetch();
```

**원리**
- JOIN FETCH
	: INNER JOIN
	연관된게 있는 교집합이어야 조회 됨

- LEFT JOIN FETCH
	: LEFT OUTER JOIN
	부모 + 교집합이면 조회 됨

# 다중 OneToMany 문제
**DB 설계시**
DB 테이블 설계시 
하나의 Entity에 대해 OneToMany가 2개 이상 들어갈 경우가 많음

- Entity 예시
```java
@Entity
@Getter
@Setter
public class CountryEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true)
    private String country;

    @OneToMany(mappedBy = "countryEntity")
    private List<CityEntity> cityEntities = new ArrayList<>();

    public void addCityEntity(CityEntity cityEntity) {
        cityEntities.add(cityEntity);
        cityEntity.setCountryEntity(this);
    }

    @OneToMany(mappedBy = "countryEntity")
    private List<ReligionEntity> religionEntities = new ArrayList<>();

    public void addReligionEntity(ReligionEntity religionEntity) {
        religionEntities.add(religionEntity);
        religionEntity.setCountryEntity(this);
    }
}
```

**FETCH 조회시 문제**
2개 이상의 OneToMany를 FETCH 조회할 경우도 생김

- JPQL
```java
@Query("SELECT co FROM CountryEntity co " +
"LEFT JOIN FETCH co.cityEntities ci " +
"LEFT JOIN FETCH co.religionEntities re")
List<CountryEntity> findAllFetch();
```

이때 findAllfetch() 메소드 동작시 아래와 같은 예외가 발생
![[Pasted image 20260131124350.png]]
**- 문제는**
JPA 구현체 중 하나인 Hibernate는 2개 이상의 OneToMany fetch를 강제로 막고 있음

**- 이유는**
CountryEntity를 기준으로 : 
City, Religion Entity가 각각 OneToMany로 연관되어 있고 
실제 데이터는 아래와 같다고 가정
![[Pasted image 20260131124723.png]]
Country : 1개
City : 3개
Religion : 2개

이를 SQL로 JOIN해서 가져온 결과는
Country를 기준으로 나머지 Religion, Country, City 데이터가 곱셈 복제가 됨
![[Pasted image 20260131125117.png]]
Hibernate는 가져온 이 SQL 데이터를 Java 객체로 변환 시키는 과정에서 중복을 제거해야 하는데,
중복을 제거하는 조건이 까다로워 이 자체를 막아둠

**1개의 OneToMany fetch는 하는데?**

기준인 CountryEntity를 id 기반 (hashCode 및 equals)로 처리 후 CityEntity는 중복이 있을 수 없음
하지만, OneToMany가 2개 이상인 경우 CityEntity에 중복이 있을 수 있음

- 하나의 OneToMany 연관이라면 
	CountryEntity 중복을 제거 시키고 연관된 CityEntity는 어차피 중복이 없음

- 하지만 OneToMany가 2개라면 
	CityEntity가 ReligionEntity 때문에 중복이 생겨 있음
	Hibernate는 List에 대해 설계 원칙을 위반하지 못하기 때문에 
	List< CityEntity >에 대해 중복 제거를 못함

**해결 1 : Set**
권장하는 방법은 아님 주의
기준 Entity의 OneToMany 연관 필드를 List<>가 아닌 Set<> 화 시키는 방법
```java
@OneToMany(mappedBy = "countryEntity")
private Set<CityEntity> cityEntities = new HashSet<>();
    
@OneToMany(mappedBy = "countryEntity")
private Set<ReligionEntity> religionEntities = new HashSet<>();
```

단점으로는 Set의 특성을 이해하면 됨

1. Set은 순서 보장 불가
2. Set을 호출하며 발생하는 중복 제거 체킹 자원 발생
3. 정확한 Id기반 hashCode equals 구현 필요

**해결 2 : 한쪽만 fetch 나머지 in절**
우선 한쪽 OneToMany는 기존과 같이 fetch로 유지

나머지쪽은 단순 Lazy로 두면 1+N 문제가 동일하게 발생하기 때문에 
Batch 설정을 통한 IN 절 처리를 수행
```java
@BatchSize(size = 250)
```

이렇게 하면 1+N 쿼리는 아니지만 1+소수 개의 쿼리로 문제를 해결할 수 있음 
(배치 사이즈는 250~500)

