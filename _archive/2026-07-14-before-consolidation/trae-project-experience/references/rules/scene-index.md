---
alwaysApply: false
description: 项目业务场景规则索引
---
# 项目业务场景规则索引

本文档用于把 README 中的业务主线、`.trae/rules` 中的规则文件以及 `.trae/skills` 中的技能建立统一映射。

## 使用原则

- 本文档优先使用通用场景名称，不直接把当前项目的特定业务词当作主分类。
- 当前项目的具体业务只应作为“示例”，不能作为未来其它项目复用本索引时的依赖前提。
- 如果某一类需求只能通过当前项目专有词汇才能成立，说明抽象层级还不够，需要继续上提。

## 1. 身份建立与资料准备主线

典型需求：

- 登录、注册、授权
- 基础资料补全
- 业务归属信息选择
- 偏好、规格或基础档案选择

当前项目示例：

- 资料补全
- 学校选择
- 鞋码选择

优先规则：

- `project-logic-readme.md`
- `mini-program-api-rule.md`

可联动技能：

- `skills/lanyan-uniapp-style/SKILL.md`
- `skills/lanyan-wx-request/SKILL.md`

## 2. 交易创建与履约启动主线

典型需求：

- 预览
- 正式创建
- 地址、时间、优惠、备注等创建参数处理
- 会员卡、余额、券、权益等资源抵扣
- 支付
- 0 成本或免支付场景直通
- 支付成功后的回调与履约启动

当前项目示例：

- 订单预览
- 正式下单
- 洗衣卡抵扣
- 微信支付
- 微信发货回调

优先规则：

- `order-payment-consistency-rule.md`
- `status-enum-consistency-rule.md`
- `mini-program-api-rule.md`

按需追加：

- 涉及库存占用、取消释放：
  - `product-stock-rule.md`
- 涉及公共订单查询能力：
  - `public-controller-reuse-rule.md`

最近沉淀处理方式：

- 订单预览与正式下单必须共用一套服务层构建逻辑
- 0 元订单不能依赖前端继续发起支付，创建订单时要后端兜底
- 微信支付正式回调应复用正式支付成功逻辑，不要写独立假流程
- 洗衣卡按商品逐个匹配，可同时命中多张卡，同商品优先最早过期卡

## 3. 个人资产与服务管理主线

典型需求：

- 用户资产查询
- 服务能力查询
- 权益详情
- 记录列表
- 服务申请与进度跟踪

当前项目示例：

- 我的订单
- 洗衣卡
- 会员
- 优惠券
- 消息
- 售后
- 合伙人

优先规则：

- `status-enum-consistency-rule.md`
- `mini-program-api-rule.md`

按需追加：

- 涉及售后关联主订单摘要状态：
  - `status-enum-consistency-rule.md`
- 涉及公共查询能力：
  - `public-controller-reuse-rule.md`

最近沉淀处理方式：

- 洗衣卡详情默认补商品价格、图片、到期时间
- 主订单售后状态与售后单详细状态要分层，不能混用一套状态机
- 撤销、审核、已打款等售后节点应形成完整闭环

## 4. 后台管理主线

典型需求：

- 列表页整理
- 状态主筛选
- 关联对象筛选
- 聚合列与摘要展示
- 详情弹窗或抽屉
- 操作区固定
- 主从结构整理

优先规则：

- `admin-list-scene-rule.md`
- `admin-ui-render-rule.md`
- `project-memories.md`

可联动技能：

- `skills/lanyan-admin-biz-list/SKILL.md`
- `skills/lanyan-frontend-view/SKILL.md`

最近沉淀处理方式：

- 订单、售后、提现、佣金、洗衣卡订单默认补远程用户筛选
- 默认补一批远程用户选项，下拉展开和清空搜索词都要恢复默认数据
- 用户 ID、学校 ID、商品 ID 等外键优先后端回显业务信息
- 列表强主状态优先改 tabs，详情优先 dialog/drawer 分块展示

## 5. 资金审核主线

典型需求：

- 收益结算
- 提现申请
- 审核流转
- 金额回退
- 打款完成

优先规则：

- `finance-audit-flow-rule.md`
- `status-enum-consistency-rule.md`

可联动技能：

- `skills/lanyan-admin-biz-list/SKILL.md`

最近沉淀处理方式：

- 审核流程至少覆盖待审核、处理中、已完成、已拒绝
- 驳回时回退金额，完成时累计已提现金额和流水

## 6. 公共接口与小程序边界场景

典型需求：

- 客户端接口
- 公共查询接口
- 匿名读取能力
- 用户态、公共态、后台态接口职责划分

优先规则：

- `mini-program-api-rule.md`
- `public-controller-reuse-rule.md`
- `project-logic-readme.md`

最近沉淀处理方式：

- 小程序端禁止直接走 `/system/**`
- 可匿名或登录可读的查询能力优先收口到 `/public/**` 或用户态 `/api/**`
- 能复用已有 domain/vo 时优先复用，不新增一层无意义对象

## 7. 跨场景通用规则

以下规则不只属于单一主线，遇到对应问题需主动联动：

- `scene-handling-framework.md`
  - 通用场景抽象、识别、定位、方案选型、落地验证的上层方法论
- `status-enum-consistency-rule.md`
  - 状态枚举、状态机、主订单摘要状态、前后端状态口径统一
- `project-memories.md`
  - 最近任务的高频处理方式和落地经验
- `project-logic-readme.md`
  - 所有任务的入口规则
