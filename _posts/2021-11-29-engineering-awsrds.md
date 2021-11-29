---
layout: post
title:  "AWS RDS란? 그리고, Aurora와의 차이"
subtitle:   AWS RDS란? 그리고, Aurora와의 차이
categories: datascience
tags: engineering cloud aws rds ongoing
comments: true
---

# 목차
1. [RDS 란](#rds-란)
2. RDS vs Aurora
0. [Reference](#reference)
<br>

---

<br>

# RDS 란

> RDS(Relational Database Service)란 AWS 클라우드에서 관계형 데이터베이스를 사용할 수 있는  서비스로, 크기 조절이 가능한 용량을 제공하고 관리 작업을 관리합니다. 하드웨어 프로비저닝, 데이터베이스 설정, 패치 및 백업과 같은 관리 작업을 자동화할 수 있습니다. <br><br> - *[AWS Docs](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/UserGuide/Welcome.html)*

Amazon Aurora, MySQL, MariaDB, Oracle, PostgreSQL, SQL Server 등 6개의 DB 엔진을 제공합니다. <br> Amazon RDS의 기본 블록은 DB 인스턴스로 제공되며, 각 DB 인스턴스는 DB 엔진을 실행합니다. DB 인스턴스의 계산 및 메모리 용량은 해당 **DB 인스턴스 클래스**에 의해 결정되며, DB 인스턴스는 추후 변경 가능합니다.


DB 인스턴스 스토리지는 마그네틱, 범용(SSD) 및 프로비저닝된 IOPS(Input/Output Operation Per Second) 등 세 가지 유형이 제공됩니다. 이 세가지 유형은 성능 특성과 가격이 다르므로, 요구 사항을 충족하는 성능의 DB 인스턴스를 선택하세요.
> [*RDS 인스턴스 유형별 요금 참고*](https://aws.amazon.com/ko/rds/mysql/pricing/)

<br>

#### RDS 이용 시 고려해야할 사항
- RDS는 DNS만을 통해 접근할 수 있다.
- 사용자는 root 권한이 주어지지 않는다.
- Multi-AZ 사용 시 EC2 대비 30~50% 정도 비싸다.
- 가장 많은 문제가 IOPS로 인해 발생된다. 따라서 적당한 사이즈의 EBS를 사용해야한다.
- Oracle, SQL Server와 같은 상용 플랫폼 이용시, License를 확인해야 한다.







<br>

# Reference
> - [카니슈카](https://kanisuka.tistory.com/43)님의 블로그: AWS RDS에서 Aurora vs MySQL
> - [Popit](https://www.popit.kr/aws-ebselastic-block-storage%EC%9D%98-%EB%B9%84%EC%9A%A9-%EC%B5%9C%EC%A0%81%ED%99%94/): AWS EBS 비용 최적화
> - [이두잉](https://blog.leedoing.com/43)님의 블로그: AWS RDS 요약 정리
> - [회사원 krheyjin](https://onborders.tistory.com/85)님의 블로그: AWS 데이터 베이스 정리