---
title: "NVIM 打造 Java IDE"
series: "学习使用 neovim"
tags: ["vim"]
---

# NVIM 打造 Java IDE

[JavaHello/nvim](https://github.com/JavaHello/nvim/tree/nvim-lsp)

## `Java`开发环境配置

- 安装 `jdk11 +`配置好环境变量
  ```sh
  # Java runtimes
  export JAVA_8_HOME=/path/jdk8 # default
  export JAVA_11_HOME=/path/jdk11
  export JAVA_17_HOME=/path/jdk17
  export JAVA_HOME=$JAVA_17_HOME # jdt.ls 运行依赖
  ```
- 安装 `maven` 配置好环境变量
  ```sh
  # Maven
  export MAVEN_HOME=/path/maven
  ```
- 安装 `gradle` 可选
- 安装 `vscode` 相关 `Java` 扩展包 (vscode/mason.nvim 任意选择一种方式安装)
  - [vscode-java-pack](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack)
    `jdt.ls`,`junit test`, `debug` 依赖
  - [java-decompiler](https://marketplace.visualstudio.com/items?itemName=dgileadi.java-decompiler)
    反编译依赖
- 如果你不是使用 vscode, 可使用 `mason.nvim` 安装 java 插件 `:MasonInstall jdtls java-debug-adapter java-test`
- 配置 [lombok](https://projectlombok.org/download) 无需手动下载, `vscode-java`, `mason.nvim jdtls` 都包含了

  ```sh
  # 默认为Y启用
  # 优先读取 vscode-java 扩展的 lombok-x.x.x.jar
  # 未找到读取 jdtls 扩展 lombok.jar
  # 最后没有读取到不会启用
  export LOMBOK_ENABLE=N
  ```

- 配置 `jdt.ls` server (可选)
  ```sh
  # jdt.ls 的 路径，默认使用 vscode-java 扩展 或者 mason.nvim 安装的 jdtls
  # 可以自己 clone 代码编译
  export JDTLS_HOME=/path/jdt.ls
  ```
- 打开 `Java` 工程测试
