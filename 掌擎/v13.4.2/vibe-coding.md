# 一刀法 · vibe-coding 实现指令

## 顶部元信息

| 项 | 值 |
|---|---|
| 系统名 | 掌擎 |
| 版本号 | v13.4.2 |
| 栈 | Element-浅色（Vue 2.7 + element-ui 2.15.14 UMD + bi-eleme 2.4.9 + bi-element-ui 1.6.14） |
| 主题色 | #00bf8a（公司青绿，bi-eleme 编译期硬覆盖） |
| 生成日期 | 2026-06-30 |
| 包含 html | 营销目的与场景弹窗.html、一刀法-投放基础信息配置.html、一刀法-创意组配置.html、一刀法-正文标题配置.html、一刀法-投放配置.html、一刀法-公共主页配置.html |

## 真实源码参考

参考掌擎前端项目（`/Users/admin/Desktop/掌擎-frontend`）的 Facebook createV2 模块：
- 入口弹窗：`views/Facebook/popularize/createV2/components/inletDialog/index.vue`
- 全屏配置页：`views/Facebook/popularize/createV2/index.vue`（`.quickly-create-v1` 样式）
- plate 配置块：`components/plate/index.vue`（`.plate-block` 公共组件）
- 创意组：`components/configPart2/creative.vue` + `configDrawer/material/index.vue`
- 正文标题：`components/configPart2/textTitle.vue` + `configDrawer/textTitle/`
- 公共主页：`configDrawer/publicHomepage/index.vue`

**注意**：源码只取**视觉样式**，功能项以《一刀法 功能 PRD》6.2.2 为准（源码有但 PRD 没写的不画）。

## 通用视觉规格

| 项 | 值 |
|---|---|
| 主色 | #00bf8a（按钮/链接/选中态/统计数字） |
| 主文字 | #323335 |
| 次要文字 | #86909c |
| 禁用文字 | #c9cdd4 |
| 边框 | #e3e8f5（plate）/ #eaeaea（卡片）|
| 浅灰底 | #f7f8fa（plate 头部、表头底、li 矩形底）|
| 主色浅底 | #ecfff6（radio 选中）/ #f7fffb（卡片选中）|
| 全局尺寸 | $ELEMENT.size = 'small' |
| icon override | 稿子顶部 `<style>` 已加 7 行 bi-icons runtime override（select/input/table 原生图标走新形状）|
| icon scope | 删除按钮的 `el-icon-close` 实心圆✕ override 必须限定 scope（`.业务容器 .el-icon-close:before`），禁止全局覆盖（会误伤 el-drawer/dialog 原生关闭按钮）|

## 抽屉统一规格（创意组/正文标题/投放配置/公共主页）

参考 BiDrawer（`components/BiDrawer/index.vue`）：
- `el-drawer` direction="rtl" 右滑
- header：element 原生 `el-drawer__header`（padding:10px margin:0），标题 + 关闭按钮（el-icon-close 细线✕，bi-icons \e90a）
- body：padding:20px、max-height:100%、overflow-y:auto
- footer：position:absolute 底部固定，padding:10px 20px，box-shadow，取消+确定按钮
- body 底部留 padding-bottom:52px 给 footer

---

## 各 html 详细

### 1. 营销目的与场景弹窗.html
- **页面身份**：一刀法入口弹窗（点「一刀法」按钮弹出），1:1 还原 FB createV2 inletDialog
- **DOM 结构**：`el-dialog`（custom-class="facebook-creatv1-dialog"，width:605px，top:0px）> `.inlet-dialog` > `el-form`（label-width:105px）> 4 个 el-form-item
- **字段**：
  - 业务：el-radio-group（短剧 is-active / 小说 / 游戏 is-disabled）
  - 推广目的：`.tgmd-box > .gird-box` 卡片网格（grid repeat(2,220px) gap20），销量/应用推广，active-card-li 选中态，img 48px（images/JLCreatV2-jl_icon_2.png / _1.png）
  - 转化发生位置：el-select（:clearable="false"），选项动态：销量→网站+网站和应用(仅短剧)/应用推广→应用
  - 追踪：el-radio-group（网站事件/应用事件，随推广目的联动）
- **状态**：radio 选中态浅底 #ecfff6 + 主色字（源码覆盖，非 bi-eleme 默认实心底）
- **交互**：tiveChange 切推广目的联动转化位置/追踪；changeAdType 切业务联动

### 2. 一刀法-投放基础信息配置.html
- **页面身份**：一刀法全屏配置页主骨架（确定入口弹窗后进入）
- **DOM 结构**：`.quickly-create-v1` > 顶部信息栏（`.card.card-1`）+ 三行 `.preview-window`（plate-block 配置块）+ 底部固定辅助栏
- **顶部信息栏**：`.basis-info` 3 列等宽（platform/login_account/advertiser_lists，label-width:100px）+ 右上角推广目的/转化模式回显 + 右侧实时统计（系列/组/广告数）
- **6 模块 plate**（每行 3 个，flex 等宽 540px，高 350px，共享边框）：
  - 行1：广告基本信息（展开态 7 组可折叠列表）/ 目标网址 / 地区组&受众组（两个 mini-plate 嵌套，紧贴无间距）
  - 行2：创意组 / 正文标题 / 投放配置（含总体配置 + 生成结果预览 4 项）
- **plate 视觉**：边框 #e3e8f5、圆角 2px、头部 #f7f8fa 浅灰条（16px 标题 + 配置按钮）、内容区 padding:20
- **辅助栏**（底部固定）：自定义受众(disabled) / 广告状态 / Pixel / 公共主页 / 自定义回传 / 排期 / 保存为模版 / 全部重置 / 预览广告
- **辅助按钮图标色规则**：disabled→#c9cdd4 / 未配置可点→#86909c / 已配置→#00bf8a（图标+文字）
- **右侧悬浮**：`.fixed_template` 智能模板入口（128×55，right:-85px 默认藏，hover 滑出），内联 svg 图标（药丸+模板图标+「智能模板」文字）
- **不画的壳**：左导航/顶栏/Layout（这是全屏页，无外壳）

### 3. 一刀法-创意组配置.html
- **页面身份**：创意组配置抽屉（点主骨架创意组「配置」打开）
- **结构**：el-drawer（1100px）> 顶部操作行（已选统计 + 一键去重/清空所有组）+ 按书籍分组
- **书组**：`.book-group`（书名 + 折叠），每书内 视频/图片 Tab（选中主色 #00bf8a）+ 素材网格
- **素材卡片**（`.media-card` 200×200）：
  - 视频区：缩略图 + 播放图标（SVG 半透明黑圆+白三角，top:42% 居中）+ 时长（左上）+ 名称（框内底部白底 14px）
  - 图片区：缩略图 + 放大查看（el-icon-search 居中）+ 名称
  - hover 右上：复制 + 删除（24×24 圆形按钮，删除用实心圆✕ \e944 限定 scope）
  - 名称超长：el-tooltip 悬浮展示
- **缺省页**：无素材时 el-empty + 自定义 svg 插图（fill #C9CDD4）+ 「暂未选择素材，点击"添加素材"开始配置」（"添加素材"主色可点链接）
- **PRD ⑥ 不画**：源码 material/index.vue 的「格式」「规则」「进阶赋能型素材美化」PRD 没要求，已删

### 4. 一刀法-正文标题配置.html
- **页面身份**：正文标题配置抽屉
- **结构**：el-drawer > el-tabs（新增正文标题 / 正文标题库）
- **新增正文标题 Tab**：
  - 操作行：一键去重（question 图标 #86909c，hover 主色）+ 清空所有组
  - 按书籍分组（`.book-wrap`，书名+折叠+添加标题组/引用/清空）
  - 每书内标题组列表（`.group`）：head-g（组名+复制/删除）+ body-g 三段
  - 三段（正文/标题/描述）：li-head（提示+X/5 计数主色）+ li-input（textarea 批量输入，enter 添加 ctrl+enter 换行，满5禁用）+ emoji 笑脸图标
  - 保存为模板：el-switch（不是 checkbox）+ 模板名输入（500px）
  - 无标题组缺省页：el-empty + 「暂未添加正文标题，点击"添加正文和标题"开始配置」
- **正文标题库 Tab**：
  - 顶部：书籍筛选下拉 + 标题名称搜索 + 关键词搜索（各带 search append 按钮）
  - 全选 + 已选统计行（背景 #f7f8fa）
  - 树形列表（`.tree-li`）：li-label（checkbox + 展开箭头 + 组名 + 删除组 popconfirm「是否确定删除吗？」）+ li-content（展开后 正文/标题/描述 三块，每条 li-form = checkbox + 文案 + 编辑/删除图标）
  - 分页 el-pagination
- **PRD 不画**：源码的「生成正文」「添加正文和标题」AI 功能 PRD 没要求，已删

### 5. 一刀法-投放配置.html
- **页面身份**：投放配置抽屉
- **结构**：el-drawer（900px）> 总体配置 + 生成结果预览
- **总体配置**：2 个 el-input-number（一个广告系列 N 个广告组 / 一个广告组 N 个素材），label 固定宽 100px + nowrap + margin-right:12px（两个 input 左对齐）
- **生成结果预览**：按书籍区分（书组可折叠），每书 4 项 stat-grid（广告系列数量/广告组数量/每组广告数/广告总数，主色大字）
- **计算规则**：广告组数=系列数×每系列广告组数；广告总数=广告组数×每组素材数（改 input 实时重算）
- **标题**：`.section-title`（左 3px 主色竖条 + 6px 间距）

### 6. 一刀法-公共主页配置.html
- **页面身份**：公共主页配置抽屉（PRD 6.2.3）
- **结构**：el-drawer（1000px）> 操作行（同步/快速选择）+ 三栏 plate
- **三栏布局**：账户（240px）/ 公共主页 / 已选（260px），账户与公共主页无间距共享边框，公共主页与已选 20px 间距
- **账户 plate**：ad-list > ul > li > ad-item（选中态主色左边框 3px + #f7fffb 底，无四周边框，无外边距），显示账户名 + 已选主页数/未配置
- **公共主页 plate**：顶部全选 + 搜索名称或ID；主页列表 page-item（checkbox + 头像 + 名称 + ID + 可用数量徽章 + IG 下拉），可用为0 置灰禁用
- **已选 plate**：`.ul-select > li`（每个 li 是 #f7f8fa 底矩形 + 边框 + 圆角，高 38px，矩形间 10px 间距），li = left-text（主页名+账户名，超长 tooltip）+ el-icon-close（#86909c）；头部「已选(N)」+ 清空
- **交互**：勾选主页→出 IG 下拉 + 已选区加 li；快速选择一键选可用主页；单个✕删除、清空全部；不同账户独立配置

---

## 给 AI 的实现指令

前端把这份 md 喂给 Cursor/Claude Code 时，请它：

1. 先 Read 本目录所有 .html，每份对应实现一个页面/抽屉组件
2. 栈走「Element-浅色」，组件优先级：项目复杂组件库（Bi*） > antd/element 原生 > 自写
3. **严格不补 md 里没写的功能 / 占位 / 装饰**（一刀法按 PRD 6.2.2 出，源码有但 PRD 没写的功能如「格式/规则/AI生成正文」不要加）
4. 命中复用件直接 import（BiDrawer / Plate / SvgIcon / el-empty 等），不要照着 html 重画一遍空壳
5. 视觉规格以 md 为准；md 没写的项保持默认主题
6. icon override 必须 scope 限定（见通用视觉规格 icon scope 条）
7. 抽屉统一走 BiDrawer（rtl + 原生 header + 底部固定 footer + body 滚动）
8. 缺省页统一用 el-empty + 自定义 svg 插图（fill #C9CDD4）+ 主色可点链接文案
