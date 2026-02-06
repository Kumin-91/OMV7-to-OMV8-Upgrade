# DNS Issue During OMV-Extras Repository Update

* **Domain resolution failure when running `wget` (Name or service not known)**  

```text
--2026-01-02 22:25:25--  https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install
Resolving github.com (github.com)... failed: Name or service not known.
wget: unable to resolve host address ‘github.com’
```

* **Reset network interface using OMV management tool** 

```bash
sudo omv-firstaid
```

* **Select `1. Configure network interface` menu and proceed**
