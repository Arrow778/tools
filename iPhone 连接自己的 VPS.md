# iPhone 连接自己的 VPS

由于经常使用Android，后来换了iOS 也开始慢慢习惯了，但是发现使用国际网络很不方便。于是乎在Gemini的帮助写下这篇教程。希望帮助到有需要的人。

在此之前，你需要：

> - iOS US账号
> - 会使用`Xshell`等可以连接VPS的工具
> - 拥有自己的VPS （且使用 Sing-Box）
> - 可用的国际网络
> - Windows 笔记本（因为需要开热点）
> - 或者一台ROOT了的Android手机。但是我手机没有root，如果你root了你可以去下载下面这个软件可以代替步骤一，具体用法得自己寻找其他教程，也不难使用。<img src="C:\Users\win11_leon\AppData\Roaming\Typora\typora-user-images\image-20260513193602748.png" alt="image-20260513193602748" style="zoom:33%;" />

关于转区，我的账号之前也一直是国区，后来转的美区。目前来说，只要你连接了US的国际网络，就很方便的转区，很可能不需要借记卡。

在进行操作之前，:warning:提醒你注意：

- 本教程仅供个人技术交流。请勿将其用于任何违反您所在地法律法规及网络服务商 (ISP) 条款的活动。
- 网络配置涉及系统底层修改，操作不当可能导致服务器 IP 被封禁、iOS账号、VPS 账号被暂停或数据丢失。作者不对因按照本教程操作产生的任何直接或间接后果负责。
- 本教程基于作者当时的实际操作经验编写，不保证在所有网络环境、硬件设备或软件版本下均完全有效。请在操作前做好数据备份。
- 转载请注明出处，并保留本免责声明。

如果你执行了这个教程，则说明同意以上条款。

---

我现在有两个办法转区：

1. 重新注册或者买一个美区账号
2. 使用自己的小号转区（但是需要梯子），不能保证100%使用。

我是使用第二个方法，但是这就要求你自己想办法让你的iPhone连接到国际网络。

# 步骤一

对于转区这有一计，如果你有ROOT安卓手机则需要参考其他教程。

在可以开热点的PC使用Clash Verge软件进行流量转发。

![image-20260513183420160](C:\Users\win11_leon\AppData\Roaming\Typora\typora-user-images\image-20260513183420160.png)

按照图中办法的顺序打开，然后一下命令查看本机的IPv4.

```bash
ipconfig
```

我这里是WLAN2，你们的可能与我的不一样

![](C:\Users\win11_leon\AppData\Roaming\Typora\typora-user-images\image-20260513183631436.png)

记住红色框中的IPv4地址，打开Verge的“系统代理”，然后连接到一个WIFI最好就连接到同一个网段下的WIFI。

然后打开手机的“设置”->“无线局域网”->“当前已连接的WIFI 蓝色 感叹号”->“配置代理”->"手动"。

依次填入IPv4和Verge的端口（可以在设置中查看，一般是7897，具体与你们的为主）。

最后“认证”不需要打开，之后保存。在iPhone打开浏览器输入google.com查看是否可以正常打开。

<img src="C:\Users\win11_leon\AppData\Roaming\Typora\typora-user-images\image-20260513194403982.png" alt="image-20260513194403982" style="zoom: 50%;" />

正常则进行转区，剩下的步骤是和网上步骤是一样的。

---

# 步骤二

转完区去后，连接你的VPS服务器输入`sb -n`，复制红色框框中的配置信息（绿色的`JSON`信息）。一般会有多个节点比如`TUIC`、`Hysteria2`、`Vless`等等，具体看你的节点开放了哪些。我这里是复制了`TUIC`节点的配置信息

<img src="C:\Users\win11_leon\AppData\Roaming\Typora\typora-user-images\image-20260513184750916.png" alt="image-20260513184750916" style="zoom:50%;" />

![image-20260513185416463](C:\Users\win11_leon\AppData\Roaming\Typora\typora-user-images\image-20260513185416463.png)

复制好完毕后，将上图的内容替换下面JSON的outbounds

```json
{
    "log": {
        "level": "info",
        "timestamp": true
    },
    "dns": {
        "servers": [
            {
                "tag": "dns_proxy",
                "address": "tcp://1.1.1.1",
                "address_resolver": "dns_resolver",
                "strategy": "ipv4_only",
                "detour": "proxy"
            },
            {
                "tag": "dns_direct",
                "address": "https://dns.alidns.com/dns-query",
                "address_resolver": "dns_resolver",
                "strategy": "ipv4_only",
                "detour": "direct"
            },
            {
                "tag": "dns_resolver",
                "address": "223.5.5.5",
                "detour": "direct"
            },
            {
                "tag": "dns_success",
                "address": "rcode://success"
            },
            {
                "tag": "dns_refused",
                "address": "rcode://refused"
            },
            {
                "tag": "dns_fakeip",
                "address": "fakeip"
            }
        ],
        "rules": [
            {
                "outbound": "any",
                "server": "dns_resolver"
            },
            {
                "rule_set": "geosite-geolocation-!cn",
                "query_type": [
                    "A",
                    "AAAA"
                ],
                "server": "dns_fakeip"
            },
            {
                "rule_set": "geosite-geolocation-!cn",
                "query_type": [
                    "CNAME"
                ],
                "server": "dns_proxy"
            },
            {
                "query_type": [
                    "A",
                    "AAAA",
                    "CNAME"
                ],
                "invert": true,
                "server": "dns_refused",
                "action": "route-options",
                "disable_cache": true
            }
        ],
        "final": "dns_direct",
        "independent_cache": true,
        "fakeip": {
            "enabled": true,
            "inet4_range": "198.18.0.0/15",
            "inet6_range": "fc00::/18"
        }
    },
    "route": {
        "rule_set": [
            {
                "tag": "geosite-geolocation-!cn",
                "type": "remote",
                "format": "binary",
                "url": "https://raw.githubusercontent.com/SagerNet/sing-geosite/rule-set/geosite-geolocation-!cn.srs",
                "download_detour": "direct"
            },
            {
                "tag": "geoip-cn",
                "type": "remote",
                "format": "binary",
                "url": "https://raw.githubusercontent.com/SagerNet/sing-geoip/rule-set/geoip-cn.srs",
                "download_detour": "direct"
            }
        ],
        "rules": [
            {
                "inbound": "tun-in",
                "action": "sniff"
            },
            {
                "protocol": "dns",
                "action": "hijack-dns"
            },
            {
                "port": 853,
                "network": "tcp",
                "action": "reject"
            },
            {
                "port": 443,
                "network": "udp",
                "action": "reject"
            },
            {
                "rule_set": "geosite-geolocation-!cn",
                "outbound": "proxy"
            },
            {
                "rule_set": "geoip-cn",
                "outbound": "direct"
            },
            {
                "ip_is_private": true,
                "outbound": "direct"
            }
        ],
        "final": "proxy",
        "auto_detect_interface": true
    },
    "inbounds": [
        {
            "type": "tun",
            "tag": "tun-in",
            "address": [
                "172.16.0.1/30",
                "fd00::1/126"
            ],
            "mtu": 1492,
            "auto_route": true,
            "strict_route": true,
            "stack": "system"
        }
    ],
    "outbounds": [你需要替换的地方],
    "experimental": {
        "cache_file": {
            "enabled": true,
            "path": "cache.db",
            "store_fakeip": true,
            "store_rdrc": true
        }
    }
}
```

替换好后，保存为`.json`格式。名字随便但最好还是不要中文名。

将保存好的json文件发到手机上，然后去Store下载Sing-box VT。

<img src="C:\Users\win11_leon\AppData\Roaming\Typora\typora-user-images\image-20260513191150779.png" alt="image-20260513191150779" style="zoom:50%;" />

打开SingBox VT软件，按照下面顺序操作：

![image-20260513191501862](C:\Users\win11_leon\AppData\Roaming\Typora\typora-user-images\image-20260513191501862.png)

最后最后在仪表启用即可。

<img src="C:\Users\win11_leon\AppData\Roaming\Typora\typora-user-images\image-20260513191628127.png" alt="image-20260513191628127" style="zoom:50%;" />

看看状态栏是否有对应的标识，最后访问一下google.com是否正常。

![image-20260513191651388](C:\Users\win11_leon\AppData\Roaming\Typora\typora-user-images\image-20260513191651388.png)