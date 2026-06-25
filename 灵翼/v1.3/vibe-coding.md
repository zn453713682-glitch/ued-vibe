# 灵翼 v1.3 · 视频制作页（VideoGeneration）vibe coding 交付

> 给前端拿这份 md 喂 Cursor / Claude Code 跑 vibe coding 用。一份 md 覆盖本版本目录全部 html。

## 元信息

| 字段 | 值 |
|---|---|
| **系统** | 灵翼（漫剧制作） |
| **版本** | v1.3 |
| **栈** | Element-暗黑（Vue 2.7 UMD + element-ui 2.15.14 UMD + bi-eleme 编译期硬覆盖主色 + bi-element-ui + old-project-dark） |
| **主题色** | 主色 `#00bf8a`（亮 `#03a578` / `#05a77a` 用于 hover/active 文字态；bi-eleme 已把 element 蓝压成绿） |
| **图标** | IconPark sprite — `https://lf1-cdn-tos.bytegoofy.com/obj/iconpark/svg_42197_145.f8ff6b146312d80f7fdb7d06a62bc690.js` 145 个 symbol，统一 `<svg class="vibe-icon"><use href="#icon-xxx"></use></svg>` |
| **生成日期** | 2026-06-25 |
| **本次 html** | 1 份 — `2026-06-25-作品管理-AI精品漫-视频制作.html` |

---

## 1. `2026-06-25-作品管理-AI精品漫-视频制作.html`

### 1.1 页面身份

源码路径对应 `CreateNovelWorks/index.vue`（父路由壳：顶栏 breadcrumb + 4 个 step radio + 右上工具区）+ `VideoGeneration/index.vue`（第 4 步 tab 内容：视频制作主页）。本稿是 **「作品管理 → AI精品漫 → 开始制作 → 视频制作」深页**，第 4 步默认激活，右栏停在 **「分镜图生成」** tab 的 **已选素材态**（带 2 张已选素材卡 + 提示词面板 + 历史素材网格）。

### 1.2 DOM 结构骨架

```
#app .create-novel-works                     ← 全屏 flex 列容器 (100vh)
├─ .top_bar (64h, 3 列 grid: 面包屑 / 4-step radio / 右侧工具)
│  ├─ .page-header-breadcrumb (返回按钮 + el-breadcrumb + 作品设定 tag)
│  ├─ .radio_box (4 × .radio_item, .is-active = 第 4 步「视频制作」)
│  └─ .box-right (任务列表/日志/帮助 icon + divider + 用户菜单 + 九宫格)
└─ .page-container (flex:1, 黑底)
   └─ .video-generation-container (padding 16/20/20, flex 列)
      ├─ .top-bar (智能成片按钮 + 分镜/场景/描述信息 + 历史记录/批量增强/导出)
      └─ .video-generation-container-wrapper (flex 横, 主体两列)
         ├─ .video-generation-container-wrapper-left (flex:1)
         │  ├─ .main-content (主视频预览, 局部重绘悬浮按钮)
         │  ├─ .subtitle-overlay-list (字幕条独立卡 + 台词与配音按钮)
         │  └─ bi-block.bottom-panel
         │     ├─ .controls-row (章节 select + 多选分镜 + 共 N 个分镜 + 播放/时间 + 画质 select)
         │     └─ .carousel-container (左右 nav + 22 张 .shot-card + .add-shot-card)
         └─ .video-generation-container-wrapper-tabs (540 宽, #141414 卡)
            └─ .right_box > el-tabs (3 个: 分镜图生成 / 视频生成 / 视频增强)
               └─ .tab-content > .new-create-image-by-ai .panel-card
                  ├─ .section-block (选择素材: 2 × .material-card + 1 × .material-add-bundle)
                  ├─ .section-block (提示词 + 优化链接 + .prompt-panel)
                  │  └─ .prompt-panel (textarea + .prompt-footer: model-select + config-trigger-box)
                  ├─ .generate-btn (开始生图, 40h 实心 #00bf8a)
                  └─ .new-production-log (历史素材: header + .history-grid)
                     └─ 4 × .history-card (1 上传 + 3 缩略, 中间一张 .is-selected)
```

### 1.3 命中的复用件

| 组件 | 出处 | 用法 |
|---|---|---|
| `BiBlock` | 源码 `src/components/bi-block`（也对应 [`references/complex-components-element/bi-components-cheatsheet.md`](../../../../vibe%20design/skill/references/complex-components-element/bi-components-cheatsheet.md) 第 1 节）| 底部分镜条外层白卡（暗主题色 `#141414`），本稿用本地 shim 注册（template `<div class="bi-block"><slot/></div>`），前端直接 `import BiBlock from '@/components/bi-block'` 替换 |
| `el-popover` | element-ui 原生 | 智能成片悬浮工作流面板（`video-gen-workflow-popover`）+ 提示词优化（`optimize-prompt-popper`）+ 配置 trigger（`task-list-config-popover`）+ 视频版本（`video-version-popover`）+ 提示词回退 tooltip。**全部用 popper-class 隔离样式** |
| `el-breadcrumb` / `el-breadcrumb-item` | element-ui 原生 | 顶栏路径：漫剧制作 → 《请君入我怀1633 王建杭》 |
| `el-button` | element-ui 原生 | 智能成片 / 历史记录 / 批量视频增强 / 导出 / 局部重绘 / 台词与配音 / 开始生图 / 应用 / 取消 / 开始优化。**默认 size 走全局 `Vue.prototype.$ELEMENT = { size: 'medium' }`**；底部分镜条工具栏的 select 用 `size="small"` |
| `el-select` / `el-option` | element-ui 原生 | 4 处：章节 / 多选分镜（multiple + collapse-tags） / 画质 / 模型（model-select，带 prefix `.model-icon`） |
| `el-input` (textarea) | element-ui 原生 | 提示词主输入（`type=textarea` + `maxlength=3000` + `show-word-limit` + `resize=none`）+ 优化 popover 内 textarea（`:rows=5`） |
| `el-tabs` / `el-tab-pane` | element-ui 原生 | 右栏 3 tab（视频增强 tab `v-if=hasVideoInfo` 控制显隐） |
| `el-dropdown` / `el-dropdown-menu` / `el-dropdown-item` | element-ui 原生 | 顶部「导出」3 项（台词文件 / 分镜视频 / 成片） |
| `el-icon-arrow-left/right/down` `el-icon-plus` `el-icon-download` `el-icon-delete` | element-ui 原生 font icon | 配合 IconPark 使用 |
| **IconPark 145 sprite** | 远程 CDN（见元信息） | 主用：`#icon-renwuliebiao` `#icon-wenjian` `#icon-wenhao` `#icon-kapianshitu` `#icon-piliangshengchengtishici` `#icon-piliangshengchengfenjingtu-hui` `#icon-piliangzhuanshipin-hui` `#icon-buzhoutiao-hui` `#icon-AItubiao` `#icon-shangchuan` `#icon-jubuzhonghui` `#icon-jiaosepeiyin` `#icon-bofang-zanting` `#icon-bofang-bofang` `#icon-shipinyulan-zuo` `#icon-shipinyulan-you` `#icon-zanwufenjingtu` `#icon-jia` `#icon-copy` `#icon-chongzhi` `#icon-AIgongju` `#icon-jubuchonghui` |

> **不引复杂组件库**。所有 Bi* 复杂件（BiTable / BiSearchBlock / BiNoticeBox 等）在本稿没出现，前端按上面表里走原生 + 本地视觉即可。

### 1.4 视觉规格（**只记和默认主题不同的，相同不重复**）

#### 全局背景层级
- 页面底：`#000`
- 一级卡（主视频、字幕条、底部分镜条 BiBlock、右栏 tabs 容器、popover 面板）：`#141414`
- 二级卡（material-card / material-add-bundle / history-card / prompt-panel / config 选项底）：`#1f1f1f`
- 三级填充（model-select inner / config-trigger-box）：`#2a2a2a`
- 描边：`#303030`（material/prompt-panel 1px 实线）/ `rgba(255,255,255,.14)`（popover 实线）/ `rgba(48,48,48,1)`（顶栏 / tab 底部分隔）

#### 文字层级
- 主：`rgba(255,255,255,.95)` 或 `.9`
- 次：`rgba(255,255,255,.85)`
- 辅：`rgba(255,255,255,.65)` (label / hint / desc 等)
- 占位：`rgba(255,255,255,.45)` (placeholder / 计数 / disabled icon)
- 极弱：`rgba(255,255,255,.25)` (整组 disabled，如 workflow popover 中第二/三步 icon)

#### 主色专用文字态
- 命中文字 `#03a578`（radio active / 字幕条 hover / play icon hover）
- 命中文字 `#05a77a`（workflow `__mode-item.is-active`）
- 标准主色 `#00bf8a`（按钮实底 / config-option active / config-trigger-box hover 边 / 头像渐变起点 / new-production-log__more hover / optimize-link hover）

#### 关键尺寸
- 顶栏 `top_bar` 64h；breadcrumb 行 50h
- step radio 32h × auto-width；`__step` 18² 圆
- 工具区 icon 32²
- VideoGeneration 内边距 `16px 20px 20px`
- 主视频右下「局部重绘」`.ch-btn` absolute right/bottom 12
- 字幕条 `.subtitle-overlay-list` 上下 padding 10、右 padding 140（让出右上「台词与配音」按钮位）、上下 margin 12
- 分镜缩略卡 160 × 90，圆角 4，`.thumbnail-wrapper.active` 2px `#03a578` 边
- 分镜 hover 右侧 `.shot-operation` 白色竖条 16 × 90，圆角右开
- 字幕条/底部 BiBlock 用 `#141414`，不是默认 element 白
- 右栏 `.video-generation-container-wrapper-tabs` 固定 540 宽
- `.tab-content` 高度 `calc(100vh - 64px - 215px)`，溢出滚但 hide scrollbar
- material-card / material-add-bundle / history-card 全部 158² 正方
  - material-card 圆角 4，material-add-bundle 圆角 8（hover gap 注意）
- material-card 上左 tag：20h `padding 0 8`、`bg rgba(31,31,31,.6) + backdrop-blur 4 + 1px rgba(255,255,255,.15) 边`
- material-card 底部 info：内 padding `16/8/6`，渐变 mask `linear-gradient(to bottom, 0/0 → .4/49.94% → .8/100%)`
- 提示词 `.optimize-link`：14/500，灰 `.65`，hover 主色 `#00bf8a`（**不是 AI 渐变**）
- `.prompt-panel`：1f1f1f + 1px #303030 实线，圆角 8，padding 8，**flex 列 + gap 8，不锁高度**（关键：锁 269h 会把 footer 挤出可见区，textarea 走 `min-height: 180`）
- `.prompt-footer`：flex 行 gap 8；左 model-select `flex:1`、右 config-trigger-box 160 固定宽，都 36h
- `.model-select .model-icon`：16² 圆角 2，渐变 `linear-gradient(180deg, #343773 0%, #1b3c47 100%)`，白色「即」9/700
- `.generate-btn`：100% 宽 × 40h，实底 `#00bf8a !important`（**不带渐变，不是 44h**），上 margin 16
- `.history-card.is-thumb`：158²，**`.is-selected` 2px 纯白边 + 永远显示 .bottom-apply**；hover 也显示 apply
- `.history-card.is-thumb .hover-actions .icon-btn`：24² 圆角 6，`bg rgba(31,31,31,.5) + backdrop-blur 5 + 1px rgba(255,255,255,.21) 边`，hover bg 加深到 .8
- `.config-option`：min-width 44 × 28h，padding 0 10，bg `rgba(255,255,255,.06)`，active 状态 bg `rgba(0,191,138,.15) + 文字 #00bf8a`
- `.config-option .ratio-icon`：`1px solid currentColor` 圆角 2 的小框，16:9 = 14×8，9:16 = 8×14
- 用户头像：24² 圆，渐变 `linear-gradient(135deg, #00bf8a, #04caa1)`
- 「作品设定」tag：24h padding 0 8，`bg rgba(0,191,138,.15) + color #00bf8a`，圆角 2

### 1.5 状态 & 交互

> 全部 SPA 内态切换；任何"toast"= `this.$message({ ... duration: 1200 })`。

| 触发 | 行为 |
|---|---|
| 父 radio 点击 | `activeRouteTab = tab.value` → page-container 切到对应页（v1.3 只完整实现第 4 步「视频制作」，其他 tab 显示「未在 v1.3 主路径展开」） |
| 「智能成片」hover | 弹出 320 宽 popover：顶部 2 个 mode（标准 / 全能），切换时 `workflowTransitionName` 决定左/右进入动画；标准模式 3 行 step（第一步主色实心按钮，二三步 disabled） |
| 「智能成片 → 标准 → 批量生成提示词」点击 | `handleBatchText()` toast |
| 顶部 mode 切换 (`handleWorkflowModeChange`) | 同模式不切，不同模式比当前更靠后用 left/前用 right 动画 |
| 「历史记录」/「批量视频增强」 | 各自 toast |
| 「导出」dropdown | 3 项：导出台词文件 / 导出分镜视频 / 导出成片 |
| 「局部重绘」(主视频右下 + history hover) | toast |
| 「台词与配音」(字幕条右上) | `openDubbing()` toast |
| 分镜 chapter / shot multi / quality select | 纯 v-model，不打外接 |
| play/pause | toggle `isPlaying`，icon use `#icon-bofang-zanting` ↔ `#icon-bofang-bofang` |
| 缩略条左/右 nav | `prevShot/nextShot`，到边界 `.is-disabled` 不可点 |
| 分镜卡点击 | `selectShot(idx)` 切 `currentShotIndex`；`currentShot.shotNumber` 同步刷顶栏「分镜:」 |
| 分镜卡 hover | 右侧 `.shot-operation` 16 宽白条出现（新增 / 复制），点击 `stopPropagation`，各自 toast |
| 「添加分镜」末尾卡 | `handleAddShot()` toast |
| 右栏 tab 切换 | `activeTab = '分镜图生成' / '视频生成' / '视频增强'`；只「分镜图生成」走完整内容，其他两个 tab 灰文字占位 |
| material-card / add-bundle 行点击 | `openMaterial(type)`：role/scene/replace 三种 toast |
| 「提示词优化」点击 | toggle `optimizePopoverVisible`；popover 内 textarea + 取消/开始优化 |
| 优化 popover 「开始优化」 | 空校验 → toast 提示；非空 → toast 显示输入内容、关 popover |
| 「提示词回退」icon | `rollbackPrompt()` toast |
| 配置 trigger-box（`{分辨率} · {比例} · {数量}`） | 点击切 `configVisible`；popover 内 3 组 chip 切换 active；配置实时反到 trigger 文案 |
| 「开始生图」 | toast 拼 `ai_code · configText` |
| 历史素材「查看更多」/「上传」/卡点击/下载/删除/应用 | 各自 toast |

#### v-if 显隐总览（按源码）

- `activeRouteTab === 4` → 显示 VideoGeneration；其他显示 `「{tab 名}（未在 v1.3 主路径展开）」`占位
- workflow popover `workflowMode === 'standard' / 'omnipotent'`：两种 panel 互斥
- `activeTab === '分镜图生成'`：完整内容；其他两个 tab 灰文字占位
- 右栏 tab 第 3 个「视频增强」: `v-if="hasVideoInfo"` 控制是否渲染 tab-pane（本稿默认 false 不出）
- `shot.isOriginal` 控制左上「原片」tag 出不出
- `shot.dubbing` 有值显示 `.dubbing-chip`（带 dur 主色文字 + 内容），无值显示 `.dubbing-empty`「0s -」
- `optimizePopoverVisible` / `configVisible` 各自 manual 控

### 1.6 不画的边界

- **左侧全局导航 + 整站顶栏（外层路由 / 系统 layout 壳）已经存在**，本稿只到 `.create-novel-works` 这一层（即父路由 CreateNovelWorks 的 64h `.top_bar` + 内嵌 page-container）；前端别再套一层左侧导航。
- **「视频生成」「视频增强」两个 tab 不展开**（业务在源码 `CreateVideoByAI` / `CreateVideoEnhance` 里，v1.3 本路径不包），前端按真实组件接入即可，本稿只保 tab 切换。
- **「角色配置」「场景配置」「分镜表」三步占位**（v1.3 主交付的就是第 4 步），前端从其他版本/路由接入。
- **真实数据接口**全部 mock（22 个分镜静态数组、章节 1-3、画质 5 项、模型 3 项），前端按各自接口接。
- **BiBlock 是本地 shim**（一个 `<div class="bi-block">` 包 slot），前端 import 真组件 `@/components/bi-block` 替换；本稿样式只在 `.bi-block` 命名空间加暗主题 padding，不重画其他装饰。
- **静态图片资源**`https://frontend-oss.zzwwiidd.com/lingyi/images/queshengVideo.png`（缺省视频占位），前端按业务环境替换。

---

## 给 AI 的实现指令

前端把这份 md 喂给 Cursor/Claude Code 时，请它：

1. 先 `Read` 本目录所有 `.html`，每份对应实现一个页面组件（本版本只 1 份 = 视频制作页 `VideoGeneration/index.vue`）
2. 栈走「**Element-暗黑**」（Vue 2.7 + element-ui 2.15.14 + bi-eleme + bi-element-ui + old-project-dark），组件优先级：项目复杂组件库（BiBlock）> 全局复杂组件库 > element 原生 > 自写
3. 严格不补 md 里没写的功能 / 占位 / 装饰（特别注意：**「视频生成」「视频增强」tab 内不要补内容**，本版本明确未展开）
4. 命中复用件直接 `import`，不要照着 html 重画一遍空壳（BiBlock 必须 import 源码组件，html 里的 `.bi-block` 暗色 padding 可保留，但别把 shim 那行 `Vue.component('bi-block', …)` 抄过去）
5. 视觉规格以 md 为准；md 没写的项保持默认主题
6. **暗色主题硬约束**：背景层级严格按「页面 #000 / 一级卡 #141414 / 二级 #1f1f1f / 三级 #2a2a2a」走，不要任何位置出现 element 默认浅色边框/灰白底
7. **主色用法**：实底按钮一律 `#00bf8a`（不带渐变）；命中文字态可用 `#03a578` 或 `#05a77a`（保持 hover/active 微差）
8. **prompt-panel 千万别锁死高度** — 用 `display:flex column + gap` 让 textarea 用 `min-height` 撑高，footer 走自然流。锁 `height: 269` + textarea `flex:1` 会让 footer 被挤出可见区（已踩过坑）
