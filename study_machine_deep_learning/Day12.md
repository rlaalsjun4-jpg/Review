## 분류

- 지도학습의 한 종류로, 입력 데이터 `X`와 정답 레이블 `y`를 함께 사용해 학습함
- 학습한 모델이 새로운 데이터를 정해진 클래스(범주)로 분류함
- 이진분류 : 클래스가 2개인 경우 ex) 정상(0) / 사기(1)
- 다중 클래스 분류 : 클래스가 3개 이상인 경우 ex) Iris의 꽃 품종 3종

일반적인 분류 흐름

- 데이터준비 -> X(특성), y(정답) 분리 -> train / test 분할
- -> 필요시 전처리(스케일링 등) -> 모델학습(fit)
- -> 예측(predict) -> 성능 평가


## 로지스틱 회귀(Logistic Regression)

- 이름에는 회귀가 있지만 분류에 사용하는 모델
- 여러 특성의 선형 결합을 시그모이드 함수에 통과시켜 클래스 1일 확률을 계산함

```Python
from sklearn.linear_model import LogisticRegression

model = LogisticRegression(max_iter=1000)
model.fit(X_train, y_train)
y_pred = model.predict(X_test)          # 예측 클래스
proba = model.predict_proba(X_test)     # 클래스별 예측 확률
```

| 매개변수/메서드             | 의미                                                  |
| --------------------------- | ----------------------------------------------------- |
| `max_iter=1000`           | 최적화 최대 반복 횟수. **클래스 가중치가 아님** |
| `class_weight='balanced'` | 데이터 비율을 반영한 클래스별 가중치 자동 설정        |
| `fit(X_train, y_train)`   | 학습                                                  |
| `predict(X_test)`         | 클래스 예측                                           |
| `predict_proba(X_test)`   | 클래스별 확률 반환                                    |


## 분류 성능 평가 지표

#### 혼동행렬(Confusion Matrix)

- 1 을 탐지 대상(Positive)으로 간주한 이진 분류의 경우

|                  | 실제 0 (Negative)        | 실제 1 (Positive)        |
| ---------------- | ------------------------ | ------------------------ |
| **예측 0** | TN: 정상을 정상으로 예측 | FN: 사기를 정상으로 놓침 |
| **예측 1** | FP: 정상을 사기로 오탐   | TP: 사기를 사기로 탐지   |



## 평가지표

| 지표              | 수식                                                | 해석                                   |
| ----------------- | --------------------------------------------------- | -------------------------------------- |
| 정확도(Accuracy)  | `(TP + TN) / (TP + TN + FP + FN)`                 | 전체에서 정답을 맞힌 비율              |
| 정밀도(Precision) | `TP / (TP + FP)`                                  | 1이라고 예측한 것 중 실제 1인 비율     |
| 재현율(Recall)    | `TP / (TP + FN)`                                  | 실제 1을 얼마나 놓치지 않고 탐지했는가 |
| F1 Score          | `2 × Precision × Recall / (Precision + Recall)` | 정밀도와 재현율의 조화평균             |

- **Precision이 중요할 때**: 정상 대상을 잘못 양성으로 분류하는 FP를 줄여야 할 때.
- **Recall이 중요할 때**: 실제 양성을 놓치는 FN을 줄여야 할 때(질병, 사기 탐지 등).
- **F1 Score**: Precision과 Recall 간 균형을 확인할 때.
- 데이터가 불균형하면 **Accuracy만으로는 모델을 평가하기 어렵다.**

```Python
from sklearn.metrics import (
    accuracy_score, precision_score, recall_score,
    f1_score, confusion_matrix, classification_report
)

print('정확도:', accuracy_score(y_test, y_pred))
print('정밀도:', precision_score(y_test, y_pred))
print('재현율:', recall_score(y_test, y_pred))
print('F1:', f1_score(y_test, y_pred))
print(confusion_matrix(y_test, y_pred))
print(classification_report(y_test, y_pred))
```



## 데이터 불균형과 가중치

**클래스 불균형(DataImbalance)** : 정상 데이터는 매우 많고 사기처럼 중요한 소수 클래스는 극히 적은 경우

#### 대응 방법

1. `class_weight='balanced'`: 소수 클래스의 분류 오류에 더 큰 가중치
2. 임계값 변경: 양성 판단 확률 기준을 조절
3. 오버샘플링 / SMOTE: 소수 클래스 데이터 증가 또는 합성
4. 언더샘플링: 다수 클래스 일부 제거

```Python
# 자동 가중치
lr = LogisticRegression(
    class_weight='balanced', max_iter=1000
)

# 직접 가중치 지정: 정상=1, 사기=10
lr = LogisticRegression(
    class_weight={0: 1, 1: 10}, max_iter=1000
)
```

- `class_weight`를 높이면 일반적으로 소수 클래스의 Recall 상승 가능성이 있지만 FP증가로 Precision 및 F1이 하락할수 있어 지표를 함께 비교해야 함


## 스케일링

- 특성 간 숫자 범위를 표준화한다. 클래스 불균형 자체를 해결하는 방법은 아님

```Python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)          # 위에서 X_train에 기준을 맞췄기때문에
												  # X_test는 fit_transform이아닌 transform
```

- `fit()`: 학습 데이터로 기준(평균,표준편차) 계산
- `transform()`: 계산된 기준으로 변환
- `fit_transform()`: 기준 계산과 변환을 함께 수행
- 테스트 데이터에 `fit()`이나 `fit_transform()`을 새로 적용하지 않는다(데이터 누수 방지)


## KNN 알고리즘

- K-Nearest Neighbors(K-최근접 이웃) : 예측하려는 데이터와 가까운 `K`개 이웃을 찾아 다수결(분류) 또는 평균(회귀)으로 결과를 결정하는 거리 기반 방법

**동작순서**

1. 학습 데이터와 레이블 준비
2. 새 데이터와 학습 데이터 간 거리 계산
3. 가장 가까운 K개 이웃 선택
4. 분류에서는 다수결 회귀에서는 평균 등으로 예측

**특징**

- 학습 시 복잡한 파라미터 추정보다 데이터 저장이 중심인 Lazy Learning 방식
- 예측 시 거리 계산이 많아 대용량 데이터에서는 느릴 수 있음
- 변수 간 값의 범위 차이에 민감해서 스케일링이 중요


| K값    | 특징                                                         |
| ------ | ------------------------------------------------------------ |
| 작은 K | 주변 변화에 민감, 노이즈와 과적합 위험 증가                  |
| 큰 K   | 예측이 안정적일 수 있지만 지역적인 특징을 놓쳐 과소적합 가능 |
| 홀수 K | **이진 분류**에서 투표 동률을 줄이는 데 도움           |



**거리 방식 및 가중치**

| 매개변수        | 예시            | 의미                                               |
| --------------- | --------------- | -------------------------------------------------- |
| `n_neighbors` | `3`, `5`    | 참고할 이웃의 개수                                 |
| `weights`     | `'uniform'`   | 모든 이웃의 투표 비중 동일(기본값)                 |
| `weights`     | `'distance'`  | 가까운 이웃에 더 높은 비중 부여                    |
| `metric`      | `'euclidean'` | 직선거리                                           |
| `metric`      | `'manhattan'` | 각 좌표 차이의 절댓값을 합한 거리                  |
| `metric`      | `'minkowski'` | 일반적인 거리 방식(기본 `p=2`는 유클리드와 같음) |

```Python
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.neighbors import KNeighborsClassifier
from sklearn.metrics import accuracy_score, classification_report

iris = load_iris()
X = iris.data
y = iris.target

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

k = 3
knn = KNeighborsClassifier(n_neighbors=k)
knn.fit(X_train, y_train)

y_pred = knn.predict(X_test)
print('정확도:', accuracy_score(y_test, y_pred))
print(classification_report(y_test, y_pred,
                            target_names=iris.target_names))

# 새로운 샘플 1개(특성 4개) 예측
new_data = [[5.3, 3.0, 1.3, 0.2]]
new_pred = knn.predict(new_data)
print(iris.target_names[new_pred])
```



## K값 선택과 교차검증

**5-Fold 교차검증으로 최적 K찾기**

- 데이터를 5부분으로 나누고 학습,검증을 번갈아 수행
- 각 K마다 검증 점수 5개를 평균 낸다
- `scores`는 배열, ``scores.mean()``은 숫자 하나
- 스케일링을 교차검증 바깥에서 하지 않도록 `Pipeline`활용

```Python
from sklearn.model_selection import cross_val_score
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.neighbors import KNeighborsClassifier

best_k = 1
best_score = -1

for k in range(1, 21, 2):
    pipeline = make_pipeline(
        StandardScaler(),
        KNeighborsClassifier(n_neighbors=k)
    )
    scores = cross_val_score(pipeline, X_train, y_train, cv=5)
    mean_score = scores.mean()

    if mean_score > best_score:
        best_k = k
        best_score = mean_score  # scores(배열) 아님

print(f'최적 K: {best_k}, 교차검증 평균 정확도: {best_score:.4f}')
```

**최종 모델 평가**

```Python
final_knn = make_pipeline(
    StandardScaler(),
    KNeighborsClassifier(n_neighbors=best_k)
)
final_knn.fit(X_train, y_train)
y_pred = final_knn.predict(X_test)
print(classification_report(y_test, y_pred))
```

- 반복이 끝난 뒤 `k`는 마지막 반복값이다. **최적값은 `best_k`를 사용**
- 교차검증을 기본 설정으로 했다면 최종 모델도 우선 같은 설정으로 비교한다. 거리 방식/가중치를 바꾸려면 그 조합도 검증해야 함
