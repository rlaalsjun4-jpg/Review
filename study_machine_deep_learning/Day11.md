## 머신러닝

- 사람이 모든 규칙을 직접 작성하는 대신, 데이터에서 규칙과 패턴을 학습하여 새로운 결과를 예측하거나 판단하는 기술

#### 지도학습, 비지도학습

| 구분        | 지도학습(Supervised Learning) | 비지도학습(Unsupervised Learning) |
| ----------- | ----------------------------- | --------------------------------- |
| 정답(label) | 있음                          | 없음                              |
| 데이터      | 입력 X + 정답 y               | 입력 X                            |
| 목표        | 입력과 정답의 관계 학습       | 데이터 속 구조·패턴·군집 탐색   |
| 예시        | 연봉 예측, 스팸 분류          | 고객 군집화                       |

- **Feature(X)**: 예측에 사용하는 독립변수(입력 특성).
- **Target(y)**: 예측하려는 종속변수(정답).

#### 회귀, 분류

| 구분      | 회귀(Regression)      | 분류(Classification)    |
| --------- | --------------------- | ----------------------- |
| 출력      | 연속적인 숫자         | 정해진 범주/클래스      |
| 예시      | 집값, 연봉, 시험 점수 | 스팸 여부, 이미지 종류  |
| 대표 기법 | 선형회귀              | 로지스틱 회귀, SVM, KNN |

---

## 데이터 준비, 탐색적 데이터 분석(EDA)

- EDA : 모델 학습 전 데이터의 분포, 결측치, 이상치, 특성 간 관계를 확인하는 과정

```Python
import pandas as pd
import matplotlib.pyplot as plt

# 예: csv를 읽었을 경우
df = pd.read_csv('data.csv')

df.head()             # 상위 5행
df.info()             # 자료형, 결측 여부, 행/열 수
df.describe()         # 수치형 데이터의 기초통계량
df.isnull().sum()     # 컬럼별 결측치 수
df.corr(numeric_only=True)  # 수치형 컬럼 간 상관계수
```

---

## 훈련 / 테스트 데이터 분리

- Train : 모델의 규칙을 학습시키는 데이터
- Test : 학습에 사용하지 않고 모델의 예측 성능을 확인하는 데이터
- 분리하는 이유 : 이미 학습한 데이터에서만 성능이 좋은지, 처음 본 데이터에서도 잘 맞는지를 판단하기 위해서

```Python
from sklearn.model_selection import train_test_split

X = df[['Age', 'PurchaseAmount']]  # 입력 X는 2차원
y = df['Satisfaction']            # 타깃 y는 보통 1차원

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)
```

- `X_train`: 학습용 입력 데이터
- `X_test`: 테스트용 입력 데이터
- `y_train`: 학습용 정답
- `y_test`: 테스트용 정답

---

## 회귀와 선형회귀의 원리

#### 회귀 종류

- 단순선형회귀 : X가 1개. ex) 공부 시간 -> 시험점수
- 다중선형회귀 : X가 여러 개. ex) 집크기, 위치, 층수 -> 집값
- 비선형회귀 : X와 y의 관계가 직선이 아닌 경우


#### 단순선형회귀

**`ŷ`**`=mx+b`

- `x`: 입력값, `ŷ`: 예측값.
- `m`: 기울기(특성이 1만큼 증가할 때 예측값 변화량).
- `b`: 절편(X가 0일 때의 예측값).
- 목표는 실제값과 예측값의 **오차가 작아지는 직선**을 찾는 것.


#### 모델 학습 과정

1. 입력 X와 정답 y준비
2. 모델 정의 및 학습
3. 학습된 모델로 예측
4. 실제값과 예측값을 비교하여 평가

```Python
from sklearn.linear_model import LinearRegression

model = LinearRegression()      # 모델 생성
model.fit(X_train, y_train)     # 학습
pred = model.predict(X_test)    # 예측

print(model.coef_)              # 특성별 회귀계수
print(model.intercept_)         # 절편
```



#### 경사하강법, 최소제곱법

- **손실함수**: 모델 예측이 얼마나 틀렸는지 수치화한 함수.
- **OLS**: 오차 제곱합이 최소가 되는 계수/절편을 찾는 방식.
- **경사하강법**: 손실이 줄어드는 방향으로 기울기·절편을 반복적으로 조정하는 최적화 방법.
- 강의자료의 수동 경사하강법 예제에서는 `learning_rate`, `epochs`, `m_gradient`, `b_gradient`를 사용함.
- 참고 : `sklearn.linear_model.LinearRegression`은 일반적으로 OLS 기반 해를 구하므로, 수동 경사하강법 코드를 직접 실행할 필요는 없음.


#### 손실함수와 회귀 성능 평가

실제값: `y_test` / 예측값: `y_pred`

| 지표           | 의미                    | 해석                             |
| -------------- | ----------------------- | -------------------------------- |
| **MAE**  | 오차 절댓값의 평균      | 작을수록 좋음, y와 같은 단위     |
| **MSE**  | 오차 제곱의 평균        | 작을수록 좋음, 큰 오차에 더 민감 |
| **RMSE** | MSE의 제곱근            | 작을수록 좋음, y와 같은 단위     |
| **R²**  | 모델의 설명력(결정계수) | 1에 가까울수록 좋음              |

```Python
import numpy as np
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score

y_pred = model.predict(X_test)

mae = mean_absolute_error(y_test, y_pred)
mse = mean_squared_error(y_test, y_pred)
rmse = np.sqrt(mse)
r2 = r2_score(y_test, y_pred)

print(f'MAE: {mae:.2f}')
print(f'MSE: {mse:.2f}')
print(f'RMSE: {rmse:.2f}')
print(f'R2: {r2:.4f}')
```

---

## 과소적합, 과적합

| 구분        | 과소적합(Underfitting)      | 과적합(Overfitting)                  |
| ----------- | --------------------------- | ------------------------------------ |
| 원인        | 모델이 지나치게 단순        | 데이터의 노이즈까지 외울 정도로 복잡 |
| 훈련 성능   | 나쁨                        | 매우 좋음                            |
| 테스트 성능 | 나쁨                        | 나쁠 수 있음                         |
| 핵심 문제   | 패턴을 충분히 학습하지 못함 | 새로운 데이터에 일반화하지 못함      |

- **과소적합 대응**: 유용한 특성 추가, 적합한 모델 복잡도 선택, 학습 충분히 수행.
- **과적합 대응**: 모델 복잡도 낮추기, 교차검증, 데이터 추가, 규제(Regularization) 등.
- `L1` 규제: 일부 계수를 0으로 만들 수 있어 특성 선택 효과.
- `L2` 규제: 큰 계수에 패널티를 줌.
