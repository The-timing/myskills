---
name: "lanyan-code-style"
description: "根据 LanYan 项目特定的模式（如 RuoYi/MyBatis-Plus）生成或重构代码。当用户要求生成/修复代码或“基于项目风格”时调用。"
---

# LanYan Project Code Style Guide

本技能提供了 LanYan 项目生成和重构代码的综合指南，确保与基于 RuoYi-Vue 的现有架构保持一致。

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
  - `@TableField(exist = false)`: 用于关联字段（例如 `LittleClass` 中的 `TeacherVo teacher`）或任何追加的暂存字段（如前端传入的 List 列表结构等非单列字段）。如果没有添加此注解，会导致 MyBatis 报错 `Type handler was null on parameter mapping...`。
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
  - 由于继承原因，标准的 Lombok `@Builder` 通常不够用；如果没有使用 `@SuperBuilder`，首选自定义静态内部类 `Builder`。

### 1.5 Excel 对象
- **类**: `DomainExcel` (通常在 `domain.excel.module` 包中)。
- **用途**: 定义 Excel 导出的结构。
- **注解**:
  - `@Data`
  - 来自 `cn.afterturn.easypoi.excel.annotation.Excel` 的 `@Excel(name = "列名")`。
- **用法**: 在 Controller 的 `export` 方法中使用，将 Entity/VO 数据映射为 Excel 行。

## 2. Controller 模式

- **基类**: 必须继承 `com.lanyan.common.core.controller.BaseController`。
- **注解**:
  - 类: `@RestController`, `@RequestMapping("/module/domain")`, `@Api(tags = {"标签名"})`。
- **方法**:
  - **列表/分页 (List/Page)**:
    ```java
    @GetMapping("/page")
    @ApiOperation("查询列表")
    @ApiImplicitParams({
        @ApiImplicitParam(name = "pageNum", value = "页码", dataType = "int", dataTypeClass = int.class),
        @ApiImplicitParam(name = "pageSize", value = "每页数量", dataType = "int", dataTypeClass = int.class)
    })
    public PageR<DomainVo> list(@ApiIgnore DomainSub domain) {
        startPage(); // 来自 BaseController
        return service.selectDomainVoList(domain);
    }
    ```
  - **获取详情 (Get Info)**:
    ```java
    @GetMapping(value = "/info")
    @ApiOperation("获取详情")
    public R<DomainSub> getInfo(Long id) {
        return R.ok(service.selectDomainByDomainId(id));
    }
    ```
  - **新增/修改 (Add/Update)**:
    - 使用 `@Log(title = "模块", businessType = BusinessType.INSERT/UPDATE)`
    - 使用 `@RepeatSubmit` 防止重复提交。
    - 返回 `AjaxResult` 或 `R<DomainSub>`。
    ```java
    @Log(title = "Domain", businessType = BusinessType.INSERT)
    @PostMapping("save")
    @RepeatSubmit
    public R<DomainSub> add(@ApiIgnore @RequestBody DomainSub domain) {
        // 校验和逻辑
        service.insertDomain(domain);
        return R.ok(domain);
    }
    ```
  - **删除 (Delete)**:
    - 使用 `@Log` 配合 `BusinessType.DELETE`.
    - 返回 `AjaxResult`.
  - **导出 (Export)**:
    - 使用 `@PreAuthorize("@ss.hasPermi('module:domain:export')")`.
    - 使用 `@Log(title = "标题", businessType = BusinessType.EXPORT)`.
    - 返回 `void`，接收 `HttpServletResponse`.
    - 使用 `MyExcelUtils.exportExcel`.
    ```java
    @PreAuthorize("@ss.hasPermi('system:domain:export')")
    @Log(title = "Domain Export", businessType = BusinessType.EXPORT)
    @PostMapping("/export")
    public void export(HttpServletResponse response, DomainSub domain) {
        // 权限检查或数据过滤
        List<DomainSub> list = service.selectDomainList(domain);
        List<DomainExcel> data = list.stream().map(x -> {
            DomainExcel excel = new DomainExcel();
            BeanUtils.copyProperties(x, excel);
            // 处理特定字段（例如：根据 ID 获取名称）
            return excel;
        }).collect(Collectors.toList());

        MyExcelUtils.exportExcel(data, "SheetName", "Title", DomainExcel.class, "filename.xlsx", response);
    }
    ```

- **返回类型**:
  - `PageR<T>`: 用于分页列表。
  - `R<T>`: 用于单个对象或无返回值操作。
  - `AjaxResult`: 基于 Map 的返回（常用于简单的成功/失败消息）。

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
      - **原因**: SQL 简单，缓存更好，符合 Mybatis-Plus 哲学。
      - **示例**:
        ```java
        List<LittleClassSub> list = service.list(wrapper);
        List<Long> teacherIds = list.stream().map(LittleClassSub::getTeacherId).collect(Collectors.toList());
        Map<Long, TeacherSub> teacherMap = teacherService.listByIds(teacherIds).stream()
            .collect(Collectors.toMap(TeacherSub::getTeacherId, t -> t));
        list.forEach(item -> item.setTeacher(teacherMap.get(item.getTeacherId())));
        ```
    - **选项 2: Wrapper + 自定义 SQL (用于复杂过滤/排序)**:
      - 在 Service 中使用 `QueryWrapper` 构建条件（包括 `FIND_IN_SET` 或距离计算等动态 SQL）。
      - 将 wrapper 传递给注解了 `@Select` 和 `${ew.customSqlSegment}` 的 Mapper 方法。
      - **示例**:
        ```java
        // Service
        public List<Merchant> selectList(Merchant merchant) {
            QueryWrapper<Merchant> wrapper = new QueryWrapper<>();
            wrapper.eq("m.del_flag", "0").leftJoin("product p on m.id = p.m_id"); // 逻辑
            if (merchant.hasParams()) {
                wrapper.between("m.create_time", start, end);
            }
            // 包含计算的自定义查询
            wrapper.select("m.*", "calculation_sql as distance");
            return mapper.selectJoinList(wrapper);
        }

        // Mapper
        @Select("select ${ew.sqlSelect} from merchant m left join product p on m.id = p.m_id ${ew.customSqlSegment}")
        List<Merchant> selectJoinList(@Param("ew") QueryWrapper wrapper);
        ```

## 4. 编码规范与最佳实践

- **导入 (Imports)**:
  - **严格检查**: 移除未使用的导入。
  - **格式**: 显式导入类。
- **注释规范**:
  - `CRUD` 常规增删改查代码默认少注释，避免把显而易见的代码重复翻译一遍。
  - 非 `CRUD` 的业务逻辑必须补充直白明确的注释，尤其是业务判断、状态流转、对象组装、映射转换、匹配规则、兜底分支、补偿释放等场景。
  - 注释优先解释“为什么这样做”和“这段逻辑在解决什么业务问题”，不要写“给变量赋值”这类同义复述式注释。
- **工具类 (Utils)**:
  - 字符串: `com.lanyan.common.utils.StringUtils`
  - 日期: `com.lanyan.common.utils.DateUtils`
  - 集合: `com.baomidou.mybatisplus.core.toolkit.CollectionUtils` 或 `java.util.stream`
- **Lombok 用法**:
  - 始终使用 `@Data`。
  - 对于子类，使用 `@EqualsAndHashCode(callSuper = true)` 和 `@ToString(callSuper = true)`。
- **空安全**:
  - 使用对象前始终检查 `getById()` 是否返回 null。
  - 使用前验证配置/设置。

## 5. 用户规则 (关键)

1.  **严格的导入检查**:
    - 引入新依赖或类时（如 `Date`, `List`, `@Excel`, `@TableField`），**严格检查**并手动添加缺失的 import 语句。
    - **清理**: 绝对**禁止**保留未使用的 import 语句和注解。

2.  **Lombok 使用规则**:
    - 在子类上使用 `@Data` 或类似注解时，必须确保处理好继承关系（例如在 `@EqualsAndHashCode` 中使用 `callSuper = true`）。

3.  **自动持久化**:
    - 代码更改必须立即持久化到磁盘。避免“未保存”状态。

4.  **保留字处理**:
    - 如果实体字段名是数据库保留字（例如 `order`, `group`, `desc`, `asc`, `limit`, `key`, `index`, `type`, `status`, `value`, `name`, `grade`, `subject`, `week`, `online`, `offline`, `character`, `state`, `sort`, `is_group`），你**必须**添加 `@TableField("`field_name`")` 注解，并将字段名用反引号包裹。

5.  **环境与框架**:
    - 项目: LanYan-Vue-master
    - 语言: Java (MyBatis-Plus + Lombok)
    - 框架: RuoYi SpringBoot 前后端分离版本

6.  **回复语言**:
    - 所有回复和注释应使用 **中文**。

## 6. 项目技能与模式 (核心记忆)

1.  **UI 数据填充**:
    - 为 UI 视图（如“收到的赞”）添加列表端点时，确保 VO 包含所有显示字段（头像、昵称、内容缩略图/标题），如果基础实体查询中不可用，则在 Service 层填充它们。

2.  **实体构建者模式**:
    - 优先使用 **构建者模式** 创建对象（特别是 Entity/Sub/VO）。如果类提供了 `builder()` 方法，使用它代替 `new` + setter，以提高可读性。

3.  **数据敏感性分离**:
    - **VO**: 低敏感（公开）数据。
    - **Entity**: 高敏感（私有）数据。
    - **Entity 继承 VO** 以继承公开字段。

4.  **多租户 (可选)**:
    - 大多数项目**不**需要多租户。
    - 仅在明确要求时：表应包含 `tenant_id` 列（`varchar(20) default '000000'`），并且 Entity 继承 `BaseEntity` 来处理它。

## 7. 高级最佳实践 (优化)

为了进一步提升代码质量和可维护性：

### 7.1 常量与枚举
- **无魔法值**: 避免硬编码的字符串或数字（例如 `if (status == 1)`）。
- **使用枚举**: 为固定状态创建枚举类（例如 `PayStatus`, `OrderType`）以提高可读性和类型安全。
- **常量类**: 使用 `Constants.java` 或特定的 `DomainConstants` 存放共享常量值。

### 7.2 代码简洁性
- **Stream API**: 优先使用 Java 8 Stream API 进行集合处理（过滤、映射、分组），代替传统循环，但要确保可读性。
- **提前返回**: 使用卫语句（提前返回）减少嵌套深度，避免“箭头代码”。
- **工具使用**: 优先使用 `Hutool` 或框架提供的工具（`StringUtils`, `DateUtils`），而不是编写自定义实现。

### 7.3 异常处理
- **业务异常**: 抛出 `ServiceException`（带有清晰的消息）来处理业务逻辑违规。
- **全局处理**: 依赖全局异常处理器处理标准 HTTP 响应；不要捕获异常后只打印堆栈跟踪而不重新抛出或处理。

### 7.4 日志记录
- **占位符语法**: 使用 SLF4J 占位符（`log.info("Order id: {}", id)`）代替字符串拼接。
- **适当的级别**:
  - `ERROR`: 系统故障，需要干预的意外异常。
  - `WARN`: 业务异常（例如库存不足、支付失败）。
  - `INFO`: 关键业务流程里程碑（例如订单创建、状态变更）。

### 7.5 数据库性能
- **选择特定列**: 在自定义 SQL 中避免 `select *`；只获取需要的字段。
- **批量操作**: 使用 `saveBatch` 或 `updateBatchById` 处理多条记录，减少数据库往返次数。
- **索引使用**: 确保查询命中数据库索引，特别是 `where`, `order by`, 和 `group by` 子句。

### 7.6 主键命名规范
- **命名规则**: 主键列必须命名为 `tablename_id`（忽略前缀）。
- **示例**:
  - 表 `bus_user` (前缀 `bus_`) -> 主键 `user_id`。
  - 表 `bus_member_type` -> 主键 `member_type_id`。
  - 表 `bus_course` -> 主键 `course_id`。
- **原因**: 确保连接条件的一致性和清晰度（例如，其他表中的 `user_id` 匹配 `user_id`），避免模棱两可的 `id` 列。

## 8. 前端开发模式

### 8.1 一对多关系 (主从) 模式
- **场景**: 编辑包含子实体列表的实体时（例如 订单 -> 订单项, VIP -> VIP项）。
- **UI 模式**: 在 `el-dialog` 中使用 `el-tabs` 将基本信息与子列表分开。
- **结构**:
  - **Tab 1 (基本设置)**: 包含主实体属性的 `el-form`。
  - **Tab 2+ (子列表)**: 包含管理子列表的独立组件。
- **实现**:
  - 通过 `props` 将子列表数组从父表单传递给子组件。
  - 监听来自子组件的更新事件（例如 `@backList` 或 `@update:list`）以更新父表单数据。
  - 如果需要，使用 `:key` 强制重新渲染（例如 `:key="Math.random()*10000"`）。
- **示例**:
  ```html
  <el-dialog :title="title" :visible.sync="open" width="1000px" append-to-body>
    <el-tabs v-model="form.activeName">
      <!-- Tab 1: 主实体字段 -->
      <el-tab-pane label="基本设置" name="first">
        <el-form ref="form" :model="form" :rules="rules" label-width="120px">
          <el-form-item label="名称" prop="name">
            <el-input v-model="form.name" />
          </el-form-item>
          <!-- 其他字段 -->
        </el-form>
      </el-tab-pane>

      <!-- Tab 2: 子实体列表 -->
      <el-tab-pane label="会员权益" name="second">
        <!-- 通过 props 传递列表并监听更新 -->
        <VipItem 
           :key="Math.random()*10000" 
           :vip-item-list="form.vipItem"
           @backList="(e) => form.vipItemList = e"/>
      </el-tab-pane>
    </el-tabs>
    <div slot="footer" class="dialog-footer">
      <el-button type="primary" @click="submitForm">确 定</el-button>
      <el-button @click="cancel">取 消</el-button>
    </div>
  </el-dialog>
  ```

## 9. DTO 返回与注解规范（项目要求）

- 对接口返回含多个字段的响应，优先采用对象（DTO/VO）返回，而非 Map。
- 复用优先：先检查是否已有满足需求的对象（VO/DTO）。存在则复用；不存在则新建轻量 DTO，避免与实体（Entity）耦合。
- 新建 DTO 的放置路径固定为：
  - `lanyan-system/src/main/java/com/lanyan/system/domain/dto`
- DTO 字段注解要求：
  - 所有字段必须添加 `@ApiModelProperty("字段含义")`，用于 Swagger 文档说明。
  - 日期字段必须同时添加：
    - `@JSONField(format = "yyyy-MM-dd HH:mm:ss")`
    - `@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")`
  - 金额类字段统一使用 `BigDecimal` 类型。
- 控制器返回类型更新范式：
  - 将 `R<Map<String, Object>>` 等改为具体对象，如 `R<UserProfileDTO>`、`R<WalletInfoDTO>`。
- Import 检查与清理：
  - 严格手动添加缺失的 import（如 `Date`、`BigDecimal`、`ApiModelProperty`、`JSONField`、`JsonFormat` 等）。
  - 禁止保留未使用的 import 与注解。

## 10. 二值字段 UI 规范（el-switch）

- 适用范围：取值仅有两种状态（如 0/1、true/false、启用/停用、展示/不展示）的字段。
- 列表展示与操作：
  - 在 `el-table` 中使用 `el-switch` 直接进行切换操作。
  - 使用 `:active-value="1"`、`:inactive-value="0"` 映射后端枚举值。
  - 在 `@change` 事件中调用更新接口（如 `updateXxx({ id, status })`），保持与后端状态同步。
  - 示例：
    ```html
    <el-table-column label="首页展示" align="center" prop="status">
      <template slot-scope="scope">
        <el-switch
          v-model="scope.row.status"
          :active-value="1"
          :inactive-value="0"
          @change="handleStatusChange(scope.row)"
        />
      </template>
    </el-table-column>
    ```
    ```js
    // 切换事件：仅提交变更字段与主键
    handleStatusChange(row) {
      updateBusCategory({
        categoryId: row.categoryId,
        status: row.status
      })
    }
    ```
- 表单编辑：
  - 在 `el-form-item` 中使用 `el-switch`，并设置 `active-text` / `inactive-text` 以增强可读性。
  - 示例：
    ```html
    <el-form-item label="首页展示" prop="status">
      <el-switch
        v-model="form.status"
        :active-value="1"
        :inactive-value="0"
        active-text="展示"
        inactive-text="不展示"
      />
    </el-form-item>
    ```
- 统一规范：
  - 前端与后端的二值枚举值必须一致（推荐使用 1/0）。
  - 切换操作应为轻量更新，仅提交必要字段，避免全量对象更新。

## 11. Java 语法检查清单（自动化验证后补充）

为避免常见的编译错误，生成代码时请务必检查以下点：

1.  **完整性检查**：
    - 类必须有完整的包声明 (`package ...;`)。
    - 所有引用的外部类必须有 `import` 语句（特别是 DTO、Entity、Service、Utils）。
    - 确保大括号 `{}` 成对匹配。

2.  **方法定义**：
    - 方法必须有返回类型（构造函数除外）。
    - 带有返回值的方法必须包含 `return` 语句（除非抛出异常）。
    - 注解（如 `@PostMapping`）必须正确闭合括号。

3.  **常见错误模式**：
    - **中文符号**：严禁在代码逻辑中使用中文分号 `；`、括号 `（）`。
    - **拼写错误**：检查关键字（如 `implements`, `extends`, `return`）拼写。
    - **注解参数**：`@ApiImplicitParams` 内的 `@ApiImplicitParam` 必须以逗号分隔，且大括号正确闭合。

4.  **Lombok 陷阱**：
    - 使用 `@Data` 时，确保依赖库已正确引入。
    - 继承类建议加 `@EqualsAndHashCode(callSuper = true)`。
