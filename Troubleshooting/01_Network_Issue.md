# OMV-Extras 리포지토리 갱신 중 DNS 이슈

* **`wget` 실행 시 도메인 해석 실패 (Name or service not known)**  

```text
--2026-01-02 22:25:25--  https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install
Resolving github.com (github.com)... failed: Name or service not known.
wget: unable to resolve host address ‘github.com’
```

* **OMV 관리 도구로 네트워크 인터페이스 재설정** 

```bash
sudo omv-firstaid
```

* **`1. Configure network interface` 메뉴 선택 후 설정 진행**