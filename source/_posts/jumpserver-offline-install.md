---
title: JumpServer 离线部署
date: 2024-12-30 16:48:05
categories:
    - 内容分享
tags:
    - 教程
    - 服务部署
---

# JumpServer 离线部署
JumpServer 是一款开源的堡垒机，提供身份验证、授权、审计等功能，用于保护服务器安全。本文将介绍如何离线部署 JumpServer，并配置相关功能。


### 准备工作
- docker 全家桶（docker、docker-compose）
- 镜像文件（JumpServer 离线包，可以自行从[官网](https://community.fit2cloud.com/#/products/jumpserver/downloads)下载）


### Step 1: 上传和解压离线包
- 创建安装目录
```
mkdir -p /data/jumpserver/data
```
- 进入对应目录，下载和解压离线包
[下载地址](https://community.fit2cloud.com/#/products/jumpserver/downloads)
```
cd /data/jumpserver

# 请将下载好的安装包通过sftp或者其它方式上传到当前目录

tar -zxvf jumpserver-offline-installer-xxx.tar.gz

# 将解压出的文件移动到当前目录
mv 解压出来的文件夹名称/* ./
```

### Step 2: 修改配置文件
- 进入安装目录
```
cd /data/jumpserver
```
- 编辑 config-example.txt 文件
```
vim config-example.txt
```
- 修改以下配置项
```
# 修改数据存储位置
VOLUME_DIR=/data/jumpserver/data
```

### Step 3: 安装和启动 JumpServer
- 开始安装JumServer，安装过程中一直选择默认就好
```
./jmsctl.sh install
```
- 启动JumpServer
```
./jmsctl.sh start
```

### Step 4: 访问 JumpServer
- 在浏览器中输入 `http://服务器IP地址:8080` 访问 JumpServer
- 默认用户名是 `admin` 密码是 `ChangeMe` 
- 登录后，可以开始配置和使用 JumpServer 了


### END
一些额外的jumpserver启动项
```
# 停止
./jmsctl.sh down

# 卸载
./jmsctl.sh uninstall

# 帮助
./jmsctl.sh -h
```

根据上述的离线部署 JumpServer 方式，基本没有额外配置，属于傻瓜式安装。

你也可以在安装过程中修改 `config-example.txt` 文件来自定义一些配（数据库、redis、访问端口、docker网络......）。

在安装完成后也可以通过修改 ` /opt/jumpserver/config/config.txt` 文件来修改配置。