# Hive 테이블을 csv 포맷의 파일로 저장하기

## Hdfs 에 저장할 경우 
* `OpenCSVSerde` 를 사용하여 delimeter 나 quote 문자 처리 방식을 지정할 수 있다.
```sql
INSERT OVERWRITE DIRECTORY 'hdfs://{host}/{directory-path}' 
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
WITH SERDEPROPERTIES (
    "serialization.encoding"="UTF-8",
    "separatorChar" = ",",
    "quoteChar"     = "'",
    "escapeChar"    = "\\"
)  
STORED AS TEXTFILE
SELECT * FROM table-name ;
```

## 로컬 파일시스템에 저장할 경우
* 로컬 데스크탑에서 hive cli 로 서버에 접속한 경우
* `INSERT` 구문에 `LOCAL` 키워드를 추가해준다.
```sql
INSERT OVERWRITE LOCAL DIRECTORY '{directory-path}' 
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
WITH SERDEPROPERTIES (
    "serialization.encoding"="UTF-8",
    "separatorChar" = ",",
    "quoteChar"     = "'",
    "escapeChar"    = "\\"
)  
STORED AS TEXTFILE
SELECT * FROM table-name ;
```

## csv 파일을 엑셀로 열었을 때 인코딩이 깨진다면?
* 엑셀에서 세 파일을 연다
* 데이터 > 텍스트에서 > csv 파일 선택
* 가져오기 마법사에서 올바른 인코딩 포맷 선택 (보통 `UTF-8`)
* delimiter 를 `,` 로 지정
* 가져오기 완료

## References
* https://cwiki.apache.org/confluence/display/Hive/CSV+Serde
* https://stackoverflow.com/questions/6002256/is-it-possible-to-force-excel-recognize-utf-8-csv-files-automatically