
- [前言](#前言)
- [USD](#usd)
  - [什么是USD](#什么是usd)
  - [几种格式](#几种格式)
- [研究点](#研究点)
  - [python解析 usd-core](#python解析-usd-core)
  - [借助工具查看 usd 文件结构](#借助工具查看-usd-文件结构)
    - [UsdView](#usdview)
  - [web展示USD文件](#web展示usd文件)
    - [文章记录](#文章记录)
  - [ply to usd](#ply-to-usd)
    - [借助工具 3dgrut](#借助工具-3dgrut)


# 前言
最近研究关于usd的相关内容，发现其与tree的机制类似，都是将usd文件解析成tree的过程。



# USD
openusd 文档：https://openusd.org/release/index.html

## 什么是USD
皮克斯的开源通用场景描述（Universal Scene Description，简称USD）不仅为他们工作流程的核心，还为其他VFX和动画工作室的流程提供了一种允许3D数据在不同数字内容创建应用之间互换通用的方式。

## 几种格式
- USD：通用场景描述（Universal Scene Description）的基本文件格式 ，它可以是 ASCII 编码的文本格式，也可以是二进制编码格式。作为一种 3D 文件格式，它能描述场景图，包含模型、材质、灯光、相机等各种 3D 图形元素，还支持复杂场景的灵活扩展、实时创作和多人协作。
- USDZ：是 USD 的零压缩格式，本质是将 USD 文件（可以是 USDA、USDC 等）和纹理图片（如 png、jpeg ）等资源，采用 64 字节对齐方式，打包压缩到一个单一文件中，类似 zip 存档文件，便于在网络上传输和分享，适合在 iOS、iPadOS、macOS 、tvOS 等系统上进行 AR（增强现实）渲染展示。
- USDA：以 ASCII 编码的纯文本格式的 USD 文件，方便人类阅读和编辑，开发者可以直接通过文本编辑器查看和检查文件内容，理解 3D 文件的内部结构，比如模型的网格数据、网格关系、材质处理方式等。



# 研究点
- [openusd](https://github.com/PixarAnimationStudios/OpenUSD/tree/dev?tab=readme-ov-file)



## python解析 usd-core
> usd-core 文档：https://openusd.org/release/tut_simple_shading.html

## 借助工具查看 usd 文件结构

### UsdView
> usdview是由皮克斯动画工作室开发的一个轻量级应用程序，用于查看，导航和反省美元阶段。Usdview的自省功能帮助用户理解prim组成和属性值分辨率。它是调试USD阶段不可缺少的工具。
> 
> https://docs.omniverse.nvidia.com/usd/latest/usdview/index.html

1. 下载UsdView [url](https://developer.nvidia.com/usd?sortBy=developer_learning_library%2Fsort%2Ffeatured_in.usd_resources%3Adesc%2Ctitle%3Aasc&hitsPerPage=6#section-getting-started)
2. 启动服务
![alt text](assets/usd-research/image.png)


## web展示USD文件

### 文章记录
1. https://extra-ordinary.tv/2023/07/16/displaying-usd-3d-models-in-a-web-browser/
2. https://github.com/PixarAnimationStudios/OpenUSD/tree/dev?tab=readme-ov-file
3. https://github.com/needle-tools/usd-viewer
4. https://github.com/autodesk-forks/USD/blob/adsk/feature/archive-webgpu/pxr/usdImaging/bin/usdviewweb/README.md

## ply to usd
> 关于高斯模型在issac中显示，看到有说法是，目前isaac-sim5.0以下不支持高斯渲染，所以尝试将ply模型转换成usd模型，在5.0以上版本中可以正常显示。

### 借助工具 3dgrut
> github: https://github.com/nv-tlabs/3dgrut