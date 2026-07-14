---
name: rich-text-ueditor-flow
description: "处理后台富文本、UEditor、协议内容、上传适配、预览、XSS 白名单、光标跳头和编辑器初始化稳定性。"
---

# 富文本与 UEditor

## 什么时候用

- 富文本改 UEditor。
- 协议、公告、说明、文章等后台富文本维护。
- UEditor config 加载失败。
- 保存富文本报错。
- 输入光标跳头、预览滑不动、左右分栏异常。

## 必查位置

- 前端组件：UEditor 包装、RichTextSplitEditor、上传适配。
- 静态资源：UEditor 文件路径和 public 目录。
- 后端：上传接口、富文本保存接口。
- 安全过滤：XSS 白名单是否误伤后台可信富文本。
- UI：编辑器初始化时容器是否可见，iframe 是否抢事件。

## 常见坑

- 隐藏容器里初始化，工具栏错乱。
- 上传路径被拼成 undefined + url。
- XSS 过滤导致 JSON 或 HTML 保存失败。
- @input 反向 setContent 导致光标跳开头。
- iframe 抢拖拽事件，左右分栏不好用。

## 标准做法

编辑器、上传、预览、XSS、初始化时机要一起处理。不要只换组件名。

后台可信富文本接口需要明确排除不合适的 XSS 清洗，同时保留必要安全边界。

## 验收

打开、输入、上传图片、保存、重新打开回显、预览、小程序展示都正常；拖拽和滚动不被 iframe 影响。