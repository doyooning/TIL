## Programmers SQL
#MySQL 

------------------------------------
**1. 연도별 대장균 크기의 편차 구하기**

```
-- 1.  
select year(differentiation_date) as year,  
    (max(size_of_colony) over (partition by year(differentiation_date))) - size_of_colony as year_dev,  
    id  
from ecoli_data  
order by year, year_dev;
```
`max(size_of_colony) over (partition by year(differentiation_date))`  
   : 그룹 별(= year(differentiation_date)), 
 최댓값 반환(=max(size_of_colony) over ...

```
-- 2.  
with base as ( -- with ... as (...) : 가상의 테이블 생성  
    select  
        year(differentiation_date) as y,  
        id,  
        size_of_colony,  
        max(size_of_colony) over (partition by year(differentiation_date)) as y_max  
    from ecoli_data  
)  
select  
    y as `year`,  
    (y_max - size_of_colony) as year_dev,  
    id  
from base  
order by y, year_dev;
```
`with ... as (...`) 
: 가상의 테이블 생성
base 라는 이름의 가상 테이블을 as() 내부의 쿼리문을 따라 구성
