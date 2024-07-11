---
title: Dubbo 使用记录
tags: ["dubbo"]
date: 2024-07-11
---

## 使用 remote mate

```yml
dubbo:
  application:
    metadata-type: remote # 上报元数据到远程，消费端去远程获取元数据
  metadata-report:
    address: zookeeper://zookeeper01:2181 # 使用 zookeeper 作为元数据中心
```

为什么使用 remote mate？在服务器中配置了端口限制，使用原有的自省模式会导致无法获取元数据，所以使用 remote mate。

- 自省模式直接使用 dubbo 协议获取元数据

```txt

    ┌────────────┐      ┌──────────┐
    │            │◄─────┤ provider │◄─┬─┐
    │ zookeeper  │      └──────────┘  │ │
    │            │                    │ │
    │ /services/ │      ┌──────────┐  │ │ MetadataService
    │ /dubbo/    │◄─────┤ consumer ├──┘ │
    │  mapping/  │      └──────────┘    │
    │            │                      │
    │            │      ┌──────────┐    │
    │            │◄─────┤ consumer ├────┘
    └────────────┘      └──────────┘

```

- remote 模式 dubbo 服务会上报 metadata 数据到 zookeeper, 其他客户端直接在 zookeeper 中获取元数据

```txt

    ┌────────────┐      ┌──────────┐
    │            │◄────►│ provider │
    │ zookeeper  │      └──────────┘
    │            │
    │ /services/ │      ┌──────────┐
    │ /dubbo/    │◄────►│ consumer │
    │  metadata/ │      └──────────┘
    │   server01 │
    │            │      ┌──────────┐
    │            │◄────►│ consumer │
    └────────────┘      └──────────┘

```
