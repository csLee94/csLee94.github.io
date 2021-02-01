---
layout: post
title:  "시계열 자료 분석 방법"
subtitle:   시계열 자료 분석 방법
categories: lecture
tags: analistic statistics timeseries
comments: true
---

# 목차
1. [ARIMA](#arima)

<br>

# ARIMA
0. **용어**
    |Word|Word Detail|Description|
    |---|---|---|
    |Stationary|정상성|시간에 관계없이 평균과 분산이 일정한 데이터|
    |Nonstationary|비정상성|시간에 따라서 평균 혹은 분산이 일정하지 않은 데이터|
    |ACF|Autocorrelation Function|현재 데이터와 t시점의 데이터와의 correlation|
    |ACF plot|Autocorrelation plot|ACF에 대한 그래프|
    |AR|Autogressive model|서로 다른 시점을 가지고 특정 시점에 대한 데이터를 Regressioin|
    |MA|Moving Average model|연속적인 error를 가지고 특정 시점에 대한 데이터를 Regression|

    <br>

    - 시계열 데이터의 정상성 확인
        1. ACF(Autocorrelation Function) Plot<br>
        ACF plot에서 천천히 떨어지거나, 특정한 패턴이 있는 경우 Nonstationary<br>
        lag1,2에서 급격하게 떨어지거나, 특정한 패턴이 없을 경우 Stationary 
        > 이 방법은 그래프를 기반한 주관적인 방법이다.
        
    - AR, MA ARIMA Model



# 참고자료
- [ARIMA 개념_1](https://www.youtube.com/watch?v=ma_L2YRWMHI) : 김성범_[ARIMA 모델 개요 part-1]
- [ARIMA 개념_2](https://www.youtube.com/watch?v=P_3808Xv76Q&t=619s) : 김성범_[ARIMA 모델 개요 part-2]
- https://byeongkijeong.github.io/ARIMA-with-Python/
