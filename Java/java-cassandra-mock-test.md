# JVM 환경에서 cassandra mock test 

## cassandra-unit 적용 방법
* gradle 에 아래 의존성 추가
```
dependencies {
    ...
    testImplementation("org.cassandraunit:cassandra-unit:3.11.2.0")
}
```
* src/test/resources 에 목업할 테이블 생성 및 초기 데이터 추가 
    * @ src/test/resources/sample.cql

```sql
CREATE TABLE sample_table (
    id text,
    name text,
    PRIMARY KEY (id)
)
```

* 테스트 코드의 셋업 단계에서 목업 서버를 띄워주기
```kotlin
class TestClass {
    private var session: Session? = null

    @Before
    fun setUp() {
        System.setProperty("cassandra.native_transport_port", "9042")
        EmbeddedCassandraServerHelper.startEmbeddedCassandra()
        session = EmbeddedCassandraServerHelper.getSession()
        CQLDataLoader(session).load(ClassPathCQLDataSet("sample.cql", "sample_keyspace"))
    }

    @Test
    fun test() {
        println(session?.executeAsync("SELECT * FROM sample_keyspace.sample_table"))
    }
}
```

## references
http://www.scassandra.org/
https://github.com/scassandra/scassandra-server
https://scassandra-docs.readthedocs.io/en/latest/

https://github.com/jsevellec/cassandra-unit
https://github.com/jsevellec/cassandra-unit-examples