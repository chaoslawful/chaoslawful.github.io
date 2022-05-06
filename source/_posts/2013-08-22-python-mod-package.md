---
title: 如何将 Python 模块打成 DEB/RPM 包
tags:
    - Python
    - deb
    - rpm
date: 2013-08-22 12:34:00
---

Python 模块的安装可以用 easy_install 或 pip 方便地完成，但此类工具难以应用在生产系统的部署中。使用 DEB/RPM 包的好处是大规模部署简单、容易回滚且能以一致的方式管理依赖，所以需要将 Python 模块打成此类原生包。
<!-- more -->

打 RPM 包比较简单，现在的模块一般都是 setuptools 规范的，只要简单地进入模块源码目录，运行 `python setup.py bdist_rpm` 即可打出 RPM 包。

打 DEB 包需要 [stdeb](https://pypi.python.org/pypi/stdeb) 工具的支持，安装该工具后可以用如下命令打出 DEB 包：
- 下载要打的模块（假设为 uncertainties 模块）源码 tarball，可用命令 `pip install -d . uncertainties`
- 假设源码包为 `uncertainties-2.4.1.tar.gz` ，执行 `py2dsc uncertainties-2.4.1.tar.gz` 将其转换为 Debian 源码包
- 进入 `py2dsc` 生成的打包目录打包：
  - `cd deb_dist/uncertainties-2.4.1/`
  - `dpkg-buildpackage -rfakeroot -uc -us`
- 生成的 DEB 包就在上级目录里

如果只是想直接安装转换后的 DEB 包，则可以简单地用命令 `sudo pypi-install uncertainties` 完成，无需使用上面较麻烦的步骤。

参考：
- https://pypi.python.org/pypi/stdeb
- http://docs.python.org/2.0/dist/creating-rpms.html
