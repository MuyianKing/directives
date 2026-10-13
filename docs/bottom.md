# vBottom（v-bottom）

> 包：`@mu/directives` ｜ 监听元素自身滚动，接近底部（距底小于 20px）且**向下滚动**时触发回调。典型场景：自绘滚动容器分页加载更多。

## 1. 用途与场景

- 给一个**真正的滚动容器**（固定高度 / 内容溢出）绑定，滚到接近底部时执行回调。
- 与 `v-top` 成对：一个管底部、一个管顶部。
- 如果目标是 `el-select` 下拉（DOM 结构特殊），用 [`v-loadmore`](./loadmore.md)。

## 2. 引入与注册

```js
import { vBottom } from '@mu/directives' // 本包不在自动导入范围，必须显式 import
```

- `<script setup>`：导入变量名必须是 `vBottom`，模板即可写 `v-bottom`（Vue 局部指令命名约定）。
- 全局：`app.directive('bottom', vBottom)`，模板写 `v-bottom`。
- 源码：`src/bottom.js`。

## 3. 绑定值

| 项 | 说明 |
| --- | --- |
| 类型 | `Function`，**无参数** |
| 语义 | 触底时被调用；内部不做防重复保护，防重由 300ms 防抖承担 |
| 非函数时 | `mounted` 中直接跳过，不绑定监听，也不报错 |

## 4. 行为契约（源码可验证）

1. `mounted(el, binding)` 时若 `binding.value` 不是函数，**什么都不做**。
2. 回调用 `useDebounceFn(..., 300)`（VueUse）包装，**300ms 防抖**（最后一次触发后延迟 300ms 才真正执行）。
3. 元素上维护 `el.lastScrollTop`，初始为 `0`；每次 scroll 比较：
   - `el.lastScrollTop - scrollTop > 0`（本次是**向上滚**）→ 更新 `lastScrollTop` 后直接 `return`，**不触发**。
   - 否则更新 `lastScrollTop`，继续判断阈值。
4. 触发阈值：`el.scrollHeight - (el.clientHeight + el.scrollTop) < 20`，即滚动条距底小于 20px。
5. 监听的是**元素自身的 `scroll` 事件**，不是 window/document。

## 5. 示例

```vue
<script setup>
import { vBottom } from '@mu/directives'

function handleBottom() {
  // 加载下一页
}
</script>

<template>
  <div class="h-[300px] overflow-y-auto" v-bottom="handleBottom">
    <!-- 长内容 -->
  </div>
</template>
```

## 6. 注意事项

- 元素必须是滚动容器：需要有确定高度且内容溢出，否则永远没有 scroll 事件。
- 不会在挂载时自动触发一次，只有真实滚动才触发。
- 向上滚动一律不触发，所以「先滚到底、再往上滚回底部附近」不会触发。
- **无 `beforeUnmount` 清理**：元素销毁时不会移除该 `scroll` 监听（源码事实）。在会被反复创建销毁的容器上使用时注意监听泄漏风险，必要时自行用其它方式控制生命周期。
