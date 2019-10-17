# Java Garbage Collection on the Cloud
* 세미나에서 나온 키워드들을 정리함

* how to rescue jvm application
    * GC + Runtime + JIT Compiler

## Explicit Memory Management
### Free-list Allocation (malloc / free)
* issue 
    * fragmentation
    * scalability
    * fitting algorithm : first fit vs best fit
* TC Malloc (가장 빠르다고 함)

## Automatic Memory Management
* Observation 
    * once dead always dead (no resurrection)
    * liveness : reachability from outside the heap
    * unrechable can not ever be used
    * find and preserve reachable objects
* Performance : space / time tradeoff
    * time proportional to dead objects (ref counting) or live objects (mark and sweep, semi-space)
    * throughput vs pause time
    * hidden locality benefit

## Implement Reachability
### Counting
* Reference Counting
    * update the counter on write (need lock)
    * cyclic reference problem
* Performance
    * slow allocation - free list
    * incremental reclamation
* python, swift
    * smart pointers in c++

### Tracing
* Mark and sweep

* What are GC roots? (mark)
    * global variables - static variables in Java
    * stack variables - method arguments
    * live registers

* implementation
    * marking bits - object header vs bitmap
    * marking - O(# of live objects)
    * sweep - O(# of live + dead objects)
    * allocation - free list 방식
    * fragmentation

### semi-space collector
* 영역을 두개로 나눠서 살아있는 object 를 다른 영역으로 복사한 후 old 영역을 날림

* 2-space 메모리를 반밖에 못 쓴다는 문제점이 있음
* 3-space 로 나눈다 (eden, from, to) - 8:1:1 정도의 비율
* 병렬처리 가능, 압축 (locality 효과가 있음), O(# of live objects)
* Bump pointer allocation 이 가능 (빠름!)

## GC strategies
### Allocation
* free list
* bump alloction
### Identification
* tracing
* reference counting
### Reclamation
* sweep to free
* compact
* evacuate

## Generational Hypothesis (G1GC)
* old gen vs young gen
* most of the objects die early (using heap only)
* survived objects live long
* use differenct gc strategies for these generations
    * shorter frequent young gen - 2-space collect
    * infrequent for old gen - mark and sweep

* old gen 에서 young gen 의 object 를 가지고 있는 경우는 young gen 에서 바로 gc 를 못함
* linked list 가 안좋은 케이스 (g1) 해시테이블을 쓰는 것이 좋음
* 상용으로 가장 많이 사용
* 몇번의 survive 를 거쳐야 old 할 것이냐 를 결정하는 것도 튜닝 포인트

* write barrier - put field byte code 에 자동으로 들어감
* card marking
    * remembered set
        * expensive

## Concurrent vs Parallel
### Concurrent GC
* application 과 함께 GC 가 동작하는 것
* stop-the-world GC 와 반대되는 개념

### Parallel GC
* GC 업무 자체를 병렬처리

### Concurrent and Parallel GC 
* 동시에 하는 것도 가능

## Real world garbage collector

* CMS 가 가장 보편적으로 많이 사용하는 방식

* G1 은 메모리 / cpu 오버헤드가 큼

* young generation
    *  3 space semi-space collector
* old generation
    * mark sweep + compact
* full gc - single thread
    * old gc 를 해도 부족할 경우

## GC Optimization on the Cloud
* Google GC uses 10-30% of the java cpu (튜닝 전)
* GC 쓰레드 수를 조정하는 것이 키 포인트

## Parallelizing GC phases
* Parallelizing Full GC (JEP-8130200)
    * 2-4x speed up
* Parallelizing Remark and Initial Mark GC
    * lower gc pause
    * already integrated to upstream
* Parallelizing Reference Processing
    * parallel processing final/weak/soft/phantom reference
        * soft 는 성능관리가 어려워 안 쓰는 것이 좋음

## Marking GC Safe (heuristic)
* Mutator (application) Utilization
    * old gc 가 늦어서 young gc 에서 collect 가 안되는 경우
    * memory threshold 로 트리거되기 때문에 young gc 가 계속 죽은 오브젝트를 들고 불필요한 연산을 하고있을 수 있음
    * MU = % of the time mutators run without gc
* MU 로 gc 를 트리거

## Practical GC Tuning
* Choose the right gc
    * CMS GC
    * Parallel GC
        * pause 가능
        * 예측이 쉽고 단순, 성능은 안 나올 수 있음
* No resizing
    * 상황이 달라지면 다 틀어질 수 있음
* Right size for young gen or old gen
    * -XX:NewSize=xxxx - XX:MaxNewSize=xxxx
* Number of GC Thread tuning
    * -XX:ConcGCThreads
    * -XX:ParallelGCThreads

## Conclusion
* Comparable performance to C++
* Huge resource savings on the Cloud
* Improved product quality

---

## + JIT Compiler
* runtime 에 있는 정보로 컴파일하면 굉장히 효율적일 수 있음
    * 어떤 것을 인라이닝 할 것인지..

* byte code is easy to understand from the sw

* dynamic compiler
    * uses runtime profiling data
    * branch prediction 에 사용
    * only compiles hot methods
    * easy to support reflective calls

* tiered compilation
    * quick compiler vs optimizing compiler
    * hotspot
        * interpreter
        * c1 - quick compiler
        * c2 - optimizing compiler
        * each method : interpreter > c1 > c2

* profiler
    * profile run-time information
        * hot methods, branches, allocations 
    * data is fed to compiler
    * sampling 
        * hotspot - event based
        * time based
        * bursty sampling

* inlining 이 50% 정도의 성능 개선 폭을 갖는다
    * 어떻게 잘 해야 성능이 잘 나오냐의 노하우는 학계에서도 불분명
    * 컴파일러나 프로파일러도 버그가 있어서 라고 하던데...

* Dynamic class-loading & unloading
    * Java classes can be loaded/unloaded at runtime

* java vs c++
    * faster
        * inlining effect with runtime profiling data
        * gc improveds locality
    * slower
        * memory overhead
        * startup 
        * gc
        * resource

* make gc-friendly code
    * 새로 object 를 생성하는 것은 괜찮음
    * object 가 최대한 짧은 생명주기를 갖도록 하기
    * linked list 는 조심해서 사용하기
* make JIT-friendly code
    * branch condition as constant (static final 을 쓰면 jit compiler 에서 상수를 사용함)
    * 작은 함수를 만들면 적극적으로 inlining 을 할 수 있다. overhead 걱정하지 말기
    * native call 을 남용하지 말자 (inlining 불가)