---
title: Python_list/set 합집합, 차집합, 교집합
author: B
date: 2022-12-14 17:13:00 +0900
categories: [Python]
---

- set을 사용하지 않은 list의 합집합, 차집합, 교집합 코드

```python
list_1 = [1, 2, 3, 4]
list_2 = [1, 2, 5, 6]

# 합집합
list_3 = []
list_3.extend(list_1)
list_3.extend(list_2)
list_3 = list(set(list_3))

# 차집합
list_4 = []
for i in list_1:
    if j not in list_2:
        list_4.append(j)

# 교집합
list_5 = []
for i in list_1:
    if i in list_2:
        list_5.append(i)

# 결과
합집합 = [1, 2, 3, 4, 5, 6]
차집합 = [3, 4]
교집합 = [1, 2]
```


- set을 사용한 list의 합집합, 차집합, 교집합 코드

```python
# 합집합
list_3 = list(set(list_1) | set(list_2))

# 차집합
list_4 = list(set(list_1) - set(list_2))

# 교집합
list_5 = list(set(list_1) & set(list_2))

# 결과는 위와 동일하다.
# 합집합과 교집합의 경우 set을 사용함으로 인해 자동으로 중복제거가 진행된다.
```