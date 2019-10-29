# deview 2019 - vlive

* https://deview.kr/2019/schedule/306

* 미션 - 셀럽과 팬의 체감거리 줄이기

## keyword 
* live
    * 실시간성 보장이 중요
* global
    * 천만재생
* traffic
    * 실시간으로 40배 증가

## 실시간 메시지
* 방송 시작 알람이 중요
* 기존 배치시스템
    * 100만-200만 -> 1분이내
    * 1000만 -> 5-10분
* 신규
    * rabbitMQ -> 비동기 worker pool
* 중복 수신자 처리
    * 기존은 중복제거를 한번에 하고 발송
    * 신규는 가장 많은 그룹을 먼저 발송하고 다른 그룹 발송은 중복제거를 하고 발송

## 장애 고립화 . failover
* circuit breaker 적용되어있음

## global
* 국내 / 해외 it 환경 차이
    * 국내 - 4ms
    * 해외 - 국내의 수십배가 되기도 함
    * latency
    * bandwidth
    * 증설 소요시간
    * 테스트
* 송출 회선 타입 최적화
    * main / backup 회선 이중 사용
    * leased line
* saudi 1011
    * 지난 10월 사우디에서 bts 라이브 당시 사용
    * 유선을 사용하지 않고 위성으로 트래픽 전달
    * backup 도 위성 방송 방식을 사용 (2개 준비)
    * 위성은 날씨에 따라 품질에 영향을 받음
* global pop
    * naver mbp
    * 해외에 위치한 global interlocker
    * apache

## traffic
* 변화량이 매우 크다는 것이 특징
* 트래픽 현황 파악
    * 기존에는 access log 로 파악
    * 서버가 늘어나면서 pssh 를 사용하기도함
    * as-is 는 ES 에 액세스로그를 모두 저장
    * 그러나 색인속도가 로그 쌓이는 속도를 따라가지 못했음
    * 큐를 추가해서 병렬로 처리해서 해결 (???)
* 다중캐시
    * 1차 gpop - 2차 encache - 3차 arcus - 최종 mysql
* cdn routing path 최적화
    * 공용네트워크 구간 과부하로 재생 품질 저하
    * 네트워크 구간 트래픽을 모니터링하고 우회 경로 마련
* throttling
    * 호출 주기를 5초당 1회 -> 10초당 1회로 제한할 수 있도록 구현

## 실감형 컨텐츠의 기술 비전
* 한계
    * 화려한 영상에서는 여전히 블럭화 현상 발생
    * 리액션 채널은 채팅뿐
    * 평면으로만 전달되는 컨텐츠
* Being There 기술
    * 공연장 자체에 있는 것 같이 전달하는 기술
    * 3요소 정의
        * high resolution
            * 4k, 8k video
            * immersive sound
        * expended screen
            * xr : vr, ar, mr
            * next vision tech
        * interactive
            * unity, unreal
            * social & connected
            * multi interface
* 분산되어 제공되어 사용자 체감 만족도는 그리 높지 않았음
* 분산된 기술을 모으고자 vlive vr 서비스를 오픈
    * vr contents
        * 360 영상 신기하긴 하지만 180, 3d 컨텐츠가 더 효과적일 수 있음
    * vr platforms
        * infra 는 기존것 사용
        * unity 기반
        * 하드웨어는 oculus, steam 등등
        * 단말 최적화보다는 단말에 들어간 칩셋 기준으로 최적화 작업
    * vr infra
        * 고효율 압축코덱 적용
        * 1995 mpeg-2 -> 2005 avc / h.264 -> 2015 hevc / h.265
    * binaural sound 2020 적용 예정
    * vr interactive
        * 시나리오 분기구조
        * 모션캡쳐 / 햅틱
        * 아바타
* 실감형 기술 = balance 의 문제
    * 기존 기술 가공 융합
    * 현재 인프라 환경에서 효율적으로 구현
    * 컨텐츠 컨셉 / 타겟에 맞는 제작, 최적화