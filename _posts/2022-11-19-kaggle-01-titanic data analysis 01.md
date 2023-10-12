---
title: Kaggle_01_Titanic Data Analysis 01
author: B
date: 2022-11-19 00:23:00 +0900
categories: [Kaggle, Titanic]
image:
    path: /commons/titanic01/01.png
---

![Desktop View](/commons/titanic01/01.png){: width="100%"}
- 사용 환경 : Python 3.11
- 사용 툴 : VS Code

## 1. 데이터 수집 및 분석 환경 설정
- `pip install -U pandas numpy seaborn matplotlib spicy scikit-learn`
```python
import os
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
```

- pandas : Python에서 데이터 조작 및 분석을 위해 사용되는 라이브러리 이다.
- numpy : Python을 이용한 과학 컴퓨팅을 위한 기본 패키지이다. 다차원 원소배열, 이산/통계 연산과 마스킹 배열을 포함한 파생 객체를 지원하여 배열의 빠른 연산을 지원하는 라이브러리이다[^fn-01].
- seaborn : Matplotlib을 기반으로 다양한 색상 테마와 통계용 차트 등의 기능을 추가한 시각화 패키지이다. 기본적인 시각화 기능은 Matplotlib 패키지에 의존하며 통계 기능은 Statsmodels 패키지에 의존한다.[^fn-02][^fn-03]
- Matplotlib : Python에서 자료를 chart나 plot으로 시각화하는 패키지이다. [^fn-04]

## 2. 데이터 클리닝
- 데이터 클리닝이란, 데이터 집합 내에서 오류가 있거나, 손상되었거나, 형식에 문제가 있거나, 중복되었거나, 어떤 형태로든 불완전한 데이터를 수정하거나 제거하는 프로세스이다. [^fn-05]

### 2-1. 데이터 확인
```python
data = pd.read_csv('train.csv')
print(data.info())
```
- 아래와 같이 데이터 인덱스와 인덱스 범위 및 타입과 대략적인 null 여부 등의 정보를 알 수 있다.

![Desktop View](/commons/titanic01/02.png)

### 2-2. question mark 처리
- 지금은 따로 정제가 필요 없지만, 이전 학습 데이터 파일 내에서는 데이터를 사용할 수 없는 셀을 식별하기 위해 물음표("?")가 사용되었다.
- Pandas는 이 값을 DataFrame으로 읽을 수 있지만 age와 같은 열의 경우 데이터 유형이 지금과 깉이 float64가 아닌 object로 설정되어 그래프 작성에 문제가 되었다.
- 이 경우에는 아래와 같이 물음표를 pandas가 이해할 수 있는 numpy NaN값으로 바꾸며 값을 바꾼 후 열의 데이터 유형을 변경해야한다.

```python
data = pd.read_csv('train.csv') # train 데이터가 저장된 csv 파일 읽기

# NULL 데이터 처리
data.replace('?', np.nan, inplace = True)
data = data.astype({"Age": np.float64, "Fare": np.float64})
```

### 2-3. 결측치 확인
```python
data_null_ratio = data.isnull().sum() / len(data) * 100
```
- 위 data_null_ratio를 출력 할 경우 아래와 같이 각 행에 존재하는 null의 비율들을 확인할 수 있다.

![Desktop View](/commons/titanic01/03.png)

- 현재 결측치가 확인되는 컬럼은 Age, Cabin, Embarked이다.
- Cabin 컬럼과 같은 경우에는 사고 발생 시 선박의 대략적인 위치와 SES(Socio-Econobic Status)에 대한 feature가 가능하지만 대부분의 값이 결측값임을 확인할 수 있으며, NaN값을 대체할 수 있는 데이터가 존재하지 않으므로 큰 가치가 없는 이상 제외한다. (SES와 같은 경우에는 Pclass 컬럼으로 대체가능하다.)

### 2-4. Age 결측치 처리
- Age 컬럼의 경우에는 '그룹 별 평균값으로 결측치 대체'를 진행하며 두 가지의 방법이 있다.
- 첫번째 방법으로는 아래는 이미 데이터들이 분류가 되엉ㅆ는 컬럼들과 그룹화하여 평균값을 도출하는 방법이다.

```python
data[data['Age'].notnull()].groupby(['Sex'])['Age'].mean()
data[data['Age'].notnull()].groupby(['Pclass'])['Age'].mean()
```

<p align="center" width="100%">
    <a href="/commons/titanic01/04.png" class=""><img src="/commons/titanic01/04.png" width="44%"></a>
    <a href="/commons/titanic01/05.png" class=""><img src="/commons/titanic01/05.png" width="34%"></a>
</p>

    + Age 데이터를 Sex컬럼과 Pclass 컬럼에 각각 그룹화 하여 평균값을 도출하였을 때, Sex 컬럼보다는 Pclass 컬럼으로 그룹화하였을 때 조금 더 명확한 나이선이 구분되는 것을 볼 수 있다.

- 두번째 방법은 준비된 데이터를 분리하여 더욱 유용한 데이터를 도출하는 방법이다.
    + 이 방법은 Name 컬럼을 사용하여 이름 내에서 각 호칭들을 분리한 뒤, 분리된 호칭을 사용하여 평균 나이를 도출한다.
    + 도출된 평균 나이를 이용하여 각 Age의 결측치를 대체한다.
```python
data['Title'] = 0
for i in data:
    data['Title'] = data.Name.str.extract('([A-Za-z]+)\.') # 정규표현식을 사용하여 호칭 도출
pd.crosstab(data.Title, data.Sex).T.style.background_gradient(cmap='coolwarm')
```
![Desktop View](/commons/titanic01/06.png){: width="100%"}

    + crosstab을 사용하면 위와 같이 각 Title에 해당하는 성별의 분포도를 확인할 수 있다.

```python
data['Title'] = data['Title'].replace(['Capt', 'Col', 'Countess', 'Jonkheer', 'Major', 'Rev', 'Sir'], 'Others')
data['Title'] = data['Title'].replace(['Mlle', 'Ms'], 'Miss')
data['Title'] = data['Title'].replace('Don', 'Mr')
data['Title'] = data['Title'].replace(['Mme', 'Lady', 'Dona'], 'Mrs')

data.groupby('Title')['Age'].mean().round()
```

![Desktop View](/commons/titanic01/07.png){: width="215" height="157"}

- 호칭은 Mrs, Mr, Miss, Master, Others 다섯가지로 정리하며 정리된 호칭 별로 반올림된 평균 나이를 확인하고 출력되는 평균값을 Age 컬럼의 결측치들과 대체한다.

```python
data.loc[(data.Age.isnull()) & (data.Title=='Master'), 'Age'] = 5
data.loc[(data.Age.isnull()) & (data.Title=='Miss'), 'Age'] = 22
data.loc[(data.Age.isnull()) & (data.Title=='Mr'), 'Age'] = 32
data.loc[(data.Age.isnull()) & (data.Title=='Mrs'), 'Age'] = 36
data.loc[(data.Age.isnull()) & (data.Title=='Others'), 'Age'] = 46
```

- 본문에서는 두번째 방법을 사용하여 Age 컬럼의 결측치를 대체하고 각 호칭별 생존자를 예측했을 경우 아래와 같은 결과를 도출한다.

```python
data[['Title', 'Survived']].groupby(['Title'], as_index=False).mean().sort_values(by='Survived', ascending=False)
```
![Desktop View](/commons/titanic01/08.png){: width="202" height="215"}

### 2-5. Embarked 결측치 처리
- Embarked의 결측치는 2개 이므로 최빈값으로 대체한다.

```python
f, ax = plt. subplots(1, 2, figsize = (20, 10))
sns.countplot(x = 'Embarked', data = data, ax = ax[0])
ax[0].set_title('Passengers Boarded')
sns.countplot(x = 'Embarked', hue = 'Survived', data = data, ax = ax[1])
ax[1].set_title('Embarked vs Survived')
plt.subplots_adjust(wspace = 0.2, hspace = 0.5)
plt.show()
```

![Desktop View](/commons/titanic01/09.png)

- 최빈값 확인을 위해 countplot을 사용해 출력 후 그래프 확인 상 빈도수가 가장 많은 'S' 값으로 대체한다. [^fn-06] [^fn-07]
```python
data['Embarked'].fillna('S', inplace=True)
```

## * 진행 과정 메모 사항
- 초기 진행의 경우 python 환경에서는 작동을 하였으나, py파일이 아닌 jupyter notebook 사용.
- py 파일에서 동작할 경우 tkinter 내 포함된 lib 에러 발생 확인
- 이에 py 환경 변경을 위한 python 3.11 패키지 재설치 (+tk 포함된 버전) 후 환경 변수에 설정되어 있는 기존 python 3.6에서 3.11 버전으로 재설정
- 그 후 figsize에서 에러 발생 확인 → 단순 오타로 인한 오류 발생으로 확인

## * 다음 포스트 내용
- 본격적인 학습을 진행하기 앞서 모델의 정확도를 높이기 위한 Feature Engineering과 데이터 시각화 진행

## 참고 URL
[^fn-01]: <https://numpy.org/>
[^fn-02]: <http://seaborn.pydata.org/>
[^fn-03]: <https://datascienceschool.net/01%20python/05.04%20%EC%8B%9C%EB%B3%B8%EC%9D%84%20%EC%82%AC%EC%9A%A9%ED%95%9C%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EB%B6%84%ED%8F%AC%20%EC%8B%9C%EA%B0%81%ED%99%94.html>
[^fn-04]: <https://datascienceschool.net/01%20python/05.01%20%EC%8B%9C%EA%B0%81%ED%99%94%20%ED%8C%A8%ED%82%A4%EC%A7%80%20%EB%A7%B7%ED%94%8C%EB%A1%AF%EB%A6%AC%EB%B8%8C%20%EC%86%8C%EA%B0%9C.html#:~:text=%EB%A7%B7%ED%94%8C%EB%A1%AF%EB%A6%AC%EB%B8%8C(Matplotlib)%EB%8A%94,%EC%8B%9C%EA%B0%81%ED%99%94%20%EA%B8%B0%EB%8A%A5%EC%9D%84%20%EC%A0%9C%EA%B3%B5%ED%95%9C%EB%8B%A4.>
[^fn-05]: <https://double-d.tistory.com/14#:~:text=%EB%8D%B0%EC%9D%B4%ED%84%B0%20%ED%81%B4%EB%A6%AC%EB%8B%9D%EC%9D%80%20%EC%9D%B4%EB%9F%B0%20%EB%8D%B0%EC%9D%B4%ED%84%B0,%ED%95%98%EA%B1%B0%EB%82%98%20%EC%A0%9C%EA%B1%B0%ED%95%98%EB%8A%94%20%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4%EC%9D%B4%EB%8B%A4.>
[^fn-06]: <https://www.kaggle.com/code/ldfreeman3/a-data-science-framework-to-achieve-99-accuracy/notebook>
[^fn-07]: <https://dsbook.tistory.com/62>