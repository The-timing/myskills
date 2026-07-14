---
name: "lanyan-wx-request"
description: "LanYan小程序端网络请求与静态资源配置规范。当用户需要配置请求和静态资源文件时触发。"
---

# 小程序端网络请求与静态资源规范

本规范定义了 LanYan 项目中小程序端（uniapp）发起网络请求、环境配置以及全局静态资源访问的标准做法。

## 1. 环境变量与接口地址 (`env.js`)

项目使用 `utils/http/env.js` 统一管理环境配置。

- **切换环境**：通过修改 `CURRENT_ENV` 变量来切换 `development` 或 `production`。发布前请务必确保设为 `production`。
- **配置项**：
  - `baseUrl`：后端接口的基础路径。
  - `staticPrefix`：静态资源（如图片）的访问前缀。

```javascript
// utils/http/env.js
const CURRENT_ENV = 'production'; // 开发调试时修改为 'development'

const configs = {
  development: {
    baseUrl: 'http://localhost:3241',
    staticPrefix: 'https://api3241.alirongyun.com/image'
  },
  production: {
    baseUrl: 'https://api3241.alirongyun.com',
    staticPrefix: 'https://api3241.alirongyun.com/image'
  }
}
export default configs[CURRENT_ENV];
```

## 2. 发起网络请求 (`request.js`)

项目封装了 `utils/http/request.js`，并已挂载到全局 Vue 实例上。

- **基本用法**：
  在页面或组件中，通过 `this.$request`（或组合式 API 中的相应方式） 发起请求。
  ```javascript
  this.$request({
    url: '/public/banner/list',
    method: 'GET',
    params: { type: 1 }
  }).then(res => {
    // 成功处理（res 已经是后端返回的 data 层，包含了数据）
    console.log(res);
  }).catch(err => {
    // 失败处理
  });
  ```

- **请求拦截与鉴权**：
  - 默认会在 Header 中自动携带 `Authorization: Bearer <token>`。
  - 若某个接口不需要 token（如公开接口），可以在请求参数中添加 `headers: { isToken: false }`。

- **响应拦截**：
  - 自动处理业务状态码（`code`），未设置状态码则默认200。
  - `code === 401`：自动拦截并弹窗提示登录过期，确认后清除 token 并跳转到 `/pages/login/login`。
  - `code === 500 / 601` 或其他非 200 状态码：会自动使用 `uni.showToast` 弹出后端返回的错误信息 `msg`。
  - 成功时直接返回数据主体 `resolve(res.data)`。

## 3. 全局静态资源前缀 (`staticPrefix`)

为了方便在页面中统一引用远程静态资源（如 OSS/CDN 上的图片），项目在 `main.js` 中通过全局 `mixin` 注入了 `staticPrefix` 变量。

- **底层实现**：
  ```javascript
  // main.js
  import envConfig from './utils/http/env'
  
  // 全局混入 staticPrefix
  Vue.mixin({
    data() {
      return {
        staticPrefix: envConfig.staticPrefix || ''
      }
    }
  })
  ```

- **在页面中使用**：
  在 Vue 模板中，由于 `staticPrefix` 已全局混入，可以直接作为变量拼接路径使用，无需在每个组件里单独定义。
  ```html
  <template>
    <view class="container">
      <!-- 直接拼接全局混入的 staticPrefix -->
      <image :src="staticPrefix + '/home/banner.png'" mode="widthFix"></image>
      <view :style="{ backgroundImage: 'url(' + staticPrefix + '/bg.png)' }"></view>
    </view>
  </template>
  ```
  这样可以确保当环境切换（从开发切到生产）时，图片资源前缀能随 `env.js` 自动更新，避免硬编码。

## 4. 快速接入模板

当在新项目（或尚未包含这些配置的项目）中，你需要快速搭建此套网络请求体系时，请按照以下文件结构创建并填入模板代码。

### 4.1 创建工具类文件

**1. `utils/http/env.js` (环境变量配置)**
```javascript
// ==========================================
// 统一环境配置开关
// ==========================================
const CURRENT_ENV = 'development'; // 可选值：'development' 或 'production'

const configs = {
  development: {
    baseUrl: 'http://localhost:3241', 
    staticPrefix: 'http://localhost:3241/image' 
  },
  production: {
    baseUrl: 'https://api.yourdomain.com', 
    staticPrefix: 'https://api.yourdomain.com/image' 
  }
}

export default configs[CURRENT_ENV];
```

**2. `utils/http/auth.js` (Token 管理)**
```javascript
const TokenKey = 'User-Token'

export function getToken() {
  return uni.getStorageSync(TokenKey)
}

export function setToken(token) {
  return uni.setStorageSync(TokenKey, token)
}

export function removeToken() {
  return uni.removeStorageSync(TokenKey)
}
```

**3. `utils/http/errorCode.js` (错误码映射)**
```javascript
export default {
  '401': '认证失败，无法访问系统资源',
  '403': '当前操作没有权限',
  '404': '访问资源不存在',
  'default': '系统未知错误，请反馈给管理员'
}
```

**4. `utils/http/request.js` (核心请求封装)**
```javascript
import env from './env'
import errorCode from './errorCode'
import { getToken, removeToken } from './auth'

let isRelogin = { show: false };
const baseUrl = env.baseUrl;

function tansParams(params) {
  let result = ''
  for (const propName of Object.keys(params)) {
    const value = params[propName];
    var part = encodeURIComponent(propName) + "=";
    if (value !== null && value !== "" && typeof (value) !== "undefined") {
      if (typeof value === 'object') {
        for (const key of Object.keys(value)) {
          if (value[key] !== null && value[key] !== "" && typeof (value[key]) !== 'undefined') {
            let params = propName + '[' + key + ']';
            var subPart = encodeURIComponent(params) + "=";
            result += subPart + encodeURIComponent(value[key]) + "&";
          }
        }
      } else {
        result += part + encodeURIComponent(value) + "&";
      }
    }
  }
  return result
}

const request = (options) => {
  return new Promise((resolve, reject) => {
    const isToken = (options.headers || {}).isToken === false
    let header = {
      'Content-Type': 'application/json;charset=utf-8',
      ...options.headers
    }

    if (getToken() && !isToken) {
      header['Authorization'] = 'Bearer ' + getToken()
    }

    if ((options.method === 'get' || options.method === 'GET') && options.params) {
      let url = options.url + '?' + tansParams(options.params);
      url = url.slice(0, -1);
      options.params = {};
      options.url = url;
    }

    let data = options.data || {};
    if (options.method === 'post' || options.method === 'POST') {
      if (header['Content-Type'] === 'application/x-www-form-urlencoded' && typeof data === 'object') {
        data = tansParams(data);
        if (data.endsWith('&')) {
          data = data.slice(0, -1);
        }
      }
    }

    uni.request({
      url: baseUrl + options.url,
      method: options.method || 'GET',
      data: data,
      header: header,
      timeout: options.timeout || 10000,
      success: (res) => {
        const code = res.data.code || 200;
        const msg = errorCode[code] || res.data.msg || errorCode['default']

        if (options.responseType === 'arraybuffer') {
          resolve(res.data)
          return
        }
        if (code === 401) {
          if (!isRelogin.show) {
            isRelogin.show = true;
            uni.showModal({
              title: '系统提示',
              content: '登录状态已过期，是否前往登录',
              confirmText: '前往登录',
              cancelText: '取消',
              success: function (res) {
                isRelogin.show = false;
                if (res.confirm) {
                  removeToken()
                  uni.reLaunch({ url: '/pages/login/login' })
                }
              }
            });
          }
          reject('无效的会话，或者会话已过期，请重新登录。')
        } else if (code === 500) {
          uni.showToast({ title: msg, icon: 'none' })
          reject(new Error(msg))
        } else if (code !== 200) {
          uni.showToast({ title: msg, icon: 'none' })
          reject('error')
        } else {
          resolve(res.data)
        }
      },
      fail: (error) => {
        let { errMsg } = error;
        if (errMsg === 'request:fail') {
          errMsg = '后端接口连接异常';
        } else if (errMsg.includes('timeout')) {
          errMsg = '系统接口请求超时';
        }
        uni.showToast({ title: errMsg, icon: 'none', duration: 3000 })
        reject(error)
      }
    })
  })
}

export default request
```

### 4.2 全局注入 (在 `main.js` 中配置)

在项目的 `main.js` 中引入上述文件并进行全局挂载：

```javascript
import App from './App'
import request from './utils/http/request'
import envConfig from './utils/http/env'
import { setToken, getToken, removeToken } from './utils/http/auth'

// #ifndef VUE3
import Vue from 'vue'
Vue.config.productionTip = false
Vue.prototype.$request = request
Vue.prototype.$setToken = setToken
Vue.prototype.$getToken = getToken
Vue.prototype.$removeToken = removeToken

// 全局混入 staticPrefix
Vue.mixin({
  data() {
    return {
      staticPrefix: envConfig.staticPrefix || ''
    }
  }
})

App.mpType = 'app'
const app = new Vue({
  ...App
})
app.$mount()
// #endif

// #ifdef VUE3
import { createSSRApp } from 'vue'
export function createApp() {
  const app = createSSRApp(App)
  app.config.globalProperties.$request = request
  app.config.globalProperties.$setToken = setToken
  app.config.globalProperties.$getToken = getToken
  app.config.globalProperties.$removeToken = removeToken
  
  // Vue3 全局混入 staticPrefix
  app.mixin({
    data() {
      return {
        staticPrefix: envConfig.staticPrefix || ''
      }
    }
  })
  
  return {
    app
  }
}
// #endif
```