---
id: "KB-BACKEND-00000023"
title: "微信小程序发货信息录入应按订单键模式与物流模式最小化组装请求"
tech_tags:
  - "backend"
  - "java"
  - "wechat-mini-program"
problem_tags:
  - "DESIGN"
  - "RUN"
scenario_tags:
  - "DEV"
  - "VERIFY"
stage_tags:
  - "DEV"
keywords:
  - "upload_shipping_info"
  - "order_number_type"
  - "logistics_type"
  - "transaction_id"
error_message: "无固定报错；常见现象是 access_token 已获取成功，但 `wxa/sec/order/upload_shipping_info` 返回参数错误、订单键不匹配、`10060001 支付单不存在`，或参考代码把延迟调度塞进工具方法导致调用边界混乱。"
trigger_conditions:
  - "项目需要调用微信小程序发货信息录入接口 `wxa/sec/order/upload_shipping_info`"
  - "团队准备直接复用网络上的参考代码或旧工具方法"
  - "调用方同时关心支付回调、延迟上报、立即发货等场景"
impact_scope:
  - "微信发货信息录入"
  - "支付成功后的履约补充动作"
  - "微信小程序订单履约状态同步"
status: "active"
owners:
  - "shared-kb-maintainer"
created_at: "2026-07-03T19:05:00+08:00"
updated_at: "2026-07-03T19:05:00+08:00"
related_entries:
  - "KB-FLOW-00000013 支付回调中的外部补充动作必须与主链路隔离并异步延迟执行"
  - "KB-BACKEND-00000019 微信支付协议升级前先按证书材料与配置托管方式完成选型"
---

# 微信小程序发货信息录入应按订单键模式与物流模式最小化组装请求

## 1. 问题描述

- 微信小程序发货信息录入接口看起来只是一个普通 POST，但网络上常见参考代码往往把“参数组装”“延迟调度”“异常处理”“控制台打印”混在一个方法里。
- 这会导致几类高频问题：
  - `order_number_type=1` 时仍错误混传 `transaction_id`
  - `logistics_type=3` 仍硬塞 `tracking_no`、`express_company`、`contact`
  - 日志打印显示按 `transaction_id` 上报，但实际请求体仍固定发 `order_number_type=1`
  - 已经改成虚拟商品发货后仍返回 `10060001 支付单不存在`
  - 工具方法内部自己 `Timer` 延迟，导致调用边界和职责混乱
  - accessToken 为空或微信返回空响应时，没有明确兜底错误
- 这类问题的本质不是“接口地址错”，而是没有先把“订单键选型”“物流模式字段口径”“调用方职责边界”拆清楚。

## 2. 触发条件与影响范围

- 触发条件：
  - 业务需要调用 `https://api.weixin.qq.com/wxa/sec/order/upload_shipping_info`
  - 参考实现来源于旧工具类、博客示例或临时测试代码
  - 调用场景同时涉及支付成功回调、立即发货或延迟上报
- 影响范围：
  - 微信小程序订单发货信息录入成功率
  - 支付成功后履约补充动作的稳定性
  - 调用方对“同步上传”和“异步延迟”的职责划分

## 3. 复现步骤

1. 按网络常见示例组装请求，设置 `order_number_type=1`，同时又写入 `transaction_id`。
2. 对 `logistics_type=3` 的立即发货请求继续传空字符串 `tracking_no`、`express_company`、`contact` 等字段。
3. 在工具方法内部直接 `Timer.schedule(...)` 延迟执行，并把微信异常直接作为工具层异常抛出。
4. 日志打印声称走 `order_number_type=2 + transaction_id`，但实际 payload 仍发 `order_number_type=1 + out_trade_no`。
5. 观察到代码虽然“像是能跑”，但会出现参数语义混乱、排查困难，甚至把调用时序问题误当成接口本身问题。

## 4. 根因分析

- 根因结论：
  - 参考代码没有按微信接口的“订单键模式”和“物流模式”做最小化请求建模，也没有把工具层和调用层职责拆开。
- 直接原因：
  - `order_key` 二选一语义没有收敛，导致 `mchid + out_trade_no` 与 `transaction_id` 混传。
  - `logistics_type=3` 时仍保留快递物流字段，制造无意义噪音。
  - 延迟调度写在工具方法内部，破坏了调用层对回调时序和异常隔离的控制。
  - 业务日志与实际请求体不一致，导致排查方向被错误日志带偏。
- 深层原因：
  - 参考代码偏“演示能跑”，没有区分正式流程里哪些属于请求组装，哪些属于支付回调后的副作用编排。
  - 没有把微信接口需要的最小字段集与业务链路的异步策略分层设计。
  - 没有结合官方 `10060001 支付单不存在` 说明，对“订单键错误”和“支付单尚未进入微信发货管理可识别范围”做分层判断。
- 为什么此前没有暴露：
  - 本地联调通常只验证接口能否返回，不会系统检查“字段是否语义正确”“延迟职责是否放错层”“异常是否污染上游主链路”。

## 5. 分步处置方案

1. 先按订单键模式和物流模式收敛最小请求体

```java
OrderKeyDTO orderKey = new OrderKeyDTO();
orderKey.setOrder_number_type(2);
orderKey.setTransaction_id(transactionId);

ShippingDTO shipping = new ShippingDTO();
shipping.setItem_desc(content);

PayerDTO payer = new PayerDTO();
payer.setOpenid(openId);

WxDeliverGoodsDTO request = new WxDeliverGoodsDTO();
request.setOrder_key(orderKey);
request.setLogistics_type(3);
request.setDelivery_mode(1);
request.setShipping_list(Collections.singletonList(shipping));
request.setUpload_time(new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSSZ").format(new Date()));
request.setPayer(payer);
```

注意事项：

- 虚拟商品发货如明确按微信支付单号上报，应固定 `order_number_type=2`，只传真实 `transaction_id`，不要再混传 `mchid`、`out_trade_no`。
- `logistics_type=3` 表示无实体物流的虚拟商品场景，此时只保留必要的 `item_desc`，不要硬塞空字符串物流字段。

2. 工具方法只负责同步上传，延迟和异常隔离放在调用方

```java
String accessToken = getMPToken(appId, secret);
if (accessToken == null || accessToken.trim().isEmpty()) {
    throw new ServiceException("获取微信 accessToken 失败，无法录入发货信息");
}

String url = "https://api.weixin.qq.com/wxa/sec/order/upload_shipping_info?access_token=" + accessToken;
String requestBody = JSONObject.toJSONString(request);
log.info("微信小程序虚拟商品发货请求，orderNumber={}, transactionId={}, requestBody={}",
        orderNumber, transactionId, requestBody);
String responseBody = HttpUtil.post(url, requestBody);
log.info("微信小程序虚拟商品发货响应，orderNumber={}, transactionId={}, responseBody={}",
        orderNumber, transactionId, responseBody);
JSONObject response = JSON.parseObject(responseBody);
Integer errCode = response == null ? null : response.getInteger("errcode");
if (errCode == null || errCode != 0) {
    String errMsg = response == null ? "微信返回空响应" : response.getString("errmsg");
    throw new ServiceException("微信小程序-发货信息录入接口异常：" + errMsg);
}
```

注意事项：

- 工具层不要内置 `Timer`、`sleep` 或线程调度；是否延迟执行，由支付回调或服务层调用方决定。
- 如果当前调用发生在支付回调里，异步延迟和失败隔离要参考 `KB-FLOW-00000013`，不要让补充动作反向污染支付主链路。
- 若接口返回 `10060001 支付单不存在`，且请求体已经严格符合单一键模式，优先排查：支付刚完成时微信侧同步存在延迟、该交易是否真正进入微信购物订单/发货管理体系、当前交易是否由小程序绑定商户号完成支付，而不是继续在快递字段或空字符串字段上反复试错。

## 6. 验证与验收标准

- 功能验证点：
  - 能成功获取 accessToken 并正确调用 `upload_shipping_info`
  - 请求体中 `order_key` 与 `logistics_type` 对应字段语义一致
  - 微信返回 `errcode=0` 时，调用方能明确记录成功
- 回归验证点：
  - accessToken 为空时，工具方法直接报清晰错误
  - 微信返回空响应或非 0 `errcode` 时，调用方能拿到明确异常信息
  - 延迟执行策略改动不会影响工具方法本身的同步上传职责
  - 业务日志中的 `order_number_type` 与实际请求 JSON 完全一致，不再出现“日志写 2 实际发 1”
- 可执行测试用例：
  - 使用真实 `openid`、`transactionId` 测试 `order_number_type=2` + `logistics_type=3` 请求，确认最小虚拟商品请求体可以被稳定发出
  - 如返回 `10060001`，先比对微信后台是否能识别该 `transactionId`，并在支付完成后拉开足够时间再重试
  - 在支付回调链路中把上传动作改为异步延迟执行，验证支付主链路与发货补充动作彼此隔离
- 通过判定条件：
  - 参考代码不再混淆字段口径，工具层只负责同步上传，请求组装最小且语义清晰，调用层可以独立控制延迟和异常隔离

## 7. 更新记录

- `2026-07-03` `shared-kb-maintainer`：首次入库，沉淀微信小程序发货信息录入在订单键选型、物流模式字段裁剪与调用边界上的通用做法。
- `2026-07-06` `shared-kb-maintainer`：补充虚拟商品发货强制按 `transaction_id` 单一口径组装、必须打印完整请求/响应 JSON，以及 `10060001 支付单不存在` 的分层排查顺序。

## 8. 关联条目

- `KB-FLOW-00000013 支付回调中的外部补充动作必须与主链路隔离并异步延迟执行`
- `KB-BACKEND-00000019 微信支付协议升级前先按证书材料与配置托管方式完成选型`
