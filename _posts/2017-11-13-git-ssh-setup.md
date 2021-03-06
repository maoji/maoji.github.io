---
layout: post
title: git SSH配置
tags:
  - git
---


### 安装
```
sudo apt-get install git
```
### 配置
1. 用户信息
```
git config --global user.name "maoji"
git config --global user.email 2991508929@qq.com
git config --list
```
2. 生成SSH Key，后面会提示输入密码，密码为登陆`github`的密码
```
ssh-keygen -t rsa -b 4096 -C "2991508929@qq.com"
```
如果提示有权限问题，可设置先获取`root`权限，[参考](http://wiki.open.qq.com/wiki/root%E6%9D%83%E9%99%90%E8%AF%B4%E6%98%8E#1._.E8.8E.B7.E5.8F.96root.E6.9D.83.E9.99.90)
```
sudo /bin/su - root
```
3. 添加SSH Key 到 ssh-agent
```
eval $(ssh-agent -s)
```
```
ssh-add ~/.ssh/id_rsa
```
4. 查看SSH key，将其复制后添加到`github`上
```
cat  ~/.ssh/id_rsa.pub
```

### 其他操作
1. 检查是否存在SSH key
```
ls -al ~/.ssh
```
2. 测试SSH key
```
ssh -T git@github.com
```
 如果提示未建立连接，输入`yes`
