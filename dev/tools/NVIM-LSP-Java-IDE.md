# NVIM 打造 Java IDE

当你习惯了 `Vim` 文本编辑器，你就习惯了 `Vim` 文本编辑器(🐶)。

## 配置

[配置地址](https://github.com/JavaHello/nvim/tree/nvim-lsp) 将配置拷贝到本机的 `~/.config/` 文件中。 配置都写了中文描述，使用前需自己看下。

### 关键配置和插件

- `<Leader>` 键修改为了`;`.
- 使用 `packer.nvim` 管理插件。
- `nvim-lsp` 实现常见编程语言代码提示等功能。
- `vim-fugitive` 最好用的 `Git` 插件, 可查看修改，提交等。
- `LeaderF` 文件缓冲区等搜索插件, 配合 `rg` 使用全局搜索起飞。
- `asynctasks.vim` 异步执行任务插件，可执行终端命令，我用于编译打包构建发布。
- `vim-floaterm` 浮动终端，我用于执行一下命令，配合 `asynctasks.vim` 使用。

### `Java IDE` 实现功能描述

- 代码提示，使用 `nvim-cmp` `cmp-nvim-lsp` `nvim-jdtls` 插件, `eclipse.jdt.ls` 后端支持。支持单文件工程，`Maven`, `Gradle` 工程
- 跳转到定义 `gd`, 跳转到实现 `gi`, 查看引用 `gr`, 查看文档 `K`。
- 命令模式下 `:OR` 自动导包, `Format` 格式化代码
- 大纲快捷键`<space>o`，`vista.vim` 插件实现
- `get`,`set`, 构造方法, `impl` 等操作，快捷键 `<Leader> ca`
- `debug` 使用 `nvim-dap` 插件, 依赖 `java-debug` 和 `vscode-java-test`。

### `Java` 开发配置描述

#### 依赖

- `nvim 0.6+`
- `jdk11 +`
- `python3.9`(3.x 其它版本应该也可以)

#### 安装 nvim

参考官方文档[Installing-Neovim](https://github.com/neovim/neovim/wiki/Installing-Neovim)

- Mac 下使用 homebrew 安装
  ```sh
  brew install neovim
  ```

#### 安装 jdk

参考[Mac M1 Java 开发环境配置](/笔记/开发工具/Mac-M1-Java-开发环境配置.md)

#### 主要配置

##### `nvim-jdtls`

> 注意标识 🍓 的需要修改

官方文档[nvim-jdtls](https://github.com/mfussenegger/nvim-jdtls), 我的完整配置[ftplugin/java.lua](https://github.com/JavaHello/nvim/blob/nvim-lsp/ftplugin/java.lua)

- 配置文件在 `ftplugin/java.lua` 下, 目的是当打开 `java` 文件自动加载配置
- 下载[eclipse.jdt.ls](https://download.eclipse.org/jdtls/snapshots/?d)(建议最新版本)到 `/opt/software/jdtls/` 目录下, 目录可自定义, 用于配置 `config.cmd` 下的启动 `jar` 包路径。
- `eclipse.jdt.ls` 启动参数配置

  ```lua
  local config = {
    -- The command that starts the language server
    -- See: https://github.com/eclipse/eclipse.jdt.ls#running-from-the-command-line
    cmd = {
      -- 🍓 需要 jdk11 或以上 
      '/Library/Java/JavaVirtualMachines/zulu-11.jdk/Contents/Home/bin/java', -- or '/path/to/java11_or_newer/bin/java'
      '-Declipse.application=org.eclipse.jdt.ls.core.id1',
      '-Dosgi.bundles.defaultStartLevel=4',
      '-Declipse.product=org.eclipse.jdt.ls.core.product',
      '-Dlog.protocol=true',
      '-Dlog.level=ALL',
      '-XX:+UseParallelGC',
      '-XX:GCTimeRatio=4',
      '-XX:AdaptiveSizePolicyWeight=90',
      '-Dsun.zip.disableMemoryMapping=true',
      '-Xms100m',
      '-Xmx2g',
      --  🍓lombok 支持（需下载lombok.jar）
      '-javaagent:/opt/software/lsp/lombok.jar',
      '--add-modules=ALL-SYSTEM',
      '--add-opens', 'java.base/java.util=ALL-UNNAMED',
      '--add-opens', 'java.base/java.lang=ALL-UNNAMED',
      -- 🍓启动jar路径
      '-jar', '/opt/software/jdtls/plugins/org.eclipse.equinox.launcher_1.6.400.v20210924-0641.jar',
      '-configuration', '/opt/software/jdtls/config_mac',
      -- 🍓data 工作空间目录，每个项目建议唯一
      '-data', workspace_dir,
    },
    ...
  }


  local jdtls = require('jdtls')
  local capabilities = require('cmp_nvim_lsp').update_capabilities(vim.lsp.protocol.make_client_capabilities())
  config.capabilities = capabilities;
  -- 启动
  jdtls.start_or_attach(config)
  ```

- 配置环境变量`JAVA_HOME`
- 配置`Maven`到`PATH`中(打开 Maven 工程需要)
- 自定义`Maven settings.xml` 文件, 配置 `MAVEN_SETTINGS` 变量（可选）

##### 配置快捷键
```lua
local map = vim.api.nvim_set_keymap
-- 快捷键统一配置在了 keybindings.lua 文件中, 这里加载一下就可以了
require('keybindings').maplsp(map)
```
到这一步就可以愉快到使用了

##### debug 和 junit test 配置

- 下载安装 [java-debug](https://github.com/microsoft/java-debug) 和 [vscode-java-test](https://github.com/microsoft/vscode-java-test)插件(微软为 vscode 开发的)
    ```sh
    # java-debug
    cd /opt/software/lsp/java
    git clone git@github.com:microsoft/java-debug.git
    cd java-debug
    ./mvnw clean install

    # vscode-java-test
    git clone git@github.com:microsoft/vscode-java-test.git
    cd vscode-java-test
    npm install
    npm run build-plugin
    ```
- 添加配置到`ftplugin/java.lua`
  ```lua
  -- This bundles definition is the same as in the previous section (java-debug installation)
    local bundles = {
      -- 🍓注意路径
      vim.fn.glob("/opt/software/lsp/java/java-debug/com.microsoft.java.debug.plugin/target/com.microsoft.java.debug.plugin-*.jar")
    };

    -- 🍓注意路径
    -- /opt/software/lsp/java/vscode-java-test/server
    vim.list_extend(bundles, vim.split(vim.fn.glob("/opt/software/lsp/java/vscode-java-test/server/*.jar"), "\n"))
    config['init_options'] = {
      bundles = bundles;
    }

    config['on_attach'] = function(client, bufnr)
      -- With `hotcodereplace = 'auto' the debug adapter will try to apply code changes
      -- you make during a debug session immediately.
      -- Remove the option if you do not want that.
      require('jdtls').setup_dap({ hotcodereplace = 'auto' })
    end

  ```
- 快捷键
    ```txt
    <Leader>b 添加取消断点
    命令模式下
    :TestClass
    :TestMethod
    执行当前class的 junit 测试方法
    :TestClassUI
    :TestMethodUI
    会打开调试页面(依赖 nvim-dap-ui 插件)

    F5 启动 main 方法, 如无法找到配置,需要先执行 :JdtRefreshDebugConfigs  
    F10 单步
    其它功能可自己探索
    ```

### 使用视频

<iframe src="//player.bilibili.com/player.html?aid=681042787&bvid=BV19S4y1L7xK&cid=494660450&page=1&high_quality=1&danmaku=1" width="100%" height="500" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" sandbox="allow-top-navigation allow-same-origin allow-forms allow-scripts"> </iframe>
