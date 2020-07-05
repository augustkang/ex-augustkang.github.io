# [요약]DevOps와 SE를 위한 리눅스 커널 이야기 4장


# 4장 - free 명령이 숨기고 있는 것들

### 메모리 사용량 확인하기
```bash
root@August-PC:~# free -m
              total        used        free      shared  buff/cache   available
Mem:           8047        6161        1662          17         223        1755
Swap:         24576         214       24361
# 출력 데이터 단위에 따라 옵션이 다르다.
# -b: byte 단위, -k: KB 단위, -m: MB 단위, -G: GB 단위
```
total : 전체, used : 사용중, free : 사용 안하고 있는 남는 용량

buff/cache : buffers와 cache 영역에서 차지하는 용량을 보여줌

Swap : swap 영역에 대한 내용. 전체, 사용중인 용량, 남는 용량

### buffers와 cached 영역

* buffers 영역은 buffer cache 영역을 뜻함.

super block, inode block 과 같이 파일의 내용이 아닌 파일시스템 관리에 필요한 메타데이터들을 위한 cache

* cached 영역은 page cache 영역을 뜻함.

파일 자체의 내용을 저장하는데 쓰이는 cache

### /proc/meminfo 읽기

```bash
[root@August-PC ~]# cat /proc/meminfo 
MemTotal:        7957324 kB
MemFree:          299176 kB
MemAvailable:    4854520 kB
Buffers:          141072 kB
Cached:          4768400 kB
SwapCached:        75588 kB
Active:          4118228 kB
Inactive:        2813128 kB
Active(anon):    1330700 kB
Inactive(anon):  1069180 kB
Active(file):    2787528 kB
Inactive(file):  1743948 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:       8388604 kB
SwapFree:        7214696 kB
Dirty:               136 kB

... 생략
```

* SwapCached: swap으로 사용된 영역 중 다시 메모리로 돌아온 영역. Swap으로 빠졌다고 돌아온 영역이 얼마나 되는지 나타냄.

* Active(anon): Page Cache 영역을 제외한 메모리 영역을 의미. 주로 프로세스들이 사용하는 메모리 영역 지칭. 그 중에서도 최근에 참조된(active) 영역 나타낸다. Active는 최근에 접근된 이력이 있는 영역이라는 뜻.

* Inactive(anon): 참조된 지 오래되어 Swap영역으로 이동할 수도 있는 메모리 영역. Inactive는 말 그대로 active 하지 않은, 접근된지 오래된, 최근에는 접근하지 않은 영역..

* Active(file): 커널이 I/O 성능 향상을 위해 캐시 목적으로 사용하는 영역.  buffer cache, page cache 영역이 여기에 속함. 그 중에서도 자주 접근된 영역

* Inactive(file): 마찬가지로 커널이 캐시 목적으로 사용하는 영역 중에서도 접근한지 오래된 영역.

* Dirty: Active(file), Inactive(file)과 비슷한 용도로 I/O 성능 향상을 위해 커널이 캐시 목적으로 사용하는 영역 중  작업이 이루어져서 실제 블록 디바이스의 블록에 씌어져야 할 영역을 의미. 즉 디스크 쓰기 이후 commit이 이루어져야 하는 영역. 보통 block device에 write하면 block device write 연산은 메모리에 쓰는 것 보다 굉장히 latency가 크기 때문에(데이터 쓰는데 오래 걸리기 때문에) 그걸 끝날때까지 기다리기 보단 바로바로 처리 안하고 처리하기로 해놓고 나중에, 미래에 시스템에 여유가 있을때 실질적인 쓰기를 처리 하게 하는데 이 실질적으로 처리가 되어야 하는 영역을 의미.

* `/fs/proc/meminfo.c` 파일에서 active와 inactive를 구분하는 기준을 확인할 수 있다.

* LRU 기반의 리스트로 관리. 단순히 시간을 기반으로 시간이 지나 오래된 것, 최신의 것으로 나누지는 않음.

* vm.min_free_kbytes 커널 파라미터보다 여유 메모리 공간이 적게 될 경우 kswapd가 실행되면서 active 영역 페이지 중 오래된 페이지를 우선적으로 inactive로 옮긴 후 메모리를 해제(free)하는 작업을 수행한다.

### slab 메모리 영역

`cat /proc/meminfo`

```bash
... 생략
Slab:             473356 kB
SReclaimable:     402036 kB
SUnreclaim:        71320 kB
... 생략
```

* slab: 메모리 영역 중 커널이 직접 사용하는 영역. 여기에는 dentry cache, inode cache 등 커널이 사용하는 메모리가 포함.

* SReclaimable: Slab 영역 중 재사용될 수 없는 영역. 캐시 용도로 사용하는 메모리들이 여기에 포함됨. 메모리 부족 현상 발생하면 해제(free)되어 프로세스에 할당이 가능한 영역

* SUnreclaim: Slab 영역 중 재사용 될 수 없는 영역. 커널이 현재 사용중이며 해제해서 다른 용도로 쓸 수 없는 영역.

```bash
Active / Total Objects (% used)    : 1718090 / 1895312 (90.6%)
 Active / Total Slabs (% used)      : 48101 / 48101 (100.0%)
 Active / Total Caches (% used)     : 82 / 106 (77.4%)   
 Active / Total Size (% used)       : 435110.21K / 469842.19K (92.6%)
 Minimum / Average / Maximum Object : 0.01K / 0.25K / 12.75K

  OBJS ACTIVE  USE OBJ SIZE  SLABS OBJ/SLAB CACHE SIZE NAME
842205 743124  88%    0.10K  21595       39     86380K buffer_head
382368 368286  96%    0.19K   9104       42     72832K dentry
167216 167216 100%    0.94K   4919       34    157408K xfs_inode
115668 111428  96%    0.57K   4131       28     66096K radix_tree_node
 55040  47535  86%    0.06K    860       64      3440K kmalloc-64
 28176  28176 100%    0.16K    587       48      4696K xfs_ili
 28152  28152 100%    0.12K    828       34      3312K kernfs_node_cache
 26344  16899  64%    0.21K    712       37      5696K vm_area_struct
 19278  10360  53%    0.19K    459       42      3672K kmalloc-192
 16000  15514  96%    0.03K    125      128       500K kmalloc-32
 15912  15912 100%    0.04K    156      102       624K selinux_inode_security
... 생략
```
* buddy system : linux에서 메모리를 할당해주는 시스템. 가능한 적당하게 메모리 요청을 만족하도록 메모리를 여러 부분으로 나누는 메모리 할당 시스템.
메모리의 크기를 절반씩 분할하면서 가장 잘 맞는 크기의 메모리를 찾아 할당해 준다. 2의 거듭제곱 값으로 메모리를 할당.
* slab allocator : 버디 시스템을 통해 기본 페이지 크기인 4KB 영역을 할당 받은 후에 각각의 캐시 크기에 맞게 영역을 나눠서 사용.
* slab 영역은 `free`명령에서 used 영역에 포함된다. 커널이 사용하기 때문에 buffers/cached에 포함될거라 생각할 수 있지만, 실제로는 used에 포함됨.

출처 : [DevOps와 SE를 위한 리눅스 커널 이야기](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966264049&orderClick=LEA&Kc=)




