# 掌视 · 工作台 v1.1 · vibe-coding 实现指令

> 本文件面向前端 LLM（Cursor / Claude Code），不是给人看的设计说明。前端把本目录的 html + 本 md 一起喂给 AI 跑 vibe coding 还原。

## 元信息

| 项 | 值 |
|---|---|
| 系统 | 掌视 |
| 版本 | v1.1 |
| 栈 | **Antd-浅色**（React 18 + antd 6，defaultAlgorithm） |
| 主题色 | 青绿 `#00bf8a`（来源 `theme.json`，`colorPrimary`/`colorInfo` 同值） |
| 圆角 | 6（`borderRadius`） |
| 生成日期 | 2026-06-27 |
| 本版本 html | `workbench.html`（1 份） |

## 复用件清单（命中即 import，别照 html 重画空壳）

| 组件 | 出自（源码位置） | 本稿用法 |
|---|---|---|
| GlobalHeader 顶栏 | `storybook/antd/global-header.html`（原理 `references/complex-components-antd/global-header.md`） | 整页顶栏，`leftMode="select"`，应用切换 Select 定位「掌视工作台」，右侧导出/通知(徽标 3)/用户菜单/应用启动器 |
| LeftSider 左导航 | `storybook/antd/left-sider.html`（原理 `references/complex-components-antd/left-sider.md`） | 整页侧栏 220/60 可收起，两级 Menu，**必须连配套 `.vibe-sider` CSS 一起搬** |
| PageCanvas 页面地基 | `references/complex-components-antd/_foundation.md` | 灰底 `#f5f5f5` + 独立白卡 + 卡间距 16 |
| IconPark 图标系统 | `references/complex-components-antd/_foundation.md` | GlobalHeader 右侧 3 图标默认走 IconPark sprite（`el-icon-export`/`el-icon-warn1`/`el-icon-more1`） |
| 主题 theme.json | `theme.json` | `token.colorPrimary/colorInfo=#00bf8a` + `Layout.headerBg=#141414` + `Menu.*` 覆盖 |

> 工作台内容区（欢迎条/快捷入口/数据概览/任务/动态）均为 antd 原子拼装（Statistic / Progress / Timeline / Card / Row-Col / Flex / Tag / Button），无 DataTable / FilterBar（工作台非列表页）。

---

## workbench.html → 掌视工作台

**一句话定位**：掌视（AI 动态漫 / 分镜创作平台）创作者落地页，聚合快捷入口、数据概览、进行中任务与最近动态。

### DOM 结构骨架

```
Layout（minHeight 100vh）
├── LeftSider（Sider，220/60，.vibe-sider）
│   ├── Logo 区（50px，绿底「视」mark）
│   ├── Menu inline（两级：工作台 / 素材管理▾ / 创作中心▾ / 素材广场 / 数据中心▾ / 系统设置）
│   └── 底部收起按钮（48px）
└── Layout
    ├── sticky 顶栏容器（zIndex 10）
    │   └── GlobalHeader（高 50，左 Select / 右 导出·通知·用户·启动器）
    └── Content（灰底 #f5f5f5，padding 16，overflow auto）
        └── Flex vertical gap 16
            ├── 欢迎条 Card（头像「施」+ 你好施杨烁 + VIP标 + 日期 + [新建AI动态漫][导入素材]）
            ├── 快捷入口 Row（4 Col：新建动态漫 / 分镜工坊 / 上传素材 / 素材广场，hoverable）
            ├── 数据概览 Row（4 Col Statistic：今日生成12 / 素材总数1286 / 进行中任务3 / 算力用量320+圆环32%）
            └── Row gutter 16
                ├── Col xl=16 → 进行中的任务 Card（3 条带 Progress active）
                └── Col xl=8  → 最近动态 Card（Timeline 5 条）
```

### 视觉规格（只记和默认主题不一样 / 需落实的）

- 页面灰底 `#f5f5f5`（`colorBgLayout`）透出，内容分独立白卡（`#fff` + 圆角 6 + padding 20），卡间距 16。
- 顶栏高 50、白底（浅色）、底分隔线 `colorBorderSecondary`；侧栏白底 220/60、右边框 `colorBorderSecondary`。
- 主色 `#00bf8a`：快捷入口图标底 `colorPrimaryBg` + 图标 `colorPrimary`；算力圆环 `strokeColor=colorPrimary`；菜单选中 `itemSelectedBg=rgba(0,191,138,.15)` + 文字 `colorPrimary`。
- 统计大数字 `fontSizeHeading3` + `fontWeight 600`；趋势↑用 `colorSuccess`，↓用 `colorError`（本稿均为↑）。
- 侧栏菜单项高 50、左右 margin 10、选中胶囊圆角 8（`.vibe-sider` CSS 硬约束，别改）。

### 状态 & 交互

- **侧栏**：默认展开 `defaultOpenKeys=["create"]`、选中 `defaultSelectedKeys=["workbench"]`；点底部按钮收起（60px，vertical 模式 hover 弹右侧浮层）；两级 submenu 可展开收起。
- **顶栏 Select**：应用切换，默认值 `workbench`（掌视工作台），4 选项（AI动态漫/掌视工作台/素材广场/分镜工坊）。
- **顶栏通知**：`notificationCount=3` 显示数字徽标；用户菜单 Dropdown（需求反馈/同步权限/返回认证中心）；应用启动器 Dropdown。
- **快捷入口**：4 张 `Card hoverable`，hover 抬升。
- **数据概览**：3 个数值卡 + 1 个算力卡（`Progress type="circle" percent=32 size=64`）。
- **进行中任务**：3 条，每条 `Progress size="small" status="active"` 动画进度条；状态 Tag（即将完成=success / 生成中=processing）；右侧「查看」link。
- **最近动态**：`Timeline` 5 条，`color` 用 token（成功=colorSuccess / 主色=colorPrimary / 中性=colorTextTertiary），`dot` 自定义图标。

### 不画的边界（前端别重复写）

- **左导航 / 顶栏是复用的真组件**（GlobalHeader / LeftSider），前端直接 import 库里的组件，**不要照 html 重画导航壳**。本稿把它们内联进单文件只是因为 vibe 草稿是单 HTML 无 import——生产代码走组件库。
- 顶栏右侧 3 图标走 IconPark sprite（`<script src="...iconpark...">` + `Icon` 壳组件），前端项目里 IconPark 已接入则复用，勿回退 antd icons。
- 工作台内容区的菜单结构、快捷入口、数据指标、任务/动态文案均为**示例假数据**，前端按真实业务数据替换，结构保留。

---

## 给 AI 的实现指令

前端把这份 md 喂给 Cursor/Claude Code 时，请它：

1. 先 Read 本目录 `workbench.html`，对应实现一个工作台页面组件。
2. 栈走 **Antd-浅色**（React 18 + antd 6 + `theme.json` 主题），组件优先级：项目复杂组件库 > 全局复杂组件库 > antd 原生 > 自写。
3. **顶栏 / 左导航直接 import 复用件**（GlobalHeader / LeftSider，含 `.vibe-sider` 配套 CSS + IconPark sprite），不照 html 重画壳。
4. 严格不补 md 里没写的功能 / 占位 / 装饰；菜单/快捷入口/数据/任务/动态用真实业务数据替换，布局结构保留。
5. 视觉规格以 md 为准；md 没写的项保持默认主题（主色 `#00bf8a`、圆角 6、灰底白卡 16 间距）。
