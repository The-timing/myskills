---
name: "lanyan-backend-entity"
description: "生成符合 LanYan 项目规范的后端实体类（Entity/VO/Sub）。当用户需要创建新实体或修改现有实体结构时调用。"
---

# LanYan Backend Entity Generator

本技能用于生成或重构符合 LanYan 项目架构规范的实体类代码。

## 1. 架构与继承模式

项目遵循严格的领域对象 3 层继承结构：

`Sub` (业务逻辑/扩展) -> `Entity` (数据库表) -> `Vo` (视图对象/基础属性) -> `BaseEntity`

### 1.1 基类 (Base Classes)
- **BaseEntity**: `com.lanyan.common.core.domain.BaseEntity`
  - 提供 `createTime`, `updateTime`, `createBy`, `updateBy`, `remark`, `searchValue`。
  - 注解：内部字段使用 `@JsonIgnore`，日期使用 `@JsonFormat`。

### 1.2 视图对象 (View Object - VO)
- **类**: `DomainVo` 继承 `BaseEntity`。
- **用途**: 定义 Entity 和 Sub 共享的字段，以及基础属性。
- **注解**:
  - `@Data`
  - `@Excel(name = "列名")`: 用于导出功能。
  - `@ApiModelProperty("描述")`: 用于 Swagger 文档。
  - `@RequiredField(update = true, delete = true)`: 用于主键。
  - `@TableId`: 用于主键。
  - `@TableField`: 用于标准字段。

### 1.3 实体 (Entity)
- **类**: `Domain` 继承 `DomainVo`。
- **用途**: 直接映射数据库表结构。
- **注解**:
  - `@Data`
  - `@EqualsAndHashCode(callSuper = true)`
  - `@ToString(callSuper = true)`
  - `@TableName("table_name")`
  - `@TableField(exist = false)`: 用于关联字段（例如 `LittleClass` 中的 `TeacherVo teacher`）或任何追加的暂存字段（如前端传入的 List 列表结构，非直接映射到单列的数据）。如果没有添加此注解，会导致 MyBatis 报错 `Type handler was null on parameter mapping...`。
- **保留字**: 如果字段名是数据库保留字（如 `order`, `status`, `group`），**必须**使用 `@TableField("`field_name`")` 并用反引号包裹。

### 1.4 子类 (Sub Class)
- **类**: `DomainSub` 继承 `Domain`。
- **用途**: 包含复杂业务逻辑、DTO 用法、非数据库扩展字段以及构建者模式。
- **注解**:
  - `@Data`
  - `@EqualsAndHashCode(callSuper = true)`
  - `@ToString(callSuper = true)`
  - `@TableName("table_name")`: 冗余但通常保留以确保 MyBatis 上下文正确。
- **构建者模式**:
  - 如果需要复杂的构造，**必须**实现一个返回自定义内部 Builder 类的静态 `builder()` 方法。

## 5. 用户规则 (关键)

1.  **严格的导入检查**:
    - 引入新依赖或类时（如 `Date`, `List`, `@Excel`, `@TableField`），**严格检查**并手动添加缺失的 import 语句。
    - **清理**: 绝对**禁止**保留未使用的 import 语句和注解。

2.  **Lombok 使用规则**:
    - 在子类上使用 `@Data` 或类似注解时，必须确保处理好继承关系（例如在 `@EqualsAndHashCode` 中使用 `callSuper = true`）。

3.  **主键命名规范**:
    - **命名规则**: 主键列必须命名为 `tablename_id`（忽略前缀）。
