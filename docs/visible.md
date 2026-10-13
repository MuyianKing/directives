# vVisible（v-visible）

> 包：`@mu/directives` ｜ 监听元素进入 / 离开可视区，回调收到 `{ visible: boolean }`。典型场景：需要同时感知「出现」和「消失」两侧状态。

## 1. 用途与场景

- 与 [`v-display`](./display.md) 的区别：`v-display` 只在元素「变为可见」时回调（无参数，不关心离开）；`v-visible` 在**可见 ↔ 不可见两侧都会回调**，并告知当前状态。

## 2. 引入与注册

```js
import { vVisible } from '@mu/directives' // 本包不在自动导入范围，必须显式 import
```

- `<script setup>`：导入变量名必须是 `vVisible`，模板即可写 `v-visible`。
- 全局：`app.directive('visible', vVisible)`。
- 源码：`src/visible.js`。

## 3. 绑定值

| 项 | 说明 |
| --- | --- |
| 类型 | `Function`，**一个参数** `{ visible: boolean }` |
| 语义 | 每次交叉状态变化时调用；`visible` 为 `true` 表示进入可视区 |
| 非函数时 | `mounted` 中直接跳过（不创建 observer） |

## 4. 行为契约（源码可验证）

1. `mounted`：若绑定值是函数，创建 `IntersectionObserver` 观察 `el`（未配置 `thresholds`，默认 `[0]`）。
2. observer 回调：`binding.value({ visible: entries[0]?.intersectionRatio > 0 })` —— **进入（`true`）和离开（`false`）都会调用**。
   - 默认阈值 `[0]`：只在「不可见 ↔ 可见」跨界时回调；**持续可见或持续不可见期间都不会重复回调**（比例在 0 与 1 之间的变化不触发）。
3. 实例挂在 `el._intersectionObserver` 上。
4. `beforeUnmount`：`observer.disconnect()` 并置空 —— **有清理**。

## 5. 示例

```vue
<script setup>
import { vVisible } from '@mu/directives'

function onVisibleChange({ visible }) {
  if (visible) {
    // 进入可视区
  }
  else {
    // 离开可视区
  }
}
</script>

<template>
  <div v-visible="onVisibleChange">内容</div>
</template>
```

## 6. 注意事项

- `entries[0]` 可能为 `undefined`（理论上 observer 回调数组不为空，源码仍做了可选链保护）；此时 `visible` 为 `false`。
- `IntersectionObserver` 在 `observe` 时会**立即回调一次**，用于上报初始状态，请在回调里正确区分初始与后续变化。
- 判断依据是 `intersectionRatio > 0`，即只要有任何像素可见就算 `visible: true`。
- 「进入 / 离开」按**跨界**计：元素一直可见（哪怕比例从 30% 变成 80%）不会重复回调；只有从「完全不可见」变为「可见」或反之才回调。
