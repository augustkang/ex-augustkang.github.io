# [요약]DevOps와 SE를 위한 리눅스 커널 이야기 2장


# 2장 - top을 통해 프로세스 정보 살펴보기

### top 명령어 실행 결과

```bash
root@August-PC:~# top -b -n 1
top - 01:57:36 up 58 min,  0 users,  load average: 0.52, 0.58, 0.59
Tasks:   8 total,   1 running,   7 sleeping,   0 stopped,   0 zombie   
%Cpu(s):  9.0 us, 23.3 sy,  0.0 ni, 67.4 id,  0.0 wa,  0.3 hi,  0.0 si,KiB Mem :  8240388 total,  2032052 free,  5978984 used,   229352 buff/cKiB Swap: 25165824 total, 24837212 free,   328612 used.  2127672 avail 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+    
    1 root      20   0    8916    328    284 S   0.0  0.0   0:00.07    
  880 root      20   0    8916    228    176 S   0.0  0.0   0:00.01    
  881 august    20   0   18356   3764   3448 S   0.0  0.0   0:00.38    
 1074 root      20   0    8916    232    180 S   0.0  0.0   0:00.00    
 1075 august    20   0   18224   3492   3364 S   0.0  0.0   0:00.09    
 1087 root      20   0   16260   2116   2088 S   0.0  0.0   0:00.01    
 1088 root      20   0   18088   3392   3304 S   0.0  0.0   0:00.06    
 1104 root      20   0   18780   1888   1420 R   0.0  0.0   0:00.01
```
* line 1: 현재 시스템의 시간과 uptime, 그리고 load average가 나온다
* line 2: 현재 실행중인 process들
* line 3: 리소스 사용량 통계
* PID: process id, PR : Priority, NI : Nice value
* VIRT : VIRTual memory, 해당 프로세스에게 할당된 가상메모리 전체의 크기
* RES: RESident memory, 해당 프로세스가 실제로 사용중인 물리메모리의 양
* S : Process Status

### Memory Commit 의 개념

커널은 프로세스가 메모리를 요청할 때 그에 맞는 크기를 가상으로 할당해준다. 실제로 요청받은 크기 전체만큼 물리메모리에 assign해주지는 않음.
그렇게 있다가 실제로 사용이 필요할 때마다 물리메모리를 조금씩 더 할당을 해줌(늘려줌). 이를 memory commit 이라고 한다.

* vm.overcommit_memory는 커널의 Memory commit 동작 방식을 변경할 수 있게 해주는 커널 파라미터.
* 0=최댓값 바탕으로 Memory commit, 1=무조건 overcommit, 2=vm.overcommit_ratio에 설정된 비율을 바탕으로 제한적으로 memory commit

#### 커널 파라미터 보는법? sysctl

```bash
root@August-PC:~# sysctl -a
fs.binfmt_misc.status = enabled
fs.binfmt_misc.WSLInterop = enabled
fs.binfmt_misc.WSLInterop = interpreter /init
...
...
vm.min_free_kbytes = 45056
vm.overcommit_memory = 0
vm.swappiness = 60
```
* `sysctl -a | grep pid_max` 하면 현재 시스템의 생성 가능한 최대 pid 확인 가능

### 프로세스 상태


* Process Status 종류 : D, R, S, T, Z
* D = Uninterruptible sleep - 디스크, 네트워크 디바이스에 작업 요청 후 대기중인 상태(Run queue가 아닌 wait queue에 넣어놓음)
* R = Running - 현재 CPU에서 실행되고 있는 상태
* S = Sleep - 그냥 자고있는 애들. D와의 차이는 요청한 리소스를 즉시 사용할 수 있는지 여부이다.
* T = traced or stopped. strace 등으로 프로세스가 추적되고 있는 상태
* Z = zombie 상태. parent process가 죽은 child process를 의미

### 프로세스 우선 순위

* 기본 PR값은 20이다.
* PR값에서 NI값을 뺀 값이 최종 우선순위가 된다. 숫자 0에 가까울수록 우선순위 높은것(뭔개소리죠??? 직접 해보면 안다...)
* `nice -n N command` 명령으로 해당 프로세스 nice 값을 부여해줄 수 있음.(N은 원하는 숫자)
* RT는 RealTime의 약자로써, RT 스케줄러에 의해 실시간으로 커널에 의해 처리되는 프로세스들임. CFQ(Completely Fair Queueing) 스케줄러보다 먼저 실행된다.

*아 커널 소스 나와서 당황했다. 절대 만만치 않은 책인 듯 싶다.*

출처 : [DevOps와 SE를 위한 리눅스 커널 이야기](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966264049&orderClick=LEA&Kc=)

