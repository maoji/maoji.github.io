---
layout: post
title: mongodb安装配置
tags:
  - mongodb
---

## windows安装
### 1. 下载安装包
x32: <https://www.mongodb.org/dl/win32/i386>
x64: <http://dl.mongodb.org/dl/win32/x86_64>
### 2. 安装
如果遇到 `the error code is 2503` 等等错误提示，需要以管理员权限安装，实在不行，可以以管理员权限开一个`cmd`窗口，然后在`cmd`里面运行安装包。
安装完成后，将`C:\Program Files\MongoDB\Server\3.2\bin`路径添加到环境变量`path`里面。
### 3. 配置
#### 3.1 方法1
1. 以管理员权限开一个命令行窗口
2. 创建数据库目录和日志目录
```
mkdir c:\data\db
mkdir c:\data\log
```
3. 在`data`目录下创建`mongod.cfg`文件，指定 `systemLog.path`和`storage.dbPath`
```
systemLog:
    destination: file
    path: c:\data\log\mongod.log
storage:
    dbPath: c:\data\db
```
4. 安装服务，默认的服务名是`MongoDB`。
```
mongod.exe --config "c:\data\mongod.cfg" --install
```
5. 启动服务
```
net start MongoDB
```
6. 停止服务
```
net stop MongoDB
```
7. 删除服务
```
mongod --remove
```

#### 3.2 方法2
1. 按方法1的1、2、3部创建好数据库目录和日志目录，并创建一个新的配置文件。
2. 安装服务
```
sc.exe create MongoDB binPath= "\"C:\Program Files\MongoDB\Server\3.2\bin\mongod.exe\" --service --config=\"C:\data\mongod.cfg\"" DisplayName= "MongoDB" start= "auto"
```
3. 启动和停止服务的方法与方法1相同
4. 删除服务
```
sc.exe delete MongoDB
```