---
title: Teamspeak服务器搭建教程
date: 2024-10-24 11:04:14
categories:
    - 内容分享
tags:
    - 服务部署
    - 教程
---

### 准备工作
- *一台服务器（或一台电脑）*
> 想要拥有一个稳定的teamspeak服务器，你首先得拥有一台服务器主机~
- *Docker*
> 推荐使用Docker来部署teamspeak服务器，这将更加方便快捷~

本教程只针对与TeamSpeak Server的搭建，并不会涵盖其它内容例如：Docker、内网穿透。

***

### Step 1: 拉取镜像
```
docker pull teamspeak:latest
```

### Step 2: 编写 `compose.yaml` 文件
- 创建一个文件夹作为`teamspeak server`的数据文件夹，并在该文件夹中创建一个名为 `compose.yaml` 的文件。
```
mkdir -p /data/teamspeak
cd /data/teamspeak
touch compose.yaml
```
- 在 `compose.yaml` 文件中写入以下内容：
```
services:
  teamspeak:
    container_name: my-teamspeak
    image: teamspeak:latest
    restart: always
    ports:
      - 9987:9987/udp
      - 10011:10011
      - 30033:30033
    volumes:
      - ./data:/var/ts3server/
    environment:
      TS3SERVER_DB_PLUGIN: ts3db_sqlite3
      TS3SERVER_DB_SQLCREATEPATH: create_sqlite
      TS3SERVER_LICENSE: accept
```
- 关于 `compose.yaml` 文件中各字段的解释：
```
TS3SERVER_DB_PLUGIN : 使用那种数据库插件，这里使用sqlite数据库 （还支持: postgresql、mariadb）
TS3SERVER_DB_SQLCREATEPATH : sql文件路径,依据选择的数据库插件来确定，与TS3SERVER_DB_PLUGIN对应
TS3SERVER_LICENSE : 接受TeamSpeak的许可协议

# 如果使用的是 postgresql 或者 mariadb 数据库，请添加以下环境变量
TS3SERVER_DB_HOST : 数据库主机地址
TS3SERVER_DB_USER : 数据库连接账户
TS3SERVER_DB_PASSWORD : 数据库连接密码
TS3SERVER_DB_NAME : 数据库名称 （需要在数据库软件中自行建立）
```

### Step 3: 启动容器，并连接服务器
- 启动容器，并让其在后台运行
```
docker compose up -d
```
- 查看容器日志，获取默认管理员账号密码以及服务器权限密钥
```
docker compose logs
```
- 默认管理员账号密码以及服务器权限密钥在日志中，请及时记录，否则将无法找回！
- 日志中会有两个段落，第一个段落是默认管理员账号密码，第二个段落是服务器权限密钥。
- 切记一定要自行复制保存好数据，下面是一个示例，你看到的日志应该和下面的类似：
```
| ------------------------------------------------------------------
|                       I M P O R T A N T                           
| ------------------------------------------------------------------
|                Server Query Admin Account created                 
|          loginname= "serveradmin", password= "5dLUj9J+"
|          apikey= "BAAPh5f5biOPw5vnFp_u7esZoAJ5KxpvB6Zj0GZ"
| ------------------------------------------------------------------


| ------------------------------------------------------------------
|                       I M P O R T A N T                           
| ------------------------------------------------------------------
|       ServerAdmin privilege key created, please use it to gain 
|       serveradmin rights for your virtualserver. please
|       also check the doc/privilegekey_guide.txt for details.
| 
|        token=hyCWSTHynIh9UhAjXqVl8eYchrChesFZflUd+beR
| ------------------------------------------------------------------
```
- 打开TeamSpeak客户端，输入 `服务器的地址:9987` 来连接。
- 连接服务器之后，右键服务器 -> 使用权限密钥 -> 输入日志中 `token=hyCWSTHynIh9UhAjXqVl8eYchrChesFZflUd+beR` 中的数据，然后点击 `确定`。
- 查看你的头像下方是否有 `红色S` 标志，如果有，说明权限密钥已经成功应用，你现在是服务器管理员，剩下的就可以自行在客户端修改了。

***

### END
- 至此，你已经成功安装并配置了TeamSpeak服务器，并获得了管理员权限。你可以开始使用你的服务器了！