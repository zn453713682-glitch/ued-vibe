# vibe-coding.md · 灵翼 v1.4

## 顶部元信息

| 字段 | 值 |
|---|---|
| 系统 | 灵翼 |
| 版本 | v1.4 |
| 栈 | Element-暗黑（Vue 2.7 + element-ui 2.15.14 UMD + bi-eleme/bi-element-ui CSS + old-project-dark.css） |
| 主题色 | 暗色主色 `#05a77a`（hover `#21bb8a`，由 old-project-dark.css 派生；≠ 浅色 `#00bf8a`） |
| 生成日期 | 2026-06-25 |
| 本次 html | 作品管理.html |
| 还原来源 | `lingyi-frontend/src/views/Creation/index.vue` 1:1 还原（含交互） |

---

## 作品管理.html

### 页面身份
文件名 `作品管理.html` → 业务名「作品管理」（路由 `Creation/index.vue`，`CreationIndex`）。一句话定位：AI 精品漫作品管理列表页，列表/卡片双视图切换 + 新建/编辑作品 + 剧集/素材/成本管理 + 封面预览。设计师本次选「全部能点·弹窗轻量 + 图标走 element 字体」。

### DOM 结构骨架
```
#app
└── .creation-page                        // 业务页根，模拟 .app-main padding(0 15px 15px)
    ├── bi-block.creation-page__search    // 筛选区（源码用 BiBlock，非 BiSearchBlock）
    │   └── bi-form > bi-form-item × 6
    │       ├── 作品名称(el-input clearable)
    │       ├── 创建人(el-input clearable)
    │       ├── 内容组(el-select multiple filterable collapse-tags)
    │       ├── 作品类型(el-select filterable；选项=国内/海外短剧)
    │       ├── 状态(el-select；全部/制作中/制作完成)
    │       ├── [按钮] 查询 + 重置
    │       └── [按钮] (空占位，照源码)
    ├── bi-block                          // 内容区
    │   ├── .creation-page__toolbar       // 视图 toggle(列表/卡片) + 新建作品
    │   ├── v-if viewMode==='table'
    │   │   └── .bi-table.table-table
    │   │       ├── .bi-table-wrapper > el-table(12 列, size=medium, row-key=id)
    │   │       └── .pagination-container > el-pagination
    │   └── v-else
    │       └── .work-card-grid > .grid-container > .work-card × N (infinite scroll)
    ├── el-dialog (NewProjectForm)        // 新建/编辑作品，:visible.sync="visible"
    ├── el-drawer (DramaSeries)           // 剧集管理，:visible.sync="dramaSeriesOptions.visible"
    └── el-dialog (NewPreview)            // 封面预览，:visible.sync="previewVisible"
```

### 命中复用件（含源码位置）

| 组件 | 出处 |
|---|---|
| BiBlock / BiForm / BiFormItem（稿内起 shim） | `references/complex-components-element/bi-components-cheatsheet.md` §1/§3/§4 |
| BiTable 容器（`bi-table>bi-table-wrapper>el-table` + `pagination-container`） | `references/complex-components-element/bi-components-cheatsheet.md` §5 |
| layout-shell（壳，本稿不画，仅参考 padding/层级） | `references/complex-components-element/layout-shell.md` |
| **原页（还原本体）** | `lingyi-frontend/src/views/Creation/index.vue` |
| WorkCardGrid | `lingyi-frontend/src/views/Creation/components/WorkCardGrid.vue` |
| WorkCard | `lingyi-frontend/src/views/Creation/components/WorkCard.vue` |
| NewProjectForm | `lingyi-frontend/src/views/Creation/components/NewProjectForm/index.vue`（+ `dataSource/index.js` + `schemas/`） |
| DramaSeries | `lingyi-frontend/src/views/Creation/components/DramaSeries.vue` |
| NewPreview | `lingyi-frontend/src/components/NewPreview/index.vue` |
| dataSource（typeWorkOptions / TYPE_WORK_NAME_MAP / CONTENT_TYPE_NAME_MAP / styleOptions / domestic·overseasContentOptions） | `lingyi-frontend/src/views/Creation/components/NewProjectForm/dataSource/index.js` |
| API（getDramaList / deleteDrama / createDrama / updateDrama / getDramaDetail / getContentGroupOptions / getScriptOptions） | `lingyi-frontend/src/api/creationProject.js` + `costManagement.js` + `public.js` |

### 视觉规格（只记与 element 默认不同的项；暗黑）

- 页面背景 `#000`；卡片/组件背景 `#141414`；表头 `#1d1d1d`；悬浮/弹层 `#1f1f1f`；分隔线 `#303030`
- 主色 `#05a77a`（暗色派生；hover `#21bb8a`）—— **不要改成浅色 `#00bf8a`**
- 状态点：制作完成 `#00b42a` / 制作中 `#3491fa`（源码硬编码，非主色）
- 视图 toggle：70px 宽，`border:1px solid #303030`，bg `#141414`；toggle-btn 32×28，`is-active` bg `#2b2b2b`，hover bg `#1d1d1d`
- 卡片 WorkCard：`border-radius:8px`，cover `aspect-ratio:4/3`，info 区 `#1f1f1f` 高 68px；tag warning `#dc6d03`/边`#7e430b`，success `#039c27`/边`#0b5c1e`；hover 浮操作 24×24 `backdrop-filter:blur(5px)`，hover 变 `#00bf8a`
- 表格作品信息列：封面 `38×50` 圆角 4，名称两行省略
- 全局 `Vue.prototype.$ELEMENT = { size: 'small' }`（el-button 默认 32px，跟源码查询按钮 `height:32px` 对齐）；表格 `size="medium"`
- icon override CSS（§C 7 行）必须带：把 element runtime 内部硬编码的 `el-icon-arrow-up/down/left/right` / `circle-close` / `caret-top/bottom` 重新指到 bi-icons 新码位，让 select/input/table 内部图标走新形状
- 图标 class 名走 bi-icons 新名：`el-icon-down`（更多箭头）/ `el-icon-add`（新建）/ `el-icon-edit` / `el-icon-delete` / `el-icon-more`；`el-icon-folder`/`el-icon-video-camera`/`el-icon-money` bi-icons 无等价新名，保留老名

### 状态 & 交互（前端照抄 state machine）

- `viewMode`：`'table'` / `'card'`，切换写 `localStorage('creation_view_mode')`，mounted 读；切换时重置对应视图数据并加载
- 查询 `handleSearchAll` / 重置 `reset`：过滤 `allData` → 表格切片分页 / 卡片重载
- 分页 `handleSizeChange` / `handleCurrentChange`：`page-sizes [20,50,100,200]`，layout `total, sizes, prev, pager, next, jumper`
- 新建作品 `openAdd`：开 NewProjectForm dialog，`handler_type='add'`，viewType=1，表单重置
- 编辑 `handlerEdit`：开 dialog，`handler_type='edit'`，回显 rowData（viewType/scriptOptions/formData）
- 操作列：`status===2` 显示「查看」、否则「开始制作」，都走 `handleCreate` → `$router.push('Creation/taskInfo', {...})`
- 更多 popover（`trigger="hover"`，`popper-class="works-manage-popover"`，竖向菜单）：素材管理 / 剧集管理 / 成本分配 / 删除
- 删除 `handleDelete`：`$confirm('确定删除该作品吗？删除后不可恢复')` → 删 → 刷新当前视图
- 卡片操作 `getCardOperations`：编辑(el-icon-edit) + 素材管理(folder) + 剧集管理(video-camera) + 成本分配(money) + 删除(delete)；`operationMax=2` → 前 1 个明摆 + 其余进 `el-dropdown`(more)
- 卡片无限滚动：grid-container `@scroll` 滚到底 100px 触发 `loadCardData`，noMore 显示「没有更多了」；空 `el-empty`
- 封面图点击 `handlePreview`：开 NewPreview 全屏预览
- NewProjectForm：标题行「新建作品/编辑作品」+ viewType el-select（编辑时 disabled）；viewType 切换 → 作品类型选项随 domestic/overseas 变；footer 取消/确定
- DramaSeries：el-drawer 1000px rtl，标题「剧集管理」+ 上传状态 select(上传中/已完成)，tabs(完整版/纯净版)，本地上传按钮，剧集表格（剧集/文件名/时长/大小/付费集 switch/操作）
- NewPreview：全屏 100vw×100vh 黑底，左上返回按钮，中间大图，右侧 80×80 缩略图列表

### 不画的边界（前端别重复写）

Sidebar / Navbar / Logo / 面包屑 / Hamburger / 路由壳 —— 由 `src/layout/index.vue` 提供（layout-shell 沉淀，`.app-main` 出 `padding:0 15px 15px`，`.page-container` 内不要再加 padding）。本稿根 `.creation-page` 直接是 page-container 内容。

### 与源码的偏差（前端还原时换回）

1. **视图切换两图标**：源码 `<svg-icon svg-name="liebiaoshitu/kapianshitu">`（项目 svg sprite），稿内按设计师选「element 字体」用 `el-icon-menu`/`el-icon-s-grid` 近似替代 → **还原换回 svg-icon**
2. **NewProjectForm 弹窗**：按设计师选「弹窗轻量」——能开能关 + 新建/编辑标题切换 + 作品类型 select 可切 + footer 确定/取消；内部表单放示意字段（作品名/内容组/剧本/作品类型/比例/封面上传占位）可填但**不接 JsonForm schema 联动** → **还原照 `schemas/` 矩阵接 JsonForm**（国内外 × 内容类型 4 套 schema，`schemaMap[viewTypeKey_contentTypeKey]`）
3. **DramaSeries 抽屉**：轻量——标题+tabs+上传按钮+剧集表格(mock 5 条)，付费集 switch 可拨，上传/播放/下载给 `$message` 提示 → **还原照源码接 OSS 分片上传 / getVideoList / editVideo / 付费集接口**
4. **NewPreview**：轻量——全屏+返回+大图+缩略图，没接 zoom/rotate/多视图/副视图 → **还原照源码 NewPreview 全能力**（scale/rotate/drag/subView/footerAction）
5. **路由跳转**（查看/开始制作/素材管理/成本分配）：稿内 `$message` 提示不真跳 → **还原照源码 `$router.push`**
6. **数据**：稿内 mock 25 条（picsum 占位封面），权限 `buttonInfo` 全 true（所有按钮显示，`showMoreBtn` 走 `buttonInfo['creation-*']` 或运算） → **还原接真接口**（getDramaList/deleteDrama/createDrama/updateDrama/getDramaDetail/getContentGroupOptions/getScriptOptions）+ 真 `buttonInfo` 权限 + `v-permissionBtn` 指令
7. **筛选区**：照源码用 BiBlock+BiForm+BiFormItem（源码此页用 BiBlock，非 BiSearchBlock）；未按 antd 栈 FilterBar 三段式规范（FilterBar 仅 antd 栈，element 栈无此规范）
8. **localStorage**：稿内 `viewMode` 持久化照源码；`searchData` 筛选条件持久化稿内未做（源码有 `saveSearchConditions`/`initSearchConditions`）→ **还原照源码补**

---

## 给 AI 的实现指令
前端把这份 md 喂给 Cursor/Claude Code 时，请它：
1. 先 Read 本目录所有 .html，每份对应实现一个页面组件
2. 栈走「Element-暗黑」，组件优先级：项目复杂组件库 > 全局复杂组件库 > antd/element 原生 > 自写
3. 严格不补 md 里没写的功能 / 占位 / 装饰
4. 命中复用件直接 import，不要照着 html 重画一遍空壳
5. 视觉规格以 md 为准；md 没写的项保持默认主题
