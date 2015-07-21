## 背景

在很多场景下（手头没有 U 盘、某台电脑 USB 口不可用或没有足够权限安装存储设备驱动等等），我们都需要通过网线直连的方式在两台电脑之间传输数据，这样就要求正确地将两台电脑设置为同一子网下的不同静态 IP。对于电脑知识较为丰富的用户来说此设置过程非常简单，但对于一般用户来说就比较困难了，此时就希望通过程序自动配置好这种网络环境。

## 分析

这里主要针对 Windows XP 系统自动配置进行分析，因 Vista 以上版本系统已经有了 `zeroconf` 的实现，有现成的解决方案可用。

网卡地址要么是静态设置，要么是动态 DHCP 分配的。对于静态设置 IP 的情况，拔掉再插上网线后 Windows 系统会监测到媒介状态变化，并自动发送若干源 MAC 为对应网卡 MAC，源/目标 IP 均为当前网卡静态设置 IP 的 ARP 包（称为 Gratuitous ARP 包，用来向子网内其他机器或网关主张自己的静态 IP 所有权），若将网线另一侧机器的 IP 设置为同一子网内的地址，两台机器就能正常通讯了。

![Static IP Capture](/uploads/2015-07-21/static-ip-pcap.png "静态 IP 抓包结果")

对于动态 DHCP 分配的情况，拔掉再插上网线后 Windows 系统会自动发送 DHCP 相关请求，要求可能存在的 DHCP 服务器分配 IP，若在网线另一侧机器上运行 DHCP 服务，即可自动设置双机 IP 在同一子网中，也能正常通讯。

![DHCP IP Capture](/uploads/2015-07-21/dhcp-ip-pcap.png "动态 DHCP 抓包结果")

## 实现

参考实现程序[在此](https://github.com/chaoslawful/nullconf)。该程序使用 `pcapy` 模块操作 `WinPCAP` 在指定网卡（Windows 环境下通过 `wmi` 模块自动获得首个可用的有线网卡）上监听数据包，通过 `impacket` 模块解析后根据数据包特征决定使用静态还是动态配置逻辑，最后用 OS 相关方法设置本地监听的有线网卡 IP 后（Windows 环境下通过 `wmi` 模块实现）完成双机 IP 配置。

## 参考

- http://timgolden.me.uk/python/wmi/tutorial.html
- https://msdn.microsoft.com/en-us/library/bg126473(v=vs.85).aspx
- https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol

