# Chapter 2: 파이썬스러운 코드

## 인덱스와 슬라이스

~~~
ex) 마지막 부분의 인덱스를 찾을 때 
my_numbers = [1,3,5,8]
my_numbers[-1] -> 8

ex) slice로 찾기
my_numbers = [1,3,5,8]
my_numbers[1:4] -> [3, 5, 8]

ex) 간격을 정해서 출력 
my_numbers[0:4:2] -> [0, 5]

ex) slice 함수로도 가능 -> (시작, 중지, 간격)
interval = slice(0,4,2)
my_numbers[interval] -> [0, 5]
~~~

### 자체 시퀀스 생성
- ```__getitem__```이라는 매직 메서드로 위와 같은 출력 결과가 나옴.
- 클래스가 표준 라이브러리 객체를 감싸는 래퍼인 경우 기본 객체에 가능한 많은 동작을 위임할 수 있음.
    - 리스트의 래퍼인 경우 리스트의 동일한 매서드를 호출하여 호환성을 유지할 수 있음. 

~~~
EX) -> 상속 방식으로 활용할 있음
class Items:
    def __init__(self, *values):
        self._values = list(values)
    
    def __len__(self):
        return len(self._values)
    
    def __getitem__(self, item):
        return self._values.__getitem__(item)
~~~

- 자체 시퀀스를 만들 때 고려해야되는 요소
    - 범위로 인덱싱하는 결과는 해당 클래스와 같은 타입의 인스턴스여야 한다. 
    - slice에 의해 제공된 범위는 파이썬이 하는 것처럼 마지막 요소는 제외해야 한다.

## 컨텍스트 관리자
- 주요 동작의 전후에 작업을 실행하려고 할 때 유용함

~~~
EX)
fd = open(filename)
try:
    process_file(fd)
finally:
    fd.close()

개선된 방식 -> close 없이도 자동으로 닫혀짐. 
with open(filename) as fd:
    process_file(fd)
~~~
- 컨텍스트 관리자는 ```__enter__```와 ```__exit__```로 구성되어 있음.

### 컨텍스트 관리자 구현
- contextlib 모듈을 쓰면 간결하게 컨텍스트 관리자를 구현할 수 있음
- contextmanager 데코레이터를 적용하면 해당 함수의 코드를 컨텍스트 관리자로 변환시킬 수 있음
- 함수는 제너레이터 형태여야 함. 

~~~
EX)
import contextlib

@contextlib.contextmanager
def db_handler():
    stop_database()
    yield
    start_database()

with db_handler():
    db_backup()
~~~

- 제너레이터 함수에 데코레이터를 적용하면 yield 앞은 ```__enter__```의 일부가 된다.
- yield 이후의 모든 코드는 ```__exit__```의 일부가 된다.
- contextlib을 쓰면 리팩토링할 때 쉬워진다.
- ContextDecorator는 클래스에 적용할 때 쓸 수 있음
~~~
class dbhandler_decorator(contextlib.ContextDecorator):
    def __enter__(self):
        stop_database()
    
    def __exit__(self, ext_type, ex_value, ex_traceback):
        start_database()
    
@dbhandler_decorator
def offline_backup():
    run("pg_dump database")
~~~
- with 없이 사용할 수 있음
- 완전히 독립적으로 사용해야되는 것이 단점이다. 데코레이터에서는 좋지만 컨텍스트 관리자에서는 좋지 않다. 
- contextlib.suppress는 컨텍스트 관리자에서 util 패키지로 제공한 예외 중 하나가 발생한 경우에는 실패하지 않도록 한다. 

~~~
import contextlib

with contextlib.suppress(DataConversionException):
    parse_data(input_json)
~~~

## 프로퍼티, 속성과 객체 메서드의 다른 타입들
- 보통 파이썬에서는 _으로 시작하면 외부에서 실행되지 않기를 기대하는 함수라고 생각할 수 있음
- 이중 밑줄을 사용하는 케이스도 있지만 되도록이면 쓰지 않는 것이 좋다. 

### 프로퍼티
- 객체의 상태나 다른 속성의 값을 기반으로 어떤 계산을 해야할 때 사용한다. 
- 프로퍼티는 객체의 어떤 속성에 대한 접근을 제어하려할 때 사용.
- 자바에서는 접근메서드와 비슷하다.

~~~
import re

EMAIL_FORMAT = re.compile(r"[^@]+@[^@]+[^@]+")

def is_valid_email(potentially_valid_email: str):
    return re.match(EMAIL_FORMAT, potentially_valid_email) is not None

class User:
    def __init__(self, username):
        self.username = username
        self._email = None
    
    @property 
    def email(self):
        return self._email
    
    @email.setter
    def email(self, new_email):
        if not is_valid_email(new_email):
            raise ValueError(f"유효한 이메일 아님")
        self._email = new_email
~~~

- 프로퍼티를 사용하면 @property로 decorate된 함수에서만 특정 attribute를 추출할 수 있다.
- 함수명.setter를 통해 attribute의 이름을 수정할 수 있다. 
- 프로퍼티는 CC08(명령 쿼리 분리 원칙)을 따르기 위한 좋은 방법임. 

## 이터러블 객체
- 반복을 위해 정의한 로직을 사용해서 이터러블 생성 가능
- 이터러블은 ```__iter__```로 만들어진 객체이고, 이터레이터는 ```__next__```로 만들어진 객체이다.

~~~
from datetime import timedelta

class DateRangeIterable:
    def __init__(self, start, end):
        self.start_date = start
        self.end_date = end
        self._present_day = start
    
    def __iter__(self):
        return self

    def __next__(self):
        if self._present_day >= self.end_date:
            raise StopIteration
        today = self._present_day
        self._present_day += timedelta(days=1)
        return today
~~~
- for문이나 next() 함수를 통해 다음 단계를 호출할 수 있음
- 이 이터러블 객체는 한번 끝나면 다시는 이터러블하지 않게 된다. (반복 프로토콜이 발생하기 때문)

~~~
class DateRangeIterable:
    def __init__(self, start, end):
        self.start_date = start
        self.end_date = end
    
    def __iter__(self):
        current_day = self.start_date
        while current_day < self.end_date:
            yield current_day
            current_day += time_delta(days=1)
~~~
- ```__iter___``` 함수에 제너레이터를 넣어서 다시는 이터러블하지 않은 문제를 해결할 수 있음. (컨테이너 이터러블)


### 시퀀스 만들기
- ```__iter__``` 없이 ```__getitem__```으로 이터러블하게 만들 수 있음.
- ```__len__```과 ```__getitem__```으로 구현할 수 있음.
- 메모리를 적게 사용할 수 있는 장점을 가지고 있음. 
- 시퀀스로 구현하게 된다면 n번째 요소를 찾을 때 O(n) 대신에 O(1)로 찾을 수 있다. 

~~~
class DateRangeSequence:
    def __init__(self, start, end):
        self.start_date = start
        self.end_date = end
        self._range = self._create_range()

    def _create_range(self):
        days = []
        current_day = self.start_date
        while current_day < self.end_date:
            days.append(current_day)
            current_day += timedelta(days=1)

    def __getitem__(self, day_no):
        return self._range[day_no]

    def __len__(self):
        return len(self._range)
~~~

## 컨테이너 객체

- ```__contains__``` 객체로 구현된다. -> 가독성이 매우 높아짐. 
- 파이썬의 in 키워드와 같다고 생각하면 된다. 

~~~
EX)
element in container -> container.__contains__(element)
~~~
~~~
EX)
class Boundaries:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def __contains__(self, coord):
        x,y = coord
        return 0 <= x < self.width and 0 <= y < self.height

class Grid:
    def __init__(self, width, height):
        self.width = width
        self.height = height
        self.limits = Boundaries(width, height)

    def __contains__(self, coord):
        return coord in self.limits
~~~
