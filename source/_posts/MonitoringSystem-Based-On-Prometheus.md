---
title: 搭建一个监控系统，实现可视化数据监控、自动报警(prometheus+alertmanager+grafana)
date: 2024-12-27 15:00:41
categories:
    - 内容分享
tags:
    - 教程
---

# 搭建一个监控系统，实现可视化数据监控、自动报警(prometheus+alertmanager+grafana)
prometheus是一个很强大的数据监控软件，结合alertmanager实现自动报警，然后搭配grafana的仪表盘功能作为监控，可以组成一个简单实用的服务器监控系统，本教程将教会你如何去搭建这样一套系统。

### 准备工作
- docker & docker compose
> 本次搭建的系统除node_exporter以外的服务都将使用docker compose来部署，所以需要自行准备好docker
- 安装包: node_exporter & docker镜像: prom/prometheus prom/alertmanager grafana/grafana
> node_exporter 安装包可去[prometheus官网](https://prometheus.io/download/)下载，docker镜像自行拉取即可

### Step 1 ：为需要监控的主机安装 node_exporter
- 创建程序文件夹，并解压 node_exporter 到创建的文件夹中
```
# 创建一个文件夹来存放 node_exporter 程序文件，并进入文件夹
mkdir -p /data/node_exporter && cd /data/node_exporter

# 解压 node_exporter 到指定文件夹
tar -zxvf node_exporter.tar.gz -C /data/node_exporter
```

- 编写一个service文件来运行和管理 node_exporter 服务

```
# 首先需要创建一个node_exporter的service配置文件
touch /etc/systemd/system/node_exporter.service
# 下面是具体的文件内容
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=nobody
Group=nogroup
ExecStart=/data/node_exporter/node_exporter
Restart=on-failure

[Install]
WantedBy=default.target
```
- 运行并查看这个服务是否正常
```
# 运行 node_exporter
systemctl start node_exporter

# 查看 node_exporter 的运行状态
systemctl status node_exporter

# 设置 node_exporter 开机自启动
systemctl enable node_exporter
```

### Step 2 ：创建 Prometheus、Alertmanager、Grafana的数据文件存放文件夹，及配置文件
- 
```
# prometheus
mkdir -p /data/MonitoringSystem/prometheus/data
mkdir -p /data/MonitoringSystem/prometheus/config/rules
touch /data/MonitoringSystem/prometheus/config/prometheus.yml
touch /data/MonitoringSystem/prometheus/config/rules/lifecycle.rules.yml

# alertmanager
mkdir -p /data/MonitoringSystem/alertmanager/data
mkdir -p /data/MonitoringSystem/alertmanager/config/templates
touch /data/MonitoringSystem/alertmanager/config/alertmanager.yml
touch /data/MonitoringSystem/alertmanager/config/templates/email.tmpl

# grafana
mkdir -p /data/SimpleMonitor/grafana/data

# docker compose
touch /data/SimpleMonitor/compose.yml
touch /data/SimpleMonitor/.env
```

### Step 3 ：编辑Prometheus的配置文件
- prometheus.yml 这是主配置文件
```
global:
  scrape_interval: 15s # 数据采集频率
  evaluation_interval: 15s # 评估规则的频率 [报警]
  scrape_timeout: 15s # 数据采集超时时间
alerting:
  alertmanagers:
  - static_configs: # 报警服务器的地址
    - targets: ["192.80.8.11:9093"]
rule_files: # 导入规则配置
  - "rules/*.rules.yml"
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
  # 这是一个抓取数据的任务，可以自定义名称，下方的targets是一个抓取节点，9100是node_exporter的默认端口
  - job_name: node_self
    static_configs:
      - targets: ["192.168.40.190:9100"]
```
- lifecycle.rules.yml 这是一个规则配置文件，这里只是一个实例掉线检测的示例，详细编写教程可以移步[官方文档](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)
```
groups:
- name: HeartRateMonitoring
  rules:
  - alert: InstanceDown
    expr: up == 0
    for: 15s
    keep_firing_for: 15s
    labels:
      severity: page
    annotations:
      summary: 实例意外关闭 !!
      description: 实例{{$labels.instance}}意外关闭。
```

### Step 4 ：编辑Alertmanager配置文件
- alertmanager.yml 这是主配置文件
```
global:
  resolve_timeout: 5m # 报警超时时间，如果超时未收到后续信息，则判定为已解决
  smtp_from: "123@qq.com" # smtp 邮箱 发件人
  smtp_smarthost: "smtp.qq.com:587" # smtp 服务器，默认smtp是开启了ssl验证的，请使用开启了ssl的校验端口
  smtp_auth_username: "123@qq.com" # smtp 用户名称，一般默认就是邮箱
  smtp_auth_password: "???" # smtp 用户密码，这个需要自己去邮箱网站获取（icloud叫应用ID）
templates:
  - "templates/*.tmpl"
route:
  receiver: "default-receiver"
  group_by: ['alertname']
  group_wait: 5s
  group_interval: 30s
  repeat_interval: 30s
  routes:
    - receiver: "email"
      continue: true
receivers:
  - name: "default-receiver"
    webhook_configs:
      - url: 'http://127.0.0.1:5001/'
  - name: "email"
    email_configs:
      - to: "123@qq.com"
```
- email.tmpl 这是邮件的格式配置文件，默认邮件发送会使用这个配置文件
```
{{ define "email.default.html" }}
{{ range $i, $alert :=.Alerts }}
<p>========监控报警==========</p>
<p>告警状态：{{   .Status }}</p>
<p>告警级别：{{ $alert.Labels.severity }}</p>
<p>告警类型：{{ $alert.Labels.alertname }}</p>
<p>告警应用：{{ $alert.Annotations.summary }}</p>
<p>告警主机：{{ $alert.Labels.instance }}</p>
<p>告警详情：{{ $alert.Annotations.description }}</p>
<p>触发阀值：{{ $alert.Annotations.value }}</p>
<p>告警时间：{{ $alert.StartsAt.Local.Format "2006-01-02 15:04:05" }}</p>

{{ end }}
{{ end }}
```

### Step 5 ：编辑 docker compose 文件和 .env 文件
- compose.yml
```
services:
    prometheus:
        image: prom/prometheus
        container_name: simple_monitor-prometheus
        user: 0:0
        ports:
            - 9090:9090
        volumes:
            - /etc/timezone:/etc/timezone:ro
            - /etc/localtime:/etc/localtime:ro
            - ./prometheus/config:/etc/prometheus
            - ./prometheus/data:/prometheus
        restart: unless-stopped
        networks:
            simple-monitor:
                ipv4_address: ${SUBNET_PREFIX}.10
    alertmanager:
        image: prom/alertmanager
        container_name: simple_monitor-alertmanager
        user: 0:0
        ports:
            - 9093:9093
        volumes:
            - /etc/timezone:/etc/timezone:ro
            - /etc/localtime:/etc/localtime:ro
            - ./alertmanager/config:/etc/alertmanager
            - ./alertmanager/data:/alertmanager
        restart: unless-stopped
        networks:
            simple-monitor:
                ipv4_address: ${SUBNET_PREFIX}.11
    grafana:
        image: grafana/grafana
        container_name: simple_monitor-grafana
        user: 0:0
        ports:
            - 3000:3000
        environment:
            - GF_USERS_DEFAULT_LANGUAGE=zh-Hans
        volumes:
            - /etc/timezone:/etc/timezone:ro
            - /etc/localtime:/etc/localtime:ro
            - ./grafana/data:/var/lib/grafana
        restart: unless-stopped
        networks:
            simple-monitor:
                ipv4_address: ${SUBNET_PREFIX}.12

networks:
    simple-monitor:
        name: simple-monitor
        driver: bridge
        ipam:
            driver: default
            config:
                - subnet: ${SUBNET_PREFIX:?SUBNET_PREFIX required}.0/24
                  gateway: ${SUBNET_PREFIX:?SUBNET_PREFIX required}.1
```
- .env 这个是环境变量配置文件，用于配置IP网段的，如果修改了网段需要去修改prometheus、alertmanager、grafana配置文件或者软件配置中的IP
```
SUBNET_PREFIX=192.168.123
```

### Step 6 ：运行compose 测试Prometheus+Alertmanager的邮件报警是否正常工作
```
# 启动 compose
docker compose up -d

# 去关闭一个node_exporer节点，触发报警规则
systemctl stop node_exporter

# 等待一段时间，查看邮箱是否受到邮件，如果无法正常收到邮件可以通过观察alertmanager的日志排除问题
docker compose logs -f alertmanager
```

### Step 7 ：配置Grafana的Web监控仪表盘
- grafana 的默认账户密码都是 `admin`
- 登录上 grafana 之后找到 数据源 > 添加新数据源 > 选择添加Prometheus > Connection输入http://(docker容器ip):9090 > save&test > 通过测试即可
- 找到仪表盘 > 新建 > 导入 > 输入ID: 1860 （可以[自行选择](https://grafana.com/grafana/dashboards/)合适的面版） > 数据源选择上一步新增的即可

### 至此整个监控系统的搭建就全部完成了
## end