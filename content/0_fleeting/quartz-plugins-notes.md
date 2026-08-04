---
noteType: experience
aliases:
tags:
  - quartz
description: 本站五个自研 Quartz 插件的介绍：no-referrer-images、lxgw-font、image-zoom、mermaid-selfhost、markdown-image-size，逐一说明解决的问题并附上 GitHub 仓库地址 (不贴源码)。
created: 2026-07-31 14:46
modified: 2026-08-04 21:17
cssclasses:
title: Quartz 自定义插件介绍
---

本站基于 Quartz v5 构建，目前使用了五个自行开发的插件，均已发布到 GitHub 公开仓库，通过 git 源直接引用 (无需在本机建立符号链接)。下面逐一说明每个插件解决了什么问题，并附上仓库地址。

### 1. no-referrer-images(外链图片防盗链绕过)

**解决的问题**

- Quartz 默认渲染的外链图片 (例如 CSDN、各类图床) 在浏览器请求时会带上当前页面的 referrer 信息；很多图床据此开启防盗链，导致图片加载失败、页面出现裂图。该插件会在构建阶段给所有外链 `<img>` 自动加上 `referrerpolicy="no-referrer"`，使请求不带来源信息，从而绕过防盗链，图片正常显示。

**仓库地址**

- https://github.com/mgxhkefate/no-referrer-images

### 2. lxgw-font(中文 Web 字体美化)

**解决的问题**

- Quartz 默认字体对中文的渲染观感一般，且依赖访客系统已安装的中文字体。该插件注入“霞鹜文楷”(LXGW) 中文 Web 字体，让所有页面统一使用这套字体显示中文，提升阅读体验，且访客无需在本地安装任何字体。

**仓库地址**

- https://github.com/mgxhkefate/lxgw-font

### 3. image-zoom(图片灯箱放大)

**解决的问题**

- Quartz 正文里的图片默认只能以原始大小查看，点击无法放大。该插件为所有正文图片加上灯箱 (lightbox) 效果：点击图片即打开全屏视图，支持鼠标滚轮缩放 (约 0.2 倍到 8 倍)、拖拽平移、双击复位、按 Esc 或点击空白处关闭。配合 no-referrer-images 插件，即便是 CSDN 等外链图片也能点开正常放大。

**仓库地址**

- https://github.com/mgxhkefate/image-zoom

### 4. mermaid-selfhost(Mermaid 流程图自托管渲染)

**解决的问题**

- Quartz 内置的 Mermaid 流程图通过公共 CDN(cdnjs) 加载脚本，在国内网络环境下常被拦截，且旧版浏览器不支持其依赖的 import map 特性，导致流程图渲染失败、整块空白。该插件改为自托管 Mermaid 打包文件，直接从站点内加载并渲染，不依赖外部 CDN，也能兼容较旧的浏览器。

**仓库地址**

- https://github.com/mgxhkefate/mermaid-selfhost

### 5. markdown-image-size(标准 Markdown 图片的 Obsidian 尺寸语法)

**解决的问题**

- 在 Obsidian 里可以用 `![alt|100](url)` 这种语法给图片指定宽度，但 Quartz 自带的 obsidian-flavored-markdown 插件只解析 Obsidian 专属的 `![[image|100]]` wikilink 嵌入语法，对这种标准 Markdown 图片完全不处理。结果 `|100` 被原样塞进 img 的 alt 属性，产物里既没有 width、alt 还带着 `|100`。
- 该插件在构建阶段遍历标准 Markdown 图片节点，把 alt 末尾的 `|宽[x高]` 提取出来写成 img 的 `width` / `height` 属性，并从 alt 里清掉 `|100`，使网页端尺寸与 Obsidian 显示一致。只在 alt 以 `|数字` 结尾时触发，不会误伤中间含 `|` 的普通文本；`![[...]]` wikilink 嵌入由自带插件处理，也不会冲突。
- 已发布到 GitHub 并通过 git 源接入；`quartz.config.yaml` 中 `order: 36`。

**仓库地址**

- https://github.com/mgxhkefate/markdown-image-size

---

### 附：graph 插件(社区插件)的本地修复

本站的关系图谱使用的是社区插件 `github:quartz-community/graph`(非自研，故未列入上面五个)。该插件在「中文文件名的当前页节点」上有一个编码显示 bug，我已打补丁修复；详细的根因、补丁位置与重放步骤见 [[quartz-pull-overrides-reminder#5. graph 插件中文节点 %xx% 修复]]。

- **现象**：打开一个中文文件名的笔记，其关系图谱里「当前页」那个节点显示成 `%xx%` 编码串(如 `0_fleeting/%E5%A5%A5…`)，而非中文；其余节点正常。
- **根因**：图谱取「当前页 slug」(`window.location.pathname` / SPA 导航事件 `e.detail.url`)在你的环境里是 URL 编码态，而图谱数据 `contentIndex.json` 的 key / `title` 是未编码中文，两者匹配不上 → `title` 取不到 → 回退显示编码 slug。
- **修法**：在插件实际加载的 `dist` 产物(`.quartz/plugins/graph/dist/components/index.js` 与 `dist/index.js`)里，对所有 slug 统一加 `decodeURIComponent`，共五处(当前页 slug、数据 key、链接目标、标签 slug、文本回退值)；`src/components/scripts/graph.inline.ts` 也同步加了 `decodeSlug` 安全封装(build 不读 src，仅供参考)。
- **持久性提醒**：补丁打在 `dist/` 里，`dist` 不进 git，`npx quartz build --upgrade` 或插件被重新拉取时会被覆盖、问题复现。彻底方案：fork 到 `mgxhkefate/graph` 并提交打好补丁的 `dist/`，把 `quartz.config.yaml` 的 `source` 改成 `git+https://github.com/mgxhkefate/graph.git`(沿用 `markdown-image-size` 的同款模式)。