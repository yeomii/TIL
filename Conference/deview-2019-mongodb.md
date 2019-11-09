# MongoDB

* https://deview.kr/2019/schedule/298

## 네이버 검색에서 몽고DB
* 40여개 주제 250여개 컬렉션으로 서비스
* 통합검색 전체 질의어 중 30% 대응

## Why mongo db?
* fast + scalable + ha
* 네이버 통검의 1초 rule
* api 플랫폼 평균 응답속도 10ms 미만
* 서버당 초당 6천건 수준 / db 쿼리로는 초당 만 건 이상

## Mongo DB 속도 올리기 - index
* 컬렉션 당 최대 64개의 인덱스만 생성 가능
* 너무 많은 인덱스 추가시 side effect 발생
* frequent swap
* write performance 감소
* index prefix 를 사용하자
* 멀티 소팅
    * compound 인덱스의 경우 소팅 방향이 중요하다.

## Mongo DB 속도 올리기 - ^index
* 하나의 컬렉션을 여러개로 나누자
    * 하나의 컬렉션에 너무 많은 문서가 있을 경우, 인덱스 사이가 증가하고 인덱스 필드의 cardinality 가 낮아질 수 있다.
    * query processor 가 중복되는 인덱스 key 를 룩업하는 것을 방지해야 함.
* 쓰레드를 이용해 대량의 document 를 upsert
* mongo db 4.0 이전에는 non-blocking secondary read 기능 없었음
    * 업그레이드 하기

## 미운 index
* 쿼리가 엉뚱한 인덱스를 타기도 함
* 몽고 db 가 최선의 쿼리 플랜을 찾는 방법
    * 이전에 실행한 쿼리 플랜을 캐싱해놓고, (같은 query shape 일 경우)
      캐싱된 쿼리 플랜이 없다면 가능한 모든 쿼리 플랜들을 조회해서 첫 배치 (101) 를 가장 좋은 성능으로 가져오는 플랜을 다시 캐싱한다.
    * 성능이 너무 안좋아지면 위 작업을 반복
* 엉뚱한 인덱스를 타는 이유는, 첫번째 101 개를 가져오는 성능을 테스트 했을 때 모든 데이터에 대해서는 성능이 안 좋게 나올 수 있는 인덱스가 성능이 가장 잘 나올 수 있기 때문이다.
* 만약에 동점이 나올 경우, in-memorysort를 하지 않아도 되는 쿼리 플랜을 선택함
* hint 이용 vs 부적절한 인덱스 지우기
    * hint -> 바로 장애 발생했음, 쿼리에 사용하는 값에 따라 최적의 쿼리 플랜이 달라질 수 있음
    * 인덱스 삭제 -> 데이터에 따라 더 효율적인 인덱스가 생기면 자동 대응, 그러나 삭제로 인해 영향받는 다른 쿼리가 생길 수 있음

* no pain, no gain