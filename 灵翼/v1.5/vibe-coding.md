# 灵翼整站 · vibe-coding 实现指令

## 元信息

- **系统**：灵翼
- **版本**：v1.5
- **栈**：Element-暗黑（Vue 2.7 + element-ui UMD + bi-eleme/bi-element-ui CSS + old-project-dark.css）
- **主题色**：暗色主色 `#05a77a`（hover `#21bb8a`）；页面背景 `#000`，卡片/组件 `#141414`，表头 `#1d1d1d`
- **生成日期**：2026-06-29
- **本次包含 html**：`灵翼整站.html`（1 份，整站可导航 SPA）
- **还原来源**：`/Users/apple/Downloads/产品所有原型/`（`index.html` + `assets/js/app.js` 5634 行 SPA 原型），保留信息架构，做视觉/布局打磨

---

## 灵翼整站.html

**页面身份**：`灵翼整站.html` → 灵翼 · AI漫剧制作平台（整站可交互）。一句话定位：把原产品原型做成一份带壳的可导航 SPA——左侧导航 + 顶栏 + 9 个主列表页真切换、弹窗真弹、筛选真生效、AI生图真演出 loading→完成。

### DOM 结构骨架

```
#app (v-cloak)                          ← dark.css 作用域前提，必须包
└─ .app-wrapper [:class sider-collapsed]
   ├─ <left-sider>                      ← 复用全局通用库 LeftSider（aside.vibe-sider 子树）
   │   ├─ .vibe-logo (slot=header)      ← 50px，蝴蝶 logo + "灵翼"
   │   ├─ .vibe-menu-wrap > el-menu     ← 两级 el-submenu / el-menu-item
   │   └─ .vibe-footer                  ← 48px，收起按钮
   └─ .main-container [margin-left:220/60]
      ├─ header.navbar                  ← 50px，面包屑 crumbs + top-actions(下载/通知/头像/九宫格)
      └─ section.app-main > .page-container
         └─ v-if currentPage 切 9 个分支：
            dashboard / script-explain / script-boutique /
            works-explain / works-boutique / material-role /
            material-scene / ai-erase / ai-enhance / ai-image
```

### 命中的复用件（前端跳源码用）

| 组件 | 出自 |
|---|---|
| `LeftSider`（`aside.vibe-sider` 子树 + 配套 CSS） | 全局通用库 `references/complex-components-element/left-sider.md` + 真相 `storybook/element/left-sider.html` |
| `el-menu` / `el-submenu` / `el-menu-item`（LeftSider 内部真组件） | bi-eleme `el-menu` |
| `el-table`（包在 `.bi-table` 容器内 = BiTable 骨架） | bi-element-ui BiTable；容器类 `.bi-table` |
| `el-dialog` / `el-form` / `el-form-item` / `el-input` / `el-select` / `el-radio` / `el-radio-group` / `el-radio-button` / `el-checkbox` / `el-checkbox-group` / `el-button` / `el-dropdown` / `el-dropdown-menu` / `el-dropdown-item` / `el-badge` | bi-eleme 原生 |
| 业务容器类 `.block-box` / `.search-row` / `.search-item` / `.toolbar` / `.page-title` | 稿内 `<style>`（dark.css 风格，非库件） |

> 前端实现时：LeftSider 直接按 `left-sider.md` 注册成项目组件复用，**不要照着 html 重画 el-menu**；表格走项目 BiTable 容器（`.bi-table`），不要裸 `el-table`。

### 视觉规格（仅记与默认主题不同的暗黑值）

- 页面背景 `#000`；卡片/`.block-box`/弹层内表单底 `#141414`；表头 `#1d1d1d`
- 主色 `#05a77a`（按钮/链接/激活态/选中胶囊淡底 `rgba(5,167,122,.18)`），hover `#21bb8a`
- 主要文字 `rgba(255,255,255,.85)`；次要 `.65`；描述/占位 `.45`
- 分隔线 `rgba(255,255,255,.08)`；卡片边 `rgba(255,255,255,.05~.06)`
- 状态点：绿 `#05a77a` / 蓝 `#2d8cf0` / 红 `#f5655b` / 黄 `#e6a23c` / 灰 `rgba(255,255,255,.35)`
- 侧栏：展开 220 / 收起 60；一级菜单项高 50、二级 44（缩进 padding-left 40）；选中态圆角胶囊 8
- navbar 高 50；`.app-main` padding `0 15px 15px`；`.page-container` padding-top 14
- `.block-box` 圆角 4、padding `18px 20px`
- 图标 runtime override：`el-icon-arrow-up/down/left/right`、`circle-close`、`caret-top/bottom` 已重指到 bi-icons 码位（§8.C），前端沿用项目 bi-icons 字体即可

### 状态 & 交互（前端照抄 state machine）

- **导航切换**：`<left-sider @select="go">` → `go(key)` 设 `currentPage`，`v-if` 切页；`@collapse` 折叠侧栏（`.sider-collapsed` 改 `margin-left`）
- **筛选**：每页搜索字段 `v-model` 绑 data，`computed` `filteredXxx` 真过滤表格/卡片数据；查询按钮 toast、重置清空
- **新建弹窗**：4 个 `el-dialog :visible.sync`——新增剧本 / 新建作品 / 新建项目 / 新建任务（字幕擦除·视频增强共用，`taskKind` 切标题与"增强方案"字段）
- **表格行操作**：查看 / 编辑 / 锁定↔解锁（改 `row.locked`）/ 删除（`$confirm` 确认后 toast）/ 更多（`el-dropdown`：素材管理、删除）
- **AI精品漫**：列表↔卡片视图切换 `boutiqueView`；卡片内项目状态可下拉改
- **AI生图**：`generateImage` 演出"提交→loading 骨架卡→1.6s 后完成 toast"，记录 `unshift` 进 `aiRecords`，可按模型过滤；提示词带 `{{aiPrompt.length}}/10000` 计数
- **素材广场**：角色/场景 media-card 网格，hover 出下载/删除操作
- **反馈**：`toast` 走 `$message` success；删除走 `$confirm` warning

### 不画的边界（前端别补）

- **detail 子页未做**：剧本编辑、素材详情、字幕擦除详情、视频增强详情、漫剧制作、项目云盘分享——本次只做 9 个主列表页，详情页不在本稿
- **分页器未加**：演示数据量小，未放 `el-pagination`；前端按真实接口分页补
- **顶栏图标为占位**：下载/通知 badge、头像、九宫格仅视觉，未接下拉
- **首页（小说库）**：banner + 资讯卡片为占位视觉，未接真实资讯源

---

## 给 AI 的实现指令

前端把这份 md 喂给 Cursor/Claude Code 时，请它：
1. 先 Read 本目录所有 .html，每份对应实现一个页面组件
2. 栈走「Element-暗黑」，组件优先级：项目复杂组件库 > 全局复杂组件库 > antd/element 原生 > 自写
3. 严格不补 md 里没写的功能 / 占位 / 装饰
4. 命中复用件直接 import，不要照着 html 重画一遍空壳（LeftSider、BiTable 尤其重要）
5. 视觉规格以 md 为准；md 没写的项保持默认主题
