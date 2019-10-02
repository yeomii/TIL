# Impala 에서 Parquet format 을 사용하는 테이블의 스키마 변경시 주의사항

* [Schema Evolution for Parquet Tables (CDH 문서)](https://docs.cloudera.com/documentation/enterprise/5-12-x/topics/impala_alter_table.html)

impala 에서 테이블의 스키마를 변경할 때는 `ALTER TABLE ... REPLACE COLUMNS` 또는 `ALTER TABLE ... CHANGE` 구문을 사용하여 변경할 수 있다. ([SQL syntax](https://docs.cloudera.com/documentation/enterprise/5-12-x/topics/impala_alter_table.html))

위 구문은 테이블의 메타데이터만 변경하고, 테이블의 데이터가 되는 파일은 변경하지 않기 때문에 Decimal 타입의 precision 을 변경하거나 Bianry 타입을 변경할 경우 읽기 쿼리 에러가 발생될 수 있다.

[Decimal precision 변경시 읽기 쿼리 에러 발생 관련 impala 이슈](https://issues.apache.org/jira/browse/IMPALA-7087)

위 이슈는 impala 에서 발생하고 Hive 에서는 발생하지 않으므로 주의하도록 하자.

추가로, 해당 column 의 값만 update 하려면 kudu 테이블을 사용하거나 orc 포맷의 테이블이어야 한다.
