# [요약]DevOps와 SE를 위한 리눅스 커널 이야기 3장


# 3장 - Load average와 시스템 부하

### Load average의 정의

* Load average는 R(run queue에서 cpu 사용을 위해 대기중인 프로세스들)과 D(네트워크/디스크 i/o 를 위해 wait queue 에서 대기중인 프로세스들)상태에 있는 프로세스 개수의 1분, 5분, 15분마다의 평균값이다.  
CPU 코어 갯수가 많으면 코어 갯수가 하나일 때 보다는 각 프로세스들이 여러개의 코어에 의해 더 빨리 처리될 수 있다.

### Load average 계산 과정

```bash
root@August-PC:~# uptime
 21:44:37 up 6 min,  0 users,  load average: 0.52, 0.58, 0.59
```

```bash
root@August-PC:~# strace -s 65535 -f -t -o uptime_dump uptime
 05:36:52 up  5:31,  0 users,  load average: 0.52, 0.58, 0.59
```
`/proc/loadavg` -> `/fs/proc/loadavg.c`에서 `loadavg_proc_show()` 함수 -> `/kernel/sched.c` 에서 `get_avenrun()` 함수 -> `avenrun[]` 배열 -> `calc_global_load()` 함수 -> `calc_load_account_active()` 함수 순으로 추적하다보면 `nr_active` 라는 변수에 `nr_running` + `nr_uninterruptible` 값들을 합하고 이를 1분, 5분, 15분 평균으로 나눠 계산하는 것을 확인할 수 있다.

`nr_running`은 run queue에 있는 R상태의 프로세스들의 갯수이며, `nr_uninterruptible`은 wait queue에 있는 D상태의 프로세스들의 갯수이다.

### CPU Bound vs I/O Bound
위에서 확인했듯 Load average는 cpu 사용을 기다리는 프로세스 갯수로만 계산되는 것이 아니라, 디스크/네트워크 I/O를 위해 대기중인 프로세스의 갯수도 계산에 포함된다.  즉, Load average가 높다는 것은 단순히 CPU를 사용하려는 프로세스가 많다는 의미가 아니고, I/O 병목이 생겨 I/O 작업을 대기하는 프로세스가 많을 수도 있다는 의미이다.  
그러므로 Load average 숫자만 보고 시스템에 정확히 어떤 상태의 부하가 일어나고 있는지 파악하기가 어렵다.  
그럼 부하의 종류를 어떻게 확인할까?

### vmstat으로 부하의 정체 확인하기

```bash
[root@august-PC ~]# vmstat
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0 1211796 143872 141040 5541164    0    0    15    19    0    0 26  3 69  1  0
```
* r : The number of processes waiting for run time.  
실행되기 위해 기다리는 프로세스 갯수(nr_running)  
* b : The number of processes in uninterruptible sleep  
I/O를 위해 기다리는 프로세스 갯수(nr_uninterruptible)  

`/proc/sched_debug` 파일을 확인하면 각 CPU의 Run queue 상태와 스케줄링 정보를 살펴볼 수 있다.  

```bash
[root@august-PC ~]# cat /proc/sched_debug 
Sched Debug Version: v0.11, 3.10.0-862.6.3.el7.x86_64 #1
ktime                                   : 31318815611.994170
sched_clk                               : 31318700701.301538
cpu_clk                                 : 31318700701.301627
jiffies                                 : 35613482909

...중략

cpu#0, 2200.007 MHz
  .nr_running                    : 0
  .load                          : 0
  .nr_switches                   : 6296361526
  .nr_load_updates               : 15707632257
  .nr_uninterruptible            : 434324
  .next_balance                  : 35613.482909
  .curr->pid                     : 0
  .clock                         : 31318700701.633510
  .cpu_load[0]                   : 0
  .cpu_load[1]                   : 0

...중략

runnable tasks:
            task   PID         tree-key  switches  prio     wait-time             sum-exec        sum-sleep
----------------------------------------------------------------------------------------------------------
     ksoftirqd/0     3 13546607321.808677 190671117   120         0.000000   1013193.525038         0.000000 0 /
    kworker/0:0H     5       544.548835         6   100         0.000000         0.029044         0.000000 0 /
      watchdog/0    12        -3.904516   7829709     0         0.000000    125610.812095         0.000000 0 /
       kdevtmpfs    51 1877281760.662029       185   120         0.000000         3.293578         0.000000 0 /
         kswapd0    70 13546604016.431647    923034   120         0.000000    894054.731239         0.000000 0 /
       scsi_eh_0   282       485.192906         2   120         0.000000         0.012182         0.000000 0 /
    kworker/0:1H   341 13546605978.612818  37669618   100         0.000000    601004.086434         0.000000 0 /
              su  4429 13546605952.403489        23   120         0.000000        33.713593         0.000000 0 /
           mysql  4582 13546606319.488393         5   120         0.000000        15.271120         0.000000 0 /
             sed  4708 13546607347.312431         2   120         0.000000         1.549732         0.000000 0 /
...생략
```

출처 : [DevOps와 SE를 위한 리눅스 커널 이야기](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966264049&orderClick=LEA&Kc=)

