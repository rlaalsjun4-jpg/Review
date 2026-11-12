## 인공지능 데이터 처리 과정

데이터 파이프라인 6단계

1. 수집(Data Collection) : API, 웹 스크래핑 DB조회, 센서 데이터
2. 저장(Data Storage) : CSV, JSON, SQL(RDBMS), NoSQL, Data Lake
3. 전처리(Data Preprocessing) : 결측치 처리, 이상치 제거, 스케일링, 인코딩
4. 피처 엔지니어링(Feature Engineering) : 파생변수 생성, 자원축소, 피처 선택
5. 학습,평가(Modeling & Evaluation) : 모델 학습, 하이퍼파라미터 튜닝
6. 서빙, 배포(Serving & Deployment) : API 서버 및 추론 파이프라인 구축

핵심 : 데이터 수집은 이후 분석과 모델링의 출발점

## 데이터 수집 방법

- 웹 스크래핑 : 웹의 HTML에서 필요한 정보 추출 ex) BeautifulSoup으로 상품명,가격 추출
- API 활용 : 서버에 데이터 요청, 주로 JSON 수신 ex) 날씨 API, 공공데이터 API
- 데이터베이스 조회 : 저장된 데이터를 SQL로 조회 ex) 고객정보 조회
- 공개 데이터셋 : 제공된 파일,데이터를 다운로드 ex) Kaggle, AI Hub, KOSIS
- IoT/센서 : 장치가 생성하는 실시간 데이터 수집 ex) 온도센서, GPS, MQTT
- 소셜미디어 : 플랫폼 API로 데이터 수집 ex) 소셜미디어 게시물

---



## HTTP

- `GET` : 서버에서 데이터 조회
- `POST`: 서버로 새 데이터 전송
- `PUT`: 데이터 갱신
- `DELETE`: 데이터삭제
- `status_code`: HTTP 응답 상태 코드 (예 : 200 성공, 4xx 클라이언트오류, 5xx 서버오류)
- `response.text`: 응답 내용을 문자열로 반환
- `response.json()`: JSON 응답을 파이썬 객체(딕셔너리, 리스트 등)로 변환

---



## requests

- ```Python
  import requests

  url = 'https://jsonplaceholder.typicode.com/posts'
  res = requests.get(url)

  if res.status_code == 200:
      print('정상응답')
      data_obj = res.json()
      print(type(data_obj))  # 이 API에서는 list

      for post in data_obj:
          print(f"title: {post['title'][:10]}")
          print(f"body: {post['body'][:10]}")
          print('-' * 20)
  else:
      print('정상적인 수신이 되지 않았습니다.')
      print(f'오류발생: {res.status_code}')
  ```

  - `requests.get(url)` : 서버에 GET 요청
  - `res.status_code == 200` : 성공 응답 여부 확인
  - `res.json()` : JSON 데이터를 파이썬 자료형으로 변환

---



## HTML 기초

- HTML : 웹 문서의 구조를 정의하는 마크업 언어
- 태그(Tag): `h1`, `p`, `a`처럼 문서 요소를 표시
- 속성(Attribute) : 태그의 추가 정보 (`id`, `class`, `href`, `src`)
- DOM : HTML 요소를 부모-자식 관계의 트리로 표현한 구조
  - ```HTML
    <h1 id="title">안녕하세요</h1>
    <p class="description">본문입니다.</p>
    <a href="https://example.com">링크</a>
    ```

---



## BeautifulSoup

```Python
from bs4 import BeautifulSoup

html_doc = """
<html>
  <head><title>Example Page</title></head>
  <body>
    <h1>Welcome to Web Scraping</h1>
    <p class="description">This is an example paragraph.</p>
    <a href="https://example.com">Visit Example</a>
  </body>
</html>
"""

soup = BeautifulSoup(html_doc, 'html.parser')

print(soup.title.string)  # Example Page
print(soup.h1.string)     # Welcome to Web Scraping
print(soup.p.string)      # This is an example paragraph.
print(soup.a['href'])     # https://example.com
```

- `BeautifulSoup(html_doc, 'html.parser')`: HTML 문자열 파싱
- `.string`: 단일 텍스트 노드 가져오기
- `['href']`: 태그의 `href` 속성 값 추출

---

## find / find_all / select / select_one

| 메서드                         | 기능                              | 예시                                |
| ------------------------------ | --------------------------------- | ----------------------------------- |
| `find('a')`                  | 첫 번째 일치 태그                 | `soup.find('a')`                  |
| `find_all('a')`              | 일치하는 모든 태그 목록           | `soup.find_all('a')`              |
| `select('a')`                | CSS 선택자로 모든 요소 선택       | `soup.select('a')`                |
| `select_one('.description')` | CSS 선택자로 첫 번째 요소 선택    | `soup.select_one('.description')` |
| `.get('href')`               | 속성 값 가져오기, 없으면 `None` | `tag.get('href')`                 |
| `.get_text()`                | 태그 안의 텍스트 추출             | `tag.get_text()`                  |

CSS 선택자

- `a`: 태그 이름
- `.description`: `class="description"`
- `#title`: `id="title"`
- `.product-list .item`: 특정 부모 영역 안의 하위 요소

---

## 웹 스크래핑

```Python
import requests
from bs4 import BeautifulSoup

url = 'https://news.ycombinator.com/'
res = requests.get(url)
soup = BeautifulSoup(res.text, 'html.parser')

for item in soup.find_all('a'):
    href_text = item.get('href')
    title_text = item.get_text()

    if href_text and title_text:
        print(title_text, ' | ', href_text)
```

- 실행 흐름 : `requests.get()` → 응답 HTML → `BeautifulSoup` 파싱 → `find_all('a')` → `href`와 텍스트 추출

---

## REST API와 수집 시 주의사항

- URL / Resource : 조회·변경할 자원을 나타냄
- HTTP 메서드 : GET, POST, PUT, DELETE 등
- Header : `Content-Type`, `Authorization` 등의 부가 정보
- Body : 서버에 전달하는 데이터(주로 JSON)

### 웹 크롤링 주의사항 (강의)

- `robots.txt`에서 접근 제한 및 크롤링 정책 확인
- `User-agent`, `Disallow`, `Allow`, `Crawl-delay` 지시어 이해
- 짧은 간격의 과도한 요청을 피하고 서버 부하 최소화
- API 키·인증정보를 소스 코드에 노출하지 않기
- 수집 전 사이트 이용약관 및 관련 권리·정책 확인

## 데이터 저장 형식

| 형식   | 특징                         | 주 사용처             |
| ------ | ---------------------------- | --------------------- |
| CSV    | 쉼표로 구분한 가벼운 텍스트  | 표 형태 데이터 교환   |
| Excel  | 시트 기반 표와 분석          | 보고서·업무 데이터   |
| JSON   | 키-값 구조, 중첩 데이터 가능 | API 데이터 교환       |
| SQLite | 파일 기반 관계형 DB          | 테이블 저장, SQL 조회 |

---

## SQLite

- SQLite : 파일 형태로 보관할 수 있는 관계형 데이터베이스(RDBMS) 이다.
- Python의 기본 `sqlite3`모듈로 데이터 베이스에 연결할수 있다.
- SQL을 사용해 데이터를 생성, 조회, 수정, 삭제한다.
  - `CREATE TABLE`: 테이블 생성
  - `INSERT INTO`: 데이터 삽입
  - `SELECT`: 데이터 조회
  - `UPDATE`: 데이터 수정
  - `DELETE FROM`: 데이터 삭제


## 데이터베이스 연결과 종료

```Python
import sqlite3

conn = sqlite3.connect('first1.db')  #connect('first1.db') : DB파일연결 ( 파일이 없으면 생성)
print('데이터베이스 연결 성공!')   
conn.close()   #연결 종료
```



## 테이블 생성

```Python
import sqlite3

conn = sqlite3.connect('first1.db')
cursor = conn.cursor()    #SQL을 실행할 커서 객체 생성

sql_create = '''
CREATE TABLE user (
    id INTEGER PRIMARY KEY AUTOINCREMENT,  
    name TEXT NOT NULL,
    age INTEGER,
    city TEXT
)
'''

cursor.execute(sql_create)  #SQL 문 실행
print('테이블 생성!')
conn.close()
```



## 데이터 입력

```Python
conn = sqlite3.connect('first1.db')
cursor = conn.cursor()

sql_insert = 'INSERT INTO user (name, age, city) VALUES (?, ?, ?)'
cursor.execute(sql_insert, ('COOL', 20, 'SEOUL'))

conn.commit()
conn.close()
```

- `?`는 SQL 값 자리표시자이며 값을 별도 전달할 때 사용
- `commit()`은 입력, 수정, 삭제 등의 변경사항을 저장
- 문자열을 직접 이어 붙이는 것보다 매개변수 전달 방식이 안전


## 데이터 조회

```Python
conn = sqlite3.connect('first1.db')
cursor = conn.cursor()

sql_select = 'SELECT * FROM user'
cursor.execute(sql_select)
rows = cursor.fetchall()

print(rows)
for id, name, age, city in rows:
    print(id, name, age, city)

conn.close()
```

- `SELECT * FROM user`: `user` 테이블의 모든 열 조회
- `fetchall()`: 조회 결과의 모든 행 가져오기
- `for id, name, age, city in rows`: 행의 값들을 변수로 나누어 출력(튜플 언패킹)
