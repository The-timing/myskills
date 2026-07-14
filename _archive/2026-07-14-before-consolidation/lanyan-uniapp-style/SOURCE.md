---
name: "lanyan-uniapp-style"
description: "定义 LanYan 项目小程序（uniapp）端前端页面的通用样式和布局规范，特别是自定义导航栏悬浮和背景色同步。当用户需要编写小程序页面或调整导航栏时调用。"
---

# LanYan UniApp 样式规范

此技能定义了 LanYan 项目中 UniApp (小程序) 端页面的通用 UI 和布局规范，确保小程序内各页面的体验一致性。

## 1. 核心需求：自定义导航栏（头部）规范

在编写包含自定义头部（通常是 `<view class="nav-bar">` 或类似类名）的页面时，**必须**遵循以下规范：

### 1.1 头部悬浮固定 (Sticky/Fixed)
- **要求**：所有的页面如果含有自定义头部，必须使其悬浮在页面顶部，页面向下滚动时头部不能跟随滚动消失。
- **实现方式**：
  - 首选 `position: sticky; top: 0; z-index: 999;`。
  - 在特定场景下（例如背景是整张大图），可以使用 `position: fixed; top: 0; left: 0; width: 100%; z-index: 999;`。

### 1.2 头部背景色同步 (Background Color)
- **要求**：头部导航栏的背景色必须与页面相应位置（顶部区域）的背景色保持同步和一致，避免向下滚动时文字或内容透底。
- **实现方式**：
  - 如果页面顶部是纯色背景（如 `#F8D161`、`#FCF9F2` 或 `#FFFFFF`），导航栏的 `background-color` 必须设置为相同的颜色。
  - 如果页面顶部是渐变色背景（如 `linear-gradient(180deg, #FDE3A0 0%, ...)`），导航栏的 `background-color` 必须设置为渐变色的起始颜色（例如 `#FDE3A0`）。
  - 如果页面顶部是图片，且设计要求背景图透出，则将导航栏设置为 `background-color: transparent;`。

## 2. 示例代码

### 纯色/渐变起始色背景
```html
<template>
  <view class="container">
    <view class="top-bg"></view> <!-- 可能是渐变背景 -->
    <view class="nav-bar" :style="{ paddingTop: statusBarHeight + 'px' }">
      <!-- 导航栏内容 -->
    </view>
    <!-- 页面内容 -->
  </view>
</template>

<style scoped>
/* 导航栏 */
.nav-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 44px;
  padding: 0 30rpx;
  
  /* 必须包含的悬浮与背景色属性 */
  position: sticky;
  top: 0;
  z-index: 999;
  background-color: #FDE3A0; /* 必须与页面顶部背景或渐变起始色一致 */
}
</style>
```

## 3. 其他注意事项
- 适配刘海屏：使用 `uni.getSystemInfoSync().statusBarHeight` 动态设置导航栏的 `paddingTop`。
- 底部安全区：使用 `padding-bottom: env(safe-area-inset-bottom);` 来适配 iPhone 底部小黑条。
