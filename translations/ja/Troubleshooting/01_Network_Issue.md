# OMV-Extrasリポジトリ更新中のDNS問題

* **`wget` 実行時のドメイン解決失敗 (Name or service not known)**  

```text
--2026-01-02 22:25:25--  https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install
Resolving github.com (github.com)... failed: Name or service not known.
wget: unable to resolve host address ‘github.com’
```

* **OMV管理ツールでネットワークインターフェース再設定** 

```bash
sudo omv-firstaid
```

* **`1. Configure network interface` メニュー選択後、設定進行**
