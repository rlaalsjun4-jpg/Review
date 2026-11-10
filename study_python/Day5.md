## 람다함수 (lambda)

- 람다함수 : def 대신 간단한 함수를 한줄로 작성하는 방식
- 기본 구조 : lambda 매개변수1, 매개변수2 : 반환할_표현식

  - ```Python
    add = lambda n1, n2 : n1 + n2

    print(type(add))   # <class 'function'>
    print(add(10,30))  # 40
    ```
- `lambda`도 함수 객체를 만든다.
- 간단한 계산이나 `filter(), map()` 과 함께 자주 사용한다.

---

## 내장함수

- `enumerate()` : 인덱스와 값을 함께 가져오기
  - ```Python
    st_names = ['king','scott','son','lee']

    # 인덱스를 직접 증가시키는 방법
    index = 0 
    for name in st_names:
      print(index, name)
      index += 1

    # enumerate() 사용
    for index, name in enumerate(st_names, start=1):
      print(index, name)
    ```
- `enumerate()` 는 인덱스요소를 순서데로 제공
- `start=1` 을 주면 인덱스가 1부터 시작(기본값0)
- `for index, name in ...` 은 튜플의 값을 두 변수에 나눠 담는 방식


- `dir()` : 사용할수 있는 속성과 메서드 확인
- `filter()` : 조건에 맞는 요소만 걸러내기
  - ```Python
    def positive_fn(x):
      return x > 0

    num = [3,10,-8,0,10,-2,7]

    print(list(filter(positive_fn,nums)))       #[3,10,10,7]

    #lambda버전
    print(list(filter(lambda x : x > 0, nums))) #[3,10,10,7]
    ```
- filter(함수, 반복가능객체) 는 함수의 결과가 참인 요소만 남김
- `x > 0` 이 `True`인 양수만 추출하므로 0과 음수는 제외
- `filter()` 결과는 filter 객체이므로 목록으로 보려면 `list()` 로 변환함


- `map()` : 모든 요소에 같은 처리 적용
  - ```Python
    result = list(map(lambda x : x * 5, [1,2,3,4,5]))
    print(result)    # [5,10,15,20,25]
    ```
- map(함수, 반복가능객채) 는 모든 요소에 함수를 적용
- `map()` 도 map 객체를 반환하므로 필요한 경우 `list()` 로 변환함

#### filter(), map() 차이

- `filter()`
  - 목적 : 조건에 맞는 요소 선별
  - 함수 반환값 : 참/거짓으로 판정 가능한 값
  - 결과 요소 개수 : 줄어들 수 있음
- `map()`
  - 목적 : 각 요소를 변환
  - 함수 반환값: 변환된 값
  - 결과 요소 개수 : 보통 원본가 동일

---

## 예외처리

- 예외처리 없이 오류가 나는 경우
  - ```Python
    print('start')
    print(10 / 0)  # ZeroDivisionError 발생
    print('end')   # 실행되지 않음
    ```
- 예외처리로 프로그램 흐름 유지
  - ```Python
    print('start')
    try:
        print(10 / 0)
    except:
        print('에러발생!!!')
    print('end')
    # start
    # 에러발생!!!
    # end
    ```
- `try` : 오류가 발생할 수 있는 코드를 작성한다.
- `except` : 오류가 발생했을  때 대신 실행할 코드를 작성한다.
- 가능하면 except : 보다 구체적인 예외 종류를 지정하는 것이 좋다.


- 오류 종류에 따라 다르게 처리

  - ```Python
    # ex)
    st_names = ['king', 'scott', 'son', 'lee']

    try:
        print(10 / 0)
        print(st_names)
        st_names[10]
    except ZeroDivisionError as e:
        print('0으로 나누는 연산은 불가능합니다.')
        print(e)
    except (NameError, TypeError):
        print('이름 또는 자료형 관련 오류가 발생했습니다.')
    except IndexError:
        pass
    except:
        print('알려지지 않은 오류가 발생했습니다.')
    ```
- ZeroDivisionError : 0으로 나눌 때
- NameError : 정의되지 않은 변수 이름을 사용할 때
- TypeError : 지원되지 않는 변수 이름을 사용할 때
- IndexError : 리스트 범위를 벗어난 인덱스에 접근할 때
- KeyError : 딕셔너리에 없는 키를 조회할 때
- FileNotFoundError : 존재하지 않는 파일을 열려고 할 때

---

## 클래스와 객체

- 클래스(class) : 변수와 함수를 묶어 객체를 만들기 위한 설꼐도
- 객체(object) : 클래스를 기반으로 생성한 실제 값(인스턴스)
- 메서드(method) : 클래스 안에 정의된 함수
- 속성(attribute) : 객체에 저장된 데이터(ex: first, second)
  - ```Python
    class Cookie:
        pass

    c1 = Cookie()
    c2 = Cookie()

    print(type(c1))  # <class '__main__.Cookie'>
    print(type(c2))
    print(id(c1))
    print(id(c2))
    ```
- `Cooket()` 로 새로운 객체를 생성
- `type()` 은 객체의 자료형/클래스를 확인
- `id()` 는 객체의 식별값을 확인, c1과 c2는 서로 별개의 객체
- `pass` 는 아직 클래스 내부에 구현할 코드가 없을 때 사용할 수 있음


#### 생성자와 self, 객체 생성 및 메서드

- ```Python
  class FourCal:
      # 객체 생성 시 초기화
      def __init__(self, first: int, second: int):
          self.first = first
          self.second = second

      # 속성 값 변경
      def setData(self, first: int, second: int):
          self.first = first
          self.second = second

      def add(self):
          return self.first + self.second

      def sub(self):
          return self.first - self.second

      def mul(self):
          return self.first * self.second

      def div(self):
          return self.first / self.second


  cal1 = FourCal(10, 20)

  print(cal1.first)  # 10
  print(cal1.second) # 20
  print(cal1.add())  # 30
  print(cal1.sub())  # -10
  print(cal1.mul())  # 200
  print(cal1.div())  # 0.5

  cal1.setData(100, 30)
  print(cal1.add())  # 130
  ```



* `__init__()`: 객체를 생성할 때 자동 호출되는 생성자(초기화 메서드)
* `self`: 메서드를 호출한 현재 객체 자신을 가리킴
* `self.first`,`self.second`: 각 객체가 가진 인스턴스 변수
* `cal1 = FourCal(10,20)` : 새로운 객체 생성과 초기값 전달
* `cal1.add()` : 해당 객체의 메서드 호출
* `cal1.setData(100,30)`: 기존 객체의 `first, second`값 변경

---

## 상속

- 상속 : 기존 클래스의 속성과 메서드를 물려받아 새로운 클래스를 만드는 기능
- 부모 클래스 : ex)FourCal
- 자식 클래스 : ex)MoreFourCal
  - ```Python
    class MoreFourCal(FourCal):
        def pow(self):
            return self.first ** self.second

    cal4 = MoreFourCal(11, 5)

    print(cal4.first)  # 11
    print(cal4.second) # 5
    print(cal4.add())  # 16 : 부모 클래스에서 상속
    print(cal4.pow())  # 161051 : 자식 클래스에서 추가
    ```



* `class MoreFourCal(FourCal)`에서 괄호 안의 `FourCal`이 **부모 클래스**다.
* 자식 클래스는 부모 클래스의 `__init__`, `add` 등 메서드를 사용할 수 있다.
* `pow()`는 자식 클래스에서 새로 정의한 제곱 메서드다.

---

## 메서드 오버라이딩(Method Overriding)

- 부모 클래스에서 물려받은 같은 이름의 메서드를 자식 클래스에서 다시 정의하는 것

  - ```Python
    class MoreFourCal(FourCal):
        def pow(self):
            return self.first ** self.second

        # 부모 클래스의 div()를 재정의(오버라이딩)
        def div(self):
            if self.second == 0:
                return 0
            return self.first / self.second

        # 출력용 문자열을 만드는 특수 메서드
        def __str__(self):
            return f'{self.first}, {self.second} 값을 이용한 사칙연산과 제곱 연산을 하는 객체'

    cal5 = FourCal(10, 0)
    # cal5.div()  # ZeroDivisionError 발생

    cal6 = MoreFourCal(10, 0)
    print(cal6.pow())  # 1 (10 ** 0)
    print(cal6.div())  # 0 (오버라이딩된 메서드 사용)

    cal7 = MoreFourCal(10, 0)
    print(cal7)
    # 10, 0 값을 이용한 사칙연산과 제곱 연산을 하는 객체
    ```
- `__str__()`은 객체를 사람이 읽기 좋은 문자열로 나타낼 때 사용하는 특수 메서드(매직 메서드)
- `print(cal7)`을 호출하면 해당 객체의 `__str__()` 결과가 사용됨
- `__str__()`은 **문자열(`str`)을 반환**해야 함

---

## 핵심

- `lambda`: 간단한 익명함수
- `enumerate()`: 반복하면서 인덱스와요소를 함께 가져오기
- `dir()`: 객체의 속성,메서드 이름 조회
- `filter()`: 조건을 만족하는 요소만 선별
- `map()`: 각 요소에 함수를 적용해 값 변환
- `try/except`: 실행 중 오류를 잡아처리
- `class`: 객체를 만들기 위한 설계도
- 객체 : 클래스로 생성한 인스턴스
- `self` : 메서드를 호출한 현재 객체
- `__init__()`: 객체 생성 시초기화
- 상속 : 부모 클래스의 기능 물려받기
- 오버라이딩 : 부모 메서드를 같은 이름으로 재정의
- `__str__()`: 객체의 출력용 문자열 정의
