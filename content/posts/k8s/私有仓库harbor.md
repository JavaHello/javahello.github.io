---
title: "Harbor 安装使用"
---

# Harbor 安装使用

安装参考 [install-config](https://goharbor.io/docs/2.9.0/install-config/)

## 注意事项

- 如果使用 IP 访问, 生成证书需要添加 IP
  ```txt
  openssl req -sha512 -new \
    -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=luokai.com" \
    -key luokai.com.key \
    -out luokai.com.csr \
    -addext "subjectAltName=IP:192.168.50.165,IP:127.0.0.1" # IP 添加在这里
  ```
- 如果使用 IP 访问, harbor.yml 配置
  ```txt
  # harbor.yml
  # hostname: 192.168.50.165
  hostname: 0.0.0.0
  ```
- 如果使用 IP 访问或 HTTP, docker 配置
  ```json
  # /etc/docker/daemon.json
  {
    "insecure-registries": ["192.168.50.165"]
  }
  ```
