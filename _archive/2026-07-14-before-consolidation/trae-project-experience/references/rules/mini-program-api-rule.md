# 小程序端接口与 Redis 操作规范

## 1. 小程序端接口调用规范
小程序端（非后台）禁止直接调用带有 `/system/` 前缀的后台管理接口。

### 详细要求
1. **禁止直接调用 `/system/`**：小程序端发送的任何请求都不应包含 `/system/` 路径前缀。
2. **使用公共接口前缀**：小程序端请求必须使用对应的小程序专属接口路由，当前项目约定公共接口前缀为 `/public/`。
3. **接口示例**：
   - 错误：`/system/busBanner/list`
   - 正确：`/public/banner/list`
4. **权限考量**：`/system/` 接口通常绑定了后台用户的权限校验，小程序端无法通过此类校验，因此必须通过无需 token 或基于小程序鉴权体系的 `/public/` 等前缀接口获取数据。
5. **获取登录用户信息**：通过 `SecurityUtils` 能够获取当前登录用户信息。

## 2. Redis 键统一前缀规范
在进行任何涉及到 Redis 操作（如设置缓存对象 `setCacheObject`、获取或监听过期 Key 等）的开发或修改时必须遵守以下规范：

### 详细要求
1. **统一前缀**：所有存入 Redis 的自定义业务键值对，都**必须**使用 `LanYanConfig.getDataBase()` 作为前缀。
2. **作用**：防止多个项目部署在同一个服务器并且共用同一个 Redis 实例时发生键冲突，确保环境间的数据相互隔离。
3. **格式示例**：
   - 错误：`"task_warning:" + taskId`
   - 正确：`LanYanConfig.getDataBase() + ":task_warning:" + taskId`
4. **监听器匹配**：在 Redis 过期监听器等场景下，判断过期 Key 的 `startsWith` 或解析切割时，必须将 `LanYanConfig.getDataBase()` 的前缀考虑在内。