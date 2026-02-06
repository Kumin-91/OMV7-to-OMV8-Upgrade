# Dockerリポジトリ404 Not Foundおよびパッケージインストールエラー

## Problem

* **`omv-release-upgrade` 進行中または完了後の `apt update` 時に、Docker公式リポジトリから特定パッケージが見つからずエラー発生**  

* **アップグレード失敗ではなく、公式リポジトリの一時的な問題**  

```text
E: Failed to fetch https://download.docker.com/linux/debian/dists/trixie/pool/stable/amd64/containerd.io_2.2.1-1~debian.13~trixie_amd64.deb  404  Not Found [IP: 18.244.60.126 443]
E: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?
```

## Cause

* **Debian 13 (Trixie)用Dockerリポジトリのメタデータにはバージョン2.2.1-1が登録されているが、実際にはそのパスに `.deb` ファイルがアップロードされていないために発生するサーバー側の問題**  

## Solution

* **リポジトリに実在する安定した下位バージョン(2.2.0-2)を手動インストール後、バージョンを固定** 

* **[リポジトリ](https://download.docker.com/linux/debian/dists/trixie/pool/stable/amd64/)に存在するバージョン2.2.0-2を指定してインストール**  

![Repo](/Images/02_1.png)

```bash
sudo apt install containerd.io=2.2.0-2~debian.13~trixie
```

* **パッケージバージョン固定 (自動アップデート防止)**

```bash
sudo apt-mark hold containerd.io
```

## Future

* **パッケージ固定解除**  

```bash
sudo apt-mark unhold containerd.io
```

* **固定解除確認**  

```bash
apt-mark showhold
#出力: リストにcontainerd.ioがなければ正常
```

* **パッケージアップグレード**  

```bash
sudo apt update && sudo apt upgrade -y
```
