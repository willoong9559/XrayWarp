现在机场大多数使用的是Dnsmasq将网站解析劫持到SNI proxy反向代理的页面上。

这里是使用WARP Client进行代理解锁Netflix等。
可以去cloudfalre官方页面有详细的安装流程和原理，不赘述。
个人认为官方socks这种代理方式更灵活且优雅。
https://developers.cloudflare.com/warp-client/setting-up/linux

这里写下我的配置过程

1.注册客户端
```
warp-cli register
```
2.设置WARP代理模式
```
warp-cli set-mode proxy
```
3.连接WARP
```
warp-cli connect
```
此时WARP会使用socks5本机代理127.0.0.1：40000
4.打开warp always-on
```
warp-cli enable-always-on
```
6.测试socks代，理检查ip是否改变
```
export ALL_PROXY=socks5://127.0.0.1:40000
curl ifconfig.me
```
7.修改v2ray/xray outbounds和分流规则，这里给出我的配置可自由发挥。
```
vim /usr/local/etc/xray/config.json
```
```
 "outbounds": [
        {
            "tag": "default",
            "protocol": "freedom"
        },
        {
            "tag":"socks_out",
            "protocol": "socks",
            "settings": {
                "servers": [
                     {
                        "address": "127.0.0.1",
                        "port": 40000
                    }
                ]
            }
        }
    ],
    "routing": {
        "rules": [
            {
                "type": "field",
                "outboundTag": "socks_out",
                "domain": ["geosite:netflix"]
            },
            {
                "type": "field",
                "outboundTag": "default",
                "network": "udp,tcp"
            }
        ]
    }
```
8.重新启动v2ray/xray
```
systemctl restart v2ray/xray
systemctl status v2ray/xray
```
xray可能需要下载geosite geoip
google github上就能找到，wget url /usr/local/bin

