---
layout: post
title: pycrypto安装
tags:
  - pycrypto
---

### python 2.7
1. windows版本
直接下载编译好的安装包安装即可<http://www.voidspace.org.uk/python/modules.shtml#pycrypto>
>

### python 3.6
1. 下载一份`pycrypto`源码<https://github.com/dlitz/pycrypto/releases/tag/v2.6.1>
2. 安装`visual-cpp-build-tools`<http://landinghub.visualstudio.com/visual-cpp-build-tools>
3. 添加环境变量`VCINSTALLDIR`
```
C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC
```
4. 在`pycrypto`目录下开一个命令行窗口，执行：
```
set CL=/FI"%VCINSTALLDIR%\\INCLUDE\\stdint.h" %CL%
```
5. 安装`pycrypto`
```
python setup.py install
```
