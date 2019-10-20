# Elasticsearch 
* 엘라스틱서치는 분산된 문서 저장소 + 검색 및 분석 엔진으로, 모든 타입의 데이터에 대해서 실시간 검색과 분석 기능을 제공한다.

## Getting Started
### macOS 로컬 환경에서 3개 노드로 이루어진 클러스터 띄우기
1. 바이너리 다운로드
```bash
$ curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.4.0-darwin-x86_64.tar.gz
$ tar -xvf elasticsearch-7.4.0-darwin-x86_64.tar.gz
```

2. 데이터, 로그 경로를 다르게 하여 3개 노드 띄우기 (데몬으로 띄우려면 -d 옵션 추가)
```bash
$ ~/bin/elasticsearch-7.4.0/bin/elasticsearch -Epath.data=~/es-data/data/data1 -Epath.logs=~/es-data/logs/log1
$ ~/bin/elasticsearch-7.4.0/bin/elasticsearch -Epath.data=~/es-data/data/data2 -Epath.logs=~/es-data/logs/log2
$ ~/bin/elasticsearch-7.4.0/bin/elasticsearch -Epath.data=~/es-data/data/data3 -Epath.logs=~/es-data/logs/log3
```

3. REST API 를 이용해 3개 노드의 health check
```bash
$ curl -X GET "localhost:9200/_cat/health?v&pretty"
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1570942644 04:57:24  elasticsearch green           3         3      0   0    0    0        0             0                  -                100.0%
```

## Document API
* 아주 기본적인 usage 부터 정리
### 문서 저장하기
```
POST localhost:9200/sample-index/_doc/
PUT localhost:9200/sample-index/_doc/<_id>
```
* 문서 아이디를 정해주려면 PUT 메소드를 사용하도록 하자

### 문서 가져오기
```
GET localhost:9200/<index>/_doc/<_id>
HEAD localhost:9200/<index>/_doc/<_id>
```
* HEAD 는 해당 문서가 존재하는지 확인하기 위해 사용한다.

### 문서 삭제하기
```
DELETE localhost:9200/<index>/_doc/<_id>
```

### 문서 업데이트하기
```
POST localhost:9200/<index>/_update/<_id>
```
* uri 의 `_update` 주의
* 문서의 업데이트를 스크립트화 할 수 있다.
* 스크립트는 업데이트, 삭제, 혹은 수정을 스킵할 수도 있다.
* 위 api 는 문서의 일부를 수정할수 있도록 해주는데, 기존 문서를 완전히 바꾸고 싶다면 `index API` 사용을 권장한다.