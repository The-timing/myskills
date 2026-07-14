---
alwaysApply: true
description: Trae 国际版稳定开发流程
---
# Trae 国际版稳定开发流程规则

## 触发条件

全局生效。用户在 Trae 国际版中提出任何开发、修复、重构、整理、生成页面、生成接口、排查问题或沉淀经验类需求时，默认按本规则执行。

本规则的目标是让 Trae 自动使用当前项目的 `.trae` 工程体系，而不是每次依赖用户重复粘贴 Prompt。

## 1. 固定读取顺序

处理需求前，必须按以下顺序建立上下文：

1. 读取项目根目录 `README.md`，并把它作为当前项目唯一项目画像
2. 读取 `.trae/README.md`
3. 读取 `.trae/rules/scene-index.md`
4. 根据业务主线读取对应 `.trae/rules/*-rule.md`
5. 如果问题较复杂，读取 `.trae/rules/scene-handling-framework.md`
6. 如果问题具备通用处置价值，检索 `.trae/kb`
7. 如果进入明确实现场景，联动 `.trae/skills/*/SKILL.md`

禁止只看用户一句需求就直接改代码。除非需求非常小，例如纯文案、单字段名称或一次性配置值修改。

## 2. 需求归类规则

编码前必须先判断需求属于哪类主线：

- 身份建立与资料准备
- 交易创建与履约启动
- 个人资产与服务管理
- 后台管理
- 资金审核
- 公共接口与小程序边界
- 跨场景通用问题

归类后再进入对应规则和 skill。

## 3. `.trae/kb` 默认联动规则

当需求属于以下类型时，必须先检查 `.trae/kb` 是否已有可复用经验：

- 接口边界分层
- 状态流转或状态摘要设计
- 资源占用与释放
- 正式流程和测试流程复用
- 后端兜底
- 预览与正式执行一致性
- 远程筛选、聚合回显、后台列表治理
- 多对象匹配、选择态与执行态分离

命中知识库后，应优先复用历史处理模式，再结合当前项目规则落地。

## 4. Skill 使用规则

当需求已经明确进入某类实现场景时，应主动查找并使用对应 skill：

- 后台业务列表：`skills/lanyan-admin-biz-list/SKILL.md`
- 后端 Controller：`skills/lanyan-backend-controller/SKILL.md`
- 后端 Entity/VO：`skills/lanyan-backend-entity/SKILL.md`
- 后端 Service：`skills/lanyan-backend-service/SKILL.md`
- 前端后台页面：`skills/lanyan-frontend-view/SKILL.md`
- 小程序请求与样式：`skills/lanyan-wx-request/SKILL.md`、`skills/lanyan-uniapp-style/SKILL.md`

规则文件负责判断“何时这样做”，skill 负责约束“具体怎么做”。

## 5. 编码前输出要求

复杂需求开始编码前，Trae 应先输出一版简短执行判断：

- 当前需求属于哪条业务主线
- 命中了哪些 rules
- 是否命中 `.trae/kb`
- 是否需要使用 skill
- 本次修改范围
- 验证方式

如果用户明确要求直接改，仍应在内部完成上述判断后再改。

## 6. 编码约束

默认遵守以下项目开发原则：

- 先后端边界，再前端调用
- 核心业务判断放服务层
- 前端不承担金额、状态、资源占用等核心兜底
- 小程序端不直接调用后台 `/system/**`
- 公共只读能力优先收口到 `/public/**`
- 用户态能力围绕当前登录用户设计
- 后台能力留在管理端接口
- 状态、金额、库存、权益、佣金等关键链路必须考虑回滚或释放

## 7. 任务结束后的经验蒸馏

复杂任务完成后，必须评估是否需要沉淀经验：

1. 一次性问题：不沉淀
2. 当前项目会重复出现：更新 `.trae/rules/project-memories.md`
3. 已稳定为项目规则：新增或更新 `.trae/rules/*-rule.md`
4. 属于实现模板：新增或更新 `.trae/skills/*/SKILL.md`
5. 可跨项目复用：新增 `.trae/kb/entries/KB-*.md`，并重建知识索引

如果本次经验值得沉淀，但用户没有明确要求立即入库，应在回复中提示建议沉淀位置。

## 8. 默认用户交互方式

当用户只说“整理一下”“优化一下”“修一下这个问题”时，不要直接泛化处理。应先按本规则完成：

1. 业务主线归类
2. 规则命中
3. 知识库检索判断
4. 修改范围确认
5. 实现与验证
6. 蒸馏评估

本规则与 `.trae/docs/TRAE-INTERNATIONAL-WORKFLOW.md` 配套使用。该文档是人工操作手册，本文件是 Trae 自动生效规则。
