---
layout: post
emoji: 🔍
title: "jQuery 정규식 [id^=] 사용해보기"
date: '2022-04-10 19:25:44'
author: Liam
categories: [ Javascript, jQuery ]
image: assets/images/jquery-regex/regex.png
---

<br>
<br>

## 💡 Intro


- FastAPI로 관리자페이지를 제작 중, 앞부분이 동일한 id로 끝나는 엘리먼트들에 대한 접근이 필요해졌습니다.
- [jQuery](https://jquery.com/)에 다양한 정규식이 있다는 것은 알고 있었지만, [id^=]라는 정규식은 이번에 처음 접하게 되었습니다. 사용법도 매우 간단해서 짧게 포스팅 해보려 합니다.🙌


<br>
<br>


## 🔎 jQuery정규식이란?

***정규표현식*** :
- 특정한 규칙을 가진 문자열의 직합을 표현하는데 사용하는 형식 언어를 [정규표현식](https://www.nextree.co.kr/p4327/)이라고 합니다.
- jQuery에도 search(), replace()등 다양한 정규식이 존재하는데, 기초적인 예제는 아래와 같습니다.

***정규표현식 예제***

![정규식예제]({{ site.baseurl }}/assets/images/jquery-regex/regex.png)*이미지 1. 정규식예제*

```js
    var re = /a/         --a 가 있는 문자열
    var re = /a/i        --a 가 있는 문자열, 대소문자 구분 안함
    var re = /apple/    -- apple가 있는 문자열
    var re = /[a-z]/    -- a~z 사이의 모든 문자
    var re = /[a-zA-Z0-9]/    -- a~z, A~Z 0~9 사이의 모든 문자
    var re = /[a-z]|[0-9]/  -- a~z 혹은 0~9사이의 문자
    var re = /a|b|c/   --  a 혹은 b 혹은 c인 문자
    var re = /[^a-z]/  -- a~z까지의 문자가 아닌 문자("^" 부정)
    var re = /^[a-z]/  -- 문자의 처음이 a~z로 시작되는 문장
    var re = /[a-z]$/  -- 문자가 a~z로 끝남
    var re = /s$/;          -- 공백체크
    var re = /^ss*$/;   -- 공백문자 개행문자만 입력 거절
    var re = /^[-!#$%& amp;'*+./0-9=?A-Z^_a-z{|}~]+@[-!#$%&'*+/0-9=?A-Z^_a-z{|}~]+.[-!#$%& amp;'*+./0-9=?A-Z^_a-z{|}~]+$/; --이메일 체크
    var RegExpHG = "(/[ㄱ-ㅎ|ㅏ-ㅣ|가-힣]/)";  -- 한글 제거  
    var RegExpJS = "<script[^>]*>(.*?)</script>";  -- 스크립트 제거 

```

<br>
<br>


## 📚 [id^=]사용하기

### 📕 1. id가 id_로 시작하는 엘리먼트들 접근

***📝 Html***

```html


<div>
    <button id="id_1" value="id_1"></button>
    <button id="id_2" value="id_2"></button>
    <button id="id_3" value="id_3"></button>
    <button id="id_4" value="id_4"></button>
    <button id="id_5" value="id_5"></button>
    <button id="id_6" value="id_6"></button>
    <button id="id_7" value="id_7"></button>
    <button id="id_8" value="id_8"></button>
    <button id="id_9" value="id_9"></button>
    <button id="id_10" value="id_10"></button>
    <button id="id_11" value="id_11"></button>
    <button id="id_12" value="id_12"></button>
    <button id="id_13" value="id_13"></button>
    <button id="id_14" value="id_14"></button>
    <button id="id_15" value="id_15"></button>
    <button id="id_16" value="id_16"></button>
</div>


```

<br>

***📝 jQuery***

```js
$("[id^=id_]").click(function(){
    //id가 id_로 시작하는 엘리먼트들 접근
});
```

<br>

### 📘 2. id가 id_로 끝나는 엘리먼트들 접근

***📝 Html***

```html


<div>
    <button id="1_id" value="1_id"></button>
    <button id="2_id" value="2_id"></button>
    <button id="3_id" value="3_id"></button>
    <button id="4_id" value="4_id"></button>
    <button id="5_id" value="5_id"></button>
    <button id="6_id" value="6_id"></button>
    <button id="7_id" value="7_id"></button>
    <button id="8_id" value="8_id"></button>
    <button id="9_id" value="9_id"></button>
    <button id="10_id" value="10_id"></button>
    <button id="11_id" value="11_id"></button>
    <button id="12_id" value="12_id"></button>
    <button id="13_id" value="13_id"></button>
    <button id="14_id" value="14_id"></button>
    <button id="15_id" value="15_id"></button>
    <button id="16_id" value="16_id"></button>
</div>


```

<br>

***📝 jQuery***

```js


$("[id$=_id]").click(function(){
    //id가 _id로 끝나는 엘리먼트들 접근
});


```


<br>
<br>


**[참고자료]**

- [https://www.nextree.co.kr/p4327/](https://www.nextree.co.kr/p4327/)
- [https://dotheright.tistory.com/165](https://dotheright.tistory.com/165)
- [https://develop88.tistory.com/entry/Jquery-정규식-예제](https://develop88.tistory.com/entry/Jquery-정규식-예제)
