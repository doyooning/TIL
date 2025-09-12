#MySQL 

--------
##### 1. 저장 프로시저(Stored Procedure)
MySQL에서 제공되는 프로그래밍 기능
쿼리문의 집합(=쿼리 모듈화), 특정 동작을 일괄 처리하기 위한 용도로 사용

기본 형식:
```
DELIMITER $$
CREATE PROCEDURE 프로시저 이름(IN(입력값) 또는 OUT(출력값) 파라미터)
BEGIN

	# 이 부분에 SQL 프로그래밍 코딩
	
END $$
DELIMITER ;
CALL 프로시저 이름();

DROP PROCEDURE 프로시저 이름;  -- 프로시저 삭제 
```

프로시저 내부에서 사용할 변수 선언 및 할당 가능

다중 분기 활용
: IF문, CASE문

```
# CASE문
DELIMITER $$
CREATE PROCEDURE scoretest3()
BEGIN
	DECLARE score int;
    DECLARE grade char(1);
    SET score = 77;
    
    CASE 
		WHEN (score >= 90) THEN
			SET grade = 'A';
		WHEN (score >= 80) THEN
			SET grade = 'B';
        WHEN (score >= 70) THEN
			SET grade = 'C';    
		WHEN (score >= 60) THEN
			SET grade = 'D';
		WHEN
			SET grade = 'E';
	END CASE;
	SELECT concat('취득점수 ==> ', score) AS 취득점수, concat('학점 ==> ', grade) AS 학점;
END $$
DELIMITER ;
CALL scoretest3();
```
구문 안에서 SELECT 해준 테이블 또는 속성을 출력해준다.

복잡한 테이블 구조(JOIN 관계)에서의 프로시저
: ex)
`구매테이블(buytbl)에 구매액(price * amount)이 1500원 이상인 고객은 'VIP',` 
`1000원 이상인 고객은 'GOLD', 500원 이상인 고객은 'SILVER', 500원 미만인 고객은 'CUSTOMER',`
`가입은 되었지만 구매 실적이 없는 고객은 'GHOST'`

```
DELIMITER $$
CREATE PROCEDURE UserBuyProductList()
BEGIN
    SELECT u.userid, u.name, sum(price*amount) as 총구매액,
        CASE    WHEN (sum(price*amount) >=1500) THEN 'VIP'
				WHEN (sum(price*amount) >=1000) THEN 'GOLD'
			    WHEN (sum(price*amount) >=500) THEN 'SILVER'
                WHEN (sum(price*amount) < 500) THEN 'CUSTOMER'
                ELSE 'Ghost'
        END AS 'GRADE'
    FROM usertbl u            -- 또는 buytbl b 
    LEFT JOIN buytbl b        --      RIGHT OUTER JOIN usertbl u
        ON b.userid = u.userid
GROUP BY u.userid, u.name
ORDER BY sum(price*amount) DESC;
END $$
DELIMITER ;
CALL UserBuyProductList();
```

반복문을 구현한 프로시저
: WHILE ... DO ... END WHILE

```
DELIMITER $$
CREATE PROCEDURE whileProc()
BEGIN
    -- 변수 선언
    DECLARE i int;
    DECLARE total int;
    -- 값 할당 (변수 초기화)
    SET i = 1;
    SET total = 0;
    -- 수행문
  myWhile: WHILE (i <= 100) DO      -- WHILE에 label지정 : myWhile
		IF(i%7 = 0) THEN
		    SET i = i + 1;
	        ITERATE myWhile;            -- 지정한 label문으로 가서 계속 진행
		END IF;
	    SET total = total + i;
	    IF(total > 1000) THEN
	        LEAVE myWhile;           -- 지정한 label문을 떠나라! while 종료
	    END IF;
    SET i = i + 1;
    END WHILE;
    -- 출력문
    SELECT total AS '1-100 누적총합';
END $$
DELIMITER ;
CALL whileProc();
```

