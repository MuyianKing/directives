# vLoadmore（v-loadmore）

> 包：`@mu/directives` ｜ 为 `el-select` 下拉设计：监听下拉滚动容器，滚动到底部（距底小于 20px）时触发回调，用于下拉分页加载更多。

## 1. 用途与场景

- 源码注释即「el-select 滚动到底部触发」，`mu-select` 内部用它做下拉加载更多。
- 监听的目标**不是绑定元素本身**，而是 `el.parentNode.parentNode`（下拉面板的滚动容器）。

## 2. 引入与注册

```js
import { vLoadmore } from '@mu/directives' // 本包不在自动导入范围，必须显式 import
```

- `<script setup>`：导入变量名必须是 `vLoadmore`，模板即可写 `v-loadmore`。
- 全局：`app.directive('loadmore', vLoadmore)`。
- 源码：`src/loadmore.js`。

## 3. 绑定值

| 项 | 说明 |
| --- | --- |
| 类型 | `Function`，**无参数** |
| ⚠️ 校验 | `mounted` 中**未做 `typeof` 校验**（与 `v-bottom` / `v-top` 不同），必须传函数，否则滚动到底时会抛错 |

## 4. 行为契约（源码可验证）

1. `mounted(el, binding)`：在 `el` 上写 `el.time_id = 0`，然后给 **`el.parentNode.parentNode`** 绑定 `scroll` 监听。
2. 每次 scroll：先 `clearTimeout(this.time_id)`，再 `setTimeout(..., 200)` —— 即 **200ms 节流**，停止滚动 200ms 后判断。
3. 触发阈值：`this.scrollHeight - (this.clientHeight + this.scrollTop) < 20`，满足则调用 `binding.value()`。
4. 定时器句柄存在绑定元素 `el.time_id` 上，靠 `this`（= 滚动的 `parentNode.parentNode`）访问。

## 5. 示例

```vue
<script setup>
import { vLoadmore } from '@mu/directives'

function loadMore() {
  // 追加下一页选项
}
</script>

<template>
  <!-- 用于 el-select 的下拉容器结构，el 是 select 内部元素，滚动容器是其 parentNode.parentNode -->
  <el-select v-model="value">
    <div v-loadmore="loadMore">
      <!-- 选项 -->
    </div>
  </el-select>
</template>
```

## 6. 注意事项

- **强依赖 DOM 结构**：只在「绑定元素的上上级就是滚动容器」时可用（`el-select` 的固定结构满足）。结构变化即失效，通用滚动场景请用 [`v-bottom`](./bottom.md)。
- 200ms 节流：连续滚动期间不触发，停止后触发一次；滚动到底且保持不动不会重复触发（除再次滚动）。
- **无 `beforeUnmount` 清理**：不会移除 scroll 监听、也不会清理 `time_id` 定时器（源码事实）。
- 绑定值必须传函数，源码未做类型保护。
