# DS101: Introduction to Apache Cassandra

* https://academy.datastax.com/resources/ds101-introduction-cassandra

# Relational Overview

## Medium Data
* Fits on 1 machine
* RDBMS is fine
* supports hundreds of concurrent user
* ACID makes us feel good (guarantee ACID)
* scales vertically (크기가 커질 경우 scale up 해야함)

## Replication: ACID is a lie
* master-slave 구조에서 replication 은 async 하게 수행되고, 이때 발생하는 지연은 replication lag 이라 한다.
* master 에 쓰고, slave 에서 읽는다면 consistency 가 깨질 수 있음

## Third Normal Form Doesn't Scale
* 쿼리가 매우 길고 속도가 느리면 데이터가 denormalized 될것.
* 데이터가 메모리보다 크다면 실시간이 아닌 데이터를 가져올 수도 있음

## Shardinng is a nightmare
* 모든것을 denormalize 해야하고
* secondary index 를 사용하는 쿼리는 모든 샤드를 조회해야함
* 샤드를 늘리려면 데이터를 수동으로 움직여줘야함
* 스키마가 변경된다면 또 변경이 필요함

## HA.. not really
* master-slave 구조로 복제할때
* failover 를 수동이든 자동이든 돌려야하는데, 수동으로 하자니 바로 대응하기 어렵고, 자동으로 돌리자니 failover 하는 컴포넌트에 대한 failover 를 또 고려해야함
* db 세팅 변경, os 업데이트 등 다운타임이 자주 발생
* DC 가 여러개라도 되면 관리가 너무 어렵다.

## 정리하자면 
* scaling is a pain
* ACID is naive at best
* Re-sharding is a manual process
* denormalization for performance
* HA is complicated.

## Lessons Learned
* consistency 는 비현실적 -> 버리기로함
* 수동으로 하는 샤딩이나 리밸런싱은 어렵다 -> 내부적으로 구현
* 모든 움직이는 파트는 시스템을 복잡하게 한다 -> 아키텍처를 매우 심플하게 한다, no master&slave
* 스케일업은 비용이 높다 -> 범용 하드웨어를 원함
* scatter / gather 는 안좋음 -> 실시간 쿼리성능을 위해 denormalize, 항상 한대의 머신만 조회하는게 목표

---

# Cassandra Overview

## What is Cassandra?
* Fast distributed db
* HA
* Linear Scalability
* Predictable Performance
* No SPOF
* Multi-DC
* Commodity Hardware
* Easy to manage operationally
* not a drop in replacement for RDBMS

## Hash Ring
* No master / slave / replica sets
* No config servers, zookeeper
* Data is partitioned around the ring
* Data is replicated to RF=N servers
* All nodes hold data and can answer queries (both reads & writes)
* Location of data on ring is determined by partition key

## CAP Tradeoffs
* Impossible ot be both consistent and highly available during a network partition
* Latency between data centers also makes consistency impractical
* Cassandra chooses Availability & Partition Tolerance over Connsistency

## Replication 
* Data is replicated automatically
* You pick number of servers
* Called "replication factor" or RF
  * 보통 3
* Data is always replicated to each replica
* If a machine is down, missing data is replayed via hinted handoff
* keyspace 를 만들 때 설정한다

## Consistency Levels
* Per query consistency
* ALL, QUORUM, ONE
* How many replicas for query to respond OK
* read, write 요청 모두에 설정가능

## Multi DC
* Typical usage: client write to local DC, replicates async to other DCs
* Replication factor per keyspace per datacenter
* Datacenters can be physical or logical

---

# Choosing a Distribution

## The write path
* Writes are written to any node in the cluster (coordinator)
* Writes are written to commit log, then to memtable
* Every write includes a timestamp
* Memtable flushed to disk periodically (sstable)
* New memtable is created in memory
* Deletes are a special write case, called a tombstone

## What is an SSTable?
* Immutable data file for row storage
* Every write includes a timestamp of when it was written
* Partition is spread across multiple SSTables
* Same column can be in multiple SSTable
* Merged throuhgh compaction, only latest timestmp is kept
* Deletes are written as tombstones
* Easy backups!

## The read path
* Any server may be queried, it acts as the coordinator 
* Contacts nodes with the requested key
* On each node, data is pulled from SSTables and mergd
* Consistency < ALL performs read repair in background (read_repair_chance)

## Open Source
* Latest, bleeding edge features
* File JIRAs
* support via mailing list & IRC
* Fix bugs
* cassandra.apache.org
* Perfect for hacking

## DataStax Enterprise
* Integrated Multi-DC Search
* Integrated Spark for Analytics
* Included on USB






