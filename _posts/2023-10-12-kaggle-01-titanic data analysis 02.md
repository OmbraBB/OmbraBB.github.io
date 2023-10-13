---
title: Kaggle_01_Titanic Data Analysis 02
author: B
date: 2023-10-12 15:33:00 +0900
categories: [Kaggle, Titanic]
image:
    path: /commons/titanic01/01.png
---

## 3. Feature Engineering
Feature Engineering은 모델 정확도를 높이기 위해서 주어진 데이터를 예측 모델의 문제를 잘 표현할 수 있는 features로 변형시키는 과정으로 `머신러닝 알고리즘을 작동하기 위해 데이터의 도메인 지식을 활용해 feature를 만드는 과정`이다.[^fn-01]
기본적으로 Imputation, Handling Outliers, Binning, Log Transform, One Hot Encoding, Grouping Operation, Scaling 등 7가지 기법을 자주 사용하며, 데이터 분석가의 객관적인 시각에 따라 활용하는 것이 중요하다.

Binning (Bin으로 묶기)
: 통계에서 Bin이라 하면 히스토그램의 하나의 막대 정도를 나타내는데, '통' 정도로 이해하면 될듯하다. 근처의 값들을 하나의 범주로 묶게 되면 좀 더 Robust한 모델을 얻을 수 있다. 물론 이에 따라 주요 정보를 잃을 수 있어서 얼만큼을 하나의 Bin으로 묶을지 주의해야한다.[^fn-02] (* 즉 한 컬럼의 데이터를 범주로 묶는 것을 의미)

### 3-1. Age Binning
- 연속형 변수인 Age를 번주형 변수로 변환시킨다. 15세 이하, 30세 이하, 50세 이하, 65세 이하, 그 이상으로 5가지 범주로 구분한다.

```python
data['age_bin'] = 0
data.loc[data['Age'] <= 15, 'age_bin'] = 0
data.loc[(data['Age'] > 15) & (data['Age'] <= 30), 'age_bin'] = 1
data.loc[(data['Age'] > 30) & (data['Age'] <= 50), 'age_bin'] = 2
data.loc[(data['Age'] > 50) & (data['Age'] <= 65), 'age_bin'] = 3
data.loc[data['Age'] > 65, 'age_bin'] = 4
```

```python
sns.catplot(x = 'age_bin', y = 'Survived', data = data, col = 'Pclass')
plt.show()
```

- 기존에는 sns.factorplot()을 사용하였지만 최신 버전에서는 지원하지 않으며, 대신 catplot에서 kind 옵션을 point로 지정해 준다. [^fn-03]

![Desktop View](/commons/titanic02/01.png)

- 나이를 범주형 변수로 변환시키고 난 후 생존율을 확인한 결과, 젊을수록 생존율이 높게 나타남을 확인할 수 있다.

### 3-2. Fare Binning
- 탑승 요금도 Age와 동일한 방식으로 4개의 구간으로 범주화한다.

```python
data['fare_range'] = pd.qcut(data['Fare'], 4)
data.groupby(['fare_range'])['Survived'].mean().to_frame().style.background_gradient(cmap='coolwarm')
```

<!-- ![Desktop View](/commons/titanic02/02.png) -->
![Desktop View](/commons/titanic02/02.JPG)

- 위와 같이 나누어진 fare_range로 fare_bin 컬럼을 생성한다.

```python
data['fare_bin'] = 0
data.loc[data['Fare'] <= 7.91, 'fare_bin'] = 0
data.loc[(data['Fare'] > 7.91) & (data['Fare'] <= 14.454), 'fare_bin'] = 1
data.loc[(data['Fare'] > 14.454) & (data['Fare'] <= 31), 'fare_bin'] = 2
data.loc[(data['Fare'] > 31) & (data['Fare'] <= 512.329), 'fare_bin'] = 3
```

### 3-3. 동승객 Engineering

- Parch와 Sibsp를 더한 값으로 같이 탑승한 전체 가족 수에 대한 column을 생성한다. 그 후 혼자 탑승한 승객은 Alone으로 표시해서 동승객의 여부에 관해 생존율을 비교한다.

```python
data['family'] = 0
data['family'] = data['Parch'] + data['SibSp']
data['Alone'] = 0
data.loc[data.family == 0, 'Alone'] = 1

f, ax = plt.subplots(1, 2, figsize = (15, 5))
sns.catplot(x = 'family', y = 'Survived', data = data, ax = ax[0], kind = 'point')
ax[0].set_title('family vs Survived')
sns.catplot(x = 'Alone', y = 'Survived', data = data, ax = ax[1], kind = 'point')
ax[1].set_title('Alone vs Survived')
plt.close(1)
plt.show()
```

<p align="center" width="100%">
    <a href="/commons/titanic02/03.png" class=""><img src="/commons/titanic02/03.png" width="344" height="270"></a>
    <a href="/commons/titanic02/04.png" class=""><img src="/commons/titanic02/04.png" width="344" height="270"></a>
</p>

- 혼자 탑승한 승객은 Alone에 1로 표시했다. 그래프를 확인한 결과, 함께 탑승한 가족이 3명일 경우 가장 높은 생존율이 나타났으며, 혼자 탑승한 승객의 생존율은 생각보다 낮은것을 확인할 수 있다.

### 3-4. mapping

- 범주형 변수인 성별, 탑승 위치, 이니셜을 mapping 한다.

```python
data['Sex'].replace(['male', 'female'], [0, 1], inplace = True)
data['Embarked'].replace(['S', 'C', 'Q'], [0, 1, 2], inplace = True)
data['Title'].replace(['Mr', 'Mrs', 'Miss', 'Master', 'Others'], [0, 1, 2, 3, 4], inplace = True)
```

## 4. 무의미한 데이터 삭제

- 머신러닝 과정에서 사용하지 않을 데이터인 Name, Age, Fare, Cabin, Fare_range, PassengerId column을 삭제하고 전처리를 한 변수들의 상관관계를 확인한다.

```python
data.drop(['Name', 'Age', 'Ticket', 'Fare', 'Cabin', 'fare_range', 'PassengerId'], axis = 1, inplace = True)
sns.heatmap(data.corr(), annot = True, cmap = 'YlGnBu', linewidths = 0.1, annot_kws = {'size': 20})
fig = plt.gcf()
fig.set_size_inches(15, 12)
plt.xticks(fontsize = 8)
plt.yticks(fontsize = 8)
plt.show()
```

![Desktop View](/commons/titanic02/05.png)

- 이제 어느 정도 데이터 정리가 완료되었다. 이후 머신러닝 모델을 사용하여 test 데이터에 담긴 탑승객의 생존율을 예측한다.

## 5. 모델링 기법

- 생존율 예측을 위해 6가지 모델을 학습시켜서 정확도가 가장 높은 것을 테스트에서 사용한다. 여기서 사용할 모델은 다음과 같다.

1. Logistic Regression : 독립 변수의 선형 결합을 이용해서 사건의 발생 가능성의 예측을 위한 통계 기법 [^fn-04]

2. Decision Tree : 어떤 항목에 대한 관측값과 목표값을 연결시켜주는 예측 모델로서 결정트리를 사용한다. Flow chart로 자주 사용되는 모델 [^fn-05]

3. Support Vector Machine(SVM) : 주어진 데이터를 바탕으로 새로운 데이터가 어느 카테고리에 포함될 지 판단하는 모델이다. 패턴 인식, 자료 분석을 위한 지도 학습 모델이며, 주로 분류와 회귀 분석을 위해 사용한다. [^fn-06]

4. Random Forest : 결정 트리의 분산이 크다는 점을 고려해서, 다수의 결정 트리들을 학습하는 앙상블 방법이다. 랜덤 포레스트는 검출, 분류, 그리고 회귀 등 다양한 문제에 활용되고 있다. [^fn-07]

5. k-Nearest Neighbors(k-NN) : 패턴인식에서 분류나 회귀에 사용되는 비모수 방식이며, 두 경우 모두 입력이 특징 공간 내 k개의 가장 가까운 훈련 데이터로 구성되어 가장 연관있는 데이터의 class로 결과가 도출되는 모델 [^fn-08]

6. Naive Bayes : 특성들 사이의 독립을 가정하는 베이즈 정리를 적용한 확률 분류기의 일종으로 주로 텍스트 분류에 활용되는 모델 [^fn-09]

## 6. 데이터 모델링

- 본격적인 train 데이터 학습 전 터미널에서 pip 내 sklearn을 install 한다.

```
# pip3 install -U scikit-learn scipy matplotlib
```

- 아래와 같이 모델링에 필요한 머신러닝 모델들과 추가로 필요한 라이브러리들을 import 한다.

```python
# import numpy as np # 전에 이미 import 했으므로 주석처리

# 머신러닝 패키지 import
from sklearn.linear_model import LogisticRegression # 로지스틱 회귀
from sklearn.tree import DecisionTreeClassifier # 의사결정 나무
from sklearn.svm import SVC # SVM
from sklearn.ensemble import RandomForestClassifier # 랜덤포레스트
from sklearn.neighbors import KNeighborsClassifier # k-NN
from sklearn.naive_bayes import GaussianNB # 나이브 베이즈

# 교차검증
from sklearn.model_selection import KFold
k_fold = KFold(n_splits=10, shuffle=False)

# 정확도 측정
from sklearn.model_selection import cross_val_score
```

- 여기서 교차검증이란, 과적합을 방지하기 위해 데이터 셋을 k개로 나누어 test 셋을 중복되지 않도록 바꾸면서 모델을 학습시키는 것이다. 여기서는 데이터 셋을 10개로 나누어 진행한다. K-fold 외에도 교차검증에는 Holdout방법, Leave-ont-out 교차 검증 등이 있다.

```python
learning = data['Survived']
data = data.drop('Survived', axis=1)
scoring = 'accuracy'
```

- 생존여부를 학습하기 위해 learning으로 생존 결과만 따로 분리하고, train 데이터의 생존 여부는 삭제한다. 정확도 평가는 accuracy(TN+TP/전체)를 사용한다.

```python
model = LogisticRegression()
score = cross_val_score(model, data, learning, cv=k_fold, n_jobs=1, scoring=scoring)
round(np.mean(score)*100, 2)
```
: 로지스틱 회귀 : 80.25

```python
model = DecisionTreeClassifier()
score = cross_val_score(model, data, learning, cv=k_fold, n_jobs=1, scoring=scoring)
round(np.mean(score)*100, 2)
```
: 의사결정나무 : 79.58

```python
model = SVC()
score = cross_val_score(model, data, learning, cv=k_fold, n_jobs=1, scoring=scoring)
round(np.mean(score)*100, 2)
```
: SVM : 82.38

```python
model = RandomForestClassifier(n_estimators = 13)
score = cross_val_score(model, data, learning, cv=k_fold, n_jobs=1, scoring=scoring)
round(np.mean(score)*100, 2)
```
: 13개의 의사결정나무를 묶은 랜덤 포레스트 : 81.38

```python
model = KNeighborsClassifier(n_neighbors=13)
score = cross_val_score(model, data, learning, cv=k_fold, n_jobs=1, scoring=scoring)
round(np.mean(score)*100, 2)
```
: 주변 13개의 데이터 중 가장 근접한 경우를 확인하는 K-NN : 82.61

```python
model = GaussianNB()
score = cross_val_score(model, data, learning, cv=k_fold, n_jobs=1, scoring=scoring)
round(np.mean(score)*100, 2)
```
: 나이브 베이즈 : 79.8

- 참고한 사이트는 SVM이 가장 높게 나왔으나, 나의 결과 기준으로는 현재 K-NN의 정확도가 높게 나왔으므로, 이를 테스트 모델로 사용한다.

## 7. 데이터 테스트

```python
test_model = KNeighborsClassifier(n_neighbors=13)
test_model.fit(data, learning)

testing = test.drop('PassengerId', axis = 1).copy()
prediction = test_model.predict(testing)
```

test 데이터도 train 데이터와 마찬가지로 데이터 전처리 완료 후, 테스트를 진행한다.
> 위에서 임의로 추가하였던 컬럼 'Title', 'age_bin', 'family', 'fare_bin'가 test 데이터에도 동일하게 존재해야한다. {: .prompt-tip}

```python
result = pd.DataFrame({
    "PassengerId" : test["PassengerId"],
    "Survived": prediction
})

result.head(10)
```

- 예측 결과를 result에 담아 결과를 확인한다.

![Desktop View](/commons/titanic02/06.png)





## 참고 URL

Base URL : <https://dsbook.tistory.com/>

[^fn-01]: <https://velog.io/@baeyuna97/Feature-engineering%EC%9D%B4%EB%9E%80>
[^fn-02]: <https://magoker.tistory.com/118>
[^fn-03]: <https://www.tutorialspoint.com/plotting-different-types-of-plots-using-factor-plot-in-seaborn>
[^fn-04]: <https://ko.wikipedia.org/wiki/%EB%A1%9C%EC%A7%80%EC%8A%A4%ED%8B%B1_%ED%9A%8C%EA%B7%80>
[^fn-05]: <https://ko.wikipedia.org/wiki/%EA%B2%B0%EC%A0%95_%ED%8A%B8%EB%A6%AC_%ED%95%99%EC%8A%B5%EB%B2%95>
[^fn-06]: <https://ko.wikipedia.org/wiki/%EC%84%9C%ED%8F%AC%ED%8A%B8_%EB%B2%A1%ED%84%B0_%EB%A8%B8%EC%8B%A0>
[^fn-07]: <https://ko.wikipedia.org/wiki/%EB%9E%9C%EB%8D%A4_%ED%8F%AC%EB%A0%88%EC%8A%A4%ED%8A%B8>
[^fn-08]: <https://ko.wikipedia.org/wiki/K-%EC%B5%9C%EA%B7%BC%EC%A0%91_%EC%9D%B4%EC%9B%83_%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98>
[^fn-09]: <https://ko.wikipedia.org/wiki/%EB%82%98%EC%9D%B4%EB%B8%8C_%EB%B2%A0%EC%9D%B4%EC%A6%88_%EB%B6%84%EB%A5%98>