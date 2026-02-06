# Ping Command Permission Error Immediately After Upgrade

* **Execute `ping`**  

```bash
ping google.com
```

* **Check Error Message**  

```text
ping: socktype: SOCK_RAW
ping: socket: Operation not permitted
ping: => missing cap_net_raw+p capability or setuid?
```

* **Grant necessary capabilities to `ping` executable**  

```bash
sudo setcap cap_net_raw+p /usr/bin/ping
```

* **Verify Permission Settings**  

```bash
sudo getcap /usr/bin/ping
#Output: /usr/bin/ping cap_net_raw=p
```

* **Verify Normal Execution**  

```text
PING google.com (142.250.196.14) 56(84) bytes of data.
64 bytes from maa03s44-in-f14.1e100.net (142.250.196.14): icmp_seq=1 ttl=116 time=34.6 ms
64 bytes from maa03s44-in-f14.1e100.net (142.250.196.14): icmp_seq=2 ttl=116 time=35.0 ms
64 bytes from maa03s44-in-f14.1e100.net (142.250.196.14): icmp_seq=3 ttl=116 time=35.0 ms
```
