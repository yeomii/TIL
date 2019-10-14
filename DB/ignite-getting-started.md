# Ignite Getting Started 따라하기

* [Getting Started](https://apacheignite.readme.io/docs/getting-started)

## 첫 클러스터 구성
* binary release 다운로드
    * [링크](https://ignite.apache.org/download.cgi#binaries)

* unzip 후 IGNITE_HOME 환경 변수를 지정하고 Path 에 설정한다
    * 이 때 경로 맨 마지막 글자가 / 로 끝나지 않도록 한다.
    * ~/.bashrc 설정

        export IGNITE_HOME=~/apache-ignite-fabric-2.3.0-bin
        export PATH=$PATH::$IGNITE_HOME/bin

* ignite.sh 로 노드를 시작할 수 있다

    $ ignite.sh

* 위 명령어로 시작할 때는 $IGNITE_HOME/config/default-config.xml 설정을 사용
* 다른 설정으로 노드를 띄우고 싶다면 아래 명령어 실행

    $ ignite.sh custom-ignite-config.xml

* config 를 interactive 하게 선택하고싶다면 아래 명령어 실행
* 파일 목록은 IGNITE_HOME 에 있는 설정 파일들이 출력된다

    $ ignite.sh -i

## SQL CLI 사용하기

* sqlline.sh 로 SQL 쿼리를 날릴 수 있는 cli 를 시작할 수 있다.

    $ ./sqlline.sh --color=true --verbose=true -u jdbc:ignite:thin://127.0.0.1/

* 자세한 사용법은 [공식 문서](https://apacheignite-sql.readme.io/docs/sqlline)를 참고.
