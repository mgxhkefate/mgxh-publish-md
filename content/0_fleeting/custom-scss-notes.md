---
aliases:
tags:
  - quartz
description: 站点 custom.scss 自定义样式片段的整理笔记：逐节说明各样式的作用、根因与修法，并附带真实 SCSS 代码 (涵盖正文排版、标题、库/侧栏标题、表格、代码块限高与间距、霞鹜文楷、高亮、滚动条、阅读模式按钮、Bases 表格、Callout 间距折叠与 margin 归零等)。
created: 2026-07-31 14:09
modified: 2026-08-09 13:32
cssclasses:
noteType: experience
title: custom.scss 片段整理
---

> **源文件**：`quartz/styles/custom.scss`
> 本文档是站点自定义样式的「介绍笔记」：每一节先给标题与正文说明 (作用、根因、修法、开关方式)，再附上真实的 SCSS 外观代码，便于直接对照。

>[!info] 关于 quartz 主题的说明
>quartz 的主题主要采用 `github:saberzero1/quartz-themes` 插件实现，选用的主题是“tokyo-night”。考虑到该主题并不是完全适配 quartz [^1]，为了实现我想要的渲染模式，我通过 workbuddy 在 custom.scss 里写了许多自定义的片段，最终实现了现在站点所见的样子。尽管如此，目前还有很多渲染模式和我想象的不太一样 (主要是和我的 obsidian 渲染不太相同)。

## 目录

- [1) 正文排版](#sec1)
- [2) 明暗按钮图标](#sec2)
- [3) 库标题字体大小](#sec3)
- [4) explorer 探索标题字体大小](#sec4)
- [5) explorer 折叠标题裁切修复](#sec5)
- [6) 正文霞鹜文楷](#sec6)
- [7) 图片与 Mermaid 居中](#sec7)
- [8) 代码块限高](#sec8)
- [8.1) 代码块间距](#sec81)
- [9) 正文加粗增强](#sec9)
- [10) 正文高亮马克笔风格](#sec10)
- [11) 表格居中与圆角](#sec11)
- [12) 点状背景](#sec12)
- [13) 全站滚动条](#sec13)
- [14) 正文斜体下划线](#sec14)
- [15) 阅读模式按钮](#sec15)
- [16) Bases 单元格内边距](#sec16)
- [18) 列表缩进](#sec18)
- [19) 表格内段落行高](#sec19)
- [21) 全局图谱按钮图标](#sec21)
- [22) Bases 表格斑马纹含表头](#sec22)
- [24) 分割线 \* \* \*](#sec24)
- [25) 引用块左侧上引号](#sec25)
- [26) 首页隐藏 Properties、content-meta、recent-notes，并清空左右侧栏内容](#sec26)
- [27) Callout 内相邻块间距折叠](#sec27)
- [28) Callout 首块/折叠 margin 归零](#sec28)

<a id="sec1"></a>

## 1) 正文排版

本节仅作用于正文内容区 (`.center` 内的 `article`)，不影响左侧 explorer、库标题、目录 (TOC) 等侧边栏元素。base.scss 中正文容器为 `.center > article`，explorer 在 `.sidebar.left` 内，二者互不重叠。

这一部分首先对标题进行了调整，原有主题的标题从 h3 开始，字体就非常的小，因此我重新调整了每一级标题的字体大小。其次就是去除一些多余的上下边距，比如两个标题相邻，且之间没有任何内容时。最后是自动给该文档内最大级别的标题添加下划线，考虑到不同文档的最大级别标题不一定相同，因此做了一个选择器的筛选。

```scss
.center article {
  h1 { font-size: 2rem; }
  h2 { font-size: 1.83rem; }
  h3 { font-size: 1.67rem; }
  h4 { font-size: 1.5rem; }
  h5 { font-size: 1.33rem; }
  h6 { font-size: 1rem; }

  // 列表标记：加粗 + 数字 / 符号统一着色
  ::marker {
    font-weight: bold;
    color: #3f739a;
  }

  // ol / ul 相邻时去掉彼此间隔(包裹 div 版 + 裸相邻兜底)
  div.el-ol:has(+ div.el-ul) > ol { margin-bottom: 0 !important; }
  div.el-ol + div.el-ul > ul { margin-top: 0 !important; }
  div.el-ul:has(+ div.el-ol) > ul { margin-bottom: 0 !important; }
  div.el-ul + div.el-ol > ol { margin-top: 0 !important; }
  ol:not(li ol):has(+ ul:not(li ul)),
  ul:not(li ul):has(+ ol:not(li ol)) { margin-bottom: 0 !important; }
  ol:not(li ol) + ul:not(li ul),
  ul:not(li ul) + ol:not(li ol) { margin-top: 0 !important; }

  // 标题上 / 下间距(基础值)
  h1, h2, h3, h4, h5, h6 { margin-top: 2.5rem !important; margin-bottom: 0 !important; }
  // 相邻标题 → 彼此间距固定 0.5rem
  :is(h1, h2, h3, h4, h5, h6) + :is(h1, h2, h3, h4, h5, h6),
  div:has(> :is(h1, h2, h3, h4, h5, h6)) + div:has(> :is(h1, h2, h3, h4, h5, h6)) > :is(h1, h2, h3, h4, h5, h6) { margin-top: 0.5rem !important; }
  // 最大一级标题下划线(粗细固定 2px，颜色跟随该级标题文字色)
  :has(h2) h2,
  :not(:has(h2)):has(h3) h3,
  :not(:has(h2)):not(:has(h3)):has(h4) h4,
  :not(:has(h2)):not(:has(h3)):not(:has(h4)):has(h5) h5,
  :not(:has(h2)):not(:has(h3)):not(:has(h4)):not(:has(h5)):has(h6) h6 {
    border-bottom: 2px solid currentColor !important;
    padding-bottom: 0.15em;
  }
}
```

<a id="sec2"></a>

## 2) 明暗按钮图标

确保亮、暗两种主题下按钮图标都清晰可见。

- **根因**：quartz-themes 的 default 主题把按钮里的 `<svg>` 隐藏 (`display:none`)，改用 CSS `mask-image` + `background: var(--quartz-icon-color)` 绘制图标；主题还会把 `--dark` / `--light` 重定义为文字色 / 背景色，故不能用它们。
- **修法**：直接覆盖 `--quartz-icon-color`，给亮 / 暗主题各一个固定高对比色 (取自 Quartz 默认主题前景色：亮色 `#2b2b2b`、暗色 `#ebebec`)。
- **说明**：阅读模式按钮的颜色 / 图标已改到第 15 节 (灰色 + 开 / 关两套图标)，本节只管暗色模式 (`.darkmode`) 按钮，不再包含 `.readermode`。

>[!info] 然而“tokyo-night”主题下的暗色模式实在难看，我直接关闭了 dark-mode 插件......

```scss
.darkmode {
  --quartz-icon-color: #2b2b2b;
}
html[saved-theme="dark"] .darkmode {
  --quartz-icon-color: #ebebec;
}
```

<a id="sec3"></a>

## 3) 库标题字体大小

由于库标题是 h1 标题，这导致我在调整正文时也会影响到库标题，因此写了一个单独的片段来控制库标题字体大小。

作用于左侧栏最顶部的站点 / 库标题 (page-title 组件渲染的 `<h2 class="page-title">`，显示文字为 `mgxh-publish-md`)。Quartz 原始默认字号 1.75rem；与第 4 节的 explorer“探索”标题是两回事，互不影响。

- **修改方式**：改下方 `--library-title-size`(当前 1.5rem，为用户设定值) 即可，支持任意合法 CSS 长度，如 `1.75rem` / `18px` / `1.3em`。
- **手动开关**：给 `<body>` 加 class“library-title-off”可恢复原始字号 1.75rem。

```scss
:root {
  --library-title-size: 1.5rem; // 当前值(用户设定)；Quartz 原始默认 1.75rem
}
// 前缀 html body 仅为提高特异性；本规则位于 @layer 之外，优先级高于 tokyo-night 主题(其在 @layer obsidian-theme 内)
html body .page-title {
  font-size: var(--library-title-size) !important;
}
html body .page-title a {
  font-size: inherit;
}
body.library-title-off .page-title {
  font-size: 1.75rem !important;
}
```

<a id="sec4"></a>

## 4) explorer 探索标题字体大小

侧边栏文件树顶部的“Explorer/探索”标题。与第 3 节的库标题 (`mgxh-publish-md`) 互不影响。Quartz 原始默认字号 1rem。

- **修改方式**：改下方 `--explorer-title-size` 即可。

```scss
:root {
  --explorer-title-size: 1rem;
}
html body .explorer button.desktop-explorer h2,
html body .explorer button.mobile-explorer h2 {
  font-size: var(--explorer-title-size) !important;
}
```

<a id="sec5"></a>

## 5) explorer 折叠标题裁切修复

修复 explorer“探索”折叠时标题被裁切约一半的问题 (这应该是 tokyo-night 主题的一个 bug 吧，很莫名其妙)。

- **根因**：折叠仅由 JS 给 `.explorer` 加 `.collapsed` 类；原 CSS(explorer.scss) 用 `.explorer.collapsed { flex: 0 1 1.2rem }` 把整块侧边栏高度压到 1.2rem，再靠父级 `overflow-y:hidden` 裁掉文件树，但标题栏 (“探索”标题 h2 + 折叠箭头，实际高约 1.9rem) 也被一起裁，故标题被遮一半。且桌面端折叠时 `.explorer-content` 并未真正隐藏，只是被高度裁剪。
- **修法**：桌面端折叠态改为“标题栏完整显示 + 文件树隐藏”，不再压扁整块。
- **手动开关**：给 `<body>` 加 class“explorer-collapse-off”可恢复原始 (压扁) 行为。

```scss
@media all and (min-width: 801px) {
  .explorer.collapsed {
    flex: 0 1 auto;
    min-height: 0;
  }
  .explorer.collapsed > .explorer-content {
    display: none;
  }
  body.explorer-collapse-off .explorer.collapsed {
    flex: 0 1 1.2rem;
    min-height: 1.2rem;
  }
  body.explorer-collapse-off .explorer.collapsed > .explorer-content {
    display: block;
  }
}
```

<a id="sec6"></a>

## 6) 正文霞鹜文楷

正文 (pageBody) 使用霞鹜文楷，代码块除外 (如果想要实现正文霞鹜文楷，那么就需要先启用我写的 [[quartz-plugins-notes#2. lxgw-font(中文 Web 字体美化)]] 插件)。

- **字体**：由 `.quartz/plugins/lxgw-font` 插件通过 `<link>` 注入 (LXGW WenKai，霞鹜文楷简体)。
- **作用范围**：`.center` 内容列 (文章正文、标题、面包屑等)，即页面主体。
- **排除**：代码块 (`pre` / `code`) 保持等宽字体 `--codeFont`(IBM Plex Mono)，不被文楷覆盖；元信息 (`.content-meta`，如日期 / 阅读时长) 与 frontmatter 属性 (`.note-properties` 及其子元素) 使用站点默认字体，不套用文楷。
- **调整**：如只想改文章正文 (不含标题 / 面包屑)，可把选择器 `.center` 改为 `.center article`。

```scss
.center {
  font-family: "LXGW WenKai", var(--bodyFont), system-ui, sans-serif;
}
.center pre,
.center code,
.center pre code {
  font-family: var(--codeFont);
}
// 排除：元信息(content-meta)与 frontmatter 属性(note-properties)使用站点默认字体
.center .content-meta,
.center .note-properties,
.center .note-properties * {
  font-family: var(--bodyFont), system-ui, sans-serif;
}
```

<a id="sec7"></a>

## 7) 图片与 Mermaid 居中

正文图片、Mermaid 图表默认居中。

- **作用范围**：`.center` 内容列 (文章正文区)，不影响侧边栏 / 标题等。
- **图片**：转为块级元素，左右 auto 外边距居中，并限制最大宽度不超过容器。
- **Mermaid**：外层 `pre`(含 `code.mermaid` / `code.mermaid-selfhost`) 用 flex 居中；同时给渲染后的 `svg` 兜底居中 (Quartz 中 Mermaid 代码块渲染为 `<pre><code class="mermaid">…<svg>…</svg></code></pre>`)。
- **关闭**：直接注释掉 `.center article` 内对应的图片 / Mermaid 规则即可。

```scss
.center article {
  // 图片默认居中
  img {
    display: block;
    margin-inline: auto;
    max-width: 100%;
    height: auto;
  }
  // Mermaid 外层块：去掉代码块边框(居中交给 code.mermaid-selfhost 内的 text-align)
  pre:has(> code.mermaid-selfhost) {
    border: none;
  }
  // Mermaid 渲染后的 svg 兜底居中
  code.mermaid-selfhost,
  .mermaid {
    svg {
      margin-inline: auto;
      max-width: 100%;
      height: auto;
    }
  }
}
```

<a id="sec8"></a>

## 8) 代码块限高

代码块最大高度限制为 15 行，超出滚动。

- **作用范围**：`.center` 内容列里的代码块 (`pre`)，不影响侧边栏 / 标题等。
- **计算**：Quartz 代码行高 1.6rem(base.scss 中 `code { line-height: 1.6rem }`)，15 行 = 15 × 1.6rem = 24rem，故 `max-height` 取 24rem；超出出现纵向滚动条。
- **排除**：Mermaid 图表 (`pre > code.mermaid-selfhost`，由 mermaid-selfhost 插件改名) 不被截断，避免图表被裁切。
- **关闭**：直接注释掉下面这条 `max-height` 规则即可。

```scss
.center article pre:not(:has(> code.mermaid-selfhost)) {
  max-height: 24rem; // 15 × 1.6rem
  overflow: auto;
}
```

<a id="sec81"></a>

## 8.1) 代码块间距

代码块上下间距与正文段落保持一致（相邻块只算一个间距），并在 callout 开头取消代码块的上边距。

- **结构**：rehype-pretty-code 把代码块包进 `<figure data-rehype-pretty-code-figure>`，内层是 `<pre>`。
- **根因**：`base.scss` 里 figure 为 `margin:0`，而内层 `<pre>` 未被显式设 margin，落到浏览器 UA 默认 `margin:1em 0`。① 正文里 figure 无 padding/border，`<pre>` 的 1em 会“塌陷穿透” figure，与相邻段落折叠成一个间距——表现正常；② 但 `.callout-content` 被第 27 节改成 `display:flow-root`（建 BFC），BFC 会“阻断父子 margin 塌陷”，于是 `<pre>` 的 1em 不再穿透、变成 figure 实打实的上边距，而第 28 节 `:first-child{margin-top:0}` 只命中外层 figure（本就 0）、命中不到里面泄漏出来的 pre margin → callout 开头出现无法取消的大空白（与公式 bug 同源）。
- **修法**：给 figure wrapper 显式 `margin:1rem 0`（与正文段落 UA 1em≈1rem 一致），并清零内层 `>pre` 的 UA margin，让间距完全可控、且能被 `:first-child` 归零；相邻块仍走正常 margin 折叠（只算一个间距）。极少数未被 rehype-pretty-code 包裹的裸 `<pre>` 同步设为 `margin:1rem 0`。
- **关闭**：注释掉下方两条规则即可。

```scss
.center article figure[data-rehype-pretty-code-figure] {
  margin: 1rem 0; // 与正文段落间距一致
  > pre {
    margin: 0;    // 清掉 UA 默认 1em，避免从 figure 内泄漏、在 flow-root 里变开头空白
  }
}
// 极少数未被 rehype-pretty-code 包裹的裸 <pre>（如插件生成）同步处理
.center article > pre {
  margin: 1rem 0;
}
```

<a id="sec9"></a>

## 9) 正文加粗增强

考虑到霞鹜文楷的 bold 实在是没啥区别，所以用描边增加了粗体的区分度。正文加粗 (strong / b) 视觉增强。

- **根因**：正文霞鹜文楷 (LXGW WenKai) 经 lxgw-font 插件注入时，`@font-face` 只声明了单一常规字重 (`font-weight: 400`)，没有真实粗体文件。因此 `font-weight` 设成 700 / 800 / 9000 都无效——浏览器只能在同一个 regular 字形上做“合成加粗”(faux bold)，幅度极小，肉眼几乎看不出变化。
- **修法**：改用 `-webkit-text-stroke` 给加粗文字描边，叠加在合成粗体之上，不依赖字体字重即可让 **加粗** 明显更醒目。stroke 宽 0.4px 较稳妥，若仍嫌不够可加到 0.6px；描边取 `currentColor`，与文字同色，不会发灰 / 发白。
- **作用范围**：仅作用于正文内容列，不影响侧边栏 / 标题栏。
- **关闭**：直接注释掉下面这条规则即可。

```scss
.center article strong,
.center article b {
  font-weight: bold; // 保留合成粗体打底
  -webkit-text-stroke: 0.3px currentColor; // 描边增强视觉粗细，必须带单位
}
```

<a id="sec10"></a>

## 10) 正文高亮马克笔风格

原来正文高亮的颜色是亮黄色，而且也不是手绘的风格。因此调整了颜色和高亮的抖动效果。正文高亮 (==文字==，实际渲染为 `<span class="text-highlight">`，兼容 `<mark>`) 手绘马克笔风格。

- **颜色**：固定 `#ffcf5c`(金黄)。
- **效果**：用 104° 倾斜、起止透明、中间深浅不均的线性渐变只刷文字下半截，像荧光笔斜着刷了一道手绘划痕 (非规整矩形块)。
- **细节**：不规则圆角 + 两侧内边距轻微溢出，弱化石板感；多行高亮每行独立笔触。
- **关闭**：直接注释掉下面这段即可。

```scss
.center article mark,
.center article .text-highlight {
  color: inherit; // 文字颜色不变
  background-color: transparent; // 清掉主题默认 --textHighlight 底色
  background-image: linear-gradient(
    104deg,
    rgba(255, 207, 92, 0.4) 0%,
    rgba(255, 207, 92, 0.78) 8%,
    rgba(255, 207, 92, 0.78) 92%,
    rgba(255, 207, 92, 0.4) 100%
  );
  background-position: 0 50%;
  background-size: 100% 0.95em;
  background-repeat: no-repeat;
  border-radius: 0.25em 0.6em 0.3em 0.5em;
  padding: 0.08em 0.1em;
  -webkit-box-decoration-break: clone;
  box-decoration-break: clone; // 多行高亮每行独立笔触
}
```

<a id="sec11"></a>

## 11) 表格居中与圆角

表格：整体居中 + 圆角矩形外边框；并豁免“正文间距样式”对表格内嵌套内容的影响。

- **居中 / 圆角边框**：作用于 `.table-container > table`(Quartz 表格默认由 `.table-container` 包裹，用于横向滚动；边框画在 table 上，`border-radius` 需配合 `border-collapse: separate` 才生效，collapse 下无效)。
- **豁免间距**：第 1 / 7 / 8 节的正文标题、列表、图片、代码块间距规则，若表格单元格内嵌了 h1–h6、ol/ul、img、pre，会被这些规则命中而错位；此处在 table 作用域内把它们的 margin 等重置为紧凑值。加粗 (strong/b)、高亮 (mark/text-highlight) 按需求保留，不做豁免。
- **关闭**：不需要本节的居中 / 圆角 / 豁免，直接注释掉下方对应规则即可 (不提供 body class 开关)。圆角大小改下方 `--table-radius` 即可。

```scss
:root {
  --table-radius: 10px; // 表格圆角半径；改此值即可调整
  --table-stripe: rgba(0, 0, 0, 0.045); // 斑马纹奇数行底色(亮色模式：淡灰)
}
// 暗色模式：斑马纹改为淡白叠加
html[saved-theme="dark"] {
  --table-stripe: rgba(255, 255, 255, 0.05);
}
.center article {
  // 表格整体居中 + 圆角矩形外边框
  .table-container > table {
    margin: 1rem auto;
    border: 1px solid var(--lightgray);
    border-radius: var(--table-radius);
    border-collapse: separate;
    border-spacing: 0;
    overflow: hidden;
    padding: 0;
  }
  .table-container > table th,
  .table-container > table td {
    border-right: 1px solid var(--lightgray);
    border-bottom: 1px solid var(--lightgray);
  }
  .table-container > table th:last-child,
  .table-container > table td:last-child {
    border-right: none;
  }
  .table-container > table tbody tr:last-child td {
    border-bottom: none;
  }
  .table-container > table thead th {
    border-bottom: 1px solid var(--lightgray);
  }
  .table-container > table thead tr {
    background-color: var(--table-stripe);
  }
  .table-container > table tbody tr:nth-child(odd) {
    background-color: transparent;
  }
  .table-container > table tbody tr:nth-child(even) {
    background-color: var(--table-stripe);
  }
  // 表格内嵌套正文：重置“正文间距样式”(加粗 / 高亮保留)
  table {
    h1, h2, h3, h4, h5, h6 {
      margin-top: 0 !important;
      margin-bottom: 0.4rem !important;
      border-bottom: none !important;
      padding-bottom: 0 !important;
    }
    div.el-ol:has(+ div.el-ul) > ol,
    div.el-ol + div.el-ul > ul,
    div.el-ul:has(+ div.el-ol) > ul,
    div.el-ul + div.el-ol > ol,
    ol:not(li ol):has(+ ul:not(li ul)),
    ul:not(li ul):has(+ ol:not(li ul)),
    ol:not(li ol) + ul:not(li ul),
    ul:not(li ul) + ol:not(li ol) {
      margin-top: 0 !important;
      margin-bottom: 0 !important;
    }
    img {
      display: inline-block !important;
      margin: 0.2rem 0 !important;
      max-width: 100%;
      height: auto;
    }
    pre:not(:has(> code.mermaid-selfhost)) {
      max-height: none !important;
      overflow: visible !important;
    }
    pre:has(> code.mermaid-selfhost) {
      display: block !important;
      justify-content: initial !important;
      border: none !important;
    }
  }
}
```

<a id="sec12"></a>

## 12) 点状背景

网页点状背景 (dot grid)：在页面底色上叠加淡淡的重复圆点纹理。

- **原理**：用 `radial-gradient` 画一个 1px 圆点，`background-size` 控制点间距 (密度)。
- **说明**：只设 `background-image`，不动主题底色 (base.scss 的 body 用 `background-color` 设底色)，圆点极淡地叠在底色之上，不干扰阅读。暗色模式用浅色点、亮色模式用深色点，明暗自动适配。
- **关闭**：注释掉下方 `html body` 规则即可 (不提供 body class 开关)。
- **可调项**：圆点颜色透明度、`background-size`(越大越稀)、`background-attachment`(`fixed` 让点阵不随滚动移动；去掉则该属性恢复默认)。

```scss
html body {
  background-image: radial-gradient(rgba(0, 0, 0, 0.05) 1px, transparent 1.4px);
  background-size: 16px 16px;
  background-position: 0 0;
  background-attachment: fixed; // 点阵固定，不随内容滚动而移动
}
html[saved-theme="dark"] body {
  background-image: radial-gradient(rgba(255, 255, 255, 0.06) 1px, transparent 1.4px);
}
```

<a id="sec13"></a>

## 13) 全站滚动条

全站滚动条：只保留可拖动的细灰条 (thumb=#999)，其余成分去除。

- track / button / corner 全部透明或移除；滚动条为常驻细条，不做默认隐藏 / 悬停显形 / 拖动变粗。
- 亮色 `#999`；暗色用稍亮的 `#bbb` 保证可见。
- **关闭**：注释掉本节省略即可。

```scss
* {
  scrollbar-width: thin;              // Firefox 细条
  scrollbar-color: #999 transparent; // thumb 灰、track 透明
}
::-webkit-scrollbar {
  width: 8px;
  height: 8px;
  background: transparent;
}
::-webkit-scrollbar-track,
::-webkit-scrollbar-track-piece {
  background: transparent;
  border: none;
}
::-webkit-scrollbar-button {
  display: none;
  width: 0;
  height: 0;
}
::-webkit-scrollbar-corner {
  background: transparent;
}
::-webkit-scrollbar-thumb {
  background: #999;
  border-radius: 4px;
}
html[saved-theme="dark"] ::-webkit-scrollbar-thumb {
  background: #bbb;
}
```

<a id="sec14"></a>

## 14) 正文斜体下划线

正文斜体 (em / i) 增加下划线。

- **根因**：正文霞鹜文楷 (LXGW WenKai) 经 lxgw-font 插件注入时，`@font-face` 只声明了单一常规字重，斜体也由浏览器做“合成斜体”(faux italic，靠 skew 模拟)，与正文字形区分度有限；给斜体再加一条下划线，可让 *强调* 更易辨认。
- **修法**：给正文斜体元素 (Markdown 的 `*文字*` / `_文字_` 均渲染为 `<em>`，`<i>` 同理) 加下划线，并用 `text-underline-offset` 把线稍微下移，避免和字母降部 (g/y/p) 粘连。
- **作用范围**：`.center` 内容列 (文章正文)，不影响侧边栏 / 标题栏。
- **手动开关**：给 `<body>` 加 class“italic-underline-off”可恢复原始 (无下划线) 样式。
- **关闭**：直接注释掉下面这段即可。

```scss
.center article em,
.center article i {
  text-decoration: underline;
  text-underline-offset: 0.15em;
  text-decoration-thickness: 1px;
}
body.italic-underline-off .center article em,
body.italic-underline-off .center article i {
  text-decoration: none;
}
```

<a id="sec15"></a>

## 15) 阅读模式按钮

阅读模式按钮：开 / 关两套图标 + 灰色 (不再近黑)。

- **根因**：quartz-themes 用 `button.readermode { background: var(--quartz-icon-color); mask-image: var(--readermode-icon) }` 把按钮画成单色图标。原 `--readermode-icon` 是固定的一本“打开的书”(Lucide book-open，描边风)，且 `--quartz-icon-color` 在亮色模式为 `#2b2b2b`(近黑)，故按钮一直是一本书、且是黑色。
- **修法**：① 关 (reader-mode=off)→ 打开的书 book-open；开 (reader-mode=on)→ 带勾的书 book-open-check，两套图标随 reader-mode 属性切换 (与暗色模式 sun / moon 同源机制)。② 图标底色改为灰色 `var(--gray)`(亮 `#b8b8b8` / 暗 `#646464`)，不再近黑。
- **说明**：图标用 Lucide 描边风 SVG，作 mask 时描边即不透明区域；本规则写在 custom.scss(无 @layer)，优先级高于主题的 `@layer obsidian-theme`，可直接覆盖 `--readermode-icon` / `--quartz-icon-color`。
- **关闭**：注释掉本节省略 (恢复主题默认：单图标 + 近黑)；不提供 body class 开关。

```scss
.readermode {
  --quartz-icon-color: var(--gray);                            // 图标底色改灰色
  --readermode-icon: url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIyNCIgaGVpZ2h0PSIyNCIgdmlld0JveD0iMCAwIDI0IDI0IiBmaWxsPSJub25lIiBzdHJva2U9ImJsYWNrIiBzdHJva2Utd2lkdGg9IjEuNSIgc3Ryb2tlLWxpbmVjYXA9InJvdW5kIiBzdHJva2UtbGluZWpvaW5ibGluZSI+PHBhdGggZD0iTTEyIDV2MTYiLz48cGF0aCBkPSJNMjAuMDAxIDE5QTIgMiAwIDAwMjIgMTdWNWEyIDIgMCAwMC0xLjk5OS0yTDE2IDMuMDAyQTUgNSAwIDAwMTIgNWE1IDUgMCAwMC00LTJoNGEyIDIgMCAwMC0yIDJ2MTJhMiAyIDAgMDAxLjk5OSAySDhhNSA1IDAgMDE0IDIgNSA1IDAgMDE0LTJ6Ii8+PC9zdmc+');   // 关：打开的书
}
:root[reader-mode="on"] .readermode {
  --readermode-icon: url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIyNCIgaGVpZ2h0PSIyNCIgdmlld0JveD0iMCAwIDI0IDI0IiBmaWxsPSJub25lIiBzdHJva2U9ImJsYWNrIiBzdHJva2Utd2lkdGg9IjEuNSIgc3Ryb2tlLWxpbmVjYXA9InJvdW5kIiBzdHJva2UtbGluZWpvaW5ibGluZSI+PHBhdGggZD0iTTEyIDV2MTYiLz48cGF0aCBkPSJtMTYgMTIgMiAyIDQtNCIvPjxwYXRoIGQ9Ik0yMiA2VjVhMiAyIDAgMDAtMS45OTktMkwxNiAzLjAwMkE1IDUgMCAwMDEyIDVhNSA1IDAgMDA0LTJoNEEyIDIgMCAwMC0yIDJ2MTJhMiAyIDAgMDAxLjk5OSAySDhhNSA1IDAgMDE0IDIgNSA1IDAgMDE0LTJoNC4wMDFBMiAyIDAgMDAyMiAxN3YtMS4zNDQiLz48L3N2Zz4=');   // 开：带勾的书
}
```

<a id="sec16"></a>

## 16) Bases 单元格内边距

压缩 Bases 表格单元格的上下内边距。

- **根因**：bases-page 插件 (quartz-community/bases-page) 的组件样式 `.bases-table th, .bases-table td { padding: 10px 12px }` 把单元格上下留白设成 10px，行高偏大、表格整体显得松散。
- **修法**：把单元格上下内边距从 10px 收到 4px(左右 12px 保持不变)，让表格更紧凑。选择器 `.bases-page .bases-table` 比插件原规则多包了一层类，优先级更高，无论样式加载顺序如何都能稳定覆盖 (实测插件规则在 component-*.css、custom.scss 在 resource-style-*.css 且后加载)。
- **手动开关**：给 `<body>` 加 class“bases-cell-tight-off”可恢复插件默认的 10px。
- **关闭**：直接注释掉下面这段即可。

```scss
.bases-page .bases-table {
  th,
  td {
    padding-top: 4px;
    padding-bottom: 4px;
  }
}
body.bases-cell-tight-off .bases-page .bases-table th,
body.bases-cell-tight-off .bases-page .bases-table td {
  padding-top: 10px;
  padding-bottom: 10px;
}
```

<a id="sec18"></a>

## 18) 列表缩进

列表缩进 (悬挂式：bullet 落在正文左缘，文字内缩)。

- **根因**：Quartz 的 base.scss 只给 ul/ol/li 设了文字颜色，没有重置 `padding-left`，列表沿用浏览器默认的约 40px(≈2.5rem) 左缩进，显得过宽。
- **机制**：`padding-inline-start` 作用在 ul/ol 容器上，是相对其包含块的左内边距；ul/ol 与正文 p 是 `.center article` 下的同级块，左边缘在同一条线上，所以这个值 = 列表文字相对正文文字的右移量 (`list-style-position` 默认 outside，bullet 绘制在 padding 左部、约等于正文左缘)。即它不是“和正文平齐”，而是“悬挂缩进”：bullet 对齐正文、文字内缩此值。
- **取值**：一级 1.5rem 为实测与正文视觉平齐 (bullet 贴正文左缘) 的值；嵌套层级每级均匀再右移 1.2rem。如需列表文字与正文严格左对齐，改 `list-style-position: inside` 或 `padding-inline-start:0`。
- **作用范围**：`.center article`(文章正文)，与第 3~7 节列表规则一致。

```scss
.center article ul,
.center article ol {
  padding-inline-start: 2rem;          // 一级列表：bullet 贴正文左缘，文字内缩 1.5rem
}
.center article li > ul,
.center article li > ol {
  padding-inline-start: 1.2rem;          // 嵌套层级：每级再右移 1.2rem
}
// 任务列表(checkbox)整列左对齐，不额外缩进
.center article ul:has(li > input[type="checkbox"]) {
  padding-inline-start: 0;
}
```

<a id="sec19"></a>

## 19) 表格内段落行高

表格内段落行高调整，同时作用于 base-table 和普通 table。

`.table-container>table>*` 设 `line-height: 1.5rem`；`.bases-view-meta` 设上下 margin 收紧。

```scss
.table-container>table>* {
  line-height: 1.5rem;
}
.bases-view-meta {
  margin-top: 0.5rem;
  margin-bottom: 0rem;
}
```

<a id="sec21"></a>

## 21) 全局图谱按钮图标

恢复文章底部图谱右上角“查看全局图谱”按钮 (global-graph-icon) 的图标显示。

- **根因**：tokyo-night 主题把该按钮的原始 `<svg>` 设为 `display:none`，改用 `mask-image + background` 重绘图标。但这个 mask 机制在部分环境下不能正常显示 (mask 渲染失败会让按钮整块透明；且背景色变量链 `--quartz-icon-color → --icon-color → --darkgray` 在暗色下存在自引用循环，解析结果不可见)，导致按钮“消失”。
- **修法**：放弃 mask 方案，恢复 Quartz 默认的原始 `<svg>` 图标渲染 (`svg` 自带 `fill="currentColor"`，沿用按钮 `color` 上色，明暗自适应)，并清除主题的 `mask-image` 与背景填充约束。已用真实浏览器 (Playwright + Chromium) 渲染验证：亮色、暗色下按钮均 `VISIBLE: true`、`mask-image: none`、`svg display: inline-block`。
- **手动开关**：给 `<body>` 加 class“graph-icon-off”可再次隐藏该按钮。

```scss
button.global-graph-icon {
  background: transparent !important;
  -webkit-mask-image: none !important;
  mask-image: none !important;
}

button.global-graph-icon > svg {
  display: inline-block !important;
  width: 1.25rem !important;
  height: 1.25rem !important;
}

body.graph-icon-off button.global-graph-icon {
  display: none !important;
}
```

<a id="sec22"></a>

## 22) Bases 表格斑马纹含表头

Bases 表格的斑马纹从表头开始 (表头即第一道暗纹)，与普通 Markdown 表格 (第 11 节) 视觉一致。

- **根因**：bases-page 的 .bases-table 表头单独设为 background: var(--light)，不参与斑马纹，与普通表格 (表头即第一道暗纹) 不一致。
- **修法**：表头与 tbody 偶数行都复用 --table-stripe(与普通表格暗纹同一颜色、不另取更深的色)，tbody 奇数行透明继承页面背景，形成“表头暗 - 行 1 亮 - 行 2 暗……”的规律，明暗主题自动适配。
- **手动开关**：给 `<body>` 加 class“bases-zebra-off”恢复插件默认 (亮表头、无斑马纹)。
- **关闭**：直接注释掉下面这段即可。

```scss
.bases-page .bases-table {
  thead th {
    background: var(--table-stripe);
  }
  tbody tr:nth-child(odd) {
    background-color: transparent;
  }
  tbody tr:nth-child(even) {
    background-color: var(--table-stripe);
  }
}
body.bases-zebra-off .bases-page .bases-table {
  thead th {
    background: var(--light);
  }
  tbody tr:nth-child(odd),
  tbody tr:nth-child(even) {
    background-color: transparent;
  }
}
```

<a id="sec24"></a>

## 24) 分割线 * * *

Obsidian 的 `---` 在 Quartz 中渲染为 `<hr>`，默认是整条灰色横线 (`quartz/styles/base.scss` 给 `hr` 设了 `border` + `background-color: var(--lightgray)` 的灰色矩形)。改为居中的黑色加粗「* * *」分隔符。

- **根因**：`quartz/styles/base.scss` 的 `hr` 默认样式是灰色横线 (含 `--lightgray` 背景矩形)。
- **修法**：① `.center article hr` 清除 `border` 并设 `background: transparent`(去掉灰色矩形与横线)；② `::before` 伪元素输出居中、黑色 (`#000`)、加粗的「* * *」；③ `letter-spacing: 0.25em` 让三颗星更舒展，`text-indent: 0.25em` 等量补偿其末尾空白导致的整体偏左。
- **暗色模式**：纯黑在深色背景不可见，故暗色下改用浅色文字 (`var(--light)`)。
- **手动开关**：给 `<body>` 加 class「hr-stars-off」恢复原始横线。

```scss
.center article hr {
  border: none;             // 清除默认横线
  background: transparent;  // 去除 base.scss 的灰色背景(--lightgray 整条矩形)
  height: auto;
  overflow: visible;
  text-align: center;       // 「* * *」整体居中
  margin-top: 2rem;
  margin-bottom: 1.5rem;
}

.center article hr::before {
  content: "* * *";      // 居中的「* * *」分隔符
  display: block;
  color: #000;           // 黑色
  font-weight: bold;     // 加粗
  line-height: 1rem
  font-size: 1.2rem;
  letter-spacing: 0.25em;
  text-indent: 0.25em;   // 抵消 letter-spacing 末尾空白导致的整体偏左
}

// 暗色模式兜底：黑色不可见，改用浅色文字
html[saved-theme="dark"] .center article hr::before {
  color: var(--light);
}

// 手动开关：恢复原始横线
body.hr-stars-off .center article hr {
  border-top: 1px solid var(--lightgray);
  height: 0;
  margin: 2rem 0;
}
body.hr-stars-off .center article hr::before {
  content: none;
}
```

<a id="sec25"></a>

## 25) 引用块左侧上引号

去掉 Obsidian 引用块 (`>` 渲染为 `<blockquote>`) 默认的左侧竖线，改为在正文块**左侧**、与**第一行文字平齐**放一个装饰性左双引号「"」(U+201C)。

- **根因**：`quartz/styles/base.scss` 给 `blockquote` 设了 `border-left: 3px solid var(--secondary)`(左侧竖线)+ `padding-left: 1rem`，是传统“左边一条线”引用样式。
- **修法**：① 去 `border-left` 清掉竖线；② `::before` 绝对定位在 `left:0`、`top: 0.3rem`(与第一行文字视觉平齐)，放一个 `font-size: 3rem`、Georgia 衬线字体的左双引号；③ 颜色用淡灰 (`gray`) + `opacity: 0.6` 做水印感，不抢内容；④ 整个 `blockquote` 去掉上下 `padding`/`margin`，仅保留左侧 `2.2rem` 给引号留位；⑤ 不加斜体 (中文斜体不自然)。
- **对齐要点**：引号用收紧的 `line-height` + `top: 0.3rem`，使其字形与正文第一行文字视觉平齐 (而非与段落 box 顶边对齐)。
- **⚠️ 排除 callout**：Quartz 把 Obsidian 的 callout 也渲染成 `<blockquote class="callout">`，若不排除会被误加大引号、且被清掉上下间距/左缩进。因此本节省略所有选择器都加 `:not(.callout)`，仅作用于「普通引用块」。
- **手动开关**：给 `<body>` 加 class「quote-mark-off」恢复原始竖线样式 (同样排除 callout)。

```scss
.center article blockquote:not(.callout) {
  position: relative;        // 给引号做定位基准
  border-left: none;         // 去掉左侧竖线
  margin: 0;                 // 无上下 margin
  padding: 0 1rem 0 2.2rem;  // 无上下 padding，左侧留出引号的空间
  color: inherit;
}

.center article blockquote:not(.callout)::before {
  content: "\201C";          // 左双引号 "
  position: absolute;
  left: 0;                   // 贴左边
  top: 0.3rem;               // 与第一行文字视觉平齐
  font-size: 3rem;           // 比正文大但不夸张
  line-height: 0.8;          // 收紧引号行高，使字形与正文首行视觉平齐
  font-weight: 700;
  color: gray;               // 淡灰装饰
  opacity: 0.6;
  font-family: Georgia, "Times New Roman", serif;
  pointer-events: none;
  user-select: none;
}

// 手动开关：恢复原始竖线样式(同样排除 callout)
body.quote-mark-off .center article blockquote:not(.callout) {
  border-left: 3px solid var(--secondary);
  padding: 1rem 1rem 1rem 1rem;
}
body.quote-mark-off .center article blockquote:not(.callout)::before {
  content: none;
}
```

<a id="sec26"></a>

## 26) 首页隐藏 Properties、content-meta、recent-notes，并清空左右侧栏内容

仅在**首页**(index 页) 隐藏「Properties」折叠块 (`.note-properties`)、元信息行 (`.content-meta`，即日期 / 标签那一行)、「最近笔记」区块 (`.recent-notes`，含其内部的 `.recent-ul` / `.recent-li` 列表)；同时把**左右两侧栏内部所有内容清空**，但**保留两侧栏容器本身**（做成干净的落地页）。

- **根因**：首页 `<body>` 带 `data-slug="index"`，该页不想展示自己的 frontmatter 属性表、元信息行、最近笔记列表；也不想显示左侧栏里的文件树（explorer）/ 站点标题 / 搜索框 / 阅读模式，以及右侧栏里的**关系图谱**（`.graph`，已确认在首页中它就位于 `.right.sidebar` 内，含 `button.global-graph-icon` 按钮）。
- **修法**：用 `body[data-slug="index"]` 把作用域严格限定在首页，只隐藏前三项；其他任意页面**(含子目录索引页，其 `data-slug` 形如 `子目录/index`) 均不受影响**。左右侧栏均**不是整栏 `display:none`**，而是用 `.sidebar.left > *` / `.sidebar.right > *` 隐藏其内部所有子元素——这样两侧栏容器仍占着布局列（页面不会因少栏而扩展、正文区位置与其余页面保持一致），但栏内变成空白。关系图谱随右侧栏清空一并隐藏，不必再单独写 `.graph` 规则。
- **注意**：`data-slug="index"` 仅精确匹配首页；若想连子目录索引页也一并处理，需改用更宽的选择器 (如 `body[data-slug$="/index"]`)。如果反而想“整栏直接消失、正文区扩展占满”，把 `.sidebar.left > *` / `.sidebar.right > *` 改成 `.sidebar.left` / `.sidebar.right` 即可。
- **关闭**：注释掉本节省略即可。

```scss
body[data-slug="index"] {
  .note-properties,
  .content-meta,
  .recent-notes {
    display: none;
  }
  // 左侧栏：保留栏本身（布局列不变），仅清空其内部所有内容
  .sidebar.left > * {
    display: none;
  }
  // 右侧栏：同理；首页右栏仅含关系图谱（.graph），一并清空
  .sidebar.right > * {
    display: none;
  }
}
```

<a id="sec27"></a>

## 27) Callout 内相邻块间距折叠

修复 callout 内“段落与列表相邻”时间距被叠加 (翻倍) 的问题，使其与正文 (普通文档流) 的 margin 折叠行为一致；同时为第 28 节的「首块 margin-top 归零」建立 BFC 基础。

- **根因**：`callouts.scss` 给 `.callout-content` 设了 `display: grid`。关键点是——**grid/flex 容器的子项不会发生 margin 折叠 (margin collapsing)**：因此 callout 里“段落后接列表”会算成 `p.margin-bottom + ul.margin-top` 相加，间距翻倍；而正文 `.center article` 是普通文档流，相邻块级元素的 margin 会折叠成较大者 (正常行为)。同一份 Markdown 在正文里间距正常、放进 callout 却异常变宽，就是这个差异。
- **修法**：把 `.callout-content` 改为 `display: flow-root`（普通流 + 建立 BFC）。相对 `block` 的两点考量：① 仍让内部块级元素走 margin 折叠，与正文一致；② 建立 BFC 能**阻断第一个块的 margin-top 塌陷穿透 content 顶部、叠加 `.callout-title` 的 padding**（若不建 BFC，第 28 节归零首块上边距后仍会有塌陷外溢的残留空白）。callout 折叠动画靠子元素 `height:0` 实现，与 `display` 无关，因此不受影响。
- **手动开关**：给 `<body>` 加 class“callout-collapse-off”可恢复主题的 grid 行为 (即保留原 bug)。

```scss
.center article .callout .callout-content {
  display: flow-root;
}

body.callout-collapse-off .center article .callout .callout-content {
  display: grid;
}

<a id="sec28"></a>

## 28) Callout 内首个块的 margin-top 归零（仅首块、绝不碰下边距）+ 折叠态上下 margin 归零

- **callout 真实结构**：`<blockquote class="callout">` > `<div class="callout-content">` > 块（公式 / 代码 / 列表 / 段落直接是子节点）。
- **背景**：`callouts.scss` 的「`.callout-content > :first-child { margin-top:0 }`」与 MathJax 注入的「`mjx-container[display="true"] { margin:1em 0 }`」特异性同为 (0,2,0)，后者后加载 → 胜出，首个块的 margin-top 未被归零；在 flow-root（第 27 节）下该 margin-top 塌陷外溢、叠加 `.callout-title` 的 `padding-bottom:1rem` → 公式 / 代码块与标题间出现大空白（代码块还会因内层 pre 泄漏而放大）。
- **修法（仅首块）**：用更高特异性（`.center article` 前缀 → (0,5,0)）只对「callout 内首个块」归零 `margin-top`。公式直接是 `mjx-container`（首子节点）命中 `:first-child` 即归零；代码块是 `figure[...]` 同理；若公式被包裹则 `:first-child mjx-container` 兜底。
- **⚠ 关键：只归零 margin-top，不碰 margin-bottom**。否则当 callout 仅含一个块（单个列表 / 单个段落 / 单个公式）时，该块同时是首尾，`margin-bottom` 会被误杀 → “单个块下边距消失”的 bug。`.callout-content` 是 flow-root（BFC），会把子块 margin 包含在自身高度内，因此单个块的下边距能正常显示。
- **折叠态（is-collapsed）**：内容 `height:0` 收起，但 margin 不随 `height` 消失、仍撑出约 2em 空白（`overflow:clip` 只裁内容不裁 margin）。故折叠态需把**全部子块**的上下 margin 一起归零（独立规则，不影响普通 callout）。
- **说明**：不依赖 `!important`，仅凭特异性 (0,5,0) > (0,2,0) 压过 MathJax，与加载顺序无关。仅首个块受影响；callout 内后续块、正文、非首个公式 / 代码块的上边距均保持不变。

```scss
// 仅首个块：归零上边距（不动下边距）
.center article .callout .callout-content > :first-child,
.center article .callout .callout-content > :first-child mjx-container[display="true"] {
  margin-top: 0;
}
// 折叠态：全部子块上下 margin 归零
.center article .callout.is-collapsed .callout-content > *,
.center article .callout.is-collapsed .callout-content > * mjx-container[display="true"],
.center article .callout.is-collapsed .callout-content > * figure[data-rehype-pretty-code-figure] {
  margin-top: 0;
  margin-bottom: 0;
}
```
```

[^1]: [saberzero1/quartz-themes: Obsidian 🤝 Quartz. Quartz-compatible Obsidian themes.](https://github.com/saberzero1/quartz-themes)