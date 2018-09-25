---
title: Ubuntu 安装配置 Shadowsocks
date: 2018-09-25 23:51:53
category: Softwares
---

# 安装

使用命令:

```
sudo apt-get install python-pip && export LC_ALL=C && pip install setuptools && pip install shadowsocks
```

# 配置

创建配置文件`/etc/shadowsocks.json`，内容如下:

```
{
    "server": "0.0.0.0",
    "server_port": 8388,
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "password": "zhangwen123",
    "timeout": 300,
    "method": "aes-256-cfb",
    "fast_open": false
}
```

修改服务器地址`server`。

# 启动

执行命令:

```bash
/usr/local/bin/ssserver -c /etc/shadowsocks.json
```

开机自启动：把上面的命令加在`/etc/rc.local`的末尾。
