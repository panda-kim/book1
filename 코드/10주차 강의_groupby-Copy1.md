# 10주차

**groupby**



<br>

본 실습은 **pandas 1.5.3**버전이며, 주피터 노트북으로 진행됩니다.

---


### 목차 

[**언피벗(unpivot)**](#chapter1)
* [1. 언피벗(unpivot)을 하는 이유](#chapter2)
* [2. stack + reset_index로 언피벗하기](#chapter3)
* [3. melt 함수로 언피벗하기](#chapter4)
* [4. 언피벗 후 다시 피벗테이블 만들기](#chapter5)
* [**예제** 예제 마트 매출 집계 데이터로 특정 데이터만 집계하기](#chapter6)

[**groupby**](#chapter7)
* [1. groupby가 필요한 이유 ](#chapter8)
* [2. groupby 함수](#chapter9)
    * [2-1. groupby로 feature engineering 하기](#chapter10)
    * [2-2. groupby로 집계함수 적용하기](#chapter11)
* [**예제** 채권 데이터로 여러가지 그룹 집계하기](#chapter12)

---
<br>


## 1. 언피벗(unpivot) <a class="anchor" id="chapter1"></a>

### 1. 언피벗(unpivot)을 하는 이유 <a class="anchor" id="chapter2"></a>

**원시(raw) 데이터와 피벗 테이블(pivot table)의 장단점**

원시 데이터(raw data)

- 데이터를 쌓고 관리하기 쉽다

- 데이터의 의미 파악이 어렵다

피벗 테이블(pivot table)

- 데이터 의미 파악이 쉽다

- 데이터를 쌓고 관리하기 어렵다

→ 그래서 원시(raw) 데이터로 데이터를 쌓고 관리하며, 피벗 테이블로는 요약을 해야 한다

그럼에도 실무에서는 피벗 테이블로 데이터를 관리하는 경우가 많다.

---

<br>

**피벗 테이블로 데이터를 관리하는 경우의 문제점**

<img src=https://i.ibb.co/mhJ9kbR/image.png, width=600>

위 그림은 왼쪽과 같은 피벗테이블로 데이터를 관리하고 있는 상황이다.

피벗 테이블로 데이터를 관리하면 우측과 같이 새로운 피벗테이블 만드는 것이 어렵다

위 예시는 단순한 구조라서 그나마 가능하겠지만 복잡한 피벗테이블을 다른 복잡한 피벗테이블로 전환은 정말 쉽지 않다.


물론 왼쪽 피벗테이블을 만드는 원시 데이터가 있었다면 우측의 피벗테이블도 쉽게 만들 수 있을 것이다

다만 데이터를 피벗 테이블로 관리하면 원시 데이터는 존재하지 않는 경우가 대부분이다.

이럴 때는 어떻게 해야 할까?

→ 그 때 필요한 것이 언피벗(unpivot)이다.


---
<br>

**언피벗(unpivot)이란?**

<img src=https://i.ibb.co/CKZqNyd/image.png, width=600>

언피벗은 피벗 테이블을 마치 원시(raw) 데이터처럼 열은 특성(feature), 행은 개별 데이터인 테이블로 만드는 것이다.

언피벗하면 values는 하나의 열이 되고 값에 맞춰 index와 columns가 새로운 열이 된다

언피벗을 하면 원시 데이터와 같은 특징을 가지기 때문에 새로운 방향으로 피벗 테이블을 만들기 쉽고 데이터를 쌓기도 쉽다



---
<br>

**언피벗을 하는 방법**

판다스에서는 두가지 방법으로 언피벗이 가능하다

1. `stack` + `reset_index` : `stack`은 그 자체로 유용한 함수라 unpivot을 위해 함수를 추가적으로 익히지 않아도 된다


2. `melt` : `melt`는 unpivot 전용 함수라 편리하다

이 두가지 방법을 알아보자

---

<br>




### 2. stack + reset_index로 언피벗하기 <a class="anchor" id="chapter3"></a>

**새로운 함수 소개**

> pandas stack & unstack

<img src=https://i.ibb.co/LZNRJCN/image.png, width=600>

stack은 columns를 index로 보내고 unstack은 index를 columns로 보낸다

> stack 함수의 인자(parameter)들

**level** (level의 레이블 혹은 로케이션, 또는 그것들의 리스트 / 기본값은  -1)

index로 보낼 columns의 level을 지정하는 인자. 기본값은 -1이라서 맨 마지막 columns를 보낸다

 

**dropna** (인수는 bool / 기본값은 True)

값이 NaN인 행은 생성하지 않고 삭제하는 인자. 기본값은 True


---

<br>

> unstack 함수의 인자(parameter)들

**level** (level의 레이블 혹은 로케이션, 또는 그것들의 리스트 / 기본값은  -1)

columns로 보낼 index의 level을 지정하는 인자. 기본값은 -1이라서 맨 마지막 index를 보낸다

 

 

**fill_value**

NaN을 대체할 값을 지정하는 인자


---

<br>


```python
# 실습 준비 코드
import pandas as pd
data1 = [[10, 20, 30, 40], [15, 25, 35, 45]]
data2 = [[10, 30], [20, 40], [15, 35]]
col1 = pd.MultiIndex.from_product([['남', '여'], ['A반', 'B반']])
df1 = pd.DataFrame(data1, index=['1학년', '2학년'], columns=col1)
df2 = pd.DataFrame(data2, index=list('ABC'), columns=['남', '여'])
```


```python
# 실습에 쓰일 df1출력
df1
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead tr th {
        text-align: left;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th colspan="2" halign="left">남</th>
      <th colspan="2" halign="left">여</th>
    </tr>
    <tr>
      <th></th>
      <th>A반</th>
      <th>B반</th>
      <th>A반</th>
      <th>B반</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1학년</th>
      <td>10</td>
      <td>20</td>
      <td>30</td>
      <td>40</td>
    </tr>
    <tr>
      <th>2학년</th>
      <td>15</td>
      <td>25</td>
      <td>35</td>
      <td>45</td>
    </tr>
  </tbody>
</table>
</div>




```python
# stack으로 df1의 구조 바꾸기 - 반의 구분을 index로 보내기
df1.stack()
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
      <th></th>
      <th>남</th>
      <th>여</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="2" valign="top">1학년</th>
      <th>A반</th>
      <td>10</td>
      <td>30</td>
    </tr>
    <tr>
      <th>B반</th>
      <td>20</td>
      <td>40</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">2학년</th>
      <th>A반</th>
      <td>15</td>
      <td>35</td>
    </tr>
    <tr>
      <th>B반</th>
      <td>25</td>
      <td>45</td>
    </tr>
  </tbody>
</table>
</div>




```python
# stack으로 level을 0으로 지정해 성별의 구분을 index로 보내기
df1.stack(0)
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
      <th></th>
      <th>A반</th>
      <th>B반</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="2" valign="top">1학년</th>
      <th>남</th>
      <td>10</td>
      <td>20</td>
    </tr>
    <tr>
      <th>여</th>
      <td>30</td>
      <td>40</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">2학년</th>
      <th>남</th>
      <td>15</td>
      <td>25</td>
    </tr>
    <tr>
      <th>여</th>
      <td>35</td>
      <td>45</td>
    </tr>
  </tbody>
</table>
</div>




```python
# level을 복수로 지정해 반과 성별을 모두 index로 보내기
df1.stack([0, 1])
```




    1학년  남  A반    10
            B반    20
         여  A반    30
            B반    40
    2학년  남  A반    15
            B반    25
         여  A반    35
            B반    45
    dtype: int64



**stack과 언피벗(unpivot)과의 관계**

<img src=https://i.ibb.co/TTSh45t/image.png, width=600>

`stack`으로 모든 columns를 index로 보내고 `reset_index`를 하면 언피벗(unpivot)한 결과가 된다.

단 이 경우 `reset_index` 적용 후에 열 이름은 바꿔줘야 한다

---
<br>
    


```python
# 실습에 쓰일 df2
df2
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
      <th>남</th>
      <th>여</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>A</th>
      <td>10</td>
      <td>30</td>
    </tr>
    <tr>
      <th>B</th>
      <td>20</td>
      <td>40</td>
    </tr>
    <tr>
      <th>C</th>
      <td>15</td>
      <td>35</td>
    </tr>
  </tbody>
</table>
</div>




```python
# df2를 stack + reset_index로 unpivot 하기
df2.stack().reset_index()
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
      <th>level_0</th>
      <th>level_1</th>
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>남</td>
      <td>10</td>
    </tr>
    <tr>
      <th>1</th>
      <td>A</td>
      <td>여</td>
      <td>30</td>
    </tr>
    <tr>
      <th>2</th>
      <td>B</td>
      <td>남</td>
      <td>20</td>
    </tr>
    <tr>
      <th>3</th>
      <td>B</td>
      <td>여</td>
      <td>40</td>
    </tr>
    <tr>
      <th>4</th>
      <td>C</td>
      <td>남</td>
      <td>15</td>
    </tr>
    <tr>
      <th>5</th>
      <td>C</td>
      <td>여</td>
      <td>35</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 위 결과의 열이름을 반, 성별, 인원수로 지정하기
df2.stack().reset_index().set_axis(['반', '성별', '인원수'], axis=1)
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
      <th>반</th>
      <th>성별</th>
      <th>인원수</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>남</td>
      <td>10</td>
    </tr>
    <tr>
      <th>1</th>
      <td>A</td>
      <td>여</td>
      <td>30</td>
    </tr>
    <tr>
      <th>2</th>
      <td>B</td>
      <td>남</td>
      <td>20</td>
    </tr>
    <tr>
      <th>3</th>
      <td>B</td>
      <td>여</td>
      <td>40</td>
    </tr>
    <tr>
      <th>4</th>
      <td>C</td>
      <td>남</td>
      <td>15</td>
    </tr>
    <tr>
      <th>5</th>
      <td>C</td>
      <td>여</td>
      <td>35</td>
    </tr>
  </tbody>
</table>
</div>



### 3. melt 함수로 언피벗하기 <a class="anchor" id="chapter4"></a>


**새로운 함수 소개**

> pandas melt


<img src=https://i.ibb.co/vBMZHB4/image.png, width=600>


데이터 프레임을 unpivot 하는 함수


> melt 함수의 인자(parameter)들

**id_vars** (열의 레이블 혹은 그것들의 리스트)

index처럼 unpivot 하지 않을 열을 지정하는 인자

**value_vars** (열의 레이블 혹은 그것들의 리스트)

unpivot될 열들을 지정하는 인자

기본값으로 id_vars로 지정되지 않은 모든 열을 value_vars로 지정해 unpivot한다

**var_name** (scalar)

unpivot 후 'variable'열의 레이블을 지정하는 인자

**value_name** (scalar/ 기본값은 'value')

unpivot 후 'value'열의 레이블을 지정하는 인자

---
**vars의 의미**

variables의 약자이지만 여기서는 column을 의미한다

id_vars : index처럼 처리할 column들을 지정하는 인자

value_vars : values 처럼 처리할 column들을 지정하는 인자

var_name : 언피벗 하기 전에 column들의 이름들이 언피벗후에 들어가게 되는 열이 있는데 그 열의 이름을 지정하는 인자 

(기본 값은 variable) 

---

<br>


```python
# 실습 준비 코드
import pandas as pd
data = [['A', 10, 30], ['B', 20, 40], ['C', 15, 35]]
df = pd.DataFrame(data, columns=['반', '남', '여'])
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
      <th>반</th>
      <th>남</th>
      <th>여</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>10</td>
      <td>30</td>
    </tr>
    <tr>
      <th>1</th>
      <td>B</td>
      <td>20</td>
      <td>40</td>
    </tr>
    <tr>
      <th>2</th>
      <td>C</td>
      <td>15</td>
      <td>35</td>
    </tr>
  </tbody>
</table>
</div>




```python
# melt로 반은 특성으로 두고 남,여 열을 언피벗
df.melt('반')
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
      <th>반</th>
      <th>variable</th>
      <th>value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>남</td>
      <td>10</td>
    </tr>
    <tr>
      <th>1</th>
      <td>B</td>
      <td>남</td>
      <td>20</td>
    </tr>
    <tr>
      <th>2</th>
      <td>C</td>
      <td>남</td>
      <td>15</td>
    </tr>
    <tr>
      <th>3</th>
      <td>A</td>
      <td>여</td>
      <td>30</td>
    </tr>
    <tr>
      <th>4</th>
      <td>B</td>
      <td>여</td>
      <td>40</td>
    </tr>
    <tr>
      <th>5</th>
      <td>C</td>
      <td>여</td>
      <td>35</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 언피벗 후 열 이름을 반, 성별, 인원수로 바꾸기 (set_index 함수 사용)
df.melt('반').set_axis(['반', '성별', '인원수'], axis=1)
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
      <th>반</th>
      <th>성별</th>
      <th>인원수</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>남</td>
      <td>10</td>
    </tr>
    <tr>
      <th>1</th>
      <td>B</td>
      <td>남</td>
      <td>20</td>
    </tr>
    <tr>
      <th>2</th>
      <td>C</td>
      <td>남</td>
      <td>15</td>
    </tr>
    <tr>
      <th>3</th>
      <td>A</td>
      <td>여</td>
      <td>30</td>
    </tr>
    <tr>
      <th>4</th>
      <td>B</td>
      <td>여</td>
      <td>40</td>
    </tr>
    <tr>
      <th>5</th>
      <td>C</td>
      <td>여</td>
      <td>35</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 언피벗 후 열 이름을 반, 성별, 인원수로 바꾸기 (melt 함수의 인자 사용)
df.melt('반', var_name='성별', value_name='인원수')
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
      <th>반</th>
      <th>성별</th>
      <th>인원수</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>남</td>
      <td>10</td>
    </tr>
    <tr>
      <th>1</th>
      <td>B</td>
      <td>남</td>
      <td>20</td>
    </tr>
    <tr>
      <th>2</th>
      <td>C</td>
      <td>남</td>
      <td>15</td>
    </tr>
    <tr>
      <th>3</th>
      <td>A</td>
      <td>여</td>
      <td>30</td>
    </tr>
    <tr>
      <th>4</th>
      <td>B</td>
      <td>여</td>
      <td>40</td>
    </tr>
    <tr>
      <th>5</th>
      <td>C</td>
      <td>여</td>
      <td>35</td>
    </tr>
  </tbody>
</table>
</div>



## 4. 언피벗 후 다시 피벗테이블 만들기<a class="anchor" id="chapter6"></a>

**언피벗 후 다시 피벗테이블 만들기**

1. index에 필요한 데이터가 있다면 `stack` + `reset_index`로 언피벗하고

   index가 그냥 RangeIndex이거나 없어도 되는 데이터라면 `melt`로 언피벗한다
   
<br>   

2. 언피벗 후 원하는 집계 방식에 맞춰 `pivot_table` 혹은 `crosstab` 혹은 `groupby`등을 이용해 집계한다

---

<br>


```python
# 실습 준비 코드
data3 = {'1등': {'1회차': '김판다', '2회차': '박효신', '3회차': '김판다'}, 
         '2등': {'1회차': '권보아', '2회차': '권보아', '3회차': '박효신'}, 
         '3등': {'1회차': '박효신', '2회차': '강승주', '3회차': '김범수'}}
df3 = pd.DataFrame(data3)
df3
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
      <th>1등</th>
      <th>2등</th>
      <th>3등</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1회차</th>
      <td>김판다</td>
      <td>권보아</td>
      <td>박효신</td>
    </tr>
    <tr>
      <th>2회차</th>
      <td>박효신</td>
      <td>권보아</td>
      <td>강승주</td>
    </tr>
    <tr>
      <th>3회차</th>
      <td>김판다</td>
      <td>박효신</td>
      <td>김범수</td>
    </tr>
  </tbody>
</table>
</div>



이 경우 index가 필요한 데이터이기 때문에 stack + reset_index가 효율적이다


```python
# df3를 stack + reset_index로 언피벗하고 각 열의 이름을 회차, 등수, 이름으로 지정하기
df3.stack().reset_index().set_axis(['회차', '등수', '이름'], axis=1)
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
      <th>회차</th>
      <th>등수</th>
      <th>이름</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1회차</td>
      <td>1등</td>
      <td>김판다</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1회차</td>
      <td>2등</td>
      <td>권보아</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1회차</td>
      <td>3등</td>
      <td>박효신</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2회차</td>
      <td>1등</td>
      <td>박효신</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2회차</td>
      <td>2등</td>
      <td>권보아</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2회차</td>
      <td>3등</td>
      <td>강승주</td>
    </tr>
    <tr>
      <th>6</th>
      <td>3회차</td>
      <td>1등</td>
      <td>김판다</td>
    </tr>
    <tr>
      <th>7</th>
      <td>3회차</td>
      <td>2등</td>
      <td>박효신</td>
    </tr>
    <tr>
      <th>8</th>
      <td>3회차</td>
      <td>3등</td>
      <td>김범수</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 언피벗 결과를 df4로 지정한 다음 각 사람별 1등, 2등, 3등 횟수 구하기 (pivot_table 함수)
df4 = df3.stack().reset_index().set_axis(['회차', '등수', '이름'], axis=1)
df4.pivot_table('회차', index='이름', columns='등수', aggfunc='count').fillna(0).astype('int')
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
      <th>등수</th>
      <th>1등</th>
      <th>2등</th>
      <th>3등</th>
    </tr>
    <tr>
      <th>이름</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>강승주</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>권보아</th>
      <td>0</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>김범수</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>김판다</th>
      <td>2</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>박효신</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 언피벗 결과를 df4로 지정한 다음  각 사람별 1등, 2등, 3등 횟수 구하기 (crosstab 함수)
df4 = df3.stack().reset_index().set_axis(['회차', '등수', '이름'], axis=1)
pd.crosstab(df4['이름'], df4['등수'])
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
      <th>등수</th>
      <th>1등</th>
      <th>2등</th>
      <th>3등</th>
    </tr>
    <tr>
      <th>이름</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>강승주</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>권보아</th>
      <td>0</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>김범수</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>김판다</th>
      <td>2</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>박효신</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



**예제** 마트 매출 집계 데이터로 특정 데이터만 집계하기 <a class="anchor" id="chapter6"></a>

파일명 : e_09_market.xlsx


<img src=https://i.ibb.co/R08ntcf/image.png, width=600>

<img src=https://i.ibb.co/bj4MgJ8/image.png, width=600>

---
<br>


```python
# 첫번째 시트만 데이터 프레임으로 불러오기
import pandas as pd
pd.options.display.float_format = '{:.2f}'.format # 소수점 두자리까지 출력옵션
pd.options.display.max_rows = 6 # 6행까지 출력 옵션
df_market1 = pd.read_excel('e_09_market.xlsx', sheet_name='2023-08')
df_market1
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
      <th>제품</th>
      <th>강남점</th>
      <th>서초점</th>
      <th>송파점</th>
      <th>노원점</th>
      <th>은평점</th>
      <th>금천점</th>
      <th>신도림점</th>
      <th>용인 죽전점</th>
      <th>용인 수지점</th>
      <th>수원 영통점</th>
      <th>인천 주안점</th>
      <th>인천 검단점</th>
      <th>동인천점</th>
      <th>분당점</th>
      <th>일산점</th>
      <th>부평점</th>
      <th>부천 옥길점</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>종합어묵 (300g)</td>
      <td>1930000</td>
      <td>1350000</td>
      <td>630000</td>
      <td>420000</td>
      <td>1360000</td>
      <td>1370000</td>
      <td>1880000</td>
      <td>1210000</td>
      <td>710000</td>
      <td>1130000</td>
      <td>280000</td>
      <td>1260000</td>
      <td>1480000</td>
      <td>960000</td>
      <td>960000</td>
      <td>710000</td>
      <td>1220000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>유기농떡튀밥 (40g)</td>
      <td>320000</td>
      <td>380000</td>
      <td>490000</td>
      <td>1320000</td>
      <td>1380000</td>
      <td>1350000</td>
      <td>1280000</td>
      <td>860000</td>
      <td>720000</td>
      <td>1500000</td>
      <td>1090000</td>
      <td>1850000</td>
      <td>420000</td>
      <td>570000</td>
      <td>720000</td>
      <td>1680000</td>
      <td>1020000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>염장다시마 (400g)</td>
      <td>1310000</td>
      <td>1700000</td>
      <td>670000</td>
      <td>640000</td>
      <td>1180000</td>
      <td>1840000</td>
      <td>910000</td>
      <td>1080000</td>
      <td>240000</td>
      <td>1510000</td>
      <td>660000</td>
      <td>1420000</td>
      <td>260000</td>
      <td>1440000</td>
      <td>1880000</td>
      <td>410000</td>
      <td>780000</td>
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
    </tr>
    <tr>
      <th>197</th>
      <td>생선가스 (300g)</td>
      <td>850000</td>
      <td>1440000</td>
      <td>200000</td>
      <td>300000</td>
      <td>410000</td>
      <td>300000</td>
      <td>1090000</td>
      <td>1830000</td>
      <td>610000</td>
      <td>1690000</td>
      <td>1390000</td>
      <td>1210000</td>
      <td>390000</td>
      <td>940000</td>
      <td>650000</td>
      <td>1300000</td>
      <td>1300000</td>
    </tr>
    <tr>
      <th>198</th>
      <td>게맛살 (178g)</td>
      <td>1240000</td>
      <td>380000</td>
      <td>1680000</td>
      <td>1500000</td>
      <td>730000</td>
      <td>550000</td>
      <td>440000</td>
      <td>1730000</td>
      <td>300000</td>
      <td>1700000</td>
      <td>450000</td>
      <td>1980000</td>
      <td>330000</td>
      <td>1430000</td>
      <td>1620000</td>
      <td>1780000</td>
      <td>1610000</td>
    </tr>
    <tr>
      <th>199</th>
      <td>멸치가루 (120g)</td>
      <td>250000</td>
      <td>1170000</td>
      <td>1980000</td>
      <td>340000</td>
      <td>1760000</td>
      <td>550000</td>
      <td>920000</td>
      <td>280000</td>
      <td>400000</td>
      <td>1370000</td>
      <td>1310000</td>
      <td>1710000</td>
      <td>980000</td>
      <td>1370000</td>
      <td>1890000</td>
      <td>930000</td>
      <td>1030000</td>
    </tr>
  </tbody>
</table>
<p>200 rows × 18 columns</p>
</div>




```python
# 첫번째 시트 언피벗하기 (열 이름은 제품, 지점, 매출액으로)
df_market1.melt('제품', var_name='지점', value_name='매출액')
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
      <th>제품</th>
      <th>지점</th>
      <th>매출액</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>종합어묵 (300g)</td>
      <td>강남점</td>
      <td>1930000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>유기농떡튀밥 (40g)</td>
      <td>강남점</td>
      <td>320000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>염장다시마 (400g)</td>
      <td>강남점</td>
      <td>1310000</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>3397</th>
      <td>생선가스 (300g)</td>
      <td>부천 옥길점</td>
      <td>1300000</td>
    </tr>
    <tr>
      <th>3398</th>
      <td>게맛살 (178g)</td>
      <td>부천 옥길점</td>
      <td>1610000</td>
    </tr>
    <tr>
      <th>3399</th>
      <td>멸치가루 (120g)</td>
      <td>부천 옥길점</td>
      <td>1030000</td>
    </tr>
  </tbody>
</table>
<p>3400 rows × 3 columns</p>
</div>




```python
# 월별 매출 시트의 이름들을 리스트(days)에 담기
days = ['2023-08', '2023-09', '2023-10', '2023-11', '2023-12']
days
```




    ['2023-08', '2023-09', '2023-10', '2023-11', '2023-12']




```python
# for문으로 만들어서 5개의 시트(월별데이터)를 리스트에(dfs) 담기
dfs = []
for i in days:
    df = pd.read_excel('e_09_market.xlsx', sheet_name=i)
    df = df.melt('제품', var_name='지점', value_name='매출액')
    df['날짜'] = i  
    dfs.append(df)
dfs
```




    [                제품      지점      매출액       날짜
     0      종합어묵 (300g)     강남점  1930000  2023-08
     1     유기농떡튀밥 (40g)     강남점   320000  2023-08
     2     염장다시마 (400g)     강남점  1310000  2023-08
     ...            ...     ...      ...      ...
     3397   생선가스 (300g)  부천 옥길점  1300000  2023-08
     3398    게맛살 (178g)  부천 옥길점  1610000  2023-08
     3399   멸치가루 (120g)  부천 옥길점  1030000  2023-08
     
     [3400 rows x 4 columns],
                     제품      지점      매출액       날짜
     0      종합어묵 (300g)     강남점   520000  2023-09
     1     유기농떡튀밥 (40g)     강남점  1790000  2023-09
     2     염장다시마 (400g)     강남점  1080000  2023-09
     ...            ...     ...      ...      ...
     3397   생선가스 (300g)  부천 옥길점  1490000  2023-09
     3398    게맛살 (178g)  부천 옥길점  1740000  2023-09
     3399   멸치가루 (120g)  부천 옥길점   410000  2023-09
     
     [3400 rows x 4 columns],
                     제품      지점      매출액       날짜
     0      종합어묵 (300g)     강남점  1860000  2023-10
     1     유기농떡튀밥 (40g)     강남점   650000  2023-10
     2     염장다시마 (400g)     강남점   810000  2023-10
     ...            ...     ...      ...      ...
     3397   생선가스 (300g)  부천 옥길점   550000  2023-10
     3398    게맛살 (178g)  부천 옥길점  1340000  2023-10
     3399   멸치가루 (120g)  부천 옥길점   210000  2023-10
     
     [3400 rows x 4 columns],
                     제품      지점      매출액       날짜
     0      종합어묵 (300g)     강남점  1380000  2023-11
     1     유기농떡튀밥 (40g)     강남점   900000  2023-11
     2     염장다시마 (400g)     강남점  1400000  2023-11
     ...            ...     ...      ...      ...
     3397   생선가스 (300g)  부천 옥길점   350000  2023-11
     3398    게맛살 (178g)  부천 옥길점   910000  2023-11
     3399   멸치가루 (120g)  부천 옥길점   390000  2023-11
     
     [3400 rows x 4 columns],
                     제품      지점      매출액       날짜
     0      종합어묵 (300g)     강남점  1980000  2023-12
     1     유기농떡튀밥 (40g)     강남점  1580000  2023-12
     2     염장다시마 (400g)     강남점   540000  2023-12
     ...            ...     ...      ...      ...
     3397   생선가스 (300g)  부천 옥길점   480000  2023-12
     3398    게맛살 (178g)  부천 옥길점   670000  2023-12
     3399   멸치가루 (120g)  부천 옥길점  1940000  2023-12
     
     [3400 rows x 4 columns]]




```python
# dfs를 concat해서 전부 합친다 (결과는 df_result로 선언)
df_result = pd.concat(dfs)
df_result
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
      <th>제품</th>
      <th>지점</th>
      <th>매출액</th>
      <th>날짜</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>종합어묵 (300g)</td>
      <td>강남점</td>
      <td>1930000</td>
      <td>2023-08</td>
    </tr>
    <tr>
      <th>1</th>
      <td>유기농떡튀밥 (40g)</td>
      <td>강남점</td>
      <td>320000</td>
      <td>2023-08</td>
    </tr>
    <tr>
      <th>2</th>
      <td>염장다시마 (400g)</td>
      <td>강남점</td>
      <td>1310000</td>
      <td>2023-08</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>3397</th>
      <td>생선가스 (300g)</td>
      <td>부천 옥길점</td>
      <td>480000</td>
      <td>2023-12</td>
    </tr>
    <tr>
      <th>3398</th>
      <td>게맛살 (178g)</td>
      <td>부천 옥길점</td>
      <td>670000</td>
      <td>2023-12</td>
    </tr>
    <tr>
      <th>3399</th>
      <td>멸치가루 (120g)</td>
      <td>부천 옥길점</td>
      <td>1940000</td>
      <td>2023-12</td>
    </tr>
  </tbody>
</table>
<p>17000 rows × 4 columns</p>
</div>




```python
# 지점 구분 merge
df_result = df_result.merge(pd.read_excel('e_09_market.xlsx', sheet_name=5), how='left')
df_result
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
      <th>제품</th>
      <th>지점</th>
      <th>매출액</th>
      <th>날짜</th>
      <th>지역</th>
      <th>분류</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>종합어묵 (300g)</td>
      <td>강남점</td>
      <td>1930000</td>
      <td>2023-08</td>
      <td>서울</td>
      <td>직영점</td>
    </tr>
    <tr>
      <th>1</th>
      <td>유기농떡튀밥 (40g)</td>
      <td>강남점</td>
      <td>320000</td>
      <td>2023-08</td>
      <td>서울</td>
      <td>직영점</td>
    </tr>
    <tr>
      <th>2</th>
      <td>염장다시마 (400g)</td>
      <td>강남점</td>
      <td>1310000</td>
      <td>2023-08</td>
      <td>서울</td>
      <td>직영점</td>
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
      <th>16997</th>
      <td>생선가스 (300g)</td>
      <td>부천 옥길점</td>
      <td>480000</td>
      <td>2023-12</td>
      <td>경기</td>
      <td>직영점</td>
    </tr>
    <tr>
      <th>16998</th>
      <td>게맛살 (178g)</td>
      <td>부천 옥길점</td>
      <td>670000</td>
      <td>2023-12</td>
      <td>경기</td>
      <td>직영점</td>
    </tr>
    <tr>
      <th>16999</th>
      <td>멸치가루 (120g)</td>
      <td>부천 옥길점</td>
      <td>1940000</td>
      <td>2023-12</td>
      <td>경기</td>
      <td>직영점</td>
    </tr>
  </tbody>
</table>
<p>17000 rows × 6 columns</p>
</div>




```python
# 제품 구분 merge
df_result = df_result.merge(pd.read_excel('e_09_market.xlsx', sheet_name=6), how='left')
df_result
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
      <th>제품</th>
      <th>지점</th>
      <th>매출액</th>
      <th>날짜</th>
      <th>지역</th>
      <th>분류</th>
      <th>구분</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>종합어묵 (300g)</td>
      <td>강남점</td>
      <td>1930000</td>
      <td>2023-08</td>
      <td>서울</td>
      <td>직영점</td>
      <td>기타</td>
    </tr>
    <tr>
      <th>1</th>
      <td>유기농떡튀밥 (40g)</td>
      <td>강남점</td>
      <td>320000</td>
      <td>2023-08</td>
      <td>서울</td>
      <td>직영점</td>
      <td>기타</td>
    </tr>
    <tr>
      <th>2</th>
      <td>염장다시마 (400g)</td>
      <td>강남점</td>
      <td>1310000</td>
      <td>2023-08</td>
      <td>서울</td>
      <td>직영점</td>
      <td>기타</td>
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
      <th>16997</th>
      <td>생선가스 (300g)</td>
      <td>부천 옥길점</td>
      <td>480000</td>
      <td>2023-12</td>
      <td>경기</td>
      <td>직영점</td>
      <td>기타</td>
    </tr>
    <tr>
      <th>16998</th>
      <td>게맛살 (178g)</td>
      <td>부천 옥길점</td>
      <td>670000</td>
      <td>2023-12</td>
      <td>경기</td>
      <td>직영점</td>
      <td>기타</td>
    </tr>
    <tr>
      <th>16999</th>
      <td>멸치가루 (120g)</td>
      <td>부천 옥길점</td>
      <td>1940000</td>
      <td>2023-12</td>
      <td>경기</td>
      <td>직영점</td>
      <td>기타</td>
    </tr>
  </tbody>
</table>
<p>17000 rows × 7 columns</p>
</div>




```python
# 지역별 월별 청과물 매출액 집계하기
df_result[df_result['구분'].eq('청과')].pivot_table('매출액', index='날짜', columns='지역', aggfunc=sum)
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
      <th>지역</th>
      <th>경기</th>
      <th>서울</th>
      <th>인천</th>
    </tr>
    <tr>
      <th>날짜</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2023-08</th>
      <td>57850000</td>
      <td>75540000</td>
      <td>35930000</td>
    </tr>
    <tr>
      <th>2023-09</th>
      <td>57050000</td>
      <td>71980000</td>
      <td>35410000</td>
    </tr>
    <tr>
      <th>2023-10</th>
      <td>58170000</td>
      <td>63510000</td>
      <td>37840000</td>
    </tr>
    <tr>
      <th>2023-11</th>
      <td>48510000</td>
      <td>70250000</td>
      <td>36810000</td>
    </tr>
    <tr>
      <th>2023-12</th>
      <td>55950000</td>
      <td>68270000</td>
      <td>39870000</td>
    </tr>
  </tbody>
</table>
</div>




```python
# matplotlib 폰트설정
import matplotlib.pyplot as plt
plt.rc('font', family='gulim')
```

**matplotlib colormap**

<img src=https://i.ibb.co/2qV3FLy/image.png, width=600>

이 외도 많은 colormap이 있으니 구글에서 matplotlib colormap으로 검색해보거나

아래 링크를 참조하자

https://matplotlib.org/stable/tutorials/colors/colormaps.html



```python
# 위 결과를 간단한 그래프로 그리기 (.plot(kind='bar', width=.9, colormap='tab20c'))
(df_result[df_result['구분'].eq('청과')]
 .pivot_table('매출액', index='날짜', columns='지역', aggfunc=sum)
 .plot(kind='bar', width=.9, colormap='tab20c'))
```




    <Axes: xlabel='날짜'>




    
![png](output_34_1.png)
    


그 외 다양한 피벗테이블을 만들 수 있다

**아래의 코드를 쓰면 보다 간편하게 언피벗이 가능하다**


```python
# 각 시트별로 언피벗하고 합치는 것을 좀더 간단한 코드로 하는 방법
df_dict = pd.read_excel('e_09_market.xlsx', sheet_name=None)
df_list = list(df_dict.items())[:5]
dfs = [df.melt('제품', var_name='지점', value_name='매출액').assign(날짜=name) for name, df in df_list]
df_result1 = pd.concat(dfs)
df_result1
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
      <th>제품</th>
      <th>지점</th>
      <th>매출액</th>
      <th>날짜</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>종합어묵 (300g)</td>
      <td>강남점</td>
      <td>1930000</td>
      <td>2023-08</td>
    </tr>
    <tr>
      <th>1</th>
      <td>유기농떡튀밥 (40g)</td>
      <td>강남점</td>
      <td>320000</td>
      <td>2023-08</td>
    </tr>
    <tr>
      <th>2</th>
      <td>염장다시마 (400g)</td>
      <td>강남점</td>
      <td>1310000</td>
      <td>2023-08</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>3397</th>
      <td>생선가스 (300g)</td>
      <td>부천 옥길점</td>
      <td>480000</td>
      <td>2023-12</td>
    </tr>
    <tr>
      <th>3398</th>
      <td>게맛살 (178g)</td>
      <td>부천 옥길점</td>
      <td>670000</td>
      <td>2023-12</td>
    </tr>
    <tr>
      <th>3399</th>
      <td>멸치가루 (120g)</td>
      <td>부천 옥길점</td>
      <td>1940000</td>
      <td>2023-12</td>
    </tr>
  </tbody>
</table>
<p>17000 rows × 4 columns</p>
</div>



# 2. groupby <a class="anchor" id="chapter7"></a>

## 1. groupby가 필요한 이유 <a class="anchor" id="chapter8"></a>


<img src=https://i.ibb.co/s5098LD/image.png, width=600>

이 데이터에서 점수를 이용해 전체 등수를 구하는 것은 어렵지 않다.

**그런데 성별내에서의 등수를 구하려면 어떻게 해야 할까?**

→ 이럴 때 필요한 것이 `groupby` 함수

`groupby` 함수를 사용하면 다른 열의 특성으로 그룹화해서 함수를 적용하게 해준다

위 그림에서 `groupby`로 성별 열로 그룹화해서 `rank` 함수를 적용하면 성별내 등수를 구할 수 있다


원시(raw)데이터는 많은 열(특성)들이 존재하기에 그룹화해서 함수를 적용하지 못하면 원시데이터를 다룰 수 없다

→ `groupby`는 원시(raw) 데이터를 다루기 위해 가장 필수적인 함수

## 2. groupby 함수 <a class="anchor" id="chapter9"></a>

새로운 함수 소개

> pandas groupby

<img src=https://i.ibb.co/09D6rDh/image.png, width=600>


그룹 내에서 함수를 적용하게 해주는 함수

> groupby 함수의 인자(parameter)들

**by** 

그룹을 나누는 기준이 되는 열이나 행을 지정하는 인자. 복수라면 리스트로 입력

 

**axis** (0 or 1)

그룹해서 함수를 행에 적용할 것인지 열에 적용할 것인지 지정하는 인자. 기본값은 열

 

**level**

인덱스로 그룹을 나눌 때 사용하는 인자

 

**as_index** (bool / 기본값은 True)

groupby로 집계 함수를 사용할 때 그룹이 인덱스가 될지 지정하는 인자

 

**sort** (bool / 기본값은 True)

그룹의 key로 정렬을 할 것인지 지정하는 인자

---

<br>

**groupby 함수의 구조**

`groupby` 함수는 인자보다 구조를 숙지하는 것이 더 중요하다



<img src=https://i.ibb.co/c2t4ykz/image.png, width=600>


1. 소괄호 안에 by (그룹의 기준)

2. 대괄호 안에 data(함수를 적용할 열)

3. 적용할 함수

이 세 부분의 구조로 되어 있다

---
<br>


```python
# 실습 준비 코드
import pandas as pd
data1 = [['김판다', 'A', '남', 95], ['송중기', 'B', '남', 93],
         ['김나현', 'B', '여', 88], ['박효신', 'A', '남', 85],
         ['강승주', 'B', '여', 78], ['권보아', 'A', '여', 72]]
data2 = [['2021-01-01', '김판다', 10000], ['2021-01-01', '강승주', 2000],
         ['2021-01-02', '김판다', 20000], ['2021-01-02', '강승주', 5000],
         ['2021-01-03', '강승주', 8000], ['2021-01-03', '김판다', 5000]]
data3 = {'제품': ['A', 'B', 'B', 'A', 'C', 'A'], '판매량': [10, 20, 30, 40, 50, 60]}

df = pd.DataFrame(data1, columns=['이름', '반', '성별', '점수'])
df1 = df.copy()
df2 = pd.DataFrame(data2, columns=['날짜', '이름', '입금'])
df3 = pd.DataFrame(data3)
```

**groupby로 feature engineering 하기** <a class="anchor" id="chapter10"></a>

`groupby` 함수와 `rank` 함수 사용하기


```python
# df1을 출력
df1
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
      <th>이름</th>
      <th>반</th>
      <th>성별</th>
      <th>점수</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>김판다</td>
      <td>A</td>
      <td>남</td>
      <td>95</td>
    </tr>
    <tr>
      <th>1</th>
      <td>송중기</td>
      <td>B</td>
      <td>남</td>
      <td>93</td>
    </tr>
    <tr>
      <th>2</th>
      <td>김나현</td>
      <td>B</td>
      <td>여</td>
      <td>88</td>
    </tr>
    <tr>
      <th>3</th>
      <td>박효신</td>
      <td>A</td>
      <td>남</td>
      <td>85</td>
    </tr>
    <tr>
      <th>4</th>
      <td>강승주</td>
      <td>B</td>
      <td>여</td>
      <td>78</td>
    </tr>
    <tr>
      <th>5</th>
      <td>권보아</td>
      <td>A</td>
      <td>여</td>
      <td>72</td>
    </tr>
  </tbody>
</table>
</div>




```python
# df1에서 반으로 그룹화해서 점수열에 rank함수를 내림차순으로 적용해 반등수 구하기
df1.groupby('반')['점수'].rank(ascending=False)
```




    0   1.00
    1   1.00
    2   2.00
    3   2.00
    4   3.00
    5   3.00
    Name: 점수, dtype: float64




```python
# 위 결과를 반등수 열로 만들기
df1['반등수'] = df1.groupby('반')['점수'].rank(ascending=False)
df1
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
      <th>이름</th>
      <th>반</th>
      <th>성별</th>
      <th>점수</th>
      <th>반등수</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>김판다</td>
      <td>A</td>
      <td>남</td>
      <td>95</td>
      <td>1.00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>송중기</td>
      <td>B</td>
      <td>남</td>
      <td>93</td>
      <td>1.00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>김나현</td>
      <td>B</td>
      <td>여</td>
      <td>88</td>
      <td>2.00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>박효신</td>
      <td>A</td>
      <td>남</td>
      <td>85</td>
      <td>2.00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>강승주</td>
      <td>B</td>
      <td>여</td>
      <td>78</td>
      <td>3.00</td>
    </tr>
    <tr>
      <th>5</th>
      <td>권보아</td>
      <td>A</td>
      <td>여</td>
      <td>72</td>
      <td>3.00</td>
    </tr>
  </tbody>
</table>
</div>




```python
# df1에서 반과 성별로 그룹화해서 점수열에 rank함수를 내림차순으로 적용해 반등수 구하기
df1.groupby(['반', '성별'])['점수'].rank(ascending=False)
```




    0   1.00
    1   1.00
    2   1.00
    3   2.00
    4   2.00
    5   1.00
    Name: 점수, dtype: float64



`groupby` 함수와 `shift` 함수 사용하기


```python
# df2를 출력
df2
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
      <th>날짜</th>
      <th>이름</th>
      <th>입금</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2021-01-01</td>
      <td>김판다</td>
      <td>10000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2021-01-01</td>
      <td>강승주</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2021-01-02</td>
      <td>김판다</td>
      <td>20000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2021-01-02</td>
      <td>강승주</td>
      <td>5000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2021-01-03</td>
      <td>강승주</td>
      <td>8000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2021-01-03</td>
      <td>김판다</td>
      <td>5000</td>
    </tr>
  </tbody>
</table>
</div>




```python
# df2에서 각 사람의 전일 입금액 구하기
df2.groupby('이름')['입금'].shift()
```




    0        NaN
    1        NaN
    2   10000.00
    3    2000.00
    4    5000.00
    5   20000.00
    Name: 입금, dtype: float64




```python
# df2에서 각 사람의 전일 입금액 구하기
df2['전일입금'] = df2.groupby('이름')['입금'].shift()
df2
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
      <th>날짜</th>
      <th>이름</th>
      <th>입금</th>
      <th>전일입금</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2021-01-01</td>
      <td>김판다</td>
      <td>10000</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2021-01-01</td>
      <td>강승주</td>
      <td>2000</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2021-01-02</td>
      <td>김판다</td>
      <td>20000</td>
      <td>10000.00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2021-01-02</td>
      <td>강승주</td>
      <td>5000</td>
      <td>2000.00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2021-01-03</td>
      <td>강승주</td>
      <td>8000</td>
      <td>5000.00</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2021-01-03</td>
      <td>김판다</td>
      <td>5000</td>
      <td>20000.00</td>
    </tr>
  </tbody>
</table>
</div>



`groupby` 함수와 `cumcount` 함수 사용하기

새로운 함수 소개

> pandas cumcount

<img src=https://i.ibb.co/5RHRgtx/image.png, width=600>


그룹화해서 각 항목마다 순번을 매기는 함수 (반드시 groupby와 쓴다)

excel의 countif와 유사하다


> cumcount 함수의 인자(parameter)들

**ascending**(True 또는 False / 기본값은 True)


오름차순으로 순번을 부여할 것인지 내림차순으로 순번을 부여할 것인지 결정하는 인자

기본값은 True이며 오름차순이고 위에서부터 0부터 넘버링 한다

---

<br>


```python
# 실습에 쓰일 df3 출력
df3
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
      <th>제품</th>
      <th>판매량</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>10</td>
    </tr>
    <tr>
      <th>1</th>
      <td>B</td>
      <td>20</td>
    </tr>
    <tr>
      <th>2</th>
      <td>B</td>
      <td>30</td>
    </tr>
    <tr>
      <th>3</th>
      <td>A</td>
      <td>40</td>
    </tr>
    <tr>
      <th>4</th>
      <td>C</td>
      <td>50</td>
    </tr>
    <tr>
      <th>5</th>
      <td>A</td>
      <td>60</td>
    </tr>
  </tbody>
</table>
</div>




```python
# cumcount 함수를 이용해 각 제품을 그룹화해서 나온 순서대로 순번 붙이기
df3.groupby('제품').cumcount()
```




    0    0
    1    0
    2    1
    3    1
    4    0
    5    2
    dtype: int64




```python
# 위 결과를 순번 열로 만들기
df3['순번'] = df3.groupby('제품').cumcount()
df3
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
      <th>제품</th>
      <th>판매량</th>
      <th>순번</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A</td>
      <td>10</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>B</td>
      <td>20</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>B</td>
      <td>30</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>A</td>
      <td>40</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>C</td>
      <td>50</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>A</td>
      <td>60</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>




```python
# index를 제품, columns를 순번으로 판매량을 values로 하는 피벗테이블 만들기
df3.pivot_table('판매량', index='제품', columns='순번').fillna(0)
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
      <th>순번</th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
    </tr>
    <tr>
      <th>제품</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>A</th>
      <td>10.00</td>
      <td>40.00</td>
      <td>60.00</td>
    </tr>
    <tr>
      <th>B</th>
      <td>20.00</td>
      <td>30.00</td>
      <td>0.00</td>
    </tr>
    <tr>
      <th>C</th>
      <td>50.00</td>
      <td>0.00</td>
      <td>0.00</td>
    </tr>
  </tbody>
</table>
</div>



그 외 feature engineering에 쓰였던 많은 함수들을 `groupby`와 함께 사용할 수 있다

**groupby로 집계함수 적용하기** <a class="anchor" id="chapter11"></a>

- `groupby` 함수로 집계함수도 적용할 수 있다.

- `unstack` 과 함께 사용하면 마치 피벗테이블과 같은 결과를 얻을 수 있다

<img src=https://i.ibb.co/P4BMjPx/image.png, width=600>


```python
# 실습에 쓰일 df 출력(df1과 같다)
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
      <th>이름</th>
      <th>반</th>
      <th>성별</th>
      <th>점수</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>김판다</td>
      <td>A</td>
      <td>남</td>
      <td>95</td>
    </tr>
    <tr>
      <th>1</th>
      <td>송중기</td>
      <td>B</td>
      <td>남</td>
      <td>93</td>
    </tr>
    <tr>
      <th>2</th>
      <td>김나현</td>
      <td>B</td>
      <td>여</td>
      <td>88</td>
    </tr>
    <tr>
      <th>3</th>
      <td>박효신</td>
      <td>A</td>
      <td>남</td>
      <td>85</td>
    </tr>
    <tr>
      <th>4</th>
      <td>강승주</td>
      <td>B</td>
      <td>여</td>
      <td>78</td>
    </tr>
    <tr>
      <th>5</th>
      <td>권보아</td>
      <td>A</td>
      <td>여</td>
      <td>72</td>
    </tr>
  </tbody>
</table>
</div>




```python
# df에서 반과 성별로 나눠 점수의 평균 구하기
df.groupby(['반', '성별'])['점수'].mean()
```




    반  성별
    A  남    90.00
       여    72.00
    B  남    93.00
       여    83.00
    Name: 점수, dtype: float64




```python
# 위 결과를 피벗테이블과 같은 교차표로 만들기
df.groupby(['반', '성별'])['점수'].mean().unstack()
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
      <th>성별</th>
      <th>남</th>
      <th>여</th>
    </tr>
    <tr>
      <th>반</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>A</th>
      <td>90.00</td>
      <td>72.00</td>
    </tr>
    <tr>
      <th>B</th>
      <td>93.00</td>
      <td>83.00</td>
    </tr>
  </tbody>
</table>
</div>



**채권 데이터로 여러가지 그룹 집계하기** <a class="anchor" id="chapter12"></a>

파일명 : c_10_bond.csvt


<img src=https://i.ibb.co/gM6dq5T/image.png, width=600>

이 데이터는 일자별, 팀별, 종목별로 나눠 액면가와 시장가가 나와있는 데이터 프레임이다

위 데이터로 일별 시장가와 액면가의 추이를 집계해보자

또한 2023Q1과 2023Q2의 일자별 추이를 집계해보자

csvt 파일이지만 `read_csv` 함수로 읽을 수 있다.

이 파일은 CP949로 인코딩하는 것이 필요한데 `read_csv`함수의 `encoding` 인자를 사용하면 된다

일자열을 datetime 자료형으로 불러오고 싶다면 `parse_dates` 인자를 사용하자

(주의 : `parse_dates` 인자는 단수의 열이라도 리스트로 입력해야 한다)

---

<br>

<a class="anchor" id="chapter12"></a>


```python
# 위 파일을 encoding은 CP949, 일자열의 자료형은 datetime으로 지정해 데이터프레임으로 불러 df_bond로 지정
import pandas as pd
pd.options.display.max_rows = 6
df_bond = pd.read_csv('c_10_bond.csvt', encoding='CP949', low_memory=False, parse_dates=['일자'])
df_bond
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
      <th>일자</th>
      <th>종목코드</th>
      <th>팀점코드</th>
      <th>펀드코드</th>
      <th>통화</th>
      <th>발행자등급</th>
      <th>종목등급</th>
      <th>환율</th>
      <th>액면가</th>
      <th>시장가</th>
      <th>상품분류</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2023-01-01</td>
      <td>KR310607AC75797</td>
      <td>4120500</td>
      <td>L0050797</td>
      <td>KRW</td>
      <td>AAA</td>
      <td>AAA</td>
      <td>NaN</td>
      <td>53632</td>
      <td>52963</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2023-01-01</td>
      <td>KR310202AC73824</td>
      <td>4120500</td>
      <td>L0050797</td>
      <td>KRW</td>
      <td>AAA</td>
      <td>AAA</td>
      <td>NaN</td>
      <td>268160</td>
      <td>262526</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2023-01-01</td>
      <td>KR310203AC82377</td>
      <td>4120500</td>
      <td>L0050797</td>
      <td>KRW</td>
      <td>AAA</td>
      <td>AAA</td>
      <td>NaN</td>
      <td>67040</td>
      <td>66349</td>
      <td>NaN</td>
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
    </tr>
    <tr>
      <th>168619</th>
      <td>2023-06-30</td>
      <td>HW9999900031763</td>
      <td>9999999</td>
      <td>S1090987</td>
      <td>KRW</td>
      <td>AA-</td>
      <td>AA-</td>
      <td>1.00</td>
      <td>5706</td>
      <td>6016</td>
      <td>CD/CP</td>
    </tr>
    <tr>
      <th>168620</th>
      <td>2023-06-30</td>
      <td>HW9999900036461</td>
      <td>9999999</td>
      <td>S1090987</td>
      <td>KRW</td>
      <td>AA-</td>
      <td>AA-</td>
      <td>1.00</td>
      <td>6393</td>
      <td>6093</td>
      <td>CD/CP</td>
    </tr>
    <tr>
      <th>168621</th>
      <td>2023-06-30</td>
      <td>HW9999900033824</td>
      <td>9999999</td>
      <td>S1090987</td>
      <td>KRW</td>
      <td>AA-</td>
      <td>AA-</td>
      <td>1.00</td>
      <td>6462</td>
      <td>6501</td>
      <td>CD/CP</td>
    </tr>
  </tbody>
</table>
<p>168622 rows × 11 columns</p>
</div>



필터링 실습


```python
# df_bond를 2023년 3월 1일 이후의 데이터만 필터링하기
cond1 = df_bond['일자'] >= '2023-03'
df_bond[cond1]
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
      <th>일자</th>
      <th>종목코드</th>
      <th>팀점코드</th>
      <th>펀드코드</th>
      <th>통화</th>
      <th>발행자등급</th>
      <th>종목등급</th>
      <th>환율</th>
      <th>액면가</th>
      <th>시장가</th>
      <th>상품분류</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>52047</th>
      <td>2023-03-01</td>
      <td>KR103501GA25723</td>
      <td>4121000</td>
      <td>L0040564</td>
      <td>KRW</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>40224</td>
      <td>-40032</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>52048</th>
      <td>2023-03-01</td>
      <td>KR310101GC16357</td>
      <td>4121000</td>
      <td>L0040564</td>
      <td>KRW</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>53632</td>
      <td>52956</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>52049</th>
      <td>2023-03-01</td>
      <td>KR310505AD24222</td>
      <td>4121000</td>
      <td>L0040564</td>
      <td>KRW</td>
      <td>AAA</td>
      <td>AAA</td>
      <td>NaN</td>
      <td>40224</td>
      <td>38740</td>
      <td>NaN</td>
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
    </tr>
    <tr>
      <th>168619</th>
      <td>2023-06-30</td>
      <td>HW9999900031763</td>
      <td>9999999</td>
      <td>S1090987</td>
      <td>KRW</td>
      <td>AA-</td>
      <td>AA-</td>
      <td>1.00</td>
      <td>5706</td>
      <td>6016</td>
      <td>CD/CP</td>
    </tr>
    <tr>
      <th>168620</th>
      <td>2023-06-30</td>
      <td>HW9999900036461</td>
      <td>9999999</td>
      <td>S1090987</td>
      <td>KRW</td>
      <td>AA-</td>
      <td>AA-</td>
      <td>1.00</td>
      <td>6393</td>
      <td>6093</td>
      <td>CD/CP</td>
    </tr>
    <tr>
      <th>168621</th>
      <td>2023-06-30</td>
      <td>HW9999900033824</td>
      <td>9999999</td>
      <td>S1090987</td>
      <td>KRW</td>
      <td>AA-</td>
      <td>AA-</td>
      <td>1.00</td>
      <td>6462</td>
      <td>6501</td>
      <td>CD/CP</td>
    </tr>
  </tbody>
</table>
<p>116575 rows × 11 columns</p>
</div>



`groupby` 실습

- 시간의 흐름에 따라 액면가와 시장가 살펴보기


```python
# 시간의 흐름에 따라 일자별 액면가와 시장가 집계하기
df_bond.groupby('일자')[['액면가','시장가']].sum()
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
      <th>액면가</th>
      <th>시장가</th>
    </tr>
    <tr>
      <th>일자</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2023-01-01</th>
      <td>6732328</td>
      <td>6814690</td>
    </tr>
    <tr>
      <th>2023-01-02</th>
      <td>6722595</td>
      <td>6805099</td>
    </tr>
    <tr>
      <th>2023-01-03</th>
      <td>6870190</td>
      <td>6959289</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2023-06-28</th>
      <td>7464718</td>
      <td>7710832</td>
    </tr>
    <tr>
      <th>2023-06-29</th>
      <td>7581121</td>
      <td>7826814</td>
    </tr>
    <tr>
      <th>2023-06-30</th>
      <td>7360874</td>
      <td>7604406</td>
    </tr>
  </tbody>
</table>
<p>181 rows × 2 columns</p>
</div>




```python
# 시간의 흐름에 따라 일자별 액면가와 시장가 집계결과를 그래프로 (.plot(lw=3, colormap='Set1'))
df_bond.groupby('일자')[['액면가','시장가']].sum().plot(lw=3, colormap='Set1')
```




    <Axes: xlabel='일자'>




    
![png](output_67_1.png)
    



```python
# 위 피벗테이블을 df_pivot으로 지정하기
df_pivot = df_bond.groupby('일자')[['액면가','시장가']].sum()
df_pivot
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
      <th>액면가</th>
      <th>시장가</th>
    </tr>
    <tr>
      <th>일자</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2023-01-01</th>
      <td>6732328</td>
      <td>6814690</td>
    </tr>
    <tr>
      <th>2023-01-02</th>
      <td>6722595</td>
      <td>6805099</td>
    </tr>
    <tr>
      <th>2023-01-03</th>
      <td>6870190</td>
      <td>6959289</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2023-06-28</th>
      <td>7464718</td>
      <td>7710832</td>
    </tr>
    <tr>
      <th>2023-06-29</th>
      <td>7581121</td>
      <td>7826814</td>
    </tr>
    <tr>
      <th>2023-06-30</th>
      <td>7360874</td>
      <td>7604406</td>
    </tr>
  </tbody>
</table>
<p>181 rows × 2 columns</p>
</div>




```python
pd.options.display.max_rows = 6
url = "C://Users//KimPanda//ipynb//hanhwa//7주차//e_03_04_sheet02.xlsx"
df = pd.read_excel(url, parse_dates=['date'])
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
      <th>date</th>
      <th>nation</th>
      <th>continent</th>
      <th>H/A</th>
      <th>oppenent</th>
      <th>o_continent</th>
      <th>score</th>
      <th>o_score</th>
      <th>tournament</th>
      <th>result</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1993-08-08</td>
      <td>Bolivia</td>
      <td>South America</td>
      <td>Home</td>
      <td>Uruguay</td>
      <td>South America</td>
      <td>3</td>
      <td>1</td>
      <td>Others</td>
      <td>Win</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1993-08-08</td>
      <td>Uruguay</td>
      <td>South America</td>
      <td>Away</td>
      <td>Bolivia</td>
      <td>South America</td>
      <td>1</td>
      <td>3</td>
      <td>Others</td>
      <td>Lose</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1993-08-08</td>
      <td>Mexico</td>
      <td>North America</td>
      <td>Home</td>
      <td>Brazil</td>
      <td>South America</td>
      <td>1</td>
      <td>1</td>
      <td>Frendly</td>
      <td>Draw</td>
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
    </tr>
    <tr>
      <th>47839</th>
      <td>2022-06-14</td>
      <td>Japan</td>
      <td>Asia</td>
      <td>Away</td>
      <td>Tunisia</td>
      <td>Africa</td>
      <td>0</td>
      <td>3</td>
      <td>Others</td>
      <td>Lose</td>
    </tr>
    <tr>
      <th>47840</th>
      <td>2022-06-14</td>
      <td>Korea</td>
      <td>Asia</td>
      <td>Home</td>
      <td>Egypt</td>
      <td>Africa</td>
      <td>4</td>
      <td>1</td>
      <td>Frendly</td>
      <td>Win</td>
    </tr>
    <tr>
      <th>47841</th>
      <td>2022-06-14</td>
      <td>Egypt</td>
      <td>Africa</td>
      <td>Away</td>
      <td>Korea</td>
      <td>Asia</td>
      <td>1</td>
      <td>4</td>
      <td>Frendly</td>
      <td>Lose</td>
    </tr>
  </tbody>
</table>
<p>47842 rows × 10 columns</p>
</div>




```python
df1 = df.groupby([df['date'].dt.to_period(freq='Y'), 'continent'])['score'].mean().unstack()
df1
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
      <th>continent</th>
      <th>Africa</th>
      <th>Asia</th>
      <th>Europe</th>
      <th>North America</th>
      <th>Oceania</th>
      <th>South America</th>
    </tr>
    <tr>
      <th>date</th>
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
      <th>1993</th>
      <td>1.09</td>
      <td>1.55</td>
      <td>1.48</td>
      <td>1.54</td>
      <td>1.00</td>
      <td>1.46</td>
    </tr>
    <tr>
      <th>1994</th>
      <td>1.20</td>
      <td>1.10</td>
      <td>1.38</td>
      <td>1.10</td>
      <td>1.17</td>
      <td>1.34</td>
    </tr>
    <tr>
      <th>1995</th>
      <td>1.21</td>
      <td>1.17</td>
      <td>1.44</td>
      <td>1.32</td>
      <td>1.51</td>
      <td>1.55</td>
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
      <th>2020</th>
      <td>1.15</td>
      <td>1.16</td>
      <td>1.20</td>
      <td>1.39</td>
      <td>NaN</td>
      <td>1.68</td>
    </tr>
    <tr>
      <th>2021</th>
      <td>1.19</td>
      <td>1.40</td>
      <td>1.48</td>
      <td>1.45</td>
      <td>2.00</td>
      <td>1.16</td>
    </tr>
    <tr>
      <th>2022</th>
      <td>1.09</td>
      <td>1.29</td>
      <td>1.24</td>
      <td>1.26</td>
      <td>1.44</td>
      <td>1.58</td>
    </tr>
  </tbody>
</table>
<p>30 rows × 6 columns</p>
</div>




```python
from pandasecharts import echart
df1.round(3).reset_index().echart.line(x='date', ys=['Asia', 'Europe'], xtype='category').render_notebook()
```





<script>
    require.config({
        paths: {
            'echarts':'https://assets.pyecharts.org/assets/v5/echarts.min'
        }
    });
</script>

        <div id="1126f76f09c5480cb3a53bffe8483102" style="width:900px; height:500px;"></div>

<script>
        require(['echarts'], function(echarts) {
                var chart_1126f76f09c5480cb3a53bffe8483102 = echarts.init(
                    document.getElementById('1126f76f09c5480cb3a53bffe8483102'), 'white', {renderer: 'canvas'});
                var option_1126f76f09c5480cb3a53bffe8483102 = {
    "animation": true,
    "animationThreshold": 2000,
    "animationDuration": 1000,
    "animationEasing": "cubicOut",
    "animationDelay": 0,
    "animationDurationUpdate": 300,
    "animationEasingUpdate": "cubicOut",
    "animationDelayUpdate": 0,
    "aria": {
        "enabled": false
    },
    "color": [
        "#5470c6",
        "#91cc75",
        "#fac858",
        "#ee6666",
        "#73c0de",
        "#3ba272",
        "#fc8452",
        "#9a60b4",
        "#ea7ccc"
    ],
    "series": [
        {
            "type": "line",
            "name": "Asia",
            "connectNulls": false,
            "xAxisIndex": 0,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [
                [
                    "1993",
                    1.553
                ],
                [
                    "1994",
                    1.099
                ],
                [
                    "1995",
                    1.172
                ],
                [
                    "1996",
                    1.721
                ],
                [
                    "1997",
                    1.605
                ],
                [
                    "1998",
                    1.408
                ],
                [
                    "1999",
                    1.5
                ],
                [
                    "2000",
                    1.528
                ],
                [
                    "2001",
                    1.607
                ],
                [
                    "2002",
                    1.332
                ],
                [
                    "2003",
                    1.557
                ],
                [
                    "2004",
                    1.509
                ],
                [
                    "2005",
                    1.357
                ],
                [
                    "2006",
                    1.308
                ],
                [
                    "2007",
                    1.567
                ],
                [
                    "2008",
                    1.359
                ],
                [
                    "2009",
                    1.373
                ],
                [
                    "2010",
                    1.271
                ],
                [
                    "2011",
                    1.362
                ],
                [
                    "2012",
                    1.256
                ],
                [
                    "2013",
                    1.229
                ],
                [
                    "2014",
                    1.294
                ],
                [
                    "2015",
                    1.485
                ],
                [
                    "2016",
                    1.388
                ],
                [
                    "2017",
                    1.383
                ],
                [
                    "2018",
                    1.204
                ],
                [
                    "2019",
                    1.356
                ],
                [
                    "2020",
                    1.158
                ],
                [
                    "2021",
                    1.4
                ],
                [
                    "2022",
                    1.29
                ]
            ],
            "hoverAnimation": true,
            "label": {
                "show": false,
                "margin": 8
            },
            "logBase": 10,
            "seriesLayoutBy": "column",
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0,
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "line",
            "name": "Europe",
            "connectNulls": false,
            "xAxisIndex": 0,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [
                [
                    "1993",
                    1.475
                ],
                [
                    "1994",
                    1.377
                ],
                [
                    "1995",
                    1.439
                ],
                [
                    "1996",
                    1.349
                ],
                [
                    "1997",
                    1.341
                ],
                [
                    "1998",
                    1.377
                ],
                [
                    "1999",
                    1.333
                ],
                [
                    "2000",
                    1.229
                ],
                [
                    "2001",
                    1.516
                ],
                [
                    "2002",
                    1.299
                ],
                [
                    "2003",
                    1.41
                ],
                [
                    "2004",
                    1.414
                ],
                [
                    "2005",
                    1.368
                ],
                [
                    "2006",
                    1.371
                ],
                [
                    "2007",
                    1.264
                ],
                [
                    "2008",
                    1.292
                ],
                [
                    "2009",
                    1.337
                ],
                [
                    "2010",
                    1.283
                ],
                [
                    "2011",
                    1.264
                ],
                [
                    "2012",
                    1.352
                ],
                [
                    "2013",
                    1.41
                ],
                [
                    "2014",
                    1.276
                ],
                [
                    "2015",
                    1.249
                ],
                [
                    "2016",
                    1.343
                ],
                [
                    "2017",
                    1.445
                ],
                [
                    "2018",
                    1.248
                ],
                [
                    "2019",
                    1.563
                ],
                [
                    "2020",
                    1.201
                ],
                [
                    "2021",
                    1.475
                ],
                [
                    "2022",
                    1.235
                ]
            ],
            "hoverAnimation": true,
            "label": {
                "show": false,
                "margin": 8
            },
            "logBase": 10,
            "seriesLayoutBy": "column",
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0,
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        }
    ],
    "legend": [
        {
            "data": [
                "Asia",
                "Europe"
            ],
            "selected": {},
            "show": true,
            "padding": 5,
            "itemGap": 10,
            "itemWidth": 25,
            "itemHeight": 14,
            "backgroundColor": "transparent",
            "borderColor": "#ccc",
            "borderWidth": 1,
            "borderRadius": 0,
            "pageButtonItemGap": 5,
            "pageButtonPosition": "end",
            "pageFormatter": "{current}/{total}",
            "pageIconColor": "#2f4554",
            "pageIconInactiveColor": "#aaa",
            "pageIconSize": 15,
            "animationDurationUpdate": 800,
            "selector": false,
            "selectorPosition": "auto",
            "selectorItemGap": 7,
            "selectorButtonGap": 10
        }
    ],
    "tooltip": {
        "show": true,
        "trigger": "item",
        "triggerOn": "mousemove|click",
        "axisPointer": {
            "type": "line"
        },
        "showContent": true,
        "alwaysShowContent": false,
        "showDelay": 0,
        "hideDelay": 100,
        "enterable": false,
        "confine": false,
        "appendToBody": false,
        "transitionDuration": 0.4,
        "textStyle": {
            "fontSize": 14
        },
        "borderWidth": 0,
        "padding": 5,
        "order": "seriesAsc"
    },
    "xAxis": [
        {
            "type": "category",
            "name": "date",
            "show": true,
            "scale": false,
            "nameLocation": "end",
            "nameGap": 15,
            "gridIndex": 0,
            "inverse": false,
            "offset": 0,
            "splitNumber": 5,
            "minInterval": 0,
            "splitLine": {
                "show": true,
                "lineStyle": {
                    "show": true,
                    "width": 1,
                    "opacity": 1,
                    "curveness": 0,
                    "type": "solid"
                }
            },
            "data": [
                "1993",
                "1994",
                "1995",
                "1996",
                "1997",
                "1998",
                "1999",
                "2000",
                "2001",
                "2002",
                "2003",
                "2004",
                "2005",
                "2006",
                "2007",
                "2008",
                "2009",
                "2010",
                "2011",
                "2012",
                "2013",
                "2014",
                "2015",
                "2016",
                "2017",
                "2018",
                "2019",
                "2020",
                "2021",
                "2022"
            ]
        }
    ],
    "yAxis": [
        {
            "type": "value",
            "show": true,
            "scale": false,
            "nameLocation": "end",
            "nameGap": 15,
            "gridIndex": 0,
            "inverse": false,
            "offset": 0,
            "splitNumber": 5,
            "minInterval": 0,
            "splitLine": {
                "show": true,
                "lineStyle": {
                    "show": true,
                    "width": 1,
                    "opacity": 1,
                    "curveness": 0,
                    "type": "solid"
                }
            }
        }
    ],
    "title": [
        {
            "show": true,
            "target": "blank",
            "subtarget": "blank",
            "padding": 5,
            "itemGap": 10,
            "textAlign": "auto",
            "textVerticalAlign": "auto",
            "triggerEvent": false
        }
    ]
};
                chart_1126f76f09c5480cb3a53bffe8483102.setOption(option_1126f76f09c5480cb3a53bffe8483102);
        });
    </script>





```python
from pandasecharts import echart
df_pivot.reset_index().echart.line(x='일자', ys=['액면가', '시장가'], xtype='category').render_notebook()
```





<script>
    require.config({
        paths: {
            'echarts':'https://assets.pyecharts.org/assets/v5/echarts.min'
        }
    });
</script>

        <div id="e088725c503c4ba483d57c6168494635" style="width:900px; height:500px;"></div>

<script>
        require(['echarts'], function(echarts) {
                var chart_e088725c503c4ba483d57c6168494635 = echarts.init(
                    document.getElementById('e088725c503c4ba483d57c6168494635'), 'white', {renderer: 'canvas'});
                var option_e088725c503c4ba483d57c6168494635 = {
    "animation": true,
    "animationThreshold": 2000,
    "animationDuration": 1000,
    "animationEasing": "cubicOut",
    "animationDelay": 0,
    "animationDurationUpdate": 300,
    "animationEasingUpdate": "cubicOut",
    "animationDelayUpdate": 0,
    "aria": {
        "enabled": false
    },
    "color": [
        "#5470c6",
        "#91cc75",
        "#fac858",
        "#ee6666",
        "#73c0de",
        "#3ba272",
        "#fc8452",
        "#9a60b4",
        "#ea7ccc"
    ],
    "series": [
        {
            "type": "line",
            "name": "\uc561\uba74\uac00",
            "connectNulls": false,
            "xAxisIndex": 0,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [
                [
                    "2023-01-01",
                    6732328
                ],
                [
                    "2023-01-02",
                    6722595
                ],
                [
                    "2023-01-03",
                    6870190
                ],
                [
                    "2023-01-04",
                    7059499
                ],
                [
                    "2023-01-05",
                    7130709
                ],
                [
                    "2023-01-06",
                    7265437
                ],
                [
                    "2023-01-07",
                    7265437
                ],
                [
                    "2023-01-08",
                    7265437
                ],
                [
                    "2023-01-09",
                    7401866
                ],
                [
                    "2023-01-10",
                    7411745
                ],
                [
                    "2023-01-11",
                    7448034
                ],
                [
                    "2023-01-12",
                    7575520
                ],
                [
                    "2023-01-13",
                    7741946
                ],
                [
                    "2023-01-14",
                    7741946
                ],
                [
                    "2023-01-15",
                    7741946
                ],
                [
                    "2023-01-16",
                    7768080
                ],
                [
                    "2023-01-17",
                    7834403
                ],
                [
                    "2023-01-18",
                    7855970
                ],
                [
                    "2023-01-19",
                    7932067
                ],
                [
                    "2023-01-20",
                    7879866
                ],
                [
                    "2023-01-21",
                    7881206
                ],
                [
                    "2023-01-22",
                    7854927
                ],
                [
                    "2023-01-23",
                    7854927
                ],
                [
                    "2023-01-24",
                    7834815
                ],
                [
                    "2023-01-25",
                    8152047
                ],
                [
                    "2023-01-26",
                    8156847
                ],
                [
                    "2023-01-27",
                    8149974
                ],
                [
                    "2023-01-28",
                    8149974
                ],
                [
                    "2023-01-29",
                    8149974
                ],
                [
                    "2023-01-30",
                    8212180
                ],
                [
                    "2023-01-31",
                    8236473
                ],
                [
                    "2023-02-01",
                    8271774
                ],
                [
                    "2023-02-02",
                    8201291
                ],
                [
                    "2023-02-03",
                    8172253
                ],
                [
                    "2023-02-04",
                    8172253
                ],
                [
                    "2023-02-05",
                    8038173
                ],
                [
                    "2023-02-06",
                    8045709
                ],
                [
                    "2023-02-07",
                    8047424
                ],
                [
                    "2023-02-08",
                    8098010
                ],
                [
                    "2023-02-09",
                    8102510
                ],
                [
                    "2023-02-10",
                    8035046
                ],
                [
                    "2023-02-11",
                    8035046
                ],
                [
                    "2023-02-12",
                    8008230
                ],
                [
                    "2023-02-13",
                    7993206
                ],
                [
                    "2023-02-14",
                    8004142
                ],
                [
                    "2023-02-15",
                    8097527
                ],
                [
                    "2023-02-16",
                    8029928
                ],
                [
                    "2023-02-17",
                    8195643
                ],
                [
                    "2023-02-18",
                    8195643
                ],
                [
                    "2023-02-19",
                    8195643
                ],
                [
                    "2023-02-20",
                    8138738
                ],
                [
                    "2023-02-21",
                    8145176
                ],
                [
                    "2023-02-22",
                    8117058
                ],
                [
                    "2023-02-23",
                    8160073
                ],
                [
                    "2023-02-24",
                    8257072
                ],
                [
                    "2023-02-25",
                    8257072
                ],
                [
                    "2023-02-26",
                    8203440
                ],
                [
                    "2023-02-27",
                    8363306
                ],
                [
                    "2023-02-28",
                    8330134
                ],
                [
                    "2023-03-01",
                    8330134
                ],
                [
                    "2023-03-02",
                    8352299
                ],
                [
                    "2023-03-03",
                    8185503
                ],
                [
                    "2023-03-04",
                    8185503
                ],
                [
                    "2023-03-05",
                    8185503
                ],
                [
                    "2023-03-06",
                    8170496
                ],
                [
                    "2023-03-07",
                    8201730
                ],
                [
                    "2023-03-08",
                    8142214
                ],
                [
                    "2023-03-09",
                    8174050
                ],
                [
                    "2023-03-10",
                    8030593
                ],
                [
                    "2023-03-11",
                    8030593
                ],
                [
                    "2023-03-12",
                    8030593
                ],
                [
                    "2023-03-13",
                    7821927
                ],
                [
                    "2023-03-14",
                    7889732
                ],
                [
                    "2023-03-15",
                    7904486
                ],
                [
                    "2023-03-16",
                    7787305
                ],
                [
                    "2023-03-17",
                    7900846
                ],
                [
                    "2023-03-18",
                    7900846
                ],
                [
                    "2023-03-19",
                    7900846
                ],
                [
                    "2023-03-20",
                    7892090
                ],
                [
                    "2023-03-21",
                    7909475
                ],
                [
                    "2023-03-22",
                    7923750
                ],
                [
                    "2023-03-23",
                    7882188
                ],
                [
                    "2023-03-24",
                    7867575
                ],
                [
                    "2023-03-25",
                    7867567
                ],
                [
                    "2023-03-26",
                    7867567
                ],
                [
                    "2023-03-27",
                    7898051
                ],
                [
                    "2023-03-28",
                    7874454
                ],
                [
                    "2023-03-29",
                    7826187
                ],
                [
                    "2023-03-30",
                    7822707
                ],
                [
                    "2023-03-31",
                    7576494
                ],
                [
                    "2023-04-01",
                    7597253
                ],
                [
                    "2023-04-02",
                    7570437
                ],
                [
                    "2023-04-03",
                    7526832
                ],
                [
                    "2023-04-04",
                    7599696
                ],
                [
                    "2023-04-05",
                    7695714
                ],
                [
                    "2023-04-06",
                    7825625
                ],
                [
                    "2023-04-07",
                    7765328
                ],
                [
                    "2023-04-08",
                    7765328
                ],
                [
                    "2023-04-09",
                    7765328
                ],
                [
                    "2023-04-10",
                    7753080
                ],
                [
                    "2023-04-11",
                    7732005
                ],
                [
                    "2023-04-12",
                    7603553
                ],
                [
                    "2023-04-13",
                    7658981
                ],
                [
                    "2023-04-14",
                    7691936
                ],
                [
                    "2023-04-15",
                    7691936
                ],
                [
                    "2023-04-16",
                    7691936
                ],
                [
                    "2023-04-17",
                    7776241
                ],
                [
                    "2023-04-18",
                    7812790
                ],
                [
                    "2023-04-19",
                    7849779
                ],
                [
                    "2023-04-20",
                    7956205
                ],
                [
                    "2023-04-21",
                    7953572
                ],
                [
                    "2023-04-22",
                    7953572
                ],
                [
                    "2023-04-23",
                    7953572
                ],
                [
                    "2023-04-24",
                    7924645
                ],
                [
                    "2023-04-25",
                    7976033
                ],
                [
                    "2023-04-26",
                    7890332
                ],
                [
                    "2023-04-27",
                    8054480
                ],
                [
                    "2023-04-28",
                    7753674
                ],
                [
                    "2023-04-29",
                    7753674
                ],
                [
                    "2023-04-30",
                    7732906
                ],
                [
                    "2023-05-01",
                    7751074
                ],
                [
                    "2023-05-02",
                    7842336
                ],
                [
                    "2023-05-03",
                    8018568
                ],
                [
                    "2023-05-04",
                    7873723
                ],
                [
                    "2023-05-05",
                    7875036
                ],
                [
                    "2023-05-06",
                    7875036
                ],
                [
                    "2023-05-07",
                    7875036
                ],
                [
                    "2023-05-08",
                    7993478
                ],
                [
                    "2023-05-09",
                    7935676
                ],
                [
                    "2023-05-10",
                    7859803
                ],
                [
                    "2023-05-11",
                    7974388
                ],
                [
                    "2023-05-12",
                    7949043
                ],
                [
                    "2023-05-13",
                    7949579
                ],
                [
                    "2023-05-14",
                    7948239
                ],
                [
                    "2023-05-15",
                    7988844
                ],
                [
                    "2023-05-16",
                    8096461
                ],
                [
                    "2023-05-17",
                    8099691
                ],
                [
                    "2023-05-18",
                    8011905
                ],
                [
                    "2023-05-19",
                    7964805
                ],
                [
                    "2023-05-20",
                    7964805
                ],
                [
                    "2023-05-21",
                    7951397
                ],
                [
                    "2023-05-22",
                    8028192
                ],
                [
                    "2023-05-23",
                    8022904
                ],
                [
                    "2023-05-24",
                    8116069
                ],
                [
                    "2023-05-25",
                    8048855
                ],
                [
                    "2023-05-26",
                    7895959
                ],
                [
                    "2023-05-27",
                    7897365
                ],
                [
                    "2023-05-28",
                    7870549
                ],
                [
                    "2023-05-29",
                    7870549
                ],
                [
                    "2023-05-30",
                    7830781
                ],
                [
                    "2023-05-31",
                    7765620
                ],
                [
                    "2023-06-01",
                    7905033
                ],
                [
                    "2023-06-02",
                    7777608
                ],
                [
                    "2023-06-03",
                    7777608
                ],
                [
                    "2023-06-04",
                    7764200
                ],
                [
                    "2023-06-05",
                    7805493
                ],
                [
                    "2023-06-06",
                    7805493
                ],
                [
                    "2023-06-07",
                    7771304
                ],
                [
                    "2023-06-08",
                    7767722
                ],
                [
                    "2023-06-09",
                    7740774
                ],
                [
                    "2023-06-10",
                    7740774
                ],
                [
                    "2023-06-11",
                    7727366
                ],
                [
                    "2023-06-12",
                    7706878
                ],
                [
                    "2023-06-13",
                    7743443
                ],
                [
                    "2023-06-14",
                    7365044
                ],
                [
                    "2023-06-15",
                    7334915
                ],
                [
                    "2023-06-16",
                    7384395
                ],
                [
                    "2023-06-17",
                    7384395
                ],
                [
                    "2023-06-18",
                    7370987
                ],
                [
                    "2023-06-19",
                    7454644
                ],
                [
                    "2023-06-20",
                    7572565
                ],
                [
                    "2023-06-21",
                    7621847
                ],
                [
                    "2023-06-22",
                    7683890
                ],
                [
                    "2023-06-23",
                    7504162
                ],
                [
                    "2023-06-24",
                    7504162
                ],
                [
                    "2023-06-25",
                    7504162
                ],
                [
                    "2023-06-26",
                    7584710
                ],
                [
                    "2023-06-27",
                    7413252
                ],
                [
                    "2023-06-28",
                    7464718
                ],
                [
                    "2023-06-29",
                    7581121
                ],
                [
                    "2023-06-30",
                    7360874
                ]
            ],
            "hoverAnimation": true,
            "label": {
                "show": false,
                "margin": 8
            },
            "logBase": 10,
            "seriesLayoutBy": "column",
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0,
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "line",
            "name": "\uc2dc\uc7a5\uac00",
            "connectNulls": false,
            "xAxisIndex": 0,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [
                [
                    "2023-01-01",
                    6814690
                ],
                [
                    "2023-01-02",
                    6805099
                ],
                [
                    "2023-01-03",
                    6959289
                ],
                [
                    "2023-01-04",
                    7151938
                ],
                [
                    "2023-01-05",
                    7225212
                ],
                [
                    "2023-01-06",
                    7358230
                ],
                [
                    "2023-01-07",
                    7359121
                ],
                [
                    "2023-01-08",
                    7360006
                ],
                [
                    "2023-01-09",
                    7473023
                ],
                [
                    "2023-01-10",
                    7480899
                ],
                [
                    "2023-01-11",
                    7519011
                ],
                [
                    "2023-01-12",
                    7648738
                ],
                [
                    "2023-01-13",
                    7817258
                ],
                [
                    "2023-01-14",
                    7818173
                ],
                [
                    "2023-01-15",
                    7819092
                ],
                [
                    "2023-01-16",
                    7841325
                ],
                [
                    "2023-01-17",
                    7904631
                ],
                [
                    "2023-01-18",
                    7937397
                ],
                [
                    "2023-01-19",
                    8019982
                ],
                [
                    "2023-01-20",
                    7962933
                ],
                [
                    "2023-01-21",
                    7965179
                ],
                [
                    "2023-01-22",
                    7939797
                ],
                [
                    "2023-01-23",
                    7940717
                ],
                [
                    "2023-01-24",
                    7921525
                ],
                [
                    "2023-01-25",
                    8223751
                ],
                [
                    "2023-01-26",
                    8229064
                ],
                [
                    "2023-01-27",
                    8222428
                ],
                [
                    "2023-01-28",
                    8223356
                ],
                [
                    "2023-01-29",
                    8224273
                ],
                [
                    "2023-01-30",
                    8288971
                ],
                [
                    "2023-01-31",
                    8305537
                ],
                [
                    "2023-02-01",
                    8352721
                ],
                [
                    "2023-02-02",
                    8286963
                ],
                [
                    "2023-02-03",
                    8261826
                ],
                [
                    "2023-02-04",
                    8262701
                ],
                [
                    "2023-02-05",
                    8129082
                ],
                [
                    "2023-02-06",
                    8130580
                ],
                [
                    "2023-02-07",
                    8137215
                ],
                [
                    "2023-02-08",
                    8190029
                ],
                [
                    "2023-02-09",
                    8193118
                ],
                [
                    "2023-02-10",
                    8154757
                ],
                [
                    "2023-02-11",
                    8155611
                ],
                [
                    "2023-02-12",
                    8129588
                ],
                [
                    "2023-02-13",
                    8109420
                ],
                [
                    "2023-02-14",
                    8133722
                ],
                [
                    "2023-02-15",
                    8253970
                ],
                [
                    "2023-02-16",
                    8186732
                ],
                [
                    "2023-02-17",
                    8338429
                ],
                [
                    "2023-02-18",
                    8339343
                ],
                [
                    "2023-02-19",
                    8340260
                ],
                [
                    "2023-02-20",
                    8274601
                ],
                [
                    "2023-02-21",
                    8283428
                ],
                [
                    "2023-02-22",
                    8248538
                ],
                [
                    "2023-02-23",
                    8318952
                ],
                [
                    "2023-02-24",
                    8386368
                ],
                [
                    "2023-02-25",
                    8387285
                ],
                [
                    "2023-02-26",
                    8334588
                ],
                [
                    "2023-02-27",
                    8462450
                ],
                [
                    "2023-02-28",
                    8369279
                ],
                [
                    "2023-03-01",
                    8373500
                ],
                [
                    "2023-03-02",
                    8393457
                ],
                [
                    "2023-03-03",
                    8241986
                ],
                [
                    "2023-03-04",
                    8242926
                ],
                [
                    "2023-03-05",
                    8243842
                ],
                [
                    "2023-03-06",
                    8250786
                ],
                [
                    "2023-03-07",
                    8282942
                ],
                [
                    "2023-03-08",
                    8302630
                ],
                [
                    "2023-03-09",
                    8339693
                ],
                [
                    "2023-03-10",
                    8200483
                ],
                [
                    "2023-03-11",
                    8201400
                ],
                [
                    "2023-03-12",
                    8202299
                ],
                [
                    "2023-03-13",
                    8062223
                ],
                [
                    "2023-03-14",
                    8131560
                ],
                [
                    "2023-03-15",
                    8142135
                ],
                [
                    "2023-03-16",
                    8029829
                ],
                [
                    "2023-03-17",
                    8144966
                ],
                [
                    "2023-03-18",
                    8145841
                ],
                [
                    "2023-03-19",
                    8146687
                ],
                [
                    "2023-03-20",
                    8140692
                ],
                [
                    "2023-03-21",
                    8158104
                ],
                [
                    "2023-03-22",
                    8174556
                ],
                [
                    "2023-03-23",
                    8144171
                ],
                [
                    "2023-03-24",
                    8129480
                ],
                [
                    "2023-03-25",
                    8118253
                ],
                [
                    "2023-03-26",
                    8119077
                ],
                [
                    "2023-03-27",
                    8146642
                ],
                [
                    "2023-03-28",
                    8125402
                ],
                [
                    "2023-03-29",
                    8073436
                ],
                [
                    "2023-03-30",
                    8071512
                ],
                [
                    "2023-03-31",
                    7844367
                ],
                [
                    "2023-04-01",
                    7846476
                ],
                [
                    "2023-04-02",
                    7820386
                ],
                [
                    "2023-04-03",
                    7773243
                ],
                [
                    "2023-04-04",
                    7849732
                ],
                [
                    "2023-04-05",
                    7936009
                ],
                [
                    "2023-04-06",
                    8058077
                ],
                [
                    "2023-04-07",
                    7964819
                ],
                [
                    "2023-04-08",
                    7965597
                ],
                [
                    "2023-04-09",
                    7966398
                ],
                [
                    "2023-04-10",
                    7956217
                ],
                [
                    "2023-04-11",
                    7942108
                ],
                [
                    "2023-04-12",
                    7812272
                ],
                [
                    "2023-04-13",
                    7871128
                ],
                [
                    "2023-04-14",
                    7900478
                ],
                [
                    "2023-04-15",
                    7901267
                ],
                [
                    "2023-04-16",
                    7902053
                ],
                [
                    "2023-04-17",
                    7956055
                ],
                [
                    "2023-04-18",
                    7990385
                ],
                [
                    "2023-04-19",
                    8030259
                ],
                [
                    "2023-04-20",
                    8132366
                ],
                [
                    "2023-04-21",
                    8144324
                ],
                [
                    "2023-04-22",
                    8145141
                ],
                [
                    "2023-04-23",
                    8145960
                ],
                [
                    "2023-04-24",
                    8139203
                ],
                [
                    "2023-04-25",
                    8188350
                ],
                [
                    "2023-04-26",
                    8102207
                ],
                [
                    "2023-04-27",
                    8263616
                ],
                [
                    "2023-04-28",
                    7960582
                ],
                [
                    "2023-04-29",
                    7961396
                ],
                [
                    "2023-04-30",
                    7963288
                ],
                [
                    "2023-05-01",
                    7960371
                ],
                [
                    "2023-05-02",
                    8048802
                ],
                [
                    "2023-05-03",
                    8229601
                ],
                [
                    "2023-05-04",
                    8085840
                ],
                [
                    "2023-05-05",
                    8087977
                ],
                [
                    "2023-05-06",
                    8088798
                ],
                [
                    "2023-05-07",
                    8089610
                ],
                [
                    "2023-05-08",
                    8202622
                ],
                [
                    "2023-05-09",
                    8142515
                ],
                [
                    "2023-05-10",
                    8090104
                ],
                [
                    "2023-05-11",
                    8134108
                ],
                [
                    "2023-05-12",
                    8108977
                ],
                [
                    "2023-05-13",
                    8110326
                ],
                [
                    "2023-05-14",
                    8109797
                ],
                [
                    "2023-05-15",
                    8164958
                ],
                [
                    "2023-05-16",
                    8247696
                ],
                [
                    "2023-05-17",
                    8220665
                ],
                [
                    "2023-05-18",
                    8170727
                ],
                [
                    "2023-05-19",
                    8134163
                ],
                [
                    "2023-05-20",
                    8135030
                ],
                [
                    "2023-05-21",
                    8122472
                ],
                [
                    "2023-05-22",
                    8201294
                ],
                [
                    "2023-05-23",
                    8185568
                ],
                [
                    "2023-05-24",
                    8272101
                ],
                [
                    "2023-05-25",
                    8210316
                ],
                [
                    "2023-05-26",
                    8045944
                ],
                [
                    "2023-05-27",
                    8048263
                ],
                [
                    "2023-05-28",
                    8022190
                ],
                [
                    "2023-05-29",
                    8023059
                ],
                [
                    "2023-05-30",
                    7983365
                ],
                [
                    "2023-05-31",
                    7947077
                ],
                [
                    "2023-06-01",
                    8054866
                ],
                [
                    "2023-06-02",
                    7931363
                ],
                [
                    "2023-06-03",
                    7932223
                ],
                [
                    "2023-06-04",
                    7919593
                ],
                [
                    "2023-06-05",
                    7954556
                ],
                [
                    "2023-06-06",
                    7955421
                ],
                [
                    "2023-06-07",
                    7920292
                ],
                [
                    "2023-06-08",
                    7938165
                ],
                [
                    "2023-06-09",
                    7907500
                ],
                [
                    "2023-06-10",
                    7907518
                ],
                [
                    "2023-06-11",
                    7894905
                ],
                [
                    "2023-06-12",
                    7873276
                ],
                [
                    "2023-06-13",
                    7910702
                ],
                [
                    "2023-06-14",
                    7528212
                ],
                [
                    "2023-06-15",
                    7577327
                ],
                [
                    "2023-06-16",
                    7628585
                ],
                [
                    "2023-06-17",
                    7629415
                ],
                [
                    "2023-06-18",
                    7616609
                ],
                [
                    "2023-06-19",
                    7695267
                ],
                [
                    "2023-06-20",
                    7815296
                ],
                [
                    "2023-06-21",
                    7866436
                ],
                [
                    "2023-06-22",
                    7930199
                ],
                [
                    "2023-06-23",
                    7750178
                ],
                [
                    "2023-06-24",
                    7751034
                ],
                [
                    "2023-06-25",
                    7751895
                ],
                [
                    "2023-06-26",
                    7835584
                ],
                [
                    "2023-06-27",
                    7662062
                ],
                [
                    "2023-06-28",
                    7710832
                ],
                [
                    "2023-06-29",
                    7826814
                ],
                [
                    "2023-06-30",
                    7604406
                ]
            ],
            "hoverAnimation": true,
            "label": {
                "show": false,
                "margin": 8
            },
            "logBase": 10,
            "seriesLayoutBy": "column",
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0,
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        }
    ],
    "legend": [
        {
            "data": [
                "\uc561\uba74\uac00",
                "\uc2dc\uc7a5\uac00"
            ],
            "selected": {},
            "show": true,
            "padding": 5,
            "itemGap": 10,
            "itemWidth": 25,
            "itemHeight": 14,
            "backgroundColor": "transparent",
            "borderColor": "#ccc",
            "borderWidth": 1,
            "borderRadius": 0,
            "pageButtonItemGap": 5,
            "pageButtonPosition": "end",
            "pageFormatter": "{current}/{total}",
            "pageIconColor": "#2f4554",
            "pageIconInactiveColor": "#aaa",
            "pageIconSize": 15,
            "animationDurationUpdate": 800,
            "selector": false,
            "selectorPosition": "auto",
            "selectorItemGap": 7,
            "selectorButtonGap": 10
        }
    ],
    "tooltip": {
        "show": true,
        "trigger": "item",
        "triggerOn": "mousemove|click",
        "axisPointer": {
            "type": "line"
        },
        "showContent": true,
        "alwaysShowContent": false,
        "showDelay": 0,
        "hideDelay": 100,
        "enterable": false,
        "confine": false,
        "appendToBody": false,
        "transitionDuration": 0.4,
        "textStyle": {
            "fontSize": 14
        },
        "borderWidth": 0,
        "padding": 5,
        "order": "seriesAsc"
    },
    "xAxis": [
        {
            "type": "category",
            "name": "\uc77c\uc790",
            "show": true,
            "scale": false,
            "nameLocation": "end",
            "nameGap": 15,
            "gridIndex": 0,
            "inverse": false,
            "offset": 0,
            "splitNumber": 5,
            "minInterval": 0,
            "splitLine": {
                "show": true,
                "lineStyle": {
                    "show": true,
                    "width": 1,
                    "opacity": 1,
                    "curveness": 0,
                    "type": "solid"
                }
            },
            "data": [
                "2023-01-01",
                "2023-01-02",
                "2023-01-03",
                "2023-01-04",
                "2023-01-05",
                "2023-01-06",
                "2023-01-07",
                "2023-01-08",
                "2023-01-09",
                "2023-01-10",
                "2023-01-11",
                "2023-01-12",
                "2023-01-13",
                "2023-01-14",
                "2023-01-15",
                "2023-01-16",
                "2023-01-17",
                "2023-01-18",
                "2023-01-19",
                "2023-01-20",
                "2023-01-21",
                "2023-01-22",
                "2023-01-23",
                "2023-01-24",
                "2023-01-25",
                "2023-01-26",
                "2023-01-27",
                "2023-01-28",
                "2023-01-29",
                "2023-01-30",
                "2023-01-31",
                "2023-02-01",
                "2023-02-02",
                "2023-02-03",
                "2023-02-04",
                "2023-02-05",
                "2023-02-06",
                "2023-02-07",
                "2023-02-08",
                "2023-02-09",
                "2023-02-10",
                "2023-02-11",
                "2023-02-12",
                "2023-02-13",
                "2023-02-14",
                "2023-02-15",
                "2023-02-16",
                "2023-02-17",
                "2023-02-18",
                "2023-02-19",
                "2023-02-20",
                "2023-02-21",
                "2023-02-22",
                "2023-02-23",
                "2023-02-24",
                "2023-02-25",
                "2023-02-26",
                "2023-02-27",
                "2023-02-28",
                "2023-03-01",
                "2023-03-02",
                "2023-03-03",
                "2023-03-04",
                "2023-03-05",
                "2023-03-06",
                "2023-03-07",
                "2023-03-08",
                "2023-03-09",
                "2023-03-10",
                "2023-03-11",
                "2023-03-12",
                "2023-03-13",
                "2023-03-14",
                "2023-03-15",
                "2023-03-16",
                "2023-03-17",
                "2023-03-18",
                "2023-03-19",
                "2023-03-20",
                "2023-03-21",
                "2023-03-22",
                "2023-03-23",
                "2023-03-24",
                "2023-03-25",
                "2023-03-26",
                "2023-03-27",
                "2023-03-28",
                "2023-03-29",
                "2023-03-30",
                "2023-03-31",
                "2023-04-01",
                "2023-04-02",
                "2023-04-03",
                "2023-04-04",
                "2023-04-05",
                "2023-04-06",
                "2023-04-07",
                "2023-04-08",
                "2023-04-09",
                "2023-04-10",
                "2023-04-11",
                "2023-04-12",
                "2023-04-13",
                "2023-04-14",
                "2023-04-15",
                "2023-04-16",
                "2023-04-17",
                "2023-04-18",
                "2023-04-19",
                "2023-04-20",
                "2023-04-21",
                "2023-04-22",
                "2023-04-23",
                "2023-04-24",
                "2023-04-25",
                "2023-04-26",
                "2023-04-27",
                "2023-04-28",
                "2023-04-29",
                "2023-04-30",
                "2023-05-01",
                "2023-05-02",
                "2023-05-03",
                "2023-05-04",
                "2023-05-05",
                "2023-05-06",
                "2023-05-07",
                "2023-05-08",
                "2023-05-09",
                "2023-05-10",
                "2023-05-11",
                "2023-05-12",
                "2023-05-13",
                "2023-05-14",
                "2023-05-15",
                "2023-05-16",
                "2023-05-17",
                "2023-05-18",
                "2023-05-19",
                "2023-05-20",
                "2023-05-21",
                "2023-05-22",
                "2023-05-23",
                "2023-05-24",
                "2023-05-25",
                "2023-05-26",
                "2023-05-27",
                "2023-05-28",
                "2023-05-29",
                "2023-05-30",
                "2023-05-31",
                "2023-06-01",
                "2023-06-02",
                "2023-06-03",
                "2023-06-04",
                "2023-06-05",
                "2023-06-06",
                "2023-06-07",
                "2023-06-08",
                "2023-06-09",
                "2023-06-10",
                "2023-06-11",
                "2023-06-12",
                "2023-06-13",
                "2023-06-14",
                "2023-06-15",
                "2023-06-16",
                "2023-06-17",
                "2023-06-18",
                "2023-06-19",
                "2023-06-20",
                "2023-06-21",
                "2023-06-22",
                "2023-06-23",
                "2023-06-24",
                "2023-06-25",
                "2023-06-26",
                "2023-06-27",
                "2023-06-28",
                "2023-06-29",
                "2023-06-30"
            ]
        }
    ],
    "yAxis": [
        {
            "type": "value",
            "show": true,
            "scale": false,
            "nameLocation": "end",
            "nameGap": 15,
            "gridIndex": 0,
            "inverse": false,
            "offset": 0,
            "splitNumber": 5,
            "minInterval": 0,
            "splitLine": {
                "show": true,
                "lineStyle": {
                    "show": true,
                    "width": 1,
                    "opacity": 1,
                    "curveness": 0,
                    "type": "solid"
                }
            }
        }
    ],
    "title": [
        {
            "show": true,
            "target": "blank",
            "subtarget": "blank",
            "padding": 5,
            "itemGap": 10,
            "textAlign": "auto",
            "textVerticalAlign": "auto",
            "triggerEvent": false
        }
    ]
};
                chart_e088725c503c4ba483d57c6168494635.setOption(option_e088725c503c4ba483d57c6168494635);
        });
    </script>





```python
# 일자별 시장가와 액면가의 차이 구하기 (일자별 시장가 - 액면가)
df_pivot['시장가'].sub(df_pivot['액면가'])
```




    일자
    2023-01-01     82362
    2023-01-02     82504
    2023-01-03     89099
                   ...  
    2023-06-28    246114
    2023-06-29    245693
    2023-06-30    243532
    Length: 181, dtype: int64




```python
# 일자별 시장가와 액면가의 차이를 그래프로 그리기 (.plot(lw=3, colormap='Set2'))
df_pivot['시장가'].sub(df_pivot['액면가']).plot(lw=3, colormap='Set2')
```




    <Axes: xlabel='일자'>




    
![png](output_74_1.png)
    



```python
# 일자별 시장가와 액면가의 차이를 비율로 구하기 (일자별 시장가 - 액면가에서 시장가 나누어 비율 구하여 그래프)
df_pivot['시장가'].sub(df_pivot['액면가']).div(df_pivot['액면가']).plot(lw=3, colormap='Set2')
```




    <Axes: xlabel='일자'>




    
![png](output_75_1.png)
    


- 각 쿼터의 일별 시장가의 추이 집계하기


```python
# df_bond에서 quarter만 구분해서 quarter열 만들기
df_bond['quarter'] = df_bond['일자'].dt.to_period(freq='Q')
df_bond
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
      <th>일자</th>
      <th>종목코드</th>
      <th>팀점코드</th>
      <th>펀드코드</th>
      <th>통화</th>
      <th>발행자등급</th>
      <th>종목등급</th>
      <th>환율</th>
      <th>액면가</th>
      <th>시장가</th>
      <th>상품분류</th>
      <th>quarter</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2023-01-01</td>
      <td>KR310607AC75797</td>
      <td>4120500</td>
      <td>L0050797</td>
      <td>KRW</td>
      <td>AAA</td>
      <td>AAA</td>
      <td>NaN</td>
      <td>53632</td>
      <td>52963</td>
      <td>NaN</td>
      <td>2023Q1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2023-01-01</td>
      <td>KR310202AC73824</td>
      <td>4120500</td>
      <td>L0050797</td>
      <td>KRW</td>
      <td>AAA</td>
      <td>AAA</td>
      <td>NaN</td>
      <td>268160</td>
      <td>262526</td>
      <td>NaN</td>
      <td>2023Q1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2023-01-01</td>
      <td>KR310203AC82377</td>
      <td>4120500</td>
      <td>L0050797</td>
      <td>KRW</td>
      <td>AAA</td>
      <td>AAA</td>
      <td>NaN</td>
      <td>67040</td>
      <td>66349</td>
      <td>NaN</td>
      <td>2023Q1</td>
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
    </tr>
    <tr>
      <th>168619</th>
      <td>2023-06-30</td>
      <td>HW9999900031763</td>
      <td>9999999</td>
      <td>S1090987</td>
      <td>KRW</td>
      <td>AA-</td>
      <td>AA-</td>
      <td>1.00</td>
      <td>5706</td>
      <td>6016</td>
      <td>CD/CP</td>
      <td>2023Q2</td>
    </tr>
    <tr>
      <th>168620</th>
      <td>2023-06-30</td>
      <td>HW9999900036461</td>
      <td>9999999</td>
      <td>S1090987</td>
      <td>KRW</td>
      <td>AA-</td>
      <td>AA-</td>
      <td>1.00</td>
      <td>6393</td>
      <td>6093</td>
      <td>CD/CP</td>
      <td>2023Q2</td>
    </tr>
    <tr>
      <th>168621</th>
      <td>2023-06-30</td>
      <td>HW9999900033824</td>
      <td>9999999</td>
      <td>S1090987</td>
      <td>KRW</td>
      <td>AA-</td>
      <td>AA-</td>
      <td>1.00</td>
      <td>6462</td>
      <td>6501</td>
      <td>CD/CP</td>
      <td>2023Q2</td>
    </tr>
  </tbody>
</table>
<p>168622 rows × 12 columns</p>
</div>




```python
# df_bond에서 해당 일자는 각 쿼터에서 몇번째 날짜인지를 반환하는 D+열 만들기
df_bond['D+'] = df_bond.groupby('quarter')['일자'].rank(method='dense').astype('int')
df_bond
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
      <th>일자</th>
      <th>종목코드</th>
      <th>팀점코드</th>
      <th>펀드코드</th>
      <th>통화</th>
      <th>발행자등급</th>
      <th>종목등급</th>
      <th>환율</th>
      <th>액면가</th>
      <th>시장가</th>
      <th>상품분류</th>
      <th>quarter</th>
      <th>D+</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2023-01-01</td>
      <td>KR310607AC75797</td>
      <td>4120500</td>
      <td>L0050797</td>
      <td>KRW</td>
      <td>AAA</td>
      <td>AAA</td>
      <td>NaN</td>
      <td>53632</td>
      <td>52963</td>
      <td>NaN</td>
      <td>2023Q1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2023-01-01</td>
      <td>KR310202AC73824</td>
      <td>4120500</td>
      <td>L0050797</td>
      <td>KRW</td>
      <td>AAA</td>
      <td>AAA</td>
      <td>NaN</td>
      <td>268160</td>
      <td>262526</td>
      <td>NaN</td>
      <td>2023Q1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2023-01-01</td>
      <td>KR310203AC82377</td>
      <td>4120500</td>
      <td>L0050797</td>
      <td>KRW</td>
      <td>AAA</td>
      <td>AAA</td>
      <td>NaN</td>
      <td>67040</td>
      <td>66349</td>
      <td>NaN</td>
      <td>2023Q1</td>
      <td>1</td>
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
    </tr>
    <tr>
      <th>168619</th>
      <td>2023-06-30</td>
      <td>HW9999900031763</td>
      <td>9999999</td>
      <td>S1090987</td>
      <td>KRW</td>
      <td>AA-</td>
      <td>AA-</td>
      <td>1.00</td>
      <td>5706</td>
      <td>6016</td>
      <td>CD/CP</td>
      <td>2023Q2</td>
      <td>91</td>
    </tr>
    <tr>
      <th>168620</th>
      <td>2023-06-30</td>
      <td>HW9999900036461</td>
      <td>9999999</td>
      <td>S1090987</td>
      <td>KRW</td>
      <td>AA-</td>
      <td>AA-</td>
      <td>1.00</td>
      <td>6393</td>
      <td>6093</td>
      <td>CD/CP</td>
      <td>2023Q2</td>
      <td>91</td>
    </tr>
    <tr>
      <th>168621</th>
      <td>2023-06-30</td>
      <td>HW9999900033824</td>
      <td>9999999</td>
      <td>S1090987</td>
      <td>KRW</td>
      <td>AA-</td>
      <td>AA-</td>
      <td>1.00</td>
      <td>6462</td>
      <td>6501</td>
      <td>CD/CP</td>
      <td>2023Q2</td>
      <td>91</td>
    </tr>
  </tbody>
</table>
<p>168622 rows × 13 columns</p>
</div>



D+열을 만들때 위의 경우에는 모든 날짜의 데이터가 존재하기에 
```python
.rank(method='dense')
```
코드로 D+열을 만들 수 있다.

다른 경우에는 추후 배울 시계열 함수들을 이용하는 것이 낫다


```python
# df_bond를 이용해 시장가를 일자별로 index는 D+ columns는 quarter로 합산해서 집계하기(pivot_table)
df_bond.pivot_table('시장가', index='D+', columns='quarter', aggfunc='sum')
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
      <th>quarter</th>
      <th>2023Q1</th>
      <th>2023Q2</th>
    </tr>
    <tr>
      <th>D+</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>6814690.00</td>
      <td>7846476.00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6805099.00</td>
      <td>7820386.00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>6959289.00</td>
      <td>7773243.00</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>89</th>
      <td>8071512.00</td>
      <td>7710832.00</td>
    </tr>
    <tr>
      <th>90</th>
      <td>7844367.00</td>
      <td>7826814.00</td>
    </tr>
    <tr>
      <th>91</th>
      <td>NaN</td>
      <td>7604406.00</td>
    </tr>
  </tbody>
</table>
<p>91 rows × 2 columns</p>
</div>




```python
# df_bond를 이용해 시장가를 일자별로 index는 D+ columns는 quarter로 합산해서 집계하기(groupby + unstack)
df_bond.groupby(['D+', 'quarter'])['시장가'].sum().unstack()
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
      <th>quarter</th>
      <th>2023Q1</th>
      <th>2023Q2</th>
    </tr>
    <tr>
      <th>D+</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>6814690.00</td>
      <td>7846476.00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6805099.00</td>
      <td>7820386.00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>6959289.00</td>
      <td>7773243.00</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>89</th>
      <td>8071512.00</td>
      <td>7710832.00</td>
    </tr>
    <tr>
      <th>90</th>
      <td>7844367.00</td>
      <td>7826814.00</td>
    </tr>
    <tr>
      <th>91</th>
      <td>NaN</td>
      <td>7604406.00</td>
    </tr>
  </tbody>
</table>
<p>91 rows × 2 columns</p>
</div>




```python
# 위 결과를 그래프로 그리기 (.plot(lw=2, colormap='Paired'))
df_bond.groupby(['D+', 'quarter'])['시장가'].sum().unstack().plot(lw=3, colormap='Paired')
```




    <Axes: xlabel='D+'>




    
![png](output_82_1.png)
    

