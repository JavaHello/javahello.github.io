---
title: "NVIM java dependency"
series: "neovim"
tags: ["vim"]
---

# NVIM java dependency

在使用`nvim`开发 `java` 时, 我有时需要查看项目依赖和源码目录结构, 我通常使用`mvn dependency:tree`查看依赖, 但是这样不够方便,
为了方便查看, 我写了一个插件`nvim-java-dependency`, 用于查看`java`项目的依赖和源码目录结构.

## 安装

- lazy.nvim

```lua
{
    "JavaHello/java-deps.nvim",
    lazy = true,
    ft = "java",
    dependencies = "mfussenegger/nvim-jdtls",
    config = function()
      require("java-deps").setup({})
    end,
  }

```

- 配置 `jdtls init_options`, 将 `vscode-java-dependency` 的 `jar` 包添加到 jdtls_config["init_options"].bundles 中

```lua
local jdtls_config = {}
local bundles = {}
-- ...
local java_dependency_bundle = vim.split(
  vim.fn.glob(
    "/path?/vscode-java-dependency/jdtls.ext/com.microsoft.jdtls.ext.core/target/com.microsoft.jdtls.ext.core-*.jar"
  ),
  "\n"
)
jdtls_config["init_options"] = {
  bundles = bundles,
}
```

## 使用

```vim
:lua require('java-deps').toggle_outline()
```

## 效果

![Nvim gif](/nvim/nvim-java-deps.gif)
![Nvim mp4](/nvim/nvim-java-deps.mp4)
