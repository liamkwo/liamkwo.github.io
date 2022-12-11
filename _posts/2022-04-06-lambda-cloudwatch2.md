---
layout: post
emoji: ⏰
title: "Amazon EventBridge로 특정시간에 자동으로 카운트해보기 -2"
date: '2022-04-06 12:03:15'
author: Liam
tags: blog Python AWSLambda AWSCloudwatch
categories: [ AWS, Python, AWS Lambda, Aamazon Cloudwatch ]
image: assets/images/lambda-cloudwatch/cron1.png
---

<br>
<br>

## 💡 Intro

- Lambda함수를 이용해 특정시간에 자동으로 카운트를 해보려고합니다.🙌
- AWS Lambda는 실시간으로 코드를 실행할 수는 있으나, 필요시에만 함수를 실행합니다. 그래서 EventBridge를 이용해 특정 시간마다 Lambda에 트리거를 걸어 카운트를 해보려고 합니다.


<br>
<br>


## 📚 Amazon EventBridge 규칙 생성하기

### 📕 1. Amazon EventBridge 규칙 생성하기

AWS 홈페이지에서 Amazon EventBridge에 들어갑니다.

<br>

![eventbridge.png]({{ site.baseurl }}/assets/images/lambda-cloudwatch/eventbridge.png)

<br>

Amazon EventBridge에서 이벤트목록에 규칙을 들어가면 규칙생성이 있는데, 다른 부분은 건들것 없이 그냥 규칙생성만 클릭해주면 됩니다. 

<br>

![createrule.png]({{ site.baseurl }}/assets/images/lambda-cloudwatch/createrule.png)

<br>
<br>

### 📘 2. 규칙 세부 정보 설정하기

***📝 1단계 - 규칙 세부 정보 정의***

원하는 규칙이름을 정하고, 저번 포스트에 공부하였던 Cron식을 사용할 것이기 때문에, 규칙 유형을 일정으로 하고 다음버튼을 클릭합니다.

<br>

![rule1.png]({{ site.baseurl }}/assets/images/lambda-cloudwatch/rule1.png)

<br>

***📝 2단계 - 일정 정의***

규칙2에서는 패턴을 설정하는데, 저녁 12시에 매일 Lambda에 이벤트를 줄 예정이기 때문에 다음과 같이 설정했습니다. Cron식이 궁금하다면 [AWS CloudWatch로 자동 카운트 시스템 개발하기 -1](https://liampoet.github.io/Lambda-CloudWatch/)를 클릭하세요!

<br>

![rule2.png]({{ site.baseurl }}/assets/images/lambda-cloudwatch/rule2.png)

<br>

GMT는 [그리니치 평균시(Greenwich Mean Time)](https://bobbohee.github.io/2021-01-29/what-is-utc-and-gmt)라고 하는데, 그냥 대한민국 서울의 시간이 필요한 저희는 GMT에 9시간을 더한 현지 시간대를 선택하시면 됩니다.

<br>

***📝 3단계 - 대상 선택***

대상 유형에는 각각 EventBridge 이벤트 버스, [EventBridge API대상](https://docs.aws.amazon.com/ko_kr/eventbridge/latest/userguide/eb-api-destinations.html), AWS 서비스가 존재합니다. 하나의 event rule에 여러개의 대상이 등록이 가능하며, 저는 Lambda함수에 이벤트를 줄거라서 AWS서비스의 Lambda 항수를 선택했습니다. 

<br>

![rule3.png]({{ site.baseurl }}/assets/images/lambda-cloudwatch/rule3.png)

<br>

![rule3plus.png]({{ site.baseurl }}/assets/images/lambda-cloudwatch/rule3plus.png)

<br>

그리고 규칙을 생성해 줍니다.

<br>

![rule4.png]({{ site.baseurl }}/assets/images/lambda-cloudwatch/rule4.png)

<br>

아래와 같이 생성한 규칙의 세부 정보와 상태를 바로 확인할 수 있습니다.

<br>

#### 🔮 결과

<br>

![rule5.png]({{ site.baseurl }}/assets/images/lambda-cloudwatch/rule5.png)


<br>
<br>


**[참고자료]**
- [https://velog.io/@techy-yunong/AWS-EventBridge-concept](https://velog.io/@techy-yunong/AWS-EventBridge-concept)
- [Amazon EventBridge 이벤트 버스 생성](https://docs.aws.amazon.com/ko_kr/eventbridge/latest/userguide/eb-create-event-bus.html)
