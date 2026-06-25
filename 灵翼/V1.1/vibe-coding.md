# 灵翼 V1.1 · 作品管理 · 前端实现指令

> 给前端的 vibe coding 指令包。**不是设计说明、不是 PRD**——这是给 LLM（Cursor/Claude Code）直接吃的实现规约，请把本目录里 `Creation-list.inline.html` + 本文件一起喂进去。

---

## 元信息

| 字段 | 值 |
|---|---|
| 系统 | 灵翼 |
| 版本 | V1.1 |
| 栈 | **Element-暗黑**（老项目 Vue 2 + Element + Bi*） |
| 主题色 | `#00bf8a`（青绿，bi-eleme 编译期硬覆盖了 element 蓝） |
| 暗色基底 | 页面 `#0a0a0a` / 卡片白底 `#1f1f1f` / 边色 `#303030` |
| 生成日期 | 2026-06-24 |
| 本次包含 | `Creation-list.inline.html`（1 份） |

---

## Creation-list.inline.html — 作品管理列表页

**业务名**：作品管理（创作中心）。一句话定位：列表 / 卡片双视图 + 5 字段筛选 + 新建作品 dialog + 剧集管理 drawer，承接灵翼 AI 短剧的「项目→剧集→镜头」三级。

### DOM 结构骨架（关键容器层级）

```
.app-main > .page-container > .creation-page
├── <bi-search-block>             ← 筛选区容器（Bi*）
│     └── <bi-form>               ← 筛选表单
│           ├── <bi-form-item label="作品名称">      el-input
│           ├── <bi-form-item label="创建人">        el-input
│           ├── <bi-form-item label="内容组">        el-select multiple
│           ├── <bi-form-item label="作品类型">      el-select
│           ├── <bi-form-item label="状态">          el-select
│           └── <bi-form-item btn>                  el-button*2（查询/重置）
└── <bi-block>                    ← 主内容白卡（Bi*）
      ├── .creation-page__toolbar
      │     ├── .creation-page__view-toggle  ← 列表/卡片视图切换 segmented
      │     └── el-button[type=primary]      ← 「+ 新建作品」
      ├── v-if='table' → .bi-table > .bi-table-wrapper > el-table（11 列含 fixed 操作列）
      │                  + .pagination-container（el-pagination）
      └── v-else → .work-card-grid → .work-card * N（卡片视图，见下）
```

外层弹窗 / 抽屉（不在 `.creation-page` 内、用 `append-to-body`）：

- `el-dialog.new-project-dialog`：新建/编辑作品（7 字段）
- `el-drawer.drama-drawer`：剧集管理（带内嵌表格 + 上传按钮）

### 卡片（.work-card）结构 — Figma node 324:2109 1:1

> 这版按设计师 figma 选中卡片回灌过的，**不是上下分块的旧 WorkCard.vue**。

```
.work-card                                  ← aspect-ratio:260/313, min-height:313
├── img.work-card__cover                   ← 全卡封面（absolute inset:0 / object-cover）
├── .work-card__top                         ← 左状态徽章 + 右更多按钮
│     ├── span.work-card__status            ← 白底胶囊：● + 「制作中/制作完成」 + ▾
│     │     └── .work-card__status-dot      ← 8×8 圆点：橙(.--success 走绿)
│     └── el-popover[trigger=hover]         ← hover 才显示更多按钮
│           ├── slot=reference: button.work-card__more  ← 28×28 毛玻璃黑底「⋯」
│           └── .more-popover-list          ← 暗色弹层 4 项
│                 ├── 编辑
│                 ├── 素材管理
│                 ├── 项目云盘
│                 └── 删除（红色，--danger）
└── .work-card__info                        ← 底部 180px 渐变信息层（pointer-events:none）
      ├── .work-card__name                  ← 16px/600 白
      └── .work-card__meta
            ├── .work-card__meta-row（机位组）
            │     └── .work-card__meta-item--primary  ← 14px/0.88 白：⊕ 内容组名
            └── .work-card__meta-row（数字三联）
                  ├── 👤 创建人
                  ├── 📁 N作品
                  └── 🎬 N视频
```

底部渐变写死：`linear-gradient(180deg, rgba(2,19,42,0) 0%, rgba(5,16,32,0.6) 42.4%, #0c0d0f 100%)`，hover 不变。

### 命中的复用件（前端 import，**别照 html 重画**）

| 组件 | 用途 | 源码位置 |
|---|---|---|
| `BiBlock` | 主内容白卡容器 | 项目 `bi-element-ui`（cheatsheet §1） |
| `BiSearchBlock` | 筛选区容器（**注意：用 `search-block-box`，不是裸 BiBlock**） | 项目 `bi-element-ui`（cheatsheet §2） |
| `BiForm` / `BiFormItem` | 筛选表单和 5 字段 + btn slot | 项目 `bi-element-ui`（cheatsheet §3、§4） |
| `BiTable` | **正式实现请用 BiTable 替换 html 的 `el-table`**（含分页 + 吸顶 + 横滚 fixed 列兜底） | 项目 `bi-element-ui`（cheatsheet §5） |
| `el-table` 列定义 | 暂以 el-table-column 写死，迁 BiTable 时把 columns 配置抽出来 | element-ui 原生 |
| `el-pagination` | 卡片视图分页（表格视图分页归 BiTable 自己内含） | element-ui 原生 |
| `el-popover` | 卡片「更多」hover 菜单 + 表格操作列「更多」 | element-ui 原生 |
| `el-tooltip` | 视图切换两个按钮的提示 | element-ui 原生 |
| `el-dialog` | 新建/编辑作品 dialog | element-ui 原生 |
| `el-drawer` | 剧集管理抽屉 | element-ui 原生 |
| `el-form` / `el-input` / `el-select` / `el-radio-group` | dialog 内 7 字段 | element-ui 原生 |

> **不要 detach 真组件**：所有 `bi-*` / `el-*` 走真 import，html 里只是为了静态原型渲染才在 `libs/element/bi-element-ui/*` 用 shim 模拟，**前端项目里不要把 shim 也搬过去**。

### 视觉规格（**只记和默认主题不一样的**，相同的不写）

| 项 | 值 | 备注 |
|---|---|---|
| 主色 | `#00bf8a` | 老项目硬覆盖（[[bi-eleme-light-primary-is-green]]）；前端项目继承默认 |
| el-button 默认 size | `small` (32 高) | [[element-button-size-small]]，写表单/工具条按钮记得带 `size="small"`，不写会比同行高一截 |
| .work-card 卡片高度 | `min-height:313px` + `aspect-ratio:260/313` | grid 自适应列宽 |
| .work-card 圆角 | 8px | 整卡 + 封面 + 信息层共用 |
| .work-card 网格列 | `repeat(auto-fill, minmax(260px, 1fr))` gap:16 | |
| 底部渐变 | `linear-gradient(180deg, rgba(2,19,42,0) 0%, rgba(5,16,32,0.6) 42.4%, #0c0d0f 100%)` | 高 180px |
| 状态徽章 | 白底 #fff / 文字 #323335 13px / 状态点 8×8（橙=#f59a23 / 成功=#00bf8a） | figma 原色，**不要换暗色** |
| 「更多」按钮 | 28×28 / 圆角 6 / `rgba(31,31,31,0.5)` + `backdrop-filter:blur(6px)` + 边 `rgba(255,255,255,0.21)` | hover 才出 |
| 卡片 hover | 不加边、不变色，只显「更多」按钮 | 跟列表视图一致 |
| dialog 封面上传区 | 120×150 dashed `#4b4b4b` 暗色虚框，hover 边变主色 | dialog 内字段区 24px gap |
| drawer 标题区 | padding:16px 20px / border-bottom #303030 | |

### 状态 & 交互

| 元素 | 行为 |
|---|---|
| 工具条「列表/卡片」segmented | 切 `viewMode = 'table' \| 'card'`，**真切换**，不是横向并排展示（[[source-restore-is-spa-not-stage]]） |
| 工具条「+ 新建作品」 | `openNewProject()` 不传参 → 打开 dialog 走「新建」分支（清空表单） |
| el-table 操作列「编辑」 | `openNewProject(row)` 把行带进 dialog 填字段，dialog 标题切「编辑作品」 |
| el-table 操作列「查看 / 开始制作」 | 按 `row.status` 切：2=查看，1=开始制作（暂只 toast，业务侧接） |
| el-table 操作列「更多」 popover | 4 项：素材管理 / 剧集管理 / 成本分配 / 删除（最后红色） |
| 卡片整卡 click | `openDramaSeries(c)` 打开剧集管理 drawer |
| 卡片右上「⋯」 hover | 出 popover：编辑 / 素材管理 / 项目云盘 / 删除（最后红色），项目云盘是 V1.1 新加 |
| 卡片左上状态徽章 | **目前是纯展示**，figma 设计了 ▾，业务侧若要做下拉切状态再接 |
| dialog「视频比例」 | radio 三选：9:16 / 16:9 / 21:9 |
| dialog「作品类型」 | radio 三选：实拍剧 / 动漫 / 动画 |
| dialog 关闭 | `:close-on-click-modal="false"`，必须点 footer 按钮关 |
| dialog 提交 | toast「已创建/已更新」+ 关闭，业务侧自接 API |
| drawer 内表格上传 | 仅 UI 占位，实际接业务上传组件 |

### 不画的边界（**前端别重复实现**）

- 左侧导航 / 顶部 navbar / Layout 整壳：**不在本 html 里**（看 [[old-project-layout-shell-ready]]，老项目壳已沉淀），前端项目里这页只画 `page-container` 内部。
- 弹窗 / 抽屉的真 API：先用 mock toast，业务侧自接。
- BiTable 的吸顶 / 横滚 / fixed 列同步：**走 BiTable 自带逻辑**，html 里写的 `#app .bi-table .el-table__body-wrapper{overflow-x:auto}` 是 storybook vibe-patch 反盖兜底（[[element-table-body-wrapper-overflow]]），前端项目里别把它当业务样式抄过去——BiTable 自己处理。
- 主题切换：本页只在暗色站，前端不需要做主题切换的 transition：none 兜底（[[element-theme-flash-transition]]）。

### 关键暗坑提醒（迁码时别踩）

1. **BiSearchBlock 必须套 search-block-box**，裸 BiBlock 间距会塌（[[bi-element-ui-cheatsheet-ready]]）
2. **BiTable 完整壳**：`.bi-table > .bi-table-wrapper > el-table + 独立 .pagination-container`，不是裸 el-table（cheatsheet §5）
3. **el-input 必须 v-model**，不写会出现「打不了字」假象（[[element-input-needs-vmodel]]）
4. **el-popover 的 popper-class** 要带，`.more-popover-popper` 暗色样式靠它命中（裸 popover 是浅色）
5. **bi-icons 字形**：`el-icon-rank` `el-icon-folder` `el-icon-video-camera` `el-icon-arrow-down` `el-icon-add` 都是老项目 element-icons 老 glyph，class 用错名 → 拿到错字体（[[bi-icons-two-font-cascade]]）

---

## 给 AI 的实现指令

前端把这份 md 喂给 Cursor/Claude Code 时，请它：

1. 先 `Read` 本目录所有 `.html`，每份对应实现一个页面组件（这一版只有 `Creation-list.html` → `CreationList.vue`）
2. 栈走「**Element-暗黑**」，组件优先级：项目复杂组件库（Bi*）> 全局复杂组件库 > element-ui 原生 > 自写
3. 严格不补 md 里没写的功能 / 占位 / 装饰
4. 命中复用件直接 `import`，不要照着 html 重画一遍空壳（特别是 BiTable / BiSearchBlock / BiForm）
5. 视觉规格以 md 为准；md 没写的项保持默认主题
6. 卡片视图（`.work-card`）按 figma node 324:2109 还原，**底部渐变和状态徽章颜色不要改**，那是 figma 原色
