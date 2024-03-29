---
layout: post
emoji: 🗃
title: "form-data? x-www-form-urlencoded? raw?"
date: '2022-05-17 18:48:57'
author: Liam
categories: [ Json, Postman ]
image: assets/images/content-type/form_2.gif
---

<br>
<br>

## 💡 Intro


[Postman](https://www.postman.com/)을 사용하다 보면 여러 가지의 Content-Type이 있습니다. form-data, x-www-form-urlencoded, Raw, Binary, GraphQL이 있는데, 오늘은 이 요청 유형에 대해 알아보려 합니다.


<br>
<br>


## 🔎 들어가기전 Content-Type이란?

***📝 Content-Type*** : 
- API 연동시에, 보내는 자원의 형식을 명시하기 위해서 HTTP Header에 실리는 정보를 content-type이라고 합니다. 즉, api 요청 시 request에 실어 보내는 data의 type정보를 표현합니다. 
- content-type에 대한 자세한 정보는 NAMP 님의 [HTTP Content-Type 정리](https://surpassing.tistory.com/697)에 잘 정리되어져 있으니 정보가 필요하신분은 방문해주세요!

<br>
<br>


## 📚 POST 요청 유형


### 📕 1. form-data(양식 데이터)
이름에서 알 수 있듯이 양식 데이터는 양식을 채울때 입력 한 세부 정보와 같이 양식 내부에 래핑하는 데이터를 보내는데 사용됩니다. 이러한 세부 정보는 Key가 보내는 항목의 "이름" 이고 Value는 실제 값을 입력하면 됩니다. **즉, Key-Value 쌍으로 작성하여 전송**됩니다.

<br>

![form_1.png]({{ site.baseurl }}/assets/images/content-type/form_1.png)

<br>

또한, 텍스트 뿐만아니라 파일과 함께 일부 데이터를 서버에 전송하기를 원한다면 아래 GIF와 같이 form-data에서 수행할 수도 있습니다.

<br>

![form_2.gif]({{ site.baseurl }}/assets/images/content-type/form_2.gif)

<br>
<br>

### 📘 2. x-www-form-urlencoded
요즘은 많이 사용하지는 않지만 종종 사용하는 방식입니다. form-data와 거의 동일한 목적으로 사용되며 form-data와 상당히 유사합니다. form-data와의 차이점은 **x-www-form-urlencoded는 보내는 데이터를 URL인코딩 이라고 부르는 방식으로 인코딩 후에 웹서버로 보냅니다.** 또한, Raw는 본문 메시지가 요청 본문을 나타내는 [비트스트림(bitstream)](http://terms.tta.or.kr/dictionary/dictionaryView.do?subject=%EB%B9%84%ED%8A%B8%EC%8A%A4%ED%8A%B8%EB%A6%BC)으로 표시됨을 의미합니다. 이러한 비트는 문자열 서버로 해석됩니다.

<br>

![x-form_1.png]({{ site.baseurl }}/assets/images/content-type/x-form_1.png)

<br>
<br>

### 📒 3. raw 
이름 그대로 날것이라고 해석할 수 있습니다. raw를 제외한 다른 타입들의 경우 각각 용도에 맞게 Postman에서 알아서 처리를 해줍니다. 하지만 **raw타입은 문자 그대로를 보내기 때문에 개발자가 요청 형식에 맞게끔 값을 써주어야 합니다.** 

<br>

![raw_1.png]({{ site.baseurl }}/assets/images/content-type/raw_1.png)

<br>

raw를 선택하면 위 사진과 같이 세부 항목을 선택할 수 있습니다. Text, JavaScript, JSON, HTML, XML의 형식을 지원합니다. 물론 형식에 틀리게 작성한다면 틀렸다고 알려줍니다. 

<br>
<br>

### 📗 4. binary
Binary는 수동으로 입력할 수 없는 형식으로 데이터를 전송하도록 설계되었습니다. 컴퓨터의 모든 것이 바이너리로  변환되기 때문에 이미지, 파일 등과 같이 수동으로 작성할 수 없는 상황일 때 이러한 옵션을 사용합니다. 즉, 텍스트 없이 이미지나 영상, 오디오 파일 등을 보낼 때 사용합니다.

<br>

![binary_1.gif]({{ site.baseurl }}/assets/images/content-type/binary_1.gif)

<br>
<br>

### 📙 5. GraphQL
GraphQL이란 Facebook에서 만든 어플리케이션 레이어 쿼리 언어라고 합니다. 웹 클라이언트가 서버로부터 데이터를 효율적으로 가져오기 위한 언어입니다. GraphQL은 [GraphQL 개념잡기](https://americanopeople.tistory.com/330)에서 더 자세하게 보실 수 있습니다.


<br>
<br>


> 끝맺음
이렇게 오늘은 여러 가지의 요청 유형에 대해 알아보았습니다. 저는 실제로 사용할 때 JSON의 요청이 많다 보니, raw와 form-data를 가장 많이 사용하게 되더군요. 평소에는 생각지도 못한 부분들이었는데, 갑자기 ***이것들에 대한 차이가 뭐지..?*** 라는 생각을 가지게 되었고, 그렇게 이번에 공부를 하며 자료를 정리하다보니 생각보다 흥미로운 부분들이 많았던 것 같습니다.😊


<br>
<br>


**[참고자료]**
- [HTTP Content-Type 정리](https://surpassing.tistory.com/697)
- [https://jw910911.tistory.com/117](https://jw910911.tistory.com/117)
- [Postman을 사용한 POST 요청](https://testmanager.tistory.com/342)
- [(GraphQL) GraphQL 개념잡기](https://americanopeople.tistory.com/330)
