- [翻墙路由器的原理与实现](https://docs.google.com/document/d/1mmMiMYbviMxJ-DhTyIGdK7OOg581LSD1CZV4XY1OMG8/pub)
- [Laod.cn](https://laod.cn/)
- ss & pilipo

# polipo 设置全局代理

参考资料: [Ubuntu server命令行配置shadowsocks全局代理](https://jingsam.github.io/2016/05/08/setup-shadowsocks-http-proxy-on-ubuntu-server.html)

```sh
$ sudo pacman -S polipo  # arch install polipo
$ cat /etc/polipo/config
...
proxyAddress = "0.0.0.0"    # IPv4 only
...
socksParentProxy = "127.0.0.1:1080"
socksProxyType = socks5
...
chunkHighMark = 50331648
objectHighMark = 16384
$ sudo systemctl start polipo
$ export http_proxy="http://127.0.0.1:8123/"
$ curl www.google.com
```

## 注意事项

服务器重启后，如下命令需重新执行：

```sh
$ sudo systemctl start shadowsocks@config.service
$ export http_proxy="http://127.0.0.1:8213/"
```
