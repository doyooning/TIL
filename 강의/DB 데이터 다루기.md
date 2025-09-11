#MySQL 

--------
##### MySQL 데이터 형식(거의 30개)

1. 숫자데이터 형식
	정수, 실수 등 숫자 표현
	TINYINT(1바이트)
	SMALLINT(2바이트)
	INT(4바이트)
	DECIMAL(전체자릿수, 소수점자릿수)
	FLOAT, DOUBLE : 근사값을 저장
	UNSIGNED -> 예약어, 양의정수값만 사용
		ex) TINYINT : -128 ~ 127 -> 0 ~ 255
		 SMALLINT : -32768 ~ 32767 -> 0 ~ 65535
		 
2. 문자데이터 형식
	<문자 형식>
	CHAR(n), CHARACTER(n), CHAR = CHAR(1)
		: 고정길이 문자형, n을 1부터 255까지 지정
	VARCHAR(n) : 가변길이 문자형, 1 ~ 65535
	
	<TEXT 형식>
	TEXT : 1 ~ 65535, LONGTEXT : 최대 4Gbyte
	
	<BLOB형식>
	LONGBLOB : 1 ~ 4Gbyte
	ENUM값 : 65535개의 열거형 데이터 값 저장
	SET 타입 : 최대 64개의 서로 다른 값을 저장 가능
	
	<CHAR와 VARCHAR의 차이>
	CHAR(n) -> n개의 저장공간 생성, 빈 공간 존재 가능
		-> INSERT/UPDATE 시 성능상 효율적
	VARCHAR(n) -> 글자수만큼 공간 생성 후 글자 저장
		-> 효율적인 공간 운영
		
	MySQL => UTF-8 (영문, 한글 등 타입에 따라 크기 달라짐)
	
3. 날짜와 시간 데이터 형식
	DATE(3바이트) : 1001-01-01 ~ 9999-12-31
		-> 'YYYY-MM-DD' 지정 형식
	DATETIME(8바이트) : DATE에 시분초 추가
		-> 'YYYY-MM-DD HH:MM:SS' 지정 형식

##### 데이터에 파일 업로드/다운로드

###### 1. 테이블 생성
```
create table movies(
	movie_id int auto_increment,
    ...
    movie_script longtext,
    movie_film longblob,
    constraint pk_movies_movie_id primary key (movie_id)
) default charset=utf8mb4;
```

 한글을 문제없이 처리하기 위해 기본문자셋을 utf8mb4로 지정

###### 2. 데이터 삽입
```
insert into movies values(null, '쉰들러리스트', '스필버그', '리암 니슨', 
	load_file('C:/study/ssg9/sql/movies/Schindler.txt'), 
    load_file('C:/study/ssg9/sql/movies/Schindler.mp4'));
```

`load_file()` : 해당 경로의 파일을 로드
여기까지만 해주면 실제 입력된 데이터는 NULL 인 것을 확인할 수 있다.

1.  최대 패킷 크기가 설정된 시스템 변수인 max_allowed_packet 확인
	`show variables like 'max_allowed_packet';`
	파일 크기 > 최대 패킷 크기 -> 안 담아진다
	
2. 시스템 변수 secure_file_priv 설정 값 확인
	기본 경로는 `C:\ProgramData\MySQL\MySQL Server 8.0\Uploads\`
	`show variables like 'secure_file_priv';`
	파일이 있는 경로와 같은 경로로 설정해준다
	
3. 시스템 변수 수정
	시스템 변수를 수정할 때,
	MySQL 서버 중단 + 관리자 권한으로 접근
	
	- MySQL 서버 중단
		net ... 명령어 하지 말고 그냥 설정 -> 서비스 들어가서
		MySQL 서비스 중단 하면 된다
		작업 끝나면 다시 실행 필수 !!
		
	- 관리자 권한으로 접근
		cmd에서 MySQL Server 디렉토리 -> my.ini 파일 열어준다
		파일에서 시스템 변수 검색 후 값 수정

###### 3. 데이터 저장
```
select movie_script from movies where movie_id = 1
into outfile 'C:/study/ssg9/sql/movies/Schindler_out.txt' lines terminated by '\\n';

select movie_film from movies where movie_id = 3
into dumpfile 'C:/study/ssg9/sql/movies/Mohican_out.mp4';
```
`lines terminated by '\\n'` : 줄바꿈 문자도 그대로 저장한다 = 원본 그대로 저장에 가까움

