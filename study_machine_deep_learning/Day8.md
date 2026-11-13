
## 데이터과학


#### 데이터과학의 3대 축

- 수학,통계학 : 데이터에서 패턴을 분석하고 통계적으로 검증
- 컴퓨터공학 : 데이터 수집, 처리, 프로그래밍과 분석 구현
- 도메인 지식 : 분석 대상 분야의 현장 맥락과 문제 이해
  - Danger Zone : 도메인 지식 없이 통계,컴퓨팅만 사용하면 현장 문제를 잘못 해석할 수 있음


#### 비지니스 분석의 4단계

1. 기술적 분석(Descriptive) : 무슨일이 일어났는가?
2. 진단적 분석(Diagnostic) : 왜 일어났는가?
3. 예측적 분석(Predictive) : 앞으로 무슨 일이 일어날까?
4. 처방적 분석(Prescriptive) : 어떻게 대응해야 할까?


#### 보안 분야 머신러닝 적용 시 주의점

1. 문제 정의와 수치화 : 막연한 이상징후 대신 위험점수, 트래픽 지표, API호출 빈도 등 수치로 정의
2. 정확도(Accuracy)함정 : 정상 데이터가 압도적으로 많으면 모두 정상이라고 예측해도 정확도가 높을수 있음. 재현율(Recall), 정밀도(Precision), F1-score, AUC-ROC도 함께 확인
3. 데이터 편향(Data Bias) : 알려진 공격 패턴에만 치우쳐 학습하면 신종공격에 약해지거나 정상행위를 오탐할 수 있음
4. 개인정보,기밀유출 : 로그,패킷 분석 전 마스킹, 해싱, 비식별화 고려
5. 과도한 오경보(Alert Fatigue) : 오탐이 많으면 실제 위협을 놓치거나 정상 업무를 방해할 수 있음


#### 데이터 리터러시의 4대 기준

- 누가 만들었는가? : 출처와 신뢰도 확인
- 전수인가, 표본인가? : 표본 편향과 누락 확인
- 어떻게 측정했는가? : 수집 주기 및 결측치 처리 확인
- 포함, 제외 대상은? : 분석 목적에 맞게 계산 대상을 설정
  - 같은 보안 데이터라도 분모, 수집 방식, 분석 범위에 따라 결과 해석이 달라질수있다.

---

## 기초통계



#### 기초통계와 기술통계

- 통계 : 데이터의 특성을 수치로 요약하고 분석하는 기초 도구
- 기술통계(Descriptive Statistics) : 수집한 데이터의 전반적인 특징을 요약, 설명
- 중심경향 : 평균, 중앙값, 최빈값으로 데이터의 대표값을 파악
- 산포도 : 분산, 표준편차, 사분위수로 퍼짐 정도를 파악
- 상관관계 : 두 변수의 선형적 관계를 분석


## 평균, 중앙값, 최빈값

- 평균 : 모든 값의 합 + 데이터 개수   =>   극단값에 영향받기 쉬움
  - `.mean()`: 평균
- 중앙값 : 정렬했을 때 가운데 값   =>   극단값의 영향이 상대적으로 적음
  - `.median()`: 중앙값
- 최빈값 : 가장 자주 등장하는 값   =>   범주형,빈도 중심 데이터에 유용
  - `.mode()`: 최빈값
    - ```Python
      import pandas as pd

      data = [10, 15, 20, 10, 30, 10, 40]
      s = pd.Series(data)

      print(s.mean())       # 평균
      print(s.median())     # 중앙값
      print(s.mode()[0])    # 최빈값 중 첫 번째
      ```

      - `mode()`는 최빈값이 여러 개일 수도 있어서 Series형태로 반환함 [0] 은 첫 최빈값을 선택


## 상관관계

- **피어슨 상관계수`r`** 범위: `-1 ~ 1`
- `r > 0`: 양의 상관관계 (한 값 증가 → 다른 값도 증가하는 경향)
- `r < 0`: 음의 상관관계 (한 값 증가 → 다른 값은 감소하는 경향)
- `r ≈ 0`: 선형 상관관계가 거의 없음
  - 상관관계는 인과관계를 뜻하지 않음

---

## 분산

- 값들이 평균에서 얼마나 멀리 퍼져 있는지 보여주는 수치
- 데이터에서 평균을 뺀 편차를 제곱하고 평균을구함
- 값이 클수록 데이터의 퍼짐 정도가 큼
- `.var()`: 분산

## 표준편차

- 표준편차 = √분산
- 원래 데이터와 단위가 같아 분산보다 직관적으로 해석하기 좋음
- `.std()`: 표준편차

```Python
import pandas as pd

df = pd.DataFrame({'score': [55, 60, 65, 70, 75, 80, 85, 90, 95, 200]})

print(df['score'].var())  # 표본분산 (기본 ddof=1)
print(df['score'].std())  # 표본표준편차 (기본 ddof=1)
```


## 사분위수(Quartiles),IQR

- Q1 : 하위 25% 지점
- Q2 : 하위 50% 지점
- Q3 : 하위 75% 지점
- IQR(사분범위) : Q3 - Q1 : 중간 50%의 범위


## IQR 이상치 판단 경계

- 하한 = Q1 + 1.5 * IQR
- 상한= Q3 - 1.5 * IQR
  - ```Python
    # ex)
    q1 = df['score'].quantile(0.25)
    q2 = df['score'].quantile(0.5)
    q3 = df['score'].quantile(0.75)

    iqr = q3 - q1
    low = q1 - 1.5 * iqr
    up = q3 + 1.5 * iqr

    result = df[(df['score'] < low) | (df['score'] > up)]
    print(result)
    print(df.describe())
    ```

    - `describe()`는 수치형 열의 `count`, `mean`, `std`, `min`, `25%`, `50%`, `75%`, `max`를 한 번에 요약한다.


## Z-score(표준점수)

- Z = (관측값 - 평균) / 표준편차
  - Z = 0 : 평균과 같음
  - Z = +1 : 평균보다 표준편차 1개만큼 높음
  - Z = -1 : 평균보다 표준편차 1개만큼낮음

---

## 데이터 분포



#### 정규분포

- 평균을 중심으로 좌우 대칭인 종 모양의 분포
- 정규분포에서는 평균 = 중앙값 = 최빈값
- 평균 ±1 표준편차: 약 **68%**
- 평균 ±2 표준편차: 약 **95%**
- 평균 ±3 표준편차: 약 **99.7%**
  - ex) 평균 70점, 표준편차 10점이면 약 68%는 60~80점, 약 95%는 50~90점 범위


#### 왜도(Skewness)

- 데이터가 어느 쪽으로 치우쳤는지 나타내는 비대칭성 지표
- 양의 왜도 (`>0`) : 오른쪽 꼬리가 김 ex) 일부 매우 비싼 주택 가격
- 음의 왜도 (`<0`) : 왼쪽 꼬리가 김 ex) 일부 매우 낮은 시험 점수
- 왜도`≈0`: 대칭에 가까움
  - 양의 왜도 : 평균 > 중앙값 > 최빈값
  - 음의 왜도 : 평균 < 중앙값 < 최빈값


#### 첨도(Kurtosis)

- 분포의 뾰족함과 꼬리의 두꺼움을 나타내는 지표로 소개됨
- 큰 첨도 : 극단값이 발생할 가능성을 주의
- 작은 첨도 : 상대적으로 평평하거나 극단값이 적은 형태

---

## 그룹화

- 그룹별 집계 (`groupby`) : 공통 특성을 기준으로 데이터를 그룹화한 뒤 합계, 평균, 최대값 등을 계산
  - ```Python
    import pandas as pd

    data = {
        'Name': ['Alice', 'Bob', 'Charlie', 'David', 'Eva'],
        'Department': ['HR', 'IT', 'HR', 'IT', 'Finance'],
        'Salary': [50000, 60000, 55000, 70000, 65000]
    }
    df = pd.DataFrame(data)

    grouped = df.groupby('Department')['Salary'].mean()
    print(grouped)

    # 출력결과
    # Department
    # Finance    65000.0
    #HR         52500.0
    # IT         65000.0
    ```

    - `groupby('Department')`: `Department` 열을 기준으로 그룹화.
    - `['Salary']`: 급여 열 선택.
    - `.sum()`, `.max()`, `.min()`, `.count()`로 다른 집계도 가능.

---

## 데이터 시각화



#### 시각화 라이브러리

- `Matplotlib`: 다양한 그래프를 세밀하게 설정하는 기본 라이브러리
  - ```Python
    import matplotlib.pyplot as plt

    plt.rc('font', family='Malgun Gothic')  # 한글깨짐 방지
    plt.rc('axes', unicode_minus=False)     # `-` 깨짐 방지
    ```


- 선형그래프 (`plt.plot`) : 값의 변화 추이, 시간 흐름에 따른 변화 확인
  - ```Python
    x = [1, 2, 3, 4, 5]
    y = [4, 6, 9, 12, 20]  # 노트북 실습 데이터

    plt.plot(x, y, color='red', marker='o')   # 색깔 : 빨강, 마크 : 원  # marker : 각 점의 모양
    plt.title('선형차트')  # 그래프 제목
    plt.xlabel('번호')     # x축 이름
    plt.ylabel('점수')     # y축 이름
    plt.grid(True)         # 격자선
    plt.show()             # 그래프표시
    ```


- 막대그래프 (`plt.bar`, `plt.barh`) : 항목(범주)별 크기 비교
  - ```Python
    categories = ['A', 'B', 'C', 'D', 'E']
    values = [5, 10, 3, 8, 4]  # 노트북 실습 데이터

    plt.bar(categories, values, edgecolor='r')      # 세로 막대 그래프 # edgecolor : 막대테두리색
    plt.show()

    plt.barh(categories, values)     # 가로 막대 그래프
    plt.show()
    ```


- 히스토그램(`plt.hist`) : 연속형 데이터를 구간으로 나누어 빈도 분포 확인
  - ```Python
    import numpy as np
    import matplotlib.pyplot as plt

    data = np.random.randn(1000)    # 난수 1000개 생성

    plt.hist(data, bins=20, edgecolor='black')    # bins : 데이터를 20개 구간으로 나눔
    plt.title('Histogram')
    plt.xlabel('Value')
    plt.ylabel('Frequency')
    plt.show()
    ```
