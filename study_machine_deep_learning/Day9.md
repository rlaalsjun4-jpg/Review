## 데이터 불러오기

```Python
import pandas as pd

df = pd.read_csv('asdf.csv', encoding = 'cp949', low_memory = False)
```

- `encoding='cp949'` : 한글 파일 인코딩 지정
- `low_memory=False` : 파일을 읽을 때 데이터 타입 추론을 보다 일관되게 처리

## 데이터 전처리

#### 필요한 컬럼만 선택

```Python
df_1 = df[['컬럼1','컬럼2','컬럼3']]
```

#### 결측값이 있는 행 제거

```Python
df_2 = df_1.dropna(subset=['컬럼1'])
```

- `dropna(subset=[...])`: 지정한 컬럼에 결측값이 있는 행 제거
- `dropna(columns=(['컬럼1'])`: 컬럼1이라는 컬럼 자체를 제거

#### 문자열 조건으로 행 추출

```Python
# 서울특별시 주소만 선택
df_3 = df_2[df_2['도로명주소'].str.contains('서울특별시')]  

# 서울특별시로 시작하는 주소만 선택
df_3 = df_2[df_2['도로명주소'].str.startswith('서울특별시')]

# 서울특별시로 끝나는 주소만 선택
df_3 = df_2[df_2['도로명주소'].str.endswith('서울특별시')]

# 매장명에 '스타벅스'가 포함된 행 선택
df_4 = df_3[df_3['사업장명'].str.contains('스타벅스')]

# 영업 상태가 '영업/정상'인 행 선택
df_5 = df_4[df_4['영업상태명'] == '영업/정상']
```

- `str.contains('문자열')`: 해당 문자열 포함 여부 검사

#### 문자열 분리 및 새로운 컬럼 생성

```Python
'서울특별시 강동구 고덕로 429'.split()
# ['서울특별시', '강동구', '고덕로', '429']

df_5.loc[:, '시군구'] = df_5['도로명주소'].str.split().str[1]
```

- `str.split()` : 공백 기준으로 문자열 나누기
- `.loc[:, '시군구']` : 전체 행의 시군구 컬럼을 지정

## 그룹별 데이터 집계

```Python
df_6 = df_5.groupby('시군구').size().reset_index(name='사업장수')
```

- `groupby('시군구')` : 같은 시군구끼리 그룹화
- `size()` : 그룹별 행 개수 세기
- `reset_index(name='사업장수')` : 결과를 일반 데이터프레임으로 바꾸고 집계 컬럼명을 지정


## 데이터 품질 확인

```Python
df.shape                     # 데이터 크기
df.info()                    # 자료형 및 결측 현황
df.describe()                # 기초 통계량
df.isnull().sum()            # 컬럼별 결측값 개수
df.dtypes                    # 컬럼별 자료형
df.duplicated(subset='Name').sum()  # 이름 기준 중복 개수
```

- `isnull()` : 결측값이면 True
- `sum()` : True 값을 합산하여 개수 확인
- `duplicated(subset='Name')` :  `Name` 컬럼 기준 중복 여부 확인(첫 등장 행은 중복 처리하지 않음)


## Seaborn 

#### 히스토그램

```Python
import seaborn as sns

scores = [56, 67, 45, 89, 90, 77, 68, 88, 92, 76, 58, 70, 73, 85, 62]
sns.histplot(scores, bins=5, kde=True)
```

- `sns.histplot()` : 숫자 데이터의 분포를 구간별 빈도로 표현
- `bins=5` : 구간 수를 5개로 지정
- `kde=True` : 부드러운 밀도 추정 곡선 표시


#### 산점도

```Python
data = pd.DataFrame({
    'Ad_Spend': [100, 200, 300, 400, 500],
    'Sales': [10, 15, 20, 25, 30]
})

plt.scatter(data['Ad_Spend'], data['Sales'])
```

- `plt.scatter(x, y)`:  두 변수의 관계를 점으로 나타냄


#### 상관관계 분석 및 히트맵

```Python
data = pd.DataFrame({
    'Math': [90, 80, 70, 60, 85],
    'English': [85, 75, 65, 55, 10],
    'Science': [88, 78, 68, 58, 23]
})

corr = data.corr()
print(corr)

sns.heatmap(corr, annot=True, cmap='coolwarm')
```

- `corr()` : 수치형 컬럼 사이의 피어슨 상관계수 계산
- 상관계수는 `-1`부터 `1`까지의 값
  - `1`에 가까움 : 강한 양의 선형 관계
  - `-1`에 가까움 : 강한 음의 선형 관계
  - `0`에 가까움 : 선형 관계가 약함
- 자기 자신과의 상관계수는 `1`
- `sns.heatmap()` : 행렬의 값들을 색상으로 표시
- `annot=True` : 각 칸에 숫자 표시
- `cmap='coolwarm'` : 색상 테마 지정


#### 범주별 데이터 개수(Countplot)

```Python
tips = sns.load_dataset('tips')
sns.countplot(x='day', data=tips)
```

- `sns.countplot(x='day', data=tips)` : 요일(`day`)별 행 개수 시각화
