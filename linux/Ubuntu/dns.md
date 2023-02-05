## dnsmasqが起動しない


### Error description

```shell
dnsmasq: failed to create listening socket for port 53: Address already in use
failed to create listening socket for port 53: Address already in use
```

### 解決策

systemd-resolveがポートを占有していることが原因。

`sudo nmap -sS -sU -Pn -p 1-100` nmapでは反応しない類のものだった。別方法なら反応したかも。


```
systemctl stop systemd-resolved.service
systemctl start dnsmasq.service
```
