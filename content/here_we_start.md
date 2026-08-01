---
noteType: MOC
title: 𝕊𝕥𝕒𝕣𝕥 𝕗𝕣𝕠𝕞 𝕙𝕖𝕣𝕖.
aliases:
  - start
tags:
description: 你会在这一页了解到这个博客有哪些内容、将会有哪些内容，以及一份浏览这个博客的简单指南。
created: 2026-08-01 10:25
modified: 2026-08-01 11:50
cssclasses:
---

𝓦𝓮𝓵𝓬𝓸𝓶𝓮,

> 一半白天一半夜
> 一半短暂一半长
>
> 一半藏刀归鞘
> 一半马灯点亮
>
> 一半的刀子难免越磨越钝
> 一半的马灯总会越擦越亮
>
> 一半情种老迈，半俗半雅
> 一半浪子还乡，半嗔半怨
>
> 大道无言，一半还于天地
> 悲喜如常，一半让向人间
>
> *出自，张子选《藏地诗篇》· 一半一半*

---

你说人来这一世上，是否要把一半还于天地？张子选是我高中的时候舍友推荐给我的现代诗诗人，他的诗真的就像歌谣一样朗朗上口，并且还带有一点民间歌谣的哲理性！扯远了……

目前对于博客的规划，我主要设置了三个文件夹。由于网站默认开了阅读模式，所以*电脑端必须要鼠标移动到两侧、手机端要点一点上方或下方的空白位置*才能看到**文件夹**或者**关系图谱**：

1. `0_fleeting`：收录了一些平时写的公众号推文、别人的观点集合，或者一些理论等等。通常包含经验 (experience)、观点 (perspective) 和日常 (daily) 三类。
	- 经验类笔记偏实操类型，里面大部分都是一些事实依据且可重复性高的内容；
	- 观点类笔记偏观点、理论、看法，主观性较强且没有太多事实依据；
	- 日常类笔记就是我平时形成的一次性笔记，包括但不限于一些网站快照的记录、计划的记录。
2. `1_resource`：收录了一些平时形成好的、可公开的，且有明确维护记录的资源，相比于经验类和观点类笔记更为系统。换句话说，资源笔记就是根据主题或一定目的将经验笔记和观点笔记组织联系起来了。因此，`1_resource` 里的笔记可能与 `0_fleeting` 里的笔记有些重合之处。
3. `2_sharing`：目前没想好这个文件夹用来干什么，以后想到再说吧 (～￣▽￣)～

除此以外，你还可以从我下面配置好的**数据库**中点进你想要查看的笔记。关于笔记的创建时间、修改时间以及上次更新字段，由于我的笔记是从另一个笔记库里复制过来的，所以笔记的创建时间并不准确，而修改时间和上次更新则是根据 git push 到 github 的时间算的，因此虽然有一定的参考价值，但与现实情况并不一致。

```base
formulas:
  name: link(file.name, title)
  类型: noteType
  创建时间: created
  修改时间: modified
  上次更新: file.mtime.relative()
  描述: description
  标签: tags
properties:
  note.noteType:
    displayName: 类型
  note.tags:
    displayName: 标签
  note.created:
    displayName: 创建日期
  note.modified:
    displayName: 修改日期
  formula.name:
    displayName: 笔记名称
  formula.类型:
    displayName: 内容类型
views:
  - type: table
    name: 所有笔记
    filters:
      and:
        - and:
            - file.hasProperty("noteType")
            - '!file.name.contains("index")'
            - '!file.name.contains("here_we_start")'
    order:
      - formula.name
      - formula.类型
      - formula.上次更新
    sort:
      - property: formula.上次更新
        direction: DESC
      - property: file.mtime
        direction: DESC
    columnSize:
      formula.name: 350
      formula.类型: 50
      formula.上次更新: 100
  - type: table
    name: 碎碎念
    filters:
      and:
        - file.tags.contains("岭南碎碎念")
    order:
      - formula.name
      - formula.创建时间
      - formula.修改时间
    sort:
      - property: file.mtime
        direction: DESC
    columnSize:
      - formula.name: 300
      - formula.创建时间: 100
      - formula.修改时间: 100

```