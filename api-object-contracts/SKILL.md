---
name: api-object-contracts
description: 设计、新增、评审或重构 Java、Spring Boot、Vue、小程序、H5 项目的 HTTP 接口请求参数和响应体时使用。当接口存在三个及以上业务字段（不含分页、排序字段）时，必须使用具名请求/响应对象。
---

# 接口对象约定

使用明确的 DTO/VO 类，让调用方通过类型和 Swagger 字段说明了解接口，而不是猜测 `Map` 的键名。

## 规则

- 只统计业务字段；`pageNum`、`pageSize`、`orderByColumn`、`isAsc`、日期范围辅助字段和框架上下文字段不计入。
- 请求包含**三个及以上**业务字段时，使用一个具名请求 DTO/命令对象。GET 查询也使用查询 DTO，不要散落多个 `@RequestParam`。
- 响应包含**三个及以上**关联业务字段时，返回具名响应 VO/DTO。公开接口不得以 `Map<String, Object>`、匿名 JSON 或持久化实体作为契约。
- 分页响应保持 `PageR<ItemResponse>` 或项目等价类型；分页包装对象不能代替列表项响应对象。
- 字段按业务含义命名；公开契约字段添加 `@ApiModelProperty` 或项目等价说明。
- 请求 DTO、响应 VO、持久化实体和第三方载荷必须分离；不得暴露审计字段、内部 ID、密钥、对象键或第三方原始诊断数据。
- 本项目所有 API 请求 DTO 和响应 VO 均放在 `lanyan-system/src/main/java/com/lanyan/system/domain/dto`。请求对象命名为 `XxxRequest`，响应对象命名为 `XxxResponse`；不得在 Controller 内定义或分散存放接口对象。

## 例外

- 单个上传文件、一个路径 ID 或一个简单开关可以保留直接参数。
- 业务字段不超过两个，且含义明确、无需组合校验时，可以保留直接参数。

## 示例

```java
@PostMapping("/reports")
public R<ReportCreateResponse> create(@Valid @RequestBody ReportCreateRequest request) {
    return R.ok(reportService.create(request));
}

@Data
public class ReportCreateRequest {
    @ApiModelProperty("检测图片地址")
    private String imageUrl;

    @ApiModelProperty("顾客名称")
    private String customerName;

    @ApiModelProperty("测肤年龄")
    private Integer age;
}
```

## 检查

完成前，统计每个变更接口请求和响应两端的业务字段。符合条件的零散参数或 `Map` 响应必须改为具名对象，并同步更新调用方和 Swagger 输出。
