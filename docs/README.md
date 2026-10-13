# @mu/directives 指令文档

本目录是 `@mu/directives` 的**详细用法文档**：一个指令一个文件，目标是「只看文档、不看源码就能写出正确代码」。

- 权威版本：`0.7.1`，入口 [`../index.js`](../index.js)，具名导出 5 个指令对象：`vBottom` / `vTop` / `vLoadmore` / `vDisplay` / `vVisible`。
- 包级变更记录见 [`../README.md`](../README.md)；两者冲突时**以源码为准**。

## AI 查阅指引（先读这里）

1. 按「要监听什么」查下表，直接打开对应文件，不要先翻源码。
2. **本包不在自动导入 / 自动注册范围**：应用里的 `MuUiResolver` 只解析以 `Mu` 开头的组件，`@mu/directives` 的指令必须显式 `import` 后注册（见下一节）。
3. 文档只写源码可验证的事实；存疑处标注「以源码为准」并给出文件路径。

## 指令索引

| 指令 | 文档 | 用途 | 触发时机 | 绑定值 |
| --- | --- | --- | --- | --- |
| `v-bottom` | [bottom.md](./bottom.md) | 元素自身滚动接近底部 | 向下滚且距底 < 20px（300ms 防抖） | 函数（无参） |
| `v-top` | [top.md](./top.md) | 元素自身滚动接近顶部 | 向上滚且 `scrollTop < 10`（300ms 防抖） | 函数（无参） |
| `v-loadmore` | [loadmore.md](./loadmore.md) | `el-select` 下拉滚动到底部 | 父级滚动容器距底 < 20px（200ms 节流） | 函数（无参，未做校验） |
| `v-display` | [display.md](./display.md) | 元素进入可视区 / 页面重新可见 | 每次进入可视区，或标签页切回可见 | 函数（无参） |
| `v-visible` | [visible.md](./visible.md) | 元素进入 / 离开可视区 | 每次交叉状态变化 | 函数（`{ visible }`） |

## 通用约定（全部指令适用）

- **引入与注册**：`import { vBottom } from '@mu/directives'`。在 `<script setup>` 中导入变量命名为 `vBottom` 时，模板可直接写 `v-bottom`；全局注册用 `app.directive('bottom', vBottom)`。
- **绑定值统一为函数**：除 `v-loadmore` 外，其余指令在 `mounted` 时都会判断 `typeof binding.value === 'function'`，不满足则静默不生效（不报错）。
- **只实现 `mounted`**：`v-display` / `v-visible` 另有 `beforeUnmount` 做清理，`v-bottom` / `v-top` / `v-loadmore` **没有卸载清理**，元素销毁时不会移除已绑定的事件监听（源码事实，见各文档「注意事项」）。
- **滚动类指令监听绑定元素自身**（`v-loadmore` 例外，见其文档）：元素必须是真正的滚动容器（有固定高度且内容溢出），否则不触发。
- 仓库当前 `app/src` 未使用任何本包指令，无现成业务用例可对照；接入前请按本文档自行验证。

## 变更记录

| 日期 | 变更内容 |
| --- | --- |
| 2026-10-13 | 初始化文档目录：索引 + 5 个指令文档（内容以 `0.7.1` 源码为准）。 |
| 2026-10-13 | 修正 [display.md](./display.md) / [visible.md](./visible.md)：按 `IntersectionObserver` 默认阈值 `[0]` 说明「只在不可见 ↔ 可见跨界时回调」，`v-display` 注释「调一次」指单次出现调一次（非整个生命周期只调一次）。 |
