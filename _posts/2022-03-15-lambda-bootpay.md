---
layout: post
emoji: 💳
title: "AWS Lambda에 Bootpay연동하기 - 결제 검증"
date: '2022-03-15 19:25:44'
author: Liam
categories: [ AWS, Python, Bootpay ]
image: assets/images/logo.png
---

<br>
<br>

## 💡 Intro

BootPay는 무료로 서비스되는 결제 검증 API입니다. 초기에 일반 PG 사와 별도의 계약 없이 개발을 먼저 할 수 있어서, 인앱 결제가 급하게 필요했던 저는 [BootPay](https://docs.bootpay.co.kr)를 이용하기로 했습니다. 간단하게 사용방법을 정리해 보도록 하겠습니다.(매우 간단해요!)🙌


<br>
<br>


## 🔎 Bootpay란?

부트페이는 개발자를 위한 결제연동 서비스로, 구글 애널리틱스와 같이 결제 데이터 분석 서비스를 제공합니다. 또한 웹, 앱 SDK 모두 지원하며 국내외 여러 PG(복수 선택 가능)와 결제수단을 소스코드 한 줄로 사용할 수 있으므로 불필요한 작업을 줄일 수 있습니다.


<br>
<br>


## 📚 결제 검증


부트페이 측에서는 결제 검증에 대해, 이렇게 설명하고 있습니다. 

**결제 검증을 해야하는 이유**
- 결제 연동은 Client Side에서 동작되기 때문에 결제 금액 및 결제 상태에 대한 변조가 가능하다고 합니다. 그러므로 반드시 처음 요청했던 금액과 결제가 올바르게 이루어졌는지에 대해 서버사이드에서 서버로 영수증 키를 보내 확인하는 과정을 거쳐야 합니다.

**그렇다면 결제 검증을 무조건 해야하나요?**
- 결제 검증을 하지 않더라도 PG사에서 제공하는 결제 로직을 모두 완료할 수 있습니다. 방금 말했다시피, 실제 결제가 이루어지지만 Client Side에서 언제든 변조된 데이터를 구현하신 서버로 보내 부정적인 방법으로 구매 완료 데이터를 보낼 수 있기 때문에, 결제 검증은 필수로 하는것이 좋다고 Bootpay에서는 설명하고 있습니다.

### 📕 1. Lambda layer생성


[Bootpay 결제 검증 및 취소 모듈(python)](https://github.com/bootpay/server_python)을 다운 받은 뒤, 아래와 같이 Lambda layer를 생성합니다.

<br>

![Add_layer.png]({{ site.baseurl }}/assets/images/bootpay/Add_layer.png)

<br>


### 📘 2. Add a layer

<br>

생성한 Bootpay layer를 Lambda함수에 추가해 줍니다.

<br>

![Lambda_layer.png]({{ site.baseurl }}/assets/images/bootpay/Lambda_layer.png)

<br>

그리고 import합니다.

<br>

```py
from lib.BootpayApi import BootpayApi
```

<br>
<br>

### 📒 3. 검증코드 작성하기



아래와 같이 검증코드만 작성하면 매우 간단하게 결제 검증을 끝낼 수 있습니다.😀 저는 웹결제가 없었기 떄문에 앱에서 결제 후 서버사이드에서 결제 검증만 진행하였으며, application_id와 private_key는 Bootpay 관리자에서 확인할 수 있습니다. 또한 아래 코드와 같이 결제 금액으로 비교를 해주면 되는데, 가격말고도 원하는 다른 데이터를 추가로 비교할 수 도있습니다.

<br>

```py
from lib.BootpayApi import BootpayApi

bootpay = BootpayApi(
    '[[ application_id ]]',
    '[[ private_key ]]'
)

result = bootpay.get_access_token()
if result['status'] is 200:
    verify_result = bootpay.verify('[[ receipt_id ]]')
    if verify_result['status'] is 200:
        # 원래 주문했던 금액이 일치하는가?
        # 그리고 결제 상태가 완료 상태인가?
        if verify_result['data']['status'] is 1 and verify_result['data']['price'] is price:
            # TODO: 이곳이 상품 지급 혹은 결제 완료 처리를 하는 로직으로 사용하면 됩니다.
```


<br>
<br>


## 🔮 검증결과

```js
{
  "status": 200,
  "code": 0,
  "message": "",
  "data": {
    "receipt_id": "",
    "order_id": "12345678910",
    "name": "테스트음식",
    "price": 3000,
    "unit": "krw",
    "pg": "kcp",
    "method": "card",
    "pg_name": "KCP",
    "method_name": "카드결제",
    "payment_data": {
      "card_name": "BC카드",
      "card_no": "",
      "card_quota": "00",
      "card_auth_no": "",
      "receipt_id": "",
      "n": "테스트음식",
      "p": 3000,
      "pg": "KCP",
      "pm": "카드결제",
      "pg_a": "kcp",
      "pm_a": "card",
      "o_id": "",
      "p_at": "2021-12-27",
      "s": 1,
      "g": 2
    },
    "requested_at": "2021-12-27 16:47:52",
    "purchased_at": "2021-12-27 16:48:53",
    "status": 1
  }
}
```

<br>

결제가 완료되면 "s"값이 1이 나옵니다. "status"의 경우 현재 결제의 상태를 보여주는데, 다음과 같이 나타낼 수 있습니다.

<br>

```js
gist4
결제 상태 ( 현재 결제의 상태를 나타냅니다. 결제 검증에서 가장 중요한 지표가 됩니다. )

0 - 결제 대기 상태입니다. 승인이 나기 전의 상태입니다.
1 - 결제 완료된 상태입니다.
2 - 결제승인 전 상태입니다. transactionConfirm() 함수를 호출하셔서 결제를 승인해야합니다.
3 - 결제승인 중 상태입니다. PG사에서 transaction 처리중입니다.
20 - 결제가 취소된 상태입니다.
-20 - 결제취소가 실패한 상태입니다.
-30 - 결제취소가 진행중인 상태입니다.
-1 - 오류로 인해 결제가 실패한 상태입니다.
-2 - 결제승인이 실패하였습니다.
```

<br>

서류작성 및 PG사 심사(앱 결제테스트 등)는 거의 3주 가까이 걸렸는데, 서버사이드의 작업은 3시간도 채 걸리지 않고 작업을 끝낼 수 있었습니다.
또한 Bootpay 개발자와 원할한 소통도 가능하고 Bootpay 버그에 대한 수정 요청도 빠르게 처리해 주기 때문에,(우리는 앱단에서 요청사항이 있었는데, 4일내로 업데이트 해주셨음...)
저 처럼 빠른시일내 앱결제 시스템이 필요한경우가 있으신분은 Bootpay를 한번 고려해보는것도 좋은 선택인것 같습니다!😛


<br>
<br>


**[참고자료]**
- [https://docs.bootpay.co.kr/rest/verify](https://docs.bootpay.co.kr/rest/verify)
- [페이앱-api로-연동하기-결제-검증](https://medium.com/@bootpay.co.kr/%ED%8E%98%EC%9D%B4%EC%95%B1-api%EB%A1%9C-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0-%EA%B2%B0%EC%A0%9C-%EA%B2%80%EC%A6%9D-2-3-4659bd100f86)
