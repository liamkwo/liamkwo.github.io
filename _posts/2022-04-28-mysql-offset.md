---
layout: post
emoji: 🌎
title: "MySQL offset 활용"
date: '2022-04-28 19:48:52'
author: Liam
categories: [ MySQL, Python ]
image: assets/images/logo-background2.png
---

<br>
<br>

## 💡 Intro

API를 만들다보면 데이터에 페이징 처리를 해야하는 경우가 종종 있습니다. 그래서 오늘은 MySQL의 Limit과 offset으로 쉽게 페이징 처리를 하는 방법을 포스팅해보고자 합니다.🙌


<br>
<br>


## 🔎 MySQL의 offset

limit이 처음부터 설정된 개수 까지의 데이터를 보여준다면, offset은 시작점부터 설정된 몇 개 이후의 데이터를 보여줍니다. 이를 이용해 쉽게 페이징 처리를 할 수 있습니다. 

<br>

offset의 기본적인 사용방법은 아래와 같습니다.

<br>

```py

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


```

<br>

다른 유용한 mysql명령어는 아래 링크에서 확인할 수 있습니다!😀

[알아두면 쓸모있는 MySQL 명령어 모음](https://liampoet.github.io/mysql-command/)

<br>
<br>

### 📃 offset으로 페이징 처리하기

먼저 한 페이지당 보여줄 데이터의 개수인 "limit"과 페이지 번호인 "page"를 받습니다.

<br>

```py


limit = body.get('limit')
page = body.get('page')


```

<br>

page에 1을 뺀 수에 limit을 곱해서 우리가 사용한 offset을 만들어 줍니다.
> 🔍첫 페이지는 0부터 시작이기에 page에 1을 빼줍니다. 
<br>
그리고 쿼리를 작성하면 간단하게 페이징이 가능한 코드를 작성할 수 있습니다.

<br>

```py


offset = (page - 1) * limit

f'''select column from table order by {sort} desc limit {limit} offset {offset};'''


```

<br>

***📝 전체 코드***

```py


sort = body.get('sort')
limit = body.get('limit')
page = body.get('page')
offset = (page - 1) * limit

result = f'''select column from table order by {sort} desc limit {limit} offset {offset};'''
cur.execute(result)

...


```

<br>
<br>


**[참고자료]**
- [hhttps://liampoet.github.io/mysql-command/](https://liampoet.github.io/mysql-command/)
