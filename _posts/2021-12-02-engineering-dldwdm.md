---
layout: post
title:  "Data Lake, Data Warehouse, Data Mart 분류 및 비교"
subtitle:   Data Lake, Data Warehouse, Data Mart 분류 및 비교
categories: datascience
tags: engineering datalake datawarehouse datamart
comments: true
---

# 목차
1. [Data Lake](#data-lake)
2. [Data Warehouse](#data-warehouse)
3. [Data Mart](#data-mart)
4. [Reference](#reference)
<br>

---

<br>

데이터의 저장소인 데이터베이스, DB는 저장 목적과 데이터의 활용처에 따라 크게 3가지 분류로 나눌 수 있습니다.
- **Data Lake**: 원시 데이터를 메타 데이터(날짜, 출처 등)을 그대로 저장하는 비정형 데이터 저장소
- **Data Warehouse**: 여러 곳에서 발생하는 데이터를 전사적 구조에 맞춰 저장하는 정형 데이터 저장소
- **Data Mart**: 특정 주제 및 요구사항으로 엮은 Warehouse의 하위 집합 격인 데이터 저장소

||Data Lake|Data Warehouse|Data Mart|
|---|---|---|---|
|Usage|고급 예측 분석|운영 및 성능 분석|프런트 라인 비즈니스 보고|
|User|Low|High|Low|
|Cost|Very High|Medium-to-High|Low|
|Data growth|Very High|Low-to-Medium|Low|
|Time-to-Market|Weeks, Months|Weeks, days, hour (Depending on Approach)|Minutes, hours|
|Type|Big data OLAP|OLAP|OLTP|

<br>

![img](https://drive.google.com/uc?id=1Qzir766Ie7r6K93w17oSbm3Xn2DVvH8C)

<br>

![img](https://drive.google.com/uc?id=1OjRp_T8w_8nvjtO2UAIu_YpbIxcVzmPe)

<br>

# Data Lake
Data Lake는 구조화, 반구조화, 원시 데이터 형태로 저장된 비정형의 저장소이며, 관계형 데이터, CSV나 json, 기계 및 센서 데이터의 형태로 저장됩니다.

Data Lake는 모든 유형의 데이터를 그대로 유지하도록 설계되었고, 조직에서 대규모로 다양한 유형의 데이터를 생성되지만 분석 단위 및 분석 주제가 아직 정해지지 않은 상태에서 사용됩니다. 생성되는 데이터들을 별도로 가공처리 하지 않고 **원시 데이터**로 저장함으로써 분석에서 좀 더 근본적인 접근이 가능하게 합니다. 일반적으로 분석을 위한 데이터 엔지니어 및 데이터 사이언티스트가 필요로 합니다.

기술의 발전으로 엄청난 양의 데이터를 저렴한 비용으로 빠르고 쉽게 저장할 수 있게 되어, 그만큼 중요성이 대두되고 있습니다. 또한, 데이터 파이프라인의 중요한 구성 요소 중 하나입니다.

Data Lake로 유입되는 데이터들은 자신의 출처와 시간 같은 Meta Data가 존재해야 합니다. 또한 Data Lake는 스키마가 존재하지 않으므로, 이 원시 데이터를 어떻게 사용할지는 전적으로 사용자에게 달려있습니다.

Data Lake는 그 크기가 매우 커질 것이고, 대부분의 저장소는 스키마가 없는 큰 규모의 구조를 지향하기 때문에 일반적으로 Data Lake를 구현할 때 Hadoop과 HDFS를 비롯한 에코시스템이 사용됩니다.

> *Data Lake라는 용어는 데이터 통합 및 분석 플랫폼인 펜타호(Pentaho)의 설립자인 제임스 딕슨이 만들었습니다.*

<br>

# Data Warehouse

복수의 소스에서 데이터를 집계한다는 점에서 Data Lake와 유사하지만, 이 데이터가 저장되기 전에 **구조화** 된다는 점에서 차이점을 가집니다. 때문에 Data Warehouse에 비해 유연성이 다소 떨어집니다. 또한, 구조화 된 데이터를 저장하기 때문에 빠른 쿼리에 최적화되어 있습니다. 

다양한 원천에서 발생하는 데이터를 이용자에게 전달하기 전에 통합해 저장하는 공간으로, 서로 다른 구조나 용어 등에 따른 문제가 야기됩니다. 때문에 전사적 관점에서 통합 과정이 요구된다.

> Data Warehouse로 통합되는 과정에서 정제되거나, 버려지는 데이터가 발생하게 됩니다. '선택'의 과정에서 '선택 기준'이 모호할 때를 위해 Data Lake가 대두됩니다.

체계적이고 읽기 쉬운 스키마 온-리딩(schema-on-read) 특성으로 기술 인력 부족에도 더 쉽게 분석할 수 있습니다. 다시 말해, 비즈니스 분석가/마케터/재무 팀 등에서 비교적 데이터를 쉽게 사용할 수 있습니다.

<br>

# Data Mart

Data Mart는 Data Warehouse의 **하위 집합**으로, 특정 운영 부서 또는 주체의 보고 요구를 충족하도록 설계됩니다. 즉, Data Warehouse에서 **특정 주제로 묶인 섹션**으로 볼 수 있습니다. 따라서 특정 주제의 Data를 찾기 위해 "검색"에 대한 리소스를 줄일 수 있습니다.

Data Mart는 Data Warehouse 설계 시, 핵심 고려 사항입니다. Data Warehouse를 구축하는 방법은 **부서 별로 데이터를 통합, 모델링하고 개별 Data Mart를 만든 다음 하나로 묶어 Enterprise DataWarehouse를 구성**하는 것입니다. 이는 보다 민첩한 접근 방식으로 기업에서 비즈니스 프로세스에 대한 심층적인 이해를 높이고 개별 DataMart를 도출하기 위해 특정 요구사항에 집중할 수 있습니다.

<br>

# Reference
> - [우주먼지](https://rk1993.tistory.com/entry/DataLake-VS-DataWarehouse-VS-DataMart-%EB%B9%84%EA%B5%90)님의 블로그
> - [ehyun_kor](https://ehyun0128.github.io/miscellaneous/dm_dw_dl/)님의 블로그
> - [loustler](https://loustler.io/data_eng/diff-data_lake-data_warehouse/)님의 블로그 