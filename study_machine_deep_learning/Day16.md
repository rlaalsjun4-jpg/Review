## ChatGPT API 활용

## 1. 생성 파라미터 제어

- **Temperature**: 다음 토큰의 확률 분포를 조절. 낮을수록 일관성, 높을수록 다양성 증가.
- **Logits → Temperature → Softmax → Sampling**: 토큰 후보 점수 산출, 분포 조절, 확률 변환, 토큰 선택.
- **max_output_tokens**: 최대 출력 토큰 수 제한.
- **reasoning.effort**: 추론 수준 조절.
- **stream**: 답변 생성 중 실시간 출력.
- **Soft Rule**: 프롬프트로 답변 길이 제한.
- **Hard Rule**: 최대 출력 토큰 수로 강제 제한.
- **Truncation**: 토큰 한도에 도달해 응답이 잘리는 현상. `response.status`의 `incomplete` 여부를 확인하고 한도를 조절.

## 2. 프롬프트 템플릿과 모듈화

- 프롬프트 템플릿: 반복 사용하는 형식을 미리 정의하고 `.format()`으로 값을 삽입.
- `instructions`: AI의 역할 및 행동 지침.
- `input`: 사용자 질문 및 처리할 데이터.
- 프롬프트 인젝션: 악의적인 입력으로 기존 지시를 무시하도록 유도하는 공격. 지시와 입력의 분리는 위험을 줄이지만 완전한 방어는 아님.
- 모듈화: 기능별 파일 분리로 가독성, 재사용성, 유지보수성 향상.

| 파일              | 역할                  |
| ----------------- | --------------------- |
| config.py         | 환경설정              |
| prompts.py        | 프롬프트 관리         |
| utils.py          | 공통 기능             |
| chatbot_engine.py | API 호출 및 대화 관리 |
| main.py           | 프로그램 실행         |

## 3. 멀티모달 (Multimodal)

- 텍스트, 이미지, 음성, 영상 등 다양한 형태의 데이터를 처리하는 기술.
- URL 방식: 접근 가능한 이미지 주소 전달.
- Base64 방식: 로컬 이미지의 바이너리 데이터를 문자열로 변환하여 전달. 용량 약 33% 증가, 암호화는 아님.
- OCR: 이미지에 포함된 문자 인식 및 추출.
- Structured Outputs: 미리 정의한 데이터 구조에 맞게 응답 생성.
- `BaseModel`: Pydantic 데이터 구조 정의.
- `Field`: 필드 설명 및 조건 지정.
- `responses.parse()`: 구조화된 응답 생성 및 파싱.
- `output_parsed`: 파싱된 응답 객체.
- 이미지 분석 결과는 할루시네이션 가능성이 있으므로 검증 필요.

## 4. RAG (Retrieval-Augmented Generation)

검색 증강 생성: 외부 문서를 검색하고 검색 결과를 바탕으로 답변을 생성하는 기술.

**동작 과정:** 사용자 질문 → 관련 문서 검색 → 질문과 검색 결과 전달 → LLM 답변 생성

| 구분          | RAG                  | Fine-Tuning          |
| ------------- | -------------------- | -------------------- |
| 방식          | 외부 문서 검색       | 추가 학습            |
| 모델 가중치   | 변경 없음            | 변경                 |
| 정보 업데이트 | 문서 수정            | 추가 학습            |
| 활용          | 사내 문서, 최신 정보 | 특정 말투, 출력 형식 |

- 수동 컨텍스트 주입: 질문과 참고 데이터를 프롬프트에 직접 전달.
- Chunking: 문서를 작은 단위로 분할.
- Overlap: 문맥 유지를 위해 일부 중복.
- Embedding: 텍스트 의미를 숫자 벡터로 변환.
- Vector Store: 벡터 데이터를 저장하고 검색.
- Similarity: 벡터 간 의미적 유사도.


#### 1. API 환경설정

```Python
model = 'gpt-4o-mini'

from dotenv import load_dotenv
import os
from openai import OpenAI
load_dotenv()

# 사용할 키 가져오기!
OPENAI_API_KEY = os.getenv('OPENAI_API_KEY')
client = OpenAI(api_key=OPENAI_API_KEY)
# .env에서 API키를 불러와 클라이언트 생성
```

#### 2.벡터 스토어 생성

```Python
# 벡터 스토어 생성, 여러개 생성 가능
# 파일 객체 => 벡터스토어에 저장
vector_store = client.vector_stores.create(
    name='know_base'
)

vector_store.id
# 벡터 저장소를 생성하고 고유 ID확인
```

#### 3.파일 객체 생성 함수

```Python
def create_file(client,file_path):
    with open(file_path, 'rb') as file_content:
        result = client.files.create(
            file=file_content,
            purpose='assistants'
        )
    print(result.id)
    return result.id      #id로 벡터스토어에 등록
```

#### 4. 파일 생성 및 벡터 스토어 등록

```Python
# file 객체 생성 : create_file -> id
file_id = create_file(client, './howto-sockets.pdf')

# 파일 업로드
client.vector_stores.files.create(
    vector_store_id=vector_store.id,
    file_id=file_id
)
# 파일을 업로드하고 벡터 저장소에 연결
```

#### 5.등록된 파일 확인

```Python
file_list = client.vector_stores.files.list(
    vector_store_id=vector_store.id
)

file_list
# 저장소에 등록된 파일 목록 조회
```

#### 6. File Search 질의응답

```Python
input = '파이썬 코드로 소켓을 만드는 방법을 간단히 설명해줘'

response = client.responses.create(
    model=model,
    input=input,
    tools=[
        {
            'type' : 'file_search',
            'vector_store_ids' : [vector_store.id]
        }

    ],
    tool_choice='required'
)
print(response.output_text)

response
```

`tools`에 검색 도구와 저장소 ID 지정. `tool_choice='required'`는 도구 사용을 요구. `output_text`는 최종 텍스트. 실제 서비스에서는 색인 완료 확인 권장.

## Tools

- Tools는 AI가 웹 검색, 코드실행, 파일검색, 사용자 정의 함수 등 외부 기능을 활용하도록 하는 기능
- Agentic Loop는 요청 분석 -> 도구선택 -> 실행 -> 결과 평가 -> 답변 생성 과정

#### 웹 검색 및 코드 실행

```Python
# Tool => 웹검색, 코드실행(수학연산), 파일서치
response = client.responses.create(
    model=model,
    instructions='최신 정보가 필요하면 웹검색을 활용하고, 데이터 계산이 필요하면 코드를 실행해서 검증하세요',
    input='2026년 11월 현재 반도체관련 주요 트렌드를 검색하고, 파이썬으로 1부터 100까지의 합을 구해서 결과를 출력하고 요약해줘',
    tools=[
        {
            'type' : 'web_search',
        },
        {
            'type' : 'code_interpreter',
            'container' : {'type':'auto'}
        }
    ]
)

response.output

print(response.output_text)
```

`web_search`: 최신 정보 검색. `code_interpreter`: 코드 실행 및 계산. `container`: 격리 실행 환경. `output`: 전체 출력 항목, `output_text`: 텍스트만 추출.


## Function Calling

- Function Calling은 AI가 함수 이름과 매개변수를 반환하고, 개발자 프로그램이 실제 함수를 실행한 후 결과를 AI에게 전달하는 방식
- JSON Schema는 함수의 입력 규격을 정의함

```Python
# 함수
import requests
def get_weather(latitude, longitude):
    response = requests.get(f"https://api.open-meteo.com/v1/forecast?latitude={latitude}&longitude={longitude}&current=temperature_2m,wind_speed_10m&hourly=temperature_2m,relative_humidity_2m,wind_speed_10m")
    data = response.json()
    return data['current']['temperature_2m']
```

`requests.get()`으로 외부 API 요청, `.json()`으로 응답 변환, 현재 기온 반환.

#### 함수 명세 정의

```Python
# 함수의 정의
tools = [
    {
        'type' : 'function',    # 필수
        'name' : 'get_weather', # AI 응답시 사용하는 함수 이름
        'description' : 'Get current temperature for provided coordinates in celsius.',  # 설명
        'parameters' : {
            'type' : 'object',
            'properties' : {
                "latitude": {"type": "number"},
                "longitude": {"type": "number"}
            },
        'required' : ["latitude", "longitude"],
        'additionalProperties' : False   # 웬만하면 False
        },
        'strict' : True  # 웬만하면 True
    }
]
```

`name`: 함수 이름, `description`: 설명, `parameters`: 매개변수 규격, `properties`: 속성, `required`: 필수 속성, `additionalProperties`: 추가 속성 허용 여부, `strict`: 스키마 엄격 준수.

#### 사용자 메세지

```Python
input_message = [
    {
        'role' : 'user',
        'content' : '오늘 파리의 날씨는 어때?'
    }
]
# input = '오늘 파리의 날씨 어때?'    <- 이렇게 써도됨
```

#### 1차 API 호출

```Python
# 1차 요청
response = client.responses.create(
    model=model,
    input=input_message,
    tools=tools
)
response.output
response.output_text
```

API에게 질문과 함수 명세 전달. AI는 함수 호출 정보를 반환할 수 있음

#### 응답 객체 확인

```Python
from pprint import pprint
for item in response.output:
    d_temp = dict(item)
    pprint(d_temp,indent=2,width=50)

tool_call = response.output[0]
print(tool_call.type)
print(tool_call.arguments)   # json => obj해야함
print(tool_call.name)
```

`pprint`로 가독성 있게 출력. `arguments`는 JSON 문자열, `name`은 함수 이름, `call_id`는 호출 식별자. 실제 서비스에서는 출력 항목의 type 확인 권장.

#### JSON 파싱

```Python
import json
args = json.loads(tool_call.arguments)
args
```

`json.loads()로 JSON 문자열을 파이썬 객체로 변환.json.loads()`로 JSON 문자열을 파이썬 객체로 변환.

#### 실제 함수 실행

```Python
# 함수 호출
result = get_weather(args['latitude'],args['longitude'])
print(result)
```

AI가 반환한 위도와 경도를 함수에 전달하여 기온 조회.

#### 메시지 및 결과 병합

```Python
# 메시지 병합
input_message += response.output

# 함수의 실행 결과도 메시지에 병합
input_message.append(
    {
        'type' : 'function_call_output',
        'call_id' : tool_call.call_id,
        'output' : str(result)
    }

)
input_message
```

기존 메시지에 함수 호출 요청과 실행 결과 추가. `call_id`로 요청과 결과 연결

#### 2차 API 호출

```Python
# 2차 호출 => 최종 결과 요청
response2 = client.responses.create(
    model=model,
    input=input_message,
    tools=tools
)

#response2.output
print(response2.output_text)
```

실행 결과를 AI에 전달하여 최종 자연어 답변 생성.

## 8. Streamlit

- 파이썬만으로 웹 애플리케이션을 구현하는 프레임워크.
- 데이터 분석, 대시보드, AI 챗봇에 활용.
- 사용자 인터랙션 발생 시 스크립트 재실행.
- `st.session_state`로 재실행 사이의 대화 기록 및 상태 유지.

| 기능                            | 설명                  |
| ------------------------------- | --------------------- |
| st.title                        | 제목                  |
| st.write / st.markdown          | 데이터 및 텍스트 출력 |
| st.button / st.text_input       | 버튼 및 입력창        |
| st.chat_input / st.chat_message | 채팅 UI               |
| st.file_uploader                | 파일 업로드           |
| st.download_button              | 다운로드              |
| st.write_stream                 | 스트리밍 출력         |
| st.form                         | 입력 위젯 그룹화      |
| st.session_state                | 세션 상태 유지        |

**서비스 구조:** 사용자 브라우저 → Streamlit UI → AI 호출 및 데이터 처리 → OpenAI API / Vector Store

| 파일      | 역할                   |
| --------- | ---------------------- |
| app.py    | UI 및 세션 관리        |
| agent.py  | AI 호출 및 도구 선택   |
| tools.py  | 외부 기능 및 문서 검색 |
| config.py | API 키 및 환경설정     |
