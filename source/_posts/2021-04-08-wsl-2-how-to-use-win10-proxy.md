---
title: 在 WSL2 环境中使用 Win10 宿主系统中代理的方法
date: 2021-04-08 20:28:51
tags:
    - WSL
    - Win10
    - Networking
---

## 背景

Win10 中 WSL2 环境是一个 HyperV 管理的独立容器，其网络相对宿主系统独立，不像 WSL1 环境那样是和宿主系统网卡直接桥接起来的。WSL2 这样的架构虽然运行效率提高不少，但却无法像 WSL1 那样能直接使用宿主系统提供的代理服务，造成了不少麻烦。
<!-- more -->

## 解决方案

解决办法很简单，只要允许代理服务为非 `localhost` 来源地址服务，然后直接使用宿主系统的 IP 地址就能访问到代理了。以 Ubuntu WSL2 环境访问 Shadowsocks 代理为例，步骤如下：

1. 允许 Shadowsocks 服务非本地来源地址：
   ![](/images/wsl2-use-win10-proxy/ss-config.png)
1. 在 `.bashrc` 中加入如下的函数方便打开/关闭代理（假设宿主系统 IP 地址为 `192.168.175.173`，Shadowsocks SOCKS5 代理在 `1080` 端口）：
```bash
proxy () {
    # 设置全局代理服务器环境变量
    export ALL_PROXY="socks5://192.168.175.173:1080"
    export all_proxy="socks5://192.168.175.173:1080"
    # 给 apt-get 增加代理配置
    echo -e "Acquire::http::Proxy \"http://192.168.175.173:1080\";" | sudo tee -a /etc/apt/apt.conf > /dev/null
    echo -e "Acquire::https::Proxy \"http://192.168.175.173:1080\";" | sudo tee -a /etc/apt/apt.conf > /dev/null
    # 提示用户当前出口地址
    curl myip.ipip.net
}

noproxy () {
    # 清除全局代理服务器环境变量
    unset ALL_PROXY
    unset all_proxy
    # 清除 apt-get 代理设置
    sudo sed -i -e '/Acquire::http::Proxy/d' /etc/apt/apt.conf
    sudo sed -i -e '/Acquire::https::Proxy/d' /etc/apt/apt.conf
    # 提示用户当前出口地址
    curl myip.ipip.net
}
```
这样在终端上就可直接用 `proxy` 开启代理，`noproxy` 关闭代理。
1. 对于那些不使用全局代理服务器环境变量的应用，可以安装 `proxychains` 来实现拦截其网络通信强制走代理的目的：
```bash
$ sudo apt install proxychains tor
```
安装完毕后，向 `/etc/proxychains.conf` 增加如下配置：
```ini
[ProxyList]
socks5 192.168.175.173 1080
```
然后就能用 `proxychains <app_path> <cmdline>` 的形式强制应用使用代理了。

## 参考资料

* https://zhuanlan.zhihu.com/p/144647249

