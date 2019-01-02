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

## Datetime
**datetime**은 파이썬에서 기본으로 제공해주는 모듈로, 간단하고 복잡한 방식으로 날짜와 시간을 조작하기 위한 클래스를 제공합니다.
> The datetime module supplies classes for manipulating dates and times in both simple and complex ways.

자세한 내용은 [여기]((https://docs.python.org/3/library/datetime.html))를 참고해주세요.

## Type
datetime 의 타입으로는 naive, aware 두 가지 타입이 있습니다.
- **naive** : timezone을 포함하지 않음.  
`datetime.datetime(2019, 1, 2, 4, 58, 4, 114979)`
- **aware**(timezone-aware) : timezone을 포함함.  
`datetime.datetime(2019, 1, 2, 4, 58, 4, 114979, tzinfo=<UTC>)`

**naive 는 어느 지역을 기준으로 하는 시간인지 알 수 없으므로 aware 를 이용하는 것을 권장합니다.**  

## 직접 확인해보기
준비한 몇 가지 코드를 보며 확인해봅시다.
### 개발환경
- [python](https://www.python.org/downloads/) 3.6
- [pytz](http://pytz.sourceforge.net/#installation)==2018.7

### naive
naive 는 날짜와 시간만을 갖습니다.
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
naive 와 달리 aware 는 timezone(tzinfo) 도 갖습니다.
```python
import datetime
from pytz import utc

utc.localize(datetime.datetime.utcnow())
# UTC 기준 aware : datetime.datetime(2019, 1, 2, 4, 55, 3, 310474, tzinfo=<UTC>)
```

### Timezone 제대로 지정하기
UTC 기준 datetime의 타임존을 지정한 후 KST 로 타임존을 변경합니다. 
`utc.localize(datetime.datetime.utcnow()).astimezone(KST)`

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

replace 메소드로 날짜나 타임존을 변경할 수 있습니다.  
하지만 replace 는 그 속성 자체만을 바꿔버리는 것이기 때문에 사용에 주의할 필요가 있습니다.
```python
date = datetime.datetime.now()
# datetime.datetime(2019, 1, 2, 13, 59, 44, 872224)

date.replace(hour=10)
# datetime.datetime(2019, 1, 2, 10, 59, 44, 872224)

date.replace(tzinfo=KST)
# datetime.datetime(2019, 1, 2, 13, 59, 44, 872224, tzinfo=<DstTzInfo 'Asia/Seoul' LMT+8:28:00 STD>)

```

## 한 줄 정리
datetime 을 이용할 때 timezone-aware 를 생활화합시다.
