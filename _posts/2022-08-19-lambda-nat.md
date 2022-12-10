---
layout: post
emoji: 🏕
title: "VPC내의 AWS Lambda의 인터넷 접속하기"
date: '2022-08-19 21:01:55'
author: Liam
categories: [ AWS, AWS Lambda ]
image: assets/images/lambda-nat/gateway.png
---

<br>
<br>

## 💡 Intro

Amazon Virtual Private Cloud(Amazon VPC)에 연결된 AWS Lambda에 인터넷 접속을 가능하게 하려면, NAT GW(Network Address Translation Gateway) 설정을 해야합니다. 오늘은 이에 대해서 정리해 보려고 합니다.


<br>
<br>


## 🔎 AWS Lambda의 세 가지 상태

저는 Amazon Virtual Private Cloud(Amazon VPC)에 연결된 **AWS Lambda**에 인터넷 접속을 가능하게 하기 위해, 3번째 케이스인 **VPC**와 **NAT**이 있는 상태로 진행해보도록 하겠습니다.

- **VPC**가 없는 상태 : 웹에 자유롭게 접속할 수 있지만, 로컬 네트워크로 AWS 서비스들과 연결할 수 없습니다.
- **VPC**가 있는 상태 : 기본적인 설정으로 로컬 네트워크에 연결이 가능하지만, 웹에는 접속할 수 없습니다.
- **VPC**와 **NAT**이 있는 상태 : AWS 서비스와 웹 둘 다 접속이 가능합니다.


<br>
<br>


## 🔎 Amazon VPC(Amazon Virtual Private Cloud), Subnet, Router, Routing Table, IGW(Internet Gateway), NAT GW(Network Address Translation Gateway)란?

본 내용에 들어가기전, **Amazon VPC**, **Subnet**, **Routing Table**, **IGW**, **NAT GW**에 대해서 먼저 정리하고 넘어가 보도록 하겠습니다.


### 🌎 Amazon VPC(Amazon Virtual Private Cloud)

**Amazon VPC**란, AWS 클라우드에서 다른 고객과 완벽하게 논리적으로 격리된 네트워크 공간을 제공하여 프로비저닝하여 가상 네트워크에서 AWS 리소스를 만드는데 사용하는 리소스입니다. 쉽게 풀어서 설명하자면, 자신이 사용할 AWS Resource들을 격리 되어진 하나의 네트워크로 묶는 서비스를 의미하며 격리되어 있기 때문에 다른 사람들은 접근하고 보는 것이 불가능해집니다.

> 만약, VPC 없이 인스턴스를 생성한다면 아래와 같이 인스턴스끼리의 구분없는 연결로 인해 시스템 복잡도가 증가할 것이며, 인터넷을 통해 전달되는 트래픽의 전송이 굉장히 비효율적이게 될 것 입니다. VPC를 적용하면 인스턴스가 VPC에 속함으로써 네트워크를 구분할 수 있고, VPC 별로 필요한 설정을 통해 인스턴스에 네트워크 설정을 적용할 수 있습니다. 또한, 각각의 VPC는 독립적이기 때문에 서로 통신할 수 없습니다. 만일 통신을 원한다면 VPC 피어링 서비스를 통해 VPC 간에 트래픽을 라우팅할 수 있도록 설정할 수 있습니다. ➕ AWS는 VPC의 중요성을 강조하여 2019년부터 거의 모든 서비스에 VPC를 적용하도록 강제하였습니다.

<br>

![Amazon VPC]({{ site.baseurl }}/assets/images/lambda-nat/vpc.png)*이미지 1. Amazon VPC*

<br>

### 🌎 서브넷(Subnet)

**서브넷(Subnet)** 이란, IP 주소를 나누어 리소스가 배치되는 물리적인 주소 범위입니다. 즉, 서브넷은 VPC를 잘 개 쪼갠 것이며 서브넷을 나누는 이유는 더 많은 네트워크 망을 만들기 위함입니다. 또한 인터넷에 연결되어야 하는 리소스에는 퍼블릭 서브넷(Public Subnet)을 사용하고, 인터넷에 연결되지 않는 리소스에는 프라이빗 서브넷(Private Subnet)을 사용합니다.

> 이처럼 인터넷 연결 여부로 퍼블릭 서브넷(Public Subnet)과 프라이빗 서브넷(Private Subnet)로 구분하는 이유는 보안을 강화하기 위함입니다. 퍼블릭 서브넷에 존재하는 인스턴스는 인터넷에 연결되어 아웃바운드, 인바운드 트래픽을 주고받을 수 있습니다. 반면 프라이빗 서브넷은 외부에 노출이 되어 있지 않기 때문에 접근할 수 없습니다. 즉, 인터넷과 연결되어 외부에 노출되어 있는 면적을 최소화함으로써 네트워크 망에 함부로 접근하는 것을 막기 위함입니다.

<br>

![Subnet]({{ site.baseurl }}/assets/images/lambda-nat/subnet.png)*이미지 2. Subnet*

<br>

### 🌎 라우팅 테이블(Routing Table)과 라우터(Router)

AWS VPC 설명서에는 라우팅 테이블의 역할이 이렇게 소개되어져 있습니다.

`VPC에는 암시적 라우터가 있으며 라우팅 테이블을 사용하여 네트워크 트래픽이 전달되는 위치를 제어합니다. VPC의 각 서브넷을 라우팅 테이블에 연결해야 합니다. 테이블에서는 서브넷에 대한 라우팅을 제어합니다(서브넷 라우팅 테이블). 서브넷을 특정 라우팅 테이블과 명시적으로 연결할 수 있습니다. 그렇지 않으면 서브넷이 기본 라우팅 테이블과 암시적으로 연결됩니다. 서브넷은 한 번에 하나의 라우팅 테이블에만 연결할 수 있지만 여러 서브넷을 동일한 서브넷 라우팅 테이블에 연결할 수 있습니다.` <span style="color:gray; font-size:2px;">- 출처: AWS Docs<span>

간단하게 설명하자면 라우터(Router)는 목적지이고, 라우팅 테이블(Routing Tabl)은 이정표라고할 수 있습니다. `172.31.0.0/16가 목적지면 이리로 오고, 10.0.0.0/16가 목적지면 절로 가세요.`라고 알려주는 이정표일 뿐인거죠. 만약 목적지가 라우팅 테이블에 명시되지 않은 트래픽들을 라우팅 시키고 싶을 땐 `0.0.0.0/0`라는 주소를 통해 전달시킬 수 있습니다.

<br>

![Router]({{ site.baseurl }}/assets/images/lambda-nat/router.png)*이미지 3. Router*

<br>

### 🌎 IGW(Internet Gateway, 인터넷 게이트웨이) / NAT GW(Network Address Translation Gateway, NAT 게이트웨이)

사실상 **IGW**와 **NAT GW** 이 두 개를 통해 인터넷 간 통신을 활성화시킬 수 있습니다. **IGW**는 VPC 리소스와 인터넷 간 통신을 활성화하기 위해 VPC에 연결하며, 퍼블릭 서브넷만 외부와 통신해야 하므로 퍼블릭 서브넷의 라우팅 테이블에만 IGW로 향하는 규칙을 포함합니다. **NAT GW**는 퍼블릭 서브넷에 위치하여 프라이빗 서브넷에서 발생하는 네트워크 요청을 받은 후, IGW에 패킷을 전달하여 인터넷과 통신할 수 있도록 도와줍니다.

> 라우팅 테이블을 보면 0.0.0.0/0으로 되어 있는 것을 볼 수 있는데, 목적지의 IP 주소가 172.31.0.0/16(VPC 내부)에 해당하는지 확인하고 해당하지 않는 모든 트래픽에 대하여 IGW로 보내라는 뜻입니다. 즉, 목적지가 라우팅 테이블에 명시되지 않은 모든 트래픽을 라우팅 시키고 싶을 땐 0.0.0.0/0라는 주소를 사용할 수 있습니다.

<br>

![IGW / NAT GW]({{ site.baseurl }}/assets/images/lambda-nat/gateway.png)*이미지 4. IGW / NAT GW*

<br>
<br>


## 📚 VPC내의 AWS Lambda의 인터넷 접속하기

이제 본격적으로 작업을 실시할 수 있습니다. 🌎[Amazon Virtual Private Cloud(Amazon VPC)에 연결된 AWS Lambda 함수에 인터넷 액세스를 제공하려 합니다. 어떻게 설정해야 하나요?](https://aws.amazon.com/ko/premiumsupport/knowledge-center/internet-access-lambda-function/) 문서에 자세하게 나와있습니다. 그냥 이 순서 그대로 따라하시면 진행하는데 전혀 큰 어려움은 없습니다.


### 📕 Amazon VPC 설정

AWS Console > VPC로 들어가면, **VPC, Subnet, Routing Table, IGW, NAT GW**등 앞서 정리한 모든 항목이 있습니다.

<br>

![AWS Console > VPC]({{ site.baseurl }}/assets/images/lambda-nat/vpc_console.png)*이미지 5. AWS Console -> VPC*

<br>

**1. Amazon VPC를 생성합니다.**

- AWS Console > VPC > VPC 생성
- 이름 및 IPv4를 지정 / 생성해줍니다.

<br>

![AWS Console > VPC > VPC 생성1]({{ site.baseurl }}/assets/images/lambda-nat/create_vpc1.png)*이미지 6. VPC 생성 버튼 클릭*

<br>

![AWS Console > VPC > VPC 생성2]({{ site.baseurl }}/assets/images/lambda-nat/create_vpc2.png)*이미지 7. VPC 생성*

<br>

**2. Amazon VPC에 하나의 퍼블릭 서브넷과 둘 이상의 프라이빗 서브넷을 생성합니다.**

- AWS Console > VPC > Subnet 생성
- 이름 및 IPv4과 가용영역을 설정하고, 이전에 만든 VPC를 연결합니다.
- Amazon VPC에 하나의 퍼블릭 서브넷과 둘 이상의 프라이빗 서브넷을 생성합니다.(서브넷을 생성할 때, [이름 태그]에 퍼블릭 또는 프라이빗 서브넷을 식별하는 각 서브넷의 이름을 입력합니다. 예: public-Subnet1, private-Lambda-Subnet1, private-Lambda-Subnet2)

<br>

![AWS Console > VPC > Subnet 생성1]({{ site.baseurl }}/assets/images/lambda-nat/create_subnet1.png)*이미지 8. 서브넷 생성 버튼 클릭*

<br>

![AWS Console > VPC > Subnet 생성2]({{ site.baseurl }}/assets/images/lambda-nat/create_subnet2.png)*이미지 9. 서브넷 생성*

<br>

**3. 인터넷 게이트웨이를 생성하여 Amazon VPC에 연결합니다.**

- AWS Console > VPC > IGW(Internet Gateway)
- 이름을 생성해준 후, 생성한 IGW를 클릭하여 이전 단계에서 만든 VPC에 연결해줍니다.

<br>

![AWS Console > VPC > IGW 생성1]({{ site.baseurl }}/assets/images/lambda-nat/create_igw1.png)*이미지 10. 인터넷 게이트웨이 생성 버튼 클릭*

<br>

![AWS Console > VPC > IGW 생성2]({{ site.baseurl }}/assets/images/lambda-nat/create_igw2.png)*이미지 11. 인터넷 게이트웨이 생성*

<br>

![AWS Console > VPC > IGW 연결]({{ site.baseurl }}/assets/images/lambda-nat/create_igw3.png)*이미지 12. 만들어둔 VPC연결*

<br>

**4. NAT 게이트웨이를 생성합니다.**

- AWS Console > VPC > NAT GW(Network Address Translation Gateway)
- 이름을 설정하고 이전에 Public Subnet으로 사용하기 위해 만들었던 Subnet에 연결하여 NAT GW를 생성합니다.

> ✅ NAT Gateway는 시간당 **0.059 USD**가 부과됩니다.(리전: 아시아 태평양(서울))

<br>

![AWS Console > VPC > NAT GW 생성1]({{ site.baseurl }}/assets/images/lambda-nat/create_nat1.png)*이미지 13. NAT 게이트웨이 생성 버튼 클릭*

<br>

![AWS Console > VPC > NAT GW 생성2]({{ site.baseurl }}/assets/images/lambda-nat/create_nat2.png)*이미지 14. 만들어둔 서브넷 연결 후 NAT 게이트웨이 생성*

<br>

**5. 퍼블릭 서브넷과 프라이빗 서브넷용 사용자 지정 라우팅 테이블 두 개를 생성합니다.**

- AWS Console > VPC > Routing Table
- 이름을 지정하고 이전에 만든 VPC에 연결합니다.
- 퍼블릭 서브넷과 프라이빗 서브넷용 사용자 지정 라우팅 테이블 두 개를 생성합니다.

<br>

![AWS Console > VPC > 라우팅 테이블 생성1]({{ site.baseurl }}/assets/images/lambda-nat/create_route1.png)*이미지 15. 라우팅 테이블 생성 버튼 클릭*

<br>

![AWS Console > VPC > 라우팅 테이블 생성2]({{ site.baseurl }}/assets/images/lambda-nat/create_route2.png)*이미지 16. 라우팅 테이블 두 개 생성*

<br>

**🧩 퍼블릭 서브넷의 라우팅 테이블의 경우**
1. [Destination]에 [0.0.0.0/0]을 입력합니다.
2. [Target]에서 IGW를 선택한 다음, 이전에 생성한 IGW의 ID(my-igw1)를 선택합니다.
3. 서브넷 연결 > 서브넷 연결 편집 > Public Subnet 선택 > 저장합니다.

<br>

![AWS Console > VPC > 라우팅 테이블 생성3]({{ site.baseurl }}/assets/images/lambda-nat/create_route3.png)*이미지 17. 라우팅 테이블 편집에서 인터넷 게이트웨이 연결*

<br>

**🧩 프라이빗 서브넷의 라우팅 테이블의 경우**
1. [Destination]에 [0.0.0.0/0]을 입력합니다.
2. [Target]에서 NAT GW 선택합니다. 그런 다음 생성한 NAT GW(my-nat-gw1)의 ID를 선택합니다.
3. 서브넷 연결 > 서브넷 연결 편집 > Private Subnet들 선택 > 저장합니다.

<br>

![AWS Console > VPC > 라우팅 테이블 생성4]({{ site.baseurl }}/assets/images/lambda-nat/create_route4.png)*이미지 18. 라우팅 테이블 편집에서 NAT 게이트웨이 연결*


<br>
<br>


### 📘 Lambda 함수 구성

1. Lambda 콘솔에서 함수페이지를 연 후, 본인이 생성한 Amazon VPC에 연결할 함수를 선택합니다. 
2. [구성] > VPC > [편집]을 선택합니다.

<br>

![AWS Console > Lambda > VPC 연결할 함수 > 구성 > VPC > 편집]({{ site.baseurl }}/assets/images/lambda-nat/lambda1.png)*이미지 19. Lambda 함수 VPC 편집 선택*

<br>

3. VPC를 선택한 후, 만들어 둔 Private Subnet을 선택합니다.
4. [보안 그룹]에서 보안 그룹을 선택합니다.(참고 : [기본 보안 그룹](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html#DefaultSecurityGroup)을 사용해도 모든 아웃바운드 인터넷 트래픽을 허용하며, 대부분의 경우에 충분합니다. 자세한 내용은 [VPC의 보안 그룹](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)을 참조하십시오. <span style="color:gray; font-size:2px;">- 출처: AWS Docs<span>)

<br>

![Lambda VPC 편집]({{ site.baseurl }}/assets/images/lambda-nat/lambda2.png)*이미지 20. VPC 편집*


<br>
<br>


> 끝맺음

VPC, NAT GW, IGW 등에 대해 다시 정리를 해보려다 직접 연결을 해보면서 하는 것이 나을 것 같아서, 예시로 Lambda에 연결하는 부분까지 다시 정리해 보았습니다. 내용을 정리하면서 여러 정보를 찾았는데, 결론은 다들 요금 주의,,,,,,😭 NAT GW에서 쓸데없는 요금이 나가는 경우도 있더라고요. 다음에는 AWS를 하면서 주의해야 할 요금에 대해 정리를 해보려 합니다.(솔직히 아직 AWS에 대해 초보이다 보니, 이번에 강전희 님과 정태환 님이 번역하신 [AWS 비용 최적화 바이블 - 핀옵스를 위한 최적의 기술 활용부터 운영 노하우까지](http://www.yes24.com/Product/Goods/111715522)를 샀습니다.... 빨리 다 읽어보고 한번 정리를 해보려고 합니다!😃)


<br>
<br>


**[참고자료]**

- [AWS docs - VPC, Subnet](https://docs.aws.amazon.com/vpc/latest/userguide/working-with-vpcs.html#AddaSubnet)
- [https://aws.amazon.com/ko/premiumsupport/knowledge-center/internet-access-lambda-function/](https://aws.amazon.com/ko/premiumsupport/knowledge-center/internet-access-lambda-function/)
- [https://medium.com/@kimjnsjwj/vpc내의-aws-lambda의-인터넷-접속-bc503e9940f5](https://medium.com/@kimjnsjwj/vpc내의-aws-lambda의-인터넷-접속-bc503e9940f5)
- [https://jbhs7014.tistory.com/164](https://jbhs7014.tistory.com/164)
- [https://err-bzz.oopy.io/c4abbed2-fc30-4061-81b0-2803c4a59809](https://err-bzz.oopy.io/c4abbed2-fc30-4061-81b0-2803c4a59809)
- [AWS docs - VPC의 보안 그룹](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)

