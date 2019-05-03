# 프로세스 메모리 사용량 확인하기

* Red Hat 계열 기준으로 작성됨

## 전체 메모리 사용량 확인하기
* /proc/meminfo 의 파일 내용을 확인
```sh
$ cat /proc/meminfo
MemTotal:       65555840 kB
MemFree:         6931600 kB
MemAvailable:   32036184 kB
Buffers:              20 kB
Cached:         28247964 kB
SwapCached:            0 kB
Active:         20692744 kB
Inactive:       22345112 kB
Active(anon):   17076912 kB
Inactive(anon):  1149540 kB
Active(file):    3615832 kB
Inactive(file): 21195572 kB
Unevictable:    13894796 kB
Mlocked:        13894796 kB
SwapTotal:             0 kB
SwapFree:              0 kB
Dirty:              8120 kB
Writeback:             0 kB
AnonPages:      28684952 kB
Mapped:          2420168 kB
Shmem:           3310712 kB
Slab:             967196 kB
SReclaimable:     822504 kB
SUnreclaim:       144692 kB
KernelStack:       29616 kB
PageTables:       183876 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:    32777920 kB
Committed_AS:   20026524 kB
VmallocTotal:   34359738367 kB
VmallocUsed:      434140 kB
VmallocChunk:   34325379068 kB
HardwareCorrupted:     0 kB
AnonHugePages:         0 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
DirectMap4k:      252624 kB
DirectMap2M:     6858752 kB
DirectMap1G:    61865984 kB
```
* vmstat 명령으로도 확인 가능하다
```sh
$ vmstat -s
     65555840 K total memory
     29784612 K used memory
     21092368 K active memory
     23818120 K inactive memory
      5061412 K free memory
           20 K buffer memory
     30709796 K swap cache
            0 K total swap
            0 K used swap
            0 K free swap
   6568655064 non-nice user cpu ticks
    616988026 nice user cpu ticks
   2192092359 system cpu ticks
 140466580537 idle cpu ticks
    116889627 IO-wait cpu ticks
            0 IRQ cpu ticks
    137278067 softirq cpu ticks
            0 stolen cpu ticks
   1380671050 pages paged in
 846830147862 pages paged out
            0 pages swapped in
            0 pages swapped out
   3802233088 interrupts
    434119329 CPU context switches
   1509520335 boot time
    323473833 forks
```

## 프로세스별로 확인하기
* process id 의 status 파일 확인
```sh
$ cat /proc/{pid}/status
...
VmPeak:	118746084 kB
VmSize:	96278920 kB
VmLck:	15197836 kB
VmPin:	       0 kB
VmHWM:	44469872 kB
VmRSS:	31205008 kB
RssAnon:	28836576 kB
RssFile:	 2368432 kB
RssShmem:	       0 kB
VmData:	75024900 kB
VmStk:	     140 kB
VmExe:	       4 kB
VmLib:	   18312 kB
VmPTE:	  174368 kB
VmSwap:	       0 kB
```

## 프로세스의 메모리 매핑 정보 확인
* pmap 커맨드 사용
```
$ pmap -X {pid}
Address Perm   Offset Device      Inode     Size      Rss      Pss Referenced Anonymous Swap   Locked Mapping
00400000 r-xp 00000000 103:04 1074351795        4        4        4          4         0    0        4 java
00600000 rw-p 00000000 103:04 1074351795        4        4        4          4         4    0        4 java
4c0000000 rw-p 00000000  00:00          0 12585472 12585472 12585472   12585472  12585472    0 12585472
7c0280000 rw-p 00000000  00:00          0     3664     3664     3664       3664      3664    0        0
7c0614000 ---p 00000000  00:00          0  1042352        0        0          0         0    0        0
...
```
* Perm 의 마지막 글자는 p 또는 s 로, private 와 shared 를 나타낸다
* 메모리에 파일이 매핑되어 있는 경우 Mapping 에 파일 이름이 나타난다
* RSS 크기가 물리 메모리에 올라간 해당 영역에 실제 크기를 나타낸다