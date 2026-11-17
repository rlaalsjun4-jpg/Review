## 데이터 전처리

데이터 전처리 : 데이터를 분석하거나 머신러닝 모델에 사용하기 전에 결측치, 중복, 이상치 등을 처리해 분석에 적합한 형태로 만드는 과정이다.

기본 흐름 : 데이터 수집  ->  데이터 정제  ->  데이터 변환  ->  데이터 통합

#### 데이터의 유형

| 유형          | 특징               | 예시                 |
| ------------- | ------------------ | -------------------- |
| 정형 데이터   | 행과 열로 구성     | CSV, Excel, DB       |
| 비정형 데이터 | 고정된 구조가 없음 | 이미지, 영상, 텍스트 |
| 반정형 데이터 | 유연한 구조를 가짐 | JSON, XML, 웹 로그   |


#### 데이터 정제

결측치 : 결측치는 값이 누락된 상태 ( NaN, None 등)

| 함수                  | 설명                  |
| --------------------- | --------------------- |
| `df.isnull()`       | 결측치 여부 확인      |
| `df.isnull().sum()` | 컬럼별 결측치 개수    |
| `df.dropna()`       | 결측치가 있는 행 제거 |
| `df.dropna(axis=1)` | 결측치가 있는 열 제거 |
| `df.fillna()`       | 결측치 대체           |
| `df.mean()`         | 평균                  |
| `df.median()`       | 중앙값                |
| `df.mode()[0]`      | 첫 번째 최빈값        |

결측치처리

```Python
import pandas as pd

data = {
    'Name': ['Alice', 'Bob', None],
    'Age': [25, None, 30],
    'Score': [85, 90, None]
}
df = pd.DataFrame(data)

print(df.isnull().sum())  # 결측치 개수 확인
print(df.dropna())        # 결측치가 있는 행 제거

# 결측치 대체
df['Age'] = df['Age'].fillna(df['Age'].mean())
df['Score'] = df['Score'].fillna(df['Score'].median())
df['Name'] = df['Name'].fillna(df['Name'].mode()[0])
print(df)
```

- **평균:** 일반적인 수치형 데이터
- **중앙값:** 극단값에 민감하지 않은 대체가 필요할 때
- **최빈값:** 범주형 데이터에 자주 사용
- `mode()`는 Series를 반환하므로 `[0]`으로 첫 번째 최빈값을 선택한다.


#### 중복 데이터 처리

| 함수                   | 설명                              |
| ---------------------- | --------------------------------- |
| `duplicated()`       | 중복 여부를 `True/False`로 반환 |
| `duplicated().sum()` | 중복 행 수                        |
| `drop_duplicates()`  | 중복 행 제거                      |
| `keep='first'`       | 첫 번째 행 유지(기본값)           |
| `keep='last'`        | 마지막 행 유지                    |
| `subset=['Name']`    | 특정 컬럼 기준으로 중복 판단      |

```Python
data = {
    'Name': ['Alice', 'Bob', 'Alice', 'Charlie', 'Bob'],
    'Age': [25, 30, 25, 35, 30],
    'Score': [85, 90, 85, 95, 90]
}
df = pd.DataFrame(data)

print(df.duplicated())
print(df.duplicated(subset=['Name']).sum())
print(df[df.duplicated()])

df_clear = df.drop_duplicates()
df_cleared1 = df.drop_duplicates(keep='last')
df_cleared2 = df.drop_duplicates(subset=['Name'])
```

- `duplicated()`는 확인, `drop_duplicates()`는 제거이다.


#### 이상치 처리

이상치 : 다른값과 크게 동떨어진 데이터 -> 반드시 오류는 아니므로 실제 의미를 확인해야함


##### IQR

- `Q1`: 제1사분위수(25%)
- `Q2`: 제3사분위수(75%)
- `IQR`: `Q3 - Q1`
- 하한 : `Q1 - 1.5 * IQR`
- 상한 : `Q3 + 1.5 * IQR`

```Python
df = pd.DataFrame({'Scores': [70, 75, 80, 85, 90, 150]})

Q1 = df['Scores'].quantile(0.25)
Q3 = df['Scores'].quantile(0.75)
IQR = Q3 - Q1

lower = Q1 - 1.5 * IQR
upper = Q3 + 1.5 * IQR

outliers = df[(df['Scores'] < lower) | (df['Scores'] > upper)]
print(outliers)
```



## 데이터 스케일링

#### 표준화 - `StandardScaler`

데이터를 평균 0, 표준편차 1 이 되도록 변환한다

```Python
from sklearn.preprocessing import StandardScaler

data = pd.DataFrame({'Original': [50, 80, 90, 100, 60]})
scaler1 = StandardScaler()
data['scaler1'] = scaler1.fit_transform(data[['Original']]).ravel()
print(data)
```



#### 정규화 - `MinMaxScaler`

데이터를 보통 0~1 범위로 변환한다.

```Python
from sklearn.preprocessing import MinMaxScaler

scaler2 = MinMaxScaler()
data['scaler2'] = scaler2.fit_transform(data[['Original']]).ravel()
print(data)
```


| 구분   | 표준화             | 정규화(Min-Max)             |
| ------ | ------------------ | --------------------------- |
| 클래스 | `StandardScaler` | `MinMaxScaler`            |
| 기준   | 평균 0, 표준편차 1 | 최소 0, 최대 1              |
| 음수   | 가능               | 학습 데이터 범위에서는 없음 |
| 이상치 | 영향받음           | 특히 민감                   |



#### 여러 컬럼 한 번에 스케일링

```Python
columns = ['Age', 'Score']
scaler = StandardScaler()

df[columns] = scaler.fit_transform(df[columns])

#for문 사용 (결과 동일)
for col in columns:
    scaler = StandardScaler()
    df[[col]] = scaler.fit_transform(df[[col]])
```

**대괄호 차이**

- `df['Age']`: 1차원 Series
- `df[['Age']]`: 2차원 DataFrame
- `df[columns]`: 여러 열의 2차원 DataFrame
- `df[[columns]]`: 리스트가 중첩되어 일반적인 다중 컬럼 선택에 적합하지 않음

`fit_transform()`은 2차원 입력을 기대하므로 `df[['Age']]` 또는 `df[columns]`를 사용한다.


## 범주형 데이터 처리

#### 레이블 인코딩 - `LabelEncoder`

각 범주를 정수로 바꿈  예:`배→0`, `사과→1`, `포도→2` (매핑은 데이터에 따라 달라질 수 있음)

```Python
from sklearn.preprocessing import LabelEncoder

data = pd.DataFrame({'Fruits': ['사과', '배', '포도', '사과', '포도']})
encoder1 = LabelEncoder()
data['Label Encoding'] = encoder1.fit_transform(data['Fruits'])
print(data)
```

순서가 없는 범주에 정수를 부여하면 일부 모델은 숫자의 크기를 순서처럼 해석할 수 있다.


#### 원 핫 인코딩 - `OneHotEncoder`

범주마다 컬럼을 만들고 해당 범주이면 1, 아니면 0으로 표현

```Python
from sklearn.preprocessing import OneHotEncoder

encoder2 = OneHotEncoder(sparse_output=False)
one_hot_encoded = encoder2.fit_transform(data[['Fruits']])

df_temp = pd.DataFrame(
    one_hot_encoded,
    columns=encoder2.get_feature_names_out(['Fruits'])
)
print(df_temp)
```



| 구분             | 레이블 인코딩    | 원-핫 인코딩       |
| ---------------- | ---------------- | ------------------ |
| 결과             | 정수 0, 1, 2 ... | 여러 개의 0/1 컬럼 |
| 컬럼 수          | 그대로           | 증가               |
| 범주의 순서 암시 | 있을 수 있음     | 없음               |


## 피처 엔지니어링

기존 컬럼을 활용해 모델 학습에 도움이 되는 새로운 특성을 만듬

#### 날짜와 시간 분해

```Python
df['JoinDate'] = pd.to_datetime(df['JoinDate'])

df['JoinYear'] = df['JoinDate'].dt.year
df['JoinMonth'] = df['JoinDate'].dt.month
df['JoinDay'] = df['JoinDate'].dt.day
df['JoinHour'] = df['JoinDate'].dt.hour
df['JoinDayofweek'] = df['JoinDate'].dt.day_name()
```

| 코드                 | 기능                |
| -------------------- | ------------------- |
| `pd.to_datetime()` | 날짜 타입으로 변환  |
| `.dt.year`         | 연도                |
| `.dt.month`        | 월                  |
| `.dt.day`          | 일                  |
| `.dt.hour`         | 시간                |
| `.dt.day_name()`   | 요일 이름           |
| `.dt.weekday`      | 월요일 0 ~ 일요일 6 |



#### 로그 변환

```Python
import numpy as np

df['Incom_Log'] = np.log1p(df['AnnualIncome'])
```

- `np.log()`는 자연로그 계산
- `np.log1p(x)`는 `log(1 + x)` 계산
- 큰 값에 치우친 분포를 완화하는 데 사용한다.


#### 구간화

```Python
bins = [0, 19, 39, 59, 100]
labels = ['Youth', 'YoungAdult', 'MiddleAge', 'Senior']

df['AgeGroup'] = pd.cut(df['Age'], bins=bins, labels=labels)
```



### 교차변수

```Python
df['TotalSpend'] = df['PurchaseCount'] * df['AvgOrderValue']
```

- 구매 횟수와 평균 주문 금액을 결합해 총 구매 금액을 생성한다.
