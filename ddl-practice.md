## DDL 실습

 - 이 실습은 Docker 기반으로 돌고 있는 [ClickHouse](https://hub.docker.com/repository/docker/keeyong/clickhouse-with-tables)을 대상으로 합니다. 
 - ClickHouse에 있는 clickhouse-client shell을 사용합니다
   - ClickHouse docker container의 이름이 clickh라고 가정하겠습니다
   - 이 shell에 연결하려면 다음 명령을 터미널에서 실행합니다
```
docker exec -it clickh clickhouse-client 
```

### Database 생성

몇 개의 데이터베이스가 이미 만들어져 있지만 (raw_data, analytics, adhoc) 새로 만들려면 아래 SQL을 shell에서 실행합니다.
```
CREATE DATABASE 데이터베이스이름; 
```

### Table 생성 (PRIMARY KEY, Nullable, default)

```
CREATE TABLE adhoc.test
(
    id int PRIMARY KEY,
    value Nullable(int) default 100,
    location varchar(256) default 'seoul'
);
-- 기본키 유일성이 안 지켜지는 것 확인
INSERT INTO adhoc.test VALUES (1, 2, 3), (1, 3, 4);
SELECT * FROM adhoc.test;

-- 기본값 사용이 되는 것
INSERT INTO adhoc.test VALUES (2, , );
SELECT * FROM adhoc.test;

-- NULL 값 허용 여부가 어떻게 적용되는지?           
INSERT INTO adhoc.test VALUES (3, NULL, NULL);
SELECT * FROM adhoc.test; 
```

### CTAS 실습

```
-- 먼저 SELECT 실행해보기
SELECT DISTINCT channel
FROM raw_data.user_session_channel;

-- CTAS로 테이블 만들기
CREATE TABLE analytics.channel
PRIMARY KEY channel AS
SELECT DISTINCT channel
FROM raw_data.user_session_channel;

-- 만들어진 테이블의 스키마 확인
DESCRIBE TABLE analytics.channel
```

### RENAME TABLE과 ALTER TABLE 실습

```
-- 테이블 이름 바꾸기
RENAME TABLE adhoc.test TO adhoc.test2;

-- 컬럼 추가해보기
ALTER TABLE adhoc.test2 ADD COLUMN new_field varchar(512);
DESCRIBE adhoc.test;

-- 컬럼 이름바꾸기 (PRIMARY KEY, SHARD KEY에는 적용못함)
ALTER TABLE adhoc.test2 RENAME new_field to renamed_field;
DESCRIBE adhoc.test;

-- 컬럼 삭제하기
ALTER TABLE adhoc.test2 DROP COLUMN renamed_field;
DESCRIBE adhoc.test;
```

### Table 삭제

```
-- 테이블 삭제
DROP TABLE analytics.channel;

-- id=1인 레코드들을 adhoc.test2에서 삭제하기
SELECT * FROM adhoc.test2;
ALTER TABLE adhoc.test2 DELETE WHERE id=1;
SELECT * FROM adhoc.test2;

-- 모든 레코드들을 adhoc.test2에서 삭제하기
ALTER TABLE adhoc.test2 DELETE WHERE 1=1;
SELECT * FROM adhoc.test2;
DESCRIBE adhoc.test2;
