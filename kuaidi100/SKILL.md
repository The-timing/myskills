---
name: kuaidi100
description: "快递100配置流程。当用户需要配置快递100时触发。"
---

# 快递100接入技能

本技能用于在 LanYan 项目中接入快递100物流查询能力。

## 何时使用
当用户提出以下需求时触发：
- 配置快递100
- 对接物流轨迹查询
- 增加订单物流详情接口

## 不适用场景
- 小程序普通网络请求封装
- 静态资源配置
- 与物流轨迹无关的第三方接入

## 1. 引入maven依赖
在 `lanyan-common/pom.xml` 中引入快递100依赖：
```xml
        <dependency>
            <groupId>com.github.kuaidi100-api</groupId>
            <artifactId>sdk</artifactId>
            <version>1.0.11</version>
        </dependency>
```
## 2. 在用户指定的位置追加查询接口
在用户指定的位置追加快递100物流查询接口，例如订单物流详情页。

实现时建议：
- 先校验订单是否存在物流单号
- 快递100参数不要硬编码到控制器里
- 优先从系统配置读取 `customer`、`key`、`secret`、`userid`
- 接口返回值优先直接返回解析后的轨迹数据

```java
/**
     * 查询物流
     */
    @GetMapping("/queryTrackReq")
    @ApiOperation("查询物流")
    public AjaxResult queryTrackReq(Long ordersId) throws Exception {
        Orders orders = ordersService.getById(ordersId);
        if (StringUtils.isEmpty(orders.getExpressNo())){
            return AjaxResult.error("订单未发货");
        }
        String customer = "xxxx";
        String key = "asdf";
        String secret = "xxxxx";
        String userid = "xxxxxxx";
        String zhiNengPanDuan = "asdf";
        QueryTrackReq queryTrackReq = new QueryTrackReq();
        QueryTrackParam queryTrackParam = new QueryTrackParam();
        queryTrackParam.setNum(orders.getExpressNo());
        queryTrackParam.setPhone(orders.getUserPhone());
        String param = new Gson().toJson(queryTrackParam);

        queryTrackReq.setParam(param);
        queryTrackReq.setCustomer(customer);
        queryTrackReq.setSign(SignUtils.querySign(param ,key,customer));

        IBaseClient baseClient = new QueryTrack();
        HttpResult execute = baseClient.execute(queryTrackReq);
        System.out.println(execute);
        return AjaxResult.success(JSONObject.parse(execute.getBody()));
    }
```

## 3. 推荐补充要求
- 配置项优先放到 `bus_setting` 或系统配置中
- 控制器只负责参数校验与返回，不要堆第三方配置
- 如有多处会复用物流查询，优先沉淀到 service
