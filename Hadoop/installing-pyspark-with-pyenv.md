# pyenv 를 사용해서 pyspark 설치하기 (mac & zsh)

## pyenv 설치

    $ brew install pyenv
    $ echo 'eval "$(pyenv init -)"' >> ~/.zshrc

pyenv 로 원하는 버전의 python 을 설치한다

    $ pyenv install 3.6.5

## pyenv-virtualenv 설치

    $ brew install pyenv-virtualenv
    $ echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.zshrc

pyenv 로 설치한 버전의 독립된 개발환경을 virtualenv 로 만든다

    $ pyenv virtualenv 3.6.5 virtualenv-test

명령어로 만든 virtualenv 를 activate 시킨다. </br>
activate 가 되면 일반적으로 프롬프트에 active 한 virtualenv 명이 뜬다. </br>
deactivate 로 비활성화 시킬 수 있다.

    $ pyenv activate virtualenv-test
    (virtualenv-test) ... $ pyenv which python
    ~/.pyenv/versions/virtualenv-test/bin/python
    (virtualenv-test) ... $ deactivate

만약 콘솔에 active 한 virtualenv 가 표시되지 않는다면 zsh 테마의 설정을 변경해야 할 수도 있다.</br>
현재 zsh + powerlevel9k 테마를 사용하는데 virtualenv 가 표시되지 않아 ~/.zshrc 에 다음 설정을 추가했다.

    POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(context dir vcs virtualenv)

## pip 로 pyspark 설치

위에서 만든 virtualenv 가 active 한 상태에서 pip 로 설치한다.
설치 후에는 pyspark 로 실행한다.

    (virtualenv-test) ... $ pip install pyspark
    (virtualenv-test) ... $ pyspark
    ...
    Welcome to
       ____              __
      / __/__  ___ _____/ /__
     _\ \/ _ \/ _ `/ __/  '_/
    /__ / .__/\_,_/_/ /_/\_\   version 2.3.0
       /_/
    ... 
    >>>


## reference
* [Python 개발 환경 구축하기 - pyenv, virtualenv, autoenv](https://cjh5414.github.io/python-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD%EA%B5%AC%EC%B6%95/)