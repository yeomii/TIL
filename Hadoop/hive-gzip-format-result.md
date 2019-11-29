# 하이브 처리 결과를 gzip 압축 포맷으로 저장하는 방법

* `CTAS`, `insert into`, `load data` 등의 구문을 사용하기 전에 아래와 같이 하이브 설정에 세팅해주면 된다.

```
SET mapred.output.compression.codec=org.apache.hadoop.io.compress.GzipCodec;
SET mapred.output.compression.type=BLOCK;
SET hive.exec.compress.output=true;
```