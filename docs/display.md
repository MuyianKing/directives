# vDisplay（v-display）

> 包：`@mu/directives` ｜ 元素进入可视区，或浏览器标签页重新切回可见时，触发回调。典型场景：列表卡片曝光、可见时才拉取/刷新数据。

## 1. 用途与场景

- 元素每次**从不可见变为可见**时执行一次回调（持续可见期间不会重复）；标签页从后台切回前台时也会执行。
- 适合「需要感知用户真的看到它了」的场景。

## 2. 引入与注册

```js
import { vDisplay } from '@mu/directives' // 本包不在自动导入范围，必须显式 import
```

- `<script setup>`：导入变量名必须是 `vDisplay`，模板即可写 `v-display`。
- 全局：`app.directive('display', vDisplay)`。
- 源码：`src/display.js`。

## 3. 绑定值

| 项 | 说明 |
| --- | --- |
| 类型 | `Function`，**无参数** |
| 非函数时 | `mounted` 中直接跳过（不创建 observer、不绑事件） |

## 4. 行为契约（源码可验证）

1. `mounted`：若绑定值是函数，创建 `IntersectionObserver` 观察 `el`；回调中当 `entries[0]?.intersectionRatio > 0` 即调用 `binding.value()`。
   - 未配置 `thresholds`，即使用默认值 `[0]`：只在「不可见（比例 0）→ 可见（比例 > 0）」跨界时回调，**元素持续可见期间不会重复调用**。
2. 同时监听 `document` 的 `visibilitychange`：当 `document.visibilityState === 'visible'` 时调用 `binding.value()`。
3. observer 实例与事件处理函数分别挂在 `el._intersectionObserver`、`el.visibilitychange` 上。
4. `beforeUnmount`：`observer.disconnect()`，并 `document.removeEventListener('visibilitychange', ...)`，随后置空 —— **有清理**。

## 5. 示例

```vue
<script setup>
import { vDisplay } from '@mu/directives'

function onDisplay() {
  // 曝光埋点 / 需要时刷新数据
}
</script>

<template>
  <div v-display="onDisplay">内容</div>
</template>
```

## 6. 注意事项

- 注释「页面出现在可视区域调一次」中的「一次」限定的是**单次出现**，不是「整个生命周期只调一次」：每次从不可见变为可见调用一次，持续可见期间不重复；**离开后再出现会再次调用**。如果需要「整个页面只执行一次」，请在回调内部自行加标志位。
- `IntersectionObserver` 在 `observe` 时会**立即回调一次**（上报初始状态），所以若元素挂载时已在可视区内，回调会马上执行；反之初始不可见时不回调。
- 标签页切回可见（`visibilitychange`）也会触发一次，这条与元素是否在可视区**无关**，且与「进入可视区」共用同一个回调，回调内无法区分来源。
- 需要区分「进入 / 离开」两侧状态时用 [`v-visible`](./visible.md)。
