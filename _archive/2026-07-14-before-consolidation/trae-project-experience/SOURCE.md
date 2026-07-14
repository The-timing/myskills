---
name: trae-project-experience
description: "收录从项目 .trae 中沉淀出来的规则、经验和知识库条目。处理 LanYan/RuoYi 风格后台、Spring Boot 后端、Vue2 ElementUI、小程序、订单、支付、动态配置、富文本、状态流转、公共接口、后台列表治理等问题时使用；先读本 Skill，再按需读取 references/rules 或 references/kb-entries 中的原始经验。"
---

# Trae 项目经验索引

## 什么时候用

遇到下面任务时，先用这个 Skill：

- 想复用项目 `.trae` 里沉淀过的经验。
- 做 LanYan/RuoYi 后台、Spring Boot 后端、Vue2 ElementUI、小程序接口。
- 涉及订单、支付、回调、状态流转、动态配置、富文本、后台列表、公共接口、数据库字段、VO 聚合。
- 不确定以前有没有处理过类似问题。

## 怎么用

1. 先看下面的“快速路由”。
2. 找到对应规则或知识库文件。
3. 读取 references 里的原文。
4. 再结合当前项目代码做判断，不要机械套用旧项目字段名。

## 快速路由

### 业务场景和总入口

优先读：

- `references/rules/project-logic-readme.md`
- `references/rules/scene-index.md`
- `references/rules/scene-handling-framework.md`
- `references/rules/project-memories.md`

适合：不知道任务属于哪类、需要先抽象业务场景、需要复用最近沉淀。

### 后台列表和页面治理

优先读：

- `references/rules/admin-list-scene-rule.md`
- `references/rules/admin-ui-render-rule.md`
- `references/kb-entries/KB-UI-00000007-admin-list-governance.md`
- `references/kb-entries/KB-UI-00000009-remote-filter-default-candidates.md`
- `references/kb-entries/KB-UI-00000015-vue2-form-reactivity.md`

适合：后台列表整理、字段回显、远程筛选、详情弹窗、Vue2 表单回填。

### 后端接口边界和公共读取

优先读：

- `references/rules/mini-program-api-rule.md`
- `references/rules/public-controller-reuse-rule.md`
- `references/kb-entries/KB-API-00000008-interface-boundary-layering.md`
- `references/kb-entries/KB-API-00000016-public-detail-anonymous-read-login-extension.md`

适合：小程序接口、公共接口、匿名读取、用户态/后台态职责边界。

### 订单、支付、状态、资源占用

优先读：

- `references/rules/order-payment-consistency-rule.md`
- `references/rules/product-stock-rule.md`
- `references/rules/status-enum-consistency-rule.md`
- `references/kb-entries/KB-FLOW-00000005-backend-fallback-first.md`
- `references/kb-entries/KB-FLOW-00000011-occupy-release-pairing.md`
- `references/kb-entries/KB-FLOW-00000012-formal-test-handler-reuse.md`
- `references/kb-entries/KB-FLOW-00000013-callback-side-effects-isolation-and-delay.md`
- `references/kb-entries/KB-STATE-00000006-summary-vs-detail-state.md`

适合：预览/下单一致性、0 元订单、支付回调、资源占用释放、状态分层。

### 动态配置和配置分层

优先读：

- `references/kb-entries/KB-BACKEND-00000020-config-layering-by-ownership.md`

并结合已收录 Skill：

- `lanyan-dynamic-settings`
- `operation-settings-content`

适合：bus_setting、sys_config、application.yml 三类配置归属判断。

### 富文本和 UEditor

优先读：

- `references/kb-entries/KB-UI-00000019-ueditor-wrapper-stability-and-upload-adapter.md`

并结合已收录 Skill：

- `lanyan-rich-text-editor`
- `rich-text-ueditor-flow`

适合：UEditor、上传适配、预览稳定、光标跳头、XSS 白名单。

### 微信支付和小程序发货

优先读：

- `references/kb-entries/KB-BACKEND-00000019-wechatpay-protocol-selection-by-cert-materials.md`
- `references/kb-entries/KB-BACKEND-00000021-wechat-v2-refund-use-orderRefundByProtocol.md`
- `references/kb-entries/KB-BACKEND-00000022-wechat-transfer-to-wallet-v3-prerequisites.md`
- `references/kb-entries/KB-BACKEND-00000023-wechat-miniapp-shipping-request-assembly.md`

适合：微信支付 V2/V3、退款、转账、发货信息接口。

### 设计和抽象

优先读：

- `references/kb-entries/KB-FRAME-00000002-general-scenario-abstraction.md`
- `references/kb-entries/KB-DESIGN-00000003-selection-vs-execution.md`
- `references/kb-entries/KB-DESIGN-00000004-multi-match-compatible-single.md`
- `references/kb-entries/KB-STYLE-00000014-business-logic-comments.md`

适合：选择态/执行态、多对象匹配、业务注释、场景抽象。

## 注意

这些经验来自具体项目，使用时要先抽象成通用场景，再落到当前项目。不要直接照搬表名、字段名、接口名；要复用判断方法和验收标准。