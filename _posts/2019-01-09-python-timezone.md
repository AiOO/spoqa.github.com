---
layout: entry
title: "파이썬의 타임존에 대해 알아보기"
author: 김두리
author-email: dudu@spoqa.com
description: 파이썬의 타임존에 대해서 알아봅니다.
publish: true
---

안녕하세요. 스포카 크리에이터 김두리입니다.

파이썬의 시간 처리는 다른 언어에 비해 번거롭고 헷갈려 많은 이들이 애를 먹는 문제 중 하나이면서도 개발을 하다 보면 한 번씩은 겪게 될 중요한 문제라고 생각합니다. 타임존 처리에 대한 확실한 이해가 없으면 이와 관련하여 개발할 때 많은 시간을 소비하게 되곤 합니다.

스포카는 많은 프로덕트 측면에서 국제화 서비스를 제공하고 있습니다. 그래서 이 전에 타임존에 대해 무지했던 저는, 관련 작업을 진행할 때 헤매곤 했습니다.

시간 처리에 관련해서 겪은 이야기를 덧붙이자면, Google calendar API를 이용하여 작업할 때 타임존 처리 관련하여 골치 아픈 일을 겪었던 적도 있습니다. 분명 한국 시각을 기준으로 요청했지만, 데이터가 원하는 대로 불러와 지지 않아서 애를 먹었습니다. 몇 시간의 고생 끝에 결국, 타임존을 지정하지 않아 UTC 시각으로 인식되어서 발생했던 문제라는 것을 알게 되었습니다.
 
오늘은 제가 파이썬으로 타임존 관련 처리를 하며 모았던 정보를 정리하여 공유하고자 글을 작성하게 되었습니다.


## 타임존
나라 또는 지역마다 살아가는 시각이 다르기 때문에 시간대에 따른 편차가 존재합니다. 이 차이가 피부로 잘 와닿지 않은 채 살아가더라도 캘린더 API나 국제화 서비스 준비 등등 시간과 관련된 작업을 진행하다 보면 시간대, 즉 **타임존** 문제에 직면하게 됩니다.
> 시간대는 영국의 그리니치 천문대(본초 자오선, 경도 0도)를 기준으로 지역에 따른 시간의 차이, 다시 말해 지구의 자전에 따른 지역 사이에 생기는 낮과 밤의 차이를 인위적으로 조정하기 위해 고안된 시간의 구분 선을 일컫는다. 시간대는 협정 세계시(UTC)를 기준으로 한 상대적인 차이로 나타낸다.

UTC에 대한 더 자세한 내용은 [여기](https://ko.wikipedia.org/wiki/%ED%98%91%EC%A0%95_%EC%84%B8%EA%B3%84%EC%8B%9C)를 참고해주세요.  
타임존에 대한 더 자세한 내용은 [여기](https://en.wikipedia.org/wiki/Time_zone)를 참고해주세요.

## 왜 중요할까?
**나라별 타임존 비교해보기**  
UTC를 기준으로 시간이 빠르면 `+시차`, 시간이 느리면 `-시차`로 표시합니다.

| 시간대 | 나라 | 명칭 | 
| --- | --- | ---:|
| UTC-5 | 미국(동부) | EST |
| UTC | 영국 | GMT |
| UTC+9 | 한국 | KST |
| UTC+9 | 일본 | JST |
| UTC+10 | 오스트레일리아(동부) | AEST |

나라별 타임존 차이에 대한 더 자세한 내용은 [여기](https://ko.wikipedia.org/wiki/%EC%8B%9C%EA%B0%84%EB%8C%80)를 참고해주세요.

타임존을 명확히 표시하지 않은 시각은 혼동을 일으킬 수 있습니다. 예를 들어서, 점주가 `2018-01-01 13:00:00`부터 오늘 사이에 방문한 고객을 알고 싶어 한다고 가정해봅시다. 만약, 영국에 있는 점주가 이를 요청했다면 GMT로, 한국에 있는 점주가 요청했다면 KST로 처리해야 합니다. 하지만 `2018-01-01 13:00:00` 이 정보만 보고 해당 시각이 어느 지역을 기준으로 한 시각인지 절대 알 수 없습니다. 이런 상황을 해결하기 위해 시각은 **어떤 한 시각**을 기준으로 하여 표시되어야 합니다. 그 기준으로 정한 것이 **UTC** 이고, 위의 시각은 각각 `2018-01-01 13:00:00+00:00`, `2018-01-01 13:00:00+09:00`로 표시할 수 있습니다. 이렇게 표시하면 혼란 없이 정상적으로 처리할 수 있습니다.

## Datetime
[datetime](https://docs.python.org/3/library/datetime.html)은 파이썬에서 기본으로 제공해주는 모듈로, 간단하거나 복잡한 방식으로 날짜와 시각을 조작하기 위한 클래스를 제공합니다.
> The datetime module supplies classes for manipulating dates and times in both simple and complex ways.

datetime의 타입으로는 naive, aware 두 가지 타입이 있습니다. 자세한 내용은 아래에서 다루도록 하겠습니다.

## Type
datetime의 타입을 알아봅시다. 위에서 언급한 것과 같이 naive와 aware 타입이 있습니다.
- **naive** : 타임존을 포함하지 않음.  
`datetime.datetime(2019, 1, 29, 4, 58, 4, 114979)`
> naive 객체는 그 자체만으로 타임존을 찾을 수 있는 충분한 정보를 포함하지 않습니다. 하지만 이런 정보를 포함하지 않아 비교적 이해하고 작업하기 쉽습니다.

- **aware**(timezone-aware) : 타임존을 포함함.  
`datetime.datetime(2019, 1, 9, 4, 58, 4, 114979, tzinfo=<UTC>)`
> aware 객체는 자신의 시각 정보를 다른 aware 객체와 상대적인 값으로 조정할 수 있도록 타임존이나 일광 절약 시간 정책 혹은 적용 가능한 알고리즘 정보를 담고 있습니다.

**naive는 어느 타임존을 기준으로 하는 시각인지 모호하므로 aware를 이용하는 것을 권장합니다.**  

## 직접 확인해보기
준비한 몇 가지 코드를 보며 확인해봅시다. naive와 aware의 차이를 확인하고, 타임존 지정 방법에 대한 내용을 다룹니다.
### 개발환경
- [python](https://www.python.org/downloads/) 3.6
- [pytz](http://pytz.sourceforge.net/#installation)==2018.7

### naive
naive는 날짜와 시각만을 갖습니다.
```python
import datetime

datetime.datetime.utcnow()
# UTC 기준 naive : datetime.datetime(2019, 1, 9, 4, 54, 29, 281594)

datetime.datetime.now()
# local 기준 naive : datetime.datetime(2019, 1, 9, 13, 54, 32, 939155)
```

### aware
pytz 사용에 앞서, 원하시는 타임존을 확인하시려면 다음을 따라 해주세요.
```python
import pytz

for tz in pytz.all_timezones:
    print(tz)
```
naive와 달리 aware는 타임존 정보(tzinfo) 도 갖습니다.
```python
import datetime
from pytz import utc

utc.localize(datetime.datetime.utcnow())
# UTC 기준 aware : datetime.datetime(2019, 1, 9, 4, 55, 3, 310474, tzinfo=<UTC>)
```

### 타임존 제대로 지정하기
타임존이 무엇이고, 명시하는 것이 왜 중요한지 알게 되셨다면 타임존을 원하는 의도에 맞게 지정하는 방법에 대해 알아봅시다.

```python
import datetime
from pytz import timezone, utc

KST = timezone('Asia/Seoul')

now = datetime.datetime.utcnow()
# UTC 기준 naive : datetime.datetime(2019, 1, 9, 4, 18, 28, 805879)

utc.localize(now)
# UTC 기준 aware : datetime.datetime(2019, 1, 9, 4, 18, 28, 805879, tzinfo=<UTC>)

KST.localize(now)
# UTC 시각, 타임존만 KST : datetime.datetime(2019, 1, 9, 4, 18, 28, 805879, tzinfo=<DstTzInfo 'Asia/Seoul' KST+9:00:00 STD>)

utc.localize(now).astimezone(KST)
# KST 기준 aware : datetime.datetime(2019, 1, 9, 13, 18, 28, 805879, tzinfo=<DstTzInfo 'Asia/Seoul' KST+9:00:00 STD>)

```

replace 메소드로 날짜나 타임존을 변경할 수 있습니다.  
하지만 replace는 그 속성 자체만을 바꿔버리는 것이기 때문에 사용에 주의할 필요가 있습니다.
```python
date = datetime.datetime.now()
# datetime.datetime(2019, 1, 9, 13, 59, 44, 872224)

date.replace(hour=10)
# datetime.datetime(2019, 1, 9, 10, 59, 44, 872224)

date.replace(tzinfo=KST)
# datetime.datetime(2019, 1, 9, 13, 59, 44, 872224, tzinfo=<DstTzInfo 'Asia/Seoul' LMT+8:28:00 STD>)

```

## 한 줄 정리
시간 관련 처리를 할 때 타임존 명시를 생활화합시다.
