---
title: Docker网络模式
aliases: [Docker 网络模式]
type: knowledge
tags: [Docker, 网络]
status: draft
created: 2026-08-08
updated: 2026-08-08
---

# Docker 网络模式

## 常见驱动

- **bridge**：单机容器最常用，用户自定义 bridge 支持容器名 DNS。
- **host**：容器共享宿主网络命名空间，隔离较少，端口发布语义不同。
- **none**：禁用外部网络。
- **overlay**：用于多宿主编排网络。
- **macvlan/ipvlan**：让容器更直接地出现在物理网络中，配置和网络约束更复杂。

用户自定义 bridge 通常优于默认 bridge，因为服务发现和隔离更清晰。

```bash
docker network create app-net
docker run -d --name db --network app-net postgres
docker run -d --name api --network app-net my-api
```

容器间应通过服务名和容器端口通信，不要使用宿主发布端口。容器中的 `localhost` 指向容器自身，并不代表宿主机或另一个容器。

## 排查顺序

1. 确认容器加入了相同网络。
2. 检查服务监听地址和端口。
3. 使用服务名验证 DNS。
4. 检查端口发布、宿主防火墙和代理。
5. 检查应用协议与 TLS 配置。

## 参考资料

- [Docker networking](https://docs.docker.com/engine/network/)
