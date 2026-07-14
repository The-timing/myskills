---
name: "lanyan-mybatis-plus-join"
description: "生成MyBatis-Plus连表查询代码（支持Lambda/分页/DTO映射）。当用户需要实现多表关联、复杂报表或解决N+1问题时调用。"
---

# LanYan MyBatis-Plus 连表查询规范

本技能用于生成符合 LanYan 项目规范的 MyBatis-Plus 多表关联查询代码。核心模式为：**Service层构建Wrapper条件 + Mapper层自定义SQL注解**。

## 适用场景
- 需要关联 2 个及以上表查询时
- 单表查询无法满足的复杂条件（如 `HAVING`, `GROUP BY` 配合连表）
- 需要直接映射结果到 DTO/VO 以避免 N+1 问题时

## 核心模式

### 1. Mapper 接口 (Annotations)
使用 `@Select` 注解配合 `${ew.customSqlSegment}` 实现动态 SQL 注入。

**规范：**
- 必须使用 `@Param("ew") QueryWrapper wrapper` 参数。
- SQL 结尾必须包含 `${ew.customSqlSegment}`。
- 使用 `LEFT JOIN` / `INNER JOIN` 显式关联。
- 别名命名规范：主表 `t1` 或首字母（如 `orders` -> `o`），关联表 `t2` 或首字母。

```java
// 示例：OrdersMapper.java
@Select("select o.*, lc.title as littleClassTitle, lci.title as chapterTitle " +
        "from orders as o " +
        "left join little_class as lc on o.product_id = lc.little_class_id " +
        "left join little_class_item as lci on lc.little_class_id = lci.little_class_id " +
        "${ew.customSqlSegment}")
List<OrdersSub> selectOrdersJoinList(@Param("ew") QueryWrapper wrapper);

// 分页支持
@Select("select o.*, lc.title as littleClassTitle " +
        "from orders as o " +
        "left join little_class as lc on o.product_id = lc.little_class_id " +
        "${ew.customSqlSegment}")
IPage<OrdersSub> selectOrdersJoinPage(IPage<OrdersSub> page, @Param("ew") QueryWrapper wrapper);
```

### 2. Service 实现
在 Service 层构建 `QueryWrapper`，利用 MP 的灵活性处理动态条件。

**规范：**
- 使用 `QueryWrapper` 或 `LambdaQueryWrapper`。
- 字段引用必须带表别名（如 `o.del_flag`）。
- 复杂逻辑（如 `HAVING`）在此处拼接。

```java
// 示例：OrdersServiceImpl.java
@Override
public List<OrdersSub> selectOrdersJoinList(OrdersSub orders) {
    QueryWrapper<OrdersSub> wrapper = new QueryWrapper<>();
    // 基础条件：带别名
    wrapper.eq("o.del_flag", "0");

    // 动态条件
    if (StringUtils.isNotEmpty(orders.getType())) {
        wrapper.eq("o.type", orders.getType());
    }

    // 复杂聚合
    if ("0".equals(orders.getSearchType())) {
        wrapper.having("sum(case when lci.state = '0' then 1 else 0 end) > 0");
    }

    wrapper.groupBy("o.product_id");
    wrapper.orderByDesc("o.create_time");

    return ordersMapper.selectOrdersJoinList(wrapper);
}
```

## 生成步骤
1.  **分析需求**：确定主表、关联表及关联条件（1:1, 1:N）。
2.  **定义 VO/DTO**：在 `domain/sub` 或 `domain/vo` 包下确认或创建接收结果的实体，包含关联字段（如 `littleClassTitle`）。
3.  **编写 Mapper**：
    -   在 Mapper 接口添加 `@Select` 方法。
    -   编写 SQL，确保字段别名与 VO 属性匹配（`AS` 关键字）。
    -   添加 `${ew.customSqlSegment}`。
4.  **编写 Service**：
    -   实现接口方法。
    -   构建 `QueryWrapper`，添加 `eq`, `like`, `between` 等条件。
    -   **注意**：所有条件字段必须带表别名前缀！

## 质量门禁 (Quality Gates)
-   **零 SQL 注入**：除 `${ew.customSqlSegment}` 外，禁止在 Service 层使用字符串拼接 SQL 值，必须使用 Wrapper 的方法（如 `eq`, `like`）。
-   **元数据一致性**：SQL 中的字段名必须与数据库表结构 100% 一致。
-   **性能**：
    -   必须加上 `del_flag` 过滤（如果存在）。
    -   关联字段应有索引。
    -   只查询必要字段，避免大字段无用查询。
    -   单表 500w+ 数据量下，90% 查询耗时 < 200ms。
-   **N+1 避免**：禁止在循环中调用 SQL，必须通过连表一次性获取。
-   **代码覆盖率**：>80%，SonarQube 阻塞级问题为 0。
