---
name: "lanyan-database-design"
description: "根据 LanYan 项目规范设计数据库表结构。当用户需要设计数据库、建表或修改表结构时调用。"
---

# LanYan 数据库设计规范

当需要为项目设计新的数据库表时，必须严格遵循以下规范：

1. **表结构参考**：只参考已有表结构，不直接使用已有表。
2. **表名前缀**：所有新建的表必须以 `bus_` 为前缀。
3. **主键命名**：表主键命名规则为“去除 `bus` 前缀的表名 + `Id`”（例如表名为 `bus_order`，主键为  `order_id`）。
4. **基础字段及顺序**：
   在主键之后，必须紧跟以下基础字段，且顺序如下：
   - `del_flag`（默认值为0）
   - `create_by`
   - `create_time`（`create_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间'）
   - `update_by`
   - `update_time`（`update_time` timestamp NULL DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间'）
   - `remark`
5. **业务字段**：其他具体业务相关的字段请放置在这些基础字段之后。
