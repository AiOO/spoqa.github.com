---
layout: entry
title: "파이썬의 타임존에 대해 알아보기"
author: 김두리
author-email: dudu@spoqa.com
description: 파이썬의 타임존에 대해서 알아봅니다.
publish: true
---

안녕하세요. 스포카 크리에이터 김두리입니다.

타임존 처리는 프로그래밍을 하다 보면 한 번씩 겪을만한 문제라고 생각합니다. 
오늘은 제가 파이썬으로 타임존 관련 처리를 하며 겪었던 경험을 공유하고 이에 대해 정리해보려 합니다.

## Timezone
나라 또는 지역마다 살아가는 시간이 다르기 때문에 시간대에 따른 편차가 존재합니다. 이 차이가 피부로 잘 와닿지 않은 채 살아가더라도 캘린더 API나 국제화 서비스 준비 등등 시간과 관련된 작업을 진행하다 보면 시간대, 즉 타임존 문제에 직면하게 됩니다.
> 시간대는 영국의 그리니치 천문대(본초 자오선, 경도 0도)를 기준으로 지역에 따른 시간의 차이, 다시 말해 지구의 자전에 따른 지역 사이에 생기는 낮과 밤의 차이를 인위적으로 조정하기 위해 고안된 시간의 구분 선을 일컫는다. 시간대는 협정 세계시(UTC)를 기준으로 한 상대적인 차이로 나타낸다.

UTC에 대한 더 자세한 내용은 [여기](https://ko.wikipedia.org/wiki/%ED%98%91%EC%A0%95_%EC%84%B8%EA%B3%84%EC%8B%9C)를 참고해주세요.  
Timezone에 대한 더 자세한 내용은 [여기](https://en.wikipedia.org/wiki/Time_zone)를 참고해주세요.

## 왜 중요할까?
**나라별 시간대 비교해보기**  
UTC를 기준으로 시간이 빠르면 `+시차`, 시간이 느리면 `-시차`로 표시합니다.

| 시간대 | 나라 | 명칭 | 
| --- | --- | ---:|
| UTC-5 | 미국(동부) | EST |
| UTC | 영국 | GMT |
| UTC+9 | 한국 | KST |
| UTC+9 | 일본 | JST |
| UTC+10 | 오스트레일리아(동부) | AEST |

나라별 시간대 차이에 대한 더 자세한 내용은 [여기](https://ko.wikipedia.org/wiki/%EC%8B%9C%EA%B0%84%EB%8C%80)를 참고해주세요.

타임존을 명확히 표시하지 않은 시간은 혼동을 일으킬 수 있습니다. 예를 들어서, 유효 기간이 한국 시각을 기준으로 `2019-01-02 13:00:00` 인 쿠폰이 있습니다.
한국에 있고, 이 쿠폰을 받은 사람은 아무런 문제 없이 쿠폰을 잘 사용할 수 있습니다. 하지만 서비스가 흥행해 해외로 뻗어 나갔고, 영국에 있는 사람이 이 쿠폰을 받았다고 가정해봅시다.
그렇다면 이 쿠폰에 적힌 유효 기간에 대해 의문이 생길 것입니다. 쿠폰의 유효 기간을 `2019-01-02 13:00:00+09:00` 이렇게 표시하면 한국을 기준으로 한 시간이라는 것을 확실하게 알 수 있습니다.
깔끔해 보이진 않지만, 이해를 돕기 위한 하나의 예시로만 봐주세요.

저는 실제로 얼마 전, Google calendar API를 이용하여 작업할 때 timezone 처리 관련하여 골치 아픈 일을 겪었습니다. 분명 한국 시각을 기준으로 요청했지만, 데이터가 원하는 대로 불러와 지지 않아서 애를 먹었습니다. 몇 시간의 고생 끝에 결국, timezone을 지정하지 않아 UTC 시간으로 인식되어서 발생했던 문제라는 것을 알게 되었습니다. 그래서 이 문제를 겪으면서 모았던 정보를 정리하여 공유하고자 글을 작성하게 되었습니다.


## Datetime
[datetime](https://docs.python.org/3/library/datetime.html)은 파이썬에서 기본으로 제공해주는 모듈로, 간단하거나 복잡한 방식으로 날짜와 시간을 조작하기 위한 클래스를 제공합니다.
> The datetime module supplies classes for manipulating dates and times in both simple and complex ways.

datetime의 타입으로는 naive, aware 두 가지 타입이 있습니다. 자세한 내용은 아래에서 다루도록 하겠습니다.

## Type
datetime의 타입을 알아봅시다. 위에서 언급한 것과 같이 naive와 aware 타입이 있습니다.
- **naive** : timezone을 포함하지 않음.  
`datetime.datetime(2019, 1, 2, 4, 58, 4, 114979)`
> naive 객체는 그 자체만으로 위치를 찾을 수 있는 충분한 정보를 포함하지 않습니다. 하지만 이런 정보를 포함하지 않아 비교적 이해하고 작업하기 쉽습니다.

- **aware**(timezone-aware) : timezone을 포함함.  
`datetime.datetime(2019, 1, 2, 4, 58, 4, 114979, tzinfo=<UTC>)`
> aware 객체는 그 자체만으로 시간대 및 일광 절약 시간 정보와 같은 적용 가능한 알고리즘 및 정치 시간 조정에 대한 충분한 정보를 갖추고 있습니다.

**naive는 어느 지역을 기준으로 하는 시간인지 모호하므로 aware를 이용하는 것을 권장합니다.**  

## 직접 확인해보기
준비한 몇 가지 코드를 보며 확인해봅시다. naive와 aware의 차이를 확인하고, timezone 지정 방법에 대한 내용을 다룹니다.
### 개발환경
- [python](https://www.python.org/downloads/) 3.6
- [pytz](http://pytz.sourceforge.net/#installation)==2018.7

### naive
naive는 날짜와 시간만을 갖습니다.
```python
import datetime

datetime.datetime.utcnow()
# UTC 기준 naive : datetime.datetime(2019, 1, 2, 4, 54, 29, 281594)

datetime.datetime.now()
# local 기준 naive : datetime.datetime(2019, 1, 2, 13, 54, 32, 939155)
```

### aware
pytz 사용에 앞서, 원하시는 timezone을 확인하시려면 다음을 따라 해주세요.
```python
import pytz

for tz in pytz.all_timezones:
    print(tz)
```
naive와 달리 aware는 timezone 정보(tzinfo) 도 갖습니다.
```python
import datetime
from pytz import utc

utc.localize(datetime.datetime.utcnow())
# UTC 기준 aware : datetime.datetime(2019, 1, 2, 4, 55, 3, 310474, tzinfo=<UTC>)
```

### Timezone 제대로 지정하기
timezone이 무엇이고, 명시하는 것이 왜 중요한지 알게 되셨다면 timezone을 원하는 의도에 맞게 지정하는 방법에 대해 알아봅시다.

```python
import datetime
from pytz import timezone, utc

KST = timezone('Asia/Seoul')

now = datetime.datetime.utcnow()
# UTC 기준 naive : datetime.datetime(2019, 1, 2, 4, 18, 28, 805879)

utc.localize(now)
# UTC 기준 aware : datetime.datetime(2019, 1, 2, 4, 18, 28, 805879, tzinfo=<UTC>)

KST.localize(now)
# UTC 시간, timezone만 KST : datetime.datetime(2019, 1, 2, 4, 18, 28, 805879, tzinfo=<DstTzInfo 'Asia/Seoul' KST+9:00:00 STD>)

utc.localize(now).astimezone(KST)
# KST 기준 aware : datetime.datetime(2019, 1, 2, 13, 18, 28, 805879, tzinfo=<DstTzInfo 'Asia/Seoul' KST+9:00:00 STD>)

```

replace 메소드로 날짜나 timezone을 변경할 수 있습니다.  
하지만 replace는 그 속성 자체만을 바꿔버리는 것이기 때문에 사용에 주의할 필요가 있습니다.
```python
date = datetime.datetime.now()
# datetime.datetime(2019, 1, 2, 13, 59, 44, 872224)

date.replace(hour=10)
# datetime.datetime(2019, 1, 2, 10, 59, 44, 872224)

date.replace(tzinfo=KST)
# datetime.datetime(2019, 1, 2, 13, 59, 44, 872224, tzinfo=<DstTzInfo 'Asia/Seoul' LMT+8:28:00 STD>)

```

## 한 줄 정리
datetime을 이용할 때 timezone-aware를 생활화합시다.
