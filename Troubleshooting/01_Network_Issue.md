# OMV-Extras 리포지토리 갱신 중 DNS 이슈

* **wget 명령어 실행 시 도메인 해석 실패 (Name or service not known)**  

```Plain Text
--2026-01-02 22:25:25--  https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install
Resolving github.com (github.com)... failed: Name or service not known.
wget: unable to resolve host address ‘github.com’
```

* **OMV의 관리 도구를 이용하여 인터페이스 설정을 다시 수행** 

```Bash
sudo omv-firstaid
```

* **1. Configure network interface 선택 후 단계별 설정 진행**