# Migration: OpenMediaVault 7 to 8 with ZFS & PVE Kernel

本文書は、ZFSをメインファイルシステムとして使用するOpenMediaVault (OMV) サーバーのメジャーアップグレードプロセスを記録したものです。  
ZFSはカーネルへの依存度が高いため、アップグレード中のカーネルモジュール(DKMS)の競合を防ぐために、プラグインを一度削除してから再インストールする戦略を採用しました。

## 🛠️ Environment

- **OS: Debian 12 (Bookworm) with OMV 7**  
- **Kernel:** 6.11.11-2-pve
- **Filesystem:** ZFS
- **Key Services:** Services on Docker, SMB, SFTP

## 🎯 Target

**Debian 12 with OMV 7 システムを Debian 13 with OMV 8 へアップグレード**  
**Proxmox VE 8 (6.11) Kernel を Proxmox VE 9 (6.14) Kernel へアップグレード**  

## 🗿 Migration Strategy

**ZFSカーネルモジュールと新しいOSカーネル間の依存関係の問題を解決するため、以下の手順で進行します。**  

1. [サービスおよびデータの隔離: SMB、SFTPおよびDockerコンテナの停止](#phase-1-サービスおよびデータの隔離)

2. [ZFS保護: プール(Pool)のアンマウントおよびExportによるデータ整合性確保](#phase-2-zfs保護)

3. [依存関係の削除: アップグレード中のDKMSエラーを防ぐためのopenmediavault-zfsプラグインの先行削除](#phase-3-依存関係の削除)

4. [OSリリースアップグレード: OMV 8およびDebian 13への移行](#phase-4-osリリースアップグレード)

5. [インフラ再構築: 最新PVEカーネル(6.14)のインストールおよびZFS環境の復旧](#phase-5-インフラ再構築)

### Phase 1. サービスおよびデータの隔離

**アップグレード中のデータ書き込み作業を遮断し、システムリソースを確保します。**  

* **SMB/SFTPサービスの停止:** Web GUIを使用して共有サービスを無効化  

* **Dockerコンテナの停止:** ZFSデータセットをマウントしているコンテナを終了  

```bash
# Docker Composeを使用している場合
docker-compose down
# Docker Container
docker stop <CONTAINER_1> <CONTAINER_2> <CONTAINER_3> ...
```

### Phase 2. ZFS保護

**カーネルモジュールのアップデート時に発生する可能性のあるデータ破損を防ぐため、プール(Pool)を安全に切り離します。**  

* **ZFS Pool Export:** すべてのプールをアンマウントし、システムから安全にエクスポート状態へ移行  

```bash
sudo zpool export -a
```

### Phase 3. 依存関係の削除

**メジャーアップグレード中に旧バージョンのZFSモジュールと新しいカーネル間で発生するビルド競合(DKMS Error)を防止します。**  

* **ZFSプラグインの削除:** 設定値は維持されますが、カーネルモジュールの依存関係を断つためにプラグインを削除します  

```bash
# ZFSプラグイン削除
sudo apt remove openmediavault-zfs
# 依存パッケージ削除
sudo apt autoremove
```

### Phase 4. OSリリースアップグレード

**Debian 12(Bookworm)ベースからDebian 13(Trixie)ベースのOMV 8へ移行します。**  

* **Release Upgrade:** OMV公式アップグレードツールを実行  

```bash
sudo omv-release-upgrade
```

* **すべてのプロセス完了後、システム再起動**  
```bash
sudo reboot now
```

* **OMV-Extrasリポジトリの更新:** 新しいOSバージョン(Synchrony)に合わせてプラグインリポジトリを再設定  

```bash
wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash
```

* **[Troubleshooting: Ping Permission Issue](./Troubleshooting/00_Ping_Permission_Issue.md)**  

* **[Troubleshooting: Network Issue](./Troubleshooting/01_Network_Issue.md)**  

### Phase 5. インフラ再構築

**新しいシステム環境に合わせて、最新のカーネルとストレージ環境を復旧します。**

* **システムアップデート**  

```bash
sudo apt update && sudo apt upgrade -y
```

* **[Troubleshooting: Docker Repository 404 Not Found](./Troubleshooting/02_404_Not_Found.md)**  
    > **Note:** 2026年1月3日時点で、該当の問題は発生しないことを確認

* **PVE 9 (6.14) カーネルインストール:** `Web GUI System > Kernel`からProxmox VE Kernelをインストール (必要に応じてPVE 9.1 / 6.17をインストール) し、再起動  

* **ZFSプラグイン再インストール:** OMV 8用ZFSプラグインを再インストールし、新しいカーネルに合わせたモジュールをビルド  

```bash
sudo apt install openmediavault-zfs
```

* **ZFS Pool Import:** エクスポートしていたプールを再度システムに読み込み  

```bash
# ZFS Pool検索 (すべてのZFS Poolが表示されるか確認)
sudo zpool import
# 特定のZFS Pool Import
sudo zpool import <POOL_NAME>
# すべてのZFS Pool Import
sudo zpool import -a
```

* **起動シンボリックリンクの修正:** `/vmlinuz`および`/initrd.img`が旧バージョンのカーネルを指さないように手動更新  

```bash
sudo ln -sf /boot/vmlinuz-6.14.11-5-pve /vmlinuz
sudo ln -sf /boot/initrd.img-6.14.11-5-pve /initrd.img
sudo update-grub
```
