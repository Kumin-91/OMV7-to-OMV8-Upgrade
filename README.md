# Migration: OpenMediaVault 7 to 8 with ZFS & PVE Kernel

본 문서는 ZFS를 주 파일시스템으로 사용하는 OpenMediaVault(OMV) 서버의 메이저 업그레이드 과정을 기록합니다.  
ZFS는 커널 의존성이 높은 특성을 가지므로, 업그레이드 과정 중 커널 모듈(DKMS) 충돌을 방지하기 위해 플러그인 삭제 후 재설치 전략을 채택하였습니다.

## 🛠️ Environment

- **OS: Debian 12 (Bookworm) with OMV 7**  
- **Kernel:** 6.11.11-2-pve
- **Filesystem:** ZFS
- **Key Services:** Services on Docker, SMB, SFTP

## 🎯 Target

**Debian 12 with OMV 7 시스템을 Debian 13 with OMV8로 업그레이드**  
**Proxmox VE 8 (6.11) Kernel을 Proxmox VE 9 (6.14) Kernel로 업그레이드**  

## 🗿 Migration Strategy

**ZFS 커널 모듈과 신규 OS 커널 간의 의존성 문제를 해결하기 위해 다음과 같은 단계로 진행합니다.**  

1. [서비스 및 데이터 격리: SMB, SFTP 및 Docker 컨테이너 종료](#phase-1-서비스-및-데이터-격리)

2. [ZFS 보호: Pool 언마운트 및 Export를 통한 데이터 무결성 확보](#phase-2-zfs-보호)

3. [의존성 제거: 업그레이드 중 DKMS 에러 방지를 위한 openmediavault-zfs 플러그인 선제 삭제](#phase-3-의존성-제거)

4. [OS 릴리즈 업그레이드: OMV 8 및 Debian 13 마이그레이션](#phase-4-os-릴리즈-업그레이드)

5. [인프라 재구축: 최신 PVE 커널(6.14) 설치 및 ZFS 환경 복구](#phase-5-인프라-재구축)

### Phase 1. 서비스 및 데이터 격리

**업그레이드 중 데이터 쓰기 작업을 차단하고 시스템 자원을 확보합니다.**  

* **SMB/SFTP 서비스 중지:** Web GUI를 이용하여 공유 서비스를 비활성화  

* **Docker 컨테이너 정지:** ZFS 데이터셋을 마운트하는 컨테이너를 종료  

```Bash
#Docker Compose를 이용하는 경우
docker-compose down
#Docker Container
docker stop <CONTAINER_1> <CONTAINER_2> <CONTAINER_3> ...
```

### Phase 2. ZFS 보호

**커널 모듈 업데이트 시 발생할 수 있는 데이터 손상을 방지하기 위해 풀(Pool)을 안전하게 분리합니다.**  

* **ZFS Pool Export: 모든 풀을 언마운트하고 시스템으로부터 안전하게 내보내기 상태로 전환합니다.**  

```Bash
sudo zpool export -a
```

### Phase 3. 의존성 제거

**메이저 업그레이드 도중 구버전 ZFS 모듈과 새 커널 간의 빌드 충돌(DKMS Error)을 방지합니다.**  

* **ZFS 플러그인 삭제: 설정값은 유지되지만, 커널 모듈 의존성을 끊기 위해 플러그인을 제거합니다.**  

```Bash
#ZFS 플러그인 삭제
sudo apt remove openmediavault-zfs
#의존성 패키지 삭제
sudo apt autoremove
```

### Phase 4. OS 릴리즈 업그레이드

**Debian 12(Bookworm) 기반에서 Debian 13(Trixie) 기반의 OMV 8로 전환합니다.**  

* **Release Upgrade: OMV 공식 업그레이드 도구를 실행합니다.**  

```Bash
sudo omv-release-upgrade
````

* **모든 과정이 완료되면 시스템을 재부팅합니다.**  
```Bash
sudo reboot now
```

* **OMV-Extras 리포지토리 갱신: 새 OS 버전(Synchrony)에 맞는 플러그인 저장소를 재설정합니다.**  

```Bash
wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash
```

* **[Trouble Shooting: Ping Permission Issue](./Troubleshooting/00_Ping_Permission_Issue.md)**  

* **[Trouble Shooting: Network Issue](./Troubleshooting/01_Network_Issue.md)**  

### Phase 5. 인프라 재구축

**새로운 시스템 환경에 맞춰 최신 커널과 스토리지 환경을 복구합니다.**

* **시스템 업데이트**  

```Bash
sudo apt update && sudo apt upgrade -y
```

* **[Trouble Shooting: Docker Repository 404 Not Found](./Troubleshooting/02_404_Not_Found.md)**  
    * **2026년 01월 03일 해당 이슈는 더 이상 발생하지 않음을 확인**  

* **PVE 9 (6.14) 커널 설치: `Web GUI System > OMV-Extras > Kernel`에서 Proxmox VE 커널을 설치하고 재부팅합니다. 필요에 따라 PVE 9.1 (6.17)을 설치합니다.**  

* **ZFS 플러그인 재설치: OMV 8용 ZFS 플러그인을 다시 설치하여 새 커널에 맞는 모듈을 빌드합니다.**  

```Bash
sudo apt install openmediavault-zfs
```

* **ZFS Pool Import: 내보냈던 풀을 다시 시스템으로 불러옵니다.**  

```Bash
#ZFS Pool 검색 (모든 ZFS Pool이 뜨는지 확인)
sudo zpool import
#특정 ZFS Pool Import
sudo zpool import <POOL_NAME>
#모든 ZFS Pool Import
sudo zpool import -a
```

* **부팅 심볼릭 링크 수정: /vmlinuz 및 /initrd.img가 구버전 커널을 가리키지 않도록 수동 업데이트합니다.**  

```Bash
sudo ln -sf /boot/vmlinuz-6.14.11-5-pve /vmlinuz
sudo ln -sf /boot/initrd.img-6.14.11-5-pve /initrd.img
sudo update-grub
```