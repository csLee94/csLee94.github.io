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

> AWS 프리티어 RDS로는 `db.t2.micro` 인스턴스이다. <br> <u>단일 AZ</u> 인스턴스를 750시간 사용할 수 있으며, 20GB 범용(SSD) DB 스토리지와 자동 DB 백업, 사용자 실행 DB 스냅샷을 위한 백업 스토리지 20GB가 제공된다.

<br>

### RDS 이용 시 고려해야할 사항
- RDS는 DNS만을 통해 접근할 수 있다.
- 사용자는 root 권한이 주어지지 않는다.
- Multi-AZ 사용 시 EC2 대비 30~50% 정도 비싸다.
- Oracle, SQL Server와 같은 상용 플랫폼 이용시, License를 확인해야 한다.
- **가장 많은 문제가 IOPS로 인해 발생된다. 따라서 적당한 사이즈의 EBS를 사용해야한다.**


#### AWS EBS 비용 최적화

EBS는 EC2 인스턴스나 RDS에 연결되는 스토리지인 **Elastic Block Storage**를 말합니다. EBS의 볼륨 크기 및 종류에 따라 애플리케이션, 데이터베이스의 성능에 큰 영향을 미칩니다. 또한 이에 따라 비용이 크게 달라지므로, 요구되는 성능과 비용을 고려해, 적절한 스토리지 유형과 크기를 잘 선택해야 합니다. <br>
가장 먼저 <u>IOPS</u>를 살펴볼 수 있습니다. 각 DB 엔진의 페이지에 따라 얼마나 높은 처리량(Throughput)이 필요한지 계산해 필요한 EBS 볼륨을 추정해 볼 수 있습니다. <br>
예를 들어, 매초 100MB의 읽기 동작이 필요한 애플리케이션인 경우, 16KB 페이지 크기의 데이터 베이스는 초당 6,400번의 읽기 동작을 해야합니다. 즉, **6,400 IOPS 이상의 성능을 가진 EBS 볼륨이 필요**합니다.

> **IOPS**(Input/Output Operation Per Second) <br>
Amazon 클라우드 저장장치의 성능을 나타내는 가장 중요한 지표 중 하나로, **초당 입출력 속도**를 말합니다. 특히 RDS 인스턴스 설정 시 최대 IOPS가 정해진 스토리지를 선택하게 되는데, 한번 선택하면 할당된 최대 IOPS를 넘을 수 없으므로, 신중히 선택해야 합니다.

#### EBS 스토리지 유형
1. **범용(General Purpose) SSD (gp2)**

    지연시간(Latency)이 한 자리 밀리초가 가능한 매우 빠른 스토리지로, 대부분의 애플리케이션에서 충분한 성능을 냅니다. 16TB의 불륨크기까지 가능하고 기가바이트 당 3배의 IOPS를 할당받을 수 있으며 볼륨당 16,000 IOPS까지 가능합니다. 즉 볼륨크기가 클수록 더욱 빠른 IOPS를 할당 받을 수 있습니다.

    gp2의 최대 대역폭은 250MBps이지만, 이 속도를 내기 위해서는 충분한 EC2와 IOPS를 할당해야 합니다. 만약 스토리지는 많이 필요하지 않으나, 한정적으로 높은 IOPS가 필요하다면, **버스트 버킷**을 사용할 수 있습니다. 

2. **프로비전된(Provisioned) IOPS SSD(io1)**

    gp2 스토리지의 성능을 제대로 계산하기 위해선 '버스트버킷'이라는 구조와 그 계산식을 이해해야 합니다. 만약 이런 계산을 피하고, 미리 정해진 IOPS를 확보하고 싶다면 IOPS SSD io1을 선택할 수 있습니다. io1은 지속적으로 낮은 응답 지연시간이 필요한 OLTP형 데이터베이스에 유용하게 사용할 수 있지만, 스토리지 볼륨 크기와 프로비전된 IOPS양으로 비용이 청구되므로 비용에 대해 검토해야합니다. 

    IOPS와 스토리지 용량은 약 50:1 비율로 규정하고 있으며, 만약 32,000 IOPS를 원한다면 640GB 정도를 프로비전해야 합니다.

3. **처리량 최적화된(Throughput Opimized) HHD (st1)**

    IOPS가 아닌 처리량(Throughput)으로 성능을 정의하는 저비용 HDD 스토리지로, **AWS EMR, 빅데이터, Analytics**와 같은 애플리케이션에 적합합니다. gp2는 IOPS 버스트 버킷 모델로 제공하는 반면, st1은 처리량에 대한 버스트 버킷 모델로 제공하고 있습니다. gp2와 비슷하게 볼륨 크기가 클수록 더 많은 기준 처리량(baseline throughtput)과 버스트 처리량이 비례하여 결정됩니다.

4. **Cold HDD (sc1)**

    자주 액세스하지 않는 애플리케이션에 적합하고, 처리량으로 성능을 정의하는 저비용 HDD 스토리지를 제공합니다. sc1은 대용량 데이터를 순차적으로 액세스하는 애플리케이션에 적합합니다.

    
<br>

# Reference
> - [카니슈카](https://kanisuka.tistory.com/43)님의 블로그: AWS RDS에서 Aurora vs MySQL
> - [Popit](https://www.popit.kr/aws-ebselastic-block-storage%EC%9D%98-%EB%B9%84%EC%9A%A9-%EC%B5%9C%EC%A0%81%ED%99%94/): AWS EBS 비용 최적화
> - [이두잉](https://blog.leedoing.com/43)님의 블로그: AWS RDS 요약 정리
> - [회사원 krheyjin](https://onborders.tistory.com/85)님의 블로그: AWS 데이터 베이스 정리