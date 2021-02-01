---
layout: post
title:  "시계열 기반 미래 매출 예측"
subtitle:   시계열 기반 미래 매출 예측
categories: career
tags: dataanalytics project jupyter-notebook stock LSTM
comments: true
---


# Description



- **과제 주요 목표**: 주어진 Dataset을 통해 2020년 1월의 일자별 NetSales와 Amount를 예측합니다.



- 과제 세부 목표

  - 시계열 데이터 분석 방법론에 대한 이해를 목표로 합니다.

  - 대표적인 시계열 분석 모델인 ARIMA 모델과 딥러닝 모델 중 하나인 LSTM 모델을 활용하여 예측을 진행합니다.

  - Matplotlib와 Seaborn 라이브러리를 통하여 적절한 시각화를 진행합니다. 



<br>



- Dataset에 대한 Column Description<br>



|Columns|Description|Detail|Category|
|---|---|---|---|
|Date|일자|-|범주형|
|Sales|매출액|판매가*수량|연속형|
|NetSales|순매출액|(판매가*수량) * 할인률|연속형|
|MeanSales|평균매출액|해당 일자의 (판매가*수량)/수량|연속형|
|Amount|판매수량|-|연속형|
|Stores|제품 판매 매장 수|-|연속형|
|SKU|제품 타입 개수|판매가 발생한 수|연속형|

> 같은 제품이더라도 SKU에 따라 조금씩 상이한 가격<br>

> 평균매출액은 단위별 다른 가격에 대한 한 제품의 평균 소비자 가격



# Data Import


```
# Google drive mount

from google.colab import drive

drive.mount('/content/drive')
```

    Mounted at /content/drive
    


```
# import basic library

import pandas as pd

import numpy as np 



from matplotlib import pyplot as plt

import seaborn as sns
```


```
df = pd.read_excel('/content/drive/MyDrive/Data/SalesData.xlsx')
```


```
# DataSet의 Columns Name 변경

df.columns = ['Date', 'Sales', 'NetSales', 'Amount', 'MeanSales', 'DiscountRate', 'Stores', 'SKU']

# 기본 Columns 변수 할당

dflst = ['Sales', 'NetSales', 'MeanSales', 'Amount', 'DiscountRate', 'Stores', 'SKU']

# DataFrame 출력 format 설정_ 과학적 표기법 사용X

pd.options.display.float_format = '{:.2f}'.format

df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Sales</th>
      <th>NetSales</th>
      <th>Amount</th>
      <th>MeanSales</th>
      <th>DiscountRate</th>
      <th>Stores</th>
      <th>SKU</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2017-06-01</td>
      <td>2502000</td>
      <td>2383400</td>
      <td>18</td>
      <td>139000.00</td>
      <td>0.05</td>
      <td>48</td>
      <td>4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2017-06-02</td>
      <td>6086000</td>
      <td>5984800</td>
      <td>45</td>
      <td>135244.44</td>
      <td>0.02</td>
      <td>48</td>
      <td>8</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2017-06-03</td>
      <td>12778000</td>
      <td>12510600</td>
      <td>92</td>
      <td>138891.30</td>
      <td>0.02</td>
      <td>48</td>
      <td>8</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2017-06-04</td>
      <td>10842000</td>
      <td>10631400</td>
      <td>78</td>
      <td>139000.00</td>
      <td>0.02</td>
      <td>48</td>
      <td>7</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-06-05</td>
      <td>5560000</td>
      <td>5522100</td>
      <td>41</td>
      <td>135609.76</td>
      <td>0.01</td>
      <td>48</td>
      <td>7</td>
    </tr>
  </tbody>
</table>
</div>



## `요일` 변수 생성 


```
df['Day'] = 'temp'

tlst = ['Mon', 'Tue', 'Wed', 'Thur', 'Fri', 'Sat', 'Sun']

#2017/06/05 월요일 ~ 

for i in range(len(df)):

  df['Day'][i+4] = tlst[i%7]

# 2017/06/01 목요일 ~ 2017/06/04 일요일

for i in range(4):

  df['Day'][i] = tlst[i+3]



df.head(10)
```

    /usr/local/lib/python3.6/dist-packages/ipykernel_launcher.py:5: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      """
    /usr/local/lib/python3.6/dist-packages/ipykernel_launcher.py:8: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Sales</th>
      <th>NetSales</th>
      <th>Amount</th>
      <th>MeanSales</th>
      <th>DiscountRate</th>
      <th>Stores</th>
      <th>SKU</th>
      <th>Day</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2017-06-01</td>
      <td>2502000</td>
      <td>2383400</td>
      <td>18</td>
      <td>139000.00</td>
      <td>0.05</td>
      <td>48</td>
      <td>4</td>
      <td>Thur</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2017-06-02</td>
      <td>6086000</td>
      <td>5984800</td>
      <td>45</td>
      <td>135244.44</td>
      <td>0.02</td>
      <td>48</td>
      <td>8</td>
      <td>Fri</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2017-06-03</td>
      <td>12778000</td>
      <td>12510600</td>
      <td>92</td>
      <td>138891.30</td>
      <td>0.02</td>
      <td>48</td>
      <td>8</td>
      <td>Sat</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2017-06-04</td>
      <td>10842000</td>
      <td>10631400</td>
      <td>78</td>
      <td>139000.00</td>
      <td>0.02</td>
      <td>48</td>
      <td>7</td>
      <td>Sun</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-06-05</td>
      <td>5560000</td>
      <td>5522100</td>
      <td>41</td>
      <td>135609.76</td>
      <td>0.01</td>
      <td>48</td>
      <td>7</td>
      <td>Mon</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2017-06-06</td>
      <td>12371000</td>
      <td>12113300</td>
      <td>89</td>
      <td>139000.00</td>
      <td>0.02</td>
      <td>48</td>
      <td>7</td>
      <td>Tue</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2017-06-07</td>
      <td>3967500</td>
      <td>3801700</td>
      <td>29</td>
      <td>136810.34</td>
      <td>0.04</td>
      <td>48</td>
      <td>8</td>
      <td>Wed</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2017-06-08</td>
      <td>5282000</td>
      <td>5282000</td>
      <td>38</td>
      <td>139000.00</td>
      <td>0.00</td>
      <td>48</td>
      <td>8</td>
      <td>Thur</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2017-06-09</td>
      <td>10147000</td>
      <td>9702000</td>
      <td>73</td>
      <td>139000.00</td>
      <td>0.04</td>
      <td>48</td>
      <td>7</td>
      <td>Fri</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2017-06-10</td>
      <td>13344000</td>
      <td>13080800</td>
      <td>97</td>
      <td>137567.01</td>
      <td>0.02</td>
      <td>48</td>
      <td>9</td>
      <td>Sat</td>
    </tr>
  </tbody>
</table>
</div>



# Data EDA

## 기술통계

- pandas의 Describe에 왜도와 첨도를 추가한 기술통계 확인



- Seaborn library를 이용하여 기초 EDA 진행

  1. distplot을 확인하여 Data 분포 시각화

  2. boxplot을 확인하여 Data의 분포 및 이상치 시각화


```
eda = df.describe()



# Describe에 추가할 Column 별 왜도 첨도

sklst = []

kulst =[]

for txt in dflst:

  sklst.append(df[txt].skew())

  kulst.append(df[txt].kurt())

skulst = [sklst, kulst]

skudf = pd.DataFrame(skulst, columns=eda.columns, index=['Skew','Kurt'])



eda = eda.append(skudf)

eda
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Sales</th>
      <th>NetSales</th>
      <th>Amount</th>
      <th>MeanSales</th>
      <th>DiscountRate</th>
      <th>Stores</th>
      <th>SKU</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>944.00</td>
      <td>944.00</td>
      <td>944.00</td>
      <td>944.00</td>
      <td>944.00</td>
      <td>944.00</td>
      <td>944.00</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>8407918.43</td>
      <td>7658885.36</td>
      <td>57.44</td>
      <td>147049.30</td>
      <td>0.09</td>
      <td>45.87</td>
      <td>13.03</td>
    </tr>
    <tr>
      <th>std</th>
      <td>4959462.28</td>
      <td>4504263.47</td>
      <td>34.17</td>
      <td>9739.52</td>
      <td>0.08</td>
      <td>5.94</td>
      <td>4.42</td>
    </tr>
    <tr>
      <th>min</th>
      <td>219000.00</td>
      <td>163400.00</td>
      <td>1.00</td>
      <td>124250.00</td>
      <td>0.00</td>
      <td>28.00</td>
      <td>2.00</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>4998750.00</td>
      <td>4525550.00</td>
      <td>34.00</td>
      <td>139000.00</td>
      <td>0.04</td>
      <td>45.00</td>
      <td>10.00</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>7057500.00</td>
      <td>6462550.00</td>
      <td>48.00</td>
      <td>144814.89</td>
      <td>0.07</td>
      <td>46.00</td>
      <td>13.00</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>10891250.00</td>
      <td>10195725.00</td>
      <td>75.00</td>
      <td>152967.27</td>
      <td>0.13</td>
      <td>49.00</td>
      <td>16.00</td>
    </tr>
    <tr>
      <th>max</th>
      <td>37391000.00</td>
      <td>25800000.00</td>
      <td>269.00</td>
      <td>219000.00</td>
      <td>0.65</td>
      <td>56.00</td>
      <td>30.00</td>
    </tr>
    <tr>
      <th>Skew</th>
      <td>1.21</td>
      <td>1.02</td>
      <td>1.57</td>
      <td>1.26</td>
      <td>2.21</td>
      <td>-1.28</td>
      <td>0.31</td>
    </tr>
    <tr>
      <th>Kurt</th>
      <td>1.83</td>
      <td>0.76</td>
      <td>5.57</td>
      <td>2.20</td>
      <td>8.26</td>
      <td>2.23</td>
      <td>-0.17</td>
    </tr>
  </tbody>
</table>
</div>




```
# Dataset 분포도 & Boxplot 

i=1

plt.figure(figsize=(9,25))

for txt in dflst:

  plt.subplot(7,2,i)

  sns.distplot(df[txt])

  i+=1

  plt.subplot(7,2,i)

  sns.boxplot(data=df, x= txt)

  i +=1



plt.show()
```

    /usr/local/lib/python3.6/dist-packages/seaborn/distributions.py:2557: FutureWarning: `distplot` is a deprecated function and will be removed in a future version. Please adapt your code to use either `displot` (a figure-level function with similar flexibility) or `histplot` (an axes-level function for histograms).
      warnings.warn(msg, FutureWarning)
    /usr/local/lib/python3.6/dist-packages/seaborn/distributions.py:2557: FutureWarning: `distplot` is a deprecated function and will be removed in a future version. Please adapt your code to use either `displot` (a figure-level function with similar flexibility) or `histplot` (an axes-level function for histograms).
      warnings.warn(msg, FutureWarning)
    /usr/local/lib/python3.6/dist-packages/seaborn/distributions.py:2557: FutureWarning: `distplot` is a deprecated function and will be removed in a future version. Please adapt your code to use either `displot` (a figure-level function with similar flexibility) or `histplot` (an axes-level function for histograms).
      warnings.warn(msg, FutureWarning)
    /usr/local/lib/python3.6/dist-packages/seaborn/distributions.py:2557: FutureWarning: `distplot` is a deprecated function and will be removed in a future version. Please adapt your code to use either `displot` (a figure-level function with similar flexibility) or `histplot` (an axes-level function for histograms).
      warnings.warn(msg, FutureWarning)
    /usr/local/lib/python3.6/dist-packages/seaborn/distributions.py:2557: FutureWarning: `distplot` is a deprecated function and will be removed in a future version. Please adapt your code to use either `displot` (a figure-level function with similar flexibility) or `histplot` (an axes-level function for histograms).
      warnings.warn(msg, FutureWarning)
    /usr/local/lib/python3.6/dist-packages/seaborn/distributions.py:2557: FutureWarning: `distplot` is a deprecated function and will be removed in a future version. Please adapt your code to use either `displot` (a figure-level function with similar flexibility) or `histplot` (an axes-level function for histograms).
      warnings.warn(msg, FutureWarning)
    /usr/local/lib/python3.6/dist-packages/seaborn/distributions.py:2557: FutureWarning: `distplot` is a deprecated function and will be removed in a future version. Please adapt your code to use either `displot` (a figure-level function with similar flexibility) or `histplot` (an axes-level function for histograms).
      warnings.warn(msg, FutureWarning)
    


    
![png](https://drive.google.com/uc?id=1odGh0WgPQ9aT_PSbtUmK7Z5l_39v_00o)
    


<h4>기초 기술통계를 통해 다음과 같은 사실을 유추</h4>



<br>





1. Sales보다 NetSales의 첨도가 낮고, 비교적 분포도의 꼬리가 짧은 것으로 보아 Sales보단 밀집도가 높은 NetSales를 토대로 예측하는 것이 용이할 것 같다.



2. Sales, NetSales, MeanSales, DiscountRate 모두 오른쪽으로 긴 꼬리를 가지고 있다. 이를 토대로 다음과 같은 가정을 세워볼 수 있다.

  - 할인기간, 잉여자본 생성시기 등 어떤 이유 때문에 고객들의 소비가 특정 기간에 집중된다.

  - 오른쪽의 긴 꼬리는 단순한 이상치일 뿐, 주요 판매 매출 데이터는 밀도가 높은 왼쪽의 데이터이다.





## 연도, 요일, 일자 별 판매 데이터 EDA


```
#Date column YY-MM-DD 분할 Column 생성

df['YY'] = 'temp'

df['MM'] = 'temp'

df['DD'] = 'temp'

for i in range(len(df)):

  tstr = df.iloc[i]['Date']

  [yy, mm, dd] = str(tstr).split("-")

  dd = dd.split(" ")[0]

  df['YY'].iloc[i] = yy

  df['MM'].iloc[i] = mm

  df['DD'].iloc[i] = dd



df = df[['Date', 'YY', 'MM', 'DD', 'Day', 'Sales', 'NetSales', 'Amount', 'MeanSales', 'DiscountRate', 'Stores', 'SKU']]

df.head(10)
```

    /usr/local/lib/python3.6/dist-packages/pandas/core/indexing.py:670: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      iloc._setitem_with_indexer(indexer, value)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>YY</th>
      <th>MM</th>
      <th>DD</th>
      <th>Day</th>
      <th>Sales</th>
      <th>NetSales</th>
      <th>Amount</th>
      <th>MeanSales</th>
      <th>DiscountRate</th>
      <th>Stores</th>
      <th>SKU</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2017-06-01</td>
      <td>2017</td>
      <td>06</td>
      <td>01</td>
      <td>Thur</td>
      <td>2502000</td>
      <td>2383400</td>
      <td>18</td>
      <td>139000.00</td>
      <td>0.05</td>
      <td>48</td>
      <td>4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2017-06-02</td>
      <td>2017</td>
      <td>06</td>
      <td>02</td>
      <td>Fri</td>
      <td>6086000</td>
      <td>5984800</td>
      <td>45</td>
      <td>135244.44</td>
      <td>0.02</td>
      <td>48</td>
      <td>8</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2017-06-03</td>
      <td>2017</td>
      <td>06</td>
      <td>03</td>
      <td>Sat</td>
      <td>12778000</td>
      <td>12510600</td>
      <td>92</td>
      <td>138891.30</td>
      <td>0.02</td>
      <td>48</td>
      <td>8</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2017-06-04</td>
      <td>2017</td>
      <td>06</td>
      <td>04</td>
      <td>Sun</td>
      <td>10842000</td>
      <td>10631400</td>
      <td>78</td>
      <td>139000.00</td>
      <td>0.02</td>
      <td>48</td>
      <td>7</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-06-05</td>
      <td>2017</td>
      <td>06</td>
      <td>05</td>
      <td>Mon</td>
      <td>5560000</td>
      <td>5522100</td>
      <td>41</td>
      <td>135609.76</td>
      <td>0.01</td>
      <td>48</td>
      <td>7</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2017-06-06</td>
      <td>2017</td>
      <td>06</td>
      <td>06</td>
      <td>Tue</td>
      <td>12371000</td>
      <td>12113300</td>
      <td>89</td>
      <td>139000.00</td>
      <td>0.02</td>
      <td>48</td>
      <td>7</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2017-06-07</td>
      <td>2017</td>
      <td>06</td>
      <td>07</td>
      <td>Wed</td>
      <td>3967500</td>
      <td>3801700</td>
      <td>29</td>
      <td>136810.34</td>
      <td>0.04</td>
      <td>48</td>
      <td>8</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2017-06-08</td>
      <td>2017</td>
      <td>06</td>
      <td>08</td>
      <td>Thur</td>
      <td>5282000</td>
      <td>5282000</td>
      <td>38</td>
      <td>139000.00</td>
      <td>0.00</td>
      <td>48</td>
      <td>8</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2017-06-09</td>
      <td>2017</td>
      <td>06</td>
      <td>09</td>
      <td>Fri</td>
      <td>10147000</td>
      <td>9702000</td>
      <td>73</td>
      <td>139000.00</td>
      <td>0.04</td>
      <td>48</td>
      <td>7</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2017-06-10</td>
      <td>2017</td>
      <td>06</td>
      <td>10</td>
      <td>Sat</td>
      <td>13344000</td>
      <td>13080800</td>
      <td>97</td>
      <td>137567.01</td>
      <td>0.02</td>
      <td>48</td>
      <td>9</td>
    </tr>
  </tbody>
</table>
</div>




```
# 연도별, column별 데이터 분포 차이 확인

plt.figure(figsize=(12,20))

for i in range(len(dflst)):

  plt.subplot(4,2,i+1)

  sns.boxplot(data=df, x='YY', y=dflst[i])



plt.show()
```


    
![png](https://drive.google.com/uc?id=1KeMhZoYhYBnHWKbRFKwIlRK1bN7YQlMH)
    



```
# 월별, column별 데이터 분포 차이 확인

plt.figure(figsize=(12, 20))

for i in range(len(dflst)):

  plt.subplot(4,2,i+1)

  sns.boxplot(data=df, y=dflst[i], x="MM", order=['01', '02', '03', '04', '05', '06', '07', '08', '09', '10', '11', '12'])



plt.show()
```


    
![png](https://drive.google.com/uc?id=1tNzMyrrMITTkCFWwiTO1s1x75YgYBqCz)
    



```
# 일자별, column별 데이터 분포 차이 확인

plt.figure(figsize=(24,20))

for i in range(len(dflst)):

  plt.subplot(4,2,i+1)

  sns.boxplot(data=df, y=dflst[i], x='DD')



plt.show()
```


    
![png](https://drive.google.com/uc?id=1vaAsCsQDoRz5LOf-wgTuGCXNgLQfRH7r)
    



```
# 요일별, column별 데이터 분포 차이 확인

plt.figure(figsize=(24,20))

for i in range(len(dflst)):

  plt.subplot(4,2,i+1)

  sns.boxplot(data=df, y=dflst[i], x='Day', order=['Mon', 'Tue', 'Wed', 'Thur', 'Fri', 'Sat', 'Sun'])



plt.show()
```


    
![png](https://drive.google.com/uc?id=1ssvU4FpSZGiApZCU4UCjiwe8Qq668blr)
    



```
# 연도별 1년치 추세 그래프

# 월-일 형식의 데이터 생성

tmpdf = df.copy()

tmpdf['MMDD'] = 'temp'

for i in range(len(tmpdf)):

  tmpdf['MMDD'].iloc[i] = str(tmpdf.iloc[i]['MM'])+'-'+str(tmpdf.iloc[i]['DD'])

# 연도별 해당 날짜 매출 그래프

fig = plt.figure(figsize=(36,12))

ax = fig.add_subplot()



ax.plot(tmpdf.loc[tmpdf['YY']=='2019']['MMDD'], tmpdf.loc[df['YY']=='2019']['Sales'], label=2019)

ax.plot(tmpdf.loc[tmpdf['YY']=='2018']['MMDD'], tmpdf.loc[df['YY']=='2018']['Sales'], label=2018)

ax.plot(tmpdf.loc[tmpdf['YY']=='2017']['MMDD'], tmpdf.loc[df['YY']=='2017']['Sales'], label=2017)

plt.xticks(rotation = 90)

ax.legend()



plt.show()
```

    /usr/local/lib/python3.6/dist-packages/pandas/core/indexing.py:670: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      iloc._setitem_with_indexer(indexer, value)
    


    
![png](https://drive.google.com/uc?id=1JATUIEs55Mn4qdfYHGg97RZGUVxxAB8J)
    


- **연도별 데이터**

  1. 연도별 데이터를 boxplot으로 표현했을 시 `Sales`, `NetSales`, `Amount` 모두 근소하게 감소하고 있다.

  2. 반면 `MeanSales`는 꾸준하게 증가하고 있다.<br>

    -> 해당 제품에 대한 가격이 점차 증가하고 있다.

  3. 판매가 진행되는 `Store` 역시 감소 중에 있다.

  4. 손님들이 구매하는 한 제품 내 종류 수(`SKU`)가 증가하고 있다. <br>

    -> 손님들이 다양성을 선호한다.

> 증가하는 SKU를 바탕으로 손님들이 제품에 대한 다양성을 선호한다는 사실을 가정할 수 있고, 증가하는 MeanSales를 바탕으로 제품의 가격이 지속적으로 상승하고 있음을 유추해볼 수 있다.



<br>



- **월별 데이터**

  1. `Sales`, `NetSales`, `Amount` 모두 봄 기간(3월~5월) 기간 동안 가시적으로 감소하는 모습을 보인다.

  2. `DiscountRate`는 매출이 가장 적게 일어나는 봄 기간(3월~5월)이 가장 높다.

  3. 판매가 발생하는 `Store` 역시 봄 기간에 점차 줄어들다가 5월 달에 가시적으로 낮은 값을 보인다.

  4. 여름이 시작하며 증가세로 들어서는 다른 Column과는 달리 `MeanSales`는 6월이 가장 낮다



  > 가을 기간(9, 10, 11) 구매가 가장 많이 일어나고 봄을 제외한 여름과 겨울은 비슷한 매출을 보인다. 날씨에 영향을 많이 받지는 않지만, 환절기에 많이 사용하는 제품이라고 특성을 유추해볼 수 있다.



<br>



- **일자별&요일별 데이터**

  1. 일자별로 나누어봤을 때 크게 유의미한 인사이트는 없는 것 같다.

  2. 요일별로 나누어 봤을 때 평일과 주말의 가시적인 차이가 있고, 특히 평일 매출은 굉장히 중앙 밀집적이다. 반면 주말의 경우 다소 분포가 넓은 것을 확인할 수 있다.



  > 판매가 일어나는 매장에 따라 매출이 크게 영향을 받으며 평일과 주말에 대한 매출값이 크게 차이가 나는 것으로 보아 온라인 채널 데이터가 반영되지 않은 오프라인 채널 매출 데이터임을 유추할 수 있다. 또한 미래 매출 데이터를 예측하고자 할 때 평일과 주말이 가시적으로 차이가 나고, 분포가 중앙 밀집적인 것을 감안했을 때 요일 변수가 유의미할 수 있을 것 같다. 

# ARIMA

# LSTM

LSTM 모델을 이용하여 주어진 Dataset의 다음 달인 2020.01의 일자별 데이터를 예측해본다. 



- 예측 대상: `NetSales`, `Amount`

- 사용 변수: `NetSales`, `Amount`, `요일`

- 예측 방법: 현재 시점을 기준으로 7일 전까지, 총 7일동안의 데이터를 활용하여 30일 뒤 매출을 예측

- 예측 방법 선택 이유

  - 주어진 Dataset의 기간이 다소 짧아 충분한 학습에 대한 우려가 있다.

  - 31일 간의 데이터를 예측할 때 예측 주기가 짧아지면, 예측을 반복할수록 x_data에 학습된 예측값이 반복적으로 포함되어 오차가 증가하게 된다. 

  - 이를 방지하고자 최대한 가지고 있는 Dataset 안에서 해결하고자 30일 뒤를 예측하는 모델로 설정


```
#import library

from sklearn.preprocessing import MinMaxScaler

from sklearn.preprocessing import StandardScaler

import tensorflow as tf

from tensorflow.keras.layers import LSTM

from tensorflow.keras.models import Sequential

from tensorflow.keras.layers import Dense



from sklearn.metrics import mean_squared_error 
```

## Data Preprocessing


```
tempdf = df.copy()

tempdf.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>YY</th>
      <th>MM</th>
      <th>DD</th>
      <th>Day</th>
      <th>Sales</th>
      <th>NetSales</th>
      <th>Amount</th>
      <th>MeanSales</th>
      <th>DiscountRate</th>
      <th>Stores</th>
      <th>SKU</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2017-06-01</td>
      <td>2017</td>
      <td>06</td>
      <td>01</td>
      <td>Thur</td>
      <td>2502000</td>
      <td>2383400</td>
      <td>18</td>
      <td>139000.00</td>
      <td>0.05</td>
      <td>48</td>
      <td>4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2017-06-02</td>
      <td>2017</td>
      <td>06</td>
      <td>02</td>
      <td>Fri</td>
      <td>6086000</td>
      <td>5984800</td>
      <td>45</td>
      <td>135244.44</td>
      <td>0.02</td>
      <td>48</td>
      <td>8</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2017-06-03</td>
      <td>2017</td>
      <td>06</td>
      <td>03</td>
      <td>Sat</td>
      <td>12778000</td>
      <td>12510600</td>
      <td>92</td>
      <td>138891.30</td>
      <td>0.02</td>
      <td>48</td>
      <td>8</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2017-06-04</td>
      <td>2017</td>
      <td>06</td>
      <td>04</td>
      <td>Sun</td>
      <td>10842000</td>
      <td>10631400</td>
      <td>78</td>
      <td>139000.00</td>
      <td>0.02</td>
      <td>48</td>
      <td>7</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-06-05</td>
      <td>2017</td>
      <td>06</td>
      <td>05</td>
      <td>Mon</td>
      <td>5560000</td>
      <td>5522100</td>
      <td>41</td>
      <td>135609.76</td>
      <td>0.01</td>
      <td>48</td>
      <td>7</td>
    </tr>
  </tbody>
</table>
</div>




```
num = 37 # 30일 뒤 예측 + 7일간의 학습 데이터



for i in range(num):

  tempdf['Date_{}'.format(i)] = tempdf['Date'].shift(i)

  tempdf['Net_{}'.format(i)] = tempdf['NetSales'].shift(i)

  tempdf['Amt_{}'.format(i)] = tempdf['Amount'].shift(i)

  

tempdf = tempdf.dropna()



netlst = ['Net_%d' %(i) for i in range(num-7,num)]

amtlst = ['Amt_%d' %(i) for i in range(num-7,num)]

datelst = ['Date_%d' %(i) for i in range(num-7,num)]



tempdf = tempdf[['Date','Day', 'NetSales', 'Amount']+netlst+amtlst+datelst]

tempdf.index = [i for i in range(len(tempdf))]







tempdf
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Day</th>
      <th>NetSales</th>
      <th>Amount</th>
      <th>Net_30</th>
      <th>Net_31</th>
      <th>Net_32</th>
      <th>Net_33</th>
      <th>Net_34</th>
      <th>Net_35</th>
      <th>Net_36</th>
      <th>Amt_30</th>
      <th>Amt_31</th>
      <th>Amt_32</th>
      <th>Amt_33</th>
      <th>Amt_34</th>
      <th>Amt_35</th>
      <th>Amt_36</th>
      <th>Date_30</th>
      <th>Date_31</th>
      <th>Date_32</th>
      <th>Date_33</th>
      <th>Date_34</th>
      <th>Date_35</th>
      <th>Date_36</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2017-07-07</td>
      <td>Fri</td>
      <td>6710700</td>
      <td>52</td>
      <td>3801700.00</td>
      <td>12113300.00</td>
      <td>5522100.00</td>
      <td>10631400.00</td>
      <td>12510600.00</td>
      <td>5984800.00</td>
      <td>2383400.00</td>
      <td>29.00</td>
      <td>89.00</td>
      <td>41.00</td>
      <td>78.00</td>
      <td>92.00</td>
      <td>45.00</td>
      <td>18.00</td>
      <td>2017-06-07</td>
      <td>2017-06-06</td>
      <td>2017-06-05</td>
      <td>2017-06-04</td>
      <td>2017-06-03</td>
      <td>2017-06-02</td>
      <td>2017-06-01</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2017-07-08</td>
      <td>Sat</td>
      <td>19366200</td>
      <td>149</td>
      <td>5282000.00</td>
      <td>3801700.00</td>
      <td>12113300.00</td>
      <td>5522100.00</td>
      <td>10631400.00</td>
      <td>12510600.00</td>
      <td>5984800.00</td>
      <td>38.00</td>
      <td>29.00</td>
      <td>89.00</td>
      <td>41.00</td>
      <td>78.00</td>
      <td>92.00</td>
      <td>45.00</td>
      <td>2017-06-08</td>
      <td>2017-06-07</td>
      <td>2017-06-06</td>
      <td>2017-06-05</td>
      <td>2017-06-04</td>
      <td>2017-06-03</td>
      <td>2017-06-02</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2017-07-09</td>
      <td>Sun</td>
      <td>19945300</td>
      <td>152</td>
      <td>9702000.00</td>
      <td>5282000.00</td>
      <td>3801700.00</td>
      <td>12113300.00</td>
      <td>5522100.00</td>
      <td>10631400.00</td>
      <td>12510600.00</td>
      <td>73.00</td>
      <td>38.00</td>
      <td>29.00</td>
      <td>89.00</td>
      <td>41.00</td>
      <td>78.00</td>
      <td>92.00</td>
      <td>2017-06-09</td>
      <td>2017-06-08</td>
      <td>2017-06-07</td>
      <td>2017-06-06</td>
      <td>2017-06-05</td>
      <td>2017-06-04</td>
      <td>2017-06-03</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2017-07-10</td>
      <td>Mon</td>
      <td>8032500</td>
      <td>62</td>
      <td>13080800.00</td>
      <td>9702000.00</td>
      <td>5282000.00</td>
      <td>3801700.00</td>
      <td>12113300.00</td>
      <td>5522100.00</td>
      <td>10631400.00</td>
      <td>97.00</td>
      <td>73.00</td>
      <td>38.00</td>
      <td>29.00</td>
      <td>89.00</td>
      <td>41.00</td>
      <td>78.00</td>
      <td>2017-06-10</td>
      <td>2017-06-09</td>
      <td>2017-06-08</td>
      <td>2017-06-07</td>
      <td>2017-06-06</td>
      <td>2017-06-05</td>
      <td>2017-06-04</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-07-11</td>
      <td>Tue</td>
      <td>10302900</td>
      <td>80</td>
      <td>16889300.00</td>
      <td>13080800.00</td>
      <td>9702000.00</td>
      <td>5282000.00</td>
      <td>3801700.00</td>
      <td>12113300.00</td>
      <td>5522100.00</td>
      <td>125.00</td>
      <td>97.00</td>
      <td>73.00</td>
      <td>38.00</td>
      <td>29.00</td>
      <td>89.00</td>
      <td>41.00</td>
      <td>2017-06-11</td>
      <td>2017-06-10</td>
      <td>2017-06-09</td>
      <td>2017-06-08</td>
      <td>2017-06-07</td>
      <td>2017-06-06</td>
      <td>2017-06-05</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>903</th>
      <td>2019-12-27</td>
      <td>Fri</td>
      <td>3432600</td>
      <td>25</td>
      <td>3407800.00</td>
      <td>4063900.00</td>
      <td>4838000.00</td>
      <td>11124100.00</td>
      <td>11830700.00</td>
      <td>6067500.00</td>
      <td>3958500.00</td>
      <td>26.00</td>
      <td>29.00</td>
      <td>36.00</td>
      <td>80.00</td>
      <td>84.00</td>
      <td>44.00</td>
      <td>30.00</td>
      <td>2019-11-27</td>
      <td>2019-11-26</td>
      <td>2019-11-25</td>
      <td>2019-11-24</td>
      <td>2019-11-23</td>
      <td>2019-11-22</td>
      <td>2019-11-21</td>
    </tr>
    <tr>
      <th>904</th>
      <td>2019-12-28</td>
      <td>Sat</td>
      <td>7174700</td>
      <td>52</td>
      <td>3976600.00</td>
      <td>3407800.00</td>
      <td>4063900.00</td>
      <td>4838000.00</td>
      <td>11124100.00</td>
      <td>11830700.00</td>
      <td>6067500.00</td>
      <td>26.00</td>
      <td>26.00</td>
      <td>29.00</td>
      <td>36.00</td>
      <td>80.00</td>
      <td>84.00</td>
      <td>44.00</td>
      <td>2019-11-28</td>
      <td>2019-11-27</td>
      <td>2019-11-26</td>
      <td>2019-11-25</td>
      <td>2019-11-24</td>
      <td>2019-11-23</td>
      <td>2019-11-22</td>
    </tr>
    <tr>
      <th>905</th>
      <td>2019-12-29</td>
      <td>Sun</td>
      <td>10488900</td>
      <td>75</td>
      <td>5861800.00</td>
      <td>3976600.00</td>
      <td>3407800.00</td>
      <td>4063900.00</td>
      <td>4838000.00</td>
      <td>11124100.00</td>
      <td>11830700.00</td>
      <td>41.00</td>
      <td>26.00</td>
      <td>26.00</td>
      <td>29.00</td>
      <td>36.00</td>
      <td>80.00</td>
      <td>84.00</td>
      <td>2019-11-29</td>
      <td>2019-11-28</td>
      <td>2019-11-27</td>
      <td>2019-11-26</td>
      <td>2019-11-25</td>
      <td>2019-11-24</td>
      <td>2019-11-23</td>
    </tr>
    <tr>
      <th>906</th>
      <td>2019-12-30</td>
      <td>Mon</td>
      <td>5313000</td>
      <td>38</td>
      <td>11402500.00</td>
      <td>5861800.00</td>
      <td>3976600.00</td>
      <td>3407800.00</td>
      <td>4063900.00</td>
      <td>4838000.00</td>
      <td>11124100.00</td>
      <td>80.00</td>
      <td>41.00</td>
      <td>26.00</td>
      <td>26.00</td>
      <td>29.00</td>
      <td>36.00</td>
      <td>80.00</td>
      <td>2019-11-30</td>
      <td>2019-11-29</td>
      <td>2019-11-28</td>
      <td>2019-11-27</td>
      <td>2019-11-26</td>
      <td>2019-11-25</td>
      <td>2019-11-24</td>
    </tr>
    <tr>
      <th>907</th>
      <td>2019-12-31</td>
      <td>Tue</td>
      <td>6307600</td>
      <td>44</td>
      <td>9677500.00</td>
      <td>11402500.00</td>
      <td>5861800.00</td>
      <td>3976600.00</td>
      <td>3407800.00</td>
      <td>4063900.00</td>
      <td>4838000.00</td>
      <td>68.00</td>
      <td>80.00</td>
      <td>41.00</td>
      <td>26.00</td>
      <td>26.00</td>
      <td>29.00</td>
      <td>36.00</td>
      <td>2019-12-01</td>
      <td>2019-11-30</td>
      <td>2019-11-29</td>
      <td>2019-11-28</td>
      <td>2019-11-27</td>
      <td>2019-11-26</td>
      <td>2019-11-25</td>
    </tr>
  </tbody>
</table>
<p>908 rows × 25 columns</p>
</div>




```
daydf = pd.get_dummies(tempdf['Day'])[tlst]

daydf
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Mon</th>
      <th>Tue</th>
      <th>Wed</th>
      <th>Thur</th>
      <th>Fri</th>
      <th>Sat</th>
      <th>Sun</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>903</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>904</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>905</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>906</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>907</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>908 rows × 7 columns</p>
</div>




```
# 정규화 & 표준화 Scaler for xdata(NetSales, Amount)

x_mn_scaler = MinMaxScaler()

x_std_scaler = StandardScaler()



# x_data scaler

x_data = x_mn_scaler.fit_transform(tempdf[netlst+amtlst])

x_data = x_std_scaler.fit_transform(x_data)

x_data = pd.DataFrame(x_data)

x_data.columns = ['Net_%d' %(i) for i in range(1,8)]+['Amt_%d' %(i) for i in range(1,8)]

# x_data_scale + day dummies

x_data = pd.concat([x_data, daydf],axis=1)

x_data
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Net_1</th>
      <th>Net_2</th>
      <th>Net_3</th>
      <th>Net_4</th>
      <th>Net_5</th>
      <th>Net_6</th>
      <th>Net_7</th>
      <th>Amt_1</th>
      <th>Amt_2</th>
      <th>Amt_3</th>
      <th>Amt_4</th>
      <th>Amt_5</th>
      <th>Amt_6</th>
      <th>Amt_7</th>
      <th>Mon</th>
      <th>Tue</th>
      <th>Wed</th>
      <th>Thur</th>
      <th>Fri</th>
      <th>Sat</th>
      <th>Sun</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-0.86</td>
      <td>0.97</td>
      <td>-0.48</td>
      <td>0.64</td>
      <td>1.05</td>
      <td>-0.38</td>
      <td>-1.17</td>
      <td>-0.84</td>
      <td>0.90</td>
      <td>-0.49</td>
      <td>0.58</td>
      <td>0.99</td>
      <td>-0.38</td>
      <td>-1.16</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-0.53</td>
      <td>-0.86</td>
      <td>0.97</td>
      <td>-0.48</td>
      <td>0.64</td>
      <td>1.05</td>
      <td>-0.38</td>
      <td>-0.58</td>
      <td>-0.84</td>
      <td>0.90</td>
      <td>-0.49</td>
      <td>0.58</td>
      <td>0.99</td>
      <td>-0.38</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.44</td>
      <td>-0.53</td>
      <td>-0.86</td>
      <td>0.97</td>
      <td>-0.48</td>
      <td>0.64</td>
      <td>1.05</td>
      <td>0.44</td>
      <td>-0.58</td>
      <td>-0.84</td>
      <td>0.90</td>
      <td>-0.49</td>
      <td>0.58</td>
      <td>0.99</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1.18</td>
      <td>0.44</td>
      <td>-0.53</td>
      <td>-0.86</td>
      <td>0.97</td>
      <td>-0.48</td>
      <td>0.64</td>
      <td>1.13</td>
      <td>0.44</td>
      <td>-0.57</td>
      <td>-0.84</td>
      <td>0.90</td>
      <td>-0.49</td>
      <td>0.58</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2.02</td>
      <td>1.18</td>
      <td>0.44</td>
      <td>-0.53</td>
      <td>-0.86</td>
      <td>0.97</td>
      <td>-0.48</td>
      <td>1.95</td>
      <td>1.13</td>
      <td>0.44</td>
      <td>-0.58</td>
      <td>-0.84</td>
      <td>0.90</td>
      <td>-0.49</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>903</th>
      <td>-0.94</td>
      <td>-0.80</td>
      <td>-0.63</td>
      <td>0.75</td>
      <td>0.90</td>
      <td>-0.36</td>
      <td>-0.82</td>
      <td>-0.92</td>
      <td>-0.84</td>
      <td>-0.63</td>
      <td>0.64</td>
      <td>0.75</td>
      <td>-0.40</td>
      <td>-0.81</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>904</th>
      <td>-0.82</td>
      <td>-0.94</td>
      <td>-0.80</td>
      <td>-0.63</td>
      <td>0.75</td>
      <td>0.90</td>
      <td>-0.36</td>
      <td>-0.92</td>
      <td>-0.92</td>
      <td>-0.84</td>
      <td>-0.63</td>
      <td>0.64</td>
      <td>0.75</td>
      <td>-0.40</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>905</th>
      <td>-0.40</td>
      <td>-0.82</td>
      <td>-0.94</td>
      <td>-0.80</td>
      <td>-0.63</td>
      <td>0.75</td>
      <td>0.90</td>
      <td>-0.49</td>
      <td>-0.92</td>
      <td>-0.92</td>
      <td>-0.84</td>
      <td>-0.64</td>
      <td>0.64</td>
      <td>0.75</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>906</th>
      <td>0.81</td>
      <td>-0.40</td>
      <td>-0.82</td>
      <td>-0.94</td>
      <td>-0.80</td>
      <td>-0.63</td>
      <td>0.75</td>
      <td>0.64</td>
      <td>-0.49</td>
      <td>-0.92</td>
      <td>-0.92</td>
      <td>-0.84</td>
      <td>-0.64</td>
      <td>0.64</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>907</th>
      <td>0.43</td>
      <td>0.81</td>
      <td>-0.40</td>
      <td>-0.82</td>
      <td>-0.95</td>
      <td>-0.80</td>
      <td>-0.63</td>
      <td>0.29</td>
      <td>0.64</td>
      <td>-0.49</td>
      <td>-0.92</td>
      <td>-0.93</td>
      <td>-0.84</td>
      <td>-0.64</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>908 rows × 21 columns</p>
</div>




```
net_mn_scaler = MinMaxScaler()

net_std_scaler = StandardScaler()

amt_mn_scaler = MinMaxScaler()

amt_std_scaler = StandardScaler()



# y1(NetSale) scaler

tnp = np.array(tempdf['NetSales']).reshape(-1,1)

y1_data = net_mn_scaler.fit_transform(tnp)

y1_data = net_std_scaler.fit_transform(y1_data)



#y2(Amount) scaler

tnp=np.array(tempdf['Amount']).reshape(-1,1)

y2_data = amt_mn_scaler.fit_transform(tnp)

y2_data = amt_std_scaler.fit_transform(y2_data)




```


```
# split train/test set

#train set

x_train = x_data[:-30]

y1_train = y1_data[:-30]

y2_train = y2_data[:-30]

#test set

x_test = x_data[-30:]

y1_test = y1_data[-30:]

y2_test = y2_data[-30:]


```


```
# x_data reshape for LSTM Model

x_train_np = np.array(x_train).reshape(len(x_train), 3, 7)

x_test_np = np.array(x_test).reshape(len(x_test),3, 7)



# y_data reshape for LSTM Model

y1_train_np = y1_train.reshape(len(y1_train),1,1)

# y1_test_np = y1_test.reshape(len(y1_test),1,1)



y2_train_np = y2_train.reshape(len(y2_train),1,1)

# y2_test_np = y2_test.reshape(len(y2_test),1,1)
```

## NetSales


```
net_model = tf.keras.models.Sequential()

net_model.add(LSTM(7, input_shape=(x_train_np.shape[1], x_train_np.shape[2])))

# net_model.add(Dense(7))

net_model.add(Dense(1))

net_model.compile(loss='mean_squared_error', optimizer='adam')



net_model.fit(x_train_np, y1_train_np, epochs=100, batch_size=1)
```

    Epoch 1/100
    878/878 [==============================] - 3s 2ms/step - loss: 1.0543
    Epoch 2/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.7481
    Epoch 3/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.5630
    Epoch 4/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.6479
    Epoch 5/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.5505
    Epoch 6/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.6344
    Epoch 7/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.5275
    Epoch 8/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.5634
    Epoch 9/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.5259
    Epoch 10/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.5653
    Epoch 11/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4760
    Epoch 12/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.5139
    Epoch 13/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4571
    Epoch 14/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.5504
    Epoch 15/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.5202
    Epoch 16/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4532
    Epoch 17/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4245
    Epoch 18/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4354
    Epoch 19/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4493
    Epoch 20/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4820
    Epoch 21/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4590
    Epoch 22/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4272
    Epoch 23/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.5488
    Epoch 24/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4545
    Epoch 25/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4586
    Epoch 26/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4925
    Epoch 27/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4516
    Epoch 28/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4857
    Epoch 29/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4463
    Epoch 30/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3956
    Epoch 31/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4218
    Epoch 32/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4858
    Epoch 33/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4579
    Epoch 34/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3849
    Epoch 35/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4199
    Epoch 36/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4502
    Epoch 37/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4141
    Epoch 38/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.3872
    Epoch 39/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4363
    Epoch 40/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.3950
    Epoch 41/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3763
    Epoch 42/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4777
    Epoch 43/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4536
    Epoch 44/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4841
    Epoch 45/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.3983
    Epoch 46/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3763
    Epoch 47/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4342
    Epoch 48/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4257
    Epoch 49/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4199
    Epoch 50/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4190
    Epoch 51/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3646
    Epoch 52/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3906
    Epoch 53/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4139
    Epoch 54/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3998
    Epoch 55/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3655
    Epoch 56/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3792
    Epoch 57/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.3852
    Epoch 58/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.3747
    Epoch 59/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.3727
    Epoch 60/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4190
    Epoch 61/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.3928
    Epoch 62/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3912
    Epoch 63/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4063
    Epoch 64/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3776
    Epoch 65/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3997
    Epoch 66/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3949
    Epoch 67/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3704
    Epoch 68/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3759
    Epoch 69/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3674
    Epoch 70/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4091
    Epoch 71/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3955
    Epoch 72/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4557
    Epoch 73/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3556
    Epoch 74/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3701
    Epoch 75/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3687
    Epoch 76/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3689
    Epoch 77/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3832
    Epoch 78/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3714
    Epoch 79/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4191
    Epoch 80/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4038
    Epoch 81/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3961
    Epoch 82/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3962
    Epoch 83/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3444
    Epoch 84/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3833
    Epoch 85/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3411
    Epoch 86/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3516
    Epoch 87/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3437
    Epoch 88/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4357
    Epoch 89/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3962
    Epoch 90/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3930
    Epoch 91/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3785
    Epoch 92/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3047
    Epoch 93/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4021
    Epoch 94/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3767
    Epoch 95/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3255
    Epoch 96/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3874
    Epoch 97/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3276
    Epoch 98/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.3517
    Epoch 99/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.3511
    Epoch 100/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4066
    




    <tensorflow.python.keras.callbacks.History at 0x7f1c8d6eca90>




```
pred_net = net_model.predict(x_test_np)

# rmse 산출

# real_amt = np.array(y_scale_test_amt).reshape(30,1)

MSE_net = mean_squared_error(y1_test, pred_net) 

np.sqrt(MSE_net)
```




    0.5606738254427226




```
pd.concat([pd.DataFrame(pred_net), pd.DataFrame(y1_test)], axis=1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-0.50</td>
      <td>-0.52</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-0.13</td>
      <td>-0.52</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-0.22</td>
      <td>-0.56</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-0.18</td>
      <td>-0.61</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-0.20</td>
      <td>-0.67</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1.27</td>
      <td>0.50</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1.04</td>
      <td>0.48</td>
    </tr>
    <tr>
      <th>7</th>
      <td>-0.34</td>
      <td>-0.84</td>
    </tr>
    <tr>
      <th>8</th>
      <td>-0.26</td>
      <td>-0.72</td>
    </tr>
    <tr>
      <th>9</th>
      <td>-0.35</td>
      <td>-0.40</td>
    </tr>
    <tr>
      <th>10</th>
      <td>-0.35</td>
      <td>-0.93</td>
    </tr>
    <tr>
      <th>11</th>
      <td>-0.11</td>
      <td>-0.08</td>
    </tr>
    <tr>
      <th>12</th>
      <td>1.32</td>
      <td>-0.13</td>
    </tr>
    <tr>
      <th>13</th>
      <td>1.14</td>
      <td>1.00</td>
    </tr>
    <tr>
      <th>14</th>
      <td>-0.50</td>
      <td>-0.93</td>
    </tr>
    <tr>
      <th>15</th>
      <td>-0.29</td>
      <td>-0.68</td>
    </tr>
    <tr>
      <th>16</th>
      <td>-0.18</td>
      <td>-0.44</td>
    </tr>
    <tr>
      <th>17</th>
      <td>-0.35</td>
      <td>-1.04</td>
    </tr>
    <tr>
      <th>18</th>
      <td>-0.33</td>
      <td>-0.62</td>
    </tr>
    <tr>
      <th>19</th>
      <td>1.02</td>
      <td>0.31</td>
    </tr>
    <tr>
      <th>20</th>
      <td>0.96</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>21</th>
      <td>-0.64</td>
      <td>-0.53</td>
    </tr>
    <tr>
      <th>22</th>
      <td>-0.52</td>
      <td>0.02</td>
    </tr>
    <tr>
      <th>23</th>
      <td>0.00</td>
      <td>0.85</td>
    </tr>
    <tr>
      <th>24</th>
      <td>-0.34</td>
      <td>-0.43</td>
    </tr>
    <tr>
      <th>25</th>
      <td>-0.71</td>
      <td>-0.93</td>
    </tr>
    <tr>
      <th>26</th>
      <td>0.99</td>
      <td>-0.09</td>
    </tr>
    <tr>
      <th>27</th>
      <td>0.77</td>
      <td>0.65</td>
    </tr>
    <tr>
      <th>28</th>
      <td>-0.98</td>
      <td>-0.50</td>
    </tr>
    <tr>
      <th>29</th>
      <td>-0.65</td>
      <td>-0.28</td>
    </tr>
  </tbody>
</table>
</div>



## Amout


```
amt_model = tf.keras.models.Sequential()

amt_model.add(LSTM(7, input_shape=(x_train_np.shape[1], x_train_np.shape[2])))

# amt_model.add(Dense(len(list(x_train_np))))

# amt_model.add(Dense(0.5*len(list(x_train_np))))

# amt_model.add(Dense(7))

amt_model.add(Dense(1))

amt_model.compile(loss='mean_squared_error', optimizer='adam')



amt_model.fit(x_train_np, y2_train_np, epochs=100, batch_size=1)
```

    Epoch 1/100
    878/878 [==============================] - 3s 2ms/step - loss: 1.0823
    Epoch 2/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.6926
    Epoch 3/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.6971
    Epoch 4/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.6220
    Epoch 5/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.6640
    Epoch 6/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.5421
    Epoch 7/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.6477
    Epoch 8/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.6008
    Epoch 9/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.5007
    Epoch 10/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.5461
    Epoch 11/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.5084
    Epoch 12/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.6251
    Epoch 13/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.6218
    Epoch 14/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.5735
    Epoch 15/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4899
    Epoch 16/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.5741
    Epoch 17/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.7140
    Epoch 18/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.5706
    Epoch 19/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4918
    Epoch 20/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4605
    Epoch 21/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4733
    Epoch 22/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.5284
    Epoch 23/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.7367
    Epoch 24/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4814
    Epoch 25/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.5190
    Epoch 26/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4264
    Epoch 27/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4259
    Epoch 28/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.5092
    Epoch 29/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.5442
    Epoch 30/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4985
    Epoch 31/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4609
    Epoch 32/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4187
    Epoch 33/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4617
    Epoch 34/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4270
    Epoch 35/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4555
    Epoch 36/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4274
    Epoch 37/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4205
    Epoch 38/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3918
    Epoch 39/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4364
    Epoch 40/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4865
    Epoch 41/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4770
    Epoch 42/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4785
    Epoch 43/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4559
    Epoch 44/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4661
    Epoch 45/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4279
    Epoch 46/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4270
    Epoch 47/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4498
    Epoch 48/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4141
    Epoch 49/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4226
    Epoch 50/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.5104
    Epoch 51/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4154
    Epoch 52/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4591
    Epoch 53/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.5339
    Epoch 54/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3946
    Epoch 55/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4392
    Epoch 56/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4336
    Epoch 57/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.3995
    Epoch 58/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4126
    Epoch 59/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4226
    Epoch 60/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4680
    Epoch 61/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3932
    Epoch 62/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3865
    Epoch 63/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4507
    Epoch 64/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3941
    Epoch 65/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4325
    Epoch 66/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4120
    Epoch 67/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4689
    Epoch 68/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.3998
    Epoch 69/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4342
    Epoch 70/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4594
    Epoch 71/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4992
    Epoch 72/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3478
    Epoch 73/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4363
    Epoch 74/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3886
    Epoch 75/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4027
    Epoch 76/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4428
    Epoch 77/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.5462
    Epoch 78/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4113
    Epoch 79/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4391
    Epoch 80/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.4094
    Epoch 81/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.5584
    Epoch 82/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4211
    Epoch 83/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3668
    Epoch 84/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3668
    Epoch 85/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4104
    Epoch 86/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4156
    Epoch 87/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4229
    Epoch 88/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4297
    Epoch 89/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3294
    Epoch 90/100
    878/878 [==============================] - 1s 2ms/step - loss: 0.3367
    Epoch 91/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4628
    Epoch 92/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3370
    Epoch 93/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3837
    Epoch 94/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3802
    Epoch 95/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3744
    Epoch 96/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3665
    Epoch 97/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3758
    Epoch 98/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4709
    Epoch 99/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.3754
    Epoch 100/100
    878/878 [==============================] - 2s 2ms/step - loss: 0.4208
    




    <tensorflow.python.keras.callbacks.History at 0x7f1c8a3f0828>




```
pred_amt = amt_model.predict(x_test_np)

# rmse 산출

# real_amt = np.array(y_scale_test_amt).reshape(30,1)

MSE_amt = mean_squared_error(y2_test, pred_amt) 

np.sqrt(MSE_amt)
```




    0.5683343702014361




```
pd.concat([pd.DataFrame(pred_amt), pd.DataFrame(y2_test)], axis=1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-0.47</td>
      <td>-0.58</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-0.28</td>
      <td>-0.70</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-0.20</td>
      <td>-0.70</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-0.29</td>
      <td>-0.67</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-0.12</td>
      <td>-0.79</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1.17</td>
      <td>0.36</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1.20</td>
      <td>0.31</td>
    </tr>
    <tr>
      <th>7</th>
      <td>-0.40</td>
      <td>-0.85</td>
    </tr>
    <tr>
      <th>8</th>
      <td>-0.35</td>
      <td>-0.76</td>
    </tr>
    <tr>
      <th>9</th>
      <td>-0.45</td>
      <td>-0.37</td>
    </tr>
    <tr>
      <th>10</th>
      <td>-0.36</td>
      <td>-0.94</td>
    </tr>
    <tr>
      <th>11</th>
      <td>-0.35</td>
      <td>-0.40</td>
    </tr>
    <tr>
      <th>12</th>
      <td>1.04</td>
      <td>-0.11</td>
    </tr>
    <tr>
      <th>13</th>
      <td>1.28</td>
      <td>0.78</td>
    </tr>
    <tr>
      <th>14</th>
      <td>-0.37</td>
      <td>-0.97</td>
    </tr>
    <tr>
      <th>15</th>
      <td>-0.43</td>
      <td>-0.79</td>
    </tr>
    <tr>
      <th>16</th>
      <td>-0.21</td>
      <td>-0.46</td>
    </tr>
    <tr>
      <th>17</th>
      <td>-0.15</td>
      <td>-1.11</td>
    </tr>
    <tr>
      <th>18</th>
      <td>-0.39</td>
      <td>-0.70</td>
    </tr>
    <tr>
      <th>19</th>
      <td>0.87</td>
      <td>0.34</td>
    </tr>
    <tr>
      <th>20</th>
      <td>1.06</td>
      <td>0.01</td>
    </tr>
    <tr>
      <th>21</th>
      <td>-0.47</td>
      <td>-0.52</td>
    </tr>
    <tr>
      <th>22</th>
      <td>-0.34</td>
      <td>-0.14</td>
    </tr>
    <tr>
      <th>23</th>
      <td>0.42</td>
      <td>0.69</td>
    </tr>
    <tr>
      <th>24</th>
      <td>0.06</td>
      <td>-0.49</td>
    </tr>
    <tr>
      <th>25</th>
      <td>-0.91</td>
      <td>-0.94</td>
    </tr>
    <tr>
      <th>26</th>
      <td>1.05</td>
      <td>-0.14</td>
    </tr>
    <tr>
      <th>27</th>
      <td>0.79</td>
      <td>0.54</td>
    </tr>
    <tr>
      <th>28</th>
      <td>-0.47</td>
      <td>-0.55</td>
    </tr>
    <tr>
      <th>29</th>
      <td>-0.65</td>
      <td>-0.37</td>
    </tr>
  </tbody>
</table>
</div>



## Validation


```
net_p = net_mn_scaler.inverse_transform(net_std_scaler.inverse_transform(pred_net))

net_r = net_mn_scaler.inverse_transform(net_std_scaler.inverse_transform(y1_test))



plt.figure(figsize=(8,6))

plt.plot(net_p, color='orange', label='Predict')

plt.plot(net_r, color='blue', label= 'Real')

plt.legend(loc='best')

plt.show()
```


    
![png](https://drive.google.com/uc?id=1Bol4rYwvDgNgnaoGl7mGG0lWtXZT3FOi)
    



```
amt_p = amt_mn_scaler.inverse_transform(amt_std_scaler.inverse_transform(pred_amt))

amt_r = amt_mn_scaler.inverse_transform(amt_std_scaler.inverse_transform(y2_test))



plt.figure(figsize=(8,6))

plt.plot(amt_p, color='orange', label='Predict')

plt.plot(amt_r, color='blue', label= 'Real')

plt.legend(loc='best')

plt.show()
```


    
![png](https://drive.google.com/uc?id=1NrKTA6dVKE-m5QE0nLvUoUpMds-A28OJ)
    


## Result


```
tdf = pd.DataFrame(df[['Date', 'Day', 'NetSales', 'Amount']][-36:])

for i in range(7):

  tdf['Date_{}'.format(i)] = tdf['Date'].shift(i)

  tdf['Net_{}'.format(i)] = tdf['NetSales'].shift(i)

  tdf['Amt_{}'.format(i)] = tdf['Amount'].shift(i)



tdf = tdf.dropna()
```


```
netlst2 = ['Net_%d' %(i) for i in range(7)]

amtlst2 = ['Amt_%d' %(i) for i in range(7)]

datelst2 = ['Date_%d' %(i) for i in range(7)]



tdf = tdf[['Day']+netlst2+amtlst2+datelst2]

tdf.index = [i for i in range(len(tdf))]

tdf.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Day</th>
      <th>Net_0</th>
      <th>Net_1</th>
      <th>Net_2</th>
      <th>Net_3</th>
      <th>Net_4</th>
      <th>Net_5</th>
      <th>Net_6</th>
      <th>Amt_0</th>
      <th>Amt_1</th>
      <th>Amt_2</th>
      <th>Amt_3</th>
      <th>Amt_4</th>
      <th>Amt_5</th>
      <th>Amt_6</th>
      <th>Date_0</th>
      <th>Date_1</th>
      <th>Date_2</th>
      <th>Date_3</th>
      <th>Date_4</th>
      <th>Date_5</th>
      <th>Date_6</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Mon</td>
      <td>5249700</td>
      <td>9677500.00</td>
      <td>11402500.00</td>
      <td>5861800.00</td>
      <td>3976600.00</td>
      <td>3407800.00</td>
      <td>4063900.00</td>
      <td>37</td>
      <td>68.00</td>
      <td>80.00</td>
      <td>41.00</td>
      <td>26.00</td>
      <td>26.00</td>
      <td>29.00</td>
      <td>2019-12-02</td>
      <td>2019-12-01</td>
      <td>2019-11-30</td>
      <td>2019-11-29</td>
      <td>2019-11-28</td>
      <td>2019-11-27</td>
      <td>2019-11-26</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Tue</td>
      <td>5229700</td>
      <td>5249700.00</td>
      <td>9677500.00</td>
      <td>11402500.00</td>
      <td>5861800.00</td>
      <td>3976600.00</td>
      <td>3407800.00</td>
      <td>33</td>
      <td>37.00</td>
      <td>68.00</td>
      <td>80.00</td>
      <td>41.00</td>
      <td>26.00</td>
      <td>26.00</td>
      <td>2019-12-03</td>
      <td>2019-12-02</td>
      <td>2019-12-01</td>
      <td>2019-11-30</td>
      <td>2019-11-29</td>
      <td>2019-11-28</td>
      <td>2019-11-27</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Wed</td>
      <td>5059400</td>
      <td>5229700.00</td>
      <td>5249700.00</td>
      <td>9677500.00</td>
      <td>11402500.00</td>
      <td>5861800.00</td>
      <td>3976600.00</td>
      <td>33</td>
      <td>33.00</td>
      <td>37.00</td>
      <td>68.00</td>
      <td>80.00</td>
      <td>41.00</td>
      <td>26.00</td>
      <td>2019-12-04</td>
      <td>2019-12-03</td>
      <td>2019-12-02</td>
      <td>2019-12-01</td>
      <td>2019-11-30</td>
      <td>2019-11-29</td>
      <td>2019-11-28</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Thur</td>
      <td>4822800</td>
      <td>5059400.00</td>
      <td>5229700.00</td>
      <td>5249700.00</td>
      <td>9677500.00</td>
      <td>11402500.00</td>
      <td>5861800.00</td>
      <td>34</td>
      <td>33.00</td>
      <td>33.00</td>
      <td>37.00</td>
      <td>68.00</td>
      <td>80.00</td>
      <td>41.00</td>
      <td>2019-12-05</td>
      <td>2019-12-04</td>
      <td>2019-12-03</td>
      <td>2019-12-02</td>
      <td>2019-12-01</td>
      <td>2019-11-30</td>
      <td>2019-11-29</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Fri</td>
      <td>4567100</td>
      <td>4822800.00</td>
      <td>5059400.00</td>
      <td>5229700.00</td>
      <td>5249700.00</td>
      <td>9677500.00</td>
      <td>11402500.00</td>
      <td>30</td>
      <td>34.00</td>
      <td>33.00</td>
      <td>33.00</td>
      <td>37.00</td>
      <td>68.00</td>
      <td>80.00</td>
      <td>2019-12-06</td>
      <td>2019-12-05</td>
      <td>2019-12-04</td>
      <td>2019-12-03</td>
      <td>2019-12-02</td>
      <td>2019-12-01</td>
      <td>2019-11-30</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Sat</td>
      <td>9787000</td>
      <td>4567100.00</td>
      <td>4822800.00</td>
      <td>5059400.00</td>
      <td>5229700.00</td>
      <td>5249700.00</td>
      <td>9677500.00</td>
      <td>69</td>
      <td>30.00</td>
      <td>34.00</td>
      <td>33.00</td>
      <td>33.00</td>
      <td>37.00</td>
      <td>68.00</td>
      <td>2019-12-07</td>
      <td>2019-12-06</td>
      <td>2019-12-05</td>
      <td>2019-12-04</td>
      <td>2019-12-03</td>
      <td>2019-12-02</td>
      <td>2019-12-01</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Sun</td>
      <td>9707700</td>
      <td>9787000.00</td>
      <td>4567100.00</td>
      <td>4822800.00</td>
      <td>5059400.00</td>
      <td>5229700.00</td>
      <td>5249700.00</td>
      <td>67</td>
      <td>69.00</td>
      <td>30.00</td>
      <td>34.00</td>
      <td>33.00</td>
      <td>33.00</td>
      <td>37.00</td>
      <td>2019-12-08</td>
      <td>2019-12-07</td>
      <td>2019-12-06</td>
      <td>2019-12-05</td>
      <td>2019-12-04</td>
      <td>2019-12-03</td>
      <td>2019-12-02</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Mon</td>
      <td>3817100</td>
      <td>9707700.00</td>
      <td>9787000.00</td>
      <td>4567100.00</td>
      <td>4822800.00</td>
      <td>5059400.00</td>
      <td>5229700.00</td>
      <td>28</td>
      <td>67.00</td>
      <td>69.00</td>
      <td>30.00</td>
      <td>34.00</td>
      <td>33.00</td>
      <td>33.00</td>
      <td>2019-12-09</td>
      <td>2019-12-08</td>
      <td>2019-12-07</td>
      <td>2019-12-06</td>
      <td>2019-12-05</td>
      <td>2019-12-04</td>
      <td>2019-12-03</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Tue</td>
      <td>4343000</td>
      <td>3817100.00</td>
      <td>9707700.00</td>
      <td>9787000.00</td>
      <td>4567100.00</td>
      <td>4822800.00</td>
      <td>5059400.00</td>
      <td>31</td>
      <td>28.00</td>
      <td>67.00</td>
      <td>69.00</td>
      <td>30.00</td>
      <td>34.00</td>
      <td>33.00</td>
      <td>2019-12-10</td>
      <td>2019-12-09</td>
      <td>2019-12-08</td>
      <td>2019-12-07</td>
      <td>2019-12-06</td>
      <td>2019-12-05</td>
      <td>2019-12-04</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Wed</td>
      <td>5756100</td>
      <td>4343000.00</td>
      <td>3817100.00</td>
      <td>9707700.00</td>
      <td>9787000.00</td>
      <td>4567100.00</td>
      <td>4822800.00</td>
      <td>44</td>
      <td>31.00</td>
      <td>28.00</td>
      <td>67.00</td>
      <td>69.00</td>
      <td>30.00</td>
      <td>34.00</td>
      <td>2019-12-11</td>
      <td>2019-12-10</td>
      <td>2019-12-09</td>
      <td>2019-12-08</td>
      <td>2019-12-07</td>
      <td>2019-12-06</td>
      <td>2019-12-05</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Thur</td>
      <td>3426600</td>
      <td>5756100.00</td>
      <td>4343000.00</td>
      <td>3817100.00</td>
      <td>9707700.00</td>
      <td>9787000.00</td>
      <td>4567100.00</td>
      <td>25</td>
      <td>44.00</td>
      <td>31.00</td>
      <td>28.00</td>
      <td>67.00</td>
      <td>69.00</td>
      <td>30.00</td>
      <td>2019-12-12</td>
      <td>2019-12-11</td>
      <td>2019-12-10</td>
      <td>2019-12-09</td>
      <td>2019-12-08</td>
      <td>2019-12-07</td>
      <td>2019-12-06</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Fri</td>
      <td>7202500</td>
      <td>3426600.00</td>
      <td>5756100.00</td>
      <td>4343000.00</td>
      <td>3817100.00</td>
      <td>9707700.00</td>
      <td>9787000.00</td>
      <td>43</td>
      <td>25.00</td>
      <td>44.00</td>
      <td>31.00</td>
      <td>28.00</td>
      <td>67.00</td>
      <td>69.00</td>
      <td>2019-12-13</td>
      <td>2019-12-12</td>
      <td>2019-12-11</td>
      <td>2019-12-10</td>
      <td>2019-12-09</td>
      <td>2019-12-08</td>
      <td>2019-12-07</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Sat</td>
      <td>6983200</td>
      <td>7202500.00</td>
      <td>3426600.00</td>
      <td>5756100.00</td>
      <td>4343000.00</td>
      <td>3817100.00</td>
      <td>9707700.00</td>
      <td>53</td>
      <td>43.00</td>
      <td>25.00</td>
      <td>44.00</td>
      <td>31.00</td>
      <td>28.00</td>
      <td>67.00</td>
      <td>2019-12-14</td>
      <td>2019-12-13</td>
      <td>2019-12-12</td>
      <td>2019-12-11</td>
      <td>2019-12-10</td>
      <td>2019-12-09</td>
      <td>2019-12-08</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Sun</td>
      <td>12033600</td>
      <td>6983200.00</td>
      <td>7202500.00</td>
      <td>3426600.00</td>
      <td>5756100.00</td>
      <td>4343000.00</td>
      <td>3817100.00</td>
      <td>83</td>
      <td>53.00</td>
      <td>43.00</td>
      <td>25.00</td>
      <td>44.00</td>
      <td>31.00</td>
      <td>28.00</td>
      <td>2019-12-15</td>
      <td>2019-12-14</td>
      <td>2019-12-13</td>
      <td>2019-12-12</td>
      <td>2019-12-11</td>
      <td>2019-12-10</td>
      <td>2019-12-09</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Mon</td>
      <td>3422600</td>
      <td>12033600.00</td>
      <td>6983200.00</td>
      <td>7202500.00</td>
      <td>3426600.00</td>
      <td>5756100.00</td>
      <td>4343000.00</td>
      <td>24</td>
      <td>83.00</td>
      <td>53.00</td>
      <td>43.00</td>
      <td>25.00</td>
      <td>44.00</td>
      <td>31.00</td>
      <td>2019-12-16</td>
      <td>2019-12-15</td>
      <td>2019-12-14</td>
      <td>2019-12-13</td>
      <td>2019-12-12</td>
      <td>2019-12-11</td>
      <td>2019-12-10</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Tue</td>
      <td>4508400</td>
      <td>3422600.00</td>
      <td>12033600.00</td>
      <td>6983200.00</td>
      <td>7202500.00</td>
      <td>3426600.00</td>
      <td>5756100.00</td>
      <td>30</td>
      <td>24.00</td>
      <td>83.00</td>
      <td>53.00</td>
      <td>43.00</td>
      <td>25.00</td>
      <td>44.00</td>
      <td>2019-12-17</td>
      <td>2019-12-16</td>
      <td>2019-12-15</td>
      <td>2019-12-14</td>
      <td>2019-12-13</td>
      <td>2019-12-12</td>
      <td>2019-12-11</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Wed</td>
      <td>5582300</td>
      <td>4508400.00</td>
      <td>3422600.00</td>
      <td>12033600.00</td>
      <td>6983200.00</td>
      <td>7202500.00</td>
      <td>3426600.00</td>
      <td>41</td>
      <td>30.00</td>
      <td>24.00</td>
      <td>83.00</td>
      <td>53.00</td>
      <td>43.00</td>
      <td>25.00</td>
      <td>2019-12-18</td>
      <td>2019-12-17</td>
      <td>2019-12-16</td>
      <td>2019-12-15</td>
      <td>2019-12-14</td>
      <td>2019-12-13</td>
      <td>2019-12-12</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Thur</td>
      <td>2897300</td>
      <td>5582300.00</td>
      <td>4508400.00</td>
      <td>3422600.00</td>
      <td>12033600.00</td>
      <td>6983200.00</td>
      <td>7202500.00</td>
      <td>19</td>
      <td>41.00</td>
      <td>30.00</td>
      <td>24.00</td>
      <td>83.00</td>
      <td>53.00</td>
      <td>43.00</td>
      <td>2019-12-19</td>
      <td>2019-12-18</td>
      <td>2019-12-17</td>
      <td>2019-12-16</td>
      <td>2019-12-15</td>
      <td>2019-12-14</td>
      <td>2019-12-13</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Fri</td>
      <td>4787600</td>
      <td>2897300.00</td>
      <td>5582300.00</td>
      <td>4508400.00</td>
      <td>3422600.00</td>
      <td>12033600.00</td>
      <td>6983200.00</td>
      <td>33</td>
      <td>19.00</td>
      <td>41.00</td>
      <td>30.00</td>
      <td>24.00</td>
      <td>83.00</td>
      <td>53.00</td>
      <td>2019-12-20</td>
      <td>2019-12-19</td>
      <td>2019-12-18</td>
      <td>2019-12-17</td>
      <td>2019-12-16</td>
      <td>2019-12-15</td>
      <td>2019-12-14</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Sat</td>
      <td>8944800</td>
      <td>4787600.00</td>
      <td>2897300.00</td>
      <td>5582300.00</td>
      <td>4508400.00</td>
      <td>3422600.00</td>
      <td>12033600.00</td>
      <td>68</td>
      <td>33.00</td>
      <td>19.00</td>
      <td>41.00</td>
      <td>30.00</td>
      <td>24.00</td>
      <td>83.00</td>
      <td>2019-12-21</td>
      <td>2019-12-20</td>
      <td>2019-12-19</td>
      <td>2019-12-18</td>
      <td>2019-12-17</td>
      <td>2019-12-16</td>
      <td>2019-12-15</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Sun</td>
      <td>7805700</td>
      <td>8944800.00</td>
      <td>4787600.00</td>
      <td>2897300.00</td>
      <td>5582300.00</td>
      <td>4508400.00</td>
      <td>3422600.00</td>
      <td>57</td>
      <td>68.00</td>
      <td>33.00</td>
      <td>19.00</td>
      <td>41.00</td>
      <td>30.00</td>
      <td>24.00</td>
      <td>2019-12-22</td>
      <td>2019-12-21</td>
      <td>2019-12-20</td>
      <td>2019-12-19</td>
      <td>2019-12-18</td>
      <td>2019-12-17</td>
      <td>2019-12-16</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Mon</td>
      <td>5216100</td>
      <td>7805700.00</td>
      <td>8944800.00</td>
      <td>4787600.00</td>
      <td>2897300.00</td>
      <td>5582300.00</td>
      <td>4508400.00</td>
      <td>39</td>
      <td>57.00</td>
      <td>68.00</td>
      <td>33.00</td>
      <td>19.00</td>
      <td>41.00</td>
      <td>30.00</td>
      <td>2019-12-23</td>
      <td>2019-12-22</td>
      <td>2019-12-21</td>
      <td>2019-12-20</td>
      <td>2019-12-19</td>
      <td>2019-12-18</td>
      <td>2019-12-17</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Tue</td>
      <td>7646900</td>
      <td>5216100.00</td>
      <td>7805700.00</td>
      <td>8944800.00</td>
      <td>4787600.00</td>
      <td>2897300.00</td>
      <td>5582300.00</td>
      <td>52</td>
      <td>39.00</td>
      <td>57.00</td>
      <td>68.00</td>
      <td>33.00</td>
      <td>19.00</td>
      <td>41.00</td>
      <td>2019-12-24</td>
      <td>2019-12-23</td>
      <td>2019-12-22</td>
      <td>2019-12-21</td>
      <td>2019-12-20</td>
      <td>2019-12-19</td>
      <td>2019-12-18</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Wed</td>
      <td>11352400</td>
      <td>7646900.00</td>
      <td>5216100.00</td>
      <td>7805700.00</td>
      <td>8944800.00</td>
      <td>4787600.00</td>
      <td>2897300.00</td>
      <td>80</td>
      <td>52.00</td>
      <td>39.00</td>
      <td>57.00</td>
      <td>68.00</td>
      <td>33.00</td>
      <td>19.00</td>
      <td>2019-12-25</td>
      <td>2019-12-24</td>
      <td>2019-12-23</td>
      <td>2019-12-22</td>
      <td>2019-12-21</td>
      <td>2019-12-20</td>
      <td>2019-12-19</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Thur</td>
      <td>5622000</td>
      <td>11352400.00</td>
      <td>7646900.00</td>
      <td>5216100.00</td>
      <td>7805700.00</td>
      <td>8944800.00</td>
      <td>4787600.00</td>
      <td>40</td>
      <td>80.00</td>
      <td>52.00</td>
      <td>39.00</td>
      <td>57.00</td>
      <td>68.00</td>
      <td>33.00</td>
      <td>2019-12-26</td>
      <td>2019-12-25</td>
      <td>2019-12-24</td>
      <td>2019-12-23</td>
      <td>2019-12-22</td>
      <td>2019-12-21</td>
      <td>2019-12-20</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Fri</td>
      <td>3432600</td>
      <td>5622000.00</td>
      <td>11352400.00</td>
      <td>7646900.00</td>
      <td>5216100.00</td>
      <td>7805700.00</td>
      <td>8944800.00</td>
      <td>25</td>
      <td>40.00</td>
      <td>80.00</td>
      <td>52.00</td>
      <td>39.00</td>
      <td>57.00</td>
      <td>68.00</td>
      <td>2019-12-27</td>
      <td>2019-12-26</td>
      <td>2019-12-25</td>
      <td>2019-12-24</td>
      <td>2019-12-23</td>
      <td>2019-12-22</td>
      <td>2019-12-21</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Sat</td>
      <td>7174700</td>
      <td>3432600.00</td>
      <td>5622000.00</td>
      <td>11352400.00</td>
      <td>7646900.00</td>
      <td>5216100.00</td>
      <td>7805700.00</td>
      <td>52</td>
      <td>25.00</td>
      <td>40.00</td>
      <td>80.00</td>
      <td>52.00</td>
      <td>39.00</td>
      <td>57.00</td>
      <td>2019-12-28</td>
      <td>2019-12-27</td>
      <td>2019-12-26</td>
      <td>2019-12-25</td>
      <td>2019-12-24</td>
      <td>2019-12-23</td>
      <td>2019-12-22</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Sun</td>
      <td>10488900</td>
      <td>7174700.00</td>
      <td>3432600.00</td>
      <td>5622000.00</td>
      <td>11352400.00</td>
      <td>7646900.00</td>
      <td>5216100.00</td>
      <td>75</td>
      <td>52.00</td>
      <td>25.00</td>
      <td>40.00</td>
      <td>80.00</td>
      <td>52.00</td>
      <td>39.00</td>
      <td>2019-12-29</td>
      <td>2019-12-28</td>
      <td>2019-12-27</td>
      <td>2019-12-26</td>
      <td>2019-12-25</td>
      <td>2019-12-24</td>
      <td>2019-12-23</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Mon</td>
      <td>5313000</td>
      <td>10488900.00</td>
      <td>7174700.00</td>
      <td>3432600.00</td>
      <td>5622000.00</td>
      <td>11352400.00</td>
      <td>7646900.00</td>
      <td>38</td>
      <td>75.00</td>
      <td>52.00</td>
      <td>25.00</td>
      <td>40.00</td>
      <td>80.00</td>
      <td>52.00</td>
      <td>2019-12-30</td>
      <td>2019-12-29</td>
      <td>2019-12-28</td>
      <td>2019-12-27</td>
      <td>2019-12-26</td>
      <td>2019-12-25</td>
      <td>2019-12-24</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Tue</td>
      <td>6307600</td>
      <td>5313000.00</td>
      <td>10488900.00</td>
      <td>7174700.00</td>
      <td>3432600.00</td>
      <td>5622000.00</td>
      <td>11352400.00</td>
      <td>44</td>
      <td>38.00</td>
      <td>75.00</td>
      <td>52.00</td>
      <td>25.00</td>
      <td>40.00</td>
      <td>80.00</td>
      <td>2019-12-31</td>
      <td>2019-12-30</td>
      <td>2019-12-29</td>
      <td>2019-12-28</td>
      <td>2019-12-27</td>
      <td>2019-12-26</td>
      <td>2019-12-25</td>
    </tr>
  </tbody>
</table>
</div>




```
# df['Day'] = 'temp'

tlst = ['Mon', 'Tue', 'Wed', 'Thur', 'Fri', 'Sat', 'Sun']



for i in range(len(df)):

  tdf['Day'][i+5] = tlst[i%7]



for i in range(5):

  tdf['Day'][i] = tlst[i+2]



tdf.head()
```

    /usr/local/lib/python3.6/dist-packages/ipykernel_launcher.py:5: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      """
    /usr/local/lib/python3.6/dist-packages/ipykernel_launcher.py:8: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Day</th>
      <th>Net_0</th>
      <th>Net_1</th>
      <th>Net_2</th>
      <th>Net_3</th>
      <th>Net_4</th>
      <th>Net_5</th>
      <th>Net_6</th>
      <th>Amt_0</th>
      <th>Amt_1</th>
      <th>Amt_2</th>
      <th>Amt_3</th>
      <th>Amt_4</th>
      <th>Amt_5</th>
      <th>Amt_6</th>
      <th>Date_0</th>
      <th>Date_1</th>
      <th>Date_2</th>
      <th>Date_3</th>
      <th>Date_4</th>
      <th>Date_5</th>
      <th>Date_6</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Wed</td>
      <td>5249700</td>
      <td>9677500.00</td>
      <td>11402500.00</td>
      <td>5861800.00</td>
      <td>3976600.00</td>
      <td>3407800.00</td>
      <td>4063900.00</td>
      <td>37</td>
      <td>68.00</td>
      <td>80.00</td>
      <td>41.00</td>
      <td>26.00</td>
      <td>26.00</td>
      <td>29.00</td>
      <td>2019-12-02</td>
      <td>2019-12-01</td>
      <td>2019-11-30</td>
      <td>2019-11-29</td>
      <td>2019-11-28</td>
      <td>2019-11-27</td>
      <td>2019-11-26</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Thur</td>
      <td>5229700</td>
      <td>5249700.00</td>
      <td>9677500.00</td>
      <td>11402500.00</td>
      <td>5861800.00</td>
      <td>3976600.00</td>
      <td>3407800.00</td>
      <td>33</td>
      <td>37.00</td>
      <td>68.00</td>
      <td>80.00</td>
      <td>41.00</td>
      <td>26.00</td>
      <td>26.00</td>
      <td>2019-12-03</td>
      <td>2019-12-02</td>
      <td>2019-12-01</td>
      <td>2019-11-30</td>
      <td>2019-11-29</td>
      <td>2019-11-28</td>
      <td>2019-11-27</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Fri</td>
      <td>5059400</td>
      <td>5229700.00</td>
      <td>5249700.00</td>
      <td>9677500.00</td>
      <td>11402500.00</td>
      <td>5861800.00</td>
      <td>3976600.00</td>
      <td>33</td>
      <td>33.00</td>
      <td>37.00</td>
      <td>68.00</td>
      <td>80.00</td>
      <td>41.00</td>
      <td>26.00</td>
      <td>2019-12-04</td>
      <td>2019-12-03</td>
      <td>2019-12-02</td>
      <td>2019-12-01</td>
      <td>2019-11-30</td>
      <td>2019-11-29</td>
      <td>2019-11-28</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Sat</td>
      <td>4822800</td>
      <td>5059400.00</td>
      <td>5229700.00</td>
      <td>5249700.00</td>
      <td>9677500.00</td>
      <td>11402500.00</td>
      <td>5861800.00</td>
      <td>34</td>
      <td>33.00</td>
      <td>33.00</td>
      <td>37.00</td>
      <td>68.00</td>
      <td>80.00</td>
      <td>41.00</td>
      <td>2019-12-05</td>
      <td>2019-12-04</td>
      <td>2019-12-03</td>
      <td>2019-12-02</td>
      <td>2019-12-01</td>
      <td>2019-11-30</td>
      <td>2019-11-29</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Sun</td>
      <td>4567100</td>
      <td>4822800.00</td>
      <td>5059400.00</td>
      <td>5229700.00</td>
      <td>5249700.00</td>
      <td>9677500.00</td>
      <td>11402500.00</td>
      <td>30</td>
      <td>34.00</td>
      <td>33.00</td>
      <td>33.00</td>
      <td>37.00</td>
      <td>68.00</td>
      <td>80.00</td>
      <td>2019-12-06</td>
      <td>2019-12-05</td>
      <td>2019-12-04</td>
      <td>2019-12-03</td>
      <td>2019-12-02</td>
      <td>2019-12-01</td>
      <td>2019-11-30</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Mon</td>
      <td>9787000</td>
      <td>4567100.00</td>
      <td>4822800.00</td>
      <td>5059400.00</td>
      <td>5229700.00</td>
      <td>5249700.00</td>
      <td>9677500.00</td>
      <td>69</td>
      <td>30.00</td>
      <td>34.00</td>
      <td>33.00</td>
      <td>33.00</td>
      <td>37.00</td>
      <td>68.00</td>
      <td>2019-12-07</td>
      <td>2019-12-06</td>
      <td>2019-12-05</td>
      <td>2019-12-04</td>
      <td>2019-12-03</td>
      <td>2019-12-02</td>
      <td>2019-12-01</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Tue</td>
      <td>9707700</td>
      <td>9787000.00</td>
      <td>4567100.00</td>
      <td>4822800.00</td>
      <td>5059400.00</td>
      <td>5229700.00</td>
      <td>5249700.00</td>
      <td>67</td>
      <td>69.00</td>
      <td>30.00</td>
      <td>34.00</td>
      <td>33.00</td>
      <td>33.00</td>
      <td>37.00</td>
      <td>2019-12-08</td>
      <td>2019-12-07</td>
      <td>2019-12-06</td>
      <td>2019-12-05</td>
      <td>2019-12-04</td>
      <td>2019-12-03</td>
      <td>2019-12-02</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Wed</td>
      <td>3817100</td>
      <td>9707700.00</td>
      <td>9787000.00</td>
      <td>4567100.00</td>
      <td>4822800.00</td>
      <td>5059400.00</td>
      <td>5229700.00</td>
      <td>28</td>
      <td>67.00</td>
      <td>69.00</td>
      <td>30.00</td>
      <td>34.00</td>
      <td>33.00</td>
      <td>33.00</td>
      <td>2019-12-09</td>
      <td>2019-12-08</td>
      <td>2019-12-07</td>
      <td>2019-12-06</td>
      <td>2019-12-05</td>
      <td>2019-12-04</td>
      <td>2019-12-03</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Thur</td>
      <td>4343000</td>
      <td>3817100.00</td>
      <td>9707700.00</td>
      <td>9787000.00</td>
      <td>4567100.00</td>
      <td>4822800.00</td>
      <td>5059400.00</td>
      <td>31</td>
      <td>28.00</td>
      <td>67.00</td>
      <td>69.00</td>
      <td>30.00</td>
      <td>34.00</td>
      <td>33.00</td>
      <td>2019-12-10</td>
      <td>2019-12-09</td>
      <td>2019-12-08</td>
      <td>2019-12-07</td>
      <td>2019-12-06</td>
      <td>2019-12-05</td>
      <td>2019-12-04</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Fri</td>
      <td>5756100</td>
      <td>4343000.00</td>
      <td>3817100.00</td>
      <td>9707700.00</td>
      <td>9787000.00</td>
      <td>4567100.00</td>
      <td>4822800.00</td>
      <td>44</td>
      <td>31.00</td>
      <td>28.00</td>
      <td>67.00</td>
      <td>69.00</td>
      <td>30.00</td>
      <td>34.00</td>
      <td>2019-12-11</td>
      <td>2019-12-10</td>
      <td>2019-12-09</td>
      <td>2019-12-08</td>
      <td>2019-12-07</td>
      <td>2019-12-06</td>
      <td>2019-12-05</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Sat</td>
      <td>3426600</td>
      <td>5756100.00</td>
      <td>4343000.00</td>
      <td>3817100.00</td>
      <td>9707700.00</td>
      <td>9787000.00</td>
      <td>4567100.00</td>
      <td>25</td>
      <td>44.00</td>
      <td>31.00</td>
      <td>28.00</td>
      <td>67.00</td>
      <td>69.00</td>
      <td>30.00</td>
      <td>2019-12-12</td>
      <td>2019-12-11</td>
      <td>2019-12-10</td>
      <td>2019-12-09</td>
      <td>2019-12-08</td>
      <td>2019-12-07</td>
      <td>2019-12-06</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Sun</td>
      <td>7202500</td>
      <td>3426600.00</td>
      <td>5756100.00</td>
      <td>4343000.00</td>
      <td>3817100.00</td>
      <td>9707700.00</td>
      <td>9787000.00</td>
      <td>43</td>
      <td>25.00</td>
      <td>44.00</td>
      <td>31.00</td>
      <td>28.00</td>
      <td>67.00</td>
      <td>69.00</td>
      <td>2019-12-13</td>
      <td>2019-12-12</td>
      <td>2019-12-11</td>
      <td>2019-12-10</td>
      <td>2019-12-09</td>
      <td>2019-12-08</td>
      <td>2019-12-07</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Mon</td>
      <td>6983200</td>
      <td>7202500.00</td>
      <td>3426600.00</td>
      <td>5756100.00</td>
      <td>4343000.00</td>
      <td>3817100.00</td>
      <td>9707700.00</td>
      <td>53</td>
      <td>43.00</td>
      <td>25.00</td>
      <td>44.00</td>
      <td>31.00</td>
      <td>28.00</td>
      <td>67.00</td>
      <td>2019-12-14</td>
      <td>2019-12-13</td>
      <td>2019-12-12</td>
      <td>2019-12-11</td>
      <td>2019-12-10</td>
      <td>2019-12-09</td>
      <td>2019-12-08</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Tue</td>
      <td>12033600</td>
      <td>6983200.00</td>
      <td>7202500.00</td>
      <td>3426600.00</td>
      <td>5756100.00</td>
      <td>4343000.00</td>
      <td>3817100.00</td>
      <td>83</td>
      <td>53.00</td>
      <td>43.00</td>
      <td>25.00</td>
      <td>44.00</td>
      <td>31.00</td>
      <td>28.00</td>
      <td>2019-12-15</td>
      <td>2019-12-14</td>
      <td>2019-12-13</td>
      <td>2019-12-12</td>
      <td>2019-12-11</td>
      <td>2019-12-10</td>
      <td>2019-12-09</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Wed</td>
      <td>3422600</td>
      <td>12033600.00</td>
      <td>6983200.00</td>
      <td>7202500.00</td>
      <td>3426600.00</td>
      <td>5756100.00</td>
      <td>4343000.00</td>
      <td>24</td>
      <td>83.00</td>
      <td>53.00</td>
      <td>43.00</td>
      <td>25.00</td>
      <td>44.00</td>
      <td>31.00</td>
      <td>2019-12-16</td>
      <td>2019-12-15</td>
      <td>2019-12-14</td>
      <td>2019-12-13</td>
      <td>2019-12-12</td>
      <td>2019-12-11</td>
      <td>2019-12-10</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Thur</td>
      <td>4508400</td>
      <td>3422600.00</td>
      <td>12033600.00</td>
      <td>6983200.00</td>
      <td>7202500.00</td>
      <td>3426600.00</td>
      <td>5756100.00</td>
      <td>30</td>
      <td>24.00</td>
      <td>83.00</td>
      <td>53.00</td>
      <td>43.00</td>
      <td>25.00</td>
      <td>44.00</td>
      <td>2019-12-17</td>
      <td>2019-12-16</td>
      <td>2019-12-15</td>
      <td>2019-12-14</td>
      <td>2019-12-13</td>
      <td>2019-12-12</td>
      <td>2019-12-11</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Fri</td>
      <td>5582300</td>
      <td>4508400.00</td>
      <td>3422600.00</td>
      <td>12033600.00</td>
      <td>6983200.00</td>
      <td>7202500.00</td>
      <td>3426600.00</td>
      <td>41</td>
      <td>30.00</td>
      <td>24.00</td>
      <td>83.00</td>
      <td>53.00</td>
      <td>43.00</td>
      <td>25.00</td>
      <td>2019-12-18</td>
      <td>2019-12-17</td>
      <td>2019-12-16</td>
      <td>2019-12-15</td>
      <td>2019-12-14</td>
      <td>2019-12-13</td>
      <td>2019-12-12</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Sat</td>
      <td>2897300</td>
      <td>5582300.00</td>
      <td>4508400.00</td>
      <td>3422600.00</td>
      <td>12033600.00</td>
      <td>6983200.00</td>
      <td>7202500.00</td>
      <td>19</td>
      <td>41.00</td>
      <td>30.00</td>
      <td>24.00</td>
      <td>83.00</td>
      <td>53.00</td>
      <td>43.00</td>
      <td>2019-12-19</td>
      <td>2019-12-18</td>
      <td>2019-12-17</td>
      <td>2019-12-16</td>
      <td>2019-12-15</td>
      <td>2019-12-14</td>
      <td>2019-12-13</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Sun</td>
      <td>4787600</td>
      <td>2897300.00</td>
      <td>5582300.00</td>
      <td>4508400.00</td>
      <td>3422600.00</td>
      <td>12033600.00</td>
      <td>6983200.00</td>
      <td>33</td>
      <td>19.00</td>
      <td>41.00</td>
      <td>30.00</td>
      <td>24.00</td>
      <td>83.00</td>
      <td>53.00</td>
      <td>2019-12-20</td>
      <td>2019-12-19</td>
      <td>2019-12-18</td>
      <td>2019-12-17</td>
      <td>2019-12-16</td>
      <td>2019-12-15</td>
      <td>2019-12-14</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Mon</td>
      <td>8944800</td>
      <td>4787600.00</td>
      <td>2897300.00</td>
      <td>5582300.00</td>
      <td>4508400.00</td>
      <td>3422600.00</td>
      <td>12033600.00</td>
      <td>68</td>
      <td>33.00</td>
      <td>19.00</td>
      <td>41.00</td>
      <td>30.00</td>
      <td>24.00</td>
      <td>83.00</td>
      <td>2019-12-21</td>
      <td>2019-12-20</td>
      <td>2019-12-19</td>
      <td>2019-12-18</td>
      <td>2019-12-17</td>
      <td>2019-12-16</td>
      <td>2019-12-15</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Tue</td>
      <td>7805700</td>
      <td>8944800.00</td>
      <td>4787600.00</td>
      <td>2897300.00</td>
      <td>5582300.00</td>
      <td>4508400.00</td>
      <td>3422600.00</td>
      <td>57</td>
      <td>68.00</td>
      <td>33.00</td>
      <td>19.00</td>
      <td>41.00</td>
      <td>30.00</td>
      <td>24.00</td>
      <td>2019-12-22</td>
      <td>2019-12-21</td>
      <td>2019-12-20</td>
      <td>2019-12-19</td>
      <td>2019-12-18</td>
      <td>2019-12-17</td>
      <td>2019-12-16</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Wed</td>
      <td>5216100</td>
      <td>7805700.00</td>
      <td>8944800.00</td>
      <td>4787600.00</td>
      <td>2897300.00</td>
      <td>5582300.00</td>
      <td>4508400.00</td>
      <td>39</td>
      <td>57.00</td>
      <td>68.00</td>
      <td>33.00</td>
      <td>19.00</td>
      <td>41.00</td>
      <td>30.00</td>
      <td>2019-12-23</td>
      <td>2019-12-22</td>
      <td>2019-12-21</td>
      <td>2019-12-20</td>
      <td>2019-12-19</td>
      <td>2019-12-18</td>
      <td>2019-12-17</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Thur</td>
      <td>7646900</td>
      <td>5216100.00</td>
      <td>7805700.00</td>
      <td>8944800.00</td>
      <td>4787600.00</td>
      <td>2897300.00</td>
      <td>5582300.00</td>
      <td>52</td>
      <td>39.00</td>
      <td>57.00</td>
      <td>68.00</td>
      <td>33.00</td>
      <td>19.00</td>
      <td>41.00</td>
      <td>2019-12-24</td>
      <td>2019-12-23</td>
      <td>2019-12-22</td>
      <td>2019-12-21</td>
      <td>2019-12-20</td>
      <td>2019-12-19</td>
      <td>2019-12-18</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Fri</td>
      <td>11352400</td>
      <td>7646900.00</td>
      <td>5216100.00</td>
      <td>7805700.00</td>
      <td>8944800.00</td>
      <td>4787600.00</td>
      <td>2897300.00</td>
      <td>80</td>
      <td>52.00</td>
      <td>39.00</td>
      <td>57.00</td>
      <td>68.00</td>
      <td>33.00</td>
      <td>19.00</td>
      <td>2019-12-25</td>
      <td>2019-12-24</td>
      <td>2019-12-23</td>
      <td>2019-12-22</td>
      <td>2019-12-21</td>
      <td>2019-12-20</td>
      <td>2019-12-19</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Sat</td>
      <td>5622000</td>
      <td>11352400.00</td>
      <td>7646900.00</td>
      <td>5216100.00</td>
      <td>7805700.00</td>
      <td>8944800.00</td>
      <td>4787600.00</td>
      <td>40</td>
      <td>80.00</td>
      <td>52.00</td>
      <td>39.00</td>
      <td>57.00</td>
      <td>68.00</td>
      <td>33.00</td>
      <td>2019-12-26</td>
      <td>2019-12-25</td>
      <td>2019-12-24</td>
      <td>2019-12-23</td>
      <td>2019-12-22</td>
      <td>2019-12-21</td>
      <td>2019-12-20</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Sun</td>
      <td>3432600</td>
      <td>5622000.00</td>
      <td>11352400.00</td>
      <td>7646900.00</td>
      <td>5216100.00</td>
      <td>7805700.00</td>
      <td>8944800.00</td>
      <td>25</td>
      <td>40.00</td>
      <td>80.00</td>
      <td>52.00</td>
      <td>39.00</td>
      <td>57.00</td>
      <td>68.00</td>
      <td>2019-12-27</td>
      <td>2019-12-26</td>
      <td>2019-12-25</td>
      <td>2019-12-24</td>
      <td>2019-12-23</td>
      <td>2019-12-22</td>
      <td>2019-12-21</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Mon</td>
      <td>7174700</td>
      <td>3432600.00</td>
      <td>5622000.00</td>
      <td>11352400.00</td>
      <td>7646900.00</td>
      <td>5216100.00</td>
      <td>7805700.00</td>
      <td>52</td>
      <td>25.00</td>
      <td>40.00</td>
      <td>80.00</td>
      <td>52.00</td>
      <td>39.00</td>
      <td>57.00</td>
      <td>2019-12-28</td>
      <td>2019-12-27</td>
      <td>2019-12-26</td>
      <td>2019-12-25</td>
      <td>2019-12-24</td>
      <td>2019-12-23</td>
      <td>2019-12-22</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Tue</td>
      <td>10488900</td>
      <td>7174700.00</td>
      <td>3432600.00</td>
      <td>5622000.00</td>
      <td>11352400.00</td>
      <td>7646900.00</td>
      <td>5216100.00</td>
      <td>75</td>
      <td>52.00</td>
      <td>25.00</td>
      <td>40.00</td>
      <td>80.00</td>
      <td>52.00</td>
      <td>39.00</td>
      <td>2019-12-29</td>
      <td>2019-12-28</td>
      <td>2019-12-27</td>
      <td>2019-12-26</td>
      <td>2019-12-25</td>
      <td>2019-12-24</td>
      <td>2019-12-23</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Wed</td>
      <td>5313000</td>
      <td>10488900.00</td>
      <td>7174700.00</td>
      <td>3432600.00</td>
      <td>5622000.00</td>
      <td>11352400.00</td>
      <td>7646900.00</td>
      <td>38</td>
      <td>75.00</td>
      <td>52.00</td>
      <td>25.00</td>
      <td>40.00</td>
      <td>80.00</td>
      <td>52.00</td>
      <td>2019-12-30</td>
      <td>2019-12-29</td>
      <td>2019-12-28</td>
      <td>2019-12-27</td>
      <td>2019-12-26</td>
      <td>2019-12-25</td>
      <td>2019-12-24</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Thur</td>
      <td>6307600</td>
      <td>5313000.00</td>
      <td>10488900.00</td>
      <td>7174700.00</td>
      <td>3432600.00</td>
      <td>5622000.00</td>
      <td>11352400.00</td>
      <td>44</td>
      <td>38.00</td>
      <td>75.00</td>
      <td>52.00</td>
      <td>25.00</td>
      <td>40.00</td>
      <td>80.00</td>
      <td>2019-12-31</td>
      <td>2019-12-30</td>
      <td>2019-12-29</td>
      <td>2019-12-28</td>
      <td>2019-12-27</td>
      <td>2019-12-26</td>
      <td>2019-12-25</td>
    </tr>
  </tbody>
</table>
</div>




```
fin_day = pd.get_dummies(tdf['Day'])[tlst]

fin_day.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Mon</th>
      <th>Tue</th>
      <th>Wed</th>
      <th>Thur</th>
      <th>Fri</th>
      <th>Sat</th>
      <th>Sun</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>12</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>14</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>15</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>16</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>17</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>18</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>19</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>20</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>21</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>22</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>23</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>24</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>25</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>26</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>27</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>28</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>29</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```
ttdf = tdf[netlst2+amtlst2]

fin_x = x_std_scaler.transform(x_mn_scaler.transform(ttdf))

fin_x = pd.concat([pd.DataFrame(fin_x), fin_day], axis=1)

fin_x = np.array(fin_x).reshape(len(fin_x), 3, 7)

fin_x[0]
```




    array([[-0.53854926,  0.43353472,  0.81400201, -0.40425562, -0.82019656,
            -0.94611458, -0.80125271],
           [-0.60417456,  0.2931319 ,  0.64194696, -0.48872107, -0.9251693 ,
            -0.92614031, -0.83858617],
           [ 0.        ,  0.        ,  1.        ,  0.        ,  0.        ,
             0.        ,  0.        ]])




```
fin_pred_net = net_model.predict(fin_x)

fin_pred_amt = amt_model.predict(fin_x)
```


```
result_net = pd.DataFrame(np.ceil(net_mn_scaler.inverse_transform(net_std_scaler.inverse_transform(fin_pred_net.reshape(30,1)))))

result_amt = pd.DataFrame(np.ceil(amt_mn_scaler.inverse_transform(amt_std_scaler.inverse_transform(fin_pred_amt.reshape(30,1)))))

result = pd.concat([result_net, result_amt], axis=1)

result.columns = ['Net', 'Amt']

result.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Net</th>
      <th>Amt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>25</th>
      <td>9355740.00</td>
      <td>101.00</td>
    </tr>
    <tr>
      <th>26</th>
      <td>4058488.00</td>
      <td>33.00</td>
    </tr>
    <tr>
      <th>27</th>
      <td>4514564.00</td>
      <td>31.00</td>
    </tr>
    <tr>
      <th>28</th>
      <td>7582181.00</td>
      <td>57.00</td>
    </tr>
    <tr>
      <th>29</th>
      <td>6289746.00</td>
      <td>42.00</td>
    </tr>
  </tbody>
</table>
</div>




```
tnet = [8483159, 6307600,	5313000.00,	10488900.00,	7174700.00,	3432600.00,	5622000.00]

tamt = [63, 44,	38.00,	75.00,	52.00,	25.00,	40.00]

last = pd.DataFrame(x_std_scaler.transform(x_mn_scaler.transform(pd.DataFrame(tnet+tamt).T)))
```


```
tla = [0,0,0,0,1,0,0]

tla = pd.DataFrame(tla).T

last = pd.concat([last, tla], axis=1)



lastx = np.array(last).reshape(1,3,7)

lastx
```




    array([[[ 0.17188913, -0.30657576, -0.52371601,  0.61206708,
             -0.11791844, -0.94066644, -0.45910179],
            [ 0.14906546, -0.40188827, -0.57445417,  0.49592925,
             -0.17225135, -0.95511012, -0.52002995],
            [ 0.        ,  0.        ,  0.        ,  0.        ,
              1.        ,  0.        ,  0.        ]]])




```
last_pred_net = net_model.predict(lastx)

last_pred_amt = amt_model.predict(lastx)



last_net = np.ceil(net_mn_scaler.inverse_transform(net_std_scaler.inverse_transform(last_pred_net)))

last_amt = np.ceil(amt_mn_scaler.inverse_transform(amt_std_scaler.inverse_transform(last_pred_amt)))



tp = pd.DataFrame([int(last_net), int(last_amt)]).T

tp.columns = ['Net', 'Amt']

result = pd.concat([result, tp])

result.index = [i+1 for i in range(len(result))]

result
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Net</th>
      <th>Amt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>7345794.00</td>
      <td>98.00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5947701.00</td>
      <td>61.00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4542513.00</td>
      <td>33.00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>13436799.00</td>
      <td>96.00</td>
    </tr>
    <tr>
      <th>5</th>
      <td>10954760.00</td>
      <td>81.00</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2719977.00</td>
      <td>41.00</td>
    </tr>
    <tr>
      <th>7</th>
      <td>3974952.00</td>
      <td>31.00</td>
    </tr>
    <tr>
      <th>8</th>
      <td>4213667.00</td>
      <td>66.00</td>
    </tr>
    <tr>
      <th>9</th>
      <td>4985098.00</td>
      <td>51.00</td>
    </tr>
    <tr>
      <th>10</th>
      <td>4231632.00</td>
      <td>35.00</td>
    </tr>
    <tr>
      <th>11</th>
      <td>12230495.00</td>
      <td>89.00</td>
    </tr>
    <tr>
      <th>12</th>
      <td>9437989.00</td>
      <td>106.00</td>
    </tr>
    <tr>
      <th>13</th>
      <td>3906009.00</td>
      <td>30.00</td>
    </tr>
    <tr>
      <th>14</th>
      <td>3459846.00</td>
      <td>44.00</td>
    </tr>
    <tr>
      <th>15</th>
      <td>5284427.00</td>
      <td>42.00</td>
    </tr>
    <tr>
      <th>16</th>
      <td>5694525.00</td>
      <td>57.00</td>
    </tr>
    <tr>
      <th>17</th>
      <td>5012422.00</td>
      <td>35.00</td>
    </tr>
    <tr>
      <th>18</th>
      <td>7265251.00</td>
      <td>90.00</td>
    </tr>
    <tr>
      <th>19</th>
      <td>7307517.00</td>
      <td>66.00</td>
    </tr>
    <tr>
      <th>20</th>
      <td>3654269.00</td>
      <td>37.00</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2962862.00</td>
      <td>18.00</td>
    </tr>
    <tr>
      <th>22</th>
      <td>3888259.00</td>
      <td>80.00</td>
    </tr>
    <tr>
      <th>23</th>
      <td>6207906.00</td>
      <td>65.00</td>
    </tr>
    <tr>
      <th>24</th>
      <td>6041176.00</td>
      <td>48.00</td>
    </tr>
    <tr>
      <th>25</th>
      <td>12099037.00</td>
      <td>90.00</td>
    </tr>
    <tr>
      <th>26</th>
      <td>9355740.00</td>
      <td>101.00</td>
    </tr>
    <tr>
      <th>27</th>
      <td>4058488.00</td>
      <td>33.00</td>
    </tr>
    <tr>
      <th>28</th>
      <td>4514564.00</td>
      <td>31.00</td>
    </tr>
    <tr>
      <th>29</th>
      <td>7582181.00</td>
      <td>57.00</td>
    </tr>
    <tr>
      <th>30</th>
      <td>6289746.00</td>
      <td>42.00</td>
    </tr>
    <tr>
      <th>31</th>
      <td>6229728.00</td>
      <td>59.00</td>
    </tr>
  </tbody>
</table>
</div>




```

```
