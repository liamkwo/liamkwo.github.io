---
layout: post
emoji: 🤔🙌
title: "(문제해결)ERROR 1227 (42000) at line 18 - Access denied; you need (at least one of) the SUPER privilege(s) for this operation" 
date: '2022-06-04 16:20:18'
author: Liam
categories: [ MySQL ]
image: assets/images/logo-background2.png
---

<br>
<br>

## 💡 Intro


MySQL에서 DB의 백업파일을 import하려는데 다음과 같은 `ERROR 1227 (42000) at line 18 - Access denied; you need (at least one of) the SUPER privilege(s) for this operation`라는 에러가 발생했습니다. 


<br>
<br>


## 📚 해결방법


구글에서 정보를 찾아보니 **프로시저의 DEFINER** 때문이었는데, RDS가 제공하는 MySQL 서버가 사용자가 아닌 다른 DEFINER가 지정된 sql 파일을 허용하지 않기 때문이라고 합니다. 즉, DEFINER의 계정으로 import하지 않아서 생기는 문제였습니다.

<br>

해결방법은 너무 간단했습니다. 그냥 definer를 삭제하여 import시키면 기본 definer로 자동으로 설정이 된다고 합니다,.

<br>

모조리 삭⭐️제⭐️

<br>

[jjomnoon블로그](https://jjomnoon-diary.tistory.com/15)님의 말대로 찾을 내용에 DEFINER=DEFINER=`root`@`localhost`, 바꿀내용에 공백을 입력하여 전부 삭제하여 해결했습니다.


<br>
<br>


**[참고자료]**
- [jjomnoon블로그](https://jjomnoon-diary.tistory.com/15)
- [https://m.blog.naver.com/tpgpfkwkem0/221997595407](https://m.blog.naver.com/tpgpfkwkem0/221997595407)
- [stackoverflow-questions](https://stackoverflow.com/questions/44015692/access-denied-you-need-at-least-one-of-the-super-privileges-for-this-operat)
