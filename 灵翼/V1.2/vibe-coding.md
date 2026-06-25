# 灵翼 · 作品管理列表 · 老项目暗黑还原 · V1.2

> 给前端的实现指令（喂给 Cursor/Claude Code 跑 vibe coding）。不是给人看的设计说明。

---

## 元信息

| 字段 | 值 |
|---|---|
| 系统 | 灵翼 |
| 版本 | V1.2 |
| 栈 | **Element-暗黑**（老项目 · 真 Vue 2.7 + 真 element-ui UMD + bi-eleme + bi-element-ui + old-project-dark.css） |
| 主题色 | 主色 `#05a77a`（暗黑由 bi-eleme `#00bf8a` 派生），dark 框/底色走 `old-project-dark.css` |
| 还原源码 | `src/views/Creation/index.vue` 全量分支 |
| 生成日期 | 2026-06-25 |
| 包含文件 | `2026-06-23-作品管理列表-老黑-v3.inline.html` |

---

## 1. 作品管理列表（`2026-06-23-作品管理列表-老黑-v3.inline.html`）

### 页面身份

- 业务名：**作品管理 · 列表页**（创作中心 / Creation）
- 一句话定位：列表 / 卡片双视图切换的作品管理页，含筛选、新建作品 dialog、剧集管理 drawer、作品预览 dialog；老项目暗黑模式 1:1 还原。
- 形态：**单页 SPA**，所有分支用真 state 切换（`v-if="viewMode==='card'"` / `:visible.sync="dialogVisible"` / `<el-popover trigger="hover">`）。**绝不拆成多帧并排橱窗**。

### DOM 骨架（5 块）

```
<div class="creation-page page-container">

  <!-- ① 顶部筛选区 · BiBlock + BiForm + BiFormItem×5 + 查询/重置 -->
  <bi-block class="creation-page__search">
    <bi-form>
      <bi-form-item label="作品名称">  → el-input v-model="searchData.name"
      <bi-form-item label="创建人">    → el-input v-model="searchData.creator"
      <bi-form-item label="内容组">    → el-select multiple collapse-tags v-model="searchData.group_ids"
      <bi-form-item label="作品类型">  → el-select v-model="searchData.script_type"
      <bi-form-item label="状态">      → el-select v-model="searchData.status"
      <bi-form-item> slot=button       → el-button type=primary 查询 + el-button 重置

  <!-- ② 主体区 · BiBlock + 视图切换 + 表格/卡片 -->
  <bi-block>
    <div class="creation-page__toolbar">
      <div class="creation-page__view-toggle">
        <toggle-btn is-active="viewMode==='table'" → svg-icon: liebiaoshitu>
        <toggle-btn is-active="viewMode==='card'"  → svg-icon: kapianshitu>
      </div>
      <el-button type=primary @click="openNewProject">新建作品</el-button>
    </div>

    <!-- ②a 列表视图 v-if="viewMode==='table'" -->
    <bi-table :data :loading :column :total theme="dark" size="medium" sticky-top=50 row-key="id">
      slot #name              → 封面 + 两行省略名称（点击 handlePreview）
      slot #proportion_name   → 9:16 / 16:9 / 21:9 文本派生
      slot #work_type_name    → script_type / style 拼接
      slot #episode_num       → text-align:right
      slot #create_time       → 日期 + 时间换行
      slot #supplier_name     → finish_shot_num/total_shot_num
      slot #status            → 圆点 + 文字（绿=完成 / 蓝 #3491fa = 制作中）
      slot #update_time       → 日期 + 时间换行
      slot #handler           → 编辑 + 查看/开始制作 + 更多 popover(hover)
                                  popover 内：素材管理 / 剧集管理 / 成本分配 / 删除
    <el-pagination layout="total, sizes, prev, pager, next, jumper" :page-sizes="[20,50,100,200]">

    <!-- ②b 卡片视图 v-else (work-card-grid) -->
    <div class="grid-container">
      <div class="work-card" v-for="row in tableData">
        .work-card__cover-wrap
          .work-card-cover
            .work-card-cover__blur (背景图模糊)
            .work-card-cover__overlay
            img.work-card-cover__image
          .work-card__actions (hover 显)
            tooltip: 编辑 / 素材 / 剧集 / 更多 dropdown(成本/删除)
        .work-card__meta (名称、类型、剧集数、状态点)

  <!-- ③ 新建/编辑作品 Dialog · :visible.sync="newProjectVisible" -->
  <el-dialog title="{{ newProjectForm.id ? '编辑作品' : '新建作品' }}" width=520 custom-class="works-manage-new-project-dialog">
    <el-form :model="newProjectForm" label-width="84">
      作品类型     · radio: 国内 / 海外
      内容类型     · radio: 三档（按 script_type 派生 styleOptionsByType）
      内容组       · el-select projectGroups
      关联剧本     · el-select scripts
      作品名称     · el-input maxlength=30 show-word-limit
      视频比例     · radio: 9:16 / 16:9 / 21:9
    footer: 取消 / 确定(loading)

  <!-- ④ 剧集管理 Drawer · :visible.sync="dramaSeriesVisible" size=70% -->
  <el-drawer title="剧集管理" size="70%" custom-class="drama-series-drawer">
    el-tabs :
      tab-pane "剧集列表" → bi-table episodeColumns + 选中操作栏
      tab-pane "上传记录" → 占位

  <!-- ⑤ 作品预览 Dialog · :visible.sync="previewVisible" -->
  <el-dialog custom-class="new-preview-dialog" :show-close=false :close-on-click-modal :modal-append-to-body>
    左侧大图 / 视频 + 右侧 thumbnail 列表（previewList[previewIndex]）
```

### 命中的复用件（前端按这份 import，不要重画）

| 组件 | 出处（源码路径） | 用法 | 备注 |
|---|---|---|---|
| `<bi-block>` | `src/components/Bi/BiBlock` | 筛选区/主体区外壳 | **筛选区**外面再裹 `class="search-block-box"` 走 BiSearchBlock 路径（cheatsheet §2）；主体区裸 `<bi-block>` 即可 |
| `<bi-form>` / `<bi-form-item>` | `src/components/Bi/BiForm` | 筛选 form | `bi-form-item` 通过 `slot="content"` 装控件、`slot="button"` 装查询/重置按钮 |
| `<bi-table>` | `src/components/Bi/BiTable` | 列表视图主表 + 剧集 drawer 内表 | **`theme="dark"` `size="medium"` `:sticky-top="50"`** 必填；列定义看 §列表 columns；外层包 `.bi-table-wrapper`（cheatsheet §5） |
| `<svg-icon svg-name="liebiaoshitu" />` `<svg-icon svg-name="kapianshitu" />` | 项目 svg sprite | 视图切换 toggle 图标 | **保留项目 sprite 路径**，前端按真路径接，不要换成 element 字体 |
| `el-input` / `el-select` / `el-option` / `el-button` / `el-pagination` / `el-tooltip` / `el-popover` / `el-form` / `el-form-item` / `el-radio` / `el-radio-group` / `el-dialog` / `el-drawer` / `el-tabs` / `el-tab-pane` / `el-switch` / `el-empty` / `el-dropdown` / `el-dropdown-menu` / `el-dropdown-item` | element-ui | 各处 | 老项目栈，按 element-ui v2 真组件接，**不要手搓 div** |
| `popper-class="works-manage-popover"` | 业务 popover overlay 样式 | "更多" hover popover | 见 §4 popover 注意：`padding:0` + item 自带 padding，hover bg 不能溢出 popper 圆角 |
| `custom-class="works-manage-new-project-dialog"` / `drama-series-drawer` / `new-preview-dialog` | 业务 dialog/drawer overlay 样式 | 三个弹层 | overlay 样式跟着 dialog 走，挂 `custom-class` 即可 |

### 视觉规格（只记和默认主题不一样的）

- **栈**：Element-暗黑。CSS 顺序：`bi-eleme/index.css` → `bi-element-ui/index.css` → `old-project-dark.css`（顺序不能错）
- **主色**：bi-eleme 编译期硬覆盖 element 默认蓝为 `#00bf8a`（亮）；暗黑由 `old-project-dark.css` 派生 `#05a77a` —— **不要手填色值**，靠 css 链路即可
- **整页 padding**：`page-container { padding: 15px }`（layout-shell 已写明，**不在业务页里再加 padding**）
- **el-button 默认 size="small"（32 高）**：老项目硬规则，不写 size 会比同行高一截；本稿筛选区查询/重置已显式 `style="height: 32px"`
- **状态点**：制作完成 → 默认绿（继承主色），制作中 → 显式蓝 `#3491fa`（仅这一处）
- **图标**：走 element 字体 + svg sprite 双轨。**class 名用 bi-icons 新名**（`el-icon-down` 而不是 `el-icon-arrow-down`，`el-icon-add` 而不是 `el-icon-plus`），并在 `<style>` 粘 [html-prototype-element.md §8](references/complex-components-element/) 的 7 行 runtime override
- **popover overlay**：`works-manage-popover` 内层 `padding:0`，`el-button--text` item 自带 padding 8px 12px；hover bg `border-radius` 跟 popper 内边距对齐，**hover bg 不能溢出 popover 圆角外**
- **el-table body-wrapper overflow**：vibe-patch 全局 `overflow:visible` 会让 fixed 列横滚失效，业务页要在 `#app .bi-table` 域内反盖（cheatsheet §5 已写明）
- **暗色 input-number 边色**：补 inc/dec 边色要带 `.is-controls-right`，否则被原生 (0,3,0) 蓝灰边盖回（本稿无 input-number，留作通用提醒）

### 状态 & 交互（按源码 1:1 接）

| 元素 | 行为 | 触发 state |
|---|---|---|
| 视图切换 toggle | `switchViewMode('table'/'card')` 切 viewMode | `viewMode` |
| 查询 | 600ms 模拟 + `$message.success('查询完成')` | `loading` |
| 重置 | 清空 searchData 全字段 + `$message.info('已重置')` | `searchData` |
| 新建作品按钮 | 重置 form + `newProjectVisible=true` | `newProjectVisible`、`newProjectForm` |
| 行 · 编辑 | 填充 form + 打开 dialog | 同上 |
| 行 · 查看/开始制作 | `$message`（按 status 派生文案） | — |
| 行 · 更多 hover | 真 `<el-popover trigger="hover">` 展开 | element 自管 |
| 行 · 删除 | `$confirm` 二次确认 | element 自管 |
| 行 · 剧集管理 | `dramaSeriesVisible=true` | `dramaSeriesVisible` |
| 行 · 封面点击 | `handlePreview(row)` 打开预览 dialog | `previewVisible`、`previewList`、`previewIndex` |
| 卡片视图 hover | 真 css `:hover` 显 `.work-card__actions` | css |
| 新建作品确定 | 500ms 模拟 + 关闭 + `$message.success`（按 `id` 派生文案） | `newProjectLoading`、`newProjectVisible` |
| URL hash 自动展开 | `mounted()` 按 `#new` / `#edit` / `#drama` / `#preview` 自动开对应分支 | 演示用 |

### 表格列定义（前端照搬，**不要改字段顺序**）

```js
tableColumns: [
  { label: '作品信息',     prop: 'name',             minWidth: 340, slotScope: true },
  { label: '剧本名称',     prop: 'script_name',      minWidth: 200 },
  { label: '内容组',       prop: 'group_name',       minWidth: 96  },
  { label: '视频比例',     prop: 'proportion_name',  minWidth: 96,  slotScope: true },
  { label: '作品类型',     prop: 'work_type_name',   minWidth: 200, slotScope: true },
  { label: '剧集数',       prop: 'episode_num',      minWidth: 96,  slotScope: true,
    renderHeader: (h) => h('div', { style: 'text-align:right;width:100%' }, '剧集数') },
  { label: '负责人',       prop: 'creator',          minWidth: 96  },
  { label: '创建时间',     prop: 'create_time',      minWidth: 160, slotScope: true },
  { label: '已完成/总镜头', prop: 'supplier_name',   minWidth: 130, slotScope: true },
  { label: '状态',         prop: 'status',           minWidth: 144, slotScope: true },
  { label: '最近更新时间', prop: 'update_time',      minWidth: 160, slotScope: true },
  { label: '操作',         prop: 'handler',          fixed: 'right', slotScope: true, width: 200 },
]

// 剧集 drawer 内 columns（DramaSeries.episodeColumns 1:1）
episodeColumns: [
  { label: '',         prop: 'selection',  type: 'selection', width: 55, align: 'center' },
  { label: '剧集',     prop: 'script_no',  width: 80 },
  { label: '文件名称', prop: 'name',       minWidth: 200 },
  { label: '剧集时长', prop: 'duration',   width: 120,
    render: (h, { row }) => h('span', `${Math.floor(row.duration/60)}分${row.duration%60}秒`) },
  { label: '视频大小', prop: 'video_size', width: 120,
    render: (h, { row }) => h('span', `${(row.video_size/1024/1024).toFixed(1)}MB`) },
  { label: '付费集',   prop: 'is_pay',     width: 90,  slotScope: true },
  { label: '操作',     prop: 'actions',    width: 240, fixed: 'right', slotScope: true },
]
```

### 不画的边界（前端别重复写）

- ❌ **左导航 Sidebar / 顶栏 Navbar / Layout 外壳**：归 `src/layout/` 已有；本稿只画 `.page-container` 内部
- ❌ **page-container padding**：`.app-main` 已派生，不要再加
- ❌ **路由壳 / 资源菜单挂载点**：归 router 配置，本稿不涉及
- ❌ **真业务接口**：本稿数据全是 mock（`tableData` / `projectGroups` / `scripts` / 剧集 mock 数据等），前端接真接口

---

## 给 AI 的实现指令

前端把这份 md 喂给 Cursor/Claude Code 时，请它：

1. **先 `Read` 本目录所有 .html**（本版本只有 1 份），对应实现 `src/views/Creation/index.vue` 的暗黑模式 1:1 还原
2. **栈走 Element-暗黑**（老项目 Vue 2 + element-ui v2 + bi-eleme + bi-element-ui + old-project-dark.css）
3. **组件优先级**：项目复杂组件库（`src/components/Bi/*`）> 全局复杂组件库 > element-ui 原生 > 自写
4. **严格不补 md 里没写的功能 / 占位 / 装饰**——比如不要给「作品类型」字段加 `v-if`、不要自己排新建作品 form 字段顺序、不要给 page-container 加 padding
5. **命中复用件直接 import**，**不要照着 html 重画一遍空壳**——`<bi-block>` `<bi-form>` `<bi-table>` 都是真组件
6. **真 state 切换分支**：viewMode 切表格/卡片、`:visible.sync` 开关 dialog/drawer、`<el-popover trigger="hover">` 真 hover 展开 —— 不要拆成多帧
7. **视觉规格以 md + html 自检为准**；md 没写的项保持 element-ui v2 + bi-eleme + old-project-dark.css 默认三层链路派生值，不要手填色值
8. **图标走双轨**：`<svg-icon svg-name="…">` 走项目 sprite；element 字体 class 用 **bi-icons 新名**（`el-icon-down` 而非 `el-icon-arrow-down`），并粘 7 行 runtime override
9. **popover / dialog / drawer overlay 样式**通过 `popper-class` / `custom-class` 挂业务样式（`works-manage-popover` / `works-manage-new-project-dialog` / `drama-series-drawer` / `new-preview-dialog`），不要 inline
