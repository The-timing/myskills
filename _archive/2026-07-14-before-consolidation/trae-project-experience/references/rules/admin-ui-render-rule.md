---
alwaysApply: false
description: 后台管理页面渲染与排版规范
---
# 后台管理页面渲染规范

## 触发条件
当你在开发、修改或重构若依（RuoYi）后台管理页面时，如果任务重点是“渲染与排版效果”，而不是筛选策略、接口职责或业务状态流转，必须优先遵守此规范。

## 详细要求

### 1. 屏蔽开发级敏感信息
**关键字**: `敏感信息` | `字段屏蔽` | `el-table` | `若依后台`

**内容**: 
不要在前端页面（尤其是表格列和详情描述中）展示对业务用户无直接意义的系统级字段或开发敏感信息。

**规范**:
- **禁止直接展示的内容**: 逻辑删除标记（如 `delFlag` / `isDeleted`）、系统操作人标识（如 `createBy` / `updateBy`）
- **处理方式**: 在编写 `<el-table-column>` 或 `<el-descriptions-item>` 时，主动将这些字段剔除，不予渲染。

### 2. 详情展示优先按业务块分段
**关键字**: `详情弹窗` | `el-descriptions` | `业务分块`

**内容**:
后台详情页优先按“基础信息、金额信息、关联信息、时间信息、备注说明、子表信息”等业务块展示，不要直接复用只读表单堆字段。

**规范**:
- 优先使用 `el-dialog` 或 `el-drawer`
- 内部优先使用 `el-descriptions`
- 存在列表型关联数据时，使用内嵌 `el-table`
- 空值统一显示 `--`

### 3. 同类数据合并展示（避免列过多）
**关键字**: `数据合并` | `自定义列模板` | `UI排版` | `template slot`

**内容**: 
当表格或详情页需要展示的字段过多，导致页面横向滚动条过长、列被挤压或视觉拥挤时，严禁生硬罗列所有单独字段。必须将属于同一逻辑实体或具有强关联的多个字段合并到一个展示单元格中。

**规范**:
- **对齐方式**: `align="center"` (表头居中)，内部内容使用 `text-align: left` (左对齐) 以提高可读性。
- **内容结构**: 使用 `div` 堆叠显示多行信息，通过插槽（`<template slot-scope="scope">`）进行自定义排版和换行。
- **字段标签**: 使用中文标签 + 冒号 + 值的形式 (e.g., `联系人: {{ scope.row.contact }}`)。
- **层级处理**: 对于多组信息（如时间、长地址等），可以在一个单元格内分主次行展示。

**示例代码**:

```html
<el-table-column label="联系人与地址" align="center" prop="contactInfo" width="300">
  <template slot-scope="scope">
    <div style="text-align: left;">
      <div>姓名: {{ scope.row.contactName }}</div>
      <div>电话: {{ scope.row.contactPhone }}</div>
      <div>地址: {{ scope.row.province }}{{ scope.row.city }}{{ scope.row.district }}{{ scope.row.address }}</div>
    </div>
  </template>
</el-table-column>
```

```html
<el-table-column label="时间信息" align="center" prop="timeInfo" width="220">
  <template slot-scope="scope">
    <div style="text-align: left;">
      <div>创建时间: {{ scope.row.createTime }}</div>
      <div v-if="scope.row.updateTime">更新时间: {{ scope.row.updateTime }}</div>
    </div>
  </template>
</el-table-column>
```

### 4. 图片、长文本、状态字段按合适组件渲染
**关键字**: `image-preview` | `el-tag` | `长文本`

**内容**:
不同数据类型必须用合适的组件展示，不能一律当普通文本回显。

**规范**:
- 图片链接优先用 `image-preview`
- 状态字段优先用 `el-tag`、`dict-tag` 或 `el-switch`
- 长文本、原因说明、备注说明优先多行展示
- 二值字段默认联动 `el-switch` 规范，具体交互参考相关前端技能

### 5. 与其他规则的边界
**内容**:
本规则只负责“如何展示”，不重复定义后台列表治理、状态 tabs、远程用户筛选、只读页收敛、接口分层等内容。

**联动规则**:
- 列表治理、tabs、默认筛选、远程用户筛选：
  - 参考 `admin-list-scene-rule.md`
- 只读页收敛、按钮收敛：
  - 参考 `admin-list-scene-rule.md`
- 资金类页面的审核按钮与状态机：
  - 参考 `finance-audit-flow-rule.md`
- 订单/支付/预览一致性：
  - 参考 `order-payment-consistency-rule.md`
