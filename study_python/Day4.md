## 조건문

- 조건문: 조건의 참(True) 또는 거짓(False)에 따라 실행할 코드를 선택하는 구문
  - if : 조건이 참이면 실행
  - elif : 이전 조건이 거짓일 때 추가 조건을 검사
  - else : 앞의 조건이 모두 거짓일 때 실행
  - 조건식 뒤에는 : 을 붙이고 해당 블록은 들여쓰기
  - if 는 여러 개를 독립적으로 사용할 수 있지만 if,elif,else 는 조건을 순서대로 확인하며 처음으로 만족한 블록 하나만 실행함


- 간단한 if,else를 한줄로 표현하기
  - 형식 : 참일 때 값 if 조건식 else 거짓일 때 값
    - ```Python
      score = 50
      msg= 'PASS' if score >= 70 else 'FALE'
      print(msg)    #FAIL
      ```


---

## 반복문

- while : 조건이 참인 동안 코드를 반복 실행
  - 조건이 바뀌지 않으면 무한 반복 발생
    - ```Python
      cnt = 1
      while cnt < 6:
        	print(cnt)
        	cnt+=1
        # 1, 2, 3, 4, 5 
      ```



- break : 반복문을 즉시 종료
- continue : 현재 반복문의 나머지 코드를 건너뛰고 다음 반복으로 이동

  - ```Python
    cnt = 0
    while cnt < 10:
        cnt += 1
        if cnt % 2 == 0:
            continue
        print(cnt)
    # 1, 3, 5, 7, 9 출력
    ```
  - ```Python
    cnt = 0
    while True:
        cnt += 1
        print(cnt)
        if cnt > 4:
            break
    # 1~5 출력하고 반복 종료
    ```
- while True: 와 while 1: 은 모두 항상 참이므로 무한 루프를 만들어 break등 종료 조건을 마련해야함

---

## 반복문

- for : 리스트, 튜플, 문자열 등 반복 가능한 객체의 원소를 차례로 처리
- while 은 주로 조건을 기준으로, for는 원소를 순서대로 순회할 때 사용

  - ```Python
    list_a = [1,2,3,4,5,100]
    for n in list_a:
    	print(n+10,end='\t')  # 11, 12, 13, 14, 15, 110
    ```
- 튜플 언패킹

  - ```Python
    list_b = [(1,2),(3,4),(5,6)]

    for a,b in list_b:
    	print(a,b)
    # 1 2
    # 3 4
    # 5 6
    ```




- range( ) 함수
  - 형식 : range(start, stop, step)
    - start : 시작값(생략하면 0)
    - stop : 포함되지 않는 끝 값
    - step : 증가/감소 간격(생략하면1)
    - range( ) 의 결과는 리스트가 아니라 range객체
      - ```Python
        list(range(5))        # [0, 1, 2, 3, 4]
        list(range(1, 6))     # [1, 2, 3, 4, 5]
        list(range(2, 11, 2)) # [2, 4, 6, 8, 10]

        for num in range(5):
            print(num)  # 0~4 출력
        ```

---

## 컴프리헨션

컴프리헨션 : 반복문을 사용해새로운 자료구조를 간결하게만드는 문법

- 리스트 컴프리헨션
  - ```Python
    # 일반 for 문
    list_c = [1, 2, 3, 4]
    result = []
    for num in list_c:
        result.append(num * 3)
    print(result)  # [3, 6, 9, 12]

    # 컴프리헨션으로 표현
    result = [num * 3 for num in list_c]
    print(result)  # [3, 6, 9, 12]

    #조건 추가
    result_2 = [num * 3 for num in list_c if num % 2 == 0]
    print(result_2)  # [6, 12]
    ```


- 딕셔너리 컴프리핸션
  - ```Python
    fruits = ['apple', 'banana', 'cherry']
    fruit_dict = {f: len(f) for f in fruits}
    print(fruit_dict)
    # {'apple': 5, 'banana': 6, 'cherry': 6}
    ```


- 집합 컴프리핸션
  - ```Python
    numbers = [1, 2, 2, 3, 3, 4, 5, 5]
    result3 = {n ** 2 for n in numbers}
    print(result3)  # {1, 4, 9, 16, 25} (집합 출력 순서는 보장되지 않음)
    ```

---

## 함수

- 함수 : 특정 작업을 수행하는 코드를 묶어 필요할 때 재사용하는 기능
- 장점 : 중복코드 감소, 가독성 향상 유지보수 용이
- 형식 : def 함수이름(매개변수) :
- return : 함수 실행을종료하면서 결과값을 호출한 곳에 전달
  - ```Python
    def add(n1:int,n2:int) -> int:
      	'''숫자 두개를 받아 덧셈 후 반환'''
      	result=n1+n2
      	return result
    print(add(100,30))  #130
    ```


- 매개변수 : 함수를 정의할 때 괄호 안에 작성하는 변수(n1,n2)
- 인수 : 함수를 호출할 때 실제 전달하는 값 ( 100, 30 )
- 타입 힌트 =>  n1 : int : 매개변수에 정수형 값이 올 것으로 기대한다는 표시
- -> int : 반환 값이 정수형일 것으로 기대한다는 표시
- 타입힌트 자체가 실행 중 타입을 강제로 검사해주지는 않음


- 입력값과 반한값에 따른 함수 형태
  - 입력 O 반환 O => ex)add(a,b) : 값 받아 계산하고 반환
  - 입력 X 반환 O => ex)say() : 입력 없이 값 반환
  - 입력 O 반환 X => ex)display_msg(msg) : 입력 받아 출력 등만 수행
  - 입력 X 반환 X => ex)say_hello() : 입력 없이 출력 등만 수행
    - ```Python
      # 입력 O / 반환 X
      def display_msg(message: str) -> None:
          print('알림:', message)

      display_msg('안녕하세요!')

      # 입력 X / 반환 X
      def say_hello():
          print('Hello~~~')
          print('반갑습니다.')

      result = say_hello()  # 문장 출력
      print(result)         # None
      ```



- 위치 인수 : 전달한 순서대로 매개변수에 대입
- 키워드 인수 : 매개변수명 = 값 형태로 전달하여 이름을 기준으로 대입
  - ```Python
    def sub(n1, n2):
        return n1 - n2

    print(sub(3, 10))            # -7: 위치에 따라 전달
    print(sub(n2=3, n1=10))      # 7: 이름을 지정해 전달
    ```


- 가변인수 : *args
- 개수가 정해지지 않은 위치 인수를 튜플로 모아 받음
  - ```Python
    def display_all(*args):
        print(args)
        print(type(args))
        print(sum(args))

    display_all(1, 2, 3, 4, 5)
    # (1, 2, 3, 4, 5)
    # <class 'tuple'>
    # 15

    def add_mul_process(p_type, *args):
        print(args)
        print('연산 타입:', p_type)

    add_mul_process('+', 1, 2, 3, 4, 5)
    ```


- 키워드 전용매개변수 : *
  - 매개변수 목록에 단독 *가 있으면, 그 뒤의 매개변수는 키워드 인수로만 전달 가능
  - *args 뒤에 오는 일반 매개변수도 키워드 전용이 됨
    - ```Python
      def connect_server(host: str, port: int, *, timeout: int = 30, retry: bool = True):
          print(host, port, timeout, retry)

      connect_server('localhost', 8080, timeout=10, retry=False)
      # localhost 8080 10 False

      # connect_server('localhost', 8080, 10)  # 오류: timeout을 위치 인수로 전달
      ```


- 가변키워드 인수 : **kwargs
  - ```Python
    def print_kwargs(**kwargs):
        print(kwargs, type(kwargs))

    print_kwargs(name='son', age=30)
    # {'name': 'son', 'age': 30} <class 'dict'>
    ```
- *args => 여러 위치 인수 받기 => 모이는 자료형 : 튜플
- **kwargs => 여러 키워드 인수 받기 => 모이는 자료형 : 딕셔너리


- 언패킹 연산자 */**

  - 함수를 호출할 때 * 은 리스트,튜플 등의 요소를 위치 인수로 풀어서 전달
  - 함수를 호출할 때 ** 은 딕셔너리의 키/값을 키워드 인수로 풀어서 전달
    - ```Python
      def print_coords(x, y, z, w):
          print(f'x={x}, y={y}, z={z}, w={w}')

      point_a = [1, 2]
      point_b = [3, 4]
      print_coords(*point_a, *point_b)

      dict_a = {'x': 10, 'y': 20}
      dict_b = {'z': 30, 'w': 40}
      print_coords(**dict_a, **dict_b)
      ```

---

## 지역변수와 전역변수

- 지역변수 : 함수 안에서 정의된 변수
  - 기본적으로 함수 내부에서만 사용 가능
  - 매개변수도 해당 함수의 지역변수
  - 함수 밖에서 지역변수를 참조하면 NameError 가 날 수 있음
    - ```Python
      def example(a):
          num3 = 300
          print(a)
          print(num3)

      example(123)  # 123, 300 출력
      # print(num3)  # 오류: 함수 밖에서 num3을 찾을 수 없음
      # print(a)     # 오류: 함수의 매개변수 a는 함수 밖에 없음
      ```


- 전역 변수 : 함수 밖(모듈 수준)에서 정의한 변수
  - 함수 내부에서 전역변수를 재할당 하려면 global 변수명 을 사용
    - ```Python
      aa = 10000

      def var_test(a):
          global aa
          aa = aa + 12

          print('함수내부에서 호출:', aa)
          print('매개변수 a:', a)

          num3 = 300
          print('함수 내부에서 사용되는 변수 num3:', num3)

      print(aa)      # 10000
      var_test(123)  # aa=10012, a=123, num3=300 출력
      print(aa)      # 10012
      ```
    - global aa : 함수 안의 aa 가 바깥에 있는 전역변수임을 선언
    - aa = aa + 12 : 전역변수의 값이 10000 -> 10012 로 변경됨
    - 주의: global 을 남용하면 함수가 외부 상태에 의존하여 이해,테트스하기 어려워질 수 있음
