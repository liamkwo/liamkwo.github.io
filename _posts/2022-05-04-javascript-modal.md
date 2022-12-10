---
layout: post
emoji: 📋
title: "자바스크립트 예쁜 모달창 만들기"
date: '2022-05-04 11:01:30'
author: Liam
categories: [ Javascript, jQuery, HTML ]
image: assets/images/javascript-modal/modal1.jpg
---

<br>
<br>

## 💡 Intro


웹 페이지에 예쁜 모달창을 만드는 방법을 공부해보려 합니다.🙌


<br>
<br>


## 🔎 팝업창? 모달창?

***1) 팝업창*** 
- 팝업창이란, 현재 열려있는 브라우저 페이지에 **또 다른 브라우저 페이지를 띄우는 것**을 의미합니다.
- 창 + 창n의 개념을 가지며, 보통 웹 시작과 동시에 띄우는 형태로 사용합니다.
- 사용 의도 관점 - 사용자가 현재 **의도하는 목적에 상관없이 뜨는 창**

<br>

***2) 모달창***
- 모달창은 기존의 브라우저 페이지위에 새로운 브라우저 페이지를 띄우는 것이 아닌 레이어를 띄우는 것을 의미합니다.
- 기존의 페이지와 **부모-자식 관계**를 가지며, 사용자에게 중간중간 보여주는 경우로 많이 사용됩니다.
- 사용 의도 관점 - 다음 진행 혹은 특정한 행동을 처리하기 위해 **필요에 의해 사용되는 창**

<br>

***모달(Modal) vs 논모달(Non-modal)***
- 모달을 공부하면서 논모달을 알게 되었는데, 논모달은 "모달이지만 모달이 아니다"라고 그대로 번역할 수 있습니다. 
- 일반적으로 모달은 부모 화면을 사용자가 컨트롤할 수 없는데, 논모달은 **부모 화면을 사용자가 컨트롤**할 수 있습니다.(예를 들어, 구글메세지 보내기에서 <u>편지쓰기</u>는 논모달이라고 할 수 있습니다.)

<br>

![모달(Modal)과 논모달(Non-modal)]({{ site.baseurl }}/assets/images/javascript-modal/modal1.jpg)*이미지 1. 모달(Modal)과 논모달(Non-modal)*

<br>

모달은 위의 이미지와 같이 다양하게 구분되어질 수 있습니다. 또한, 모달/논모달이 구분되어 지는 것처럼 여러 상횡에 따라 유동적으로 바뀔 수 있습니다.
UXPlanet에 잘 정리된 의사결정 Framwork가 있는데, [JeongChanho](https://blog.toktokhan.dev/%EC%9D%B4%EA%B1%B0%EB%8F%84-%EB%AA%A8%EB%8B%AC-%EC%A0%80%EA%B1%B0%EB%8F%84-%EB%AA%A8%EB%8B%AC-%EC%9D%B4%EA%B2%8C-%EB%AA%A8%EB%9E%8C-4424a5a0c769)님이 잘 번역해서 정리해둔 이미지가 있어서 아래에 사진을 올려보았습니다.

<br>

![모달 의사결정 Framwork]({{ site.baseurl }}/assets/images/javascript-modal/modal2.png)*이미지 2. 모달 의사결정 Framwork(출처 : JeongChanho)*

<br>
<br>


## 📚 css와 자바스크립트로 모달창 만들기


### 📕 1. 모달창 기초 생성하기


***📝 CSS***

```css


.modal-dim { 
    display: none; 
    position: fixed; 
    top: 0; 
    left: 0; 
    width: 100%; 
    height: 100%; 
    z-index: 1000; 
}
.modal-dim .dim-bg { 
    top: 0; 
    left: 0;
    right: 0;
    width: 100%; 
    height: 100%; 
    margin: 0 auto; 
    background: #000; 
    opacity: .4; 
}
.modal-dim .modal-layer { 
    display: block; 
}
.modal-dim .modal-top { 
    position: sticky; 
    top: 0; 
    left: 0; 
    right: 0; 
    z-index: 10; 
}
.modal-dim .modal-tit { 
    font-size: 20px; 
    font-weight: 700; 
    color: #fff; 
}
.modal-close { 
    width: 28px; 
    height: 28px; 
    border: none; 
    background: url('../img/close.png') center center no-repeat; 
    background-size: contain; 
}

.board { 
    margin-bottom: 20px; 
    padding: 20px; 
    width: 100%;
    height: 500px;
    border: 1px solid #d6dae1; 
    box-sizing: border-box; 
    background: #fff; 
}


```

<br>

- **modal-close**는 모달창 우측 상단의 **X**버튼을 생성하기 위해 만들었습니다. 원하시는 모양의 이미지를 삽입하면 됩니다.
- **.modal-dim .dim-bg**는 모달창이 켜졌을 때, 주위를 어둡에 하기 위해 <u>[opacity](https://www.w3schools.com/cssref/css3_pr_opacity.asp)</u>를 사용했습니다.

<br>

script에 <u>show()</u>와 <u>hide()</u>로 모달창을 끄고 킬 수 있는 기능을 생성했습니다.

<br>

***📝 Javascript로 모달창 끄고 키기***

```js


$(function () {

    $(".js-modal-open").click(function () {
        $(".js-modal").show();
    });
    $(".js-modal-close").click(function () {
        $(".js-modal").hide();
    });

})


```

<br>
<br>

### 📘 2. 모달창 동작하기

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
            <div class="modal-tit">모달창 만들기</div>
            <button type="button" class="modal-close js-modal-close"></button>
        </div>

        <div class="modal-container">
            <div class="board-v2">
                <font size="40">모달창 만들기</font>
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


<!--Javascript-->
<!--id를 이용해 모달창 켜기-->
<script type="text/javascript">

    $(function () {

        $('#modal_test').on('click',  function (e) {
            $(".js-modal").show();
        })

    });

</script>


```

<br>

#### 🔮 결과(gif)

<br>

![모달창 결과]({{ site.baseurl }}/assets/images/javascript-modal/modal3.png)*이미지 3. 모달창 결과*

<br>

> 끝맺음
이렇게 CSS와 자바스크립트를 사용한다면 모달 기능을 쉽게 만들 수 있습니다. 또한 모달을 사용하게 되면 코드가 복잡해질 수 있다고 하는데, 모듈 또는 컴포넌트로 분리하여 사용하면 손쉽게 해결할 수 있습니다.


<br>
<br>


**[참고자료]**
- [JeongChanho - 이거도-모달-저거도-모달-이게-모람](https://blog.toktokhan.dev/%EC%9D%B4%EA%B1%B0%EB%8F%84-%EB%AA%A8%EB%8B%AC-%EC%A0%80%EA%B1%B0%EB%8F%84-%EB%AA%A8%EB%8B%AC-%EC%9D%B4%EA%B2%8C-%EB%AA%A8%EB%9E%8C-4424a5a0c769)
- [http://yoonbumtae.com/?p=3632](http://yoonbumtae.com/?p=3632)
