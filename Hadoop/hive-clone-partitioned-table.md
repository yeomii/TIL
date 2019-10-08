# Hive 에서 partition 된 테이블 복제하기

1. 테이블 스키마를 복제한다.
```hql
CREATE TABLE my_table_backup LIKE my_table;
```

2. dynamic partitioning 을 활성화하고 기존 테이블의 데이터를 새 테이블로 복제한다.
```hql
-- 필요할경우 파티션 수 설정 조정
SET hive.exec.max.dynamic.partitions = 1000;
SET hive.exec.max.dynamic.partitions.pernode = 1000;

SET hive.exec.dynamic.partition = true;
SET hive.exec.dynamic.partition.mode = nonstrict;

insert overwrite table my_table_backup partition (column_name)
select * from my_table;
```
