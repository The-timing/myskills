---
name: "lanyan-ui-layout"
description: "提供基于 LanYan 项目风格的简单 UI 排版规范，特别适用于后台管理系统中的列表页和详情展示。当用户需要调整页面布局风格时调用。"
---

# LanYan Simple UI Layout Skill

本技能基于 `busUser/index.vue` 的实现，提取了一套简单的 UI 排版规范，适用于后台管理系统中的数据展示。

## 1. 列表页自定义列模板

在 `el-table` 中，为了展示丰富的信息而不增加过多的列，推荐使用自定义模板 (`template slot-scope="scope"`) 将相关信息聚合在同一列中展示。

### 规范
- **对齐方式**: `align="center"` (表头居中)，内部内容使用 `text-align: left` (左对齐) 以提高可读性。
- **内容结构**: 使用 `div` 堆叠显示多行信息。
- **字段标签**: 使用中文标签 + 冒号 + 值的形式 (e.g., `用户ID: {{ scope.row.userId }}`)。
- **图片展示**: 使用 `<image-preview>` 组件，统一设置宽高 (e.g., `:width="50" :height="50"`)。

### 示例代码

```html
<el-table-column label="基本信息" align="center" prop="init" width="250">
  <template slot-scope="scope">
    <div style="text-align: left;">
      <div>头像: <image-preview :src="scope.row.avatarImg" :width="50" :height="50" /></div>
      <div>用户ID: {{ scope.row.userId }}</div>
      <div>昵称: {{ scope.row.nickname }}</div>
      <div>手机号: {{ scope.row.phone }}</div>
      <div>注册时间: {{ scope.row.createTime }}</div>
    </div>
  </template>
</el-table-column>
```

## 2. 关联信息展示

对于存在层级关系的数据（如推荐人、上级），应在前端进行空值检查 (`v-if`)，避免渲染错误。

### 规范
- **空值检查**: 使用 `v-if="scope.row.relationField"` 包裹内容。
- **字段聚合**: 将关联对象的多个属性（头像、ID、名称等）聚合在一个单元格内展示。

### 示例代码

```html
<el-table-column label="上级推荐人" align="center" prop="referrer" width="250">
  <template slot-scope="scope">
    <div v-if="scope.row.referrer" style="text-align: left;">
      <div>推荐人头像: <image-preview :src="scope.row.referrer.avatarImg" :width="50" :height="50" /></div>
      <div>推荐人ID: {{ scope.row.referrer.userId }}</div>
      <div>推荐人名称: {{ scope.row.referrer.nickname }}</div>
    </div>
  </template>
</el-table-column>
```

## 3. 搜索栏布局

使用 `el-form` + `inline` 属性实现顶部搜索栏。

### 规范
- **表单属性**: `:inline="true"`, `size="small"`, `label-width="68px"`。
- **日期范围**: 使用 `el-date-picker` (`type="datetimerange"`)，宽度统一为 `240px`。
- **操作按钮**: 搜索和重置按钮放在最后一个 `el-form-item`。

### 示例代码

```html
<el-form :model="queryParams" ref="queryForm" size="small" :inline="true" label-width="68px">
  <el-form-item label="注册时间">
    <el-date-picker v-model="daterangeCreateTime" style="width: 240px" value-format="yyyy-MM-dd HH:mm:ss"
      type="datetimerange" range-separator="-" start-placeholder="开始日期" end-placeholder="结束日期"></el-date-picker>
  </el-form-item>
  <el-form-item label="昵称" prop="nickname">
    <el-input v-model="queryParams.nickname" placeholder="请输入昵称" clearable @keyup.enter.native="handleQuery" />
  </el-form-item>
  <el-form-item>
    <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
    <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
  </el-form-item>
</el-form>
```
