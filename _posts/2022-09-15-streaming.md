---
layout: post
title: "스트리밍 서비스와 AWS Media Services"
date: '2022-09-15 20:08:49'
author: Liam
categories: [ CDN, AWS ]
image: assets/images/streaming/hls.png
---

<br>
<br>

## 💡 Intro


Youtube를 보다가 정말 정말 갑자기 스트리밍 서비스에 대해 궁금해졌습니다. 단순히 CDN(Content Delivery Network)을 사용한다 정도만 알고 있었는데, 조금 자세하게 공부를 해보려 합니다.


<br>
<br>


## 🔎 스트리밍(Streaming) 이란?

스트리밍 서비스란 **인터넷에서 음성,미디어 컨텐츠를 실시간으로 재생하는 방식**을 뜻합니다. 이러한 스트리밍 서비스에는 **Progressive download, RTSP, RTMP 스트리밍, Adaptive HTTP Streaming** 등 여러가지 프로토콜이 있습니다.

***

**프로그레시브 다운로드(Progressive Download)** : 웹 서버로부터 동영상을 다운로드하면서 파일이 도착하는 대로 재생해주는 방식

- 웹서버에 파일을 올려놓고 URL주소를 player에 링크만 걸면 되므로 사용이 편리함
- 동영상이 끊김없이 재생되기 위해서는 네트워크 속도가 동영상의 데이터 레이트 보다 높아야 함
- 내컴퓨터로 파일을 다운로드 하는 것이기 때문에 보안에 문제가 있어 유료 비디오 서비스를 위해서는 사용할 수 없음
- 데이터 소비가 사용자가 보는 만큼이 아니라 다운로드 된 만큼 소비

***

**RTSP/RTMP 스트리밍(RSTP/RTMP Streaming)** : 프로그레시브 다운로드 방식과는 달리 사용자가 현재 시청하고 있는 비디오 프레임만을 전송해주는 방식

- 유저가 보는 장면을 찾아 그 프레임부터 시작이 되며, 지나간 프레임 데이터는 자동 삭제되는 방식
- 다운로드가 없어 보안에 문제가 상대적으로 적음
- 필요한 부분만 전송하므로 bandwidth과 데이터 소비에 대해 효율성이 좋음

***

**Adaptive HTTP Streaming** : 미디어 컨텐츠를 동영상 컨텐츠 하나로 저장하는게 아니라 잘게 쪼개서 저장하는 방식. 대표적으로 Apple에서 만든 **HLS(HTTP Live Streaming)** 가 있음

***

**HLS(HTTP Live Streaming)** : HTTP 프로토콜을 사용하는 실시간 스트리밍 방식으로, 스트리밍 데이터를 m3u8의 확장자를 가진 재생목록 파일과 잘게 쪼갠 다수의 ts파일을 http을 통해 전송하는 방식

- HTTP를 프로토콜을 사용하며 HTML5 및 미디어 소스 확장으로 모든 장치의 호환성이 높으므로 도입 비용을 절약할 수 있음
- 적응형 스트리밍 방식으로 인터넷 속도에 따른 품질이 동적으로 조정됨
- 일반적인 프로토콜은 UDP형식으로 만들어진데 반해, HLS는 TCP형식이라서 라이브 스트림일 때 Delay 문제가 있음

<br>

![HLS의 구성]({{ site.baseurl }}/assets/images/streaming/hls.png)*이미지 1. HLS의 구성(출처: Apple Documentation Archive)*

<br>

> 유튜브에서 원래 프로그레시브 다운로드 방식을 사용하다가 현재는 Adaptive HTTP Streaming 방식을 사용하고 있다고 합니다.


<br>
<br>


## 🔎 AWS Media Services를 통한 라이브 및 온디맨드 동영상 워크플로 구성 및 배포하기

이를 이제 AWS에서 어떻게 배포할 수 있을지에 대해서 찾아보았는데... 🌏[AWS Builders 온라인 2021](https://www.youtube.com/playlist?list=PLORxAVAC5fUWPziIFAho12lvGl1hR7ZZ5)에서 **김영진님**께서 이 내용을 다루어주셨더라구요!(AWS.....너란 클라우드..) 서비스를 너무 잘 소개해 주셔서 아래에 Youtube 영상 주소를 올리겠습니다. 이미지를 클릭하시면 해당 영상으로 이동됩니다.

<br>

[![AWS 미디어서비스를 통한 라이브 및 온디맨드 동영상 워크플로 구성 및 배포하기 - 김영진:: AWS Builders Online Series]({{ site.baseurl }}/assets/images/streaming/youtube.png)*클릭시 Youtube로 이동됨*](https://youtu.be/fdoaUq49dSA?list=PLORxAVAC5fUWPziIFAho12lvGl1hR7ZZ5)

<br>

영상을 시청하시면 **Live Streaming**과 **VOD(Video On Demand) streaming** 두 가지다 개념부터 워크플로, Demo 버전을 만드는 것까지 자세하게 설명하고 있습니다. 그래서 저는 영상에 나오는 기본적인 AWS 서비스들에 대해서 따로 정리해 보려고 합니다.

<br>

![AWS 미디어 서비스를 이용한 워크플로우]({{ site.baseurl }}/assets/images/streaming/workflow.png)*이미지 2. AWS 미디어 서비스를 이용한 워크플로우(출처: AWS Youtube)*


<br>
<br>


## 🔎 영상속 AWS 서비스 정리

### 🌎 AWS Media Services

🌏[AWS Media Services](https://aws.amazon.com/ko/media-services/)는 클라우드에서 안정적인 브로드캐스트 품질의 비디오 워크플로를 손쉽게 구축할 수 있게 해주는 완전 관리형 서비스 패밀리입니다. 웹을 들어간 후, 상단 메뉴에 있는 서비스를 클릭하면 다양한 제품들을 확인할 수 있습니다.

<br>

#### **AWS Elemental MediaLive**

`AWS ElementalMediaLive는 브로드캐스트 및 스트리밍 전송을 위한 라이브 출력을 생성할 수 있는 실시간 비디오 서비스입니다. 이 서비스를 사용하면 텔레비전과 인터넷 연결 멀티스크린 디바이스(커넥티드 TV, 태블릿, 스마트폰, 셋톱 박스 등)로 전송할 수 있는 고품질 비디오 스트림을 생성할 수 있습니다.` <span style="color:gray; font-size:2px;">- 출처: AWS Docs<span>

<br>

AWS Elemental MediaLive는 Live 비디오 스트림을 실시간으로 인코딩하고 배포하는 서비스입니다. 쉽게 설명하자면, 고해상도의 입력신호를 적합한 화질로 변환해줍니다.

<br>

#### **AWS Elemental MediaPackage**

`MediaPackage는 AWS 클라우드에서 실행되는 적시 비디오 패키징 및 오리지네이션 서비스입니다. MediaPackage를 사용하면 매우 안전하고 확장 가능하며 신뢰할 수 있는 비디오 스트림을 다양한 재생 디바이스 및 CDN(콘텐츠 전송 네트워크, 여기서는 Amazon CloudFront)에 전달할 수 있습니다.` <span style="color:gray; font-size:2px;">- 출처: AWS - MediaPackage<span>

<br>

**MediaPackage**는 단일 비디오 입력으로부터 커넥티드 TV, 휴대폰, 컴퓨터, 태블릿 및 게임 콘솔에 맞는 형식으로 비디오 스트림을 생성해줍니다. 또한, 인기 있는 비디오 기능(다시 시작, 정지, 되감기 등)을 손쉽게 구현할 수 있고 DRM(Digital Rights Management)을 사용해 콘텐츠를 보호해준다고 합니다.

> **DRM(Digital Right Management)** 이란 정보보호 기술 중 하나로 암호화 기술을 이용해서 비허가 사용자로부터 디지털 컨텐츠를 보호하게 하는 기술을 말합니다.

AWS Elemental MediaPackage는 **라이브 패키징**과 **VOD 패키징**이라는 <u>두 가지 요금 모델</u>을 제공합니다. 라이브 패키징의 경우 채널로 수집된 비디오의 양(GB로 측정)과 채널에 대해 오리지네이션 및 패키징된 콘텐츠의 양(GB로 측정)에 따라 요금이 부과됩니다. VOD 패키징을 사용하면 VOD 패키징 그룹에서 오리지네이션 및 패키징된 비디오 컨텐츠의 양(GB로 측정)에 따라 요금이 청구됩니다. 자세한 요금 사항은 🌏[AWS Elemental MediaPackage 요금](https://aws.amazon.com/ko/mediapackage/pricing/)을 확인해 주세요.

<br>

#### **AWS Elemental MediaStore**

`AWS Elemental MediaStore는 미디어에 최적화된 AWS 스토리지 서비스입니다. 이는 라이브 스트리밍 비디오 콘텐츠를 전송하는 데 필요한 성능, 일관성 및 짧은 지연 시간을 제공합니다. AWS Elemental MediaStore는 비디오 워크플로에서 오리진 스토어의 역할을 합니다. 이 서비스의 뛰어난 성능은 비용 효율적인 장기 스토리지와 결합되어 가장 까다로운 미디어 전송 워크로드의 요건을 충족합니다.` <span style="color:gray; font-size:2px;">- 출처: AWS - MediaStore<span>

<br>

Docs를 읽고나면 S3와의 차이점에 대해 궁금하실텐데, Docs에 적힌데로 MediaStore는 **미디어에 최적화된 AWS 스토리지 서비스**입니다. cacheing layer가 있으며 S3보다 latency가 짧기 때문에 **라이브 비디오에는 MediaStore**를, **VOD(Video On Demand)에는 S3**를 사용한다고 합니다.

AWS Elemental MediaStore는 콘텐츠가 서비스에 도달할 때 GB당 **미디어 최적화 수집 비용**이 부과되고, 라이브 및 VOD 전송을 위해 서비스에 유지하는 콘텐츠에 대해 GB당 **콘텐츠 스토리지 비용**이 부과됩니다. 자세한 요금 사항은 🌏[AWS Elemental MediaStore 요금](https://aws.amazon.com/ko/mediastore/pricing/)을 확인해 주세요.

<br>

#### **AWS Elemental MediaTailor**

`AWS Elemental MediaTailor는 비디오 공급자가 브로드캐스트 수준의 서비스 품질을 떨어뜨리지 않고 비디오 스트림에 개별적으로 타겟 광고를 삽입할 수 있는 서비스입니다.` <span style="color:gray; font-size:2px;">- 출처: AWS - MediaStore<span>

<br>

![AWS Elemental MediaTailor 작동 방식]({{ site.baseurl }}/assets/images/streaming/MediaConvert.png)*이미지 3. AWS Elemental MediaTailor 작동 방식(출처: AWS - MediaTailor)*

<br>

즉, **AWS Elemental MediaTailor**란 수익 창출을 위한 광고 삽입 서비스입니다. MediaTailor는 클라이언트 측과 서버 측 광고 전달 지표 둘 다를 기준으로 자동화된 보고서를 제공해주기 때문에, 광고 노출 수와 시청자 동작을 정확하게 측정할 수 있다고 합니다.

<br>

#### **AWS Elemental MediaConvert**

`AWS Elemental MediaConvert 콘텐츠 소유자와 배포업체에게 모든 규모의 미디어 라이브러리에 대한 확장 가능한 비디오 처리를 제공하는 파일 기반 비디오 처리 서비스입니다. MediaConvert 에서는 다음과 같은 프리미엄 콘텐츠 경험을 지원하는 고급 기능을 제공합니다.` <span style="color:gray; font-size:2px;">- 출처: AWS Docs<span>

<br>

![AWS Elemental MediaConvert 작동 방식]({{ site.baseurl }}/assets/images/streaming/MediaConvert.png)*이미지 4. AWS Elemental MediaConvert 작동 방식(출처: AWS - MediaConvert)*

<br>

즉, 디바이스로 전달할 영상 파일을 인코딩 해주는 역할을 해주고 있습니다. 요금은 베이직 티어와 프로페셔널 티어로 나뉘어져 있고, 프로페셔널이 베이직 보다 1.5배 이상 비싸기 때문에 제공하는 서비스에 따라서 잘 선택하셔야 할 것 같습니다.🤔 자세한 사항은 🌏[AWS Elemental MediaConvert 요금](https://aws.amazon.com/ko/mediaconvert/pricing/)에서 확인 하시면 됩니다.

<br>

### 🌎 Other AWS Services


#### **Amazon S3**

`Amazon Simple Storage Service(Amazon S3)는 업계 최고의 확장성, 데이터 가용성, 보안 및 성능을 제공하는 객체 스토리지 서비스입니다. 모든 규모와 업종의 고객은 Amazon S3를 사용하여 데이터 레이크, 웹 사이트, 모바일 애플리케이션, 백업 및 복원, 아카이브, 엔터프라이즈 애플리케이션, IoT 디바이스, 빅 데이터 분석 등 다양한 사용 사례에서 원하는 양의 데이터를 저장하고 보호할 수 있습니다. Amazon S3는 특정 비즈니스, 조직 및 규정 준수 요구 사항에 맞게 데이터에 대한 액세스를 최적화, 구조화 및 구성할 수 있는 관리 기능을 제공합니다.` <span style="color:gray; font-size:2px;">- 출처: AWS Docs<span>

<br>

Amazon S3는 데이터를 저장하고 검색할 수 있는 간단한 API를 제공하는 완전 관리형 스토리지 서비스입니다. 즉, Amazon S3에 저장하는 데이터는 특정 서버와 관련이 없으므로 사용자가 인프라를 직접 관리할 필요가 없습니다. 또한 일반적인 파일 서버는 사용자 트래픽이 증가하면 스토리지 증설 작업을 해야하지만 S3는 시스템 적으로 트래픽 증가에 대한 처리를 미리 해두었기 때문에 파일 서버 관리자는 별도의 처리를 해주지 않아도 됩니다. bucket(최상위 디렉토리)이라는 폴더에 Object(S3에 저장되는 데이터)인 파일을 저장하여 사용합니다. 🌏[Amazon S3 Simple Storage Service 요금 - AWS](https://aws.amazon.com/ko/s3/pricing/)이곳에서 요금에 대한 정보를 확인하실 수 있습니다.

<br>

#### **AWS Step Functions**

`AWS Step Functions은 시각적 워크플로우를 사용해 분산 애플리케이션 및 마이크로서비스의 구성 요소를 손쉽게 조정하도록 해주는 웹 서비스입니다. 각각 기능 또는 작업을 수행하는 개별 구성 요소를 사용하여 애플리케이션을 구축하면 애플리케이션을 빠르게 확장하거나 변경할 수 있습니다.` <span style="color:gray; font-size:2px;">- 출처: AWS Docs<span>

<br>

쉽게 설명하면 AWS의 여러 컴퓨팅 자원들의 수행 순서를 시각적으로 설정할 수 있는 서비스입니다. 또한, AWS Lambda같은 경우는 상태 값이 없어서 데이터를 확인하려면 DB에서 데이터를 확인했는데, Step Functions을 활용해서 Lambda 함수를 각 단계에 설정할 수 있다고 합니다. Step Function은 진행중인 단계에 맞게 처리할 데이터를 Lambda 함수에 보내주게할 수 있는데, 이렇게 하면 원하는 상태 정보를 알 수 있게 되고 각각의 워크플로우들을 한눈에 관리할 수 있다고 합니다.(이전에 처음으로 Lambda를 이용할 때는 Step Functions대해 몰랐었었는데, 한번 써볼 기회가 생겼으면 좋겠네요!😃)

<br>

#### **Amazon CloudFront**

`Amazon CloudFront는 .html, .css, .js 및 이미지 파일과 같은 정적 및 동적 웹 콘텐츠를 사용자에게 더 빨리 배포하도록 지원하는 웹 서비스입니다. CloudFront는 엣지 로케이션이라고 하는 데이터 센터의 전 세계 네트워크를 통해 콘텐츠를 제공합니다. CloudFront를 통해 서비스하는 콘텐츠를 사용자가 요청하면 지연 시간이 가장 낮은 엣지 로케이션으로 요청이 라우팅되므로 가능한 최고의 성능으로 콘텐츠가 제공됩니다.` <span style="color:gray; font-size:2px;">- 출처: AWS Docs<span>

<br>

즉, **CloudFront는 AWS에서 제공하는 CDN 서비스** 입니다. 사용자로부터 요청이 발생하면 요청이 발생한 **Edge Server(컨텐츠들이 캐시에 보관되어지는 장소)** 은 요청이 발생한 데이터에 대하여 캐싱 여부를 확인합니다. 그리고 캐싱 데이터가 존재하면 사용자에 요청에 맞게 응답하고 존재하지 않으면 **Origin Server(원본 데이터를 가지고 있는 서버, S3/Ec2 instance)** 로 요청합니다. 요청 받은 데이터에 대해 Origin Server로부터 전달 받은 Edge Server는 캐싱 데이터를 생성하고 사용자에게 응답하게 됩니다.

> **CDN(Content Delivery Network, 콘텐츠 전송 네트워크)** 이란? 지리적 제약 없이 전 세계 사용자에게 빠르고 안전하게 콘텐츠를 전송할 수 있는 콘텐츠 전송 기술입니다.

<br>

![CloudFront에서 사용자에게 콘텐츠를 제공하는 방법]({{ site.baseurl }}/assets/images/streaming/CloudFront.png)*이미지 5. CloudFront에서 사용자에게 콘텐츠를 제공하는 방법(출처: AWS Docs)*

<br>

CloudFront 함수는 글로벌 CloudFront 이벤트에 대한 응답으로 함수를 실행할 때마다 호출 횟수를 계산합니다. 자세한 요금 사항은 🌏[Amazon CloudFront CDN - 요금제 및 요금](https://aws.amazon.com/ko/cloudfront/pricing/)에서 확인하실 수 있습니다.

<br>
<br>


> 끝맺음

오늘은 스트리밍 서비스에 대해 정리를 해보았습니다. 저는 미디어에 대한 지식이 많이 부족하기 때문에, 생각보다 정리할 것이 많았고(깊게 들어가면 훨씬 많은 내용이 있습니다..) 적지 않은 시간을 쓴 것 같습니다.🥲 다행히 Live Streaming과 VOD streaming 두 가지다 AWS Media Services를 사용한 워크플로(AWS에서 제시해 준)로 구축할 수 있기 때문에 조금 더 쉽게 공부를 한 것 같습니다. 

AWS Media Services의 서비스들이 아직 서울 리전이 나온 지 얼마 안 되어서 그런지 AWS를 이용한 미디어 서비스 구축 후기에 관한 한국 자료는 많이 찾아볼 수 없었습니다. 그래도 AWS에서 서비스에 대한 정보를 잘 제공해 주기 때문에, 스트리밍 서비스 구축에 고민하고 계신 분들은 AWS Media Services를 잘 활용한다면 안전하고 빠르게 스트리밍 서비스를 구축해볼 수 있을 것 같습니다. 부족한 내용이지만 읽어주셔서 감사합니다.😀 잘못된 내용은 댓글로 남겨주시면 감사하겠습니다!


<br>
<br>


**[참고자료]**

- [AWS 미디어서비스를 통한 라이브 및 온디맨드 동영상 워크플로 구성 및 배포하기 - 김영진:: AWS Builders Online Series](https://www.youtube.com/watch?v=fdoaUq49dSA)
- [AWS Docs](https://docs.aws.amazon.com/)
- [AWS](https://aws.amazon.com/ko/media-services/)
- [이두잉의 AWS 세상 - AWS Step Functions 이해](https://blog.leedoing.com/184)
- [AWS CloudFront - 개념](https://velog.io/@doohyunlm/AWS-CloudFront-개념)
