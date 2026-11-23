## 딥러닝

#### 핵심개념

- **딥러닝**: 인공신경망을 이용해 데이터 패턴을 학습하는 머신러닝의 한 분야
- **입력층(Input)**: 입력 특성을 받는 층
- **은닉층(Hidden)**: 데이터의 특징과 패턴을 학습하는 층
- **출력층(Output)**: 최종 예측값을 내보내는 층
- **가중치(Weight)**: 입력 신호에 곱하는 학습 대상 값
- **편향(Bias)**: 가중합에 더해지는 학습 대상 값

| 활성화 함수 | 역할                                  | 대표 사용처      |
| ----------- | ------------------------------------- | ---------------- |
| ReLU        | 음수는 0, 양수는 그대로 출력          | 은닉층           |
| Sigmoid     | 0~1 사이 값 출력                      | 이진 분류 출력층 |
| Softmax     | 각 클래스 확률의 합이 1이 되도록 출력 | 다중 분류 출력층 |



## 이진 분류 모델

#### 라이브러리, 데이터 준비

```Python
import numpy as np
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense, Input, Dropout
from tensorflow.keras.callbacks import EarlyStopping
from sklearn.model_selection import train_test_split
from sklearn.datasets import make_classification

X, y = make_classification(
    n_samples=1000,
    n_features=20,
    n_classes=2,
    random_state=42
)

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)
```

- `n_samples=1000`: 샘플 1,000개 생성
- `n_features=20`: 입력 특성 20개
- `n_classes=2`: 두 클래스 분류

#### 모델 생성

```Python
model = Sequential([
    Input(shape=(X_train.shape[1],)),
    Dense(8, activation='relu'),
    Dense(1, activation='sigmoid')
])
```

- `Sequential()`: 층을 순서대로 쌓는 모델
- `Input(shape=(X_train.shape[1],))`: 입력 특성 개수를 자동으로 지정
- `Dense(8, activation='relu')`: 뉴런 8개인 은닉층
- `Dense(1, activation='sigmoid')`: 이진 분류 출력층

#### 컴파일, 학습, 평가, 예측

```Python
model.compile(
    optimizer='adam',
    loss='binary_crossentropy',
    metrics=['accuracy']
)

history = model.fit(
    X_train, y_train,
    validation_split=0.2,
    epochs=100,
    batch_size=32,
    verbose=1
)

test_loss, test_accuracy = model.evaluate(X_test, y_test)
print(f"테스트 손실: {test_loss:.4f}, 테스트 정확도: {test_accuracy:.4f}")

predictions = model.predict(X_test[:5])
print("예측 결과:", predictions)
```

- `compile()`: 최적화 방법, 손실 함수, 평가지표 설정
- `optimizer='adam'`: 가중치를 업데이트하는 최적화 알고리즘
- `loss='binary_crossentropy'`: 이진 분류 손실 함수
- `metrics=['accuracy']`: 정확도 확인
- `validation_split=0.2`: 학습 데이터 중 20%를 검증에 사용
- `epochs=100`: 전체 학습 데이터 최대 100회 반복
- `batch_size=32`: 한 번에 32개 샘플씩 학습
- `evaluate()`: 테스트 성능 평가
- `predict(X_test[:5])`: 테스트 샘플 5개에 대한 예측값 출력


## 과적합(Overfitting)

- 학습 데이터에는 잘 맞지만 새로운 데이터에는 성능이 떨어지는 현상


#### 데이터 70 : 15 : 15  분

```Python
X_train, X_temp, y_train, y_temp = train_test_split(
    X, y, test_size=0.3, random_state=42
)

X_val, X_test, y_val, y_test = train_test_split(
    X_temp, y_temp, test_size=0.5, random_state=42
)
```

- **Train 70%**: 학습
- **Validation 15%**: 학습 중 성능 확인 및 조기 종료 기준
- **Test 15%**: 최종 평가
- 임시 데이터 `X_temp`는 전체의 30%이고, 이를 반으로 나누면 15%씩 됨


#### Dropout, EarlyStopping

```Python
model = Sequential([
    Input(shape=(X_train.shape[1],)),
    Dense(64, activation='relu'),
    Dropout(0.5),
    Dense(32, activation='relu'),
    Dense(1, activation='sigmoid')
])

model.compile(
    optimizer='adam',
    loss='binary_crossentropy',
    metrics=['accuracy']
)

early_stopping = EarlyStopping(
    monitor='val_loss',
    patience=5,
    restore_best_weights=True
)

history = model.fit(
    X_train, y_train,
    validation_data=(X_val, y_val),
    epochs=200,
    batch_size=32,
    callbacks=[early_stopping]
)

test_loss, test_acc = model.evaluate(X_test, y_test)
print(test_acc, test_loss)
```

- `Dropout(0.5)`: **학습 중** 해당 층 출력의 50%를 무작위로 비활성화해 과적합 완화
- `EarlyStopping()`: 검증 성능이 개선되지 않으면 학습 중단
- `monitor='val_loss'`: 검증 손실 감시
- `patience=5`: 검증 손실이 5번의 epoch 동안 개선되지 않으면 중단
- `restore_best_weights=True`: 가장 좋은 검증 손실을 기록한 가중치 복원
- `callbacks=[early_stopping]`: 학습 중 조기 종료 기능 실행


## MNIST 다중 분류

```Python
from tensorflow.keras.utils import to_categorical
from tensorflow.keras.datasets import mnist

(X_train, y_train), (X_test, y_test) = mnist.load_data()
```

- `mnist`: 0~9의 손글씨 숫자 이미지 데이터
- 이미지 한 장은 `28 × 28` 픽셀, 분류 클래스는 10개


#### 전처리

```Python
# 이미지의 28×28 픽셀을 길이 784의 벡터로 변환하고 정규화
X_train = X_train.reshape(-1, 784) / 255.0
X_test = X_test.reshape(-1, 784) / 255.0

# 정답 레이블 원핫 인코딩
y_train = to_categorical(y_train, num_classes=10)
y_test = to_categorical(y_test, num_classes=10)
```

- `reshape(-1, 784)`: 행 개수는 자동 계산, 각 이미지는 784개의 값으로 변환
- `/ 255.0`: 픽셀값 0~255를 0~1로 정규화
- `to_categorical()`: 정답 숫자를 원핫 인코딩
- 예: 숫자 3 → `[0, 0, 0, 1, 0, 0, 0, 0, 0, 0]`

#### 모델 생성, 컴파일 및 학습

```Python
model = Sequential([
    Input(shape=(784,)),
    Dense(64, activation='relu'),
    Dense(10, activation='softmax')
])

model.compile(
    optimizer='adam',
    loss='categorical_crossentropy',
    metrics=['accuracy']
)

model.fit(
    X_train, y_train,
    epochs=10,
    batch_size=32,
    validation_split=0.2
)
```

- `Dense(10, activation='softmax')`: 숫자 0~9에 대한 10개의 클래스 확률 출력
- `categorical_crossentropy`: **원핫 인코딩된 레이블**을 사용하는 다중 분류 손실 함수


#### 평가 및 예측

```Python
test_loss, test_acc = model.evaluate(X_test, y_test)
print(test_acc, test_loss)

predictions = model.predict(X_test[:1])
print(predictions)
print(predictions.argmax())
```

- `predict(X_test[:1])`: 테스트 이미지 첫 번째 1개 예측
- `predictions`: 0~9 각 숫자에 대한 예측 확률
- `argmax()`: 가장 큰 확률을 가진 인덱스, 즉 예측 숫자 반환


### 이진 분류와 다중 분류 비교

| 구분        | 이진 분류               | 다중 분류(MNIST)             |
| ----------- | ----------------------- | ---------------------------- |
| 클래스 수   | 2개                     | 10개                         |
| 출력층      | `Dense(1)`            | `Dense(10)`                |
| 출력 활성화 | `sigmoid`             | `softmax`                  |
| 손실 함수   | `binary_crossentropy` | `categorical_crossentropy` |
| 레이블      | 0, 1                    | 원핫 인코딩                  |

**실습 순서**: 데이터 준비 → 분할·전처리 → 모델 생성 → 컴파일 → 학습 → 평가 → 예측
