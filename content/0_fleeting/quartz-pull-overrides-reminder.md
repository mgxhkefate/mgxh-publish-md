---
cssclasses: 
aliases:
tags:
  - quartz
description: 从 Quartz 官方仓库 pull / 合并更新后，哪些「直接改在框架或插件文件里」的定制会被上游覆盖、需要重新调整，以及每一步的精确命令与坑点。
created: 2026-07-31 21:22
modified: 2026-07-31 21:24
noteType: experience
title: Quartz 官方 pull 后需重调的定制项
---

> **用途**：我从 Quartz 官方仓库 `git pull` / 合并更新后，有若干「直接改在框架文件或外部插件文件里」的定制会被上游覆盖。本文逐一列出，并给出重调步骤，避免每次 pull 后站点悄悄变回默认样。
> **结论先行**：我的所有样式都写在 `quartz/styles/custom.scss`(用户自己的覆盖层文件，pull 通常不会动它)；会被覆盖的是「改在框架 / 插件目录里的那两处」+「config 里的插件条目」。

## 总览 (按风险排序)

| 项 | 文件 | 是否会被 pull 覆盖 | 风险 |
|---|---|---|---|
| 1. 站点图标 favicon | `quartz/static/icon.png` | **是**(框架自带默认图标) | 高 |
| 2. 阅读模式默认开启 | 已**固化**（config 指向自建 fork `mgxhkefate/reader-mode`，默认开） | 否（除非手动同步上游） | 低 |
| 3. quartz.config.yaml 插件条目 | `quartz.config.yaml` | 合并冲突时**可能丢** | 中 |
| 4. custom.scss 覆盖层 | `quartz/styles/custom.scss` | 否 (用户文件) | 低，但需复核 |

---

## 1. 站点图标 favicon(高，必丢)

- **改了什么**：用 `D:\XL\01-图片\favicon.ico` 转成的 256×256 PNG，覆盖了框架默认的 `quartz/static/icon.png`(默认 17KB 灰图标 → 我的 130KB 图标)。`Head.tsx` 固定引用 `static/icon.png`，所以这是换 favicon 的唯一标准位置。
- **为什么会被覆盖**：`quartz/static/icon.png` 是框架追踪的文件，`git pull` 会把它还原成官方默认图标 (注意：根目录 `static/icon.png` 无效——`Static` 拷贝器最后跑，会用 `quartz/static/` 这份覆盖根目录那份)。
- **重调步骤**：

```bash
# 1) 用托管 venv 的 python(已装 Pillide+PIL，不污染系统)
PY="C:/Users/admin/.workbuddy/binaries/python/envs/default/Scripts/python.exe"

# 2) 把 ico 转成 256x256 的 PNG，直接覆盖框架图标
"$PY" - <<'PYEOF'
from PIL import Image
src = r"D:\XL\01-图片\favicon.ico"
out = r"D:\XL\quartz\quartz\static\icon.png"
with Image.open(src) as im:
    frames = []
    try:
        while True:
            frames.append(im.convert("RGBA").copy())
            im.seek(im.tell() + 1)
    except EOFError:
        pass
    if not frames:
        frames = [im.convert("RGBA")]
    frames.sort(key=lambda f: f.size[0] * f.size[1], reverse=True)  # 取最大帧
    best = frames[0]
    w, h = best.size
    if max(w, h) > 256:
        scale = 256 / max(w, h)
        best = best.resize((int(w * scale), int(h * scale)), Image.LANCZOS)
    best.save(out, "PNG")
    print("saved", out, best.size)
PYEOF

# 3) 重新构建
cd D:/XL/quartz && npx quartz build
```

- **验证**：构建后 `public/static/icon.png` 大小应恢复成你的图标 (~130KB，而非官方 17KB)。
- **注意**：`D:\XL\01-图片\favicon.ico` 是原始素材，一定要留着；以后图标变了直接改这个 ico 重新转。

---

## 2. 阅读模式默认开启（已固化，低风险）

- **当前状态（2026-07-31 固化）**：阅读模式默认开启已通过**自建 fork `mgxhkefate/reader-mode` 固化**——该 fork 的初始状态为「开启」，且 `quartz.config.yaml` 已指向 `git+https://github.com/mgxhkefate/reader-mode.git`。常规 pull 官方 Quartz 框架不会再把它打回关闭。
- **为什么会被覆盖**：这个插件是独立 git 仓库 (`.quartz/plugins/reader-mode/.git`)，`npx quartz sync` 或插件更新时会把 `dist/` 重新拉回默认 (默认 `false` = 关闭)。源码里的 `let isReaderMode = false` 也会被重置。
- **手动重调步骤（仅当你主动 `git fetch upstream` 同步官方 reader-mode 更新后才需要）**：改三处 (关键：**Quartz 实际打包的是 `dist/components/index.js` 那份 inline script**，只改 `dist/index.js` 没用——这是之前踩过的坑)。

```bash
PLUGIN=D:/XL/quartz/.quartz/plugins/reader-mode
```

  - **(a) 必须改 — `dist/components/index.js`(约第 314 行)**
    找到这一行 (inline 脚本字符串)：

    ```js
    var readermode_inline_default = 'var n=!1,o=t=>{ ... }';
    ```

    把其中的 `var n=!1` 改为 `var n=!0`(即 `isReaderMode = true`)。

    完整替换示例：

    ```js
    var readermode_inline_default = 'var n=!0,o=t=>{let e=new CustomEvent("readermodechange",{detail:{mode:t}});document.dispatchEvent(e)},d=()=>{let t=()=>{n=!n;let e=n?"on":"off";document.documentElement.setAttribute("reader-mode",e),o(e)};for(let e of document.getElementsByClassName("readermode"))e.addEventListener("click",t),window.addCleanup(()=>e.removeEventListener("click",t));document.documentElement.setAttribute("reader-mode",n?"on":"off")};document.addEventListener("nav",d);document.addEventListener("render",d);\n';
    ```

  - **(b) 平行副本 — `dist/index.js`(约第 314 行)**
    同上，把 `var n=!1` 改为 `var n=!0`，保持与 (a) 一致 (避免以后某次构建命中这份)。

  - **(c) 源码 — `src/components/scripts/readermode.inline.ts`**

    ```ts
    let isReaderMode = false;   // → 改为 true
    ```

    改这个是为了万一将来重新编译插件时一致 (直接编辑 dist 不会被 ts 重新编，但留着以防万一)。

```bash
# 改完重新构建
cd D:/XL/quartz && npx quartz build
```

- **验证**：构建后 `public/prescript-*.js` 里应出现 `var t=!0`(默认开启)；浏览器打开站点，进站即隐藏侧栏，点右上角按钮可退出。

---

## 3. quartz.config.yaml 的插件条目 (中，合并时可能丢)

- **为什么要注意**：`quartz.config.yaml` 是我的配置文件。从官方 pull 时若官方也改了这个文件，Git 会报合并冲突，处理不当会把我加的自定义插件丢掉，站点直接构建失败或功能缺失。
- **重调步骤**：pull / 合并后，打开 `quartz.config.yaml`，确认以下条目**都还在**：
  - 5 个自研插件 (在 `plugins` 段，以 `source: "./custom-plugins/<name>"` 或 git URL 引用)：`no-referrer-images`、`lxgw-font`、`image-zoom`、`mermaid-selfhost`、`markdown-image-size`。
  - 阅读模式插件：`git+https://github.com/mgxhkefate/reader-mode.git`(在 `plugins` 段，自建 fork、默认开)。
  - 这些插件的 `component` 注册 (如 `reader-mode`、`lxgw-font` 等) 也需在对应组件段保留。
- **注意**：自研插件在 `custom-plugins/` 或独立 git 仓库里 (被主仓库 `.gitignore` 忽略)，pull Quartz 本身不会动它们；但这里只检查「config 是否还引用它们」。

---

## 4. custom.scss 覆盖层 (低，但需复核)

- **为什么通常安全**：`quartz/styles/custom.scss` 是我自己的覆盖层文件，不在 Quartz 官方追踪范围内，pull 一般不会覆盖它。所有样式 (分割线 `* * *`、引用块引号、表格、Bases、首页隐藏 Properties 等) 都在这里。
- **但有一个隐患**：custom.scss 里的规则是通过**覆盖框架 `base.scss` 等的选择器**来生效的 (例如清掉 `hr` 的 `background`、清掉 `blockquote` 的 `border-left`)。如果未来某个 Quartz 版本**重命名 / 移动了这些底层选择器**，覆盖会**静默失效**(不会报错，只是样式回退到默认)。
- **重调步骤 (pull 后肉眼复核清单)**：
  1. 刷新站点，确认 favicon 仍是自己的图标 (见第 1 项)。
  2. 确认进站是阅读模式 (见第 2 项)。
  3. 查看一篇有 `---` 分割线的笔记：应是居中黑色加粗 `* * *`，无灰色背景。
  4. 查看一个引用块 `>`：左侧应无竖线、左侧有装饰性大引号、内容右缩进。
  5. 首页：不应出现 Properties 折叠块与日期 / 标签元信息行。
  6. 随便开几篇笔记，确认表格、Bases、霞鹜文楷、代码块限高、高亮、滚动条等仍正常。
- 一旦发现某项回退，通常只需微调 custom.scss 里对应的选择器以匹配新版本结构，无需重改。

---

## 统一通用步骤 (构建与坑点)

1. 改完上面任意一项后都要 `npx quartz build` 才生效。
2. **本环境注意 (WorkBuddy 删除保护)**：在 WorkBuddy 内执行 `npx quartz build` 时，可能卡在它对 `public` 目录的「回收站式删除保护」(路径转换失败、`fail-closed` 中止构建)。可先把 `public` 移走再构建：

   ```bash
   cd D:/XL/quartz
   mv public public_bak_$(date +%s)   # 若被占用，用资源管理器或 PowerShell 移走
   npx quartz build
   ```

   移走产生的 `public_bak_*` 目录无害，可随手删除。

3. 浏览器看效果时强制刷新 (`Ctrl+Shift+R`) 清掉 favicon / CSS 缓存。
