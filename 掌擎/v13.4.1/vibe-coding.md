# vibe-coding 实现指令 · 掌擎 v13.4.1 · 创意文案新增爆款库

## 元信息

- **系统**：掌擎
- **版本**：v13.4.1
- **需求**：腾讯-游戏-创意文案新增爆款库（智能投放 / 智能创建3.0 / 游戏业务）
- **栈**：Element-浅色（真 Vue 2.7 + 真 element-ui 2.15 UMD + bi-eleme/bi-element-ui CSS 皮）
- **主题色**：#00bf8a（bi-eleme 编译期把 element 默认蓝硬覆盖成青绿）
- **生成日期**：2026-06-29
- **本次包含 html**：
  1. `爆款库.html` — 模块一：创意文案配置抽屉（含 4 Tab，新建标题/标题库选择 1:1 还原 creatV3 源码，爆款库为本次新增；新建标题 tab 点 AI 按钮打开的 AI 衍生弹窗 + 爆款挑选弹窗已合并进来）
  2. `AI创意文案衍生.html` — 模块二：AI 创意文案衍生弹窗 + 爆款库快捷调用（独立展示模块二展开态）
- **需求文档**：`/Users/admin/Desktop/v13.4.1腾讯-游戏-创意文案新增爆款库.md`
- **源码项目**：`/Users/admin/Desktop/zhangqing-frontend`（creatV3 路径：`packages/vue2/src/legacy/views/ADQ/popularize/creatV3/components/configDrawer/creativeTitleConfig/`）

---

## 一、爆款库.html（模块一）

### 页面身份
创意文案配置抽屉 - 爆款库 Tab。一句话定位：在 creatV3 创意文案配置抽屉里，于「新建标题 / 标题库选择」之后新增「爆款库」并列 Tab，承载爆款文案的录入/编辑/删除/筛选；同时 1:1 还原新建标题、标题库选择两个原有 Tab；新建标题 tab 的 AI 按钮点开 AI 衍生弹窗（含爆款库快捷调用）。

### 命中的源码文件（源码还原档）
- **Tab 容器**：`creativeTitleConfig/tab-index.vue`（文案分配逻辑 radio + 4 Tab + 各 Tab 面板切换）
- **新建标题**：`creativeTitleConfig/addText.vue`（1:1 还原：保存到标题库 switch + 动态词包 + 表情 + AI衍生按钮 + 标题列表拖拽/字数/编辑/删除）
- **标题库选择**：`creativeTitleConfig/selectedText.vue`（1:1 还原：搜索 + el-table selection + popover 编辑 + popconfirm 删除 + 分页）
- **AI 衍生弹窗**：`creativeTitleConfig/aiTitle.vue`（1:1 还原：左右分栏 + 输入文案/产品名/产品类型 + 生成/批量使用/保存使用；宽 62%、bi-dialog 去 padding、单行 input、爆款库链接绝对定位贴 label 右端）
- **爆款库 Tab + 爆款挑选弹窗**：本次新增（源码无，按需求 MD 新建）

### DOM 结构骨架
```
el-drawer（创意文案配置，rtl 62%，element 标准标题+关闭按钮）
└─ .creative-text-config（padding 20px）
   ├─ 文案分配逻辑 el-radio-group（sequence / fixed；fixed 才出"选择素材名称"Tab）
   ├─ [addText] 标题打乱 / 标题去重 按钮（size=small 32px）
   ├─ el-tabs（新建标题 / 标题库选择 / 爆款库 / 选择素材名称[v-if fixed]，第一个 tab padding-left:0）
   ├─ addText 面板（1:1 还原 addText.vue）
   │  ├─ .save-form 是否保存到标题库 el-switch
   │  ├─ .add-form 插入动态词包（+城市/+性别/+日期/+星期）
   │  ├─ .add-form.emoji-aigc-form 插入表情（el-dropdown 气泡，默认1个点击展开）+ AI创意文案衍生按钮
   │  └─ .text-list 新建标题列表（head-li + drag-wrap>item-li + input-li textarea small）
   ├─ selectedText 面板（1:1 还原 selectedText.vue）
   │  ├─ .search-table 标题名称搜索
   │  └─ el-table（selection + 标题列 el-popover 编辑 + 操作列 el-popconfirm 删除）+ el-pagination
   └─ hotLibrary 面板（本次新增爆款库）
      ├─ .filter-row 实时筛选（归属产品多选 + 创建时间 + 关键字，无查询按钮即时联动）
      ├─ .panel-toolbar（共N条 + 录入爆款按钮 size=small）
      ├─ el-table（文案 / 归属产品 / 标签 / 创建日期 / 操作[全绿]）
      └─ el-pagination（15/页）

el-dialog（AI创意文案衍生，62%，bi-dialog 去 padding）
└─ .ai-rules（flex 600px）
   ├─ .left-rules（45%）：创意文案 input + [爆款库链接 绝对定位贴 label 右端] + 产品名称 + 产品类型 + 生成按钮
   └─ .right-result（55%）：空态/加载态 + 结果列表（全选+批量使用+勾选+保存使用+重新生成）

el-dialog（爆款库挑选弹窗，900px，pick-dialog）
├─ .pick-search（归属产品 + 创建时间 + 关键字）
├─ el-table（radio单选 + 文案 + 归属产品 + 标签 + 创建日期）
└─ footer 取消/确认使用
```

### 复用件（命中库 + 源码位置）
- `el-drawer` / `el-tabs` / `el-tab-pane` / `el-table` / `el-table-column` / `el-dialog` / `el-popover` / `el-popconfirm` / `el-dropdown` / `el-switch` / `el-radio-group` / `el-button` / `el-input` / `el-tag` / `el-pagination` / `el-tooltip` / `el-checkbox`（element 原生）
- 源码 addText/selectedText/aiTitle 用 BiDialog（AI 弹窗），稿子用 el-dialog 等价还原
- 全局 element 复杂组件库：`references/complex-components-element/bi-components-cheatsheet.md`

### 视觉规格（仅记跟默认主题不一样的）
- 抽屉 `size: 62%`（rtl），`.creative-text-config` padding 20px
- el-table 基础款（无 border），cell `padding: 12px 16px`，表头表体统一行高 44px（`th.el-table__cell>.cell` 单独覆盖）
- **爆款库表格列**：文案（min-width 200，clamp 2 行省略+tooltip）/ 归属产品（min-width 140，el-tag info）/ 标签（min-width 240，el-tag success，>2 个收起 +N tooltip）/ 创建日期（min-width 120，nowrap）/ 操作（min-width 120，全绿左对齐常驻）
- 标签 `el-tag type="success" size="small"`（bi-eleme success = 浅绿底 #e6f8ea + 深绿字 #00b42a）
- 归属产品 `el-tag type="info" size="small" effect="light"`
- 操作列按钮全绿（`color:#00bf8a`），常驻显示不 hover 显隐，左对齐
- 录入爆款按钮 `size="small"`（32px）
- **编辑态 el-input `size="small"`（32px）+ `position:absolute` 浮在 cell 上不占行高**（避免点编辑瞬间行高跳变错位），cell `overflow:visible`，文案+归属产品可编辑，标签不可编辑
- AI 弹窗 62% 宽、bi-dialog 去 body padding、el-form size=small、创意文案单行 input、爆款库链接 `.title-item .hot-link` 绝对定位贴 label 右端（14px）
- 爆款挑选弹窗 900px 宽，文案列同样 clamp 2 行+tooltip

### 状态 & 交互
- **Tab 切换**：v-model activeName，状态保留；fixed 才出"选择素材名称"Tab
- **新建标题**（1:1 还原）：标题列表、字数 inputReplace 算法（词包替换：{{城市}}→城市城市=4字、{{性别}}→性别=2字、{{日期}}→日期日期日期=6字）、双击编辑 textarea、Enter 保存、删除、底部输入框 Enter 新增
- **标题库选择**（1:1 还原）：搜索、selection 多选、popover 编辑、popconfirm 删除、分页
- **爆款库**（本次新增）：
  - 筛选实时联动（computed：产品交集 + 时间 + 关键字模糊）
  - 录入爆款弹窗：归属产品（必填，filterable allow-create，Enter 沉淀字典）+ 标签（选填逗号分隔）+ 爆款文案（多行按行拆分+内部去重+库内去重），底部"将录入 N 条"
  - **行内编辑：文案 + 归属产品可编辑（el-input），标签不可编辑（纯展示）**，Enter 保存 Esc 取消，空文案/重复 toast 拦截，同时只允许一行编辑，新产品名沉淀字典
  - 删除：无二次确认直接删
  - 分页 15/页
- **AI 衍生**：addText 的 AI 按钮点开；生成（稿子 setTimeout 模拟，真线上调 getAiTitleList 接口）；批量使用勾选；保存使用 push 到 title_list；爆款库链接 → 挑选弹窗（默认第一个产品近七日）→ 选一条回填创意文案

### 不画的边界
- creatV3 整页骨架、左侧创建步骤、顶部导航、配置抽屉外层路由壳 — 归父级
- "选择素材名称"Tab 非本次需求，占位

---

## 二、AI创意文案衍生.html（模块二）

### 页面身份
AI 创意文案衍生 - 快捷调用爆款库。独立展示模块二展开态（跟爆款库.html 里合并的 AI 弹窗同款，独立成页方便单独看）。

### 命中的源码文件
- `creativeTitleConfig/aiTitle.vue`（1:1 还原）
- 爆款库链接 + 挑选弹窗：本次新增

### 复用件 / 视觉规格 / 状态交互
同爆款库.html 里合并的 AI 弹窗部分（62% 宽、bi-dialog 去 padding、单行 input、爆款库链接贴 label 右端、结果列表批量勾选、爆款挑选弹窗 900px el-table 单选回填）。

### 不画的边界
- AI 弹窗的触发（addText 的 AI 按钮）在模块一，本页独立展示弹窗展开态

---

## 给 AI 的实现指令

前端把这份 md 喂给 Cursor/Claude Code 时，请它：

1. 先 Read 本目录所有 .html（爆款库.html、AI创意文案衍生.html），每份对应实现一个页面组件
2. 栈走「Element-浅色」，真 Vue 2 + element-ui（bi-eleme/bi-element-ui CSS 皮），组件优先级：项目复杂组件库 > 全局复杂组件库 > element 原生 > 自写
3. **源码还原档**：新建标题/标题库选择/AI衍生 三个 Tab 是 1:1 还原 creatV3 源码（`creativeTitleConfig/addText.vue` / `selectedText.vue` / `aiTitle.vue`），照源码实现，不要自行改动结构；爆款库 Tab 是本次新增，按需求 MD + 本稿实现
4. 严格不补 md 里没写的功能 / 占位 / 装饰（不要擅自加 icon、提示文字、占位元素）
5. 命中复用件直接 import 源码组件（BiDialog/BiDrawer 等），不要照 html 重画空壳
6. 视觉规格以 md 为准；md 没写的项保持默认主题
7. 关键交互铁律：
   - 编辑态 el-input 用 absolute 不占行高，避免点编辑瞬间行高跳变错位
   - 按钮 32px = `size="small"`（bi-eleme 默认 36px）
   - el-table cell padding 12px 16px 行高 44px，表头表体一致
   - 标签 `el-tag type="success" size="small"`，多于 2 个收起 +N tooltip
   - 归属产品 `el-tag type="info" effect="light"`
   - 操作列按钮全绿常驻显示，不 hover 显隐
   - 文案列超出换行 2 行 + `...` + tooltip 看全部（-webkit-line-clamp:2）
   - **爆款库行内编辑：文案 + 归属产品可编辑，标签不可编辑**
