---
title: AI 去除背景
date: 2024-07-22
tags: ["AI", "RMBG"]
---

# AI 去除背景

- 使用 [RMBG](https://crates.io/crates/rmbg) 库，基于 ONNX 模型，实现 AI 去除背景。

- 模型下载地址 [model.onnx](https://huggingface.co/briaai/RMBG-1.4/blob/main/onnx/model.onnx)

```rust
use rmbg::Rmbg;

fn main() -> anyhow::Result<()> {
    // 加载模型
    let rmbg = Rmbg::new("./model.onnx")?;

    // 读取原始图像
    let original_img = image::open("./image.png")?;

    // 移除背景
    let img_without_bg = rmbg.remove_background(&original_img)?;

    // 保存结果
    img_without_bg.save("./out.png")?;
    Ok(())
}
```

## 去除前后对比

- 原始图像
  ![rmbg](/rmbg/992857.jpeg)

- 去除背景后
  ![rmbg](/rmbg/out/out3.png)
