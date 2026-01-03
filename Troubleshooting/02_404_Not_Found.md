# Docker 저장소 404 Not Found 및 패키지 설치 에러

## Problem

* **omv-release-upgrade 진행 중 또는 완료 후 apt update 시 Docker 공식 저장소에서 특정 패키지를 찾지 못해 에러 발생**  

* **업그레이드 실패가 아닌, 공식 저장소의 문제**  

```Plain Text
E: Failed to fetch https://download.docker.com/linux/debian/dists/trixie/pool/stable/amd64/containerd.io_2.2.1-1~debian.13~trixie_amd64.deb  404  Not Found [IP: 18.244.60.126 443]
E: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?
```

## Cause

* **Debian 13(Trixie)용 Docker 저장소 메타데이터에는 2.2.1-1 버전이 등록되어 있으나, 실제 해당 경로에 .deb 파일이 업로드되지 않아 발생하는 서버 측 문제.**  

## Solution

* **저장소 서버에 실재하는 안정적인 하위 버전(2.2.0-2)을 수동으로 지정하여 설치한 뒤, 추가 업데이트를 방지하기 위해 버전을 고정** 

* **[저장소](https://download.docker.com/linux/debian/dists/trixie/pool/stable/amd64/)에 실재하는 2.2.0-2 버전 지정 설치**  

![Repo](./Images/02_1.png)

```Bash
sudo apt install containerd.io=2.2.0-2~debian.13~trixie
```

* **패키지 버전 고정 - 다시 2.2.1 버전으로 업데이트 시도하는 것을 방지**

```Bash
sudo apt-mark hold containerd.io
```

## Future

* **패키지 고정 해제 (Unhold)**  

```Bash
sudo apt-mark unhold containerd.io
```

* **고정 해제 확인**  

```Bash
apt-mark showhold
#목록에 containerd.io가 없어야 정상
```

* **패키지 업그레이드**  

```Bash
sudo apt update && sudo apt upgrade -y
```