---
title: 雷池Waf离线部署
date: 2024-12-31 10:10:27
categories:
    - 内容分享
tags:
    - 教程
    - 服务部署
---

# 雷池WAF 离线部署

### 准备工作
- docker 全家桶 (docker、docker-compose)
- 雷池waf [离线镜像文件](https://demo.waf-ce.chaitin.cn/image.tar.gz)
- 雷池waf [compose.yaml 文件](https://waf-ce.chaitin.cn/release/latest/compose.yaml)

### Step 1: 创建安装目录，导入docker 镜像
- 创建安装目录 （可根据需要自定义安装目录）
```
mkdir -p /data/safelinewaf
cd /data/safelinewaf
```

- 导入docker 镜像
```
# 将下载好的雷池waf离线镜像上传到当前目录

cat image.tar.gz | gzip -d | docker load
```

### Step 2: 配置环境变量
- 创建 `.env` 环境变量文件
```
touch .env
```

- 编辑 `.env` 文件，添加以下内容，根据需要修改
```
SAFELINE_DIR=/data/safelinewaf
IMAGE_TAG=latest
MGT_PORT=9443
POSTGRES_PASSWORD=yourpassword
SUBNET_PREFIX=172.22.222
IMAGE_PREFIX=swr.cn-east-3.myhuaweicloud.com/chaitin-safeline
ARCH_SUFFIX=
RELEASE=
REGION=
```
- `.env` 文件中各参数说明
```
> SAFELINE_DIR: 雷池安装目录，如 /data/safeline
> IMAGE_TAG: 要安装的雷池版本，保持默认的 latest 即可
> MGT_PORT: 雷池控制台的端口，保持默认的 9443 即可
> POSTGRES_PASSWORD: 雷池所需数据库的初始化密码，请随机生成一个
> SUBNET_PREFIX: 雷池内部网络的网段，保持默认的 172.22.222 即可
> IMAGE_PREFIX: 雷池镜像源的前缀，建议根据服务器地理位置选择合适的源
> ARCH_SUFFIX: 雷池架构的后缀，ARM 服务器需要配置为 -arm
> RELEASE: 更新通道，LTS 版本需要配置为 -lts
```

### Step 3: 上传 `compose.yml` 并启动雷池
- 将下载好的 `compose.yaml` 文件上传到当前目录
- 启动雷池
```
docker compose -f compose.yaml up -d
```
- 控制台URL: https://本机IP:雷池控制台端口/
- 默认用户名为 `admin`，默认密码为 `.env` 配置的密码，如果忘记密码可以通过下面的指令来重置 `admin` 的密码
```
docker exec safeline-mgt resetadmin
```
### END