# Chapter 2: 파이썬스러운 코드

## 인덱스와 슬라이스
```
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
```

### 자체 시퀀스 생성
- ```__getitem__```이라는 매직 메서드로 위와 같은 출력 결과가 나옴.
- 클래스가 표준 라이브러리 객체를 감싸는 래퍼인 경우 기본 객체에 가능한 많은 동작을 위임할 수 있음.
    - 리스트의 래퍼인 경우 리스트의 동일한 매서드를 호출하여 호환성을 유지할 수 있음. 

```
EX) -> 상속 방식으로 활용할 있음
class Items:
    def __init__(self, *values):
        self._values = list(values)
    
    def __len__(self):
        return len(self._values)
    
    def __getitem__(self, item):
        return self._values.__getitem__(item)
```

- 자체 시퀀스를 만들 때 고려해야되는 요소
    - 범위로 인덱싱하는 결과는 해당 클래스와 같은 타입의 인스턴스여야 한다. 
    - slice에 의해 제공된 범위는 파이썬이 하는 것처럼 마지막 요소는 제외해야 한다.

## 컨텍스트 관리자
- 주요 동작의 전후에 작업을 실행하려고 할 때 유용함

```
EX)
fd = open(filename)
try:
    process_file(fd)
finally:
    fd.close()

개선된 방식 -> close 없이도 자동으로 닫혀짐. 
with open(filename) as fd:
    process_file(fd)
```
- 컨텍스트 관리자는 ```__enter__```와 ```__exit__```로 구성되어 있음.

### 컨텍스트 관리자 구현
- contextlib 모듈을 쓰면 간결하게 컨텍스트 관리자를 구현할 수 있음
- contextmanager 데코레이터를 적용하면 해당 함수의 코드를 컨텍스트 관리자로 변환시킬 수 있음
- 함수는 제너레이터 형태여야 함. 

```
EX)
import contextlib

@contextlib.contextmanager
def db_handler():
    stop_database()
    yield
    start_database()

with db_handler():
    db_backup()
```
- 제너레이터 함수에 데코레이터를 적용하면 yield 앞은 ```__enter__```의 일부가 된다.
- yield 이후의 모든 코드는 ```__exit__```의 일부가 된다.
- contextlib을 쓰면 리팩토링할 때 쉬워진다.
- ContextDecorator는 클래스에 적용할 때 쓸 수 있음
```
class dbhandler_decorator(contextlib.ContextDecorator):
    def __enter__(self):
        stop_database()
    
    def __exit__(self, ext_type, ex_value, ex_traceback):
        start_database()
    
@dbhandler_decorator
def offline_backup():
    run("pg_dump database")
```
- with 없이 사용할 수 있음
- 완전히 독립적으로 사용해야되는 것이 단점이다. 데코레이터에서는 좋지만 컨텍스트 관리자에서는 좋지 않다. 
- contextlib.suppress는 컨텍스트 관리자에서 util 패키지로 제공한 예외 중 하나가 발생한 경우에는 실패하지 않도록 한다. 

```
import contextlib

with contextlib.suppress(DataConversionException):
    parse_data(input_json)
```

## 프로퍼티, 속성과 객체 메서드의 다른 타입들
- 보통 파이썬에서는 _으로 시작하면 외부에서 실행되지 않기를 기대하는 함수라고 생각할 수 있음
- 이중 밑줄을 사용하는 케이스도 있지만 되도록이면 쓰지 않는 것이 좋다. 

### 프로퍼티
- 객체의 상태나 다른 속성의 값을 기반으로 어떤 계산을 해야할 때 사용한다. 
- 프로퍼티는 객체의 어떤 속성에 대한 접근을 제어하려할 때 사용.
- 자바에서는 접근메서드와 비슷하다.

