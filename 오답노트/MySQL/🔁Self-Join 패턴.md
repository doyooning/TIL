## Programmers SQL

---
**1. 부모의 형질을 모두 가지는 대장균 찾기**

```
SELECT e.id, e.genotype, p.genotype AS parent_genotype  
FROM ecoli_data e  
    JOIN ecoli_data p ON e.parent_id = p.id  
WHERE (e.genotype & p.genotype) = p.genotype  
ORDER BY e.id;
```
self-join 패턴
: 부모-자식처럼 상위-하위 관계 형성시 사용  
메인으로 쓸 테이블을 e, 셀프조인할 테이블을 p로 하고 
(e : 자식, p : 부모)
메인의 parent_id를 서브의 id와 연결

