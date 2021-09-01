---
layout: post
title:  "서버리스 개념과 Serverless Framework"
subtitle:   서버리스 개념과 Serverless Framework
categories: datascience
tags: dev serverless, server, aws, lambda
comments: true
---

# 목차
1. [서버리스(Serverless)의 개념](#serverless의-개념)
2. [AWS Lambda](#aws-lambda)
3. [Serverless Framework](#serverless-framework)
4. Reference

---
<br>

# Serverless의 개념

- **Serverless는 백엔드에 서버가 없다?** [ X ]
    > **Serverless**는 백엔드인데, 직접 서버를 관리하지 않는 경우를 뜻함 <br>
*Backend without Server management* *by. [Nomad Coders](https://www.youtube.com/channel/UCUpJs89fSBXNolQGOYKn0YQ)*


- Serverless의 탄생 배경
    <br>
    1. 기존에는 서버의 하드웨어와 소프트웨어를 둘 다 관리 (물리적 서버 및 메모리 관리 등)
    2. 아마존의 EC2 서비스를 시작으로 서버를 '설치'하는 대신 아마존에 돈을 지불하고 서버를  대여.<br> 아마존과 같은 업체들이 돈을 받고, 서버의 하드웨어 부분을 관리
    3. 그러나 여전히, 업데이트/보안/데이터 백업 등 소프트웨어 관리가 필요
    4. **<mark style='background-color: #ffdce0'>서버리스의 등장</mark>**<br>
    서버리스는 백엔드를 서버에 올리는 것이 아니라, **백엔드를 작은 함수 단위로 쪼개서 직접  관리하지 않는 서버로 업로드**(eg, AWS Lambda)

<br>

- Serverless의 특징
    > 서버리스가 아닌 경우, 서버는 24식나 돌아가며 항상 요청에 응답할 준비 대기<br> **서버리스의 경우** 내가 업로드한 함수들은 잠들어 있고, request가 오는 순간 AWS가 함수를 깨워 작동. 수행을 마치고 다시 Sleep
    - 장점
    1. 저렴한 사용 가격 (항시 서버 운영이 아닌 Request에 반응해 작동)
    2. 스케일 조정이 용이 (유저에 따라 AWS에서 함수를 복제해 작동)
    3. 빠른 제품 출시 가능, 물적 자원 절약 가능
    4. 사이드프로젝트 & 프로토타입 등 서버관리에 시간을 아끼고 싶을 때 유용


    - 단점
    1. Request에 반응해 깨어나는 함수이기 때문에 아주 약간의 지연 시간이 발생
    2. AWS와 같은 서버 제공자에게 의존도가 높아진다.
        > 서버리스로 작업하면 어플리케이션의 구조자체가 바뀌게 되어, 다른 서버로 migration하기 어렵다.
    3. 서버에 대한 통제를 잃고, 구조를 변경해줘야함

<br>

- 학습 시 참고 사이트
    - SERVERLESS.COM
    - AWS Lambda
    - Google Cloud Functions
    - Apex
    - Terraform

> 위 글은 유튜버 <mark style='background-color: #fff5b1'>노마드 코더 Nomad Coders</mark>의 ['서버러시는 서버가 없는걸까? 8분 개념 설명](https://www.youtube.com/watch?v=ufLmReluPww&t=448s) 영상을 정리했습니다.

<br><br>

# AWS Lambda
> ***Lambda**는 서버를 프로비저닝하거나 관리하지 않고도, 코드를 실행할 수 있는 컴퓨팅 서비스입니다. Lambda는 고가용성 컴퓨팅 인프라에서 코드를 실행하고 서버 및 운영 체제 유지 관리/자동확장/코드 모니터링 및 로깅을 비롯한 모든 컴퓨팅 리소스 관리를 수행합니다.<br> Lambda API를 사용하여 Lambda 함수를 호출하거나 다른 AWS 서비스의 이벤트에 응답하여 Lambda 함수를 실행할 수 있습니다. <br>by. [Amazon Guide](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/welcome.html)* 

- 사용가능 언어<br>
 **Python**/ Node.js / Ruby / Java / Go / C# 

<br>

- AWS Lambda Process Summary<br>
    1. Lambda Function 정의<br>
        - 수행할 함수를 정의한다. 
        - Source Code를 작성할 때, 필요한 Library를 함께 Lambda Environment에 업로드해줘야한다.
            > **~에서 업로드** 버튼에 zip 파일 업로드할 수 있다.
    2. Lambda Function's Trigger 설정
        - **트리거 추가**버튼을 통해 Lambda Function을 실행시킬 트리거를 설정한다.
        - 트리거를 **API 게이트웨이**로 설정함으로써 외부 서비스에서 Lambda Function을 실행시킬 호출가능한 API Endpoint를 설정한다.
    3. API Gateway Console에서 생성한 API를 Lambda Function과 통합해주고, HTTP Method를 설정 
        >**POST** Method의 경우 '통합 요청'에서 '매핑 템플릿'설정을 통해 'application/json' 이름의 '메서드 요청 패스쓰루' 템플릿을 지정해준다.


<br><br>

# Serverless Framework
- Serverless Framework는 AWS에서 제공하는 AWS Lambda에서 애플리케이션을 구축하기 위해 개발된 첫 번째 Framework다. AWS Lambda, API Gateway와 같은 Serverless 자원들을 사용하여 Serverless 아키택처를 쉽게 설계, 구현, 배포, 관리할 수 있게 도와준다. 현재 AWS, Google Cloud Platform, Azure, Knative 등 다양한 Cloud 제공 업체에서 지원한다.

- 기본 언어는 Node.js이나 이 외에도 swift, ruby, Go, python, PHP 언어를 지원한다. 또한 활성화되어 있는 Framework 중 하나이기 때문에 관련 Plugin이 굉장히 많다. 이 Plugin을 활용해 HTTP, Tracing, Step Functions 등의 다양한 기능을 활용할 수 있다.

    > - 기본 언어가 Node.js이기 때문에 Serverless Framework를 python으로 사용하기 위해서는 많은 환경설정이 필요하다고 한다.
    > - 특히 사용되는 Library 관련해 Packing하는 작업 관련해서 어려움이 많다고 한다. ~~이를 해결하기 위해 **serverless-python-requirement**라는 Plugin이 나왔다고 하는데 여전히 나에게는 어렵다.~~
    > - [serverless-python-requirement plugins Rerference](https://serverless.com/blog/serverless-python-packaging/)

- **Faas의 단점 보완**: 수정 및 배포 과정 축약<br>
    Faas의 가장 큰 단점은, **코드 수정 및 배포**가 어렵다는 것이다. 코드 한 줄을 수정하더라도 다시 첨부해 업로드해야하고, 코드의 크기가 일정 크기를 초과할 경우 AWS Console에서 수정할 수 없는 상황이 초래되기도 한다. <br>
    Serverless Framework는 CLI 환경에서 간단한 설정 후 명령어 하나로 쉽게 배포할 수 있어, Faas의 배포 문제를 완화시켜 줄 수 있다.

    > Faas(Function-as-a-Service)는 서버리스 컴퓨팅을 구현하는 방식으로 AWS Lambda, Google Cloud Functions, Microsoft Azure Functions 등이 있다.

<br>

## Serverless Framework 다루기
1. 설치









<br><br>
# Reference
> -  [노마드 코더 Nomad Coders](https://www.youtube.com/channel/UCUpJs89fSBXNolQGOYKn0YQ) 유튜브: [서버리스 기초개념](https://www.youtube.com/watch?v=ufLmReluPww&t=448s)
> - [Gyullbb님의 블로그](https://velog.io/@_gyullbb/Serverless-Framework-VS-Chalice-4) : Serverless 개념과 Python에서 serverless framework 다루기
> - [Neon K.I.D님의 블로그](https://blog.neonkid.xyz/140): Serverless Framework를 사용하여 더 쉽게 서버 배포하기