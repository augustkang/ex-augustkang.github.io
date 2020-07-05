# [요약]DevOps와 SE를 위한 리눅스 커널 이야기 1장


# 1장 - 시스템 구성 정보 확인하기


### 커널 정보 확인하기 

```bash
uname -a # 커널 버전 확인dmesg #커널 디버그 메세지
cat /boot/config-`uname -r` | more # 커널 컴파일 옵션 확인
cat /boot/config-`uname -r` | grep -i config_function_tracer
```

 * ftrace와 같은 커널 함수 레벨의 추적이 필요할 경우 CONFIG_FUNCTION_TRACER 같은 옵션이 커널 컴파일 옵션에 포함되어있어야 함

### CPU 정보 확인하기

```bash
dmidecode -t processor
cat /proc/cpuinfo
lscpu
```

 * socket : number of CPU
 * core : number of cores per CPU
 * 시스템 최대 쓰레드 수 : # of socket * # of core * 2(Hyperthreading, Hyperthreading 지원 안할시 * 1)

### Memory 정보 확인하기

```bash
dmidecode -t memory
```

 * Physical memory array : 하나의 CPU socket에 함께 할당된 물리 메모리의 그룹
 * Memory device : 실제로 시스템에 꽂혀있는 메모리. 

### 디스크 정보 확인하기

```bash
df -h
smartctl -a /dev/sda
smartctl -a /dev/sda -d cciss,0 # lsmod 를 통해 알아낸 현재 시스템 사용중인 raid 컨트롤러 드라이버를 명시
```

 * sda : scsi 방식 또는 sata, sas 방식 같은 일반적 디스크
 * hda : 개인용 컴퓨터용인 IDE 방식의 디스크

### 네트워크 정보 확인하기

```bash
lspci | grep -i ether
ethtool eth0
ethtool -g eth0 # Ring buffer의 크기를 확인 가능, 볼땐 -g, 설정할땐 -G 옵션
ethtool -k eth0 # 현재 NIC의 다양한 성능 최적화 옵션 확인, 마찬가지로 볼땐 소문자 -k, 설정할땐 -K
ethtool -i eth0 # NIC의 커널 모듈 정보 표시
```

 * Ring buffer : 네트워크 카드의 버퍼 공간을 의미. 크기가 maximum으로 세팅 되어있지 않으면 네트워크 성능 저하 발생 가능
 * tcp-offload : MTU 이상의 크기를 가지는 패킷의 분할작업을 CPU 가 아닌 NIC가 직접 처리하여 CPU 성능 부담 줄이고 패킷 처리 성능 높임.

출처 : [DevOps와 SE를 위한 리눅스 커널 이야기](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966264049&orderClick=LEA&Kc=)

