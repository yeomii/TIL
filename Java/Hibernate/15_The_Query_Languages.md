# 15 The query languages

* Java Persistence with Hibernate, 2nd Edition by rajeev 의 15장을 읽고 정리한 내용
* ( ~ 369p )

# 챕터 설명
- JPQL 과 criteria 쿼리 작성하기
- join 으로 데이터를 효율적으로 추출하기
- subselect 와 reporting query

- 이 챕터에서 JPA 에서 사용 가능한 query language 인 JPQL 과 criteria query API 를 다룰 예정
- select 문 위주로 살펴볼 예정
- 이 챕터는 레퍼런스로 사용하는게 좋고, 짧은 코드 예제 위주로 진행될것

## Major new features in JPA 2
- CASE, NULLIF, COALESCE 연산자 지원
- TREAT 연산자 downcast 가능
- …



# 용어 정리
- selection
    - 어디서 데이터를 가져올 것인지 정의
- restriction
    - 특정 기준에 맞는 레코드를 찾기 위해 적용
- projection
    - 쿼리로부터 반환될 데이터를 선택하기 위해 적용


## 15.1 selection
- selection 은 FROM 구문 또는 relation variable 을 선택하는 것을 말한다
    - SELECT … 구문을 말하는 것이 아님
    - 간단히는 어떤 테이블에서 “select” 를 할 것인지에 대한 내용
    - JPQL 에서는 테이블 대신에 어떤 클래스에 대한 데이터를 얻을 것인지

```sql
select i.ID, i.NAME, ... from ITEM i
```

```java
CriteriaQuery criteria = cb.createQuery(Item.class);
criteria.from(Item.class);
```

- 위 쿼리는 JPA 호환성이 없어서 서로 portable 하지 않음
    - JPA 표준에선 JPQL 은 select 구문을 가져야하고, criteria 쿼리는 select() 함수를 호출해야 한다

### 15.1.1 Assigning aliases and query roots
- select 구문을 JPQL 에서 사용하려면 FROM 구문에서 alias 를 부여해야 한다
- as 키워드는 optional
```sql
-- JPA compliant
select i from Item as i
```
- alias 는 case-insensitive


- portable criteria 쿼리는 select() 를 호출해야 한다
```java
CriteriaQuery criteria = cb.createQuery();
Root<Item> i = criteria.from(Item.class);
criteria.select(i);
```
- criteria 의 `Root` 는 항상 entity 를 참조한다
- `Metamodel` API 를 사용해서 entity type 을 동적으로 look up 할수도 있다
```java
EntityType entityType = getEntityType(em.getMetamodel(), "Item");
criteria.select(criteria.from(entityType));
```
- `getEntityType` 은 저자가 추가한 `Metamodel#getEntities()` 를 래핑한 함수

### 15.1.2 다형적 쿼리 (Polymorphic queries)
- `BillingDetails` 가 abstract class 일 때, 아래 쿼리는 모든 하위 클래스의 인스턴스를 반환할 것이다.
```
select bd from BillingDetails bd
criteria.select(criteria.from(BillingDetails.class));
```
- FROM 구문의 클래스는 mapped persistent class 일 필요가 없어서 아래같은 쿼리도 가능하다
- 아래 쿼리는 모든 persistent object 를 반환한다.
```sql
select o from java.lang.Object o
```
- 모든 serializable 타입의 클래스를 로드하는 쿼리
```sql
select s from java.io.Serializable s
```
- 하지만 JPA 는 임의의 인터페이스에 대한 다형적 JPQL 쿼리를 표준화하지 않았음
- 해당 기능은 Hibernate 에서는 동작하지만, portable 한 앱이 되려면 FROM 구문에서는 매핑된 entity class 만 참조해야 한다
    - (BillingDetails 나 CreditCard 같은..)
- criteria 의 from() 메소드는 매핑된 entity class 만 허용한다
- 다형적이지 않은 쿼리는 `TYPE` 함수를 사용해서 가져올 수 있다
```sql
select bd from BillingDetails bd where type(bd) = CreditCard
select bd from BillingDetails bd where type(bd) in :types
```
```java
Root<BillingDetails> bd = criteria.from(BillingDetails.class);
criteria.select(bd).where(
    cb.equal(bd.type(), CreditCard.class) // cb.not 을 쓰면 except CreditCard 
);
Root<BillingDetails> bd2 = criteria.from(BillingDetails.class);
criteria.select(bd2).where(
    bd2.type().in(cb.parameter(List.class, "types"))
);
```

## 15.2 restriction
- restriction : 쿼리가 반환할 데이터에 대한 제약조건을 표현하는 부분
- 보통 WHERE 구문과 where() 함수에서 제약조건 선언

```sql
select i from Item i where i.name = 'Foo'
```
```java
Root<Item> i = criteria.from(Item.class);
criteria.select(i).where(
    cb.equal(i.get("name"), "Foo")
);
```

### Ternary logic
- SQL, JPQL, criteria 쿼리는 제약조건을 ternary logic (not binary) 로 표현한다
- WHERE 구문은 true, false, 또는 null 으로 평가되는 논리 표현식이다
- java 에서, `nonNullObject == null` 은 `false` 이고, `null == null` 은 `true`이다
- SQL 에서, `NOT_NULL_COLUMN = null`, `null = null` 은 모두 `true` 가 아닌 `null` 로 해석된다
- 따라서 `IS NULL` 또는 `IS NOT NULL` 연산자를 사용해서 `null` 값 체크를 해야한다

### 15.2.1 Comparison expressions
- 예제
```sql
select b from Bid b where b.amount between 99 and 110
```
```java
Root<Bid> b = criteria.from(Bid.class);
criteria.select(b).where(
    cb.between(
        b.<BigDecimal>get("amount"),
        new BigDecimal("99"), new BigDecimal("110")
) );
```
- criteria 의 코드는 다소 생소할 수 있는데, `Root#get()` 메소드는 엔티티 속성의 `Path<X>` 를 반환한다.
- 타입 안정성을 유지하기 위해 위와 같이 속성 타입을 명시해줘야 한다
- criteria 의 `gt()` 함수는 `Number` 타입만 허용하므로 `Date`와 같은 타입을 사용하려면 `greaterThan()` 을 사용해야 한다

- 예제들
    - IN 구문
    - enum 사용
    - IS (NOT) NULL 연산자
    - LIKE 연산자 /w escape
        - % : 문자열에 대한 wildcard
        - _ : 하나의 문자에 대한 wildcard
        - escapeChar 를 지정해서 escape 가능 (보통 `\\` 를 사용)
    - 산술 표현식
    - 논리 연산자
    - AND 로 이어진 WHERE 구문의 경우 criteria 에서는 파라미터 나열 방식으로 간단하게 작성 가능

- `Predicate` 사용
```sql
select i from Item i
where (i.name like 'Fo%' and i.buyNowPrice is not null)
    or i.name = 'Bar'/
```
```java
Root<Item> i = criteria.from(Item.class);
Predicate predicate = cb.and(
    cb.like(i.<String>get("name"), "Fo%"),
    cb.isNotNull(i.get("buyNowPrice"))
);
predicate = cb.or(
    predicate,
    cb.equal(i.<String>get("name"), "Bar")
);
criteria.select(i).where(predicate);
```

### 15.2.2 Expressions with collections
- items collection 에 원소가 있는 Category 인스턴스들을 가져오는 예제
```sql
select c from Category c
    where c.items is not empty
```
```java
Root<Category> c = criteria.from(Category.class);
criteria.select(c).where(
    cb.isNotEmpty(c.<Collection>get("items"))
);
```

- 예제들
    - collection size 비교
    - collection 내에 특정 인스턴스가 있는지
    - persistent map 을 쓴 경우에 `key()`, `value()`, `entry()` 연산자 사용 가능

### 15.2.3 Calling functions
- WHERE 구문 내에서 함수를 호출할 수 있다는 것이 쿼리의 큰 장점
```sql
select i from Item i where lower(i.name) like 'ba%'
```
```java
Root<Item> i = criteria.from(Item.class);
criteria.select(i).where(
    cb.like(cb.lower(i.<String>get("name")), "ba%")
);
```

- Hibernate 에서 지원하는 추가적인 JPQL functions

![](./images/15-hibernate-query-functions.png)

- JPQL 의 WHERE 구문에서 Hibernate 에 없는 함수를 호출했다면 해당 함수는 DB 로 직접 전달된다
- 아래 예제에서 datediff 는 H2 DB 의 함수이다
```sql
-- only works in hibernate
select i from Item i
where datediff('DAY', i.createdOn, i.auctionEnd) > 1

-- JPA standard
select i from Item i
where function('DATEDIFF', 'DAY', i.createdOn, i.auctionEnd) > 1
```

```java
// JPA standard
Root<Item> i = criteria.from(Item.class);
criteria.select(i).where(
    cb.gt(
        cb.function(
            "DATEDIFF",
            Integer.class, // return type of datediff
            cb.literal("DAY"),
            i.get("createdOn"),
            i.get("auctionEnd")
        ),
    1 )
);
```

### 15.2.4 Ordering query results
- 예제
```sql
select u from User u order by u.username desc
```
```java
Root<User> u = criteria.from(User.class);
criteria.select(u).orderBy(
    cb.desc(u.get("username"))
);
```

- null 의 순서
    - ordering 하는 컬럼의 row 가 NULL 인 경우에 DBMS 에 따라 해당 로우가 첫번째에 올 수도 있고 마지막에 올 수도 있다.
    - JPQL 에서는 `ORDER BY ... NULLS FIRST|LAST` 로 순서를 지정할 수 있다. (JPA 표준 아님)
    - 다음 설정값으로 지정할 수도 있다 `hibernate.order_by.default_null_ordering = none | first | last`
- JPA 명세는 SELECT 구문에 있는 property / path 만 ORDER BY 구문에 허용
```sql
-- maybe non-portable
select i.name from Item i order by i.buyNowPrice asc
```

- 암묵적 inner join 을 하는 경우를 주의
```sql
-- only returns entities that have seller (maybe unexpected)
select i from Item i order by i.seller.username desc
```

## 15.3 Projection
- projection : 반환되기를 원하는 column 을 정의하는 것, JPQL 의 select 구문

### 15.3.1 Projection of entities and scalar values
```sql
select i, b from Item i, Bid b
```
```java
Root<Item> i = criteria.from(Item.class);
Root<Bid> b = criteria.from(Bid.class);
criteria.select(cb.tuple(i, b));
/* Convenient alternative:
criteria.multiselect(
    criteria.from(Item.class),
    criteria.from(Bid.class)
);
*/
```
- criteria 쿼리에서 여러 개의 root 를 더할 수 있다
- result tuple 을 사용할 때는 아래와 같이 사용하는데, 결과가 두 컬럼 값의 product 이므로 비효율적일 수 있다.
```java
List<Object[]> result = query.getResultList(); // Returns List of Object[]
Set<Item> items = new HashSet();
Set<Bid> bids = new HashSet();
for (Object[] row : result) {
    assertTrue(row[0] instanceof Item);
    items.add((Item) row[0]);
    assertTrue(row[1] instanceof Bid);
    bids.add((Bid)row[1]);
}
assertEquals(items.size(), 3);
assertEquals(bids.size(), 4);
assertEquals(result.size(), 12); // 3 * 4
```

- criteria Tuple API 를 사용해서 result list 에 타입을 줄 수 있다
```java
CriteriaQuery<Tuple> criteria = cb.createTupleQuery();
// Or: CriteriaQuery<Tuple> criteria = cb.createQuery(Tuple.class);
criteria.multiselect(
    criteria.from(Item.class).alias("i"),
    criteria.from(Bid.class).alias("b")
);
TypedQuery<Tuple> query = em.createQuery(criteria);
List<Tuple> result = query.getResultList();

for (Tuple tuple : result) {
    // indexed access
    Item item = tuple.get(0, Item.class);
    Bid bid = tuple.get(1, Bid.class);
    // aliased access
    item = tuple.get("i", Item.class);
    bid = tuple.get("b", Bid.class);
    // untyped meta access
    for (TupleElement<?> element : tuple.getElements()) { 
        Class clazz = element.getJavaType();
        String alias = element.getAlias();
        Object value = tuple.get(element);
    } 
}
```

### 15.3.2 Using dynamic instantiation
- Item entity instance 가 아닌, read-only 인 ItemSummary instance 를 가져오는 예제
```sql
select new org.jpwh.model.querying.ItemSummary(
    i.id, i.name, i.auctionEnd
) from Item i
```
```java
Root<Item> i = criteria.from(Item.class);
criteria.select(
    cb.construct(
        ItemSummary.class, // Must have the right constructor 
        i.get("id"), i.get("name"), i.get("auctionEnd")
) );
```
- JPQL 사용시 fully qualified class name 을 명시해야 한다
- 또한 nesting constructor call 을 호출해서도 안된다
- mapped entity class 도 매칭되는 생성자만 있으면 사용할 수 있다
    - 이 경우 만들어진 엔티티는 identifier 값을 설정 했느냐에 따라 transient 또는 detached 상태가 된다
    - 이 기능은 데이터를 간단히 복제할 때 사용할 수 있다
- DTO 에 적절한 생성자가 없다면 ResultTransformer 를 적용할 수 있다

### 15.3.3 Getting distinct results
- distinct 사용 예
```sql
select distinct i.name from Item i
```
```java
CriteriaQuery<String> criteria = cb.createQuery(String.class);
criteria.select(
    criteria.from(Item.class).<String>get("name")
);
criteria.distinct(true);
```
- 필터링은 디비수준에서 처리된다

### 15.3.4 Calling functions in projections
- projection 에서 함수 호출 예
```sql
select concat(concat(i.name, ': '), i.auctionEnd) from Item i
```
```java
Root<Item> i = criteria.from(Item.class);
criteria.select(
    cb.concat(
        cb.concat(i.<String>get("name"), ":"),
        i.<String>get("auctionEnd")
) );
```
- 예제들
    - coalesce()
        - 해당 인자가 모두 null 로 평가될 경우 null 또는 지정한 값을 반환, 아닐 경우 null 이 아닌 첫번째 값을 반환
    - case / when

- restriction 의 경우와 다르게, hibernate 는 projection 에 있는 unknown function 은 DB 로 직접 전달하지 않는다
- projection 에서 사용하는 함수는 hibernate 가 아는 함수이거나 JPQL 의 function() 연산자로 호출되어야 한다


