---
name: project-tech-stack
description: 当前项目的技术栈与实现边界。用于若依分离版 Java 后端、Redis、MySQL、MyBatis-Plus、Vue2 后台、Element UI、ECharts、UEditor、uni-app、小程序、H5、uView、Vant 或 WotUI 的开发与修改。
---

# 项目技术栈

## 后端

- 基础框架：若依分离版。
- 技术：Java、Redis、MySQL、MyBatis-Plus。
- 沿用 Controller → Service → Mapper 的直接链路；业务规则放在 Service，SQL 放在 Mapper/XML。
- 使用现有登录、权限、分页、字典、上传和异常返回能力，不重复造基础设施。
- MySQL 字段、索引和迁移与实体、Mapper、接口返回保持一致；Redis 只缓存可重建数据，不作为唯一业务事实来源。
- 优先最小改动；涉及并发、金额、权限、状态或数据一致性时，在服务和数据库的唯一来源解决。

### MyBatis-Plus 与 Mapper 规则

- 每个实体保留 `Mapper extends BaseMapper<Entity>`；MyBatis-Plus 的 Lambda 和 Service 仍以 Mapper 为数据访问基础。
- 单表 CRUD、简单条件查询和更新，优先使用 `lambdaQuery`、`lambdaUpdate`、`save`、`updateById`，不额外写 XML。
- 单表查询且 DTO/VO 字段基本与实体一致时，可用 MyBatis-Plus Lambda 查询实体后用 BeanUtils 或 MapStruct 转换；小数据量、非复杂分页优先该方式。
- 多表关联、统计聚合、`FOR UPDATE`、条件原子更新、复杂批量操作、性能敏感 SQL、大分页列表或数据库计算字段时，新增 Mapper 方法与 XML/注解 SQL，直接返回 DTO/VO。
- 同一个简单功能不要同时保留 Lambda 和重复的 Mapper XML；优先 Lambda，复杂查询再落 XML。
- 自定义 Mapper 方法只服务真实业务查询，不为单次简单条件创建冗余 SQL。

## 管理后台

- 技术：Vue 2、Element UI、ECharts、UEditor。
- 普通后台页面使用 Vue2 + Element UI，复用现有列表、表单、分页、权限指令和请求封装。
- 统计图表使用 ECharts；富文本内容使用 UEditor。
- 不引入 Vue3、Composition API、新的 UI 框架或平行请求层。

## 小程序与移动端

- 基础：uni-app，覆盖微信小程序和 H5。
- 组件库按现有页面选择其一：uView、Vant 或 WotUI；同一页面不要混用多个组件库。
- 小程序优先使用 uni-app 与 uView/WotUI；H5 页面优先沿用项目已接入的 Vant 或现有组件库。
- API 调用统一走现有请求封装、登录态和错误提示；不在页面复制权限、金额或状态判断。

## 实现原则

- 先定位现有相同功能，优先扩展，不复制实现。
- 不为单次需求新增抽象层、DTO、工具类或组件库。
- 新增后端字段时一次完成：SQL → 实体/Mapper → Service → 接口 → 页面。
- 新增接口时明确调用端、登录身份、返回字段和错误提示。
- 默认运行受影响模块的关键编译或测试；只在需要时执行全量构建。
