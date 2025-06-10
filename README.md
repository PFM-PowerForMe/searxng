# SearxNG

[![PFM-Upstream-Sync](https://github.com/PFM-PowerForMe/searxng/actions/workflows/fork-sync.yml/badge.svg)](https://github.com/PFM-PowerForMe/searxng/actions/workflows/fork-sync.yml)

## 简介
SearXNG 是一款免费的互联网元搜索引擎，它聚合了来自各种搜索服务和数据库的结果。用户不会被追踪或分析。

## 如何部署?

1. 前置条件:
```shell
mkdir -p /etc/containers/systemd
podman network create deploy
```

2. 部署 Redis [Valkey](https://github.com/valkey-io/valkey)

```shell
nvim /etc/containers/systemd/valkey.container
```

```
# /etc/containers/systemd/valkey.container

[Unit]
Description=The valkey container
Wants=network-online.target
After=network-online.target

[Container]
AutoUpdate=registry
ContainerName=valkey
Timezone=local
Network=deploy
Volume=valkey-data:/data
Exec=/usr/local/bin/valkey-server --save 30 1 --loglevel warning
Image=ghcr.io/valkey-io/valkey:latest

[Service]
Restart=on-failure
RestartSec=30s
StartLimitInterval=30
TimeoutStartSec=900
TimeoutStopSec=70

[Install]
WantedBy=multi-user.target default.target
```

3. 部署 SearxNG:
```shell
mkdir -p /opt/podman-data/env
nvim /opt/podman-data/env/searxng.env
```

```
SEARXNG_BASE_URL=https://my-site
INSTANCE_NAME=my-name
UWSGI_WORKERS=2
UWSGI_THREADS=2
```
---

```shell
nvim /etc/containers/systemd/searxng.container
```

```
# /etc/containers/systemd/searxng.container

[Unit]
Description=The searxng container
Wants=valkey.service
After=valkey.service

[Container]
AutoUpdate=registry
ContainerName=searxng
Timezone=local
Network=deploy
EnvironmentFile=/opt/podman-data/env/searxng.env
PublishPort=127.0.0.1:6004:8080
Image=ghcr.io/pfm-powerforme/searxng:latest

[Service]
Restart=on-failure
RestartSec=30s
StartLimitInterval=30
TimeoutStartSec=900
TimeoutStopSec=70

[Install]
WantedBy=multi-user.target default.target
```