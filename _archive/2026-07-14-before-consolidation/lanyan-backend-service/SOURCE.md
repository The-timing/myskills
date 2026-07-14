---
name: "lanyan-backend-service"
description: "生成符合 LanYan 项目规范的 Service 代码。当用户需要编写业务逻辑或数据库操作时调用。"
---

# LanYan Backend Service Generator

本技能用于生成或重构符合 LanYan 项目 Service 规范的代码。

## 3. Service 层

- **接口**: `IDomainService` 继承 `IService<Domain>`。
- **实现**: `DomainServiceImpl` 继承 `ServiceImpl<DomainMapper, Domain>` 实现 `IDomainService`。
- **数据库操作**:
  - 使用 `lambdaQuery()` 和 `lambdaUpdate()` 进行类型安全的操作。
  - **并发控制**: 对敏感字段（如余额、积分、库存）使用原子更新（乐观锁）。
    ```java
    // 原子更新示例
    service.lambdaUpdate()
        .setSql("balance = balance + " + amount)
        .eq(Entity::getId, id)
        .update();
    ```
  - **事务**: 在涉及多表更新的方法上使用 `@Transactional(rollbackFor = Exception.class)`。
  - **关联查询 (复杂)**:
    - **选项 1: 内存组装 (推荐用于简单关系)**:
      - 先获取主列表，然后根据 ID 获取相关数据（在 `Service` 或 `Controller` 中），并在内存中组装（Stream API）。
    - **选项 2: Wrapper + 自定义 SQL (用于复杂过滤/排序)**:
      - 在 Service 中使用 `QueryWrapper` 构建条件，传递给注解了 `@Select` 和 `${ew.customSqlSegment}` 的 Mapper 方法。

### 3.1 渲染字段聚合默认动作

- 当接口要服务于 `列表展示`、`详情回显`、`tag 渲染`、`名称展示` 时，先检查返回对象里是否存在 `xxxId`、`xxxIds` 一类外键字段。
- 如果这些外键字段会直接被前端展示，默认在 Service 层补齐展示型聚合字段，不把“根据 ID 再查名称”的责任留给前端。
- 推荐实现顺序：
  - 先收集主列表中的外键 ID
  - 批量查询关联对象
  - 构建 `ID -> 业务展示信息` 映射
  - 统一回填到主对象的扩展字段中
- 列表和详情尽量共用同一套聚合方法，例如 `fillXxxInfo(list)`，避免列表和详情展示口径不一致。
- 原始外键字段保留用于提交、筛选、跳转；展示型扩展字段专门服务前端渲染。

## 4. 注释规范

- `CRUD` 常规增删改查代码默认少注释，避免制造注释噪音。
- 非 `CRUD` 的 Service 业务逻辑必须写直白明确的中文注释，尤其是业务规则判断、状态流转、金额或资源处理、对象组装、匹配排序、兜底策略、回滚补偿等代码块。
- 注释应优先说明“为什么这样处理”和“这段逻辑在保证什么业务结果”，不要只把代码语句再翻译一遍。

## 5. 高级最佳实践 (优化)
- **代码简洁性**: 优先使用 Java 8 Stream API，使用 `Hutool` 或框架提供的工具（`StringUtils`, `DateUtils`）。
- **异常处理**: 抛出 `ServiceException` 处理业务逻辑违规。
- **日志记录**: 使用 SLF4J 占位符（`log.info("Order id: {}", id)`）。
- **数据库性能**:
  - 在自定义 SQL 中避免 `select *`。
  - 使用 `saveBatch` 或 `updateBatchById` 处理多条记录。
  - 确保查询命中数据库索引。

## 6. 用户规则 (关键)
- **自动持久化**: 代码更改必须立即持久化到磁盘。
- **回复语言**: 所有回复和注释应使用 **中文**。
