---
name: "lanyan-rich-text-editor"
description: "封装和改造 LanYan 项目的后台富文本编辑组件。用户提出 UEditor 替换、左右实时预览、手机框预览、上传适配或输入稳定性问题时调用。"
---

# LanYan Rich Text Editor

本技能用于在 LanYan 项目中生成、接入或重构后台富文本编辑能力，目标是统一复用现有 `UEditor + RichTextSplitEditor` 组合，而不是每个页面重复造一套编辑器和预览逻辑。

## 1. 何时调用

以下场景优先调用本技能：

- 用户要求把旧版 `<editor>` 替换成 `UEditor`
- 用户要求在后台编辑页增加“左侧编辑 + 右侧预览”
- 用户要求富文本预览支持手机框、电脑模式、固定尺寸或比例拖拽
- 用户反馈富文本上传异常、OSS 地址被拼成 `undefined + url`
- 用户反馈 UEditor 菜单竖排、内容不回显、快速输入后光标跳头

如果只是单纯修改文章文案、调整标题文案，不需要调用本技能。

## 2. 默认实现口径

当前项目的稳定组合如下：

- 编辑器封装：`lanyan-ui/src/components/UEditor/index.vue`
- 分栏预览组件：`lanyan-ui/src/components/RichTextSplitEditor/index.vue`
- 上传协议适配：`lanyan-admin/src/main/java/com/lanyan/web/controller/common/UeditorController.java`
- 通用上传入口：`lanyan-admin/src/main/java/com/lanyan/web/controller/common/CommonController.java`

默认原则：

1. 页面层优先接 `RichTextSplitEditor`
2. `RichTextSplitEditor` 内部统一接 `UEditor`
3. `UEditor` 只负责编辑器协议、内容同步、上传动作配置
4. 真实上传能力统一复用 `CommonController.uploadFile`

不要在业务页面里直接复制一份 UEditor 配置，更不要重新手写一套上传返回格式。

## 3. 页面接入方式

推荐用法：

```vue
<RichTextSplitEditor
  v-model="form.contentHtml"
  :min-height="420"
  preview-mode="mobile"
  :preview-mode-switchable="false"
/>
```

适用说明：

- 默认手机模式
- 默认不可切换
- 文章、协议、公告这类后台可信富文本场景优先用这个组合

如果页面明确要求开放切换，再传：

```vue
:preview-mode-switchable="true"
```

## 4. UEditor 组件必须遵守的稳定规则

### 4.1 初始化规则

- UEditor 必须在容器可见后初始化
- 若组件处于 `el-tabs`、弹窗、折叠区中，优先在对应面板真正显示后再挂载
- 否则 toolbar 宽度可能为 0，表现为菜单竖排

### 4.2 配置规则

前端本地配置必须显式包含：

```js
loadConfigFromServer: false,
imageActionName: 'uploadimage',
imageFieldName: 'upfile',
imageUrlPrefix: '',
videoActionName: 'uploadvideo',
videoFieldName: 'upfile',
videoUrlPrefix: '',
fileActionName: 'uploadfile',
fileFieldName: 'upfile',
fileUrlPrefix: ''
```

原因：

- 当前项目前端采用本地配置，不依赖后端配置回填
- 如果 `imageUrlPrefix/videoUrlPrefix/fileUrlPrefix` 不显式设为空串，OSS 全路径可能被拼成 `undefined + url`

### 4.3 内容同步规则

必须遵守以下同步口径：

- `vue-ueditor-wrap` 不要再使用 `@input` 反向改写 `innerValue`
- 只在外部 `value` 真变化时才通过 watcher 调 `setContent`
- 编辑器内部输入时，只向外 `$emit('input', nextValue)`，不再做同值 prop 回灌

原因：

- `vue-ueditor-wrap` 内部输入后如果又收到一份同值 `value`，会触发重复刷新
- 在快速输入、回车后继续输入时，这种回灌容易把光标打回段首

### 4.4 上传协议规则

UEditor 仍然保留 `/ueditor-config?action=xxx` 协议入口，但真实上传统一走：

```text
UeditorController -> CommonController.uploadFile
```

职责分层：

- `UeditorController`：负责把 UEditor 的 `state/url/title/original` 协议字段适配出来
- `CommonController.uploadFile`：负责真正的上传方式，可能是 OSS、本地或其它项目自定义方案

这样项目切换上传方式时，只需要改 `CommonController`，不用改页面和 UEditor 封装。

## 5. RichTextSplitEditor 组件必须遵守的稳定规则

### 5.1 布局规则

- 左侧是编辑器
- 右侧是预览区
- 默认手机模式
- 默认不可切换
- 手机模式统一使用 `src/assets/images/iphone_x.png` 外框

### 5.2 拖拽规则

拖拽分栏时必须：

- 加全屏透明遮罩层
- 阻止 `UEditor iframe` 抢鼠标事件
- 使用 `requestAnimationFrame` 节流更新比例

否则很容易出现：

- 鼠标滑过编辑器区域时拖拽断续
- 左右滑动发涩
- 拖动时页面频繁重排

### 5.3 预览规则

- 预览区使用固定高度，不随内容无限撑高
- 预览内部滚动，不让整页布局被撑开
- 手机预览和电脑预览都用固定常用宽度

当前项目默认值：

- 手机：`390px`
- 电脑：`960px`

## 6. 常见问题与标准处置

### 6.1 菜单竖排

优先检查：

1. 是否在不可见容器中提前初始化
2. 是否缺少 toolbar 横向样式兜底

### 6.2 内容不回显

优先检查：

1. 是否在编辑器 ready 前就尝试 `setContent`
2. 是否通过真实 UE 实例而不是错误时机去绑定实例

### 6.3 上传成功但页面里变成 `undefined + url`

优先检查：

1. `loadConfigFromServer` 是否为 `false`
2. `imageUrlPrefix/videoUrlPrefix/fileUrlPrefix` 是否显式设为空串

### 6.4 快速输入后光标跳头

优先检查：

1. 是否还保留 `vue-ueditor-wrap` 的 `@input` 同步逻辑
2. watcher 是否在同值场景下仍调用 `setContent`

### 6.5 图片、视频、文件上传要跟项目统一

标准做法不是把 UEditor 直接指向某个具体上传实现，而是：

1. 保留 `/ueditor-config` 协议入口
2. 在 `UeditorController` 内部转发到 `CommonController.uploadFile`
3. 把 `AjaxResult` 转成 UEditor 协议返回

## 7. 修改时的验证清单

每次调整富文本组件后，至少验证：

1. 页面打开时已有 HTML 能否正确回显
2. 快速输入、回车后继续输入时光标是否稳定
3. 图片上传后地址是否是完整可访问 URL
4. 预览区是否实时同步
5. 拖拽左右比例时是否平滑
6. 手机模式下内容是否落在 `iphone_x.png` 可视区内

## 8. 输出要求

调用本技能后，优先输出：

- 本次使用的是 `UEditor` 还是 `RichTextSplitEditor`
- 上传链路是否保持 `CommonController` 统一入口
- 是否涉及可见区初始化、预览模式、输入稳定性修复
- 本次验证了哪些风险点
