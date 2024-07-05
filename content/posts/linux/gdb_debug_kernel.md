---
title: "使用 GDB + Qemu 调试 Linux 内核"
tags: ["Linux", "GDB", "Qemu"]
---

# 使用 GDB + Qemu 调试 Linux 内核

## Linux 内核编译

使用 `master` 分支

```sh
$ git clone https://github.com/torvalds/linux.git
$ cd linux
$ make mrproper # 清理旧配置和编译生成的文件
$ make x86_64_defconfig # 生成默认配置
$ make nconfig # 开启 debug 选项
# Kernel hacking --> Compile-time checks and compile options --> Debug information --> Generate DWARF Version 5 Debuginfo
$ make -j$(nproc) # 编译

# 内核文件
$ ls -hl vmlinxu
$ ls -hl ./arch/x86_64/boot/bzImage # 压缩后的内核文件
```

## 文件系统制作

使用 `busybox`

```sh
$ git clone git://busybox.net/busybox.git
# or
# git clone https://github.com/mirror/busybox.git # github 镜像

$ cd busybox
$ make menuconfig # 选择静态编译
# Settings # 按Enter 进入
# --> Build Options
#     [*] Build static binary (no shared libs) # 按 Y 选择
$ make -j$(nproc)
$ make install

# 创建系统目录
$ cd _install
$ mkdir proc
$ mkdir sys
$ touch init
$ vim init # 输入初始化程序
$ chmod +x init # 添加可执行权限

$ find . | cpio -o --format=newc > ./rootfs.img # 生成 rootfs 文件
```

`init` 程序

```sh
#!/bin/sh
echo "{==DBG==} INIT SCRIPT"
mkdir /tmp
mount -t proc none /proc
mount -t sysfs none /sys
mount -t debugfs none /sys/kernel/debug
mount -t tmpfs none /tmp

mdev -s
echo -e "{==DBG==} Boot took $(cut -d' ' -f1 /proc/uptime) seconds"

# normal user
setsid /bin/cttyhack setuidgid 1000 /bin/sh
```

## Qemu 启动内核

```sh
$ qemu-system-x86_64 -kernel ./bzImage -initrd  ./rootfs.img -append "nokaslr console=ttyS0" -s -S -nographic
```

- `-kernel ./bzImage`： 指定启用的内核镜像；
- `-initrd ./rootfs.img`：指定启动的内存文件系统；
- `-append "nokaslr console=ttyS0"` ： 附加参数，其中 nokaslr 参数必须添加进来，防止内核起始地址随机化，这样会导致 gdb 断点不能命中；参数说明可以参见这里。
- `-s` ：监听在 gdb 1234 端口；
- `-S` ：表示启动后就挂起，等待 gdb 连接；
- `-nographic`：不启动图形界面，调试信息输出到终端与参数 console=ttyS0 组合使用；

## GDB 调试

```sh
$ gdb
(gdb) file vmlinux
(gdb) target remote :1234
(gdb) break start_kernel
(gdb) c                        # 启动时会停止在 start_kernel 函数上
(gdb) add-auto-load-safe-path /opt/os/linux/
(gdb) source vmlinux-gdb.py                        # 使用 vmlinux-gdb 提供的命令
```

## 参考

- [Debugging kernel and modules via gdb](https://www.kernel.org/doc/html/latest/dev-tools/gdb-kernel-debugging.html)
- [使用 GDB + Qemu 调试 Linux 内核](https://www.ebpf.top/post/qemu_gdb_busybox_debug_kernel/)
