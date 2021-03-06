---
layout: entry
title: 다국어 환경에 맞게 타이포그래피 세팅하기 - 다국어 반응형 타이포그래피
author: 강영화
author-email: kang@spoqa.com
description: 도도 포인트 태블릿앱에 작성된 한국어, 일본어 환경에 맞게 Sass로 반응형 타이포그래피 스타일을 정의한 방법을 설명합니다.
publish: true
---
안녕하세요. 스포카의 Product Designer 강영화입니다. 이번 글에서는 스포카에서 도도 포인트 태블릿 앱(이하 스토어앱)의 다국어 환경에 어떻게 효율적으로 타이포그래피 스타일을 정의하고 있는지 적어봅니다.

언어는 그 언어에 맞는 고유한 문자가 있습니다. 그리고 그 문자별로 특징적인 형태를 지닙니다. 따라서 문자의 기본적인 문자폭과 자간 등 타이포그래피 세팅도 다르게 되어야 합니다. 이 글에서는 스포카의 대표 서비스인 도도 포인트 태블릿 앱에 적용되어있는 **CSS `font-size`속성을 한국어 환경과 일본어 환경 각각에 효과적으로 정의하는 방법**을 소개하고자 합니다. 이 글이 두 가지 이상의 다국어 언어 환경을 다뤄야 하는 개발자, 디자이너에게 도움이 되었으면 좋겠습니다.

# 다국어 UI별 타이포그래피 세팅의 필요성

다국어 UI별 타이포그래피 설정에 대한 필요성은 글로벌 서비스를 한글로 사용해본 사람들은 쉽게 공감할 수 있을 것입니다. 해외 본사를 둔 웹 서비스를 이용할 때 영문 글꼴은 굉장히 완성도 높은 글꼴로 설정되어있는데 한글은 굴림으로(...) 나오는 상황을 떠올려보세요. 혹은 줄바꿈 설정이 CJK를 고려하지 않고  `word-break: normal;`로 설정되어있는 경우[^1] 등등 .... 우리의 일상적인 웹 환경에서 이러한 상황을 쉽게 접할 수 있습니다. 이런 예시들은 CJK 문자 환경에서 웹을 사용하고 있는 사용자 입장으로서 굉장히 괴롭습니다. 디자이너에게는 이런 상황들이 더 크게 느껴지기 마련입니다.

<figure>
  <img src="/images/2016-06-03/grrrr.png"
    style="margin-right:auto; margin-left:auto;" />
  <figcaption>
    (좌) 줄바꿈 설정이 단어 단위가 아니라 너비로 인해 제목의 &lsquo;결론&rsquo;과 &lsquo;을&rsquo;이 줄바꿈된 다국어 대응 웹사이트<br>
    (우) 신경써서 지정된 영문 글꼴과 대비되는 기본 글꼴이 지정된 한글꼴
  </figcaption>
</figure>

세계 곳곳에서 사용하는 글로벌 서비스라면 모든 언어에 대해 완벽하게 타이포그래피를 적용하는 것은 현실적으로 무리가 있습니다. 하지만 스포카 처럼 두 나라에 걸쳐 서비스를 운영한다면 당연하게도 두 언어 환경 모두에서 깨지지 않고 완성도 있게 출력되도록 하는 편이 좋습니다.



# 태블릿 기기에서 보이는 국문과 일문의 특징
그렇다면 스포카에서 대응하고 있는 국문과 일문은 어떤 특징을 지니고 있을까요?

<figure>
  <img src="/images/2016-06-03/width_difference.png"
     style="margin-right:auto; margin-left:auto;" />
  <figcaption>
    가나와 한글의 글자 너비 특징 비교
  </figcaption>
</figure>

한글은 표음문자 이면서 자음과 모음에 대응하는 각각의 음소가 있는 음소문자입니다. 일본어 문자의 경우 표음문자와 표의문자가 함께 섞여져 있는 형태로 문장 구성이 됩니다. 이런 문자적 특징도 있지만, 태블릿 앱 등 디지털 사용자 인터페이스 환경에서 가장 도드라지는 특징은 가나가 한글보다 글자 너비가 넓다는 점입니다. 폰트 제작 프로그램에서 확인했을 때도 같은 것을 확인할 수 있습니다. 한글의 글자 너비가 860~1000pt에 맞춰져 있다면 일본 문자들의 경우는 이보다 넓은 너비에 맞춰져 있는 경우가 많습니다. [^2]

<figure>
  <img src="/images/2016-06-03/word-break.png"
     style="margin-right:auto; margin-left:auto;" />
  <figcaption>
    고정 너비로 인해 문장이 줄바꿈 되는 텍스트 박스
  </figcaption>
</figure>

<figure>
  <img src="/images/2016-06-03/word-break-bug.png"
     style="margin-right:auto; margin-left:auto;" />
  <figcaption>
    의도하지 않은 줄바꿈이 발생하는 도도 포인트 스토어앱 화면
  </figcaption>
</figure>

이런 특징을 가지고 있기 때문에 일본어 환경에서 한글과 같게 글자 크기를 설정하면 좁은 너비에 작성된 문자열들이 의도에 맞지 않게 줄바꿈 됩니다. 태블릿 앱은 비교적 넓은 너비의 데스크탑 모니터로 보는 웹사이트 환경과 달리 각 컴포넌트가 다소 한정된 너비를 지니기 때문에 이러한 <del>와장창</del> 현상이 더 자주 일어납니다.

CSS로 자간을 좁히거나, 문자 사이 간격을 좁히는 방법도 사용할 수 있지만, 도도 포인트 스토어앱에서는 일본어 환경일 때 글자 크기를 90%로 설정하는 방법을 선택했습니다. 그런데 이 방법을 택했을 때 각각의 코드를 수기로 계산해서 넣는다면 엄청나게 많은 시간이 소요되고 누락되는 일도 잦을 것입니다. 따라서 우리는 아래와 같은 방법을 활용했습니다.[^3]

1. CSS의 [다양한 길이 단위](https://developer.mozilla.org/en-US/docs/Web/CSS/length)를 파악해 이용하고, 
2. Sass `@mixin`으로 단위를 한 번에 치환시켜 일본어 환경일 때에만 모든 글자 크기를 90%로 설정하는 방법을 도도 포인트 스토어앱에 적용시켰습니다.

# 초기 세팅하기

다국어 인터페이스를 위한 스타일을 정의하려면 어떤 언어에 정의하려는지 인지할 수 있도록 브라우저에 힌트를 줘야 합니다. 이때 [의사 클래스](https://developer.mozilla.org/en-US/docs/Web/CSS/Pseudo-classes)(pseudo-class) `:lang()`을 씁니다. 괄호 안에는 각 언어의 고유한 코드를 넣습니다. 이를테면 일본어 환경일 때만 선택하려고 한다면 `:lang(ja)`로 일본어 언어 환경을 선택합니다. 괄호 안에는 일문을 나타내는 `ja`라는 문자열이 들어갑니다.

하지만 언어를 의사 클래스로 선택하기 전에 HTML 문서에 언어 속성이 정의가 되어있지 않았다면 이 `:lang()` 의사 클래스를 선택자로 사용할 수 없습니다. 따라서 아래와 같이 HTML 제일 상위의 인덱스 문서에 언어 속성을 추가해주어야 합니다.

```
<html lang="ja">
```
랜딩페이지 같은 간단한 구조의 웹사이트에는 위와 같은 방법으로 HTML 문서에 추가할 수 있겠습니다. 도도 포인트 스토어앱에는 python으로 HTML 문서에 언어 속성을 추가했습니다.


# CSS의 길이 단위 살펴보기
이제 기초적인 준비는 모두 끝났습니다만 한가지 더 알아야 할 게 있습니다. 이 글에서 설명하는 방법를 이해하려면 몇가지 CSS의 길이 단위와 그 기본값[^4], 이 단위들이 `font-size` 속성에서 어떻게 동작하는지 원리를 이해해야 합니다. 픽셀 단위는 절대 단위라 상위 요소에 따르거나 상대적으로 동작하지 않지만, 퍼센트 단위, 그리고 em과 rem은 상위 요소에 따라 상대적으로 동작합니다. 이 절댓값과 상댓값의 차이와 각 단위 간의 연관성을 알아야 앞으로의 예제를 이해하고 활용할 수 있습니다. **이 글에서 다룰 단위는 픽셀, 퍼센트, em과 rem입니다.**

### 픽셀 단위
픽셀 단위는 디자이너와 개발자 모두에게 익숙한 단위입니다. 픽셀은 1화소를 나타내는 절댓값입니다. 거의 모든 그래픽 툴에서 기본 단위로 설정할 수 있으며 이미지 크기를 나타낼 때에도 자주 사용합니다. 이 단위는 브라우저 너비나 상위 속성에 구애받지 않는 고정된 크기 단위입니다. 아무 설정도 해주지 않았을 때 `font-size`의 기본 픽셀값은 16px입니다.

### 퍼센트 단위
상위 요소에 정의된 픽셀값을 기준으로 상대적으로 동작하는 상대단위입니다. 퍼센트 단위로 상위 요소에 정의된 픽셀값에 따라 하위 요소에 있는 텍스트 크기를 늘리고 줄일 수 있습니다. 어떠한 값도 정의하지 않았을 때 `font-size`에 기본으로 정의되어 있는 퍼센트 값은 100%이며 이는 픽셀 단위로 치환하면 16px이 됩니다.

### em과 rem 단위
em은 대문자 M의 너비를 기준으로 폰트 크기를 측정한다는 어원을 지닌 글자 크기 단위입니다. em을 사용하면 바로 상위 요소인, 부모 요소에 정의한 크기를 기준으로 크기를 늘리거나 줄일 수 있습니다.

```
<div style='font-size: 10px;'>
  <p style='font-size: 2em;'>
    foo
  </p>
</div>
<div style='font-size: 20px;'>
  <p style='font-size: 2em;'>
    bar
  </p>
</div>
```

<figure>
  <img src="/images/2016-06-03/foo_2em_20px_bar_2em_40px.png"
   style="margin-right:auto; margin-left:auto;" />
  <figcaption>
    <a href="http://codepen.io/young-hwa/pen/beNOZX" target="_blank">2em = 20px 인 foo와 2em = 40px인 bar</a>
  </figcaption>
</figure>

위와 같이 정의 하면 foo는 10px의 2배인 20px, bar는 20px의 2배인 40px로 동작합니다. 두 텍스트가 상위 요소에 정의된 값이 다르므로 같은 em값을 지니더라도 다른 크기로 동작합니다.

한편 rem은 부모를 기준으로 크기를 조정하지 않고 최상위 요소, 즉 root를 기준으로 동작합니다. HTML 문서의 최상위 요소는 `<html>`입니다. rem 단위를 사용하면 `<html>`에 정의된 `font-size` 값이 어떤 것이든 그 값을 1rem으로 인식하고 동작합니다. 일테면 한 HTML 문서의 `<html>` 요소에 `font-size`가 12px로 정의되어있으면 그 문서의 rem 값은 12px이 되는 것입니다. 그리고 rem 단위로 어떤 요소에 정의한다 하더라도 부모 요소의 영향을 받지 않고 `<html>`요소에 정의되어있는 값을 받아서 어디서든 1rem = 12px인 값을 유지하게 됩니다. 간단히 작성해보면 아래와 같습니다. 

```
<html style='font-size: 12px;'>
  <body>
    <div style='font-size: 5px;'>
      <p style='font-size: 2rem;'>
        foo
      <p>
    </div>
  </body>
</html>
```
<figure>
  <img src="/images/2016-06-03/foo_2em_24px.png"
     style="margin-right:auto; margin-left:auto;" />
  <figcaption>
    <a href="http://codepen.io/young-hwa/pen/xObmBX" target="_blank">2em = 24px 인 foo</a>
  </figcaption>
</figure>

위 결과 처럼, 바로 위 `<div>` 요소에 `font-size: 5px;`로 정의했다고 하더라도, foo는 24px로 출력됩니다. 부모 요소가 아닌 최상위 요소인, `<html>`에 있는 `font-size`를 상속받기 때문입니다.

캡션에 연결한 Codepen 링크로 접속해서 직접 수치를 바꿔가며 테스트 해보세요.

### 서로 다른 단위 간의 연관성 파악
`font-size`의 단위는 서로 연관성이 있습니다. 어떻게 연관성이 있는지 살펴보겠습니다. 어떤 값도 정의하지 않았을 때 기본 값은 아래와 같습니다.

>100% = 1rem = 16px

하지만 `<html>` 요소에 `font-size= 90%;`를 적용하면 어떻게 될까요?  

>90% = 1rem = 14.4px

위와 같이 90%로 `font-size` 속성을 정의하면 1rem은 90%를 1rem으로 인식해 14.4px을 1rem으로 동작하게 합니다. 절댓값인 픽셀값은 변하지만 상댓값인 rem은 변하지 않습니다. **이렇게 단위 간의 연관성을 이해하고 적절하게 사용하는 것이 이 글에서 다루는 방법의 핵심입니다.** 상대적인 단위의 특성을 이용하는 것입니다.

# 퍼센트와 rem 단위를 이용해 글자 크기 한 번에 조정하기
이제 본격적으로 코드를 보면서 설명하겠습니다.

### 일본어 환경의 `<html>` 요소에 퍼센트 단위로 글자 크기 정의하기
국문은 그대로 두고 일문일 때 `<html>` 요소의 `font-size`속성을 90%로 정의할 것입니다. 그리고 모든 `font-size` 단위를 픽셀 단위에서 rem 단위로 계산해 치환하는 믹스인을 작성하고 이 믹스인을 모든 CSS `font-size` 속성에 `@include` 합니다.

```
$base-font-size: 100%;

html {
  font-size: $base-font-size;
}
```
기존 `font-size` 속성에 정의되어있는 값을 일본어 환경에서 다르게 동작하도록 하려고 CSS 속성을 추가로 작성해야 합니다. 이렇게 추가로 작성된 값을 변수를 활용해 수식화하고 그것을 오버라이드 하기 위해 기본값을 지정해주었습니다.

```
/*For Japanese locale*/

html:lang(ja) {
  font-size: $base-font-size * 0.9;
}
```

위 코드는 일본어 환경일 때 기존에 100%로 정의된 `$base-font-size`를 90%로 지정하는 코드입니다. 일본어 환경에서 `<html>`의 `font-size`속성을 90%로 정의하고 나머지 모든 `font-size`를 픽셀 단위에서 rem 단위로 치환하면 rem 단위는 최상위 단위인 `<html>`요소를 기준으로 자신의 크기를 정의하기 때문에 rem 단위의 모든 `font-size`는 90%로 조정 되어 보이도록 동작합니다. 그리고 한국어 환경에서의 `font-size`는 변경하지 않았기 때문에 기본값인 100%로 보입니다.

### 믹스인으로 px을 rem 단위로 변환하기
이제 믹스인으로 픽셀 단위로 지정되어있던 모든 글자 크기를 rem 단위로 변경해 줄 일만 남았습니다.

```
// Mixin For JP locale
@mixin font-size($sizeValue) {
  font-size: ($sizeValue / 16) + rem;
}
```

이 코드를 도도 포인트 스토어앱에 작성했을 당시, 기존에 모든 글자 크기가 이미 픽셀 단위로 지정되어있던 상태였습니다. 그래서 모든 픽셀 단위를 rem으로 치환하기 위해 위와 같은 코드를 작성해서 넣었습니다. `$sizeValue`에 기존에 픽셀 단위로 정의되어있던 숫자 값을 적어 `@include`하면 됩니다. 예를 들면 `font-size: 16px;`로 정의되었던 줄을 `@include font-size(16);`로 바꾸는 것이죠. `$sizeValue`로 정의된 값은 `font-size: (0.125 / 2 * $sizeValue) + rem;` 코드 에서 rem 단위로 치환됩니다. 

`@include font-size(16);`로 적었다면 한국어 환경에서는 `$base-font-size` 변수로 `font-size`속성을 100%로 정의했기 때문에 1rem, 16px로 보이게 동작합니다. 일본어 환경에서는 `$base-font-size` 변수에 0.9를 곱해 `font-size`속성이 90%로 정의되었으므로 1rem, 14.4px로 보이게 동작합니다. 이렇게 해서 일본어 환경에서 믹스인을 사용해 정의해준 모든 글자 크기는 90%로 보이게 동작하고, 한국어 환경에서는 100%로 각 환경에 따라 다른 글자 크기로 보여줄 수 있도록 코드 작성이 완료되었습니다.

예제로 쓰인 방법을 간단히 정리하자면 아래와 같습니다.

1. `<html>` 요소에 언어 속성을 추가합니다.
2. 일본어 환경일때만 의사 클래스로 선택해 `<html>`의 글자 크기를 90%로 지정합니다.
3. `@mixin`을 활용해 모든 글자 크기단위를 px에서 rem으로 치환합니다.

# 결과

<figure>
  <img src="/images/2016-06-03/store-app-KR-JP-bug.png"
     style="margin-right:auto; margin-left:auto;" />
  <figcaption>
    한국어와 일본어 환경에 다른 타이포그래피 속성을 정의하지 않은 도도 포인트 스토어앱 화면
  </figcaption>
</figure>

<figure>
  <img src="/images/2016-06-03/store-app-KR-JP-normal.png"
    style="margin-right:auto; margin-left:auto;" />
  <figcaption>
    위의 예제를 활용해 다국어 타이포그래피 설정을 적용한 한국어와 일본어 환경에서의 도도 포인트 스토어앱 화면
  </figcaption>
</figure>

이 글에서 설명한 예제를 활용해 다국어 타이포그래피 설정을 적용한 일본어와 한국어 환경에서의 도도 포인트 스토어앱 화면입니다. 이제 글자가 한정된 너비의 컴포넌트 안에서 넘치지 않고 각 언어 환경에서 완성도 있게 보이는 것을 확인 할 수 있습니다.

# 응용
이 예제에서 쓰인 방법을 언어별 반응형 타이포그래피로 부를 수 있을 것입니다. 보통 rem과 퍼센트 단위를 이용해 CSS 속성을 각각 다르게 적용하는 예제는 반응형 웹사이트의 타이포그래피를 정의하는 방법으로 알려졌습니다. 이 글에서 설명한 예시는 반응형 타이포그래피를 각 언어 환경에 적용한 예입니다. 반응형 웹사이트의 타이포그래피 설정을 응용한 것입니다. 이 방법을 이해하셨다면, rem과 퍼센트 단위를 이용한 반응형 타이포그래피도 쉽게 익힐 수 있습니다.

<figure>
  <img src="/images/2016-06-03/responsive-typography-with-rem.gif"
     style="margin-right:auto; margin-left:auto;" />
  <figcaption>
    rem과 퍼센트 속성을 이용한 반응형 타이포그래피 예시
  </figcaption>
</figure>

위 gif 예시에서는 이 글에서 소개한 방법을 응용한 반응형 타이포그래피가 적용되어있습니다. 브라우저 너비 0과 768px 사이는 100%로 동작하고 1023px까지는 120%, 그리고 그 1024px 이상은 150%로 모든 글자 크기가 한번에 변합니다. 

rem과 퍼센트 단위를 이용한 CSS 반응형 타이포그래피를 더 자세히 소개하면 이 글의 주제에서 벗어나는 것 같아 위 gif의 코드, 이전에 작업한 [Codepen](http://codepen.io/young-hwa/pen/wGJzpy/)을 링크합니다.

# 마치며

이 작업을 하면서 배웠던 점을 끝으로 글을 줄이고자 합니다. 

#### 디자인 구현에 대한 다양하고 효과적인 방법론 
처음 이 이슈를 받았을 때 즉, 일본어와 한국어 환경 각각에서 잘 보여줄 수 있도록 스타일을 지정해야한다고 했을 당시에는 막연히 각각 스타일을 변경하면 되겠거니 싶었습니다. 일일히 스타일 시트 모두에 `:lang()` 의사 클래스로 일본어 환경을 선택해 스타일을 지정하려고 했습니다. (...)

이 작업을 하기 전까지 저는 기존에 기능 구현된 코드에서 스타일만 지정하는 방식으로 매우 앞단의 디자인만 손대는 편이었습니다. 이 작업 이후 `@mixin`이나 변수들을 더 본격적으로 써보았습니다. 또한 Sass의 여러 기능들에 대해 배우고 효과적인 구현을 위해 공부할 수 있는 계기가 되었습니다.

#### 끊임없는 반응형 타이포그래피에 대한 관심

2000년대 중반부터 해외 개발자 포럼과 블로그에서는 웹 타이포그래피와 반응형 웹에 관련한 포스트가 정말 많습니다. [웹 디자인의 95%는 타이포그래피](https://ia.net/know-how/the-web-is-all-about-typography-period)라는 분석부터 [웹 타이포그래피는 깨져있다](http://www.studiothick.com/essays/web-typography-is-broken/)는 주장, [미디어 쿼리에 대한 CSS 길이 단위 실험](http://zellwk.com/blog/media-query-units/) 같은 다양한 반응형 웹 관련 실험 등을 찾아보았습니다. 많은 이들이 쉬지않고 주제로 삼아온 반응형 웹과 웹사이트의 타이포그래피 활용에 대한 관심을 엿볼 수 있었습니다.

---

디자이너로서 프로그래밍을 배우는 과정은 마치 신대륙을 개척해 나가는 것 같습니다. 즐겁지만 어렵습니다. 프로그래머들이 너무 당연하게 알고있는 개념들이지만, 저에게는 생소할때가 정말 많습니다. 몇날 며칠이고 머리 싸매고 골똘히 생각해 이 난관을 헤쳐나가야 할 정도로 조난 당할 때도 몇번이고 있지만, 새로운 것을 발견하는 기쁨이 괴로움을 이깁니다. 생경한 어떠한 것을 마주하는 즐거움이 무척 크고 깊습니다.

이 글을 써 내려가면서 머릿속에서 명확하지 않은 개념들이 많다는 것을 알았고, 그만큼 더 공부할 수 있었습니다. 앞으로도 반응형 웹이나 타이포그래피 관련 연구를 소개하거나, 스포카에서 비슷한 작업을 진행한 결과물을 공유하는 포스팅으로 찾아뵙겠습니다.

[^1]: CSS `word-break`속성의 기본값은 `normal`이지만 CJK 와 라틴계열 언어가 다르게 동작합니다. `word-break: keep-all`을 사용했을 때 라틴알파벳과 CJK 문자 모두 불필요한 줄바꿈을 막아줍니다. 따라서 UI의 CSS 스타일를 정의할 때 `<body>`태그에 `word-break: keep-all;`을 기본으로 정의해두는 편이 좋습니다.
[^2]: 보통의 CJK 글꼴은 고정 너비를 지닌 고정폭 글꼴입니다. 조사한 바에 의하면 국문과 일문에서 기본 글꼴로 지정된 글꼴들의 너비가 다음과 같음을 발견할 수 있었습니다. 일본어 고딕 글꼴을 조사했을 때 메이리오는 2048pt, 히라기노 가쿠고딕은 1000pt, 스포카 한 산스 JP는 1024pt의 너비 값을 지닙니다. 한글 고딕 글꼴을 조사한 경우 나눔고딕은 892pt, 애플산돌고딕네오는 865pt, 스포카 한산스 KR은 1000pt의 너비 값을 지니고 있는 것을 확인했습니다.
[^3]: 이 글에 쓰인 예제는 리액트를 사용해 새로운 스토어앱 코드를 작성하기 전, 레거시 스토어앱에 김재석 CTO님이 작성해 두셨던 코드입니다. 리액트로 작성된 곳에 이 방식의 코드가 빠져있었고, 제가 CSS로 디자인을 구현하는 코드를 작성할 때 CTO님이 작성하셨던 것과 같은 방법으로 3.0 스토어앱에 추가하였습니다.
[^4]: CSS 각 속성은 기본 값이 정해져있습니다. Google 검색창에 &lsquo;속성의 이름&rsquo;과 &lsquo;initial value&rsquo;를 함께 검색하면 스타일이 정의되지 않았을 경우의 기본 값을 찾을 수 있습니다.
