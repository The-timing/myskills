---
name: "lanyan-backend-controller"
description: "生成符合 LanYan 项目规范的 Controller 代码。当用户需要创建新接口或控制器时调用。"
---

# LanYan Backend Controller Generator

本技能用于生成或重构符合 LanYan 项目 Controller 规范的代码。

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
  - **导出 (Export)**:
    - 使用 `@PreAuthorize("@ss.hasPermi('module:domain:export')")`。
    - 使用 `@Log(title = "标题", businessType = BusinessType.EXPORT)`。
    - 返回 `void`，接收 `HttpServletResponse`。
    - 使用 `MyExcelUtils.exportExcel`。

### 9. DTO 返回与注解规范（项目要求）
- 对接口返回含多个字段的响应，优先采用对象（DTO/VO）返回，而非 Map。
- 控制器返回类型更新范式：
  - 将 `R<Map<String, Object>>` 等改为具体对象，如 `R<UserProfileDTO>`。

## 5. 用户规则 (关键)
- **严格的导入检查**: 引入新依赖或类时，严格检查并手动添加缺失的 import 语句。
- **前后端接口同步**: 修改前端页面涉及数据交互时，必须先检查后端 Controller 是否存在对应接口。若不存在，需生成或修改后端接口（Controller/Service/Entity）。
- **回复语言**: 所有回复和注释应使用 **中文**。
