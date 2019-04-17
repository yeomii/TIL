# Linux 에서 메모리 스왑 사용하지 않도록 하기

* Cassandra 와 같은 몇몇 애플리케이션은 성능 저하를 막기 위해 메모리 스왑을 사용하지 않을 것을 권장한다.

* swap 영역 사용량 확인하기
```sh
$ free -h
              total        used        free      shared  buff/cache   available
Mem:           7.6G        6.1G        381M        268M        1.2G        784M
Swap:          1.0G        154M        869M
```
* 시스템 전체 스왑 기능 비활성화하기
```sh
$ sudo swapoff -a 
```