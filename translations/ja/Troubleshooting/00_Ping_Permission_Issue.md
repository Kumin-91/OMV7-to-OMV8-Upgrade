# アップグレード直後のPingコマンド権限エラー発生

* **`ping` 実行**  

```bash
ping google.com
```

* **エラーメッセージ確認**  

```text
ping: socktype: SOCK_RAW
ping: socket: Operation not permitted
ping: => missing cap_net_raw+p capability or setuid?
```

* **`ping` 実行ファイルに必要な権限(Capability)を付与**  

```bash
sudo setcap cap_net_raw+p /usr/bin/ping
```

* **権限設定の確認**  

```bash
sudo getcap /usr/bin/ping
#出力: /usr/bin/ping cap_net_raw=p
```

* **正常実行確認**  

```text
PING google.com (142.250.196.14) 56(84) bytes of data.
64 bytes from maa03s44-in-f14.1e100.net (142.250.196.14): icmp_seq=1 ttl=116 time=34.6 ms
64 bytes from maa03s44-in-f14.1e100.net (142.250.196.14): icmp_seq=2 ttl=116 time=35.0 ms
64 bytes from maa03s44-in-f14.1e100.net (142.250.196.14): icmp_seq=3 ttl=116 time=35.0 ms
```
