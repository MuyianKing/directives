# vTop（v-top）

> 包：`@mu/directives` ｜ 监听元素自身滚动，向上滚动且接近顶部（`scrollTop < 10`）时触发回调。典型场景：长列表「回到顶部提示 / 触发顶部加载」。

## 1. 用途与场景

- 与 [`v-bottom`](./bottom.md) 成对：本指令管顶部。
- 只有「**向上滚 + 已经接近顶部**」两个条件同时满足才触发，向下滚或远离顶部都不触发。

## 2. 引入与注册

```js
import { vTop } from '@mu/directives' // 本包不在自动导入范围，必须显式 import
```

- `<script setup>`：导入变量名必须是 `vTop`，模板即可写 `v-top`。
- 全局：`app.directive('top', vTop)`。
- 源码：`src/top.js`。

## 3. 绑定值

| 项 | 说明 |
| --- | --- |
| 类型 | `Function`，**无参数** |
| 语义 | 满足「向上滚 + 接近顶部」时调用 |
| 非函数时 | `mounted` 中直接跳过，不绑定监听，也不报错 |

## 4. 行为契约（源码可验证）

1. `mounted(el, binding)` 时若 `binding.value` 不是函数，**什么都不做**。
2. 闭包内维护 `last_top`（初始 `0`），回调用 `useDebounceFn(..., 300)` 包装（**300ms 防抖**）。
3. 每次 `scroll` 计算 `top = this.scrollTop < 10`；仅当 `top === true` **且** `this.scrollTop - last_top < 0`（本次是向上滚）时触发回调。
4. 每次 scroll 结束后把 `last_top = this.scrollTop`。
5. 监听的是**元素自身的 `scroll` 事件**。

推论（源码逻辑）：元素从远处向上滚到 `scrollTop < 10` 时会触发；一直停留在顶部附近反复小幅滚动会反复触发（受 300ms 防抖约束）。

## 5. 示例

```vue
<script setup>
import { vTop } from '@mu/directives'

function handleTop() {
  // 回到顶部后要做的处理
}
</script>

<template>
  <div class="h-[300px] overflow-y-auto" v-top="handleTop">
    <!-- 长内容 -->
  </div>
</template>
```

## 6. 注意事项

- 元素必须是滚动容器（有确定高度 + 内容溢出）。
- 从未滚动过的元素不会触发（`last_top` 初始为 0，`scrollTop - 0` 不可能小于 0）。
- 与 `v-bottom` 一样**无 `beforeUnmount` 清理**，元素销毁时监听不会移除（源码事实）。
