# hive explode 사용 예제

* explode 함수는 특정 필드 또는 값이 array 또는 map 타입일 때, 해당 값을 여러개의 row 로 만들어 주기 위해 사용된다.

```
select * from (
    select 'test' as test
) t
lateral view explode(array(1, 2, 3)) a as number;
```

* 위 쿼리는 아래와 같은 결과값을 낸다

   | t.test | a.number
---|:------:|---------:
`1`| test   | 1
`2`| test   | 2
`3`| test   | 3
