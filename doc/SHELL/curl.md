#### 强制curl走shadowsocks的代理  并且不依赖于Proxychains-ng

编辑`~/.curlrc`:

`proxy=socks5://127.0.0.1:1086`

`1086` 是Shadowsocks-NG的代理端口 若是ShadowsocksX 则是1080
