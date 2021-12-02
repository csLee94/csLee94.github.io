---
layout: post
title:  "Data Lake, Data Warehouse, Data Mart 분류 및 비교"
subtitle:   Data Lake, Data Warehouse, Data Mart 분류 및 비교
categories: datascience
tags: engineering datalake datawarehouse datamart ongoing
comments: true
---

# 목차
1. [Data Lake](#data-lake)
2. Data Warehouse
3. Data Mart
4. [Reference](#reference)
<br>

---

<br>

데이터의 저장소인 데이터베이스, DB는 저장 목적과 데이터의 활용처에 따라 크게 3가지 분류로 나눌 수 있습니다.
- Data Lake
- Data Warehouse
- Data Mart

# Data Lake
Data Lake는 구조화, 반구조화, 원시 데이터 형태로 저장된 비정형의 저장소이며, 관계형 데이터, CSV나 json, 기계 및 센서 데이터의 형태로 저장됩니다.

Data Lake는 모든 유형의 데이터를 그대로 유지하도록 설계되었고, 조직에서 대규모로 다양한 유형의 데이터를 생성되지만 분석 단위 및 분석 주제가 아직 정해지지 않은 상태에서 사용됩니다. 생성되는 데이터들을 별도로 가공처리 하지 않고 **원시 데이터**로 저장함으로써 분석에서 좀 더 근본적인 접근이 가능하게 합니다. 일반적으로 분석을 위한 데이터 엔지니어 및 데이터 사이언티스트가 필요로 합니다.

기술의 발전으로 엄청난 양의 데이터를 저렴한 비용으로 빠르고 쉽게 저장할 수 있게 되어, 그만큼 중요성이 대두되고 있습니다. 또한, 데이터 파이프라인의 중요한 구성 요소 중 하나입니다.

> *Data Lake라는 용어는 데이터 통합 및 분석 플랫폼인 펜타호(Pentaho)의 설립자인 제임스 딕슨이 만들었습니다.*

<br>

# Data Warehouse

복수의 소스에서 데이터를 집계한다는 점에서 Data Lake와 유사하지만, 이 데이터가 저장되기 전에 **구조화** 된다는 점에서 차이점을 가집니다. 때문에 Data Lake와 차이가 있습니다.






# Reference
> - [우주먼지](https://rk1993.tistory.com/entry/DataLake-VS-DataWarehouse-VS-DataMart-%EB%B9%84%EA%B5%90)님의 블로그
> - [ehyun_kor](https://ehyun0128.github.io/miscellaneous/dm_dw_dl/)님의 블로그
> - [loustler](https://loustler.io/data_eng/diff-data_lake-data_warehouse/)님의 블로그 