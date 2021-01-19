---
layout: post
title:  "LSTM을 이용한 주가예측 모델"
subtitle:   LSTM을 이용한 주가예측 모델
categories: career
tags: dataanalytics project jupyter-notebook stock LSTM
comments: true
---


```python
import pandas_datareader as pdr
from sklearn.preprocessing import MinMaxScaler
import pandas as pd
import numpy as np 
from matplotlib import pyplot as plt
```


```python
import tensorflow as tf
from tensorflow.keras.layers import LSTM
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense
```

# Function


```python
def predict_stock(stock, column, period, timestep):
  stock = stock
  column = column
  period = period
  timestep = timestep
  
  select_stock(stock, period, timestep)
  prepro(df, column, timestep)
  result = pred_stock(period, timestep)
  return result
```


```python
# 필요한 변수 = ['High', 'Low', 'Close']
# 종목 선택 & 앞으로 예측할 기간 선택

def select_stock(stock, period, timestep):
    df = pdr.get_data_yahoo(stock, '2000-01-01')
    tempdf = df.copy()[['High', 'Low', 'Close']]
    #타겟 설정
    tempdf['Target_YMD'] = 0
    tempdf['Target_High'] = 0
    tempdf['Target_Low'] = 0
    tempdf['Target_Close'] = 0
    for i in range(len(tempdf)):
        try:
            tempdf['Target_YMD'].iloc[i] = str(tempdf.index[i+period]).split(' ')[0]
            tempdf['Target_High'].iloc[i] = tempdf['High'].iloc[i+period]
            tempdf['Target_Low'].iloc[i] = tempdf['Low'].iloc[i+period]
            tempdf['Target_Close'].iloc[i] = tempdf['Close'].iloc[i+period]
        except:
            continue

    tempdf_index = []
    for i in range(len(tempdf)):
        indexword =  str(tempdf.index[i]).split(' ')[0]
        tempdf_index.append(indexword)

    tempdf.index = tempdf_index
    global future_test
    future_test = tempdf[-period-(timestep-1):][['High', 'Low', 'Close']] # 3은 shift 횟수
    tempdf = tempdf[:-period]
    
    return tempdf
```


```python
# df = select_stock(stock_code, period) 
def prepro(df, column, timestep):
    x_data = df[['High', 'Low', 'Close']]
    #x_data log&MinMaxScale
    x_data = np.log(x_data)
    global sc_x
    sc_x = MinMaxScaler()
    x_data = pd.DataFrame(sc_x.fit_transform(x_data), index=x_data.index, columns=x_data.columns)
    #Time step 결정
    lst = []
    lst.append('High')
    for step in range(1,timestep):
        x_data['High_{}'.format(step)] = x_data['High'].shift(step)
        lst.append('High_'+str(step))
    lst.append('Low')
    for step in range(1,timestep):
        x_data['Low_{}'.format(step)] = x_data['Low'].shift(step)
        lst.append('Low_'+str(step))
    lst.append('Close')
    for step in range(1,timestep):
        x_data['Close_{}'.format(step)] = x_data['Close'].shift(step)
        lst.append('Close_'+str(step))
    x_data = x_data.dropna()
    x_data = x_data[lst]
    
    #y_data
    #y columns 선택변수로 만들 것 
    target_word = 'Target_'+str(column)
    y_data = df[target_word][(timestep-1):] # Timestep의 갯수만큼 
    y_ymd = df['Target_YMD'][(timestep-1):]
    y_data = np.log(y_data)
    global sc_y 
    sc_y = MinMaxScaler()
    y_data = sc_y.fit_transform(np.array(y_data).reshape(-1,1))
    #train/test set split  _ test_size = 1000
    global x_test_t
    global y_test
    y_train = y_data[:-1000]
    y_test = y_data[-1000:]
    x_train = x_data[:-1000]
    x_test = x_data[-1000:]

    #x_data reshape(len, feature, timestep)
    x_train_t = np.array(x_train).reshape(len(x_train), 3, timestep)
    x_test_t = np.array(x_test).reshape(len(x_test), 3, timestep)
    
    #Machine Learning Model
    global model
    model = tf.keras.models.Sequential()
    model.add(LSTM(10, input_shape=(x_train_t.shape[1], x_train_t.shape[2])))
    model.add(Dense(1))
    model.compile(loss='mean_squared_error', optimizer = 'adam')
    model.fit(x_train_t, y_train, epochs=10, batch_size =1, verbose =1)
    
    pred = model.predict(x_test_t)
    pred = np.exp(sc_y.inverse_transform(pred))
    predict = pd.DataFrame(pred)
    predict.columns = ['Predict_{}'.format(column)]
    predict['Real_{}'.format(column)] = np.exp(sc_y.inverse_transform(y_test))
    predict.index = y_ymd[-1000:]
    predict['Error_{}'.format(column)] = predict['Real_{}'.format(column)] - predict['Predict_{}'.format(column)]
    global adj
    lennum = round((len(predict))* (2/3))
    adj = predict['Error_{}'.format(column)][-int(lennum):].mean()
    print('Mean Error:', adj)
    return predict
    
```


```python
def pred_stock(period,timestep):
  model.fit(x_test_t, y_test, epochs=10, batch_size =1, verbose =1)
  x_data = np.log(future_test)
  x_data = pd.DataFrame(sc_x.transform(x_data), index= x_data.index, columns=x_data.columns)
  #Time step 결정
  lst = []
  lst.append('High')
  for step in range(1,timestep):
      x_data['High_{}'.format(step)] = x_data['High'].shift(step)
      lst.append('High_'+str(step))
  lst.append('Low')
  for step in range(1,timestep):
      x_data['Low_{}'.format(step)] = x_data['Low'].shift(step)
      lst.append('Low_'+str(step))
  lst.append('Close')
  for step in range(1,timestep):
      x_data['Close_{}'.format(step)] = x_data['Close'].shift(step)
      lst.append('Close_'+str(step))
  x_data = x_data.dropna()
  x_data = x_data[lst] 
  #x_data reshape(len, feature, timestep) 
  x_data_t = np.array(x_data).reshape(len(x_data), 3, timestep) 

  #Prediction
  pred = model.predict(x_data_t)
  pred = np.exp(sc_y.inverse_transform(pred))
  
  tmplst=[]
  for i in range(1,period+1):
    tmpwd = future_test.index[-1] + " +"+str(i)+"일"
    tmplst.append(tmpwd)

  predict = pd.DataFrame(pred, columns = ['Col'])
  predict['Adj_Col'] = predict['Col']+ adj
  predict['Date'] = tmplst

  return predict
```


```python
# df = select_stock(stock_code, period)  
def prepro2(df, column, timestep):   # x_data.shape(len(x_train), timestep, 3)
    x_data = df[['High', 'Low', 'Close']]
    #x_data log&MinMaxScale
    x_data = np.log(x_data)
    global sc_x
    sc_x = MinMaxScaler()
    x_data = pd.DataFrame(sc_x.fit_transform(x_data), index=x_data.index, columns=x_data.columns)
    #Time step 결정
    for step in range(1,timestep):
        x_data['High_{}'.format(step)] = x_data['High'].shift(step)
    for step in range(1,timestep):
        x_data['Low_{}'.format(step)] = x_data['Low'].shift(step)
    for step in range(1,timestep):
        x_data['Close_{}'.format(step)] = x_data['Close'].shift(step)
    x_data = x_data.dropna()
        
    #y_data
    #y columns 선택변수로 만들 것 
    target_word = 'Target_'+str(column)
    y_data = df[target_word][(timestep-1):] # Timestep의 갯수만큼 
    y_ymd = df['Target_YMD'][(timestep-1):]
    y_data = np.log(y_data)
    global sc_y 
    sc_y = MinMaxScaler()
    y_data = sc_y.fit_transform(np.array(y_data).reshape(-1,1))
    #train/test set split  _ test_size = 1000
    global x_test_t
    global y_test
    y_train = y_data[:-1000]
    y_test = y_data[-1000:]
    x_train = x_data[:-1000]
    x_test = x_data[-1000:]

    #x_data reshape(len, feature, timestep)
    x_train_t = np.array(x_train).reshape(len(x_train), timestep, 3)
    x_test_t = np.array(x_test).reshape(len(x_test), timestep, 3)
    
    #Machine Learning Model
    global model
    model = tf.keras.models.Sequential()
    model.add(LSTM(10, input_shape=(x_train_t.shape[1], x_train_t.shape[2])))
    model.add(Dense(1))
    model.compile(loss='mean_squared_error', optimizer = 'adam')
    model.fit(x_train_t, y_train, epochs=50, batch_size =1, verbose =1)
    
    pred = model.predict(x_test_t)
    pred = np.exp(sc_y.inverse_transform(pred))
    predict = pd.DataFrame(pred)
    predict.columns = ['Predict_{}'.format(column)]
    predict['Real_{}'.format(column)] = np.exp(sc_y.inverse_transform(y_test))
    predict.index = y_ymd[-1000:]
    predict['Error_{}'.format(column)] = predict['Real_{}'.format(column)] - predict['Predict_{}'.format(column)]
    global adj
    lennum = round((len(predict))* (2/3))
    adj = predict['Error_{}'.format(column)][-int(lennum):].mean()
    print('Mean Error:', adj)
    return predict
    

```


```python
def pred_stock2(period,timestep):
  model.fit(x_test_t, y_test, epochs=50, batch_size =1, verbose =1) 
  x_data = np.log(future_test)
  x_data = pd.DataFrame(sc_x.transform(x_data), index= x_data.index, columns=x_data.columns)
  #Time step 결정
  for step in range(1,timestep):
      x_data['High_{}'.format(step)] = x_data['High'].shift(step)
  for step in range(1,timestep):
      x_data['Low_{}'.format(step)] = x_data['Low'].shift(step)
  for step in range(1,timestep):
      x_data['Close_{}'.format(step)] = x_data['Close'].shift(step)
  x_data = x_data.dropna()
  #x_data reshape(len, feature, timestep) 
  x_data_t = np.array(x_data).reshape(len(x_data), timestep, 3) 

  #Prediction
  pred = model.predict(x_data_t)
  pred = np.exp(sc_y.inverse_transform(pred))
  
  tmplst=[]
  for i in range(1,period+1):
    tmpwd = future_test.index[-1] + " +"+str(i)+"일"
    tmplst.append(tmpwd)

  predict = pd.DataFrame(pred, columns = ['Col'])
  predict['Adj_Col'] = predict['Col']+ adj
  predict['Date'] = tmplst

  return predict
```


```python
# 테스트 셋 주기적으로 업데이트하는 함수
# 트레이닝 미리 해놓을 것

def setup(timestep, column):
    global x_data
    global y_data
    x_data = df[['High', 'Low', 'Close']]
    #x_data log&MinMaxScale
    x_data = np.log(x_data)
    global sc_x
    sc_x = MinMaxScaler()
    x_data = pd.DataFrame(sc_x.fit_transform(x_data), index=x_data.index, columns=x_data.columns)
    #Time step 결정
    lst = []
    lst.append('High')
    for step in range(1,timestep):
        x_data['High_{}'.format(step)] = x_data['High'].shift(step)
        lst.append('High_'+str(step))
    lst.append('Low')
    for step in range(1,timestep):
        x_data['Low_{}'.format(step)] = x_data['Low'].shift(step)
        lst.append('Low_'+str(step))
    lst.append('Close')
    for step in range(1,timestep):
        x_data['Close_{}'.format(step)] = x_data['Close'].shift(step)
        lst.append('Close_'+str(step))
    x_data = x_data.dropna()
    x_data = x_data[lst]
    
    #y_data
    #y columns 선택변수로 만들 것 
    target_word = 'Target_'+str(column)
    y_data = df[target_word][(timestep-1):] # Timestep의 갯수만큼 
    y_ymd = df['Target_YMD'][(timestep-1):]
    y_data = np.log(y_data)
    global sc_y 
    sc_y = MinMaxScaler()
    y_data = sc_y.fit_transform(np.array(y_data).reshape(-1,1))
    #train/test set split  _ test_size = 1000

    y_train = y_data[:-1000]
    x_train = x_data[:-1000]

    #x_data reshape(len, feature, timestep)
    x_train_t = np.array(x_train).reshape(len(x_train), 3, timestep)
    
    #Machine Learning Model
    global model
    model = tf.keras.models.Sequential()
    model.add(LSTM(10, input_shape=(x_train_t.shape[1], x_train_t.shape[2])))
    model.add(Dense(1))
    model.compile(loss='mean_squared_error', optimizer = 'adam')
    model.fit(x_train_t, y_train, epochs=10, batch_size =1, verbose =1)



def update(repeat):
  repeat = repeat
  first = 1000 -repeat
  second = first - repeat
  y_ymd = df['Target_YMD'][(timestep-1):]
  y_test = y_data[-1000:-first]
  x_test = x_data[-1000:-first]
  x_test_t = np.array(x_test).reshape(len(x_test), 3, timestep)
  pred = model.predict(x_test_t)
  pred = np.exp(sc_y.inverse_transform(pred))
  pred = pd.DataFrame(pred, index= y_ymd[-1000:-first], columns=['Prediction'])
  totalpred = pd.DataFrame()
  totalpred = pd.concat([totalpred, pred])

  repeatnum = int(1000 / repeat)
  for i in range(repeatnum):  #
    model.fit(x_test_t, y_test, epochs=10, batch_size =1, verbose =1)  
    if second > 0:
      x_test = x_data[-first:-second]
      y_test = y_data[-first:-second]
      x_test_t = np.array(x_test).reshape(len(x_test), 3, timestep)
      pred = model.predict(x_test_t)
      pred = np.exp(sc_y.inverse_transform(pred))
      pred = pd.DataFrame(pred, index= y_ymd[-first:-second], columns=['Prediction'])
      totalpred = pd.concat([totalpred, pred])
    elif second == 0:
      x_test = x_data[-first:]
      y_test = y_data[-first:]
      x_test_t = np.array(x_test).reshape(len(x_test), 3, timestep)
      pred = model.predict(x_test_t)
      pred = np.exp(sc_y.inverse_transform(pred))
      pred = pd.DataFrame(pred, index= y_ymd[-first:], columns=['Prediction'])
      totalpred = pd.concat([totalpred, pred])
    first = first-(repeat)
    second = second-(repeat)



  totalpred['Real']= np.exp(sc_y.inverse_transform(y_data[-1000:]))
  totalpred['Error'] = totalpred[totalpred.columns[1]] - totalpred[totalpred.columns[0]]
  return totalpred
```


```python
def pred_stock2(period,timestep):
  x_data = np.log(future_test)
  x_data = pd.DataFrame(sc_x.transform(x_data), index= x_data.index, columns=x_data.columns)
  #Time step 결정
  lst = []
  lst.append('High')
  for step in range(1,timestep):
      x_data['High_{}'.format(step)] = x_data['High'].shift(step)
      lst.append('High_'+str(step))
  lst.append('Low')
  for step in range(1,timestep):
      x_data['Low_{}'.format(step)] = x_data['Low'].shift(step)
      lst.append('Low_'+str(step))
  lst.append('Close')
  for step in range(1,timestep):
      x_data['Close_{}'.format(step)] = x_data['Close'].shift(step)
      lst.append('Close_'+str(step))
  x_data = x_data.dropna()
  x_data = x_data[lst] 
  #x_data reshape(len, feature, timestep) 
  x_data_t = np.array(x_data).reshape(len(x_data), 3, timestep) 

  #Prediction
  pred = model.predict(x_data_t)
  pred = np.exp(sc_y.inverse_transform(pred))
  
  tmplst=[]
  for i in range(1,period+1):
    tmpwd = future_test.index[-1] + " +"+str(i)+"일"
    tmplst.append(tmpwd)

  predict = pd.DataFrame(pred, columns = ['Col'])
  predict['Date'] = tmplst

  return predict
```

# Test

##Function List 

select_stock(stock, period, timestep)

prepro(df, column, timestep)

pred_stock(period,timestep)

## AAPL


```python
period = 10 
timestep = 2
df = select_stock('AAPL', period, timestep)
```

    /usr/local/lib/python3.6/dist-packages/pandas/core/indexing.py:670: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      iloc._setitem_with_indexer(indexer, value)
    


```python
df
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
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Target_YMD</th>
      <th>Target_High</th>
      <th>Target_Low</th>
      <th>Target_Close</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2000-01-03</th>
      <td>1.004464</td>
      <td>0.907924</td>
      <td>0.999442</td>
      <td>2000-01-18</td>
      <td>0.946429</td>
      <td>0.896763</td>
      <td>0.928013</td>
    </tr>
    <tr>
      <th>2000-01-04</th>
      <td>0.987723</td>
      <td>0.903460</td>
      <td>0.915179</td>
      <td>2000-01-19</td>
      <td>0.970982</td>
      <td>0.922991</td>
      <td>0.951451</td>
    </tr>
    <tr>
      <th>2000-01-05</th>
      <td>0.987165</td>
      <td>0.919643</td>
      <td>0.928571</td>
      <td>2000-01-20</td>
      <td>1.084821</td>
      <td>1.013393</td>
      <td>1.013393</td>
    </tr>
    <tr>
      <th>2000-01-06</th>
      <td>0.955357</td>
      <td>0.848214</td>
      <td>0.848214</td>
      <td>2000-01-21</td>
      <td>1.020089</td>
      <td>0.983817</td>
      <td>0.993862</td>
    </tr>
    <tr>
      <th>2000-01-07</th>
      <td>0.901786</td>
      <td>0.852679</td>
      <td>0.888393</td>
      <td>2000-01-24</td>
      <td>1.006696</td>
      <td>0.938616</td>
      <td>0.948661</td>
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
      <th>2020-11-10</th>
      <td>117.589996</td>
      <td>114.129997</td>
      <td>115.970001</td>
      <td>2020-11-24</td>
      <td>115.849998</td>
      <td>112.589996</td>
      <td>115.169998</td>
    </tr>
    <tr>
      <th>2020-11-11</th>
      <td>119.629997</td>
      <td>116.440002</td>
      <td>119.489998</td>
      <td>2020-11-25</td>
      <td>116.750000</td>
      <td>115.169998</td>
      <td>116.029999</td>
    </tr>
    <tr>
      <th>2020-11-12</th>
      <td>120.529999</td>
      <td>118.570000</td>
      <td>119.209999</td>
      <td>2020-11-27</td>
      <td>117.489998</td>
      <td>116.220001</td>
      <td>116.589996</td>
    </tr>
    <tr>
      <th>2020-11-13</th>
      <td>119.669998</td>
      <td>117.870003</td>
      <td>119.260002</td>
      <td>2020-11-30</td>
      <td>120.970001</td>
      <td>116.809998</td>
      <td>119.050003</td>
    </tr>
    <tr>
      <th>2020-11-16</th>
      <td>120.989998</td>
      <td>118.150002</td>
      <td>120.300003</td>
      <td>2020-12-01</td>
      <td>123.470001</td>
      <td>120.010002</td>
      <td>122.720001</td>
    </tr>
  </tbody>
</table>
<p>5253 rows × 7 columns</p>
</div>




```python
check = prepro2(df, 'Close', timestep)
check
```

    Epoch 1/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 0.0013
    Epoch 2/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.4808e-04
    Epoch 3/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.4623e-04
    Epoch 4/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.4619e-04
    Epoch 5/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.4326e-04
    Epoch 6/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.4135e-04
    Epoch 7/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.4283e-04
    Epoch 8/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.4136e-04
    Epoch 9/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.4463e-04
    Epoch 10/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.4426e-04
    Epoch 11/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.4009e-04
    Epoch 12/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.4057e-04
    Epoch 13/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.3666e-04
    Epoch 14/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.3539e-04
    Epoch 15/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.3477e-04
    Epoch 16/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.3483e-04
    Epoch 17/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.3644e-04
    Epoch 18/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.3640e-04
    Epoch 19/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.3403e-04
    Epoch 20/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.3072e-04
    Epoch 21/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.3469e-04
    Epoch 22/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.3613e-04
    Epoch 23/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.3151e-04
    Epoch 24/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.3056e-04
    Epoch 25/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.3336e-04
    Epoch 26/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.3068e-04
    Epoch 27/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.3248e-04
    Epoch 28/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.3233e-04
    Epoch 29/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.3082e-04
    Epoch 30/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.2940e-04
    Epoch 31/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.3125e-04
    Epoch 32/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.2869e-04
    Epoch 33/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.3162e-04
    Epoch 34/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.2957e-04
    Epoch 35/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.3166e-04
    Epoch 36/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.3060e-04
    Epoch 37/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.3040e-04
    Epoch 38/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.2852e-04
    Epoch 39/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.2924e-04
    Epoch 40/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.3155e-04
    Epoch 41/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.2909e-04
    Epoch 42/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.2705e-04
    Epoch 43/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.3096e-04
    Epoch 44/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.2715e-04
    Epoch 45/50
    4252/4252 [==============================] - 5s 1ms/step - loss: 2.2567e-04
    Epoch 46/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.2561e-04
    Epoch 47/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.2741e-04
    Epoch 48/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.2758e-04
    Epoch 49/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.2811e-04
    Epoch 50/50
    4252/4252 [==============================] - 6s 1ms/step - loss: 2.2580e-04
    Mean Error: 12.05541657615101
    




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
      <th>Predict_Close</th>
      <th>Real_Close</th>
      <th>Error_Close</th>
    </tr>
    <tr>
      <th>Target_YMD</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-12-12</th>
      <td>26.379490</td>
      <td>28.325001</td>
      <td>1.945511</td>
    </tr>
    <tr>
      <th>2016-12-13</th>
      <td>26.367498</td>
      <td>28.797501</td>
      <td>2.430002</td>
    </tr>
    <tr>
      <th>2016-12-14</th>
      <td>26.242331</td>
      <td>28.797501</td>
      <td>2.555170</td>
    </tr>
    <tr>
      <th>2016-12-15</th>
      <td>26.135492</td>
      <td>28.955000</td>
      <td>2.819508</td>
    </tr>
    <tr>
      <th>2016-12-16</th>
      <td>25.957323</td>
      <td>28.992500</td>
      <td>3.035177</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2020-11-24</th>
      <td>86.750862</td>
      <td>115.169998</td>
      <td>28.419136</td>
    </tr>
    <tr>
      <th>2020-11-25</th>
      <td>86.391640</td>
      <td>116.029999</td>
      <td>29.638359</td>
    </tr>
    <tr>
      <th>2020-11-27</th>
      <td>87.623283</td>
      <td>116.589996</td>
      <td>28.966713</td>
    </tr>
    <tr>
      <th>2020-11-30</th>
      <td>87.990311</td>
      <td>119.050003</td>
      <td>31.059692</td>
    </tr>
    <tr>
      <th>2020-12-01</th>
      <td>87.899857</td>
      <td>122.720001</td>
      <td>34.820145</td>
    </tr>
  </tbody>
</table>
<p>1000 rows × 3 columns</p>
</div>




```python
result = pred_stock2(period, timestep)
result
```

    Epoch 1/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.3947e-04
    Epoch 2/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1997e-04
    Epoch 3/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1403e-04
    Epoch 4/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1376e-04
    Epoch 5/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1314e-04
    Epoch 6/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1423e-04
    Epoch 7/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1465e-04
    Epoch 8/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1166e-04
    Epoch 9/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1180e-04
    Epoch 10/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1489e-04
    Epoch 11/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1398e-04
    Epoch 12/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1308e-04
    Epoch 13/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.2133e-04
    Epoch 14/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1842e-04
    Epoch 15/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1295e-04
    Epoch 16/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1870e-04
    Epoch 17/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1176e-04
    Epoch 18/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1496e-04
    Epoch 19/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.0997e-04
    Epoch 20/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1889e-04
    Epoch 21/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1564e-04
    Epoch 22/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1881e-04
    Epoch 23/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.0939e-04
    Epoch 24/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1406e-04
    Epoch 25/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.0857e-04
    Epoch 26/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1717e-04
    Epoch 27/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.0787e-04
    Epoch 28/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1149e-04
    Epoch 29/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1364e-04
    Epoch 30/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1346e-04
    Epoch 31/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1112e-04
    Epoch 32/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.2364e-04
    Epoch 33/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1055e-04
    Epoch 34/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1629e-04
    Epoch 35/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.0609e-04
    Epoch 36/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1093e-04
    Epoch 37/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.0908e-04
    Epoch 38/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1457e-04
    Epoch 39/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.2147e-04
    Epoch 40/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1379e-04
    Epoch 41/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1669e-04
    Epoch 42/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1496e-04
    Epoch 43/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1198e-04
    Epoch 44/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1163e-04
    Epoch 45/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1573e-04
    Epoch 46/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1057e-04
    Epoch 47/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1327e-04
    Epoch 48/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.0567e-04
    Epoch 49/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.1734e-04
    Epoch 50/50
    1000/1000 [==============================] - 1s 1ms/step - loss: 1.0974e-04
    




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
      <th>Col</th>
      <th>Adj_Col</th>
      <th>Date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>126.567406</td>
      <td>138.622818</td>
      <td>2020-12-01 +1일</td>
    </tr>
    <tr>
      <th>1</th>
      <td>126.255104</td>
      <td>138.310516</td>
      <td>2020-12-01 +2일</td>
    </tr>
    <tr>
      <th>2</th>
      <td>125.372261</td>
      <td>137.427673</td>
      <td>2020-12-01 +3일</td>
    </tr>
    <tr>
      <th>3</th>
      <td>124.934586</td>
      <td>136.990005</td>
      <td>2020-12-01 +4일</td>
    </tr>
    <tr>
      <th>4</th>
      <td>123.842461</td>
      <td>135.897873</td>
      <td>2020-12-01 +5일</td>
    </tr>
    <tr>
      <th>5</th>
      <td>121.598366</td>
      <td>133.653778</td>
      <td>2020-12-01 +6일</td>
    </tr>
    <tr>
      <th>6</th>
      <td>121.741028</td>
      <td>133.796448</td>
      <td>2020-12-01 +7일</td>
    </tr>
    <tr>
      <th>7</th>
      <td>123.148941</td>
      <td>135.204361</td>
      <td>2020-12-01 +8일</td>
    </tr>
    <tr>
      <th>8</th>
      <td>124.205513</td>
      <td>136.260925</td>
      <td>2020-12-01 +9일</td>
    </tr>
    <tr>
      <th>9</th>
      <td>126.408836</td>
      <td>138.464249</td>
      <td>2020-12-01 +10일</td>
    </tr>
  </tbody>
</table>
</div>




```python
result.to_excel('aapl_pred.xlsx', index =False)
```


```python
check.plot()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7fe13a789780>




![png](Function_Predict_files/Function_Predict_19_1.png)


## 삼성전자(005930.KS)


```python
period = 10
timestep = 5
df = select_stock('005930.KS', period, timestep)
```

    /usr/local/lib/python3.6/dist-packages/pandas/core/indexing.py:671: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      self._setitem_with_indexer(indexer, value)
    


```python
check = prepro(df, 'Close', timestep)
check
```

    Epoch 1/10
    4141/4141 [==============================] - 7s 2ms/step - loss: 0.0022
    Epoch 2/10
    4141/4141 [==============================] - 7s 2ms/step - loss: 5.9716e-04
    Epoch 3/10
    4141/4141 [==============================] - 7s 2ms/step - loss: 5.9004e-04
    Epoch 4/10
    4141/4141 [==============================] - 7s 2ms/step - loss: 5.8178e-04
    Epoch 5/10
    4141/4141 [==============================] - 7s 2ms/step - loss: 5.6538e-04
    Epoch 6/10
    4141/4141 [==============================] - 7s 2ms/step - loss: 5.8092e-04
    Epoch 7/10
    4141/4141 [==============================] - 8s 2ms/step - loss: 5.5544e-04
    Epoch 8/10
    4141/4141 [==============================] - 8s 2ms/step - loss: 5.5421e-04
    Epoch 9/10
    4141/4141 [==============================] - 7s 2ms/step - loss: 5.5031e-04
    Epoch 10/10
    4141/4141 [==============================] - 7s 2ms/step - loss: 5.4925e-04
    Mean Error: 3946.786350965143
    




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
      <th>Predict_Close</th>
      <th>Real_Close</th>
      <th>Error_Close</th>
    </tr>
    <tr>
      <th>Target_YMD</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-06-10</th>
      <td>26192.621094</td>
      <td>28120.0</td>
      <td>1927.378906</td>
    </tr>
    <tr>
      <th>2016-06-13</th>
      <td>26127.257812</td>
      <td>27420.0</td>
      <td>1292.742187</td>
    </tr>
    <tr>
      <th>2016-06-14</th>
      <td>26074.412109</td>
      <td>27600.0</td>
      <td>1525.587891</td>
    </tr>
    <tr>
      <th>2016-06-15</th>
      <td>26138.671875</td>
      <td>28260.0</td>
      <td>2121.328125</td>
    </tr>
    <tr>
      <th>2016-06-16</th>
      <td>26438.666016</td>
      <td>28180.0</td>
      <td>1741.333984</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2020-07-09</th>
      <td>47303.449219</td>
      <td>52800.0</td>
      <td>5496.550781</td>
    </tr>
    <tr>
      <th>2020-07-10</th>
      <td>47383.273438</td>
      <td>52700.0</td>
      <td>5316.726563</td>
    </tr>
    <tr>
      <th>2020-07-13</th>
      <td>47506.074219</td>
      <td>53400.0</td>
      <td>5893.925781</td>
    </tr>
    <tr>
      <th>2020-07-14</th>
      <td>47538.386719</td>
      <td>53800.0</td>
      <td>6261.613281</td>
    </tr>
    <tr>
      <th>2020-07-15</th>
      <td>47620.425781</td>
      <td>54700.0</td>
      <td>7079.574219</td>
    </tr>
  </tbody>
</table>
<p>1000 rows × 3 columns</p>
</div>




```python
result = pred_stock(period, timestep)
result
```

    Epoch 1/10
    1000/1000 [==============================] - 2s 2ms/step - loss: 3.3458e-04
    Epoch 2/10
    1000/1000 [==============================] - 2s 2ms/step - loss: 3.2647e-04
    Epoch 3/10
    1000/1000 [==============================] - 2s 2ms/step - loss: 3.1761e-04
    Epoch 4/10
    1000/1000 [==============================] - 2s 2ms/step - loss: 3.2343e-04
    Epoch 5/10
    1000/1000 [==============================] - 2s 2ms/step - loss: 3.5005e-04
    Epoch 6/10
    1000/1000 [==============================] - 2s 2ms/step - loss: 3.2447e-04
    Epoch 7/10
    1000/1000 [==============================] - 2s 2ms/step - loss: 3.3307e-04
    Epoch 8/10
    1000/1000 [==============================] - 2s 2ms/step - loss: 3.1031e-04
    Epoch 9/10
    1000/1000 [==============================] - 2s 2ms/step - loss: 3.3700e-04
    Epoch 10/10
    1000/1000 [==============================] - 2s 2ms/step - loss: 3.3359e-04
    




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
      <th>Col</th>
      <th>Adj_Col</th>
      <th>Date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>49671.929688</td>
      <td>53618.714844</td>
      <td>2020-07-15 +1일</td>
    </tr>
    <tr>
      <th>1</th>
      <td>49892.359375</td>
      <td>53839.144531</td>
      <td>2020-07-15 +2일</td>
    </tr>
    <tr>
      <th>2</th>
      <td>50470.460938</td>
      <td>54417.246094</td>
      <td>2020-07-15 +3일</td>
    </tr>
    <tr>
      <th>3</th>
      <td>50541.941406</td>
      <td>54488.726562</td>
      <td>2020-07-15 +4일</td>
    </tr>
    <tr>
      <th>4</th>
      <td>50226.781250</td>
      <td>54173.566406</td>
      <td>2020-07-15 +5일</td>
    </tr>
    <tr>
      <th>5</th>
      <td>50167.179688</td>
      <td>54113.964844</td>
      <td>2020-07-15 +6일</td>
    </tr>
    <tr>
      <th>6</th>
      <td>49999.722656</td>
      <td>53946.507812</td>
      <td>2020-07-15 +7일</td>
    </tr>
    <tr>
      <th>7</th>
      <td>50004.917969</td>
      <td>53951.703125</td>
      <td>2020-07-15 +8일</td>
    </tr>
    <tr>
      <th>8</th>
      <td>50174.214844</td>
      <td>54121.000000</td>
      <td>2020-07-15 +9일</td>
    </tr>
    <tr>
      <th>9</th>
      <td>50537.554688</td>
      <td>54484.339844</td>
      <td>2020-07-15 +10일</td>
    </tr>
  </tbody>
</table>
</div>




```python
result.to_excel('samsung_pred.xlsx', index =False)
```


```python
check.plot()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7f3af48e2dd8>




![png](Function_Predict_files/Function_Predict_25_1.png)


## MSFT


```python
period = 20
timestep = 10
column = 'Close'
df = select_stock('MSFT', period, timestep)
```

    /usr/local/lib/python3.6/dist-packages/pandas/core/indexing.py:671: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      self._setitem_with_indexer(indexer, value)
    


```python
setup(timestep,column)
```

    Epoch 1/10
    4135/4135 [==============================] - 7s 2ms/step - loss: 0.0013
    Epoch 2/10
    4135/4135 [==============================] - 7s 2ms/step - loss: 0.0011
    Epoch 3/10
    4135/4135 [==============================] - 7s 2ms/step - loss: 0.0011
    Epoch 4/10
    4135/4135 [==============================] - 8s 2ms/step - loss: 0.0011
    Epoch 5/10
    4135/4135 [==============================] - 7s 2ms/step - loss: 0.0010
    Epoch 6/10
    4135/4135 [==============================] - 7s 2ms/step - loss: 0.0010
    Epoch 7/10
    4135/4135 [==============================] - 7s 2ms/step - loss: 0.0010
    Epoch 8/10
    4135/4135 [==============================] - 8s 2ms/step - loss: 0.0010
    Epoch 9/10
    4135/4135 [==============================] - 8s 2ms/step - loss: 0.0010
    Epoch 10/10
    4135/4135 [==============================] - 8s 2ms/step - loss: 0.0010
    


```python
result = update(50)
result
```

    Epoch 1/10
    50/50 [==============================] - 0s 2ms/step - loss: 9.1269e-05
    Epoch 2/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.3101e-05
    Epoch 3/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.4630e-05
    Epoch 4/10
    50/50 [==============================] - 0s 2ms/step - loss: 8.0182e-05
    Epoch 5/10
    50/50 [==============================] - 0s 2ms/step - loss: 6.9294e-05
    Epoch 6/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.9360e-05
    Epoch 7/10
    50/50 [==============================] - 0s 2ms/step - loss: 5.2536e-05
    Epoch 8/10
    50/50 [==============================] - 0s 2ms/step - loss: 6.0518e-05
    Epoch 9/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.5363e-05
    Epoch 10/10
    50/50 [==============================] - 0s 2ms/step - loss: 6.3527e-05
    Epoch 1/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.0699e-04
    Epoch 2/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.0993e-04
    Epoch 3/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.0454e-04
    Epoch 4/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.0899e-04
    Epoch 5/10
    50/50 [==============================] - 0s 2ms/step - loss: 9.8274e-05
    Epoch 6/10
    50/50 [==============================] - 0s 2ms/step - loss: 8.4866e-05
    Epoch 7/10
    50/50 [==============================] - 0s 2ms/step - loss: 9.0441e-05
    Epoch 8/10
    50/50 [==============================] - 0s 2ms/step - loss: 7.7547e-05
    Epoch 9/10
    50/50 [==============================] - 0s 2ms/step - loss: 8.4070e-05
    Epoch 10/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.1453e-04
    Epoch 1/10
    50/50 [==============================] - 0s 2ms/step - loss: 5.0003e-05
    Epoch 2/10
    50/50 [==============================] - 0s 2ms/step - loss: 6.1968e-05
    Epoch 3/10
    50/50 [==============================] - 0s 2ms/step - loss: 3.6373e-05
    Epoch 4/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.1385e-05
    Epoch 5/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.7575e-05
    Epoch 6/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.0350e-05
    Epoch 7/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.3299e-05
    Epoch 8/10
    50/50 [==============================] - 0s 2ms/step - loss: 5.0506e-05
    Epoch 9/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.4551e-05
    Epoch 10/10
    50/50 [==============================] - 0s 2ms/step - loss: 5.6068e-05
    Epoch 1/10
    50/50 [==============================] - 0s 2ms/step - loss: 5.6171e-05
    Epoch 2/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.3464e-05
    Epoch 3/10
    50/50 [==============================] - 0s 2ms/step - loss: 5.0059e-05
    Epoch 4/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.5625e-05
    Epoch 5/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.0451e-05
    Epoch 6/10
    50/50 [==============================] - 0s 2ms/step - loss: 3.9607e-05
    Epoch 7/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.7478e-05
    Epoch 8/10
    50/50 [==============================] - 0s 2ms/step - loss: 5.4750e-05
    Epoch 9/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.9080e-05
    Epoch 10/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.4160e-05
    Epoch 1/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.1373e-04
    Epoch 2/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.2427e-04
    Epoch 3/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.1719e-04
    Epoch 4/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.1925e-04
    Epoch 5/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.0299e-04
    Epoch 6/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.1217e-04
    Epoch 7/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.3689e-04
    Epoch 8/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.4621e-04
    Epoch 9/10
    50/50 [==============================] - 0s 2ms/step - loss: 9.2797e-05
    Epoch 10/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.0244e-04
    Epoch 1/10
    50/50 [==============================] - 0s 2ms/step - loss: 3.8074e-05
    Epoch 2/10
    50/50 [==============================] - 0s 2ms/step - loss: 3.2841e-05
    Epoch 3/10
    50/50 [==============================] - 0s 2ms/step - loss: 3.2613e-05
    Epoch 4/10
    50/50 [==============================] - 0s 2ms/step - loss: 3.5054e-05
    Epoch 5/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.3999e-05
    Epoch 6/10
    50/50 [==============================] - 0s 2ms/step - loss: 3.2490e-05
    Epoch 7/10
    50/50 [==============================] - 0s 2ms/step - loss: 2.9042e-05
    Epoch 8/10
    50/50 [==============================] - 0s 2ms/step - loss: 3.5724e-05
    Epoch 9/10
    50/50 [==============================] - 0s 2ms/step - loss: 2.8305e-05
    Epoch 10/10
    50/50 [==============================] - 0s 2ms/step - loss: 3.7141e-05
    Epoch 1/10
    50/50 [==============================] - 0s 2ms/step - loss: 8.8198e-05
    Epoch 2/10
    50/50 [==============================] - 0s 2ms/step - loss: 8.0256e-05
    Epoch 3/10
    50/50 [==============================] - 0s 2ms/step - loss: 8.0282e-05
    Epoch 4/10
    50/50 [==============================] - 0s 2ms/step - loss: 9.5039e-05
    Epoch 5/10
    50/50 [==============================] - 0s 2ms/step - loss: 8.8539e-05
    Epoch 6/10
    50/50 [==============================] - 0s 2ms/step - loss: 9.3109e-05
    Epoch 7/10
    50/50 [==============================] - 0s 2ms/step - loss: 7.9496e-05
    Epoch 8/10
    50/50 [==============================] - 0s 2ms/step - loss: 7.1264e-05
    Epoch 9/10
    50/50 [==============================] - 0s 2ms/step - loss: 7.5635e-05
    Epoch 10/10
    50/50 [==============================] - 0s 2ms/step - loss: 7.5135e-05
    Epoch 1/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.5454e-04
    Epoch 2/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.3612e-04
    Epoch 3/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.0508e-04
    Epoch 4/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.1387e-04
    Epoch 5/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.0557e-04
    Epoch 6/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.2982e-04
    Epoch 7/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.4384e-04
    Epoch 8/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.1039e-04
    Epoch 9/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.1677e-04
    Epoch 10/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.5562e-04
    Epoch 1/10
    50/50 [==============================] - 0s 2ms/step - loss: 9.7033e-05
    Epoch 2/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.1252e-04
    Epoch 3/10
    50/50 [==============================] - 0s 2ms/step - loss: 9.4856e-05
    Epoch 4/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.0265e-04
    Epoch 5/10
    50/50 [==============================] - 0s 2ms/step - loss: 8.9839e-05
    Epoch 6/10
    50/50 [==============================] - 0s 2ms/step - loss: 9.9160e-05
    Epoch 7/10
    50/50 [==============================] - 0s 2ms/step - loss: 9.7565e-05
    Epoch 8/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.0110e-04
    Epoch 9/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.0282e-04
    Epoch 10/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.0627e-04
    Epoch 1/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.2477e-04
    Epoch 2/10
    50/50 [==============================] - 0s 2ms/step - loss: 8.4776e-05
    Epoch 3/10
    50/50 [==============================] - 0s 2ms/step - loss: 5.7490e-05
    Epoch 4/10
    50/50 [==============================] - 0s 2ms/step - loss: 5.7820e-05
    Epoch 5/10
    50/50 [==============================] - 0s 2ms/step - loss: 9.1274e-05
    Epoch 6/10
    50/50 [==============================] - 0s 2ms/step - loss: 6.4840e-05
    Epoch 7/10
    50/50 [==============================] - 0s 2ms/step - loss: 6.3209e-05
    Epoch 8/10
    50/50 [==============================] - 0s 2ms/step - loss: 8.8236e-05
    Epoch 9/10
    50/50 [==============================] - 0s 2ms/step - loss: 6.9087e-05
    Epoch 10/10
    50/50 [==============================] - 0s 2ms/step - loss: 7.7937e-05
    Epoch 1/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.0345e-04
    Epoch 2/10
    50/50 [==============================] - 0s 2ms/step - loss: 6.9168e-05
    Epoch 3/10
    50/50 [==============================] - 0s 2ms/step - loss: 6.6587e-05
    Epoch 4/10
    50/50 [==============================] - 0s 2ms/step - loss: 7.4108e-05
    Epoch 5/10
    50/50 [==============================] - 0s 2ms/step - loss: 7.8583e-05
    Epoch 6/10
    50/50 [==============================] - 0s 2ms/step - loss: 7.2909e-05
    Epoch 7/10
    50/50 [==============================] - 0s 2ms/step - loss: 6.8282e-05
    Epoch 8/10
    50/50 [==============================] - 0s 2ms/step - loss: 8.4517e-05
    Epoch 9/10
    50/50 [==============================] - 0s 2ms/step - loss: 7.8282e-05
    Epoch 10/10
    50/50 [==============================] - 0s 2ms/step - loss: 7.8037e-05
    Epoch 1/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.9526e-04
    Epoch 2/10
    50/50 [==============================] - 0s 2ms/step - loss: 2.0144e-04
    Epoch 3/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.6852e-04
    Epoch 4/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.6393e-04
    Epoch 5/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.7072e-04
    Epoch 6/10
    50/50 [==============================] - 0s 2ms/step - loss: 2.5687e-04
    Epoch 7/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.9744e-04
    Epoch 8/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.7149e-04
    Epoch 9/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.7218e-04
    Epoch 10/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.7016e-04
    Epoch 1/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.5113e-04
    Epoch 2/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.4390e-04
    Epoch 3/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.3172e-04
    Epoch 4/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.5226e-04
    Epoch 5/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.8996e-04
    Epoch 6/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.3542e-04
    Epoch 7/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.6668e-04
    Epoch 8/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.5286e-04
    Epoch 9/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.3409e-04
    Epoch 10/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.2607e-04
    Epoch 1/10
    50/50 [==============================] - 0s 2ms/step - loss: 2.7190e-04
    Epoch 2/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.6543e-04
    Epoch 3/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.5598e-04
    Epoch 4/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.8519e-04
    Epoch 5/10
    50/50 [==============================] - 0s 2ms/step - loss: 2.3049e-04
    Epoch 6/10
    50/50 [==============================] - 0s 2ms/step - loss: 2.0360e-04
    Epoch 7/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.7664e-04
    Epoch 8/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.5090e-04
    Epoch 9/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.5065e-04
    Epoch 10/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.5303e-04
    Epoch 1/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.8724e-04
    Epoch 2/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.7375e-04
    Epoch 3/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.4980e-04
    Epoch 4/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.6724e-04
    Epoch 5/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.7157e-04
    Epoch 6/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.6678e-04
    Epoch 7/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.5745e-04
    Epoch 8/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.8896e-04
    Epoch 9/10
    50/50 [==============================] - 0s 2ms/step - loss: 2.6394e-04
    Epoch 10/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.6558e-04
    Epoch 1/10
    50/50 [==============================] - 0s 2ms/step - loss: 7.9018e-05
    Epoch 2/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.1982e-05
    Epoch 3/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.8577e-05
    Epoch 4/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.1977e-05
    Epoch 5/10
    50/50 [==============================] - 0s 2ms/step - loss: 3.9112e-05
    Epoch 6/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.3973e-05
    Epoch 7/10
    50/50 [==============================] - 0s 2ms/step - loss: 3.9736e-05
    Epoch 8/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.6780e-05
    Epoch 9/10
    50/50 [==============================] - 0s 2ms/step - loss: 5.0239e-05
    Epoch 10/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.2415e-05
    Epoch 1/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.3305e-04
    Epoch 2/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.2999e-04
    Epoch 3/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.4036e-04
    Epoch 4/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.3371e-04
    Epoch 5/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.7802e-04
    Epoch 6/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.1886e-04
    Epoch 7/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.1178e-04
    Epoch 8/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.5105e-04
    Epoch 9/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.4053e-04
    Epoch 10/10
    50/50 [==============================] - 0s 2ms/step - loss: 1.1676e-04
    Epoch 1/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.6163e-04
    Epoch 2/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.2099e-04
    Epoch 3/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.1003e-04
    Epoch 4/10
    50/50 [==============================] - 0s 2ms/step - loss: 3.7249e-04
    Epoch 5/10
    50/50 [==============================] - 0s 2ms/step - loss: 3.5698e-04
    Epoch 6/10
    50/50 [==============================] - 0s 2ms/step - loss: 3.5193e-04
    Epoch 7/10
    50/50 [==============================] - 0s 2ms/step - loss: 3.5697e-04
    Epoch 8/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.2024e-04
    Epoch 9/10
    50/50 [==============================] - 0s 2ms/step - loss: 2.8532e-04
    Epoch 10/10
    50/50 [==============================] - 0s 2ms/step - loss: 4.0341e-04
    Epoch 1/10
    50/50 [==============================] - 0s 2ms/step - loss: 0.0012
    Epoch 2/10
    50/50 [==============================] - 0s 2ms/step - loss: 7.9958e-04
    Epoch 3/10
    50/50 [==============================] - 0s 2ms/step - loss: 8.1350e-04
    Epoch 4/10
    50/50 [==============================] - 0s 2ms/step - loss: 7.3014e-04
    Epoch 5/10
    50/50 [==============================] - 0s 2ms/step - loss: 7.2005e-04
    Epoch 6/10
    50/50 [==============================] - 0s 2ms/step - loss: 8.0338e-04
    Epoch 7/10
    50/50 [==============================] - 0s 2ms/step - loss: 6.8330e-04
    Epoch 8/10
    50/50 [==============================] - 0s 2ms/step - loss: 6.9225e-04
    Epoch 9/10
    50/50 [==============================] - 0s 2ms/step - loss: 7.6283e-04
    Epoch 10/10
    50/50 [==============================] - 0s 2ms/step - loss: 7.6555e-04
    Epoch 1/10
    50/50 [==============================] - 0s 2ms/step - loss: 3.1787e-04
    Epoch 2/10
    50/50 [==============================] - 0s 2ms/step - loss: 2.3896e-04
    Epoch 3/10
    50/50 [==============================] - 0s 2ms/step - loss: 2.3594e-04
    Epoch 4/10
    50/50 [==============================] - 0s 2ms/step - loss: 2.1226e-04
    Epoch 5/10
    50/50 [==============================] - 0s 2ms/step - loss: 2.2744e-04
    Epoch 6/10
    50/50 [==============================] - 0s 2ms/step - loss: 2.2898e-04
    Epoch 7/10
    50/50 [==============================] - 0s 2ms/step - loss: 2.2934e-04
    Epoch 8/10
    50/50 [==============================] - 0s 2ms/step - loss: 2.9737e-04
    Epoch 9/10
    50/50 [==============================] - 0s 2ms/step - loss: 2.6800e-04
    Epoch 10/10
    50/50 [==============================] - 0s 2ms/step - loss: 2.2471e-04
    




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
      <th>Prediction</th>
      <th>Real</th>
      <th>Error</th>
    </tr>
    <tr>
      <th>Target_YMD</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-07-22</th>
      <td>53.724804</td>
      <td>56.570000</td>
      <td>2.845196</td>
    </tr>
    <tr>
      <th>2016-07-25</th>
      <td>53.906975</td>
      <td>56.730000</td>
      <td>2.823025</td>
    </tr>
    <tr>
      <th>2016-07-26</th>
      <td>53.702251</td>
      <td>56.759998</td>
      <td>3.057747</td>
    </tr>
    <tr>
      <th>2016-07-27</th>
      <td>54.657906</td>
      <td>56.189999</td>
      <td>1.532093</td>
    </tr>
    <tr>
      <th>2016-07-28</th>
      <td>55.326778</td>
      <td>56.209999</td>
      <td>0.883221</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2020-07-07</th>
      <td>180.082779</td>
      <td>208.250000</td>
      <td>28.167221</td>
    </tr>
    <tr>
      <th>2020-07-08</th>
      <td>180.804672</td>
      <td>212.830002</td>
      <td>32.025330</td>
    </tr>
    <tr>
      <th>2020-07-09</th>
      <td>181.473724</td>
      <td>214.320007</td>
      <td>32.846283</td>
    </tr>
    <tr>
      <th>2020-07-10</th>
      <td>182.241852</td>
      <td>213.669998</td>
      <td>31.428146</td>
    </tr>
    <tr>
      <th>2020-07-13</th>
      <td>183.175888</td>
      <td>207.070007</td>
      <td>23.894119</td>
    </tr>
  </tbody>
</table>
<p>1000 rows × 3 columns</p>
</div>




```python
result.plot()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7f0fe92b9908>




![png](Function_Predict_files/Function_Predict_30_1.png)



```python
pred_stock2(period, timestep)
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
      <th>Col</th>
      <th>Date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>198.527267</td>
      <td>2020-07-13 +1일</td>
    </tr>
    <tr>
      <th>1</th>
      <td>199.337738</td>
      <td>2020-07-13 +2일</td>
    </tr>
    <tr>
      <th>2</th>
      <td>200.128510</td>
      <td>2020-07-13 +3일</td>
    </tr>
    <tr>
      <th>3</th>
      <td>200.558502</td>
      <td>2020-07-13 +4일</td>
    </tr>
    <tr>
      <th>4</th>
      <td>200.105331</td>
      <td>2020-07-13 +5일</td>
    </tr>
  </tbody>
</table>
</div>




```python
result[800:850]
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
      <th>Prediction</th>
      <th>Real</th>
      <th>Error</th>
    </tr>
    <tr>
      <th>Target_YMD</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2019-09-26</th>
      <td>136.434479</td>
      <td>139.539993</td>
      <td>3.105515</td>
    </tr>
    <tr>
      <th>2019-09-27</th>
      <td>136.747620</td>
      <td>137.729996</td>
      <td>0.982376</td>
    </tr>
    <tr>
      <th>2019-09-30</th>
      <td>136.774750</td>
      <td>139.029999</td>
      <td>2.255249</td>
    </tr>
    <tr>
      <th>2019-10-01</th>
      <td>136.585297</td>
      <td>137.070007</td>
      <td>0.484711</td>
    </tr>
    <tr>
      <th>2019-10-02</th>
      <td>136.468628</td>
      <td>134.649994</td>
      <td>-1.818634</td>
    </tr>
    <tr>
      <th>2019-10-03</th>
      <td>136.657471</td>
      <td>136.279999</td>
      <td>-0.377472</td>
    </tr>
    <tr>
      <th>2019-10-04</th>
      <td>136.579758</td>
      <td>138.119995</td>
      <td>1.540237</td>
    </tr>
    <tr>
      <th>2019-10-07</th>
      <td>136.566803</td>
      <td>137.119995</td>
      <td>0.553192</td>
    </tr>
    <tr>
      <th>2019-10-08</th>
      <td>136.415405</td>
      <td>135.669998</td>
      <td>-0.745407</td>
    </tr>
    <tr>
      <th>2019-10-09</th>
      <td>135.725113</td>
      <td>138.240005</td>
      <td>2.514893</td>
    </tr>
    <tr>
      <th>2019-10-10</th>
      <td>135.566193</td>
      <td>139.100006</td>
      <td>3.533813</td>
    </tr>
    <tr>
      <th>2019-10-11</th>
      <td>135.700211</td>
      <td>139.679993</td>
      <td>3.979782</td>
    </tr>
    <tr>
      <th>2019-10-14</th>
      <td>135.833313</td>
      <td>139.550003</td>
      <td>3.716690</td>
    </tr>
    <tr>
      <th>2019-10-15</th>
      <td>135.730103</td>
      <td>141.570007</td>
      <td>5.839905</td>
    </tr>
    <tr>
      <th>2019-10-16</th>
      <td>135.818802</td>
      <td>140.410004</td>
      <td>4.591202</td>
    </tr>
    <tr>
      <th>2019-10-17</th>
      <td>136.110611</td>
      <td>139.690002</td>
      <td>3.579391</td>
    </tr>
    <tr>
      <th>2019-10-18</th>
      <td>136.390961</td>
      <td>137.410004</td>
      <td>1.019043</td>
    </tr>
    <tr>
      <th>2019-10-21</th>
      <td>136.707794</td>
      <td>138.429993</td>
      <td>1.722198</td>
    </tr>
    <tr>
      <th>2019-10-22</th>
      <td>137.078293</td>
      <td>136.369995</td>
      <td>-0.708298</td>
    </tr>
    <tr>
      <th>2019-10-23</th>
      <td>137.156036</td>
      <td>137.240005</td>
      <td>0.083969</td>
    </tr>
    <tr>
      <th>2019-10-24</th>
      <td>137.070847</td>
      <td>139.940002</td>
      <td>2.869156</td>
    </tr>
    <tr>
      <th>2019-10-25</th>
      <td>136.791580</td>
      <td>140.729996</td>
      <td>3.938416</td>
    </tr>
    <tr>
      <th>2019-10-28</th>
      <td>136.619492</td>
      <td>144.190002</td>
      <td>7.570511</td>
    </tr>
    <tr>
      <th>2019-10-29</th>
      <td>136.292984</td>
      <td>142.830002</td>
      <td>6.537018</td>
    </tr>
    <tr>
      <th>2019-10-30</th>
      <td>136.065186</td>
      <td>144.610001</td>
      <td>8.544815</td>
    </tr>
    <tr>
      <th>2019-10-31</th>
      <td>136.363968</td>
      <td>143.369995</td>
      <td>7.006027</td>
    </tr>
    <tr>
      <th>2019-11-01</th>
      <td>136.714050</td>
      <td>143.720001</td>
      <td>7.005951</td>
    </tr>
    <tr>
      <th>2019-11-04</th>
      <td>137.356766</td>
      <td>144.550003</td>
      <td>7.193237</td>
    </tr>
    <tr>
      <th>2019-11-05</th>
      <td>137.868393</td>
      <td>144.460007</td>
      <td>6.591614</td>
    </tr>
    <tr>
      <th>2019-11-06</th>
      <td>138.134171</td>
      <td>144.059998</td>
      <td>5.925827</td>
    </tr>
    <tr>
      <th>2019-11-07</th>
      <td>138.422577</td>
      <td>144.259995</td>
      <td>5.837418</td>
    </tr>
    <tr>
      <th>2019-11-08</th>
      <td>138.405685</td>
      <td>145.960007</td>
      <td>7.554321</td>
    </tr>
    <tr>
      <th>2019-11-11</th>
      <td>138.710922</td>
      <td>146.110001</td>
      <td>7.399078</td>
    </tr>
    <tr>
      <th>2019-11-12</th>
      <td>138.856384</td>
      <td>147.070007</td>
      <td>8.213623</td>
    </tr>
    <tr>
      <th>2019-11-13</th>
      <td>138.780396</td>
      <td>147.309998</td>
      <td>8.529602</td>
    </tr>
    <tr>
      <th>2019-11-14</th>
      <td>138.834808</td>
      <td>148.059998</td>
      <td>9.225189</td>
    </tr>
    <tr>
      <th>2019-11-15</th>
      <td>138.896454</td>
      <td>149.970001</td>
      <td>11.073547</td>
    </tr>
    <tr>
      <th>2019-11-18</th>
      <td>139.165405</td>
      <td>150.339996</td>
      <td>11.174591</td>
    </tr>
    <tr>
      <th>2019-11-19</th>
      <td>139.385239</td>
      <td>150.389999</td>
      <td>11.004761</td>
    </tr>
    <tr>
      <th>2019-11-20</th>
      <td>139.652145</td>
      <td>149.619995</td>
      <td>9.967850</td>
    </tr>
    <tr>
      <th>2019-11-21</th>
      <td>139.864594</td>
      <td>149.479996</td>
      <td>9.615402</td>
    </tr>
    <tr>
      <th>2019-11-22</th>
      <td>140.224731</td>
      <td>149.589996</td>
      <td>9.365265</td>
    </tr>
    <tr>
      <th>2019-11-25</th>
      <td>140.592102</td>
      <td>151.229996</td>
      <td>10.637894</td>
    </tr>
    <tr>
      <th>2019-11-26</th>
      <td>140.865677</td>
      <td>152.029999</td>
      <td>11.164322</td>
    </tr>
    <tr>
      <th>2019-11-27</th>
      <td>140.956589</td>
      <td>152.320007</td>
      <td>11.363419</td>
    </tr>
    <tr>
      <th>2019-11-29</th>
      <td>140.871597</td>
      <td>151.380005</td>
      <td>10.508408</td>
    </tr>
    <tr>
      <th>2019-12-02</th>
      <td>140.867096</td>
      <td>149.550003</td>
      <td>8.682907</td>
    </tr>
    <tr>
      <th>2019-12-03</th>
      <td>141.040634</td>
      <td>149.309998</td>
      <td>8.269363</td>
    </tr>
    <tr>
      <th>2019-12-04</th>
      <td>141.344604</td>
      <td>149.850006</td>
      <td>8.505402</td>
    </tr>
    <tr>
      <th>2019-12-05</th>
      <td>141.566452</td>
      <td>149.929993</td>
      <td>8.363541</td>
    </tr>
  </tbody>
</table>
</div>



# prac


```python
check
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
      <th>Predict_Close</th>
      <th>Real_Close</th>
      <th>Error_Close</th>
    </tr>
    <tr>
      <th>Target_YMD</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-07-22</th>
      <td>92.75</td>
      <td>1.61</td>
      <td>5.91</td>
    </tr>
    <tr>
      <th>2016-07-25</th>
      <td>92.98</td>
      <td>97.34</td>
      <td>4.36</td>
    </tr>
    <tr>
      <th>2016-07-26</th>
      <td>93.37</td>
      <td>96.67</td>
      <td>3.30</td>
    </tr>
    <tr>
      <th>2016-07-27</th>
      <td>93.55</td>
      <td>102.95</td>
      <td>9.40</td>
    </tr>
    <tr>
      <th>2016-07-28</th>
      <td>94.11</td>
      <td>104.34</td>
      <td>10.23</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2020-07-07</th>
      <td>279.24</td>
      <td>372.69</td>
      <td>93.45</td>
    </tr>
    <tr>
      <th>2020-07-08</th>
      <td>282.12</td>
      <td>381.37</td>
      <td>99.25</td>
    </tr>
    <tr>
      <th>2020-07-09</th>
      <td>282.59</td>
      <td>383.01</td>
      <td>100.42</td>
    </tr>
    <tr>
      <th>2020-07-10</th>
      <td>283.42</td>
      <td>383.68</td>
      <td>100.26</td>
    </tr>
    <tr>
      <th>2020-07-13</th>
      <td>283.12</td>
      <td>381.91</td>
      <td>98.79</td>
    </tr>
  </tbody>
</table>
<p>1000 rows × 3 columns</p>
</div>




```python
temp = pd.DataFrame()
lennum = int(len(check))-1
temp['Di_Predict'] = [x for x in range(lennum)]
temp['Di_Real'] = [x for x in range(lennum)]
lennum = int(len(check))-1
for i in range(lennum):
  temp['Di_Predict'][i] = check['Predict_Close'][i+1] - check['Predict_Close'][i]
  temp['Di_Real'][i] = check['Real_Close'][i+1] - check['Real_Close'][i]

temp
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
      <th>Di_Predict</th>
      <th>Di_Real</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>102</td>
      <td>-479</td>
    </tr>
    <tr>
      <th>1</th>
      <td>11</td>
      <td>-700</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-83</td>
      <td>180</td>
    </tr>
    <tr>
      <th>3</th>
      <td>60</td>
      <td>659</td>
    </tr>
    <tr>
      <th>4</th>
      <td>341</td>
      <td>-79</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>994</th>
      <td>213</td>
      <td>-399</td>
    </tr>
    <tr>
      <th>995</th>
      <td>-33</td>
      <td>-200</td>
    </tr>
    <tr>
      <th>996</th>
      <td>265</td>
      <td>-99</td>
    </tr>
    <tr>
      <th>997</th>
      <td>-172</td>
      <td>699</td>
    </tr>
    <tr>
      <th>998</th>
      <td>343</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>999 rows × 2 columns</p>
</div>




```python
from matplotlib import pyplot as plt
i = 8
plt.figure(figsize=(20,12))
plt.grid()
plt.plot(range(100), temp['Di_Real'][0+(i*100):100+(i*100)], label = 'Real', alpha = 0.5 ,linewidth=3.0)
plt.plot(range(100), temp['Di_Predict'][0+(i*100):100+(i*100)], label = 'Predict', alpha = 0.9, linewidth=3.0)
plt.legend(loc='best')
plt.show()
```


![png](Function_Predict_files/Function_Predict_36_0.png)



```python
len(temp['Di_Real'][0:100])
```




    100




```python
temp['Di_Real'] = np.array(temp['Di_Real']).rand(5)
```


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    <ipython-input-122-e8fc0b8794b3> in <module>()
    ----> 1 temp['Di_Real'] = np.array(temp['Di_Real']).rand(5)
    

    AttributeError: 'numpy.ndarray' object has no attribute 'rand'



```python
pd.options.display.float_format = '{:.2f}'.format
```


```python
plt.scatter(temp['Di_Predict'], temp['Di_Real'])
```




    <matplotlib.collections.PathCollection at 0x7f3af2422cf8>




![png](Function_Predict_files/Function_Predict_40_1.png)


#PRACTICE FUNCTION_ 등락률


```python
df
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
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Target_YMD</th>
      <th>Target_High</th>
      <th>Target_Low</th>
      <th>Target_Close</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2000-01-04</th>
      <td>6110.00</td>
      <td>5660.00</td>
      <td>6110.00</td>
      <td>2000-01-18</td>
      <td>6160.00</td>
      <td>5980.00</td>
      <td>6100.00</td>
    </tr>
    <tr>
      <th>2000-01-05</th>
      <td>6060.00</td>
      <td>5520.00</td>
      <td>5580.00</td>
      <td>2000-01-19</td>
      <td>6040.00</td>
      <td>5960.00</td>
      <td>5960.00</td>
    </tr>
    <tr>
      <th>2000-01-06</th>
      <td>5780.00</td>
      <td>5580.00</td>
      <td>5620.00</td>
      <td>2000-01-20</td>
      <td>6040.00</td>
      <td>5820.00</td>
      <td>6040.00</td>
    </tr>
    <tr>
      <th>2000-01-07</th>
      <td>5670.00</td>
      <td>5360.00</td>
      <td>5540.00</td>
      <td>2000-01-21</td>
      <td>5980.00</td>
      <td>5880.00</td>
      <td>5880.00</td>
    </tr>
    <tr>
      <th>2000-01-10</th>
      <td>5770.00</td>
      <td>5580.00</td>
      <td>5770.00</td>
      <td>2000-01-24</td>
      <td>5900.00</td>
      <td>5700.00</td>
      <td>5700.00</td>
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
      <th>2020-06-24</th>
      <td>53900.00</td>
      <td>51600.00</td>
      <td>52900.00</td>
      <td>2020-07-08</td>
      <td>53900.00</td>
      <td>52900.00</td>
      <td>53000.00</td>
    </tr>
    <tr>
      <th>2020-06-25</th>
      <td>53000.00</td>
      <td>51900.00</td>
      <td>51900.00</td>
      <td>2020-07-09</td>
      <td>53600.00</td>
      <td>52800.00</td>
      <td>52800.00</td>
    </tr>
    <tr>
      <th>2020-06-26</th>
      <td>53900.00</td>
      <td>52200.00</td>
      <td>53300.00</td>
      <td>2020-07-10</td>
      <td>53200.00</td>
      <td>52300.00</td>
      <td>52700.00</td>
    </tr>
    <tr>
      <th>2020-06-29</th>
      <td>53200.00</td>
      <td>52000.00</td>
      <td>52400.00</td>
      <td>2020-07-13</td>
      <td>53800.00</td>
      <td>53100.00</td>
      <td>53400.00</td>
    </tr>
    <tr>
      <th>2020-06-30</th>
      <td>53900.00</td>
      <td>52800.00</td>
      <td>52800.00</td>
      <td>2020-07-14</td>
      <td>53800.00</td>
      <td>53200.00</td>
      <td>53800.00</td>
    </tr>
  </tbody>
</table>
<p>5144 rows × 7 columns</p>
</div>




```python
df = pdr.get_data_yahoo('BA', '2000-01-01')
```


```python
df['Di'] = 0
lennum = int(len(df))
for i in range(lennum -1):
  df['Di'].iloc[i+1]= df['Close'][i+1] - df['Close'][i]
```

    /usr/local/lib/python3.6/dist-packages/pandas/core/indexing.py:671: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      self._setitem_with_indexer(indexer, value)
    


```python
df = pd.DataFrame(df)
df
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
      <th>High</th>
      <th>Low</th>
      <th>Open</th>
      <th>Close</th>
      <th>Volume</th>
      <th>Adj Close</th>
      <th>Di</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2000-01-03</th>
      <td>41.69</td>
      <td>39.81</td>
      <td>41.44</td>
      <td>40.19</td>
      <td>2638200.00</td>
      <td>25.74</td>
      <td>0.00</td>
    </tr>
    <tr>
      <th>2000-01-04</th>
      <td>41.12</td>
      <td>39.75</td>
      <td>40.19</td>
      <td>40.12</td>
      <td>3592100.00</td>
      <td>25.70</td>
      <td>-0.06</td>
    </tr>
    <tr>
      <th>2000-01-05</th>
      <td>43.31</td>
      <td>41.38</td>
      <td>41.38</td>
      <td>42.62</td>
      <td>7631700.00</td>
      <td>27.30</td>
      <td>2.50</td>
    </tr>
    <tr>
      <th>2000-01-06</th>
      <td>43.44</td>
      <td>41.12</td>
      <td>42.62</td>
      <td>43.06</td>
      <td>4922200.00</td>
      <td>27.58</td>
      <td>0.44</td>
    </tr>
    <tr>
      <th>2000-01-07</th>
      <td>44.88</td>
      <td>43.69</td>
      <td>43.69</td>
      <td>44.31</td>
      <td>6008300.00</td>
      <td>28.38</td>
      <td>1.25</td>
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
      <th>2020-07-07</th>
      <td>185.07</td>
      <td>178.65</td>
      <td>185.07</td>
      <td>178.88</td>
      <td>37105300.00</td>
      <td>178.88</td>
      <td>-9.03</td>
    </tr>
    <tr>
      <th>2020-07-08</th>
      <td>181.58</td>
      <td>175.51</td>
      <td>179.05</td>
      <td>180.08</td>
      <td>38116200.00</td>
      <td>180.08</td>
      <td>1.20</td>
    </tr>
    <tr>
      <th>2020-07-09</th>
      <td>180.75</td>
      <td>172.81</td>
      <td>179.67</td>
      <td>173.28</td>
      <td>33514500.00</td>
      <td>173.28</td>
      <td>-6.80</td>
    </tr>
    <tr>
      <th>2020-07-10</th>
      <td>179.33</td>
      <td>169.75</td>
      <td>171.70</td>
      <td>178.44</td>
      <td>40955600.00</td>
      <td>178.44</td>
      <td>5.16</td>
    </tr>
    <tr>
      <th>2020-07-13</th>
      <td>183.25</td>
      <td>174.36</td>
      <td>180.20</td>
      <td>175.65</td>
      <td>43001100.00</td>
      <td>175.65</td>
      <td>-2.79</td>
    </tr>
  </tbody>
</table>
<p>5164 rows × 7 columns</p>
</div>




```python
df['Target_Di'] = 0
df['Target_Date'] = 0
num = int(len(df))
for i in range(num-10):
  df['Target_Di'].iloc[i] = df['Di'].iloc[i+10]
  df['Target_Date'].iloc[i] = str(df.index[i+10]).split(' ')[0]
df
```

    /usr/local/lib/python3.6/dist-packages/pandas/core/indexing.py:671: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      self._setitem_with_indexer(indexer, value)
    




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
      <th>High</th>
      <th>Low</th>
      <th>Open</th>
      <th>Close</th>
      <th>Volume</th>
      <th>Adj Close</th>
      <th>Di</th>
      <th>Target_Di</th>
      <th>Target_Date</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2000-01-03</th>
      <td>41.69</td>
      <td>39.81</td>
      <td>41.44</td>
      <td>40.19</td>
      <td>2638200.00</td>
      <td>25.74</td>
      <td>0.00</td>
      <td>1.00</td>
      <td>2000-01-18</td>
    </tr>
    <tr>
      <th>2000-01-04</th>
      <td>41.12</td>
      <td>39.75</td>
      <td>40.19</td>
      <td>40.12</td>
      <td>3592100.00</td>
      <td>25.70</td>
      <td>-0.06</td>
      <td>2.62</td>
      <td>2000-01-19</td>
    </tr>
    <tr>
      <th>2000-01-05</th>
      <td>43.31</td>
      <td>41.38</td>
      <td>41.38</td>
      <td>42.62</td>
      <td>7631700.00</td>
      <td>27.30</td>
      <td>2.50</td>
      <td>-1.12</td>
      <td>2000-01-20</td>
    </tr>
    <tr>
      <th>2000-01-06</th>
      <td>43.44</td>
      <td>41.12</td>
      <td>42.62</td>
      <td>43.06</td>
      <td>4922200.00</td>
      <td>27.58</td>
      <td>0.44</td>
      <td>-0.81</td>
      <td>2000-01-21</td>
    </tr>
    <tr>
      <th>2000-01-07</th>
      <td>44.88</td>
      <td>43.69</td>
      <td>43.69</td>
      <td>44.31</td>
      <td>6008300.00</td>
      <td>28.38</td>
      <td>1.25</td>
      <td>-1.38</td>
      <td>2000-01-24</td>
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
    </tr>
    <tr>
      <th>2020-07-07</th>
      <td>185.07</td>
      <td>178.65</td>
      <td>185.07</td>
      <td>178.88</td>
      <td>37105300.00</td>
      <td>178.88</td>
      <td>-9.03</td>
      <td>0.00</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2020-07-08</th>
      <td>181.58</td>
      <td>175.51</td>
      <td>179.05</td>
      <td>180.08</td>
      <td>38116200.00</td>
      <td>180.08</td>
      <td>1.20</td>
      <td>0.00</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2020-07-09</th>
      <td>180.75</td>
      <td>172.81</td>
      <td>179.67</td>
      <td>173.28</td>
      <td>33514500.00</td>
      <td>173.28</td>
      <td>-6.80</td>
      <td>0.00</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2020-07-10</th>
      <td>179.33</td>
      <td>169.75</td>
      <td>171.70</td>
      <td>178.44</td>
      <td>40955600.00</td>
      <td>178.44</td>
      <td>5.16</td>
      <td>0.00</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2020-07-13</th>
      <td>183.25</td>
      <td>174.36</td>
      <td>180.20</td>
      <td>175.65</td>
      <td>43001100.00</td>
      <td>175.65</td>
      <td>-2.79</td>
      <td>0.00</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>5164 rows × 9 columns</p>
</div>




```python
sc_xx = MinMaxScaler()
df['Di'] = sc_xx.fit_transform(np.array(df['Di']).reshape(-1,1))
df
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
      <th>High</th>
      <th>Low</th>
      <th>Open</th>
      <th>Close</th>
      <th>Volume</th>
      <th>Adj Close</th>
      <th>Di</th>
      <th>Target_Di</th>
      <th>Target_Date</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2000-01-03</th>
      <td>41.69</td>
      <td>39.81</td>
      <td>41.44</td>
      <td>40.19</td>
      <td>2638200.00</td>
      <td>25.74</td>
      <td>0.57</td>
      <td>1.00</td>
      <td>2000-01-18</td>
    </tr>
    <tr>
      <th>2000-01-04</th>
      <td>41.12</td>
      <td>39.75</td>
      <td>40.19</td>
      <td>40.12</td>
      <td>3592100.00</td>
      <td>25.70</td>
      <td>0.57</td>
      <td>2.62</td>
      <td>2000-01-19</td>
    </tr>
    <tr>
      <th>2000-01-05</th>
      <td>43.31</td>
      <td>41.38</td>
      <td>41.38</td>
      <td>42.62</td>
      <td>7631700.00</td>
      <td>27.30</td>
      <td>0.61</td>
      <td>-1.12</td>
      <td>2000-01-20</td>
    </tr>
    <tr>
      <th>2000-01-06</th>
      <td>43.44</td>
      <td>41.12</td>
      <td>42.62</td>
      <td>43.06</td>
      <td>4922200.00</td>
      <td>27.58</td>
      <td>0.58</td>
      <td>-0.81</td>
      <td>2000-01-21</td>
    </tr>
    <tr>
      <th>2000-01-07</th>
      <td>44.88</td>
      <td>43.69</td>
      <td>43.69</td>
      <td>44.31</td>
      <td>6008300.00</td>
      <td>28.38</td>
      <td>0.59</td>
      <td>-1.38</td>
      <td>2000-01-24</td>
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
    </tr>
    <tr>
      <th>2020-07-07</th>
      <td>185.07</td>
      <td>178.65</td>
      <td>185.07</td>
      <td>178.88</td>
      <td>37105300.00</td>
      <td>178.88</td>
      <td>0.45</td>
      <td>0.00</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2020-07-08</th>
      <td>181.58</td>
      <td>175.51</td>
      <td>179.05</td>
      <td>180.08</td>
      <td>38116200.00</td>
      <td>180.08</td>
      <td>0.59</td>
      <td>0.00</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2020-07-09</th>
      <td>180.75</td>
      <td>172.81</td>
      <td>179.67</td>
      <td>173.28</td>
      <td>33514500.00</td>
      <td>173.28</td>
      <td>0.48</td>
      <td>0.00</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2020-07-10</th>
      <td>179.33</td>
      <td>169.75</td>
      <td>171.70</td>
      <td>178.44</td>
      <td>40955600.00</td>
      <td>178.44</td>
      <td>0.65</td>
      <td>0.00</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2020-07-13</th>
      <td>183.25</td>
      <td>174.36</td>
      <td>180.20</td>
      <td>175.65</td>
      <td>43001100.00</td>
      <td>175.65</td>
      <td>0.54</td>
      <td>0.00</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>5164 rows × 9 columns</p>
</div>




```python
#Time step 결정
lst = []
lst.append('Di')
for step in range(1,6):
    df['Di_{}'.format(step)] = df['Di'].shift(step)
    lst.append('Di_'+str(step))

df= df.dropna()
x_data = df[lst][:-10]

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
      <th>Di</th>
      <th>Di_1</th>
      <th>Di_2</th>
      <th>Di_3</th>
      <th>Di_4</th>
      <th>Di_5</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2000-01-10</th>
      <td>0.57</td>
      <td>0.59</td>
      <td>0.58</td>
      <td>0.61</td>
      <td>0.57</td>
      <td>0.57</td>
    </tr>
    <tr>
      <th>2000-01-11</th>
      <td>0.56</td>
      <td>0.57</td>
      <td>0.59</td>
      <td>0.58</td>
      <td>0.61</td>
      <td>0.57</td>
    </tr>
    <tr>
      <th>2000-01-12</th>
      <td>0.58</td>
      <td>0.56</td>
      <td>0.57</td>
      <td>0.59</td>
      <td>0.58</td>
      <td>0.61</td>
    </tr>
    <tr>
      <th>2000-01-13</th>
      <td>0.57</td>
      <td>0.58</td>
      <td>0.56</td>
      <td>0.57</td>
      <td>0.59</td>
      <td>0.58</td>
    </tr>
    <tr>
      <th>2000-01-14</th>
      <td>0.60</td>
      <td>0.57</td>
      <td>0.58</td>
      <td>0.56</td>
      <td>0.57</td>
      <td>0.59</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2020-06-22</th>
      <td>0.60</td>
      <td>0.50</td>
      <td>0.57</td>
      <td>0.50</td>
      <td>0.67</td>
      <td>0.59</td>
    </tr>
    <tr>
      <th>2020-06-23</th>
      <td>0.57</td>
      <td>0.60</td>
      <td>0.50</td>
      <td>0.57</td>
      <td>0.50</td>
      <td>0.67</td>
    </tr>
    <tr>
      <th>2020-06-24</th>
      <td>0.42</td>
      <td>0.57</td>
      <td>0.60</td>
      <td>0.50</td>
      <td>0.57</td>
      <td>0.50</td>
    </tr>
    <tr>
      <th>2020-06-25</th>
      <td>0.55</td>
      <td>0.42</td>
      <td>0.57</td>
      <td>0.60</td>
      <td>0.50</td>
      <td>0.57</td>
    </tr>
    <tr>
      <th>2020-06-26</th>
      <td>0.51</td>
      <td>0.55</td>
      <td>0.42</td>
      <td>0.57</td>
      <td>0.60</td>
      <td>0.50</td>
    </tr>
  </tbody>
</table>
<p>5149 rows × 6 columns</p>
</div>




```python
y_data = df[:-10]['Target_Di']
sc_yy = MinMaxScaler()
y_data = sc_yy.fit_transform(np.array(y_data).reshape(-1,1))
y_data
```




    array([[0.59338174],
           [0.58139216],
           [0.55912578],
           ...,
           [0.48136469],
           [0.64524533],
           [0.53631119]])




```python
xx_train = x_data[:-1000]
xx_test = x_data[-1000:]
yy_train = y_data[:-1000]
yy_test = y_data[-1000:]

xx_train_t = np.array(xx_train).reshape(len(xx_train), 1, 6)
xx_test_t = np.array(xx_test).reshape(len(xx_test), 1, 6)
```


```python
model_di = tf.keras.models.Sequential()
model_di.add(LSTM(10, input_shape=(xx_train_t.shape[1], xx_train_t.shape[2])))
model_di.add(Dense(1))
model_di.compile(loss='mean_squared_error', optimizer = 'adam')
model_di.fit(xx_train_t, yy_train, epochs=10, batch_size =1, verbose =1)
```

    Epoch 1/10
    4149/4149 [==============================] - 6s 1ms/step - loss: 0.0032
    Epoch 2/10
    4149/4149 [==============================] - 6s 1ms/step - loss: 4.0735e-04
    Epoch 3/10
    4149/4149 [==============================] - 6s 1ms/step - loss: 3.9135e-04
    Epoch 4/10
    4149/4149 [==============================] - 6s 1ms/step - loss: 3.8053e-04
    Epoch 5/10
    4149/4149 [==============================] - 6s 1ms/step - loss: 3.7221e-04
    Epoch 6/10
    4149/4149 [==============================] - 6s 1ms/step - loss: 3.6414e-04
    Epoch 7/10
    4149/4149 [==============================] - 6s 1ms/step - loss: 3.5860e-04
    Epoch 8/10
    4149/4149 [==============================] - 6s 1ms/step - loss: 3.5397e-04
    Epoch 9/10
    4149/4149 [==============================] - 6s 1ms/step - loss: 3.5506e-04
    Epoch 10/10
    4149/4149 [==============================] - 6s 1ms/step - loss: 3.5042e-04
    




    <tensorflow.python.keras.callbacks.History at 0x7f3aec2d60f0>




```python
pred = model_di.predict(xx_test_t)
pred = np.exp(sc_yy.inverse_transform(pred))
real = sc_yy.inverse_transform(yy_test)
rs = pd.DataFrame(pred, columns=['Predcit'])
rs['Real']= real
rs['Error'] = 0 
rs['Error'] = rs['Predcit'] - rs['Real']
```


```python
plt.figure(figsize=(20,12))
plt.grid()
plt.scatter(rs['Predcit'], rs['Real'])
plt.show()
```


![png](Function_Predict_files/Function_Predict_53_0.png)



```python
for i in range(10):
  print(str(0+(i*100))+'부터', str(100+(i*100))+'까지')
  plt.figure(figsize=(20,12))
  plt.grid()
  plt.plot(range(100), rs['Real'][0+(i*100):100+(i*100)], label = 'Real', alpha = 0.5 ,linewidth=3.0)
  plt.plot(range(100), rs['Predcit'][0+(i*100):100+(i*100)], label = 'Predict', alpha = 0.9, linewidth=3.0)
  plt.legend(loc='best')
  plt.show()
```

    0부터 100까지
    


![png](Function_Predict_files/Function_Predict_54_1.png)


    100부터 200까지
    


![png](Function_Predict_files/Function_Predict_54_3.png)


    200부터 300까지
    


![png](Function_Predict_files/Function_Predict_54_5.png)


    300부터 400까지
    


![png](Function_Predict_files/Function_Predict_54_7.png)


    400부터 500까지
    


![png](Function_Predict_files/Function_Predict_54_9.png)


    500부터 600까지
    


![png](Function_Predict_files/Function_Predict_54_11.png)


    600부터 700까지
    


![png](Function_Predict_files/Function_Predict_54_13.png)


    700부터 800까지
    


![png](Function_Predict_files/Function_Predict_54_15.png)


    800부터 900까지
    


![png](Function_Predict_files/Function_Predict_54_17.png)


    900부터 1000까지
    


![png](Function_Predict_files/Function_Predict_54_19.png)



```python
rs
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
      <th>Predcit</th>
      <th>Real</th>
      <th>Error</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1.29</td>
      <td>-0.06</td>
      <td>1.35</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.42</td>
      <td>-0.53</td>
      <td>1.95</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1.37</td>
      <td>1.91</td>
      <td>-0.54</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1.47</td>
      <td>1.11</td>
      <td>0.36</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1.35</td>
      <td>-2.95</td>
      <td>4.30</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>995</th>
      <td>1.04</td>
      <td>-9.03</td>
      <td>10.07</td>
    </tr>
    <tr>
      <th>996</th>
      <td>1.20</td>
      <td>1.20</td>
      <td>0.00</td>
    </tr>
    <tr>
      <th>997</th>
      <td>0.78</td>
      <td>-6.80</td>
      <td>7.58</td>
    </tr>
    <tr>
      <th>998</th>
      <td>1.22</td>
      <td>5.16</td>
      <td>-3.94</td>
    </tr>
    <tr>
      <th>999</th>
      <td>0.78</td>
      <td>-2.79</td>
      <td>3.57</td>
    </tr>
  </tbody>
</table>
<p>1000 rows × 3 columns</p>
</div>




```python

```




    <matplotlib.axes._subplots.AxesSubplot at 0x7f3aedb46b70>




![png](Function_Predict_files/Function_Predict_56_1.png)



```python
datetime
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-28-2b93382a42f4> in <module>()
    ----> 1 datetime
    

    NameError: name 'datetime' is not defined



```python
import datetime
```


```python
datetime.timedelta(days=1)
```




    datetime.timedelta(1)




```python

```
