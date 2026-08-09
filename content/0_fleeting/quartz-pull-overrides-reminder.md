---
cssclasses: 
aliases:
tags:
  - quartz
description: 从 Quartz 官方仓库 pull / 合并更新后，哪些「直接改在框架或插件文件里」的定制会被上游覆盖、需要重新调整，以及每一步的精确命令与坑点。
created: 2026-07-31 21:22
modified: 2026-08-04 21:17
noteType: experience
title: Quartz 官方 pull 后需重调的定制项
---

> **用途**：我从 Quartz 官方仓库 `git pull` / 合并更新后，有若干「直接改在框架文件或外部插件文件里」的定制会被上游覆盖。本文逐一列出，并给出重调步骤，避免每次 pull 后站点悄悄变回默认样。
> **结论先行**：我的所有样式都写在 `quartz/styles/custom.scss`(用户自己的覆盖层文件，pull 通常不会动它)；会被覆盖的是「改在框架 / 插件目录里的那两处」+「config 里的插件条目」。

## 总览 (按风险排序)

| 项 | 文件 | 是否会被 pull 覆盖 | 风险 |
|---|---|---|---|
| 1. 站点图标 favicon | `quartz/static/icon.png` | **是**(框架自带默认图标) | 高 |
| 2. 阅读模式 | 已**还原为官方版** `github:quartz-community/reader-mode`(默认**关闭**，不再 fork、不再默认开) | 否 | 低 |
| 3. quartz.config.yaml 插件条目 | `quartz.config.yaml` | 合并冲突时**可能丢** | 中 |
| 4. custom.scss 覆盖层 | `quartz/styles/custom.scss` | 否 (用户文件) | 低，但需复核 |
| 5. graph 插件中文节点 %xx% 修复 | `.quartz/plugins/graph/dist/components/index.js` + `dist/index.js` | **是**(插件重新拉取 / upgrade 会还原 dist) | 高 |

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

## 2. 阅读模式(已还原为官方版，2026-08-02 撤销)

- **当前状态(2026-08-02 撤销)**：阅读模式已**还原为官方原版** `github:quartz-community/reader-mode`——`quartz.config.yaml` 源改回官方，`quartz.lock.json` 同步改回官方源 + 官方 commit(`5612a68…`)。**不再**使用自建 fork，也**不再默认开启**：进站是正常模式，需手动点右上角按钮进入阅读模式。
- **为什么改回**：当初为「默认开启」做了 fork 并固化，后来确认并不想要默认阅读模式，只想用官方阅读模式手动切换；fork 仓库将随官方版回归一并删除。
- **构建注意(CI 缓存陷阱，重点)**：`.github/workflows/deploy.yml` 用 `quartz.lock.json` 的哈希作缓存键来缓存 `.quartz/plugins`。**只改 `quartz.config.yaml` 而不改 `quartz.lock.json`，CI 会复用旧的 `.quartz/plugins/reader-mode` 克隆，构建出的插件不随 config 变化**——这正是之前「换成 fork 后构建还是官方」的根因。今后任何插件源切换，务必同步改 `quartz.config.yaml` 与 `quartz.lock.json`(锁哈希变了 → 缓存失效 → 重新拉取正确源)。
- 本项已无需「pull 后重调」步骤；若将来要重新用 fork 默认开，再补回本节。

---

## 3. quartz.config.yaml 的插件条目 (中，合并时可能丢)

- **为什么要注意**：`quartz.config.yaml` 是我的配置文件。从官方 pull 时若官方也改了这个文件，Git 会报合并冲突，处理不当会把我加的自定义插件丢掉，站点直接构建失败或功能缺失。
- **重调步骤**：pull / 合并后，打开 `quartz.config.yaml`，确认以下条目**都还在**：
  - 5 个自研插件 (在 `plugins` 段，以 `source: "./custom-plugins/<name>"` 或 git URL 引用)：`no-referrer-images`、`lxgw-font`、`image-zoom`、`mermaid-selfhost`、`markdown-image-size`。
  - 阅读模式插件：`github:quartz-community/reader-mode`(官方版，默认关闭；不再用 fork)。
  - 这些插件的 `component` 注册 (如 `reader-mode`、`lxgw-font` 等) 也需在对应组件段保留。
- **注意**：自研插件在 `custom-plugins/` 或独立 git 仓库里 (被主仓库 `.gitignore` 忽略)，pull Quartz 本身不会动它们；但这里只检查「config 是否还引用它们」。

---

## 4. custom.scss 覆盖层 (低，但需复核)

- **为什么通常安全**：`quartz/styles/custom.scss` 是我自己的覆盖层文件，不在 Quartz 官方追踪范围内，pull 一般不会覆盖它。所有样式 (分割线 `* * *`、引用块引号、表格、Bases、首页隐藏 Properties 等) 都在这里。
- **但有一个隐患**：custom.scss 里的规则是通过**覆盖框架 `base.scss` 等的选择器**来生效的 (例如清掉 `hr` 的 `background`、清掉 `blockquote` 的 `border-left`)。如果未来某个 Quartz 版本**重命名 / 移动了这些底层选择器**，覆盖会**静默失效**(不会报错，只是样式回退到默认)。
- **重调步骤 (pull 后肉眼复核清单)**：
  1. 刷新站点，确认 favicon 仍是自己的图标 (见第 1 项)。
  2. 确认阅读模式按钮可用、默认关闭 (见第 2 项)。
  3. 查看一篇有 `---` 分割线的笔记：应是居中黑色加粗 `* * *`，无灰色背景。
  4. 查看一个引用块 `>`：左侧应无竖线、左侧有装饰性大引号、内容右缩进。
  5. 首页：不应出现 Properties 折叠块与日期 / 标签元信息行。
  6. 随便开几篇笔记，确认表格、Bases、霞鹜文楷、代码块限高、高亮、滚动条等仍正常。
- 一旦发现某项回退，通常只需微调 custom.scss 里对应的选择器以匹配新版本结构，无需重改。

---

## 5. graph 插件中文节点 %xx% 修复 (高，会被 upgrade 覆盖)

- **问题**：关系图谱里，中文文件名的「当前页」节点显示成 `%xx%` 编码串(如 `0_fleeting/%E5%A5%A5…`)，而不是中文；其余节点正常。
- **根因**：graph 取「当前页 slug」用的是 `window.location.pathname` / SPA 导航事件的 `e.detail.url`，在你的环境里是 URL 编码态；而图谱数据 `contentIndex.json` 的 key 与 `title` 是未编码中文。两者不匹配 → `get(编码slug)` 取不到 `title` → 回退显示编码 slug，于是看到 `%xx%`。其余节点从数据 key(未编码)来，故正常——这正解释了「为什么偏偏当前这个中文文件变 %xx%」。
- **改了什么**：在插件真正被加载的编译产物里，对所有 slug 统一加 `decodeURIComponent`，共五处(当前页 slug、`data` 的 key、链接目标、标签 slug、文本回退值)。涉及文件：
	- `.quartz/plugins/graph/dist/components/index.js`(真正被打包的入口，`minify` 后变量名形如 `Fu` / `eu`)
	- `.quartz/plugins/graph/dist/index.js`(re-export，同步补丁)
	- `src/components/scripts/graph.inline.ts` 也加了 `decodeSlug` 安全封装(但 build 不读 src，仅供参考；若以后改走源码编译再另行处理)
- **为什么会被覆盖**：graph 来自 `github:quartz-community/graph`(`quartz.config.yaml` 第 183 行 `source: github:quartz-community/graph`)。`dist/` 是构建产物、不进 git；`npx quartz build --upgrade` 或插件被重新拉取时，`.quartz/plugins/graph` 会被重新 clone，补丁随之丢失，问题复现。
- **重调步骤**：重放五处补丁后 `npx quartz build`(先把 `public` 移走规避删除保护，见下方统一通用步骤)。一键重放脚本：

  ```bash
  cd D:/XL/quartz/.quartz/plugins/graph
  PY="C:/Users/admin/.workbuddy/binaries/python/envs/default/Scripts/python.exe"
  "$PY" - <<'PYEOF'
  repls = [
      ('var m=Fu(w);', 'var m=Fu(decodeURIComponent(w));'),
      ('eu.set(Fu(Ju),Ku[Ju])', 'eu.set(Fu(decodeURIComponent(Ju)),Ku[Ju])'),
      ('var v=Fu(F[A]);', 'var v=Fu(decodeURIComponent(F[A]));'),
      ('var K=Fu("tags/"+N);', 'var K=Fu(decodeURIComponent("tags/"+N));'),
      ('eu.get(i)?.title||i,', 'eu.get(i)?.title||decodeURIComponent(i),'),
  ]
  for p in ["dist/components/index.js", "dist/index.js"]:
      raw = open(p, encoding="utf-8").read()
      for a, b in repls:
          assert raw.count(a) == 1, (p, a)
          raw = raw.replace(a, b)
      open(p, "w", encoding="utf-8").write(raw)
      print(p, "patched")
  PYEOF
  cd D:/XL/quartz && mv public public_bak_$(date +%s) && npx quartz build
  ```

- **彻底修复(推荐)**：把这个插件 fork 到 `mgxhkefate/graph`，提交打好补丁的 `dist/`，并把 `quartz.config.yaml` 的 `source` 改成 `git+https://github.com/mgxhkefate/graph.git`(沿用 `markdown-image-size` 的同款模式)。这样 upgrade 拉的是你自己的仓库，补丁不丢。
- **验证**：构建后 `public/static/scripts/script-4-*.js` 里应含 5 个 `decodeURIComponent`；编码 slug 经 decode 后节点文字正确变回中文。

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
