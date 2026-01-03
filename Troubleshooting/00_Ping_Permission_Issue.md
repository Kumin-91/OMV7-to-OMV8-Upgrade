# 업그레이드 직후 Ping 명령어 권한 에러 발생

* **`ping` 실행**  

```Bash
ping google.com
```

* **아래와 같은 에러 출력**  

```Plain Text
ping: socktype: SOCK_RAW
ping: socket: Operation not permitted
ping: => missing cap_net_raw+p capability or setuid?
```

* **`setcap` 명령어를 사용하여 `ping` 실행 파일에 필요한 권한 (Capability)을 수동으로 부여**  

```Bash
sudo setcap cap_net_raw+p /usr/bin/ping
```

* **권한 설정 확인**  

```Bash
sudo getcap /usr/bin/ping
#출력: /usr/bin/ping cap_net_raw=p
```

* **정상 실행**  

```Plain Text
PING google.com (142.250.196.14) 56(84) bytes of data.
64 bytes from maa03s44-in-f14.1e100.net (142.250.196.14): icmp_seq=1 ttl=116 time=34.6 ms
64 bytes from maa03s44-in-f14.1e100.net (142.250.196.14): icmp_seq=2 ttl=116 time=35.0 ms
64 bytes from maa03s44-in-f14.1e100.net (142.250.196.14): icmp_seq=3 ttl=116 time=35.0 ms
```