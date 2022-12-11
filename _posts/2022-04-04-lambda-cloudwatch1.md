---
layout: post
emoji: ⏰
title: "Amazon EventBridge로 특정시간에 자동으로 카운트해보기 -1"
date: '2022-04-04 18:18:15'
author: Liam
categories: [ AWS, Python, AWS Lambda, EventBridge ]
image: assets/images/lambda-cloudwatch/cron1.png
---

<br>
<br>

## 💡 Intro

- Lambda함수를 이용해 특정시간에 자동으로 카운트를 해보려고합니다.🙌
- AWS Lambda는 실시간으로 코드를 실행할 수는 있으나, 필요시에만 함수를 실행합니다. 그래서 EventBridge를 이용해 특정 시간마다 Lambda에 트리거를 걸어 카운트를 해보려고 합니다.


<br>
<br>


## 🔎 들어가기전, 알고가기! -1

**Lambda**
- [Lambda](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/welcome.html)는 서버를 프로비저닝하거나 관리하지 않고도 코드를 실행할 수 있게 해주는 컴퓨팅 서비스입니다. Lambda는 고가용성 컴퓨팅 인프라에서 코드를 실행하고 서버와 운영 체제 유지 관리, 용량 프로비저닝 및 자동 조정, 코드 및 보안 패치 배포, 코드 모니터링 및 로깅 등 모든 컴퓨팅 리소스 관리를 수행할 수 있습니다. 또한 Lambda는 필요시에만 함수를 실행하며, 일일 몇 개의 요청에서 초당 수천 개의 요청까지 자동으로 확장이 가능합니다. 그리고 사용한 컴퓨팅 시간만큼만 비용을 지불하고, 코드가 실행되지 않을 때는 요금이 부과되지 않습니다.

<br>

**CloudWatch**
- [CloudWatch](https://docs.aws.amazon.com/ko_kr/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)는 Amazon Web Services(AWS) 리소스 및 AWS에서 실행되는 애플리케이션을 실시간으로 모니터링합니다. CloudWatch를 사용하여 리소스 및 애플리케이션에 대해 측정할 수 있는 변수인 지표를 수집하고 추적할 수 있으며([Amazon CloudWatch Logs](https://docs.aws.amazon.com/ko_kr/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html)), [Amazon EventBridge](https://docs.aws.amazon.com/ko_kr/eventbridge/latest/userguide/eb-what-is.html)를 이용해 특정한 이벤트를 처리할 수 도 있습니다. 
- [EventBridge](https://docs.aws.amazon.com/ko_kr/eventbridge/latest/userguide/eb-what-is.html)는 과거에는  Amazon CloudWatch Events라고 불렸으며, Zendesk 또는 Shopify와 같은 이벤트 소스의 실시간 데이터 스트림을 AWS Lambda 및 기타 SaaS 애플리케이션과 같은 대상으로 전송하는 이벤트 기반 애플리케이션을 대규모로 손쉽게 구축할 수 있는 서버리스 이벤트 버스입니다.


<br>
<br>


## 🔎 들어가기전, 알고가기! -2

**크론(Cron) 표현식이란?🤔**
- EventBridge는 이벤트를 세팅할때 기본적으로 크론 정규표현식을 사용합니다. 그럼 크론식이 무엇일까요?(저도 이번에 크론식을 처음 경험해 보았는데, 쓸 경우가 종종 생길 것 같아서 정리해두려고 합니다.)
- 유닉스 계열의 운영체제에서 시간 기반으로 job scheduling을 하는 프로세스를 [크론 표현식(Cron Expressions)](https://docs.oracle.com/cd/E12058_01/doc/doc.1014/e12030/cron_expressions.htm)라고 하며 자바 스프링의 Quartz나 Spring Scheduler에서도 사용하기도 한다고 합니다. 또한, 크론 표현식은 필드와 특수문자를 조합하여 스케쥴링을 조절할 수 있습니다.
- 리눅스/유닉스 크론 표현식에서는 5개의 필드를 사용하고, Quartz 기반의 크론 표현식에서는 7개의 필드를 사용하는 String 문자열입니다. 공백으로 구분하고(" ") 각 필드는 앞에서 부터 초, 분, 시, 일, 월, 요일, 년으로 구분합니다.

<br>

**크론 표현식 표현**

![cron1.png]({{ site.baseurl }}/assets/images/lambda-cloudwatch/cron1.png)

- * : 모든 값
- **?** : 특정한 값이 없음 
- **-** : 범위를 뜻 함 (예) 월요일에서 수요일까지는 MON-WED로 표현
- **,** : 특별한 값일 때만 동작 (예) 월,수,금 MON,WED,FRI 
- **/** : 시작시간 / 단위  (예) 0분부터 매 5분 0/5
- **L** : 일에서 사용하면 마지막 일, 요일에서는 마지막 요일(토요일)
- **W** : 가장 가까운 평일 (예) 15W는 15일에서 가장 가까운 평일 (월 ~ 금)을 찾음
- **#** : 몇째주의 무슨 요일을 표현 (예) 3#2 : 2번째주 수요일

<br>

**크론 표현식 예제**

![cron2.png]({{ site.baseurl }}/assets/images/lambda-cloudwatch/cron2.png)

- 크론 표현식에 대해 공부를 하면서 느낀점은, 어느정도만 알면 전혀 어렵지 않다는 것입니다. 표를 보고 대충 이해하여 한번만 만들어봐도 다음 부터는 쉽게 만들 수 있습니다.😀  
- [CronMaker](http://www.cronmaker.com/?1)라는 원하는 크론 표현식을 생성해 주는 사이트 존재합니다. 대충 크론 표현식을 만든 후, 내가 만든 크론 표현식을 입력하고 Calcurate  next dates 버튼을 클릭하면, 내가 만든 크론 표현식으로 언제 실행될것인지 미리 볼수 있는 기능도 존재합니다.


<br>
<br>


## 📚 Lambda 코드작성

### 📕 1. Lambda 함수생성

아래와 같이 본인이 사용하는 언어를 선택한 후, Lambda 함수를 생성합니다. 그리고 VPC및 함수에 필요한 [Lambda Layer](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/configuration-layers.html)를 설정합니다.

<br>

![Lambda.png]({{ site.baseurl }}/assets/images/lambda-cloudwatch/Lambda.png)

<br>
<br>

### 📘 2. Lambda 코드작성

자동으로 카운트 해주어야할 데이터를 조건에 따라서 가짓 수를 나누어 주었습니다. 저는 필요에 따라 S3와 RDS를 연결하였습니다.  

<br>

```py
import json
import base64, os, boto3, ast, copy
import json
import pymysql
import ast 
import db_config

AISCAN_URL = ''
rds_host = db_config.rds_host
name = db_config.name
password = db_config.password
db_name = db_config.db_name 
########### init s3 bucket info
s3 = boto3.resource('s3') 
bucket_name = ''
def lambda_handler(event, context): 
    conn = pymysql.connect(rds_host, user=name, passwd=password, db=db_name, connect_timeout=5)    
    c_query = '' 
    print(event)
    
    with conn.cursor(pymysql.cursors.DictCursor) as cur:
        ...
```

<br>
<br>

Amazon EventBridge 규칙 생성은 다음 포스팅에 이어서 작성을 해보도록 하겠습니다.😀
