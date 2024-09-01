---
title: "AMD X670E-Plus Wifi 主板启动错误日志处理"
date: 2024-09-01
tags: ["linux", "amd", "gentoo", "kernel"]
---

# AMD X670E-Plus Wifi 主板启动错误日志处理

- 硬件信息

```txt
主板: AMD X670E-Plus Wifi
CPU: AMD Ryzen 9 7950X
GPU: AMD Radeon RX 6950 XT
内存: 64GB
```

- 系统信息

```txt
OS: Gentoo 2.15
Kernel: 6.6.47-gentoo_x86_64
```

## 问题描述

```log
Aug 31 00:32:54 gentoo kernel: hub 12-0:1.0: config failed, hub doesn't have any ports! (err -19)
Aug 31 00:32:54 gentoo (udev-worker)[1324]: event15: Failed to call EVIOCSKEYCODE with scan code 0x7c, and key code 190: Invalid argument
Aug 31 00:32:55 gentoo kernel: ucsi_ccg 4-0008: failed to get FW build information
Aug 31 00:33:00 gentoo kernel: ucsi_ccg 4-0008: failed to reset PPM!
Aug 31 00:33:00 gentoo kernel: ucsi_ccg 4-0008: error -ETIMEDOUT: PPM init failed
```

- 错误1 `hub 12-0:1.0: config failed, hub doesn't have any ports! (err -19)`
  ```txt
  看起来是 USB hub 的配置失败，没有找到解决办法，但是不影响系统正常运行
  TODO: 这个错误的原因还需要进一步的研究
  ```
- 错误2 `Failed to call EVIOCSKEYCODE with scan code 0x7c, and key code 190: Invalid argument`
  ```txt
  这个错误貌似是 ASUS 的主板的问题，主板上有一个按键，似乎是用来重置 BIOS 的
  参考 https://github.com/systemd/systemd/issues/8413
  TODO: 等待后续的解决方案
  ```
- 错误3 `ucsi_ccg 4-0008: failed to get FW build information`
  ```txt
  ucsi_ccg 是 USB Type-C 控制器的驱动，在网上查了一下，貌似是 nvidia 才会使用到
  1. 开始我尝试禁用了 nvidia 的内核选项，但是这个错误还是会出现
  2. 后来禁用了 ucsi_ccg 的内核选项，这个错误就不会出现了
  TODO: 这个后续可以留意一下，看看会不会有什么问题
  ```
