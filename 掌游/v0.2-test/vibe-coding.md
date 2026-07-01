# vibe-coding · 掌游工作台

> 面向 LLM 的实现指令。前端把这份 md + 同目录 html 喂给 Cursor/Claude Code。

| 字段 | 值 |
|---|---|
| 系统 | 掌游 |
| 版本 | v0.2-test |
| 栈 | Antd-浅色（React 18 + antd 6，默认 algorithm） |
| 主题色 | `#00bf8a`（青绿） |
| 生成日期 | 2026-07-01 |
| 包含文件 | pipeline-test.html |

---

## pipeline-test.html → 掌游工作台

**一句话定位**：游戏运营数据总览工作台，含统计卡片 + 筛选表格。

### DOM 结构骨架

```
Layout (全屏)
└── Content (padding:24)
    ├── 顶部标题栏 (flex, space-between)
    │   ├── 标题 "掌游工作台" + Tag (版本标记)
    │   └── 操作按钮组 (刷新 / 新建 / 设置)
    ├── 数据概览卡片行 (Row 4×Col, gutter=16)
    │   └── Card > Statistic + 同比趋势
    └── 游戏列表卡片
        ├── extra: 搜索 Input + 类型 Select + DatePicker
        └── Table (columns: 名称/类型/日活/流水/状态/更新时间/操作)
```

### 命中复用件

| 组件 | 来源 |
|---|---|
| Layout, Content | antd 原生 |
| Card | antd 原生 |
| Table + columns + pagination | antd 原生（非 DataTable 复杂组件） |
| Statistic | antd 原生 |
| Select, Input, DatePicker | antd 原生 |
| Tag, Badge | antd 原生 |

> 本次未命中掌游-组件库/复杂组件库.md 或全局 complex-components-antd 中的复杂组件，均为 antd 原生。

### 视觉规格

与默认 theme.json 一致，无额外覆盖：

| 项 | 值 | 来源 |
|---|---|---|
| 主色 | `#00bf8a` | theme.json |
| 圆角 | 6 | theme.json |
| 页面背景 | `#f5f5f5` | antd colorBgLayout |
| 卡片背景 | `#ffffff` | antd colorBgContainer |
| 涨色 | `#52c41a` | antd colorSuccess |
| 跌色 | `#ff4d4f` | antd colorError |

### 状态 & 交互

| 元素 | 行为 |
|---|---|
| 刷新按钮 | 触发数据刷新（无 loading 态，草稿用静态数据） |
| 新建按钮 (primary) | 点击无响应（草稿占位） |
| 搜索 Input | 可输入，无实际过滤逻辑 |
| 类型 Select | 下拉切换，无实际过滤 |
| DatePicker | 弹出日期选择器 |
| Table 列排序 (日活/流水) | sorter 可用 |
| 查看/编辑 链接 | 无响应（草稿占位） |
| 统计卡片 hover | Card hoverable 阴影反馈 |

### 不画的边界

- **无左导航**：Layout 内只有 Content，不画 Sider
- **无顶栏**：不画 Layout.Header
- **无路由壳**：单页 SPA，无页面跳转

---

## 给 AI 的实现指令

前端把这份 md 喂给 Cursor/Claude Code 时，请它：

1. 先 Read 本目录所有 .html，每份对应实现一个页面组件
2. 栈走「Antd-浅色」，组件优先级：项目复杂组件库 > 全局复杂组件库 > antd 原生 > 自写
3. 严格不补 md 里没写的功能 / 占位 / 装饰
4. 命中复用件直接 import，不要照着 html 重画一遍空壳
5. 视觉规格以 md 为准；md 没写的项保持默认主题