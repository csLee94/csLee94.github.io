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
2. Serverless Framework
3. Reference

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