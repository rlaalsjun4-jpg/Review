## 모듈(Module)과 패키지(Package)

- 모듈 : 함수, 변수, 클래스 등을 정의한 하나의 .py파일
- 패키지 : 여러 모듈을 묶어 관리하는 폴더 구조
- `import` : 다른 모듈에 정의된 기능을 불러오는 키워드
  - ```Python
    # mod1.py
    def add(a, b):
        return a + b

    def sub(a, b):
        return a - b
    ```
  - ```Python
    # main.py — mod1.py와 같은 폴더에 있다고 가정
    import mod1
    print(mod1.add(100, 20))  # 120

    from mod1 import add, sub
    print(sub(10, 3))         # 7

    from mod1 import *
    print(add(100, 20))       # 120
    ```

| 가져오는 방식                 | 사용 방법                  | 설명                  |
| ----------------------------- | -------------------------- | --------------------- |
| `import mod1`               | `mod1.add(1, 2)`         | 모듈 이름을 붙여 사용 |
| `from mod1 import add`      | `add(1, 2)`              | 필요한 함수만 가져옴  |
| `from mod1 import add, sub` | `add(...)`, `sub(...)` | 여러 함수 지정a       |
| `from mod1 import *`        | `add(1, 2)`              | 공개된 이름을 가져옴. |
| `import mod1 as m`          | `m.add(1, 2)`            | 모듈에 별칭 지정      |

---

## 표준 라이브러리

- `datetime`: 날짜 게산
  - ```Python
    import datetime

    day1 = datetime.date(2026, 11, 11)
    day2 = datetime.date(2027, 1, 1)

    print(day1, type(day1))
    print(day2 - day1)            # 날짜 차이: timedelta
    print(day1.year)             # 2026
    print(day1.month)            # 11
    print(day1.day)              # 11
    ```

    - `datetime.date(연, 월, 일)`:날짜 객체 생성
    - 날짜끼리 빼면 `timedelta`객체 반환
    - `(day2 - day1).days`: 날짜 차이를 정수 일수로 확인


- `time`: 시간정보
  - ```Python
    import time

    print(time.time())       # Unix 타임스탬프(초)
    lt = time.localtime()    # 현재 지역 시간 정보를 담은 객체
    print(lt)
    print(f'{lt.tm_year}.{lt.tm_mon}.{lt.tm_mday} {lt.tm_hour}:{lt.tm_min}:{lt.tm_sec}')
    print(time.ctime())              # 읽기 쉬운 시간 문자열
    print(time.ctime().split())      # 문자열을 공백 기준으로 나눈 리스트
    ```

    - `time.time()` : 1970-01-01 UTC를 기준으로 흐른 초
    - `time.localtime()` : 연·월·일·시·분·초 등의 시간 정보
    - `time.ctime()` : 시간을 문자열로 변환


- `random`: 난수 생성
  - ```Python
    import random as r

    print(int(r.random() * 10 + 1))  # 1~10 정수 생성의 한 방식
    print(r.randint(1, 10))         # 1 이상 10 이하 정수

    list_score = []
    for s in range(10):
        list_score.append(r.randint(60, 100))

    print(list_score)
    print(r.choice(list_score))     # 리스트 중 한 개 선택
    print(r.sample(list_score, 5))  # 중복 인덱스 없이 5개 선택
    ```

    - `random()`: 0 이상 1 미만 실수 난수
    - `randint(a, b)`: a~b 범위의 정수 난수 (양 끝 포함)
    - `choice(seq)` : 시퀀스에서 요소 하나 선택
    - `sample(seq, k)` : 시퀀스에서 k개를 비복원 추출


- 운영체제와 파일,폴더

  - ```Python
    import os

    print(os.environ)                 # 환경 변수 확인
    print(os.environ.get('api_key'))  # 특정 환경 변수 조회(없으면 None)
    print(os.getcwd())                # 현재 작업 폴더 확인

    os.mkdir('test')                  # 폴더 생성
    os.rmdir('test')                  # 빈 폴더 삭제
    ```

    - `os.mkdir()` : 폴더 생성
    - `os.rmdir()` : **비어 있는** 폴더 삭제
    - `os.getcwd()` : 작업 디렉터리 확인



- JSON
  - ```Python
    import json

    # 파일(JSON) -> Python 객체(예: dict)
    with open('data.json', 'r', encoding='utf-8') as f:
        data_obj = json.load(f)
        print(data_obj)
        print(type(data_obj))

    # Python 객체 -> JSON 형식 문자열
    # 한글이 읽기 좋은 형태로 저장되도록 ensure_ascii=False 지정
    data_json = json.dumps(data_obj, ensure_ascii=False)
    print(data_json)
    print(type(data_json))          # str
    ```

    - `json.load(f)` : JSON 파일 내용을 Python 객체로 변환
    - `json.loads(s)`: JSON 문자열을 Python 객체로 변환
    - `json.dump(obj, f)` : Python 객체를 JSON 파일에 기록
    - `json.dumps(obj)` : Python 객체를 JSON 문자열로 변환


- 정규표현식 : 문자열에서 일정한 패턴을 검색할 때 사용
- 예 : 전화번호, 이메일, 특정 형식의 문자열 검색

  - ```Python
    import re

    text = '저의 개인 연락처는 010-1234-5678 이고, 사무실 연락처는 02-987-5432 입니다.'
    phone_pattern = r'\d{2,3}-\d{3,4}-\d{4}'

    first_match = re.search(phone_pattern, text)
    if first_match:
        print(first_match.group())
        # 010-1234-5678

    all_match = re.findall(phone_pattern, text)
    print(all_match)
    # ['010-1234-5678', '02-987-5432']
    ```

    - `re.search(패턴, 문자열)`: 첫 번째 일치 항목을 찾고 Match 객체 반환, 없으면 `None`
    - `.group()`: Match 객체에서 일치한 실제 문자열 꺼내기
    - `re.findall(패턴, 문자열)`: 일치하는 문자열들을 리스트로 반환
    - `r'...'` : 백슬래시를 그대로 다루기 쉬운 `**raw string**`표기

---

## Numpy

- numpy : 수치 계산 라이브러리
- 핵심 자료형 : `ndarray`(다차원 배열)


- 배열 생성
  - ```Python
    import numpy as np

    arr = np.array([1, 2, 3, 4, 5])
    print(arr, type(arr))

    print(np.zeros((3, 5)))       # 0으로 채운 3행 5열 배열
    print(np.ones((3, 5)))        # 1로 채운 3행 5열 배열
    print(np.arange(10))          # 0~9
    print(np.linspace(1, 10, 4))  # 1~10 구간을 양 끝 포함 4개로 나눔
    ```

    - `np.array(리스트)` : 리스트를 ndarray로 변경
    - `np.zeros((행, 열))` : 0으로 채운 배열
    - `np.ones((행, 열))` : 1로 채운 배열
    - `np.arange(start, stop, step)` : `range()`처럼 끝값은 제외
    - `np.linspace(start, stop, num)` : 지정 범위를 `num`개 균등 간격 값으로 나눔 (기본적으로 끝값 포함)


- 배열 속성
  - ```Python
    arr1 = np.array([[1, 2, 3], [4, 5, 6]])

    print(arr1.shape)  # (2, 3): 2행 3열
    print(arr1.ndim)   # 2: 2차원
    print(arr1.size)   # 6: 전체 원소 개수
    print(arr1.dtype)  # 원소 자료형(환경에 따라 int32 또는 int64 등)
    ```

    - `.shape`: 배열의형태(각 축 길이)
    - `.ndim`: 차원의 수
    - `size`: 원소 총 개수
    - `dtype`: 원소 자료형


- 인덱싱과 슬라이싱

  - ```Python
    arr = np.array([10, 20, 30, 40, 50])
    print(arr[2])       # 30
    print(arr[-1])      # 50
    print(arr[1:3])     # [20 30]

    arr2 = np.array([[1, 2, 3], [4, 5, 6]])
    print(arr2[0][1])   # 2
    print(arr2[0, 1])   # 2 (같은 위치)
    #슬라이싱 arr[시작:끝]은 끝 인덱스를 포함하지 않는다.
    ```


- 요소별 연산과 브로드캐스팅
  - ```Python
    arr1 = np.array([1, 2, 3])
    arr2 = np.array([4, 5, 6])

    print(arr1 + arr2)  # [5 7 9]
    print(arr1 * arr2)  # [4 10 18]
    print(arr1 ** 2)    # [1 4 9]
    print(arr1 + 10)    # [11 12 13]
    ```

    - NumPy의 `+`, `*`, `**`는 배열 원소별 계산을 수행한다.
    - 브로드캐스팅 : 크기가 다르더라도 호환 가능한 형태라면 연산할 수 있는 기능.
    - 배열의 `*`는 원소별 곱셈이고, 행렬 곱은 `np.dot()` 또는 `@`를 사용한다.


- 배열의 모양 변경
  - ```Python
    arr = np.array([1, 2, 3, 4, 5, 6])

    reshaped_arr = arr.reshape(3, 2)
    print(reshaped_arr)
    # [[1 2]
    #  [3 4]
    #  [5 6]]

    flattened_arr = reshaped_arr.flatten()
    print(flattened_arr)   # [1 2 3 4 5 6]
    ```

    - `.reshape(행, 열)` : 배열의 모양을 변경. 전체 원소 수가 일치해야 한다.
    - `.flatten()` : 다차원 배열을 1차원 배열로 펼침.


- 랜덤 배열
  - ```Python
                              #1이상 45미만의 정수를 무작위로 생성
    ran_arr1 = np.random.randint(1, 45, size=(5, 6)) #size=(5,6) : 5행6열 배열 생성
    print(ran_arr1)
    ```

---

## Pandas

- Pandas : 표 형태의 데이터를 조회,분석,가공하는 라이브러리
  - `Series`: 1차원 데이터 구조 (인덱스와 값)
  - `DataFrame`: 행과 열로 구성된 2차원 표 형태 데이터 구조


- Series 생성

  - ```Python
    import pandas as pd

    s = pd.Series([10, 20, 30])
    print(s)
    print(s[2])  # 30
    ```
- DataFrame 생성

  - ```Python
    data = {
        'name': ['cool', 'hot', 'king'],
        'age': [20, 30, 40],
        'score': [90, 70, 60]
    }
    df = pd.DataFrame(data)
    print(df)
    ```

    - 딕셔너리의 키는 열 이름이 됨
    - 각 리스트의 요소들이 각 열의 행 데이터가 됨


- 데이터 확인
  - ```Python
    print(df.head(3))   # 맨 위 3행
    print(df.tail(2))   # 맨 아래 2행
    print(df.shape)     # (행 개수, 열 개수)
    df.info()           # 컬럼, 타입, 결측치 아닌 값의 수 등
    print(df.describe())  # 수치형 열의 요약 통계
    ```


- 열 선택과 통계
  - ```Python
    print(df['age'])                # 단일 열 선택: Series
    print(df[['name', 'score']])   # 여러 열 선택: DataFrame
    print(df['age'].sum())         # 합계
    print(df['age'].mean())        # 평균
    # df['age'] 는 Series, df[['age']]는 DataFrame
    ```


- 새로운 열 추가와 조건식
  - ```Python
    df1 = df[['name', 'score']].copy()
    df1['passed'] = df1['score'] >= 70  
    print(df1[['name', 'passed']])
    ```


- 결측값 처리
  - ```Python
    df1.loc[1, 'score'] = None  # 결측값 설정
    print(df1.isnull())         # 결측값 여부 확인

    df1['score'] = df1['score'].fillna(100)  # 결측값을 100으로 채움
    ```

    - `None` / `NaN` 등 : 누락된 값(결측값)
    - `.isnull()` : 결측값 여부를 `True / False`로 확인
    - `.fillna(값)` : 결측값을 지정 값으로 채우기
    - `.dropna()` : 결측값이 들어 있는 행 등을 제거하기


- CSV 파일 저장 및 불러오기
  - ```Python
    df.to_csv('output.csv', index=False)   # CSV로 저장
    data_csv = pd.read_csv('output.csv')  # CSV 불러오기
    print(data_csv)
    ```

    - `to_csv()` : DataFrame을 CSV 파일로 저장
    - `read_csv()` : CSV 파일을 읽어서 DataFrame 생성
    - `index=False` : DataFrame 인덱스를 CSV에 별도 열로 기록하지 않음
    - `.loc` : 행·열 라벨 기반 접근
    - `.iloc` : 행·열 위치번호 기반 접근
    - `.sort_values()` : 데이터 정렬
    - `.groupby()` : 그룹별 집계
