---
layout: entry
title: 본격 1주일 만에 (고퀄리티) 회의실 탈출 게임 만들기
author: 강영화, 김민준, 김우현
description: 2018년 8월에 열린 파이콘 한국 2018에서 선보인 방탈출 게임을 만들었던 경험을 공유합니다.
publish: true
---

!! 스포일러 주의 !! <br />
(아직 풀어보지 않은 분들은 [파이콘 발표하러 가는길 : 스포카 방탈출][] 에서 해보세요!)


안녕하세요. 스포카 크리에이터팀에서 [파이콘 한국 2018][]에 부스에 참가했습니다. 이번 부스에서는 여러 이벤트를 했었는데요! 그 중 저희는 파이콘 준비 중에 사전 이벤트로 **파이콘 발표하러 가는길: 스포카 방탈출**을 선보였습니다.

<p align="center">
  <img src="/images/2018-08-30/escape_ga_data.png" width="2000px" />
</p>

총 1,441분(!!)께서 탈출에 도전하셨고, 434분께서 탈출에 성공하셨습니다. 그리고 275분께서 파이콘 부스 설명 링크를 클릭해 주셨습니다. 탈출에 도전한 숫자는 파이콘 2018에 참여한 모든 사람 수에 맞먹는 수치입니다. 생각보다 많은 분이 관심 가져주셔서 놀랍고 뿌듯했습니다!

<figure>
  <img src="/images/2018-08-30/gameplay.gif" width="2000px" />
  <figcaption>게임 플레이 장면</figcaption>
</figure>

이 게임은 360도 카메라로 촬영한 실제 스포카 회의실을 배경으로 하여, 각개 각소에 있는 단서를 모아서 회의실에서 탈출하는 것이 목표인 게임입니다. 오늘은 이 이벤트를 어떻게 만들게 되었는지 간단히 소개하고자 합니다.

<figure>
  <img src="/images/2018-08-30/360camera.jpg" width="350px" />
  <figcaption>찍은 사람이 안 찍힌 이유</figcaption>
</figure>

## 만들어진 배경

우선 파이콘 이벤트 아이데이션 중 어떤 이벤트를 하면 좋을지 익명의 설문조사를 했는데요! 그 중 미궁 게임이라는 키워드가 눈에 띄었습니다. 처음에는 암호 해독 등과 같은 프로그래머만이 풀 수 있는 문제를 내려 했었는데 기획하던 중 프로그래머뿐만 아니라 다른 직군도 저희가 어떤 일을 하는지 알리면 더 많은 분이 즐겁게 참여할 수 있겠다 싶었습니다. 그래서 프로그래머 외 직군 모두가 해볼 수 있는 이벤트를 만들어보자고 이야기를 나눴습니다. 다시 아이데이션 하는 도중 [방탈출][]이라는 요소가 생각났고, 배경을 실제 저희가 사용하는 회의실을 활용해 현장감을 살리고 그 안에 스포카의 좋은 문화를 담아 저희 회사를 소개해보는 것을 목표로 했습니다.


게임을 만들면서 크리에이터 팀만의 문화를 담고 싶었는데, 그 중 대표적인 문화가 **회고**와 **오픈소스** 였습니다.


### 회고

스포카에서는 팀별로 1~2주마다 회고를 합니다. 회고는 잘한 일은 다음에 더 잘 할 수 있게, 잘못한 일은 다음에 더 잘 할 수 있게끔 하는 데 의의가 있습니다. 구성원들이 더 나은 문화를 함께 만들어 나갈 수 있게 하는 회고를 테크 업계 대부분 팀에서도 도입할 수 있기를 바라며 회고 문화를 게임에 넣어보았습니다. “[스포카에서는 회고를 어떻게 할까?][]” 포스트를 읽어보시면 더 자세히 알 수 있습니다.


### 오픈소스

혹시 게임을 진행하시면서 프로젝터 쪽에 힌트를 찾으셨나요? 이는 스포카 오픈소스 폰트인 [스포카 한 산스][] 를 사용하였습니다. 스포카 한 산스를 소개하는 웹페이지에서 [미리 보기][]로 타이핑 한 장면을 캡처하여 요소를 넣었습니다.


팀에서 프로그래밍하면서 현실의 여러 복잡한 문제들을 마주합니다. 이 문제들은 어느 한 사람이나 회사의 역량만으로 해결하기 벅찬 경우가 많습니다. 이런 문제들을 해결하기 위해 오픈 소스의 힘을 많이 빌렸는데, 그렇기에 저희는 오픈소스가 이 생태계에 좋은 영향을 가져다주기를 바랍니다. 이러한 오픈소스 커뮤니티를 위해 팀 내에서 일정 부분 기여하고 있습니다. 더 많은 프로그래머가 저희와 함께 해주시길 바라면서요!


실제로 이 프로젝트도 [Vue.js][], [three.js][]를 활용한 [panolens.js][] 등의 다양한 오픈소스를 활용하여 만들었습니다.

<figure>
  <img src="/images/2018-08-30/ghost.png" width="2000px" />
  <figcaption>유령도 찾으셨나요?</figcaption>
</figure>

이 프로젝트는 7월 28일부터 8월 9일까지 차근차근 만들어졌는데요, 사실상 개발 기간은 일주일이었습니다. 프로젝트를 진행하면서 서비스 개발이라는 본업에서 잠시 벗어나 게임 개발이라는 새로운 장르에 도전해 보게 되었습니다. 이게 다 파이콘과 파이썬 커뮤니티 덕분이지요. ㅇㅅaㅇ
또한 교류가 보통 없는 다른 팀분들과 협업도 해볼 수 있었고, 유지보수 걱정 없는 토이 프로젝트로서 Vue.js, three.js 등 현재 [도도 포인트][]에서 쓰지 않는 기술을 적용해본 일도 즐겁고 신선한 경험이었습니다. 기회가 된다면 다시 하고 싶네요! (특별휴가는 보너스 ㅇㅅㅇ/)


재밌는 프로젝트를 같이 만들어나갈 프로그래머분들을 만나고 싶네요! 배움을 즐기고, 성장에 재미를 느끼며, 자유로움을 좋아하는 프로그래머 여러분, 스포카는 당신을 기다리고 있습니다. **I WANT YOU ㅇㅅㅇ/**


[파이콘 한국 2018]: https://www.pycon.kr/2018/
[방탈출]: https://ko.wikipedia.org/wiki/%EB%B0%A9%ED%83%88%EC%B6%9C_%EC%B9%B4%ED%8E%98
[파이콘 발표하러 가는길 : 스포카 방탈출]: https://pycon-2018-escape.spoqa.com/
[스포카 한 산스]: https://spoqa.github.io/spoqa-han-sans/
[미리 보기]: https://spoqa.github.io/spoqa-han-sans/ko-KR/#preview
[스포카에서는 회고를 어떻게 할까?]: https://spoqa.github.io/2018/08/29/retrospect.html
[Vue.js]: http://vuejs.org/
[three.js]: https://threejs.org/
[panolens.js]: https://github.com/pchen66/panolens.js
[도도 포인트]: https://www.dodopoint.com/
