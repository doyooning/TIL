## Programmers SQL
#MySQL 

---
**1. 경기도에 위치한 식품창고 목록 출력하기**

```
SELECT warehouse_id, warehouse_name, address, CASE  
    WHEN freezer_yn IS NULL THEN 'N'
    ELSE freezer_yn END AS freezer_yn
FROM food_warehouse  
WHERE address LIKE '%경기도%'  
ORDER BY warehouse_id;
```
CASE WHEN ... THEN ... ELSE ...  END
AS 테이블 이름 써서 마무리

