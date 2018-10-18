---
layout: entry
title: 도도 파이터 제출 돌아보기
author: 박준규
author-email: kyu@spoqa.com
description: 도도 파이터에 제출된 코드들을 둘러봅니다.
publish: true
---

안녕하세요. 스포카 개발팀 박준규입니다.

이번에는 [도도 파이터]에 제출된 작품들을 분석한 뒤에 어떤 참신한 제출이 있었는지를 알아보도록 하겠습니다.

[파이콘 한국 2018] 기간 중 총 71개의 제출이 등록되었습니다. 당초 계획은 코드를 하나하나씩 분석하여 분류할 계획이었으나 제출의 수가 너무 많은 관계로 전부 다루기는 어렵고, 대신 주목할 만한 제출들을 중심으로 알아보고자 합니다.

## 통계로 보는 도도 파이터

저희는 파이콘 한국 2018에서 원활한 대회 진행을 위하여 토너먼트로 진행했습니다. 토너먼트로 진행하면 빠른 진행이 가능하고 화제성을 만들기 쉽다는 장점이 있지만, 객관적인 강함을 측정하기가 어렵다는 단점이 있습니다. 에이전트간 상성에 따른 차이로 인해 객관적인 강자가 일찍 탈락할 수도 있기 때문입니다. 분석을 위해 저희는 71개의 제출을 대상으로 총 4970회의 매치를 돌려보았습니다. 그 결과 10위까지의 순위는 아래와 같습니다.

| 순위 | 참가자 | 승리 회수 | 
| --- | --- | ---:|
| 1 | [leehosung](https://github.com/leehosung) | 125 |
| 2 | [kdc0110](https://github.com/kdc0110) | 124 |
| 3 | [AiOO](https://github.com/AiOO) | 116 |
| 4 | [juo6442](https://github.com/juo6442) | 114 |
| 5 | [sunghyunzz](https://github.com/sunghyunzz) | 111 |
| 6 | [seeeturtle](https://github.com/seeeturtle) | 109 |
| 7 | [qria](https://github.com/qria) | 100 |
| 8 | [earlbread](https://github.com/earlbread) | 98 |
| 9 | [chitack](https://github.com/chitack) | 94 |
| 10 | [gurals5712](https://github.com/gurals5712) | 94 |

여기서 승리 회수를 좀 더 정확한 강함의 기준으로 판단할 근거가 될 수 있지 않을까요? 이 제출들을 분석하기 앞서 전체적인 경향을 분석해 봅시다.

도도 파이터에서 에이전트로 전달되는 게임 상태 파라미터는 아래와 같습니다.

* `distance`: 상대와 나 간의 거리
* `time_left`: 남은 턴 수
* `health`: 나의 체력
* `opponent_health`: 상대의 체력
* `opponent_action`: 직전 턴에서 상대의 움직임
* `given_damage`: 직전 턴에서 내가 가한 데미지
* `taken_damage`: 직전 턴에서 내가 받은 데미지
* `match_records`: 전적 정보

각 제출들은 이 정보를 얼마나 참조하고 있을까요?

| 파라미터 | 회수 |
|:--- | ---:|
| `distance` | 67 |
| `health` | 30 |
| `opponent_action` | 29 |
| `opponent_health` | 29 |
| `time_left` | 22 |
| `taken_damage` | 10 |
| `given_damage` | 9 |
| `match_records` | 5 |

상대와 나 사이의 거리가 0이어야 공격이 가능하기 때문에 이걸 확인하기 위해서 `distance` 정보를 참고해야 합니다. 공격이 성립하기 위해서는 필수적으로 참조해야 하는 부분이라 역시 `distance`를 참고하는 비중이 가장 높군요. 반면 `taken_damage`와 `given_damage`, `match_records`는 많이 참조하지 않는다는 것을 알 수 있습니다.

그렇다면, 에이전트가 가장 많이 취하는 행동은 무엇일까요? 4970회의 매칭 결과를 종합해 보았습니다.

| 액션 | 회수 |
|:--- | ---:|
| `punch` | 122,007 |
| `kick` | 112,016 |
| `forward` | 71,429 |
| `guard` | 46,751 |
| `backward` | 41,212 |
| `jump` | 27,786 |
| `crouch` | 27,170 |
| `idle` | 21,365 |

공격 액션이 가장 빈번한 것을 알 수 있습니다. "방어가 최선의 공격"을 의도했지만 역시 공격이 최선이었던 걸까요.

## 가장 효율적인 전략은?

상위 10개의 제출들이 각각 어떤 전략을 취하는지 알아보겠습니다.

* 10위 [gurals5712](https://github.com/gurals5712) 님의 [제출](https://gist.github.com/segfault87/c6bd3572ca48ad970fcfb087362f774b#file-gurals5712-py)

  기본적으로 동작을 무작위로 선택하는 전략이지만 상대방과의 거리에 따라서 선택하는 동작에 차이가 있습니다.

* 9위 [chitack](https://github.com/chitack) 님의 [제출](https://gist.github.com/segfault87/c6bd3572ca48ad970fcfb087362f774b#file-chitack-py)

  역시 무작위 기반의 전략이지만 상대방과 나의 거리와 체력을 비교하여 선택하는 동작에 차이가 있습니다.

* 8위 [earlbread](https://github.com/earlbread) 님의 [제출](https://gist.github.com/segfault87/c6bd3572ca48ad970fcfb087362f774b#file-earlbread-py)

  거리가 1 이상이면 근접할 때까지 앞으로 이동과 방어를 번갈아가면서 하고, 근접하면 공격과 방어를 번갈아가며 합니다.

* 7위 [qria](https://github.com/qria) 님의 [제출](https://gist.github.com/segfault87/c6bd3572ca48ad970fcfb087362f774b#file-qria-py)

  이른바 [니가와] 전술입니다. 한 칸 정도 거리를 유지하며 상대방이 와서 맞아줄 때까지 계속 공격을 반복합니다.

* 6위 [seeeturtle](https://github.com/seeeturtle) 님의 [제출](https://gist.github.com/segfault87/c6bd3572ca48ad970fcfb087362f774b#file-seeeturtle-py)

  현재 내 체력이 상대 체력보다 높다면 무작위로 선택, 그렇지 않다면 공격과 방어 중에 선택하도록 합니다.

* 5위 [sunghyunzz](https://github.com/sunghyunzz) 님의 [제출](https://gist.github.com/segfault87/c6bd3572ca48ad970fcfb087362f774b#file-sunghyunzz-py)

  * 내 체력이 상대 체력보다 낮다면,
    * 남은 턴이 6턴 이하일 경우에는 앞으로 가서 공격합니다.
    * 그 외의 경우에는 1칸 거리를 유지하고 [니가와] 모드로 공격합니다. 만약 딱 붙었을 경우에는 2/3의 확률로 방어, 1/3의 확률로 공격을 합니다.
  * 그렇지 않다면,
    * 1칸 거리를 계속 유지하면서 니가와 모드로 공격합니다. 만약 뒤로 갈 수 있다면 뒤로 빠지면서 공간을 확보하려고 시도합니다.
    * 뒤로 갈 수 없다면 앞으로 가려고 시도합니다.
  
* 4위 [juo6442](https://github.com/juo6442) 님의 [제출](https://gist.github.com/segfault87/c6bd3572ca48ad970fcfb087362f774b#file-juo6442-py)

  * 7턴 이하로 남았고 내 체력이 상대보다 높다면 무조건 뒤로 도망갑니다.
  * 남은 턴수 - 3이 상대 체력 - 내 체력보다 작다면, 무조건 앞으로 가서 공격을 반복합니다.
  * 그 외의 경우,
    * 상대방과 한 칸의 거리를 유지합니다.
    * 이전 턴에서 공격이 회피되거나 방어된 적이 있으면 다음 턴도 방어를 택합니다.
    * 그 외에는 공격을 택합니다.
    
* 3위 [AiOO](https://github.com/AiOO) 님의 [제출](https://gist.github.com/segfault87/c6bd3572ca48ad970fcfb087362f774b#file-aioo-py)

  1칸 간격으로 니가와 전술을 구사합니다. 상대가 접근하면 뒤로 도망가다가 더 이상 뒤로 도망가지 못하면 무작위로 액션을 선택합니다.

* 2위 [kdc0110](https://github.com/kdc0110) 님의 [제출](https://gist.github.com/segfault87/c6bd3572ca48ad970fcfb087362f774b#file-kdc0110-py)

  한 칸 거리를 간격으로 두고 니가와 전술을 구사합니다. 만약 내 체력이 상대보다 높은 상황이 되면 최대한 뒤로 도망을 시도합니다.

* 1위 [leehosung](https://github.com/leehosung) 님의 [제출](https://gist.github.com/segfault87/c6bd3572ca48ad970fcfb087362f774b#file-leehosung-py)

  이번 대회에서 1등을 수상한 [leehosung](https://github.com/leehosung) 님의 제출 역시 1칸 간격을 유지하며 니가와 전술을 사용하고 있습니다. 그 외에 방어 또는 회피에 성공할 경우에 공격을 시도하여 데미지의 극대화를 노리거나 이전 공격이 실패했을 경우 방어 또는 회피를 시도하며, 현재 자신의 체력에 따라서 공격과 방어 비율을 조정하는 등 추가적인 전술을 활용하고 있습니다.

## The Honorable Mention

위에서 보셨지만 전적이 높은 제출들의 전략은 대체로 수렴하는 경향이 있습니다. 하지만 남들이 시도하지 않은 참신한 코드야말로 이런 컨테스트의 묘미이겠지요. 저희는 비록 우승에 성공하지는 못했으나 참신하다고 판단되는 코드를 선정하여 상품을 드리기로 했습니다.

저희는 이 부문의 우승자로 [xenosoz](https://github.com/xenosoz) 님의 [제출](https://gist.github.com/segfault87/12f4e29fe41a676d6e8087dce53cf4f5)을 선정했습니다.

코드를 보면 들어간 정성이 어마어마하다는 것을 짐작할 수 있습니다. 다른 제출들이 정형화된 패턴을 통한 플레이 전략을 구사한다면, 이 제출은 상대방의 플레이 패턴을 분석하고 다음 행동을 예측하여 행동하는 코드입니다. 조금 더 고차원적인 인공지능에 가까운 모양이죠. 비록 우승하지는 못했지만, 총 71건이 제출 중에서 이러한 방식으로 구현된 제출은 이것이 유일합니다. 따라서 저희는 참신한 코드 상의 수상자로 xenosoz 님을 선정했습니다.

수상자께서는 [kyu@spoqa.com](mailto:kyu@spoqa.com)로 메일을 보내 주시면 상품을 수령하실 수 있도록 안내 드리도록 하겠습니다. 축하드립니다.

그럼, 다음 파이콘 한국 행사에서도 재미있는 이벤트를 가지고 찾아뵙겠습니다. 감사합니다.

  [도도 파이터]: https://pycon-2018-dodo-fighter.spoqa.com
  [파이콘 한국 2018]: https://www.pycon.kr/2018/
  [니가와]: https://namu.wiki/w/%EB%8B%88%EA%B0%80%EC%99%80c

