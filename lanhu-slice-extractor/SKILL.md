---
name: "lanhu-slice-extractor"
description: "从蓝湖页面中定位并导出切图资源。当用户要求从蓝湖设计稿、画板或页面内获取封面图、图层切图、导出资源时调用。"
---

# Lanhu Slice Extractor

本技能用于在蓝湖页面中稳定提取图层级切图资源，适用于“不要裁整张图，要拿蓝湖原生切图”的场景。

## 适用场景

- 用户给出蓝湖项目链接，要求拿某个页面里的封面图、图标、按钮图、文章配图等切图资源。
- 用户明确表示要“点到页面内部”“点击图片下载对应切图”“不要本地裁图”。
- 蓝湖详情页、标注页或右侧切图面板在自动化环境下不稳定，需要改走蓝湖内部数据和导出资源地址。

## 核心原则

- 优先复用蓝湖自身导出资源，不做本地裁图。
- 优先走蓝湖项目画布页 `stage`，不要把 `detailDetach` 空白误判成“没有数据”。
- 当右侧切图面板在自动化环境下不稳定时，优先读取蓝湖“图片信息”中的 JSON，再从导出资源中定位目标切图。

## 标准流程

### 1. 先进入项目并确认登录态

- 打开蓝湖链接后，若跳转登录页，先完成登录。
- 登录成功后，优先落到：

```text
#/item/project/stage?pid=...&image_id=...&tid=...
```

- 如果只有项目链接没有 `image_id`，先通过项目接口拿页面列表。

### 2. 通过页面列表定位目标页面

- 在登录态页面内请求：

```text
/api/project/images?project_id=<pid>&team_id=<tid>&dds_status=1&position=1&show_cb_src=1&comment=1
```

- 从返回的 `data.images` 中按页面名定位目标页面，拿到：
  - `id`
  - `name`
  - `url`
  - `position_x`
  - `position_y`

- 其中 `id` 就是后续要用的 `image_id`。

### 3. 进入目标页面的画布态

- 使用目标页面的 `image_id` 切回 `stage`：

```text
#/item/project/stage?pid=<pid>&image_id=<image_id>&tid=<tid>
```

- 注意：
  - `detailDetach` 适合看标注，但在自动化环境下可能出现主图容器被算成 `0x0` 的空白问题。
  - 真正要拿切图资源时，优先停留在 `stage`。

### 4. 触发蓝湖“图片信息”

- 画布页组件通常挂在：

```text
document.querySelector('.project_stage_view').__vue__
```

- 先通过组件内部的 `stage.imagesIdMap[image_id]` 取到当前页面对象。
- 再调用蓝湖自身能力：

```text
vm.doShowImageInfo(item)
```

- 这一步会弹出“图片信息”对话框，里面包含：
  - `meta`
  - `assets`
  - `artboard`
  - `artboard.layers`

## 解析规则

### 1. 资源优先级

- 先看 `artboard.layers` 中具体图层的 `hasExportImage` / `hasExportDDSImage`。
- 再结合图层的 `frame` / `combinedFrame` 定位目标区域。
- 最后到顶层 `assets` 中取蓝湖生成的 `FigmaSlicePNG` / `FigmaSliceSVG` 资源。

### 2. 如何匹配目标图层

- 如果用户给的是“第一个文章封面”“列表第一个卡片图”，优先按几何位置匹配：
  - 左侧区域
  - 指定 `top`
  - 指定 `width/height`
- 如果用户给的是明确图层名，例如按钮、图标、封面图层名，则优先按：
  - `name`
  - `id`
  - `path`

### 3. 输出内容

- 至少返回：
  - 页面名
  - 命中的图层名或图层 ID
  - 切图尺寸
  - 切图 URL
- 如果用户要继续批量提取，补充：
  - 页面内可导出图层总数
  - 各图层坐标和资源清单

## 已验证的稳定路径

以下路径已在真实项目中验证可用：

1. 用 `/api/project/images` 根据页面名拿到 `image_id`
2. 切到 `stage` 画布页
3. 通过 `.project_stage_view.__vue__` 取到页面对象
4. 调用 `doShowImageInfo(item)` 拉起“图片信息”
5. 解析 JSON 中的 `artboard.layers`
6. 根据目标区域定位图层
7. 打开对应 `FigmaSlicePNG` / `FigmaSliceSVG` 地址验证资源

## 常见异常与处理

### 1. `detailDetach` 页面空白

- 现象：标题已变成“标注-xxx”，但页面主图不显示。
- 根因：蓝湖详情页中的 `.image_box`、`.detail-image`、`.mini-image` 被算成 `0x0`。
- 处理：
  - 不把它当成“无数据”
  - 切回 `stage`
  - 改走 `doShowImageInfo()` 和资源 JSON

### 2. 右侧切图面板没有正常展开

- 现象：点击页面对象后只出现悬浮条，没有出现完整右侧下载面板。
- 处理：
  - 继续走蓝湖内部的“图片信息”能力
  - 以 `FigmaSlicePNG` / `FigmaSliceSVG` 资源 URL 为准

### 3. 只拿到整张稿图，没拿到图层切图

- 处理顺序：
  - 先确认是否真的弹出了“图片信息”
  - 再确认 `artboard.layers` 中有没有 `hasExportImage`
  - 最后再做图层与资源的映射，不要直接裁整图

## 结果校验

- 打开目标切图 URL，确认尺寸与目标区域一致。
- 若是封面图，需和整张页面中的目标卡片视觉内容一致。
- 若用户要求“原生切图”，结果中不应出现“本地裁剪自整张稿图”的描述。

## 回复建议

- 先说明命中的页面和图层。
- 再给出切图 URL 和尺寸。
- 如果用户确认无误，再继续批量提取同页或全项目其它切图。
