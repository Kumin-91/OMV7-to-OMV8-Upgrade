# Docker Repository 404 Not Found and Package Installation Error

## Problem

* **Error occurs during `apt update` while `omv-release-upgrade` is in progress or completed, failing to find specific packages in Docker official repository**  

* **Not an upgrade failure, but a temporary issue with the official repository**  

```text
E: Failed to fetch https://download.docker.com/linux/debian/dists/trixie/pool/stable/amd64/containerd.io_2.2.1-1~debian.13~trixie_amd64.deb  404  Not Found [IP: 18.244.60.126 443]
E: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?
```

## Cause

* **Server-side issue where version 2.2.1-1 is registered in Docker repository metadata for Debian 13 (Trixie), but the `.deb` file is not actually uploaded to that path**  

## Solution

* **Manually install a stable lower version (2.2.0-2) present in the repository, then hold the version** 

* **Install version 2.2.0-2 present in the [Repository](https://download.docker.com/linux/debian/dists/trixie/pool/stable/amd64/)**  

![Repo](/Images/02_1.png)

```bash
sudo apt install containerd.io=2.2.0-2~debian.13~trixie
```

* **Hold Package Version (Prevent automatic updates)**

```bash
sudo apt-mark hold containerd.io
```

## Future

* **Unhold Package**  

```bash
sudo apt-mark unhold containerd.io
```

* **Verify Unhold**  

```bash
apt-mark showhold
#Output: containerd.io should not be in the list
```

* **Package Upgrade**  

```bash
sudo apt update && sudo apt upgrade -y
```
