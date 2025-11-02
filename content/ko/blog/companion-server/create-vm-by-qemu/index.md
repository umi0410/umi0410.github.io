---
title: "QEMU로 Ubuntu Server VM 생성하기"
date: 2022-12-08T01:31:00+09:00
weight: 22
categories:
- companion-server
cover:
  image: preview.png
---
## 시작하며

직접 구축하는 홈서버에 VM도 추가하고 싶어졌고 이전 글[[1]](../create-vm-by-vboxmanage/)과 마찬가지로 CLI로 VM을 띄워보고 싶었다.
이전 글에서는 VirtualBox를 이용했지만 이번에는 qemu라는 도구를 이용해 VM을 만들어보고자한다.

qemu는 VM을 띄우는 도구들 중에서도 꽤나 로우한 레벨에 있으면서 손쉽게 다양한 아키텍쳐를 가상화해서 사용할 수 있게 해준다.
또한 CLI로 실행하는 프로그램이기에 VM을 손쉽게 띄워보고 싶은 나 같은 경우에 적합하다.

## 실습 환경

호스트 환경은 아래와 같다.

| 항목   | 정보                  |
|------|---------------------|
| CPU  | amd64, 8 Core       |
| RAM  | 16GB                |
| Disk | 256GB               |
| OS   | Ubuntu server 22.04 |

구성하고자하는 VM은 다음과 같다.

| 항목   | 정보                  |
| --- | --- |
| CPU | amd64, 2 Core |
| RAM | 4GB |
| Disk | 16GB |
| OS | Ubuntu server 22.04 |

VM을 생성하는 작업은 root 권한을 많이 필요로 할 수 있어 이번 글에서 수행하는 커맨드는 대부분 root 권한으로 수행해야 올바르게 동작할 것이다.
커맨드를 표기할 때에는 root 권한으로 실행한다하더라도 편의상 `#`이 아닌 `$` 문자를 커맨드의 맨 앞에 적어뒀다.

## 설치하기
```shell
# on linux
$ sudo apt update
$ sudo apt-get install qemu-kvm qemu

# on OSX
$ brew install qemu
```

위와 같이 간단히 qemu를 설치해볼 수 있다. 설치가 잘 됐으면 다음 명령어로 설치된 버전을 확인해볼 수 있을 것이다.
x86_64 아키텍쳐의 VM을 이용할 것이기에 이번 글에서는 `qemu-system-x86_64`라는 명령어를 사용할 것이다.
편의상 `qemu-system-x86_64` 명령어에 대해 `qs`라는 alias를 설정하고 진행하도록 하겠다. 

```shell
$ alias qs='qemu-system-x86_64'
$ qs --version
QEMU emulator version 6.2.0 (Debian 1:6.2+dfsg-2ubuntu6.5)
Copyright (c) 2003-2021 Fabrice Bellard and the QEMU Project developers
```
## 이미지 생성하기

이미지는 VM의 저장장치로서 사용될 수 있다. 운영체제 설치파일이나 루트로 사용할 저장장치 등이 이미지로 정의된다.

qemu와 사용할 수 있는 이미지 타입은 `raw`와 `qcow2` 이렇게 두 개가 있다. 처음에는 raw를 사용할까싶긴 했지만
스냅샷이나 암호화, 자동화, 압축, 저장공간 효율 등의 측면에서 `qcow2`를 사용하는 게 더 적합할 것 같았다.
아무래도 `raw` 타입의 장점은 실제 물리 장비에 근접한 성능인 듯한데 요즘은 `qcow2`도 성능이 좋다는 것 같았다.

```shell
$ mkdir -p vm1 && cd vm1
$ curl -LO https://cloud-images.ubuntu.com/releases/jammy/release-20221201/ubuntu-22.04-server-cloudimg-amd64.img
```

이번 글에서는 Ubuntu Server 22.04 클라우드 이미지를 통해 운영체제를 자동으로 설치할 것이다.
vm의 이름은 vm1이라고 하고 vm1이라는 디렉토리에서 작업해나갈 것이다.

```shell
$ qemu-img info ubuntu-22.04-server-cloudimg-amd64.img
image: ubuntu-22.04-server-cloudimg-amd64.img
file format: qcow2
virtual size: 2.2 GiB (2361393152 bytes)
disk size: 636 MiB
cluster_size: 65536
Format specific information:
    compat: 0.10
    compression type: zlib
    refcount bits: 16
```

qemu-img라는 프로그램을 통해 다운로드 받은 이미지를 살펴봤다. qemu-img는 qemu의 디스크 이미지와 관련된 편의 도구이다.
이미지를 살펴보니 가상 사이즈가 2.2G로 잡혀있다. VM에서는 자신의 디스크의 크기를 2.2G로 인식할 듯하다.
이를 넉넉하게 16G로 늘려주자.

```shell
$ qemu-img resize ubuntu-22.04-server-cloudimg-amd64.img 16G
Image resized.

$ qemu-img info ubuntu-22.04-server-cloudimg-amd64.img
image: ubuntu-22.04-server-cloudimg-amd64.img
file format: qcow2
virtual size: 16 GiB (17179869184 bytes)
disk size: 636 MiB
cluster_size: 65536
Format specific information:
    compat: 0.10
    compression type: zlib
    refcount bits: 16
```

이미지를 16G로 resize 해준 뒤 다시 info를 통해 정보를 조회해보면 virtual size가 16G로 늘어난 것을 확인할 수 있다.

이제 Auto install 시에 사용할 설정 정보를 정의하는 디스크 이미지를 만들 차례이다. 이와 관련해서는 cloud-init이나 user-data 같은 키워드들이가 존재하니
자세한 사항이 필요하면 별도로 찾아보는 게 나을 듯하다. 대충 [이곳의 예시](https://cloudinit.readthedocs.io/en/latest/topics/examples.html)처럼 생겼다.

간략히 설명하면 ubuntu를 자동 설치할 때 `user-data`나 `metadata`라는 파일을 통해 초기 설정을 정의할 수 있다.
이를 통해 나는 나의 ssh 공개키나 vm의 hostname등을 정의해보려한다. ssh 공개키를 정의하는 PC에서 VM에 SSH 접속 시 사용하기 위해서이다.

```yaml
#cloud-config
ssh_authorized_keys:
- {{ 본인이 SSH 접속 시 사용하는 비밀키에 대한 공개키 }}

password: ubuntu
```

`user-data` 파일을 위와 같이 작성해줬다. SSH 공개키는 나의 경우 `ssh-ed25519 FooBar(생략) Jinsu` 뭐 이런 식으로 생겼다.
주의할 점은 첫 문장은 주석일지라도 `#cloud-config`로 적어줘야 인식이 된다는 점이다! 그냥 user-data, cloud-init 스펙이 그렇다.

```yaml
local-hostname: vm1
```

앞서 언급한 대로 VM의 이름은 vm1을 사용할 것이므로 hostname을 vm1으로 변경해봤다.

그럼 이제 `user-data`와 `metadata`라는 두 파일을 볼륨으로서 사용할 수 있게 해주는 이미지를 만들어줘야한다.
이때에는 cloud-localds라는 도구를 이용하는 것이 편리했다. Cloud Init 설정용 볼륨이라는 것을 인식하기 위해서는 해당 볼륨에 CI 등의 라벨이 달려있어야하며 몇몇의 특정 파일 시스템을 이용해야한다고 한다.
cloud-localds는 그런 조건을 만족시키는 볼륨의 이미지를 알아서 잘 만들어준다.

```shell
$ apt install -y cloud-image-utils
$ cloud-localds seed.img user-data metadata
```

해당 이미지의 이름은 `seed.img`라고 지어줬다. 이 이미지는 `user-data`와 `metadata` 두 파일만을 포함하는 볼륨을 나타낼 것이다.
좀 더 확인해보고 싶다면 다음 명령어로 확인해볼 수 있었다.

```shell
$ mkdir -p /mnt/vm1-seed

$ mount seed.img /mnt/vm1-seed

$ ls -al /mnt/vm1-seed
total 7
dr-xr-xr-x 2 root root 2048 Dec  7 14:43 .
drwxr-xr-x 7 root root 4096 Dec  7 14:44 ..
-rw-r--r-- 1 root root   20 Dec  7 14:43 meta-data
-rw-r--r-- 1 root root  144 Dec  7 14:43 user-data

# 다시 umount하고 /mnt/vm1-seed 디렉토리 삭제
$ umount seed.img

$ rm -r /mnt/vm1-seed

$ umount /mnt

$ lsblk -f
root@whale-home:~/vm1# lsblk -f
NAME                    FSTYPE      FSVER            LABEL  UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
loop0                   iso9660     Joliet Extension cidata 2022-12-07-14-43-10-00                       0   100% /mnt/seed
(...생략)
sda
├─sda1                  vfat        FAT32                   B125-AD84                                   1G     0% /boot/efi
├─sda2                  ext4        1.0                     a472b2ba-5b46-4495-8dc6-abe59a2e5153      1.5G    13% /boot
└─sda3                  LVM2_member LVM2 001                D94K0G-n2UY-sxNu-nw23-962h-ZVSa-njxMOk
  └─ubuntu--vg-ubuntu--lv ext4        1.0                     6f3fa5e0-59bb-4cea-97e0-aa86b7cb68c7     53.9G    40% /
```

## 1차로 VM 띄워보기

이제 네트워크 설정만 남긴했는데 일단 우선 한 번 정도는 VM을 뜨기는 하는지를 확인해보려한다.

```shell
# qs는 alias qs=qemu-system-x86_64 로 설정된 alias이다.
$ qs -machine accel=kvm,type=q35 \
  -cpu host \
  -smp 2 \
  -m 4G \
  --nographic \
  -drive if=virtio,format=qcow2,file=ubuntu-22.04-server-cloudimg-amd64.img \
  -drive if=virtio,format=raw,file=seed.img
(...생략)
Ubuntu 22.04.1 LTS vm1 ttyS0

vm1 login: ubuntu
password: 
```

인자가 의미하는 바는 대충 이곳 링크[[2]](https://www.arthurkoziel.com/qemu-ubuntu-20-04/)에 잘 정리되어있었다.
혹은 그냥 qemu-system-x86_64 --help 에 나오는 인자 설명을 봐도 대충은 이해는 된다. 몇몇 인자에 대한 설명은 좀 이해가 안 돼서 따로 찾아봐야하긴 하지만..

대충 로그인하고 패스워드를 초기화하고 나면 쉘에 접속하게 된다.

```shell
df -h /dev/vda1
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        16G  1.5G   14G  10% /
```

로그인해서 루트 파일시스템의 크기를 조회해보니 우리가 `.img` 파일을 resize한 대로 16G로 잘 늘어나서 설정되어있는 모습이다!

```shell
ubuntu@vm1:~$ ping google.com -c 1
PING google.com (142.250.66.78) 56(84) bytes of data.
64 bytes from hkg12s27-in-f14.1e100.net (142.250.66.78): icmp_seq=1 ttl=255 time=44.9 ms

--- google.com ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 44.868/44.868/44.868/0.000 ms
```

Google에 ping을 때려보니 인터넷 통신은 가능한 듯하긴하지만 호스트가 사용 중인 공유기를 통해
같은 네트워크를 이용하고 있는 상태는 아니다. 즉, 호스트 -> VM 혹은 공유기 네트워크 -> VM으로의 접근은 불가능한 상태이다.
따라서 추가적인 네트워크 설정이 필요하다.

host로 돌아가려면 VM을 종료시키면 된다. 프로세스를 kill해도 되긴 하지만 `sudo shutdown -h now` 명령어를 통해 VM 내부에서 안전히 종료할 수도 있다.

참고) 참고로 VM에서 네트워크 통신이 원활하지 않은 경우 `sudo sysctl net.ipv4.ip_forward=1`로 packet forward를 활성화시켜줘야할 수 있다고한다.
나의 경우에는 이 값과 무관하게 잘 동작하긴했다.

## 브릿지 네트워크 설정하기

이제 네트워크 설정만 해주면 된다. 나 같은 경우에는 VM이 호스트와 동일한 공유기 네트워크를 이용하도록 하여 VM의 인터넷 통신, VM <-> 호스트 머신 간의 통신, 그리고 VM <-> 공유기 내의 디바이스들 간의 통신이 가능하도록 하고 싶다.
따라서 퍼블릭 브릿지 방식을 이용할 것이다.

하지만 이 부분이 VirtualBox를 썼을 때와는 달리 처음 경험해보는 사람에게는 약간 지옥일 수 있다. 마음을 열고 Tap network, Bridge 같은 내용을 공부해봐야 이해하고 작업이 가능하다.
VirtualBox를 이용하든 qemu를 이용하는 원리는 같긴하다. bridge용 interface를 생성하고 호스트가 공유기랑 사용하던 인터페이스를
브릿지용 인터페이스에 묶어주고 VM은 Tap 네트워크의 장치를 이용해 자신의 네트워크 인터페이스 카드를 가상화하는 듯하다. 완전 정확히 공부해보진 않았지만 대충 흐름은 그렇다.
VirtualBox는 이런 과정을 알아서 수행해주는 듯하지만 qemu는 그렇지 않으므로 직접 구성해야한다.

브릿지 네트워크를 구성하는 방법은 호스트의 환경마다 상이할 것이다. 나같은 경우에는 호스트 머신 또한 Ubuntu Server 22.04이므로 
이를 바탕으로 적어보겠다. 잘못된 작업은 호스트 머신의 네트워크에 장애를 낳을 수 있으니 백업을 떠놓거나 네트워크 없이도 얼마든지 접속이 가능한
상황에서 작업을 하는 것을 추천한다.

```yaml
# /etc/netplan/00-installer-config-wifi.yaml
# 기존 설정
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
      dhcp6: false
      match:
        macaddress: 00:e0:4c:36:3f:c6
      set-name: eth0
  wifis:
    wlp2s0:
      access-points:
        {{ YOUR AP NAME }}:
          password: {{ YOUR AP PASSWORD }}
      dhcp4: true
      dhcp6: false

```
```shell
$ ifconfig eth0
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.219.183  netmask 255.255.255.0  broadcast 192.168.219.255
        (...생략)
```

기존에는 위와 같이 공유기와 이더넷으로 연결된 인터페이스가 IP를 갖고 있는 모습이다.
이제 이 인터페이스에 대한 DHCP를 비활성화 시키고 bridge 인터페이스로 브릿지될 인터페이스 목록에 이 인터페이스(`eth0`)를 추가해주면
eth0의 IP는 사라지고 브릿지 인터페이스가 IP를 발급받게 될 것이다.

```yaml
# /etc/netplan/00-installer-config-wifi.yaml
# 변경한 설정
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      dhcp6: false
      match:
        macaddress: 00:e0:4c:36:3f:c6
      set-name: eth0
  wifis:
    wlp2s0:
      access-points:
        {{ YOUR AP NAME }}:
          password: {{ YOUR AP PASSWORD }}
      dhcp4: true
      dhcp6: false
  bridges:
    br0:
      dhcp4: true
      dhcp6: false
      interfaces: [eth0]
```

그를 위해 위와 같이 netplan 설정을 변경해줬다. eth0의 DHCP를 끄고 브릿지 인터페이스를 추가해줬다.

```shell
$ sudo netplan apply
$ sudo systemctl restart systemd-networkd
```

설정을 적용해줬다.

```shell
$ ifconfig eth0
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        ether 00:e0:4c:36:3f:c6  txqueuelen 1000  (Ethernet)
        RX packets 431  bytes 48373 (48.3 KB)
        RX errors 0  dropped 4  overruns 0  frame 0
        TX packets 438  bytes 68498 (68.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        
$ ifconfig br0
br0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.219.121  netmask 255.255.255.0  broadcast 192.168.219.255
        inet6 fe80::a0e8:49ff:fe15:547c  prefixlen 64  scopeid 0x20<link>
        ether a2:e8:49:15:54:7c  txqueuelen 1000  (Ethernet)
        RX packets 150  bytes 14431 (14.4 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 94  bytes 14486 (14.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

브릿지가 잘 적용된 모습이다. `brctl`이라는 브릿지 관리 도구로도 정보를 확인해볼 수 있다.

```shell
$ sudo apt install -y bridge-utils

$ brctl show
bridge name	bridge id		STP enabled	interfaces
br0		8000.a2e84915547c	no		eth0
```

이제 브릿지 네트워크 설정이 잘 된 듯하다. VM을 다시 띄워봤다.

```shell
$ qs -machine accel=kvm,type=q35 \
  -cpu host \
  -smp 2 \
  -m 4G \
  --nographic \
  -drive if=virtio,format=qcow2,file=ubuntu-22.04-server-cloudimg-amd64.img \
  -drive if=virtio,format=raw,file=seed.img \
  -nic bridge,br=br0,model=virtio-net-pci
```

VM을 띄우되 bridge network 설정을 위해 앞서 VM을 띄웠던 명령어에서 `-nic bridge,br=br0,model=virtio-net-pci` 인자만 추가해줬다.

```shell
ubuntu@vm1:~$ ip addr show enp0s2
2: enp0s2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:12:34:56 brd ff:ff:ff:ff:ff:ff
    inet 192.168.219.185/24 metric 100 brd 192.168.219.255 scope global dynamic enp0s2
       valid_lft 21557sec preferred_lft 21557sec
    inet6 fe80::5054:ff:fe12:3456/64 scope link
       valid_lft forever preferred_lft forever
       
ubuntu@vm1:~$ ping 192.168.219.180 -c 1
PING 192.168.219.180 (192.168.219.180) 56(84) bytes of data.
64 bytes from 192.168.219.180: icmp_seq=1 ttl=64 time=2.89 ms

--- 192.168.219.180 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 2.887/2.887/2.887/0.000 ms
```

호스트가 이용 중인 `192.168.219.0/24` 네트워크의 IP를 이용하고 있으며 현재 내 호스트 머신의 IP인 192.168.219.180로 Ping도 가능하다.

```shell
$ brctl show
bridge name	bridge id		STP enabled	interfaces
br0		8000.a2e84915547c	no		eth0
							tap0

$ ifconfig tap0
tap0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::fc35:9dff:fe3f:86b0  prefixlen 64  scopeid 0x20<link>
        ether fe:35:9d:3f:86:b0  txqueuelen 1000  (Ethernet)
        (...생략)
```

참고) 참고로 vm이 실행될 때는 자동으로 이렇게 아까 생성한 bridge interface에 tap network interface가 묶인다. 따라서 br0에 eth0를 추가해줬던 것처럼 수동으로 tap0 인터페이스를 만들고 추가해주지는 않아도 된다.

## 최종적으로 VM을 백그라운드에서 띄우기

```shell
$ qs -machine accel=kvm,type=q35 \
    -name vm1 \
    -cpu host \
    -smp 2 \
    -m 4G \
    -daemonize \
    -display none \
    -drive if=virtio,format=qcow2,file=ubuntu-22.04-server-cloudimg-amd64.img \
    -drive if=virtio,format=raw,file=seed.img \
    -nic bridge,br=br0,model=virtio-net-pci
```

`-nograhic` -> `-display none` 으로 변경하고 `-daemonize` 인자를 통해 백그라운드에서 돌 수 있게 해줬다.

🎉 이렇게 QEMU로 VM 띄워보기 완료~!

## 배운 것, 얻은 것

* QEMU가 뭔지, KVM이 뭔지, 어떻게 쓰는 건지
* 브릿지 네트워크 설정 방법
* TAP 네트워크 개념
* 가상 이미지 타입(RAW, QCOW2)

## 글에서 다룬 내용 외에 더 해볼 만한 것

* `-daemonize`를 통해 백그라운드에서 VM을 돌리도록 하긴했지만 호스트가 재부팅되거나 
  해당 qemu-system-x86_64 프로세스가 죽으면 VM이 재시작되지 못하는 형태인데 이를 개선해볼 수 있을 것이다.
* QEMU를 직접 사용하는 것은 한 번 해봤으니 이를 좀 더 편리하게 쓸 수 있는 proxmox나 virtsh 같은 도구들이 있는 것 같은데 실제로
  VM을 통해 홈랩을 구축하려면 굳이 수작업으로 하거나 직접 만들지 않고 이런 도구들을 도입해보는 게 좋을 듯. 

## 마치며

VirtualBox를 CLI로 이용했을 때에는 VirtualBox가 CLI 친화적인 느낌은 아니었던 것도 있고, 클라우드 이미지가 아닌 단순 live 이미지로 이용하다보니
VM 띄우는 것 자체는 그리 간단하게 느껴지지 않았던 반면 네트워크 설정은 VirtualBox가 알아서 잡아주는 게 많으니 되게 간단하다고 느껴졌다.

반면 QEMU는 CLI 친화적인 데다가 이번에는 클라우드 이미지를 써봤더니 VM을 띄우는 것 자체는 엄청나게 간단했다. 반면
브릿지 네트워크를 설정하는 게 처음이다보니 쉽지는 않았던 것 같다.

## 참고 자료

_[[1] - VBoxManage(VirtualBox CLI)로 VM 생성하기](../create-vm-by-vboxmanage/)_

_[[2] - Using QEMU to create a Ubuntu 20.04 Desktop VM on macOS](https://www.arthurkoziel.com/qemu-ubuntu-20-04/)_

* https://levelup.gitconnected.com/how-to-setup-bridge-networking-with-kvm-on-ubuntu-20-04-9c560b3e3991 - 브릿지 설정법을 참고함
* https://superuser.com/questions/1490188/what-is-the-difference-and-relationship-between-kvm-virt-manager-qemu-and-libv/1490196#1490196 - KVM과 QEMU의 차이를 간단히 알려줌.
* https://www.vinchin.com/en/blog/raw-vs-qcow2.html - RAW 이미지 타입과 QCOW2 이미지 타입에 대한 내용이 잘 정리됨
* https://www.joinc.co.kr/w/Site/cloud/Qemu/Network - qemu 에서 다양한 네트워크 설정을 이용하는 예시를 보여줌
* https://www.qemu.org/2018/05/31/nic-parameter/ - net, netdev, device, nic 등 헷갈리는 qemu 실행 인자를 정리해줌
* https://skylit.tistory.com/215 - 구버전 우분투에 대한 bridge 설정인듯함
* https://ubuntu.com/server/docs/virtualization-libvirt - libvirt, virsh 등의 내용을 훑어봄
* https://www.snel.com/support/nstall-qemu-guest-agent-for-debian-ubuntu/ - qemu-guest-agent에 대한 내용을 잠깐 살펴봄
