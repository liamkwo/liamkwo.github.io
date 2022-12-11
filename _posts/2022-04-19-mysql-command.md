---
layout: post
emoji: 🌎
title: "알아두면 쓸모있는 MySQL 명령어 모음"
date: '2022-04-19 20:11:23'
author: Liam
categories: [ MySQL ]
image: assets/images/logo-background2.png
---

<br>
<br>

## 💡 Intro

저는 아직 MySQL초보이기 때문에, 여러 명령어를 사용해 놓고도 시간이 조금 지나면 까먹을 때가 종종 있습니다(어....그 명령어가 뭐드라..?!?!). 그래서 제가 생각하기에 평소에 MySQL을 사용하다가,  알고있으면 쓸모있는 MySQL문법을 이곳에 간단하게 정리해 보려고 합니다.🙌


<br>
<br>


## 🔎 알아두면 쓸모있는 MySQL 명령어 모음

```py
# 기존 컬럼에 AUTO_INCREMENT, PK 속성 추가
# 문법: ALTER TABLE [테이블명] MODIFY [컬럼명] [타입] [속성]
ALTER TABLE table MODIFY id BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY;

# auto_increment 다시 1로 세팅
alter table <table name> auto_increment = 1;



  
# 기존 테이블에 신규 컬럼 추가
# 문법: ALTER TABLE [테이블명] ADD [컬럼명] [타입] [속성]
ALTER TABLE table ADD name VARCHAR(30) NOT NULL;




# 문자열에 공백 제거
# TRIM - 문자열 좌우 공백 제거
SELECT TRIM(' aabbccbbaa ');

# TRIM - 문자열 좌측 공백 제거 (LEADING)
SELECT TRIM(LEADING FROM ' aabbccbbaa ');

# TRIM - 문자열 우측 공백 제거 (TRAILING)
SELECT TRIM(TRAILING FROM ' aabbccbbaa ');




# 여러 문자열을 구분하여 하나의 문자열로 합치기
SELECT CONCAT_WS(' ', 'FirstName', 'LastName') full_name FROM table;

# TRIM과 CONCAT_WS 응용
SELECT CONCAT_WS(' ', trim(FirstName), trim(LastName)) full_name FROM table;




# 데이터 베이스에 속한 테이블들의 각종 정보 확인
show table status from `database`;




# CASE문 : MySQL에서 CASE문은 프로그래밍 언어에서 스위치(switch)문과 비슷하지만, 다수의 조건에 하나의 반환 값은 동작하지 않는다.
# 문법: SELECT CASE [컬럼명] WHEN [조건] TEHN [결과값] WHEN [조건] TEHN [결과값] ... ELSE [결과값] END FROM [테이블명]
SELECT CASE column WHEN 'N' THEN "실패" WHEN 'Y' THEN "성공" END as success_column FROM table;




# 특정 문자로 시작하는 데이터 검색
# 문법: SELECT [필드명] FROM [테이블명] WHERE [필드명] LIKE '특정 문자열%;
SELECT * FROM member WHERE name LIKE '홍%

# 특정 문자로 끝나는 데이터 검색
# 문법: SELECT [필드명] FROM [테이블명] WHERE [필드명] LIKE '% 특정 문자열;
SELECT * FROM member WHERE name LIKE '% 길동;

# 특정 문자 포함  데이터  검색
# 문법: SELECT [필드명] FROM [테이블명] WHERE [필드명] LIKE '% 특정 문자열%;
SELECT * FROM member WHERE name LIKE '% 길동%;




# 페이징 시 쿼리 - LIMIT: 행을 얼마나 가져올지, OFFSET: 어디서 부터 가져올지
# 숫자 만큼의 행을 출력
# 문법: SELECT * FROM 테이블명 ORDERS LIMIT 숫자;
SELECT * FROM USER orders LIMIT 10;
  
# (B+1) 행 부터 A 행 만큼 출력
# 문법: SELECT * FROM 테이블명 ORDERS LIMIT 숫자(A) OFFSET 숫자(B);
SELECT * FROM USER ORDERS LIMIT 10 OFFSET 10;

# (A+1)부터 B개의 행을 출력
# 문법: SELECT * FROM 테이블명 ORDER LIMIT 숫자(A), 숫자(B);
SELECT * FROM USER ORDERS LIMIT 0, 10;




# DB 테이블 구조 및 데이터 복사
# 테이블 구조 복사
CREATE TABLE IF NOT EXISTS `new_table` LIKE `old_table`;

# 테이블 구조 및 데이터 복사
CREATE TABLE IF NOT EXISTS `new_table` SELECT * FROM `old_table`;

# 테이블 데이터 복사
CREATE TABLE IF NOT EXISTS `new_table` SELECT * FROM `old_table`;

# 테이블 데이터 부분 복사
INSERT INTO `new_table` (컬럼1 [, 컬럼2 ...]) SELECT 컬럼1 [, 컬럼2 ...] FROM `old_table`;



```

<br>

페이징을 처리할 수 있는 쿼리인 [OFFSET]에 대해서는 다음 포스팅에 조금 더 자세히 다루어 보도록 하겠습니다.😀


<br>
<br>


**[참고자료]**
- [https://zzang9ha.tistory.com/295](https://zzang9ha.tistory.com/295)
- [https://extbrain.tistory.com/](https://extbrain.tistory.com/)
- [https://pangtrue.tistory.com/170](https://pangtrue.tistory.com/170)
