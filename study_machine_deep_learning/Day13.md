## 비지도학습

- 정답 레이블(`y`)없이 특성데이터(`X`)만으로 데이터 속 패턴과 구조를 찾는 머신러닝 방법
- 지도학습은 정답을 배우고 예측하지만, 비지도학습은 데이터끼리의 관계를 탐색

| 구분        | 지도학습             | 비지도학습                       |
| ----------- | -------------------- | -------------------------------- |
| 정답 레이블 | 있음                 | 없음                             |
| 목적        | 분류·회귀 등 예측   | 데이터 구조·패턴 탐색           |
| 예시        | 스팸 분류, 가격 예측 | 고객 군집화, 차원축소, 이상 탐지 |



#### 주요기법

- 군집화(Clustering) : 유사한 데이터를 그룹으로 묶음 - K-Means, 계층적 군집화, DBSCAN
- 차원축소(Dimensionality Reduction) : 많은 특성을 적은 수로 표현 - PCA, t-SNE, UMAP
- 이상치 탐지(Anomaly Detection) : 일반적인 패턴에서 벗어난 관측값 탐지 - Isolation Forest, LOF, Autoencoder
- 추천 시스템 : 사용자나상품의 유사한 패턴을 이용해 추전


#### 군집화

- 서로 비슷한 특성을 가진 데이터들을 클러스터로 묶는것
- 데이터 간 거리(ex) 유클리드 거리)를 활용해 유사성을 판단
- 예 : 구매 패턴별 고객 그룹, 비슷한 색상 픽셀 묶기, 주제별 문서 분류


#### K-Means

1. K-Means는데이터를 미리 정한 K개의 군집으로 나누는 거리 기반 알고리즘
2. 각 데이터를 가장 가까운 중심점에 할당
3. 각 군집에 속한 데이터의 평균으로 중심점 재계산
4. 중심점이 거의 변하지 않거나 최대 반복 횟수에 도달할 때 까지 반복

- K를 사용자가 미리 정해야하며, 특성별 값의 크기 차이가 클 경우 스케일링이 중요

```Python
from skleanfrom sklearn.datasets import make_blobs
from sklearn.cluster import KMeans
import matplotlib.pyplot as plt

# 1. 샘플 데이터 생성 (y는 정답 확인용으로만 반환됨)
X, y = make_blobs(n_samples=300, centers=3, random_state=42)  
#make_blobs(n_samples=300, centers=3) : 군집이 있는 가상 데이터 300개 생성

# 2. 군집화 전 데이터 시각화
plt.scatter(X[:,0], X[:,1])
plt.title('Before Clustering')
plt.show()

# 3. K-Means 모델 생성 및 학습 + 군집 결과 예측
Kmeans = KMeans(n_clusters=3, init='k-means++', random_state=42)
# n_clusters=3 : 군집을 3개로 설정, init='k-means++ : 초기 중심점을 비교적 효율적으로 선택
cluster_labels = kmeans.fit_predict(X)

# 4. 군집 결과와 중심점 시각화 
plt.scatter(X[:,0], X[:,1], c=cluster_labels, cmap='viridis')
plt.scatter(
    kmeans.cluster_centers_[:, 0],
    kmeans.cluster_centers_[:, 1],
    c='red', marker='X', s=180, label='Centroids'
)
plt.legend()
plt.title('K-Means Clustering')
plt.show()

print('군집 레이블:', cluster_labels)
print('군집 중심:', kmeans.cluster_centers_)
print('Inertia:', kmeans.inertia_)
# inertia_ : 데이터에서 해당 중심점까지의 거리 제곱합
```

 


## 엘보우 방법(Elbow Method)

- 군집 수 K를 바꿔가며 **Inertia / WCSS(군집 내 오차제곱합)** 계산
- K가 커질수록 Inertia는 줄어드는 경향이 있음
- 그래프에서 **감소 폭이 급격히 완만해지는 꺾임 지점**을 K 후보로 선택
- 꺾임 지점이 항상 뚜렷한 것은 아님

```Python
result = []

for k in range(1, 10):
    kmeans_temp = KMeans(n_clusters=k, init='k-means++', random_state=42)
    kmeans_temp.fit(X)
    result.append(kmeans_temp.inertia_)

plt.plot(range(1, 10), result, marker='o')
plt.xlabel('Number of Clusters (K)')
plt.ylabel('Inertia / WCSS')
plt.title('Elbow Method')
plt.show()
```


## 실루엣 점수(Silhouette Score)

- 한 데이터가 자신의 군집 안에서 얼마나 가깝고, 다른 군집과 얼마나 떨어져 있는지 평가

```Python
from sklearn.metrics import silhouette_score, silhouette_samples
import numpy as np

# K-Means로 구한 군집 레이블을 이용해 평가
sil_score = silhouette_score(X, cluster_labels)   # 전체 평균 실루엣 점수
sil_values = silhouette_samples(X, cluster_labels) # 데이터별 실루엣 점수

print('평균 실루엣 점수:', sil_score)
print('데이터별 실루엣 점수:', sil_values)

y_lower = 10
for i in range(3):
    values = sil_values[cluster_labels == i]
    values.sort()
    size = values.shape[0]
    y_upper = y_lower + size

    plt.fill_betweenx(np.arange(y_lower, y_upper), 0, values)
    plt.text(-0.05, y_lower + 0.5 * size, str(i))
    y_lower = y_upper + 10

plt.axvline(x=sil_score, color='red', linestyle='--', label='Average')
plt.xlabel('Silhouette Score')
plt.ylabel('Cluster')
plt.legend()
plt.show()
```

엘보우 vs 실루엣

- 엘보우: 군집 내 거리 제곱합의 감소 추이를 보고 K를 고름
- 실루엣: **군집 내부 응집도 + 군집 간 분리도**를 함께 평가
- 여러 K값에 대해 실루엣 평균을 구한 뒤 높은 값을 보이는 K를 후보로 삼을 수 있음



## 차원축소

#### 차원축소가 필요한 이유

- 특성(feature)이 많아질수록 데이터가 희소해지고 분석·학습 비용이 증가할 수 있음 (**차원의 저주**)
- 과적합 위험 완화, 계산량 감소, 노이즈 감소, **2D/3D 시각화** 목적
- 차원축소 과정에서 정보 손실이 발생할 수 있다는 점에 유의


## PCA(Principal Component Analysis, 주성분분석)

- 데이터의 **분산을 가장 잘 설명하는 새 축(주성분)을 찾는 차원축소 방법
- **PC1:** 가장 큰 분산을 설명하는 축
- **PC2:** PC1과 직교하면서 다음으로 큰 분산을 설명하는 축
- 데이터를 새 축에 투영하여 적은 차원으로 표현

적용 흐름: 스케일링 -> 주요 주성분 계산 -> 필요한 주성분 선택 -> 데이터 투영

```Python
from sklearn.decomposition import PCA
from sklearn.datasets import load_iris
from sklearn.preprocessing import StandardScaler
import matplotlib.pyplot as plt

# 1. 데이터 로드
iris = load_iris()
X = iris.data    # 특성 4개
y = iris.target  # 시각화 시 꽃 종류별 색상 표시용
print('원본 차원:', X.shape)

# 2. 특성 표준화
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# 3. PCA 차원축소
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X_scaled)
print('축소된 차원:', X_pca.shape)
print('주성분별 설명 분산 비율:', pca.explained_variance_ratio_)
print('전체 설명 분산 비율:', pca.explained_variance_ratio_.sum())

# 4. 품종별 색상 구분하여 시각화
for target, color, label in zip([0, 1, 2], ['red', 'green', 'blue'], iris.target_names):
    plt.scatter(
        X_pca[y == target, 0],
        X_pca[y == target, 1],
        color=color, label=label
    )

plt.xlabel('PC1')
plt.ylabel('PC2')
plt.title('Iris PCA (4D → 2D)')
plt.legend()
plt.show()
```



## 이상치 탐지(Anomaly / Outlier Detection)

- 대부분의 데이터와 크게 다른 특이한 관측값을 찾는 방법
- 활용 : 금융 사기 거래, 네트워크 비정상 트래픽, 장비 고장 감지 등


## Isolation Forest

- 데이터를 **무작위로 반복 분할**하여 관측값을 분리하는 방식
- 일반적으로 정상 패턴과 크게 다른 값은 **더 적은 분할로 고립되기 쉬움**
- `fit_predict()`의 결과: **`1` = 정상 / `-1` = 이상치**

```Python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.ensemble import IsolationForest

np.random.seed(42)

normal_traffic = np.random.normal(loc=50, scale=10, size=(100, 2))
anomalous_traffic = np.random.uniform(low=100, high=150, size=(10, 2))
traffic_data = np.vstack((normal_traffic, anomalous_traffic))

traffic_df = pd.DataFrame(
    traffic_data, columns=['Request Count', 'Response Time']
)

# 모델 학습과 이상치 판정
model = IsolationForest(
    n_estimators=150,
    contamination=0.09,
    random_state=42
)
traffic_df['Anomaly Score'] = model.fit_predict(
    traffic_df[['Request Count', 'Response Time']]
)

# 시각화
plt.scatter(
    traffic_df['Request Count'],
    traffic_df['Response Time'],
    c=traffic_df['Anomaly Score'].map({1: 'blue', -1: 'red'})
)
plt.xlabel('Request Count')
plt.ylabel('Response Time')
plt.title('Isolation Forest: Normal vs Anomaly')
plt.show()

# 이상치로 판단된 행만 확인
ano_data = traffic_df[traffic_df['Anomaly Score'] == -1]
print(ano_data)
```

| 파라미터                          | 의미                                   |
| --------------------------------- | -------------------------------------- |
| `n_estimators=150`              | 사용하는 트리 수                       |
| `contamination=0.09`            | 예상되는 이상치 비율(약 9%)            |
| `random_state=42`               | 결과 재현을 위한 시드                  |
| `np.vstack(...)`                | 정상·비정상 배열을 세로 방향으로 결합 |
| `df[df['Anomaly Score'] == -1]` | 이상치로 판정된 행 필터링              |


## DBSCAN

- **밀도 기반 군집화(Density-Based Clustering)** 알고리즘
- 데이터가 밀집한 부분을 군집으로 만들고, 다른 군집과 동떨어진 부분은 **노이즈(이상치)**로 분류
- K-Means와 달리 **군집 개수 K를 미리 지정하지 않아도 됨


**주요개념**

- **`eps`**: 이웃으로 판단할 거리 반경
- **`min_samples`**: 핵심점(Core Point)이 되기 위한 반경 내 최소 샘플 수(자기 자신 포함)
- **Core Point:** 주변에 충분한 데이터가 있는 점
- **Border Point:** 핵심점의 이웃이지만 자신은 핵심점이 아닌 점
- **Noise:** 어떤 군집에도 속하지 않은 점, **레이블 `-1`*

```Python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.cluster import DBSCAN

np.random.seed(42)
normal_traffic = np.random.normal(loc=50, scale=10, size=(100, 2))
anomalous_traffic = np.random.uniform(low=100, high=150, size=(10, 2))
traffic_data = np.vstack((normal_traffic, anomalous_traffic))

traffic_df = pd.DataFrame(
    traffic_data, columns=['Request Count', 'Response Time']
)

# DBSCAN 모델 생성 및 군집 할당
dbscan = DBSCAN(eps=10, min_samples=7)
traffic_df['Cluster'] = dbscan.fit_predict(
    traffic_df[['Request Count', 'Response Time']]
)

# -1이면 이상치, 그 외에는 정상 군집으로 표시
traffic_df['Anomaly'] = traffic_df['Cluster'].apply(
    lambda x: 'Anomaly' if x == -1 else 'Normal'
)

plt.scatter(
    traffic_df['Request Count'],
    traffic_df['Response Time'],
    c=traffic_df['Anomaly'].map({'Normal': 'blue', 'Anomaly': 'red'})
)
plt.xlabel('Request Count')
plt.ylabel('Response Time')
plt.title('DBSCAN: Normal vs Anomaly')
plt.show()

print(traffic_df[traffic_df['Cluster'] == -1])
```

| 코드                              | 의미                                  |
| --------------------------------- | ------------------------------------- |
| `DBSCAN(eps=10, min_samples=7)` | 반경 10, 최소 샘플 7로 밀집 영역 탐색 |
| `fit_predict(...)`              | 각 샘플의 군집 번호 반환              |
| `Cluster == -1`                 | DBSCAN이 노이즈로 판단한 데이터       |
| `.apply(lambda x: ...)`         | 군집 번호를 정상/이상 문자열로 변환   |
| `.map({'Normal': 'blue', ...})` | 결과별로 그래프 색상 지정             |


## 알고리즘 비교

| 알고리즘                   | 주요 목적                     | 핵심 원리                                 | 대표 파라미터 / 출력                               |
| -------------------------- | ----------------------------- | ----------------------------------------- | -------------------------------------------------- |
| **K-Means**          | 군집화                        | 가까운 중심점 기준으로 K개 그룹 생성      | `n_clusters`, `cluster_centers_`               |
| **PCA**              | 차원축소                      | 분산이 큰 축으로 투영                     | `n_components`, `explained_variance_ratio_`    |
| **Isolation Forest** | 이상치 탐지                   | 무작위 분할로 빠르게 고립되는 데이터 탐지 | `contamination`, `n_estimators`; 판정 `1/-1` |
| **DBSCAN**           | 밀도 기반 군집화·노이즈 식별 | 이웃 반경과 데이터 밀도로 군집 형성       | `eps`, `min_samples`; 노이즈 `-1`            |

**Isolation Forest와 DBSCAN의 차이**

- Isolation Forest는 분할 기반으로 **이상치 탐지 자체**에 초점을 둠
- DBSCAN은 밀도가 높은 **군집 형성**이 주목적이고, 군집에 속하지 않는 데이터를 노이즈로 취급
- DBSCAN은 `eps` 설정과 특성 스케일에 민감하며, 밀도가 다른 군집이 섞이면 어려울 수 있음
