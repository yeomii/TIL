# HTTP Header size 제한 기본값

* 웹 서버에서 http header 최대 크기를 제한할 수 있는데, 웹 서버 엔진별 기본 값은 다음과 같다.

* Apache 
    * 1.3, 2.0, 2.2, 2.3: 8190 Bytes (for each header field)
* IIS:
    * 4.0: 2097152 Bytes (for the request line plus header fields)
    * 5.0: 131072 Bytes, 16384 Bytes with Windows 2000 Service Pack 4 (for the request line plus header fields)
    * 6.0: 16384 Bytes (for each header fields)
* Tomcat:
    * 5.5.x/6.0.x: 49152 Bytes (for the request line plus header fields)
    * 7.0.x: 8190 Bytes (for the request line plus header fields)
* nginx
    * 4K - 8K

* 헤더 크기가 제한 값을 초과할 경우, `413 Entity Too Large` 또는 `4xx` 에러코드를 내려 줄 것이다 