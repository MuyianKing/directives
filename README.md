# @mu/directives

> 版本 `0.7.1` · Vue 3 自定义指令包，具名导出 5 个指令对象：`vBottom`、`vTop`、`vLoadmore`、`vDisplay`、`vVisible`。

## 文档位置

**详细用法见 [`docs/`](./docs/README.md)，一个指令一个文件**；本文件只保留文档入口与版本记录，两者冲突时以源码为准。

- 索引入口：[docs/README.md](./docs/README.md)
- [bottom.md](./docs/bottom.md)：`v-bottom`（元素滚动接近底部）
- [top.md](./docs/top.md)：`v-top`（元素滚动接近顶部）
- [loadmore.md](./docs/loadmore.md)：`v-loadmore`（el-select 下拉加载更多）
- [display.md](./docs/display.md)：`v-display`（进入可视区 / 页面重新可见）
- [visible.md](./docs/visible.md)：`v-visible`（进入 / 离开可视区）

注册方式：本包**不在自动导入范围**，必须 `import { vBottom } from '@mu/directives'` 后自行注册（`<script setup>` 中导入变量名用 `vXxx` 即可在模板写 `v-xxx`；全局用 `app.directive('bottom', vBottom)`）。

## 版本记录

### 0.7.1

- 增加 vDisplay
