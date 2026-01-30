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
