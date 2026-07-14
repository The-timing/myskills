---
id: "KB-BACKEND-00000021"
title: "微信支付 V2 退款在 p12 证书场景优先使用 orderRefundByProtocol"
tech_tags:
  - "backend"
  - "java"
  - "wechat-mini-program"
problem_tags:
  - "RUN"
  - "CONFIG"
scenario_tags:
  - "VERIFY"
  - "CONFIG"
  - "DEV"
stage_tags:
  - "DEV"
keywords:
  - "微信退款"
  - "orderRefundByProtocol"
  - "apiclient_cert.p12"
  - "SSLHandshakeException"
error_message: "调用 IJPay 的 WxPayApi.orderRefund 时抛出 SSLHandshakeException: No appropriate protocol (protocol is disabled or cipher suites are inappropriate)。"
trigger_conditions:
  - "项目仍使用微信支付 V2 退款链路"
  - "现场提供的是 apiclient_cert.p12 证书材料"
  - "使用 IJPay 的 orderRefund 直接按 certPath 发起退款时报 SSL 握手异常"
impact_scope:
  - "售后退款"
  - "微信支付 V2 退款"
  - "退款证书配置"
status: "active"
owners:
  - "shared-kb-maintainer"
created_at: "2026-07-03T18:25:00+08:00"
updated_at: "2026-07-03T18:25:00+08:00"
related_entries:
  - "KB-BACKEND-00000019 微信支付协议升级前先按证书材料与配置托管方式完成选型"
  - "KB-BACKEND-00000020 工程静态配置、第三方接入配置与业务运行时配置要按职责分层存放"
---

# 微信支付 V2 退款在 p12 证书场景优先使用 orderRefundByProtocol

## 1. 问题描述

- 在 Spring Boot + IJPay 的微信 V2 退款链路里，现场经常只有 `apiclient_cert.p12` 和商户号。
- 这类场景下如果直接调用 `WxPayApi.orderRefund(false, params, certPath, mchId)`，部分线上 JDK 环境会在 SSL 双向证书握手阶段直接失败。
- 常见现象是：
  - 退款参数已经组装完成
  - 证书路径存在且可读取
  - 但请求还没拿到微信业务响应，就先抛出 `SSLHandshakeException: No appropriate protocol`
- 这类问题的关键不是退款业务字段错，而是 `orderRefund` 这条按证书路径发起的底层 SSL 行为在部分环境里不稳定。

## 2. 触发条件与影响范围

- 触发条件：
  - 项目仍保留微信支付 V2。
  - 退款证书材料是 `apiclient_cert.p12`。
  - 使用 IJPay 且退款调用走 `orderRefund(...)`。
- 影响范围：
  - 售后退款审核
  - 主动退款接口
  - 任何复用 V2 退款证书的业务链路

## 3. 复现步骤

1. 准备微信支付 V2 退款参数，包含 `appid`、`mch_id`、`out_trade_no / transaction_id`、`out_refund_no`、`total_fee`、`refund_fee`。
2. 调用 `WxPayApi.orderRefund(false, params, certPath, mchId)`。
3. 在线上部分 JDK 环境中观察到以下异常：

```text
SSLHandshakeException: No appropriate protocol (protocol is disabled or cipher suites are inappropriate)
```

4. 即使提前设置 `https.protocols=TLSv1.2`，也不一定能稳定解决。

## 4. 根因分析

- 根因结论：
  - 在 `p12` 证书场景下，`orderRefund(...)` 这条“按路径直接发证书请求”的调用方式对部分运行环境的 SSL 协议协商不稳定。
- 直接原因：
  - 退款请求尚未进入微信业务层，就在双向证书 HTTPS 握手阶段失败。
- 深层原因：
  - 代码默认把“证书路径可用”误当成“底层 SSL 行为稳定”。
  - 没有优先复用 IJPay 已提供的 `orderRefundByProtocol(...)` 这种“显式传证书流”的实现路径。
- 为什么此前没有暴露：
  - 本地开发机和线上 JDK / OpenSSL / 安全策略不同，本地能过的握手在服务器上未必能过。

## 5. 分步处置方案

1. 保持 V2 退款参数与签名口径不变

```text
推荐保留：
- RefundModel.builder()/new RefundModel() 组装退款参数
- signType 继续与现有 V2 链路保持一致
- 继续读取 sys_config 中的 appId、mchId、machSecret、certPath
```

注意事项：

- 这类问题先不要一上来怀疑退款金额或售后业务流程。
- 先确认问题是不是发生在 SSL 握手前。

2. 将退款发送方式切换为 `orderRefundByProtocol(...)`

```java
try (InputStream certInputStream = FileUtil.getInputStream(certPath)) {
    String xmlResult = WxPayApi.orderRefundByProtocol(
        false,
        params,
        certInputStream,
        mchId,
        ""
    );
}
```

注意事项：

- `certPath` 仍然要先做存在性与可读性校验。
- 推荐使用 `try-with-resources` 自动关闭证书流。
- 不要同时保留两套退款发送实现，避免排查时口径混乱。

3. 日志要记录足够的退款上下文

```text
至少记录：
- orderNo
- outRefundNo
- certPath
- mchId
- 当前签名类型
```

注意事项：

- 这里记录的是排查握手问题所需的配置上下文，不是让日志泄露商户密钥。
- 不要把 `machSecret`、证书原文等敏感信息直接输出到日志。

4. 保持配置分层清晰

```text
推荐：
- 工程静态行为继续放 application.yml
- 微信商户接入参数继续放 sys_config
- 业务侧售后规则不要和支付证书配置混在 bus_setting
```

注意事项：

- 退款修复时不要顺手把配置来源改乱。
- 本次问题是“退款发送方式”问题，不是“配置存放层级”问题。

## 6. 验证与验收标准

- 功能验证点：
  - 售后审核触发退款后，不再出现 `No appropriate protocol`。
  - 微信退款接口能返回 `return_code/result_code` 业务结果。
- 回归验证点：
  - 下单、回调等其它 V2 支付链路不受影响。
  - 退款证书路径校验异常时能明确报出“文件不存在”或“不可读取”。
- 可执行测试用例：
  - 使用真实 `p12` 证书发起一笔小额退款，验证能拿到微信 XML 返回。
  - 使用错误证书路径验证系统明确报配置错误，而不是笼统报“发起微信退款失败”。
- 通过判定条件：
  - 退款请求能够稳定进入微信业务响应阶段，不再卡死在底层 SSL 握手。

## 7. 更新记录

- `2026-07-03` `shared-kb-maintainer`：首次入库，沉淀微信支付 V2 在 `p12` 证书场景下优先使用 `orderRefundByProtocol(...)` 的处理方式。

## 8. 关联条目

- `KB-BACKEND-00000019 微信支付协议升级前先按证书材料与配置托管方式完成选型`
- `KB-BACKEND-00000020 工程静态配置、第三方接入配置与业务运行时配置要按职责分层存放`
