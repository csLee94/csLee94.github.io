---
layout: post
title:  "BigQuery를 사용하기 전에 알아두어야하는 배경"
subtitle:   BigQuery를 사용하기 전에 알아두어야하는 배경
categories: datascience
tags: engineering gcp bigquery datawarehouse ongoing
comments: true
---

# 목차
> *이 포스트는 "**구글 빅쿼리 완벽 가이드**" 책을 읽고, 몇몇 래퍼런스를 참고해 남기는 학습용 기록 입니다. - 빌리아파 락쉬마난, 조던 티가니 지음* <br> ![img](https://drive.google.com/uc?id=1-1BVUps4O-9nf_qavAoSemMNYcZHnvPN)

1. [BigQuery vs RDBMS](#bigquery-vs-rdbms)
2. [ETL, EL, ELT](#etl-el-elt)
3. [기타 추가사항](#기타-추가사항)
4. [Reference](#reference)
<br>

<details>
<summary> <b>Seires Post</b> </summary>
<div markdown="1">

*시리즈 업데이트 시 업데이트 예정* 

</div>
</details>

<br>

---

# BigQuery vs RDBMS
개인적으로 BigQuery를 처음 접하면서, 제일 의문이었던 점은 'BigQuery'의 정체였다. 간단히 리서치를 했을 때는 '그냥 성능이 짱 좋은 관계형 데이터베이스 관리 시스템인건가?'라는 바보같은 생각이 들기도 했습니다. 

<br>

## OLTP vs OLAP
> - **OLTP**(Online Transaction Processing): 데이터 자체 처리에 중점을 둔 개념으로, 하나의 Transaction이 모두 성공적으로 처리가 되어야 프로세스가 완료되도록 개발한다. 따라서 빠르고 안전하게 레코드를 삽입(Insert)하는데 특화된다.
> - **OLAP**(Online Analytical Processing): 데이터를 체계적으로 저장해 데이터에 기반한 의사결정지원을 할 수 있도록 주안점을 두며, Insight 도출에 중심을 둔다. 따라서 다양한 분석업무를 수행할 때, 쿼리 작업을 속도감있게 진행할 수 있도록 데이터를 읽는데 특화된다.

||**OLTP**|**OLAP**|
|---|---|---|
|목적|Transaction 처리|데이터 분석과 보고서 작성 & 데이터 시각화|
|설계|앱 기능 지향|비즈니스 주체 지향|
|데이터|운영계에서 실시간 최신 데이터|정보계에서 통합/이력 데이터|
|크기|기가 바이트, 스냅샷|테라 데이터, 아카이브|
|SQL 쿼리| 단순 트랜잭션 및 빈번한 갱신|복잡한 집계 쿼리|
|사용자|아주 많음|분석가 포함 일부|

<br>

우리가 흔히 사용하는 MySQL이나 PostgreSQL 같은 OLTP 데이터베이스는 Ad hoc Query 실행에는 적합하지 않습니다.
> *여기서 Ad hoc query란 인덱스 같은 기능을 사용해 최상의 성능을 발휘하도록 미리 만들어 둔 데이터베이스의 쿼리가 아니라 필요에 따라 (성능이 떨어져도) 즉석에서 작성해 실행하는 쿼리를 뜻합니다.*

<br>

## 맵리듀스 프레임워크
OLTP 데이터베이스는 전체 데이터셋의 순회가 필요한 경우에 적합하지 않아서, 특수 목적의 분석은 자바나 파이썬 같은 언어로 수행했습니다. 그러나, 2003년 key/value 쌍을 처리해 중간 key/value 쌍을 생성하는 **map함수**와 동일한 중간 키와 연관된 모든 중간 값을 병합하는 **reduce**함수가 **맵리듀스**라는 패러다임으로 알려지게 되었다.

> '맵리듀스'는 아파치 하둡의 발전을 이끌게 됩니다. 

다만 이 경우, 하둡 클러스터 관리, 모니터링, 프로비저닝 대한 전문적 지식이 필요합니다.


<br>

## Big Query
Big Query가 '마법'과 같다는 평을 듣는건, 인프라스트럭처를 운영하지 않아도, RDBMS처럼 SQL 쿼리를 실행할 수 있고, 맵리듀스처럼 전체 데이터셋 탐색을 효율적으로 분산할 수 있기 때문입니다. 서버리스 서비스이므로 인프라를 관리할 필요가 없고, 전체 데이터셋에 대한 집계를 수 초에서 수 분 내에 처리하는 분석을 수행할 수 있습니다.

<br>

# ETL, EL, ELT




<br>



# Reference
> - [이광춘님의 POST](http://aispiration.com/data-science/ds-rdbms.html): OLAP, OLTP