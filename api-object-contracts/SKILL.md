---
name: api-object-contracts
description: Use when designing, adding, reviewing, or refactoring HTTP API request parameters and response bodies in Java, Spring Boot, Vue, mini-program, or H5 projects. Require named request/response objects when an endpoint has three or more business fields, excluding pagination and sorting fields.
---

# API Object Contracts

Use explicit DTO/VO classes so callers can discover an interface from its type and Swagger schema instead of guessing map keys.

## Rules

- Count only business fields. Exclude `pageNum`, `pageSize`, `orderByColumn`, `isAsc`, date-range helper fields, and framework context fields.
- When a request has **three or more** business fields, accept one named request DTO/command object. For GET queries, bind a query DTO instead of declaring scattered `@RequestParam` arguments.
- When a response has **three or more** related business fields, return a named response VO/DTO. Do not use `Map<String, Object>`, anonymous JSON, or persistence entities as public contracts.
- Keep paged responses as `PageR<ItemResponse>` or the project equivalent. The page wrapper is not a substitute for an item response object.
- Name fields by business meaning; add `@ApiModelProperty` (or the project’s equivalent) to public contract fields.
- Keep request DTOs, response VOs, persistence entities, and third-party payload objects separate. Do not expose audit fields, internal IDs, secrets, object keys, or raw provider diagnostics.
- In this project, place every API request DTO and response VO under `lanyan-system/src/main/java/com/lanyan/system/domain/dto`. Name requests as `XxxRequest` and responses as `XxxResponse`; do not create controller-local classes or scatter contracts across controller packages.

## Exceptions

- A single upload file, one path ID, or one simple toggle may remain a direct parameter.
- Two or fewer business fields may remain direct parameters when their meaning is self-evident and no grouped validation is needed.

## Pattern

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

## Review

Before finishing, count business fields on both sides of every changed endpoint. Replace any qualifying scattered parameters or map response with a named object, then update callers and Swagger output together.
