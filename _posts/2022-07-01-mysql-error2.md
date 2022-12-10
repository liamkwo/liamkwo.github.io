---
layout: post
emoji: 🤔🙌
title: "(문제해결)You do not have the SUPER privilege and binary logging is enabled (you *might* want to use the less safe log_bin_trust_function_creators variable)"
date: '2022-07-01 16:42:29'
author: Liam
categories: [ MySQL, AWS RDS ]
image: assets/images/logo.png
---

<br>
<br>

## 💡 Intro


MySQL을 사용하는중에 특정 테이블에 트리거를 생성했는데, `You do not have the SUPER privilege and binary logging is enabled (you *might* want to use the less safe log_bin_trust_function_creators variable)`라는 에러가 발생했습니다. 구글에서 정보를 찾아보니 계정에 권한이 없으니 **log_bin_trust_function_creators variable**의 설정을 변경해야 한다고 합니다.


<br>
<br>


## 🔎 RDS log_bin_trust_function_creators


**log_bin_trust_function_creators** 옵션은 MySQL이 함수를 수정 및 생성에 대한 제약을 강제할 수 있는 기능입니다.
해당 옵션의 default는 0(OFF)이며, OFF상태의 경우 권한이 있더라도 trigger를 생성할 수 없고, function을 생성할 수 없습니다. SUPER 권한이 있는 사용자만이 함수를 생성 및 변경할 수 있다고 합니다. 또한 **log_bin_trust_function_creators** 옵션을 1(ON)로 설정한 경우에는 함수 생성에 제약을 받지 않습니다.


<br>
<br>


## 📚 AWS RDS 파라미터 그룹 변경하기


### 📕 1. log_bin_trust_function_creators 옵션 변경하기


AWS RDS의 파라미터 그룹으로 들어가 줍니다. 그리고 변경할 파라미터 그룹의 이름을 클릭합니다.

<br>

![ERROR1227-1.png]({{ site.baseurl }}/assets/images/mysql-error/ERROR1227-1.png)

<br>

그리고 파리미터 편집으로  **log_bin_trust_function_creators**에 0(OFF)으로 되어져있는 옵션의 값을 1로 변경해줍니다.

<br>

![ERROR1227-2.png]({{ site.baseurl }}/assets/images/mysql-error/ERROR1227-2.png)

<br>
<br>

### 📘 2. 데이터베이스 파라미터 그룹 변경


AWS RDS의 데이터베이스에 들어갑니다. 그리고 우측의 수정을 클릭합니다.

<br>

![ERROR1227-3.png]({{ site.baseurl }}/assets/images/mysql-error/ERROR1227-3.png)

<br>

추가 구성의 데이터베이스 옵션으로 가서 **DB파라미터 그룹**을 default에서 위에서 설정한 그룹으로 변경합니다. 그리고 맨 밑의 **DB인스턴스 수정**을 클릭하면 수정됩니다.

<br>

![ERROR1227-4.png]({{ site.baseurl }}/assets/images/mysql-error/ERROR1227-4.png)

<br>
<br>


**[참고자료]**
- [https://aws.amazon.com/ko/premiumsupport/knowledge-center/rds-mysql-functions/](https://aws.amazon.com/ko/premiumsupport/knowledge-center/rds-mysql-functions/)

