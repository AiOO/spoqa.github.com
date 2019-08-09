---
layout: entry
title: "파이썬 의존성 관리자 Poetry 사용기"
author: 이강욱
author-email: rusty@spoqa.com
description: Poetry를 사용해본 경험을 소개하고 기존의 의존성 관리자 pip와 비교해봅니다.
publish: true
---

안녕하세요! 스포카 풀 스택 프로그래머 이강욱입니다.

여러분은 의존성 관리를 좋아하시나요? 의존성 관리자, Dependency Manager란
무엇일까요? 의존성 관리자 라는 단어가 생소하게 들리실 수 있지만, 여러분은
아마 이미 의존성 관리자를 사용하고 계실 겁니다. 패키지 관리자라는 단어가
더 익숙하실 수도 있습니다. pip, Yarn, npm, Cargo 등이 패키지 관리자임과 동시에
의존성 관리자에 해당합니다.

패키지 관리자가 설치하고 싶은 패키지를 공식 저장소나, 개인 저장소 등에서 찾아
다운로드하고, 설치하는 역할을 한다면, 의존성 관리자는 패키지가 요구하는 다른
패키지, 그리고 그 다른 패키지가 요구하는 또 다른 패키지를 거슬러 올라가며
설치해야 할 패키지의 목록(Dependency Tree)을 구성하는 일을 합니다.

대부분의 언어에는 보통 패키지 관리자와 의존성 관리자의 역할을 동시에 수행하는
툴이 있습니다. 그렇다고 언어에 한 가지의 의존성 관리자만 있는 것은 아닙니다.
JavaScript만 해도 npm과 Yarn이라는 두 의존성 관리자가 경쟁하고 있습니다.

저는 파이썬의 의존성 관리자에 관해 이야기해보려고 합니다. 파이썬도 여러 가지
의존성 관리자가 경쟁하고 있는데, 일단 파이썬의 공식 의존성 관리자인 pip가
있습니다. 그리고 pip와 virtualenv의 기능을 동시에 수행하는 Pipenv도 있습니다.
하지만 오늘 저는 파이썬의 (비교적) 신생 의존성 관리자, [Poetry]에 대해
소개드리겠습니다.


## Why not pip?

위에서 말했듯이 파이썬에는 이미 pip가 있습니다. 그런데 왜 다른 의존성 관리자를
쓰는 걸까요? pip의 문제점을 몇 가지 알아보겠습니다.

### Dependency Resolving

pip가 하지 못하는 것을 바로 알아보는 방법이 있습니다. 바로 1.4.0 버전의
`oslo.utils` 라는 패키지를 설치해보면 알 수 있습니다.

```bash
$ pip install oslo.utils==1.4.0
Looking in indexes: https://pypi.org/simple
Collecting oslo.utils==1.4.0
Collecting oslo.i18n>=1.3.0 (from oslo.utils==1.4.0)
Collecting Babel>=1.3 (from oslo.utils==1.4.0)
Collecting netaddr>=0.7.12 (from oslo.utils==1.4.0)
Collecting pbr!=0.7,<1.0,>=0.6 (from oslo.utils==1.4.0)
Collecting netifaces>=0.10.4 (from oslo.utils==1.4.0)
Collecting iso8601>=0.1.9 (from oslo.utils==1.4.0)
Collecting six>=1.9.0 (from oslo.utils==1.4.0)
Collecting pytz>=2015.7 (from Babel>=1.3->oslo.utils==1.4.0)
Requirement already satisfied: pip in /.../site-packages (from pbr!=0.7,<1.0,>=0.6->oslo.utils==1.4.0) (19.0.3)
oslo-i18n 3.23.1 has requirement pbr!=2.1.0,>=2.0.0, but you'll have pbr 0.11.1 which is incompatible.
Installing collected packages: six, pytz, Babel, pbr, oslo.i18n, netaddr, netifaces, iso8601, oslo.utils
  Running setup.py install for netifaces ... done
Successfully installed Babel-2.7.0 iso8601-0.1.12 netaddr-0.7.19 netifaces-0.10.9 oslo.i18n-3.23.1 oslo.utils-1.4.0 pbr-0.11.1 pytz-2019.2 six-1.12.0

$ pip check
oslo-i18n 3.23.1 has requirement pbr!=2.1.0,>=2.0.0, but you have pbr 0.11.1.

```

1.4.0 버전의 `oslo.utils`는 다음과 같은 패키지를 요구합니다.

- `pbr` (>=0.6,!=0.7,<1.0)
- `oslo.i18n` (>=1.3.0)
- (나머지는 생략)

이때 pip는 `pbr`의 (이 글을 쓰는 시점에서) 최신 버전인 0.11.1 버전을
설치합니다. 그러고 나서 `oslo.i18n`의 최신 버전인 3.23.1을 설치합니다. 그런데
`oslo.i18n` 3.23.1은 2.0.0 이상의 `pbr`을 요구합니다. 그런데 우리는 이미
0.11.1 버전의 `pbr`을 설치했습니다. Dependency Resolving에 실패한 것입니다.

### Dependency Locking

pip는 락 파일이 없습니다. 사용자가 직접 `requirements.txt`를 잘 작성하는
수밖에 없습니다. `requirements.txt`를 작성해도 패키지 버전은 결정적이지
못하고, 사람이 손으로 작성해야 하다 보니 실수하는 순간 개발 환경에서는 잘 돌던
코드가 배포하면 작동하지 않는 일을 겪을 수 있습니다.

### Virtual Environment

pip는 항상 전역에 모든 패키지를 설치합니다. 어떤 프로젝트에서 `flask`
0.12 버전을 설치해서 행복하게 개발하고 있었는데, 다른 프로젝트에서 `flask` 1.1
버전을 설치하면 덮어씌워져 버리는 것입니다.

물론 파이썬에서는 virtualenv라는 훌륭한 툴로 환경을 분리할 수 있지만,
virtualenv랑 pip가 따로 노는 것이 조금 불편합니다.


## Behold, the mighty Poetry has come

[Poetry]는 Pipenv와 비슷한 기능을 제공하는, pip와 virtualenv를 동시에 사용할
수 있게 해주는 패키지 관리자이자 의존성 관리자입니다. pip와는 다르게
`poetry.lock`이라는 락 파일을 생성하며, Pipenv와 비교하여도 Dependency
Resolving 에서 강점이 있습니다.

Poetry는 매우 간편한 설치 과정과 명령줄 인터페이스를 제공합니다.
한번 Poetry를 설치하고, 위에서 예시로 든 `oslo.utils`를 설치해볼까요?

#### Poetry Installation

```bash
$ wget -q https://raw.githubusercontent.com/sdispater/poetry/master/get-poetry.py

$ python get-poetry.py --preview
Retrieving Poetry metadata

Before we start, please answer the following questions.
You may simply press the Enter key to leave unchanged.
Modify PATH variable? ([y]/n)

# Welcome to Poetry!

This will download and install the latest version of Poetry,
a dependency and package manager for Python.

It will add the `poetry` command to Poetry's bin directory, located at:

$HOME/.poetry/bin

This path will then be added to your `PATH` environment variable by
modifying the profile file located at:

$HOME/.profile

You can uninstall at any time with `poetry self:uninstall`,
or by executing this script with the --uninstall option,
and these changes will be reverted.

Installing version: 1.0.0a5
  - Downloading poetry-1.0.0a5-linux.tar.gz (22.30MB)

Poetry (1.0.0a5) is installed now. Great!

To get started you need Poetry's bin directory ($HOME/.poetry/bin) in your `PATH`
environment variable. Next time you log in this will be done
automatically.

To configure your current shell run `source $HOME/.poetry/env`

```

#### Project Initialization

```bash
$ poetry init

This command will guide you through creating your pyproject.toml config.

Package name [my-project]:  my-project
Version [0.1.0]:
Description []:
Author [rusty <rusty@spoqa.com>, n to skip]:
License []:
Compatible Python versions [^3.7]:

Would you like to define your main dependencies interactively? (yes/no) [yes] no
Would you like to define your dev dependencies (require-dev) interactively (yes/no) [yes] no
Generated file

[tool.poetry]
name = "my-project"
version = "0.1.0"
description = ""
authors = ["rusty <rusty@spoqa.com>"]

[tool.poetry.dependencies]
python = "^3.7"

[tool.poetry.dev-dependencies]

[build-system]
requires = ["poetry>=0.12"]
build-backend = "poetry.masonry.api"


Do you confirm generation? (yes/no) [yes]

$ ls
pyproject.toml

```

#### Package Installation

```bash
$ poetry add oslo.utils==1.4.0

Updating dependencies
Resolving dependencies... (24.0s)

Writing lock file


Package operations: 8 installs, 0 updates, 0 removals

  - Installing pytz (2019.2)
  - Installing babel (2.7.0)
  - Installing pbr (0.11.1)
  - Installing iso8601 (0.1.12)
  - Installing netaddr (0.7.19)
  - Installing netifaces (0.10.9)
  - Installing oslo.i18n (2.1.0)
  - Installing oslo.utils (1.4.0)

$ ls
poetry.lock  pyproject.toml

```

생성된 `pyprojects.toml` 파일에는 프로젝트의 메타데이터가, `poetry.lock`
파일에는 설치된 패키지들의 버전과 hash가 저장되어있습니다.

Poetry는 낮은 버전의 `oslo.i18n`을 설치함으로써 위의 문제를 해결합니다.
`oslo.utils`가 요구하는 `pbr`의 버전과 호환되는 `oslo.i18n`을 찾아서 버전을
거슬러 올라가, `oslo.i18n` 2.1.0. 버전을 설치하는 것입니다.


## Epic Poetry

이제 우리는 Poetry로 락 파일도 생성할 수 있고, 복잡한 의존성 문제도 해결할
수 있습니다. 그럼 "모든 pip와 Pipenv를 당장 Poetry로 교체합시다!!"라고 주장할
수도 있지만... 아쉽게도 Poetry는 아직 신생 툴이고, 많은 불안정함을 가지고
있습니다.

이 글도 사실 Poetry _소개_ 가 아닙니다. _사용기_ 입니다. 제가 Poetry를
사용하면서 겪은 황당한 문제와 그 해결 과정을 공유해보려고 합니다.

### AttributeError

> 관련 이슈: <https://github.com/sdispater/poetry/issues/1163>

처음 프로젝트에 Poetry를 도입하기 위해 이것저것 테스트해본 후에, 괜찮겠다
싶어서 스포카의 프로그래머 워크샵에서 Poetry 소개 발표를 한 후, 일부
프로젝트에 Poetry를 도입해보았습니다.

그래서 기쁜 마음으로 프로젝트의 `requirements.txt`를 날려버리고
`pyproject.toml` 파일을 꼼꼼히 작성한 후, `poetry install`을 친 순간..
에러가 났습니다.

에러의 원인은 사실 대단치 않았습니다. 스포카의 private repository에만 존재하는
패키지를 설치하려고 해서, 당연히 Poetry는 그 패키지를 찾을 수 없으니 에러가
난 것이었습니다. 물론 Poetry는 private repository에서 설치하는 기능도
지원합니다.

먼저 `pyproject.toml` 파일에 다음과 같은 줄을 추가합니다:

```toml
[[tool.poetry.source]]
name = "spoqa-private-repository"
url = "https://some.private.repository.com/"
```

그리고 다음과 같은 명령어로 인증 토큰을 등록합니다:

```bash
$ poetry config repo.spoqa-repository https://some.private.repository.com/
$ poetry config http-basic.spoqa-repository <username> <password>
```

이렇게 하면 Poetry는 private repository에서 패키지를 찾아 설치할 수 있습니다.
그래서 다시 `poetry install`을 친 순간.... 에러가 났습니다.

```bash
$ poetry install
Updating dependencies
Resolving dependencies... (0.6s)

Writing lock file

Package operations: ...

(중략)

  - Installing some-private-package (0.1.0)

[AttributeError]
'Pool' object has no attribute 'default'

```

알 수 없는 에러입니다. 뭔가 찾을 수 없다는 것도 아니고, 설치에 실패한 것도
아니고, `AttributeError`라니? 코드에 문제가 있는 걸까요? 직접 Poetry의 코드를
뜯어보았습니다.

에러가 난 부분은
[`poetry/installation/pip_install.py L58`](https://github.com/sdispater/poetry/blob/0857f3252d9ea1927af4a346ed7a7c1e50ae10a3/poetry/installation/pip_installer.py#L58)
입니다:

```python
class PipInstaller(BaseInstaller):
    # ...

    def install(self, package, update=False):
        # ...

            args += ["--index-url", index_url]
            if self._pool.has_default():
                if repository.name != self._pool.default.name:  # HERE
                    args += ["--extra-index-url", self._pool.default.authenticated_url]

        # ...
```

`PipInstall` class의 `_pool` attribute의 타입은 `Pool` class 입니다. `Pool`
class에는 `default`라는 attribute가 없다는 것 같은데요..

[`poetry/repositories/pool.py`](https://github.com/sdispater/poetry/blob/0857f3252d9ea1927af4a346ed7a7c1e50ae10a3/poetry/repositories/pool.py)
파일을 확인해보면.. 정말 `Pool` class에는 `_default`라는 private attribute는
있지만, 아무리 찾아봐도 `default`라는 attribute는 존재하지 않습니다.

이것은 매우 심각한 문제입니다. PyPI 이외의 repository를 이용하려고 하면 무조건
발생하는 버그인데, private repository를 사용할 수 없다면 개발이 불가능합니다.
개발이 불가능하다면 Poetry를 사용하지 말아야 합니다.

저는 눈물을 흘리며 제가 제안한 Poetry를 다시 걷어내기로 했습니다.

<figure>
 <img src="/images/2019-08-09/remove-poetry-pr.png" style="margin: 0 auto;" />
</figure>

그리고 빠르게 [버그를 수정하는 PR](https://github.com/sdispater/poetry/pull/1193)
을 올렸습니다.

<figure>
 <img src="/images/2019-08-09/default-property-pr.png" style="margin: 0 auto;" />
</figure>

제가 올린 PR은 `_default` attribute와 `default` attribute 사이의 불일치 등
문제가 있어 바로 머지되진 못하고, 대신 maintainer께서
[다른 방법으로 해결한 커밋](https://github.com/sdispater/poetry/commit/6f4aa21ce65dd719cfc98951d207ff39cd1d94cf)
의 co-author로 추가해주셨습니다.

<figure>
 <img src="/images/2019-08-09/co-authored-commit.png" style="margin: 0 auto;" />
</figure>

제가 PR을 올리고 해결되기까지 시간이 조금 걸렸지만, 아무튼 1.0.0a5 버전에서
수정되었고, 걷어냈던 Poetry를 다시 도입하는 PR도 올릴 수 있었습니다.

<figure>
 <img src="/images/2019-08-09/make-poetry-great-again.png" style="margin: 0 auto;" />
</figure>

### FileNotFoundError

> 관련 이슈: <https://github.com/sdispater/poetry/issues/1179>

다시 즐겁게 Poetry를 도입한 후, 서비스를 배포하기 위해 `Dockerfile`을 작성하고
있었습니다.

제가 작성한 `Dockerfile`은 다음과 같았습니다:

```Dockerfile
FROM python:3.7.4-alpine

RUN apk --no-cache add postgresql-libs &&\
    apk --no-cache add --virtual .build-deps \
    build-base linux-headers postgresql-dev libffi-dev wget

RUN wget https://raw.githubusercontent.com/sdispater/poetry/master/get-poetry.py &&\
    python get-poetry.py --preview --yes &&\
    rm -f get-poetry.py

ENV PATH="/root/.poetry/bin:${PATH}"

ARG USERNAME
ENV USERNAME ${USERNAME}
ARG PASSWORD
ENV PASSWORD ${PASSWORD}

RUN poetry config settings.virtualenvs.create false &&\
    poetry config repo.spoqa-private-repository https://some.private.repository.com/ &&\
    poetry config http-basic.spoqa-private-repository ${USERNAME} ${PASSWORD}

WORKDIR /app

COPY pyproject.toml /app/
COPY poetry.lock /app/
RUN poetry install --no-dev

RUN apk --no-cache del --purge .build-deps

COPY . /app

ARG RELEASE
ENV RELEASE ${RELEASE}
```

그리고 `docker build .`를 친 순간.. 에러가 났습니다.

```bash
$ docker build .
...
Step 7/15 : RUN poetry config repo.spoqa-private-repository https://some.private.repository.com/ &&    poetry config http-basic.spoqa-private-repository ${USERNAME} ${PASSWORD}
 ---> Running in a6c8ea5e4173

[FileNotFoundError]
[Errno 2] No such file or directory: '/root/.config/pypoetry/config.toml'
The command '/bin/sh -c poetry config repo.spoqa-private-repository https://some.private.repository.com/ &&    poetry config http-basic.spoqa-private-repository ${USERNAME} ${PASSWORD}' returned a non-zero code: 1

```

`config.toml` 파일이 없다니요?
[Configuration 문서](https://github.com/sdispater/poetry/blob/master/docs/docs/configuration.md#L4)에서는
분명히 `config.toml` 이 없다면 새로 생성한다고 되어있었는데..? 이번에도 코드를
직접 열어보았습니다.

[`poetry/config.py L100`](https://github.com/sdispater/poetry/blob/442ff521146dc73477150110d60ee7f311e6780d/poetry/config.py#L100):

```python
class Config:
    # ...

    def dump(self):
        # ...

        if self._file.exists():
            fd = str(self._file)
        else:
            umask_original = os.umask(umask)
            try:
                fd = os.open(str(self._file), os.O_WRONLY | os.O_CREAT, mode)
            finally:
                os.umask(umask_original)

        # ...
```

코드를 보면, 권한 문제 때문에 조금 복잡하지만, 분명히 파일이 존재하지 않으면
새로 생성하게 되어있습니다. 무엇이 문제일까요? 제가 이런저런 고민을 한 결과,
깨달았습니다.

`config.toml` 파일은 생성하는데, 그 상위 폴더인 `$HOME/.config/pypoetry` 폴더를
생성하는 코드가 전혀 없었던 것입니다.

Poetry의 설정 파일 위치는 OS마다 다르지만, Linux에서는 기본값으로
`$HOME/.config/pypoetry/` 폴더를 사용합니다. 황당하게도 이 폴더가 없으면
에러가 발생했고, 특히 아무것도 없는 상태에서 시작하는 Docker image에서 이런
문제가 쉽게 발생했던 것입니다.

어려운 문제는 아니라서, 저는 또 급하게
[문제를 수정하는 PR](https://github.com/sdispater/poetry/pull/1248)을
올렸습니다.

<figure>
 <img src="/images/2019-08-09/config-directory-pr.png" style="margin: 0 auto;" />
</figure>

이번에도 이 PR은 머지되지 않고 (눈물),
[설정 동작의 구조 자체를 바꾸는 PR](https://github.com/sdispater/poetry/pull/1248)에서
한꺼번에 해결되었습니다.

아무튼 엄청 크리티컬한 문제는 아니기에, 버그가 수정된 버전이 릴리즈되기 전까지
저는 `Dockerfile`을 살짝 수정하여 사용하고 있습니다:

```Dockerfile
...


RUN mkdir -p $HOME/.config/pypoetry &&\
    poetry config settings.virtualenvs.create false &&\
    poetry config repo.spoqa-private-repository https://some.private.repository.com/ &&\
    poetry config http-basic.spoqa-private-repository ${USERNAME} ${PASSWORD}

...
```

### Zombies!

> 관련 이슈: <https://github.com/sdispater/poetry/issues/993>

위 `Dockerfile`에서, `poetry config settings.virtualenvs.create false`라는
설정이 있는데, 이 설정을 알아낸 것도 사실 삽질의 결과였습니다.

Poetry는 기본적으로 *무조건* virtual environment를 생성하여 패키지를
설치합니다. 이 동작을 바꾸는 것이 `settings.virtualenvs.create` 설정입니다.
저는 이 설정의 존재를 모르고, Docker image 내에서도 그냥 virtualenv를 생성한
다음에, `poetry run` 명령을 이용하려고 했습니다.

`poetry run` 명령은 해당 프로젝트의 virtualenv 내에서 인자로 주어진 명령을
실행시킵니다. `poetry run python`를 실행하면 해당 virtualenv의 `python`을 실행할 수
있고, `poetry run python run.py`와 같은 방식으로 어플리케이션을 실행시키도록 할
수 있습니다.

이 `poetry run` 명령에도 심각한 문제가 있었습니다. 처음에는 실행이 잘 되고,
동작도 잘하는 것처럼 보였지만... 문제는 `Ctrl-C`를 누를 때 일어났습니다.

Linux에서 어떤 프로그램이 실행 중일 때 Control 키와 C 키를 동시에 누르면 해당
프로세스에 SIGINT 시그널을 보냅니다. SIGINT는 interrupt signal이라는
뜻으로, 보통 프로세스를 종료시킬 때 사용됩니다.

그래서 `python run.py`로 실행시킨 어플리케이션도 `Ctrl-C`를 누르면 종료되고,
`poetry run python run.py`로 실행시킨 어플리케이션도 `Ctrl-C`를 누르면
종료됩니다. 하지만, _어떤_ 프로세스가 종료되는 걸까요?

`poetry run` 명령은 파이썬의 `subprocess.run` 함수를 이용해 자식 프로세스를
띄웁니다. 그리고 그 상태에서 `Ctrl-C` 를 누르면 부모 프로세스인 Poetry가
SIGINT를 받습니다. 그리고... 종료됩니다.

그렇습니다. 자식 프로세스는 그대로 둔 채, 부모 프로세스인 Poetry가 바로
종료되어버리는 것입니다. 원래 이런 자식 프로세스를 만드는 프로그램의 경우,
이런 시그널을 자식 프로세스에 전달해야 합니다. 하지만 Poetry는 그런 시그널
처리를 고려하지 못한 것입니다.

그래서 겉으로 보이는 Poetry는 종료되었지만, 사실 실제 어플리케이션은
백그라운드에서 계속 실행되어있을 수도 있고, 그것은 큰 문제를 낳을 수 있습니다.
이런 부모 프로세스가 사라져서 돌아다니는 프로세스를 _좀비_ 프로세스라고 합니다.

이것은 꽤 어려운 문제입니다. 해결하려면 Poetry가 모든 시그널을 자식
프로세스로 전파하게 만들거나, 아니면 자식 프로세스를 아예 만들지 않고 Poetry
프로세스를 아예 어플리케이션 프로세스로 교체하는 방법이 있습니다. Linux에서는
exec 시스템 콜을 이용하면 후자를 구현할 수 있습니다. 그래서 저는 후자를
선택해서, [문제를 수정하는 PR](https://github.com/sdispater/poetry/pull/1248)을
올렸습니다.

<figure>
 <img src="/images/2019-08-09/execvp-pr.png" style="margin: 0 auto;" />
</figure>

하지만 여러 가지 문제가 있었습니다. 먼저 Linux나 Mac에서는 exec 시스템 콜로
간단하게 해결되지만, 윈도우에서는 그렇지 않은 것입니다. 원인은 윈도우의
`CreateProcess` 시스템 콜 때문인데, 자세한 것은 생략하겠습니다.

그래도 어느 친절한 분이 윈도우에서도
[비슷한 동작을 만드는 방법](https://github.com/sdispater/poetry/pull/1236#issuecomment-514058388)을
소개해주셨고, 제 PR을 업데이트할 수 있었습니다.

<figure>
 <img src="/images/2019-08-09/execvp-help.png" style="margin: 0 auto;" />
</figure>

그렇지만 문제는 끝이 아닙니다. 원래 코드에서는 `sh`를 이용해서 실행하던 것을
execvp 시스템 콜로 바꿔서 생기는 동작 변화 등 하위 호환성 문제가 걱정됩니다.
아직 마땅한 해결책이 생각나지 않아서 PR은 그대로입니다.

물론 virtualenv를 생성하지 않고 전역에 패키지를 설치할 수 있는 법을 알게 돼서
`poetry run` 명령을 사용하지 않아도 `Dockerfile`을 작성하는 것이 가능해졌습니다.
그렇지만 누군가는 `poetry run` 명령을 유용하게 사용하고 싶을 것이고, 이 버그
때문에 Poetry 사용을 포기하게 된 사람이 있을지도 모릅니다. 그래서 저는 이
문제가 하루 빨리 해결되었으면 좋겠습니다.


## WE NEED YOU

Poetry는 훌륭한 툴입니다. 기존 pip나 Pipenv가 해결하지 못했던 문제를
해결해주고, 설치, 사용이 쉽습니다. 저는 Poetry에 큰 기대를 품고 있고, Poetry가
더욱 발전하고 안정적인 의존성 관리자가 되어 널리 사용되었으면 좋겠습니다.

그러나 현재의 Poetry는 아직 부족합니다. 버그가 많아 불안정하고, 기능도
부족합니다. 하지만 Poetry는 오픈 소스 프로젝트입니다. 그 말은 저도, 여러분도
언제든지 Poetry에 기여하고 발전시킬 수 있다는 것입니다. Poetry가 더 좋은 툴이
되기 위해서는 여러분의 도움이 필요합니다.

제 PR에 해결책을 제시해주셔도, 문서를 작성하셔도, 테스트 케이스를 추가하셔도,
문서를 한국어로 번역하는 것도 좋습니다. 저처럼 영어 실력이 형편없어도
괜찮습니다. Poetry는 여러분의 기여를 기다리고 있습니다. 다 함께 Poetry에
기여하고, 파이썬 생태계에 기여하고, 오픈 소스 생태계에 기여해봅시다!


[Poetry]: https://github.com/sdispater/poetry
