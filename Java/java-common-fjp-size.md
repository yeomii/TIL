# Java Common ForkJoinPool 쓰레드풀의 parallelism 조정하기

* `java.util.concurrent.ForkJoinPool` 코드 참고
* 적용 방법
    * java 실행시 argument 로 전달 `-Djava.util.concurrent.ForkJoinPool.common.parallelism={size}` 
    * java 내 코드에서 property 세팅 `System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism","{size}");`
