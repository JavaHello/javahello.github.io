---
title: Dubbo matedate 使用记录
date: 2024-07-11
tags: ["dubbo"]
---

## 使用 remote matedate

为什么使用 remote matedate？在服务器之间配置了端口限制，使用原有的自省模式会导致 dubbo 服务相互无法获取元数据，所以使用 remote matedate。

- 自省模式直接使用 dubbo 协议获取元数据

  provider1 限制只允许 consumer1 访问，consumer2 无法获取元数据

```txt
    ┌────────────┐      ┌───────────┐
    │            │◄─────┤ provider1 │◄─┬─┐
    │ zookeeper  │      └───────────┘  │ │
    │            │                     │ │
    │ /services/ │      ┌───────────┐  │ │
    │ /dubbo/    │◄─────┤ consumer1 ├──┘ │ MetadataService
    │  mapping/  │      └───────────┘    │
    │            │                       │
    │            │      ┌───────────┐    │
    │            │◄─────┤ consumer2 ├────┘
    └────────────┘      └───────────┘
```

- remote 模式 dubbo 服务会上报 metadata 数据到 zookeeper, 其他客户端直接在 zookeeper 中获取元数据
  provider1 限制只允许 consumer1 访问，consumer1,2 读取 zookeeper 中的元数据

```txt

    ┌────────────┐      ┌───────────┐
    │            │◄────►│ provider1 │
    │ zookeeper  │      └───────────┘
    │            │
    │ /services/ │      ┌───────────┐
    │ /dubbo/    │◄────►│ consumer1 │
    │  metadata/ │      └───────────┘
    │   provider1│
    │            │      ┌───────────┐
    │            │◄────►│ consumer2 │
    └────────────┘      └───────────┘

```

![Calls](/java/dubbo/Calls-getRemoteMetadata.svg)
![ControlFlow](/java/dubbo/ControlFlow-getRemoteMetadata.svg)

- 应用配置

```yml
dubbo:
  application:
    metadata-type: remote # 上报元数据到远程，消费端去远程获取元数据
  metadata-report:
    address: zookeeper://zookeeper01:2181 # 使用 zookeeper 作为元数据中心
```
