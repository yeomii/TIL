# Hive 의 map 과 struct 타입의 차이

* Map 과 Struct 는 Array 와 Union 과 함께 complex type 으로 불린다

## Map
* syntax
```
column_name MAP < primitive_type, type >

type ::= primitive_type | complex_type
```
* 예제
```sql
create TABLE map_demo
(
  country_id BIGINT,
  metrics MAP <STRING, BIGINT>,
);
```

## Struct
* syntax
```
column_name STRUCT < name : type [COMMENT 'comment_string'], ... >

type ::= primitive_type | complex_type
```
* 예제
```
CREATE TABLE struct_demo
(
  id BIGINT,
  name STRING,
  employee_info STRUCT < employer: STRING, id: BIGINT, address: STRING >,
);
```

## map 과 struct 의 차이
* 가장 큰 차이는 map 의 value 는 같은 타입이어야 한다는 것
* 또, struct 의 경우 struct 에 사용할 필드의 이름이 고정된다는 것 
* map 의 value 로 struct 가 사용될 수 있다
* struct 필드의 타입으로 struct 가 다시 사용될 수 있다


## TODO
* 내부 구현 방식도 정리해보기

## Reference
* https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types
* https://docs.cloudera.com/documentation/enterprise/5-9-x/topics/impala_complex_types.html