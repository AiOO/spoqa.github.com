---
layout: entry
title: "파이썬의 시간대(datetime.timezone)에 대해 알아보기"
author: 김두리
author-email: dudu@spoqa.com
description: 파이썬의 시간대에 대해서 알아봅니다.
publish: true
---

안녕하세요. 스포카 크리에이터 김두리입니다.

- **시간을 처리할 때 시간대는 왜 중요할까요?** 시간대가 명시되지 않은 시각은 충분한 정보를 내포하고 있지 않습니다. 저는 얼마 전, Google Calendar API를 이용하여 작업할 때 골치 아픈 일을 겪었습니다. 오늘의 일정을 불러오고 싶어서 오늘 0시~24시로 데이터를 요청했지만, 계속해서 결괏값에 다음 날의 일정도 포함되어서 반환되었습니다.  
- **왜 다음날 일정도 포함되었던 걸까요?** 아래와 같은 코드를 작성하여 Google Calendar API에 요청했습니다.

```python
today = datetime.date.today()
from_ = datetime.datetime(today.year, today.month, today.day, 0, 0, 0)
to = datetime.datetime(today.year, today.month, today.day, 23, 59, 59)
events = get_events_from_google_calendar(from_, to)
```

몇 시간 동안 머리를 싸매고 코드를 한 줄 한 줄 따져가며 고민을 했습니다. 결국, 제가 요청한 시각에 시간대가 지정되어 있지 않아 `get_events_from_google_calendar()` 함수 내부에서 `from_`과 `to`가 의도하지 않은 시간대의 시각으로 인식되어서 발생했던 문제라는 것을 알게 되었습니다.

```python
# 원래 의도했던 시간대: 대한민국 시간대(KST)에서 오늘 0시 0분 0초
KST = datetime.timezone(datetime.timedelta(hours=9))
from1 = datetime.datetime(today.year, today.month, today.day, 0, 0, 0,
                          tzinfo=KST)
# get_events_from_google_calendar()가 받아들인 시간대: UTC 시간대에서 오늘 0시 0분 0초
from2 = datetime.datetime(today.year, today.month, today.day, 0, 0, 0,
                          tzinfo=datetime.timezone.utc)
```

위 예제에서 `from2 - from1`를 하게 되면 `timedelta(hours=9)`가 계산됩니다. 우리가 원했던 것은 KST 기준 오늘 0시부터의 일정이었지만, Google Calendar API에서는 시간대를 UTC로 취급하여 KST 기준 오늘 9시부터 다음날 9시까지의 일정을 불러왔던 것입니다.

이렇듯 시간 관련 작업을 할 때 시간대에 대해 제대로 알고 있지 않으면 의도치 않게 많은 시간을 소모하게 될 수도 있습니다.

스포카는 많은 프로덕트에서 국제화 서비스를 제공하고 있습니다. 그래서 시간대와 시간을 제대로 정확하게 처리하는 것은 중요합니다. 하지만 위의 예시처럼 파이썬의 `datetime.datetime`은 날짜(`datetime.date`)와 시각(`datetime.time`)의 정보를 담고 있고, 시간대(`datetime.timezone`)의 정보는 담거나 담지 않을 수도 있으므로 헷갈리는 부분이 존재합니다.

오늘은 제가 파이썬으로 시간대 관련 처리를 하며 모았던 정보를 정리하여 공유하고자 글을 작성하게 되었습니다.


## 시간대

나라 또는 지역마다 살아가는 시각이 다르기 때문에 시간대에 따른 편차가 존재합니다. 이 차이가 피부로 잘 와닿지 않은 채 살아가더라도 캘린더 API나 국제화 서비스 준비 등등 시간과 관련된 작업을 진행하다 보면 시간대 문제에 직면하게 됩니다.

> 시간대는 영국의 그리니치 천문대(본초 자오선, 경도 0도)를 기준으로 지역에 따른 시간의 차이, 다시 말해 지구의 자전에 따른 지역 사이에 생기는 낮과 밤의 차이를 인위적으로 조정하기 위해 고안된 시간의 구분 선을 일컫는다. 시간대는 협정 세계시(UTC)를 기준으로 한 상대적인 차이로 나타낸다.

- UTC에 대한 더 자세한 내용은 [여기](https://ko.wikipedia.org/wiki/%ED%98%91%EC%A0%95_%EC%84%B8%EA%B3%84%EC%8B%9C)를 참고해주세요.  
- 시간대에 대한 더 자세한 내용은 [여기](https://en.wikipedia.org/wiki/Time_zone)를 참고해주세요.


파이썬의 `datetime.datetime.now()`는 실행 환경의 시간대에 따라서 시각을 표시합니다. 

`2019-01-01 00:00:00 +09:00`에 시간대가 `Asia/Seoul`로 설정된 제 랩탑에서 현재 시각을 가지고 오면, 아래와 같은 시각이 표시됩니다.

```pycon
>>> print(datetime.datetime.now())
2019-01-01 00:00:00.000000
```

그런데, 같은 시각에 `Asia/Taipei`로 설정된 랩탑에서는 현재 시각이 아래와 같이 표시됩니다.

```pycon
>>> print(datetime.datetime.now())
2018-12-31 23:00:00.000000
```
위의 예제처럼 시간대에 따라 시각이 다를 수 있다는 것을 알 수 있습니다. 


## 나라별 시간대 비교해보기

UTC를 기준으로 시간이 빠르면 `+시차`, 시간이 느리면 `-시차`로 표시합니다.

| 시간대 | 나라 | 코드 | 
| --- | --- | ---:|
| UTC-5 | 미국(동부) | EST |
| UTC | 영국 | GMT |
| UTC+8 | 대만 | TW |
| UTC+9 | 대한민국 | KST |
| UTC+9 | 일본 | JST |
| UTC+10 | 오스트레일리아(동부) | AEST |

- 나라별 시간대 차이에 대한 더 자세한 내용은 [여기](https://ko.wikipedia.org/wiki/%EC%8B%9C%EA%B0%84%EB%8C%80)를 참고해주세요.

시간대를 명확히 표시하지 않은 시각은 혼동을 일으킬 수 있습니다. 예를 들어서, 서울에 살고 있는 점주가 2019년 1월 1일 0시 0분에 방문한 고객을 알고 싶어 한다고 가정해봅시다. 이 데이터를 파이썬으로 표현하면 아래와 같이 적을 수 있습니다.

```python
KST = datetime.timezone(datetime.timedelta(hours=9))
korea_1_1 = datetime.datetime(2019, 1, 1, 0, 0, 0, tzinfo=KST)
```

만약, 대만에 사는 점주가 이를 요청했다면 아래와 같이 적을 수 있습니다.

```python
TW = datetime.timezone(datetime.timedelta(hours=8))
taipei_1_1 = datetime.datetime(2019, 1, 1, 0, 0, 0, tzinfo=TW)
```

위 예제에서 보이는 것 같이 대한민국과 대만에 있는 점주가 같은 시각을 요청했더라도, 시간대(KST/TW)에 따라서 별도로 처리해야 합니다.

```python
assert korea_1_1 != taipei_1_1
assert taipei_1_1 - korea_1_1 == datetime.timedelta(hours=1) # 같은 시각이지만 시간대에 따라서 시간차가 있습니다.
```

그렇기 때문에 시간대가 표시되어 있지 않은 2019년 1월 1일이라는 정보만으로는 정확한 시각을 알 수 없습니다. 

```python
naive_1_1 = datetime.datetime(2019, 1, 1, 0, 0, 0)
assert korea_1_1 != naive_1_1
assert taipei_1_1 != naive_1_1
```

이런 상황을 해결하기 위해 시각은 **어떤 한 시각**을 기준으로 하여 그 차이가 표시되어야 합니다. 그 기준으로 정한 것이 **UTC**입니다. 대한민국은 UTC를 기준으로 아홉시간 빠르기 때문에 `korea_1_1`의 시각을 UTC 시간대로 표현하면 `2018-12-31 15:00:00+00:00`입니다. 대만은 UTC를 기준으로 여덟시간 빠르기 때문에 `taipei_1_1`의 시각을 UTC 시간대로 표현하면 `2018-12-31 16:00:00+00:00`입니다. 위의 시각은 각각 대한민국(`2019-01-01 00:00:00+09:00`), 대만(`2019-01-01 00:00:00+08:00`)으로 표시할 수 있습니다. 이렇게 시간대와 같이 표시하면 혼란 없이 정상적으로 처리할 수 있습니다. 


## `datetime`

[`datetime`](https://docs.python.org/3/library/datetime.html)은 파이썬에서 기본으로 제공하는 표준 라이브러리로, 간단하거나 복잡한 방식으로 날짜와 시각을 조작하기 위한 클래스를 제공합니다.

> The datetime module supplies classes for manipulating dates and times in both simple and complex ways.

`datetime`은 시간대 포함 여부에 따라서 naive, aware 두 가지로 나눕니다.


### naive datetime / aware datetime

`datetime`의 타입을 알아봅시다. 파이썬에서 시간 관련 연산을 하다 보면 종종 아래와 같은 에러 문구를 만날 수 있습니다.

```pycon
>>> a = datetime.datetime.now()
>>> b = datetime.datetime.now(datetime.timezone.utc)
>>> a - b
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can't subtract offset-naive and offset-aware datetimes
```

- **naive** : 시간대를 포함하지 않습니다. (e.g. `datetime.datetime(2019, 2, 14, 4, 58, 4, 114979)`) naive 객체는 그 자체만으로 시간대를 찾을 수 있는 충분한 정보를 포함하지 않습니다.

- **aware**(timezone-aware) : 시간대를 포함합니다. (e.g.`datetime.datetime(2019, 2, 14, 4, 58, 4, 114979, tzinfo=<UTC>)`) aware 객체는 자신의 시각 정보를 다른 aware 객체와 상대적인 값으로 조정할 수 있도록 시간대나 [일광 절약 시간 정책](https://ko.wikipedia.org/wiki/%EC%9D%BC%EA%B4%91_%EC%A0%88%EC%95%BD_%EC%8B%9C%EA%B0%84%EC%A0%9C) 혹은 적용 가능한 알고리즘 정보를 담고 있습니다.

**naive는 어느 시간대를 기준으로 하는 시각인지 모호하므로 aware를 이용하는 것을 권장합니다.**


## 직접 확인해보기

준비한 몇 가지 코드를 보며 확인해봅시다. naive와 aware의 차이를 확인하고, 시간대 지정 방법에 대한 내용을 다룹니다.

### 개발환경

- [Python](https://www.python.org/downloads/) 3.7
- [pytz](https://pypi.org/project/pytz/2018.9/)


여기서는 `datetime`을 쉽게 다루기 위해 `pytz` 라이브러리를 사용합니다. `pytz`는 아래와 같은 장점이 있습니다.
1. 시간대를 시간차가 아닌 사람이 알아보기 쉬운 지역 이름으로 비교적 쉽게 설정할 수 있습니다.
2. 원하는 시간대의 aware로 변경해주는 `localize()` 메소드를 제공합니다.

pytz 사용에 앞서, pytz가 제공하는 시간대 식별자를 확인하시려면 다음을 따라 해주세요.
```python
import pytz

for tz in pytz.all_timezones:
    print(tz)
```
혹은 [여기](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)를 참고하셔도 좋습니다.


### naive

naive는 날짜와 시각만을 갖습니다.

```python
import datetime

datetime.datetime.utcnow()
# UTC 기준 naive : datetime.datetime(2019, 2, 14, 4, 54, 29, 281594)

datetime.datetime.now()
# 실행 환경 시간대 기준 naive : datetime.datetime(2019, 2, 14, 13, 54, 32, 939155)
```


### aware
naive와 달리 aware는 시간대 정보(tzinfo) 도 갖습니다.
```python
import datetime
from pytz import utc

utc.localize(datetime.datetime.utcnow())
# UTC 기준 aware : datetime.datetime(2019, 2, 14, 4, 55, 3, 310474, tzinfo=<UTC>)
```

`now`는 UTC를 기준으로 현재 시각을 생성합니다. 하지만, naive 시각입니다.

```python
now = datetime.datetime.utcnow()
```

이 시각은 naive 시각이므로 `pytz.timezone.localize`를 통해 timezone-aware 시각으로 변환된 시각과 동일하지 않습니다.

```python
assert now != utc.localize(now)
```

### 시간대 제대로 지정하기

시간대가 무엇이고, 명시하는 것이 왜 중요한지 알게 되셨다면 시간대를 원하는 의도에 맞게 지정하는 방법에 대해 알아봅시다.

```python
import datetime
from pytz import timezone, utc

KST = timezone('Asia/Seoul')

now = datetime.datetime.utcnow()
# UTC 기준 naive : datetime.datetime(2019, 2, 14, 4, 18, 28, 805879)

utc.localize(now)
# UTC 기준 aware : datetime.datetime(2019, 2, 14, 4, 18, 28, 805879, tzinfo=<UTC>)

KST.localize(now)
# UTC 시각, 시간대만 KST : datetime.datetime(2019, 2, 14, 4, 18, 28, 805879, tzinfo=<DstTzInfo 'Asia/Seoul' KST+9:00:00 STD>)

utc.localize(now).astimezone(KST)
# KST 기준 aware : datetime.datetime(2019, 2, 14, 13, 18, 28, 805879, tzinfo=<DstTzInfo 'Asia/Seoul' KST+9:00:00 STD>)

```

`replace()` 메소드로 날짜나 시간대를 변경할 수 있습니다.  

```python
KST = timezone('Asia/Seoul')
TW = timezone('Asia/Taipei')

date = datetime.datetime.now()
# datetime.datetime(2019, 2, 14, 13, 59, 44, 872224)

date.replace(hour=10) # hour만 변경
# datetime.datetime(2019, 2, 14, 10, 59, 44, 872224)

date.replace(tzinfo=KST) # tzinfo만 변경
# datetime.datetime(2019, 2, 14, 13, 59, 44, 872224, tzinfo=<DstTzInfo 'Asia/Seoul' LMT+8:28:00 STD>)

date.replace(tzinfo=TW) # tzinfo만 변경
# datetime.datetime(2019, 2, 14, 13, 59, 44, 872224, tzinfo=<DstTzInfo 'Asia/Taipei' LMT+8:06:00 STD>)
```

하지만 replace는 그 속성 자체만을 바꿔버리는 것이기 때문에 사용에 주의할 필요가 있습니다. 

```python
now = datetime.datetime.utcnow()

assert utc.localize(now) == now.replace(tzinfo=utc)
assert KST.localize(now) != now.replace(tzinfo=KST)
assert TW.localize(now) != now.replace(tzinfo=TW)
```
그뿐만 아니라 `replace()`를 이용할 경우 의도하지 않은 시간대로 설정될 수도 있으므로 유의해야 합니다. 그 이유는 아래와 같습니다.

- 시간대는 생각보다 자주 바뀝니다(더 자세한 내용은 [스포카의 규칙](#스포카의-규칙) 2번을 참고해주세요). 이렇게 변경되는 사항들은 tz database에 기록되는데, pytz는 이에 기반합니다. pytz의 버전이 `2018.9`와 같은 날짜로 되어있는데 `2018.9` 버전은 2018년 9월 기준 시간대 테이블을 기준으로 시간대를 만들어주는 버전입니다. 이 버전에선 `Asia/Seoul`의 시간대는 `UTC+9`입니다.
- pytz는 무슨 이유에서 인지 `datetime.replace()`나 `datetime.astimezone()`에서 호출될 때 이 tz database 타임 테이블의 맨 첫 번째(가장 오래된) 기록을 가지고 변환을 시도합니다. 서울의 경우 초기에 `UTC+8:28`이었기 때문에 이 정보를 기반으로 변환합니다.

그래서 `pytz`를 사용할 때는 `pytz.timezone.localize()`를 항상 써야 하고, `.astimezone()`같은 파이썬의 표준 메서드들을 사용하고 싶다면 `datetime.timezone`을 사용해야 합니다.


## 스포카의 규칙
스포카에서 datetime을 다룰 때 흔히 따르는 두 가지 큰 원칙이 있습니다.

### 1. naive datetime은 절대 사용하지 않습니다.
가장 큰 이유는 naive datetime과 aware datetime을 서로 섞어서 쓰지 못한다는 것입니다.

```pycon
>>> from datetime import datetime, timezone
>>> datetime.utcnow() + datetime.now(tz=timezone.utc)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for +: 'datetime.datetime' and 'datetime.datetime'
```

동적 타입 언어에서 쓸 수 있는 가장 간단한 타입 검사 수단인 `isinstance()` 체크로도 이 둘을 구별할 수가 없으므로, 코드의 어느 지점에서 naive datetime이 섞이기 시작하면 예기치 않은 지점에서 버그 발생 가능성이 급격히 올라갑니다. Python 2에서 `str`과 `unicode`를 섞으면 안 되는 것과 비슷한 이유라고 생각하시면 됩니다.

### 2. 장기적으로 보존해야 하는 datetime은 항상 UTC를 기준으로 저장합니다.
지역 시간대는 지정학적 또는 정치적인 이유로 생각보다 자주 바뀝니다. 예컨대 1961년 이전까지 한국은 UTC+08:30을 지역 시간대로 사용했었고, 1988년 올림픽 즈음에는 일광 절약 시간대를 시행하고 있었습니다. 시간대 데이터베이스(tz database)는 이런 변경 내역을 담고 있고, pytz가 제공하는 시간대 객체의 동작에도 반영되어 있습니다. 그 때문에 시간대 데이터베이스가 제때 업데이트되지 않거나, 갑작스러운 시간대 변경으로 데이터베이스에 반영이 늦어지거나 하면, 시간 계산에서 오차가 발생할 여지가 있습니다. 또한 같은 aware datetime 이어도 서로 다른 시간대를 가진 datetime끼리 연산하거나 하는 상황도 문제를 복잡하게 만들고, DB나 다른 서비스의 API를 사용할 때, 그 서비스가 시간대를 제대로 다루는 데에 필요한 복잡도를 감수하는 대신 단순히 UTC 기준의 고정 오프셋 시간대만 사용하는 등의 이유로 서로 지원 범위가 맞지 않아 곤란을 겪을 수도 있습니다.

혼선을 줄일 수 있는 좋은 규칙 중 하나는, `str`과 `unicode`를 다루던 것과 비슷하게 모든 내부적인 계산에서 UTC 기준의 aware datetime만 사용하고, 사용자에게 보여줘야 할 때만 필요한 시간대로 변환해서 보여 주는 것입니다. 

스포카에서는 메인 서버의 `dodo.datetime` 유틸리티 모듈도 이런 규칙을 따르고 있으며, 대부분의 DB 모델 객체의 DateTime 컬럼에서 `timezone=True` 옵션을 켜서 사용하고 있습니다.


## 정리

시간 관련 작업을 하신다면 아래 사항을 **꼭** 기억해주세요.
1. 시간대를 명시합시다.
2. 시각을 애플리케이션 로직이나 데이터베이스에서 저장할 때는 UTC로 사용하고, 유저에게 표시할 때만 유저의 시간대로 변환하여 보여주도록 합시다.
   - 백엔드 서버끼리 통신할 때도 항상 UTC를 사용한다는 가정을 하면, 시간대가 없더라도 [robust](https://en.wikipedia.org/wiki/Robustness_(computer_science))하게 처리할 수 있습니다.

