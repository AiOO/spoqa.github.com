---
layout: entry
title: "파이썬의 시간대(datetime.timezone)에 대해 알아보기"
author: 김두리
author-email: dudu@spoqa.com
description: 파이썬의 시간대에 대해서 알아봅니다.
publish: true
---

안녕하세요. 스포카 크리에이터 김두리입니다.

- **시간을 처리할 때 시간대는 왜 중요할까요?** 시간대가 명시되지 않은 시각은 충분한 정보를 내포하고 있지 않습니다. 저는 얼마 전, Google calendar API를 이용하여 작업할 때 골치 아픈 일을 겪었습니다. 오늘의 일정을 불러오고 싶어서 오늘 0시~24시로 데이터를 요청했지만, 계속해서 결괏값에 다음 날의 일정도 포함되어서 반환되었습니다.  
- **왜 다음날 일정도 포함되었던 걸까요?** 아래와 같은 코드를 작성하여 Google calendar API에 요청했습니다.

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

위 예제에서 `from2 - from1`를 하게 되면 `timedelta(hours=9)`가 계산됩니다. 우리가 의도했던 것은 오늘 0시부터의 데이터였지만, Google calendar API에서는 시간대를 UTC로 취급하여 오늘 9시부터 다음날 9시까지의 일정이 불러와 졌던 것입니다.

이렇듯 시간 관련 작업을 할 때 시간대에 대해 제대로 알고 있지 않으면 의도치 않게 많은 시간을 소모하게 될 수도 있습니다.

스포카는 많은 프로덕트에서 국제화 서비스를 제공하고 있습니다. 그래서 시간대와 시간을 제대로 정확하게 처리하는 것은 중요합니다. 하지만 위의 예시처럼 파이썬의 `datetime.datetime`은 날짜(`datetime.date`), 시각(`time`) 그리고 시간대(`datetime.timezone`)의 정보를 담거나 담지 않을 수도 있으므로 헷갈리는 부분이 존재합니다.

오늘은 제가 파이썬으로 시간대 관련 처리를 하며 모았던 정보를 정리하여 공유하고자 글을 작성하게 되었습니다.


## 시간대

나라 또는 지역마다 살아가는 시각이 다르기 때문에 시간대에 따른 편차가 존재합니다. 이 차이가 피부로 잘 와닿지 않은 채 살아가더라도 캘린더 API나 국제화 서비스 준비 등등 시간과 관련된 작업을 진행하다 보면 시간대 문제에 직면하게 됩니다.

> 시간대는 영국의 그리니치 천문대(본초 자오선, 경도 0도)를 기준으로 지역에 따른 시간의 차이, 다시 말해 지구의 자전에 따른 지역 사이에 생기는 낮과 밤의 차이를 인위적으로 조정하기 위해 고안된 시간의 구분 선을 일컫는다. 시간대는 협정 세계시(UTC)를 기준으로 한 상대적인 차이로 나타낸다.

- UTC에 대한 더 자세한 내용은 [여기](https://ko.wikipedia.org/wiki/%ED%98%91%EC%A0%95_%EC%84%B8%EA%B3%84%EC%8B%9C)를 참고해주세요.  
- 시간대에 대한 더 자세한 내용은 [여기](https://en.wikipedia.org/wiki/Time_zone)를 참고해주세요.


파이썬의 `datetime.datetime.now()`는 실행 환경의 시간대에 따라서 시각을 표시합니다. 

`2019-01-01 00:00:00 +09:00`에 시간대가 아시아/서울로 설정된 제 랩탑에서 현재 시각을 가지고 오면, 아래와 같은 시각이 표시됩니다.

```python
>>> print(datetime.datetime.now())
2019-01-01 00:00:00.000000
```

그런데, 같은 시각에 아시아/타이베이로 설정된 랩탑에서는 현재 시각이 아래와 같이 표시됩니다.

```python
>>> print(datetime.datetime.now())
2018-12-31 23:00:00.000000
```
위를 참고하여, 시간대에 따라 시각이 다를 수 있다는 것을 알 수 있습니다. 


## 나라별 시간대 비교해보기

UTC를 기준으로 시간이 빠르면 `+시차`, 시간이 느리면 `-시차`로 표시합니다.

| 시간대 | 나라 | 명칭 | 
| --- | --- | ---:|
| UTC-5 | 미국(동부) | EST |
| UTC | 영국 | GMT |
| UTC+9 | 대한민국 | KST |
| UTC+9 | 일본 | JST |
| UTC+10 | 오스트레일리아(동부) | AEST |

- 나라별 시간대 차이에 대한 더 자세한 내용은 [여기](https://ko.wikipedia.org/wiki/%EC%8B%9C%EA%B0%84%EB%8C%80)를 참고해주세요.

시간대를 명확히 표시하지 않은 시각은 혼동을 일으킬 수 있습니다. 예를 들어서, 서울에 살고 있는 점주가 2019년 1월 1일 0시 0분에 방문한 고객을 알고 싶어 한다고 가정해봅시다. 이 데이터를 파이썬으로 표현하면 아래와 같이 적을 수 있습니다.

```python
KST = datetime.timezone(datetime.timedelta(hours=9))
korea_1_1 = datetime.datetime(2019, 1, 1, 0, 0, 0, tzinfo=KST)
```

만약, 영국에 사는 점주가 이를 요청했다면 아래와 같이 GMT와 동일한 시간대로 표현됩니다.

```python
uk_1_1 = datetime.datetime(2019, 1, 1, 0, 0, 0, tzinfo=datetime.timezone.utc)
```

위 예제에서 보이는 것 같이 영국과 대한민국에 있는 점주가 같은 시각을 요청했더라도, 시간대(KST/GMT)에 따라서 별도로 처리해야 합니다.

```python
assert korea_1_1 != uk_1_1
assert uk_1_1 - korea_1_1 == datetime.timedelta(hours=9) # 같은 시각이지만 시간대에 따라서 시간차가 있습니다.
```

그렇기 때문에 시간대가 표시되어 있지 않은 2019년 1월 1일이라는 정보만으로는 정확한 시각을 알 수 없습니다. 이런 상황을 해결하기 위해 시각은 **어떤 한 시각**을 기준으로 하여 그 차이가 표시되어야 합니다.

```python
naive_1_1 = datetime.datetime(2019, 1, 1, 0, 0, 0)
assert korea_1_1 != naive_1_1
assert uk_1_1 != naive_1_1
```

그 기준으로 정한 것이 **UTC**이고, UTC는 GMT에 기반하기 때문에 영국 시간대와 UTC는 동일한 시간대입니다. 대한민국은 영국보다 시간대가 아홉시간 빠르기 때문에 `korea_1_1`의 시각을 GMT 시간대로 표현하면 `2018-12-31 15:00:00+00:00`입니다. 위의 시각은 각각 영국(`2019-01-01 00:00:00+00:00`), 대한민국(`2019-01-01 00:00:00+09:00`)로 표시할 수 있습니다. 이렇게 시간대와 같이 표시하면 혼란 없이 정상적으로 처리할 수 있습니다. 


## `datetime`

[`datetime`](https://docs.python.org/3/library/datetime.html)은 파이썬에서 기본으로 제공하는 표준 라이브러리로, 간단하거나 복잡한 방식으로 날짜와 시각을 조작하기 위한 클래스를 제공합니다.

> The datetime module supplies classes for manipulating dates and times in both simple and complex ways.

`datetime`은 시간대 포함 여부에 따라서 offset-naive, offset-aware 두 가지로 나눕니다.


### offset-naive datetime / offset-aware datetime

`datetime`의 타입을 알아봅시다. 파이썬에서 시간 관련 연산을 하다보면 종종 아래와 같은 에러 문구를 만날 수 있습니다.

```python
>>> a = datetime.datetime.now()
>>> b = datetime.datetime.now(datetime.timezone.utc)
>>> a - b
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can't subtract offset-naive and offset-aware datetimes
```

- **offset-naive** : 시간대를 포함하지 않습니다. (e.g. `datetime.datetime(2019, 2, 11, 4, 58, 4, 114979)`) offset-naive 객체는 그 자체만으로 시간대를 찾을 수 있는 충분한 정보를 포함하지 않습니다. 하지만 이런 정보를 포함하지 않아 비교적 이해하고 작업하기 쉽습니다.

- **offset-aware**(timezone-offset-aware) : 시간대를 포함합니다. (e.g.`datetime.datetime(2019, 2, 11, 4, 58, 4, 114979, tzinfo=<UTC>)`) offset-aware 객체는 자신의 시각 정보를 다른 offset-aware 객체와 상대적인 값으로 조정할 수 있도록 시간대나 [일광 절약 시간 정책](https://ko.wikipedia.org/wiki/%EC%9D%BC%EA%B4%91_%EC%A0%88%EC%95%BD_%EC%8B%9C%EA%B0%84%EC%A0%9C) 혹은 적용 가능한 알고리즘 정보를 담고 있습니다.

**offset-naive는 어느 시간대를 기준으로 하는 시각인지 모호하므로 offset-aware를 이용하는 것을 권장합니다.**


## 직접 확인해보기

준비한 몇 가지 코드를 보며 확인해봅시다. offset-naive와 offset-aware의 차이를 확인하고, 시간대 지정 방법에 대한 내용을 다룹니다.

### 개발환경

- [Python](https://www.python.org/downloads/) 3.7
- [pytz](https://pypi.org/project/pytz/2018.9/)


여기서는 `datetime`을 쉽게 다루기 위해 `pytz` 라이브러리를 사용합니다. `pytz`는 아래와 같은 장점이 있습니다.
1. 시간대를 시간차가 아닌 사람이 알아보기 쉬운 지역 이름으로 비교적 쉽게 설정할 수 있습니다.
2. 원하는 시간대의 offset-aware로 변경해주는 `localize()` 함수를 제공합니다.

pytz 사용에 앞서, 원하시는 시간대를 확인하시려면 다음을 따라 해주세요.
```python
import pytz

for tz in pytz.all_timezones:
    print(tz)
```


### offset-naive

offset-naive는 날짜와 시각만을 갖습니다.

```python
import datetime

datetime.datetime.utcnow()
# UTC 기준 offset-naive : datetime.datetime(2019, 2, 11, 4, 54, 29, 281594)

datetime.datetime.now()
# local 기준 offset-naive : datetime.datetime(2019, 2, 11, 13, 54, 32, 939155)
```


### offset-aware
offset-naive와 달리 offset-aware는 시간대 정보(tzinfo) 도 갖습니다.
```python
import datetime
from pytz import utc

utc.localize(datetime.datetime.utcnow())
# UTC 기준 offset-aware : datetime.datetime(2019, 2, 11, 4, 55, 3, 310474, tzinfo=<UTC>)
```


### 시간대 제대로 지정하기

시간대가 무엇이고, 명시하는 것이 왜 중요한지 알게 되셨다면 시간대를 원하는 의도에 맞게 지정하는 방법에 대해 알아봅시다.

```python
import datetime
from pytz import timezone, utc

KST = timezone('Asia/Seoul')

now = datetime.datetime.utcnow()
# UTC 기준 offset-naive : datetime.datetime(2019, 2, 11, 4, 18, 28, 805879)

utc.localize(now)
# UTC 기준 offset-aware : datetime.datetime(2019, 2, 11, 4, 18, 28, 805879, tzinfo=<UTC>)

KST.localize(now)
# UTC 시각, 시간대만 KST : datetime.datetime(2019, 2, 11, 4, 18, 28, 805879, tzinfo=<DstTzInfo 'Asia/Seoul' KST+9:00:00 STD>)

utc.localize(now).astimezone(KST)
# KST 기준 offset-aware : datetime.datetime(2019, 2, 11, 13, 18, 28, 805879, tzinfo=<DstTzInfo 'Asia/Seoul' KST+9:00:00 STD>)

```

replace 메소드로 날짜나 시간대를 변경할 수 있습니다.  
하지만 replace는 그 속성 자체만을 바꿔버리는 것이기 때문에 사용에 주의할 필요가 있습니다.
```python
date = datetime.datetime.now()
# datetime.datetime(2019, 2, 11, 13, 59, 44, 872224)

date.replace(hour=10)
# datetime.datetime(2019, 2, 11, 10, 59, 44, 872224)

date.replace(tzinfo=KST)
# datetime.datetime(2019, 2, 11, 13, 59, 44, 872224, tzinfo=<DstTzInfo 'Asia/Seoul' LMT+8:28:00 STD>)

```

`now`는 UTC를 기준으로 현재 시각을 생성합니다. 하지만, offset-naive 시각입니다.

```
now = datetime.datetime.utcnow()
```

이 시각은 offset-naive 시각이므로 `pytz.timezone.localize`를 통해 timezone-offset-aware 시각으로 변환된 시각과 동일하지 않습니다.

```
assert now == utc.localize(now) # False
```


## 정리

시간 관련 작업을 하신다면 아래 사항을 **꼭** 기억해주세요.
1. 시간대를 명시합시다.
2. 시각을 애플리케이션 로직이나 데이터베이스에서 저장할 때는 UTC로 사용하고, 유저에게 표시할 때만 유저의 시간대로 변환하여 보여주도록 합시다.
   - 백엔드 서버끼리 통신할 때도 항상 UTC를 사용한다는 가정을 하면, 시간대가 없더라도 [robust](https://en.wikipedia.org/wiki/Robustness_(computer_science))하게 처리할 수 있습니다.
