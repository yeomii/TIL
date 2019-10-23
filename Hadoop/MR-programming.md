# MR Programming

* 개발 절차
    * 맵 & 리듀스 함수 구현
    * 단위테스트 작성
    * 잡을 실행하는 드라이버 프로그램 작성
    * 데이터셋의 일부로 IDE 에서 테스트
    * 클러스터에서 실행
* 성능 개선
    * 표준검사
    * 태스크 프로파일링 수행
    * 하둡에서 제공하는 훅을 이용

## 6.1 환경설정 API
* Configuration 클래스
    * 환경설정 속성과 값의 집합
    * 리소스라 불리는 xml 파일로부터 속성 정보를 읽음
* 여러 개의 리소스를 사용하여 환경설정 할 수 있으며 나중에 추가된 리소스의 속성이 이전 속성을 오버라이드
* -Dproperty=value 라는 JVM 인자를 사용하여 속성을 오버라이드 가능

## 6.2 개발환경 설정하기
gradle 로 간단하게 설정
```gradle
ext {
    hadoop_version = "2.6.0"
}
dependencies {
    compile "org.apache.hadoop:hadoop-client:${hadoop_version}"

    # 단위테스트
    testCompile group: 'junit', name: 'junit', version: '4.12'
    # 맵리듀스 테스트 -> deprecated
    testCompile "org.apache.mrunit:mrunit"
    # 단일 JVM 으로 실행되는 하둡 클러스터에서 테스트할때 유용한 미니 클러스터를 포함함
    testCompile "org.apache.hadoop:hadoop-minicluster:${hadoop_version}"
}
```
* https://mrunit.apache.org/index.html
    * 2016/04/30 - APACHE MRUNIT HAS BEEN RETIRED.

* 환경 설정 파일 관리하기
    * 로컬모드 - hadoop-local.xml
        * 기본 파일시스템과 맵리듀스 잡을 실행하는 로컬 프레임워크에 적합한 하둡 설정
        * mapreduce.framework.name 속성값이 local
    * 의사분산 모드 - hadoop-localhost.xml
        * 로컬에서 작동하는 네임노드ㄹ와 YARN 리소스 매니저의 위치 설정
        * mapreduce.framework.name 속성값이 yarn
    * 완전분산 모드 - hadoop-cluster.xml
        * 클러스터의 네임노드와 YARN 리소스 매니저의 주소를 가진다.
        * 의사분산모드에서 localhost 를 실제 네임노드와 리소스노드의 주소로 바꾼것
* 사용자 인증 설정
    * 하둡은 클라이언트에서 whoami 로 사용자를 알아내고 HDFS 접근권한을 확인
    * 클라이언트와 하둡 사용자 계정 이름이 다르다면 환경변수에 HADOOP_USER_NAME 을 추가하자
    * 기본적으로 시스템 내에 인증기능을 제공하지 않고, 커버로스 인증 사용법은 10.4절 참조하기
* GenericOptionsParser, Tool, ToolRunner
```
# JVM 변수로 속성 오버라이딩
$ hadoop jar build/libs/com.mia.test.hadoop-1.0-SNAPSHOT.jar  com.mia.test.hadoop.ConfigurationPrinter | grep color
color=yellow
$ hadoop jar build/libs/com.mia.test.hadoop-1.0-SNAPSHOT.jar  com.mia.test.hadoop.ConfigurationPrinter -D color=green | grep color
color=green

# config 사용 
$ hadoop jar build/libs/com.mia.test.hadoop-1.0-SNAPSHOT.jar  com.mia.test.hadoop.ConfigurationPrinter | grep FS
fs.defaultFS=hdfs://hadoop-cm-test01.pg1.krane.9rum.cc:8020
$ hadoop --config ~/conf/hadoop-conf-bcr-ht jar build/libs/com.mia.test.hadoop-1.0-SNAPSHOT.jar  com.mia.test.hadoop.ConfigurationPrinter | grep FSß
fs.defaultFS=hdfs://bdr-ht-m01.dakao.io:8020
```

* import 할 때 mapred 말고 mapreduce 로 네이밍된 패키지를 import 하기

* mini cluster 사용 [CLI Mini Cluster](http://hadoop.apache.org/docs/r2.6.5/hadoop-project-dist/hadoop-common/CLIMiniCluster.html)
```
$ bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.6.5-tests.jar minicluster -rmport RM_PORT -jhsport JHS_PORT
```

