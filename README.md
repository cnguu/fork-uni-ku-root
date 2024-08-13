# Root

![Uniapp Vue3](https://img.shields.io/badge/Uniapp_Vue3-4FC08D?logo=vue.js&labelColor=18181B)
![Vite Plugin](https://img.shields.io/badge/Vite_Plugin-646CFF?logo=vite&labelColor=18181B)

借助 Vite 模拟出虚拟根组件(支持SFC的App.vue)，解决 uniapp 无法使用公共组件问题

[![NPM version](https://img.shields.io/npm/v/@uni-ku/root?color=92DCD2&labelColor=18181B&label=npm)](https://www.npmjs.com/package/@uni-ku/root)
[![NPM downloads](https://img.shields.io/npm/dw/@uni-ku/root?color=92DCD2&labelColor=18181B&label=downloads)](https://www.npmjs.com/package/@uni-ku/root)
[![LICENSE](https://img.shields.io/github/license/uni-ku/root?style=flat&color=92DCD2&labelColor=18181B&label=license)](https://www.npmjs.com/package/@uni-ku/root)

> [!TIP]
> 从 v0.2.0 开始, 已支持 HBuilderX 创建的 Vue3 项目

### 🎏 支持

- Uniapp
- Vue3
- CLI 或 HBuilderX 创建的项目

### 📦 安装

```bash
pnpm add -D @uni-ku/root
```

### 🚀 使用

1. 引入 `@uni-ku/root`

- CLI: `直接编写` 根目录下的 vite.config.*
- HBuilderX: 需要根据你所使用语言, 在根目录下 `创建`  vite.config.*

```js
// vite.config.*

import { defineConfig } from 'vite'
import UniKuRoot from '@uni-ku/root'
import Uni from '@dcloudio/vite-plugin-uni'

export default defineConfig({
  plugins: [
    // 若存在改变 pages.json 的插件，请将 UniKuRoot 放置其后
    UniKuRoot(),
    Uni()
  ]
})
```
2. 创建 `App.ku.vue`

通过标签 `<KuRootView />` 或 `<ku-root-view />` 指定视图存放位置，且可以放置到 `template` 内任意位置，但仅可有一个

> 该功能与 VueRouter 中的 RouterView 功能类似

- CLI: 需要在 `src目录` 下创建下 App.ku.vue
- HBuilderX: 直接在 `根目录` 下创建 App.ku.vue

```vue
<!-- src/App.ku.vue | App.ku.vue -->

<script setup lang="ts">
import { ref } from 'vue'

const helloKuRoot = ref('Hello AppKuVue')
</script>

<template>
  <div>{{ helloKuRoot }}</div>
  <!-- 顶级 KuRootView -->
  <KuRootView />

  <!-- 或内部 KuRootView，无论放置哪一个层级都被允许，但仅可有一个！ -->
  <div>
    <KuRootView />
  </div>
</template>
```

### ✨ 例子

> 以下例子均以CLI创建项目为例, HBuilderX 项目与以上设置同理, 只要注意是否需要包含 src目录 即可

<details>

<summary>
  <strong>1. (点击展开) 全局共享组件例子：Toast</strong>
</summary>
<br />

> 不仅是 Toast 组件，还可以是 Message、LoginPopup 等等

- 🔗 [查看以下完整项目例子](https://github.com/uni-ku/root/tree/main/examples)

1. 编写 Toast 组件

```vue
<!-- src/components/GlobalToast.vue -->

<script setup lang="ts">
import { useToast } from '@/composables/useToast'

const { globalToastState, hideToast } = useToast()
</script>

<template>
  <div v-if="globalToastState" class="toast-wrapper" @click="hideToast">
    <div class="toast-box">
      welcome to use @uni-ku/root
    </div>
  </div>
</template>

<style scoped>
.toast-wrapper{
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.toast-box{
  background: white;
  color: black;
}
</style>
```

2. 实现 Toast 组合式API

```ts
// src/composables/useToast

import { ref } from 'vue'

const globalToastState = ref(false)

export function useToast() {
  function showToast() {
    globalToastState.value = true
  }

  function hideToast() {
    globalToastState.value = false
  }

  return {
    globalToastState,
    showToast,
    hideToast,
  }
}
```

3. 挂载至 App.ku.vue

```vue
<!-- src/App.ku.vue -->

<script setup lang="ts">
import GlobalToast from '@/components/GlobalToast.vue'
</script>

<template>
  <KuRootView />
  <GlobalToast />
</template>
```

4. 视图内部触发全局 Toast 组件

```vue
<!-- src/pages/*.vue -->

<script setup lang="ts">
import { useToast } from '@/composables/useToast'

const { showToast } = useToast()
</script>

<template>
  <view>
    Hello UniKuRoot
  </view>
  <button @click="showToast">
    视图内触发展示Toast
  </button>
</template>
```

</details>

<details>

<summary>
  <strong>2. (点击展开) 全局共享布局例子：ConfigProvider</strong>
</summary>
<br />

> 不仅仅只有ConfigProvider，还能是Layout、NavBar、TabBar等等！

1. 以 Wot 组件库中 WdConfigProvider 为例子，[了解更多Wot点这里](https://github.com/Moonofweisheng/wot-design-uni)

```vue
<!-- src/App.ku.vue -->

<script setup lang="ts">
import { useTheme } from './composables/useTheme'

const { theme, themeVars } = useTheme({
  buttonPrimaryBgColor: '#07c160',
  buttonPrimaryColor: '#07c160'
})
</script>

<template>
  <div>Hello AppKuVue</div>
  <!-- 假设已注册 WdConfigProvider 组件 -->
  <WdConfigProvider :theme="theme" :theme-vars="themeVars">
    <KuRootView />
  </WdConfigProvider>
</template>
```

2. 编写主题相关组合式API

```ts
// src/composables/useTheme.ts

import type { ConfigProviderThemeVars } from 'wot-design-uni'
import { ref } from 'vue'

const theme = ref<'light' | 'dark'>(false)
const themeVars = ref<ConfigProviderThemeVars>()

export function useTheme(vars?: ConfigProviderThemeVars) {
  vars && (themeVars.value = vars)

  function toggleTheme(mode?: 'light' | 'dark') {
    theme.value = mode || (theme.value === 'light' ? 'dark' : 'light')
  }

  return {
    theme,
    themeVars,
    toggleTheme,
  }
}
```

3. 切换主题模式

```vue
<!-- src/pages/*.vue -->

<script setup lang="ts">
import { useTheme } from '@/composables/useTheme'

const { theme, toggleTheme } = useTheme()
</script>

<template>
  <button @click="toggleTheme">
    切换主题，当前模式：{{ theme }}
  </button>
</template>
```

</details>

### 📝 待办

- [x] 支持热更新
- [x] 支持VueSFC
- [x] 支持小程序PageMeta
- [ ] 支持 App.ku.vue 内直接编写控制逻辑
- [ ] 补全单元测试

### 💬 社区

- QQ 交流群 ([897784703](https://qm.qq.com/q/hX1smd93MI))

### 🏝 周边

|项目|描述|
|---|---|
|[Wot Design Uni](https://github.com/Moonofweisheng/wot-design-uni/)|一个基于Vue3+TS开发的uni-app组件库，提供70+高质量组件|

### 💖 赞赏

如果我的工作帮助到了您，可以请我吃辣条，使我能量满满 ⚡

> 请留下您的Github用户名，感谢 ❤

#### 微信赞赏

<img src="https://cdn.jsdelivr.net/gh/Skiyee/sponsors@main/assets/wechat-pay.png" alt="wechat-pay" width="320" />

#### 赞赏榜单

<p align="center">
  <a href="https://github.com/Skiyee/sponsors">
    <img alt="sponsors" src="https://cdn.jsdelivr.net/gh/Skiyee/Skiyee@main/sponsors.svg"/>
  </a>
</p>
