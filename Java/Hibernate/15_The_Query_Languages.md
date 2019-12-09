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





