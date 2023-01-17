---
layout: post
emoji: 🛎
title: "jQuery로 버튼 클릭 이벤트 만들기"
date: '2022-05-09 22:33:22'
author: Liam
categories: [ Javascript, jQuery, HTML ]
image: assets/images/jquery-btn-event/button2.gif
---

<br>
<br>

## 💡 Intro


- 저번에 포스팅한 모달창 위에서 진행됩니다! 오늘은 버튼을 클릭했을 때 클릭되는 효과와 클릭한 데이터가 입력되고 지워지는 기능을 만들어 보려고 합니다. 
- 모달창 생성법은 [자바스크립트 예쁜 모달창 만들기](https://liamkwo.github.io/javascript-modal/)포스트를 확인해주세요!


<br>
<br>


## 📚 버튼클릭이벤트 만들기


### 📕 1. CSS로 버튼 이미지 만들기


***📝 CSS***

```css


.btn-event {
    margin: 20px;
    position: relative;
    border: none;
    display: inline-block;
    padding: 15px 30px;
    border-radius: 15px;
    font-family: "paybooc-Light", sans-serif;
    box-shadow: 0 15px 35px rgba(0, 0, 0, 0.2);
    text-decoration: none;
    font-weight: 600;
    font-size: medium;
    transition: 0.25s;
    box-sizing: border-box;
    background-color: #77af9c;
    color: #ffffff;}
    button.on {
        color: white;
        font-size: large;
        background-color: #275f4d;
    }


```

<br>

- **transition**는 CSS 프로퍼티의 값이 변화할 때, 프로퍼티 값의 변화가 일정 시간(duration)에 걸쳐 일어나도록 하게합니다. 저는 버튼이 변화할 때 조금 더 부드러운 모션을 주기위해 `transition: 0.25s;`로 설정했습니다.
- **button.on** : 보통 `button:hover`을 사용하는데, 저는 후에 javascript로 `addClass("on");`로 이벤트를 주기위해 **button.on**을 사용했습니다.

<br>

#### 🔮 결과(gif)

<br>

![button1.gif]({{ site.baseurl }}/assets/images/jquery-btn-event/button1.gif)

<br>
<br>

### 📘 2. HTML작성하기

<br>

작성한 css와 javascript로 모달창 코드를 작성합니다. 그리고 모달창을 켜기위한 버튼을 생성합니다.

<br>

***📝 HTML 전체 메인코드***

```html


<!--모달창-->
<div class="modal-dim js-modal">
    <div class="dim-bg"></div>

    <div class="modal-layer">
        <div class="modal-top">
            <div class="modal-tit">버튼 이벤트 동작하기</div>
            <button type="button" class="modal-close js-modal-close"></button>
        </div>
        
        <div class="modal-top" style="height:100px">
            <textarea id="select_button_text" placeholder="" 
                    class="inp-frm js-count" data-count="400" readonly></textarea>
        </div>

        <div class="modal-container">
            <div class="">
                <div class="board">
                    <div class="tit-menu">버튼 이벤트 동작하기</div>

                    <div class="frm-line-v2"></div>

                    <div class="frm-area" id="btn-button">
                        <button class="btn-event" id="button_1" value="id_1">1</button>
                        <button class="btn-event" id="button_2" value="id_2">2</button>
                        <button class="btn-event" id="button_3" value="id_3">3</button>
                        <button class="btn-event" id="button_4" value="id_4">4</button>
                        <button class="btn-event" id="button_5" value="id_5">5</button>
                        <button class="btn-event" id="button_6" value="id_6">6</button>
                        <button class="btn-event" id="button_7" value="id_7">7</button>
                        <button class="btn-event" id="button_8" value="id_8">8</button>
                        <button class="btn-event" id="button_9" value="id_9">9</button>
                        <button class="btn-event" id="button_10" value="id_10">10</button>
                        <button class="btn-event" id="button_11" value="id_11">11</button>
                        <button class="btn-event" id="button_12" value="id_12">12</button>
                        <button class="btn-event" id="button_13" value="id_13">13</button>
                        <button class="btn-event" id="button_14" value="id_14">14</button>
                        <button class="btn-event" id="button_15" value="id_15">15</button>
                        <button class="btn-event" id="button_16" value="id_16">16</button>
                    </div>

                </div>
            </div>
        </div>

    </div>
</div>


<!--모달창 켜기위한 버튼-->
<div>
    <div>모달창 만들기</div>

    <div>
        <div style="float:left; width:20%;">
            <input type="button" value="modal_test" id="modal_test">
        </div>
    </div>
</div>


```

<br>
<br>

### 📒 3. Javascript작성하기

<br>

Javascript로 버큰 끄고/키기 이벤트와 텍스트가 입력되고 지워지는 코드를 작성합니다.

<br>

***📝 HTML 전체 메인코드***

<br>

```js


$(function () {

    let check = {
                checked_cnt1 : 0, checked_cnt2 : 0, checked_cnt3 : 0, 
                checked_cnt4 : 0, checked_cnt5 : 0, checked_cnt6 : 0, 
                checked_cnt7 : 0, checked_cnt8 : 0, checked_cnt9 : 0, 
                checked_cnt10 : 0, checked_cnt11 : 0, checked_cnt12 : 0, 
                checked_cnt13 : 0, checked_cnt14 : 0, checked_cnt15 : 0, 
                checked_cnt16 : 0, checked_cnt17 : 0, checked_cnt18 : 0, 
                checked_cnt19 : 0, checked_cnt20 : 0, checked_cnt21 : 0, 
                checked_cnt22 : 0, checked_cnt23 : 0, checked_cnt24 : 0, 
                checked_cnt25 : 0, checked_cnt26 : 0, checked_cnt27 : 0, 
                checked_cnt28 : 0, checked_cnt29 : 0, checked_cnt30 : 0
                }

    $("[id^=button_]").click(function(){

        var cnt = $(this).attr("id").substr(7)
        
        if(check['checked_cnt'+cnt] == 0){
            check['checked_cnt'+cnt] = 1

            $(this).addClass("on");
            $('#select_button_text').append($(this).attr('value') + ',');
            $('#button').append($(this).attr('value') + ',');
        }else{
            check['checked_cnt'+cnt] = 0

            $(this).removeClass("on");
            button_text = $('#select_button_text').val();
            button_text = button_text.replace($(this).attr('value') + ',', '')
            $("#select_button_text").text(button_text);
            $("#button").text(button_text);
        }
    });

})


```  

<br>

- 먼저 동적으로 변수를 할당시켜주고 싶었으나, javascript에서는 찾지를 못해 `let check = {}`로 Dict형태로 변수들을 0값으로 할당해주었습니다.***(좋은 방법이 있으면 댓글로 남겨주세요.😥)***
- `[id^=button_]`는 button으로 시작하는 **모든 id의 값을 불러오기위해**서 사용했습니다. 자세한 내용은 [jQuery 정규식 [id^=] 사용해보기](https://liamkwo.github.io/jQuery-regex/)에서 확인해 주세요! 
- `var cnt = $(this).attr("id").substr(7)`는 **id의 끝에 붙어있는 숫자를 불러오기위해**서 사용했습니다.(예시 - button_15.substr(7) => 15)
- 그 후 'checked_cnt'에 방금 불러온 id 끝값인 cnt를 더해서 check Dict안에 있는 키 값을 만들었습니다. `check['checked_cnt'+cnt]`을 통해 check의 'checked_cnt'+cnt값이 초기에 할당된 것처럼 0일 경우 1을 할당해 주었고, `$(this).addClass("on");`를 통해서 아까 **CSS**에서 작성해둔 `button.on`을 활성화 했습니다. 그리고 그 값을 `textarea`에 전달했습니다.
- 반대로 'checked_cnt'+cnt값이 0이 아니면 0을 할당해주고 `$(this).removeClass("on");`로 `button.on`을 비활성화합니다. 그리고 `button_text = button_text.replace($(this).attr('value'))`를 통해 텍스트를 빼주었습니다.

<br>

#### 🔮 결과(gif)

<br>

![button2.gif]({{ site.baseurl }}/assets/images/jquery-btn-event/button2.gif)

<br>

> 끝맺음
오늘은 버튼클릭 이벤트를 만들어 보았습니다. CSS와 Javascript 몇줄로 간단하게 예쁜 동작을 하는 버튼을 만들 수 있었습니다. 또한, 만약 **eval() 함수**를 제외하고 동적 변수 할당에 대해 괜찮은 정보가 있으면 댓글로 남겨주시면 감사하겠습니다!😊


<br>
<br>


**[참고자료]**
- [https://myhappyman.tistory.com/123](https://myhappyman.tistory.com/123)
- [예쁜 버튼 디자인 CSS 모음](https://inpa.tistory.com/entry/CSS-%F0%9F%92%8D-%EB%B2%84%ED%8A%BC-%EB%94%94%EC%9E%90%EC%9D%B8-%EB%AA%A8%EC%9D%8C)
