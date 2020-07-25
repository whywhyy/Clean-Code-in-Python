# Chapter 2 후기

문구를 정리하여 작성하였습니다.

# Chapter 2 파이썬스러운(pyythonic) 코드

## 인덱스와 슬라이스

```py
# 음수로 맨뒤를 기준으로 접근이 가능하다.
def index_last():
    """
    >>> my_numbers = (4, 5, 3, 9)
    >>> my_numbers[-1]
    9
    >>> my_numbers[-3]
    5
    """


 # [index start: index end(포함 안됨)]  
 # [index start: index end(포함 안됨) : 요소를 뛰어넘수 수]  
 # slice 객체를 만들어서 사용 가능 (1,3) == [1:3]

def get_slices():
    """
    >>> my_numbers = (1, 1, 2, 3, 5, 8, 13, 21)
    >>> my_numbers[2:5]
    (2, 3, 5)
    >>> my_numbers[:3] # 3 전까지!
    (1, 1, 2)
    >>> my_numbers[3:] # 3 부터
    (3, 5, 8, 13, 21)
    >>> my_numbers[::]
    (1, 1, 2, 3, 5, 8, 13, 21)
    >>> my_numbers[1:7:2]
    (1, 3, 8)


    >>> interval = slice(1, 7, 2)
    >>> my_numbers[interval]
    (1, 3, 8)

    >>> interval = slice(None, 3)
    >>> my_numbers[interval] == my_numbers[:3]
    True
    """
```

### 자체 시퀀스 생성

리스트, 튜플이 동작하는 원리의 코드 

```py
# 즉 __getitem__ 메소드로 리스트,튜플 시퀀스가 동작함.
class Items:
    def __init__(self, *values):
        self._values = list(values)

    def __len__(self):
        return len(self._values)

    def __getitem__(self, item):
        return self._values.__getitem__(item)

```

### 자신만의 시퀀스 구현시 주의사항

 - 범위로 인덱싱하는 결과는 해당 클래스와 같은 타입의 인스턴스여야 함.
 - slice에 의해 제공된 범위는 파이썬이 하는것처럼 마지막 요소는 제외하여야함.

```py
"""
# range 범위 인덱싱을 하는경우 같은타입의 인스턴스를 반환함.
 >>> range(1,100)[25:50]
    range(26:51)
"""
```

## 컨텍스트 관리자(context manager)

### with 로 컨텍스트를 관리하자.

 #### 방법 1. 클래스로 동작 (withn 문 사용)

 - with 문 시작시 해당 클래스의 \_\_enter\_\_ 동작
 
 - with 블락 종료시 \_\_exit\_\_ 동작함. 심지어 블락 중간에 오류가 나도 \_\_exit\_\_ 호출함.

 > 주의사항 
 >
 > 1. \_\_exit\_\_ 에서 True 를 반환하면 예외가 호출되지 않을 수 있음.  return True 를 쓰려면 충분히 그러한 이유가 타당한지 확인해야 한다.
 >
 > 2. \_\_enter\_\_ 의 반환값은 self. 객체 자체를 반환함.
 >


 #### 방법 2. contextlib 사용 (with 문 사용)

 - @contextlib.contextmanager 적용하여 메소드 작성

 - 메소드 내에서

 - (enter)method-yield-(exit)method  방식으로 작성

 - yield 뒤에 나오는 모든것들은 \_\_exit\_\_ 의 로직임.

 #### 방법 3. with 문사용하지 않고 데코레이터로 with 문 구현

- contextlib.ContextDecorator 를 class 에 상속하여

- 데코레이터로 With문을 대신하여 사용할수 있다!!

 > 단점 : \_\_enter\_\_ 의 반환으로 값을 다루는 객체를 다루고 싶은 경우, 이 방법 말고 다른방법을 써야함. 
 >
 > ex) with open(file) as fb: 에서 fb 를 사용해야하는 경우

```py
import contextlib


run = print


def stop_database():
    run("systemctl stop postgresql.service")


def start_database():
    run("systemctl start postgresql.service")

# 방법 1.
class DBHandler:
    def __enter__(self):
        stop_database()
        return self

    def __exit__(self, exc_type, ex_value, ex_traceback):
        start_database()

def db_backup():
    run("pg_dump database")

# 방법 2.
@contextlib.contextmanager
def db_handler():
    stop_database()
    yield
    start_database()

# 방법 3.
class dbhandler_decorator(contextlib.ContextDecorator):
    def __enter__(self):
        stop_database()

    def __exit__(self, ext_type, ex_value, ex_traceback):
        start_database()

@dbhandler_decorator()
def offline_backup():
    run("pg_dump database")

def main():
    with DBHandler():
        db_backup()

    with db_handler():
        db_backup()

    offline_backup()

if __name__ == "__main__":
    main()

```

## 프로퍼티, 속성과 객체 메서드의 다른 타입들

### private

파이썬은 public, private, protected 를 가지지 않고, 객체의 모든 프로퍼티와 함수는 public 임.

다만 \_\_를 붙여 클래스에 변수를 생성하는 경우, name mangling이 발생함.

\_\<class_name>\_\_\<attribute-name> 과 같은 식으로 변수명이 변경됨.

보통은 하나의 \_ 로 변수를 작성하고, 특별한 경우가 아니면 두개의 \_\_ 를 사용하지 않음. (오버라이드 충돌을 방지하기 위해 만들어짐.)

속성을 private 으로 정의하려는 경우 하나의 밑줄을 사용하도록 하자.

### 프로퍼티

 getter, setter 대신 @property 쓰자.

 ```py
 
import re

EMAIL_FORMAT = re.compile(r"[^@]+@[^@]+\.[^@]+")


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
            raise ValueError(
                f"Can't set {new_email} as it's not a valid email"
            )
        self._email = new_email

 ```

 ## 이터러블 객체

  for e in myobject : 동작시키기 위해서는 

- 객체가 \_\_next\_\_ 나 \_\_iter\_\_ 이터레어터 메서드 중 하나를 포함했는지 여부
- 객체가 시퀀스이고 \_\_len\_\_ 과 \_\_getitem\_\_를 모두 가졌는지 여부

를 확인한다.


객체를 for in 문으로 호출하자고 하자. 

```py
# 방법 1
for day in DateRangeIterable(date(2019, 1, 1), date(2019,1,5)):
    print(day)
# 방법 2
for day in DateRangeContainerIterable(date(2019, 1, 1), date(2019,1,5)):
    print(day)
# 방법 3
for day in DateRangeSequence(date(2019, 1, 1), date(2019,1,5)):
    print(day)
```

이터러블한 객체를 생성하기위해 3가지 방법을 소개한다.

### 방법 1.
  - for문 iter 객체 이터러블 반환 받음
  - iter 객체가 self 를 반환
  - 각 루프 단계에서 next() 를 호출
  - next() 는 \_\_next\_\_ 메서드에게 위임. 
  - next() 는 raise StopIteration 가 발생할때까지 계속 종료시 next()실행

#### 방법 1 문제점
  - 한번 실행은 잘되나 끝에 도달한 후, 호출하면 StopIteration 발생하여 동작하지 않음.

  - yield 를 사용하는 방법 2를 보자.
  
### 방법 2. (컨테이너 이터러블 객체)
 - yield 를 활용하여 동작
 - \_\_iter\_\_ 에 yield(제너레이터) 를 동작시켜 \_\_iter\_\_ 내부에서 동작을 마치는 방법이다.

### 방법 2 문제점
 - n 번째 요소를 가져오려면 N번 반복해야한다. 
 - 인메모리와 CPU 사이의 트레이드 오프다. 메모리는 적게사용하나, 복잡도가 n 이다.

### 방법 3.(시퀀스)

- \_\_iter\_\_() 메서드가 정의되어 있지 않으면, \_\_getitem\_\_ 을 찾고 없으면 TypeError 을 발생시킴

-  클래스 선언시 객체에 대한 데이터를 생성해 두고, for 문에서 \_\_iter\_\_ 가 없어서, \_\_getitem\_\_ 호출하며, 호출할때 O(1) 으로 응답한다.


```py
from datetime import timedelta

# 방법 1
class DateRangeIterable:
    """An iterable that contains its own iterator object."""

    def __init__(self, start_date, end_date):
        self.start_date = start_date
        self.end_date = end_date
        self._present_day = start_date

    def __iter__(self):
        return self

    def __next__(self):
        if self._present_day >= self.end_date:
            raise StopIteration
        today = self._present_day
        self._present_day += timedelta(days=1)
        return today

# 방법 2
class DateRangeContainerIterable:
    """An range that builds its iteration through a generator."""

    def __init__(self, start_date, end_date):
        self.start_date = start_date
        self.end_date = end_date

    def __iter__(self):
        current_day = self.start_date
        while current_day < self.end_date:
            yield current_day
            current_day += timedelta(days=1)

# 방법 3
class DateRangeSequence:
    """An range created by wrapping a sequence."""

    def __init__(self, start_date, end_date):
        self.start_date = start_date
        self.end_date = end_date
        self._range = self._create_range()

    def _create_range(self):
        days = []
        current_day = self.start_date
        while current_day < self.end_date:
            days.append(current_day)
            current_day += timedelta(days=1)
        return days

    def __getitem__(self, day_no):
        return self._range[day_no]

    def __len__(self):
        return len(self._range)

```

