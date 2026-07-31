---
noteType: index
title: 𝑱𝒐𝒊𝒏 𝒎𝒆 𝒊𝒏 𝒘𝒐𝒏𝒅𝒆𝒓𝒊𝒏𝒈.
description: 你会在这一页了解到关于这个网站与我的一切……
aliases:
tags:
created: 2026-07-20 00:00
modified: 2026-07-31 21:55
cssclasses:
---

𝓦𝓮𝓵𝓬𝓸𝓶𝓮,

我是岭南以北，也是 MG。这里是我的个人博客，也是沉思的漫游地。

目前我专注于提升生活里**体验性**的部分，同时正在学习一些认知流派下的**助人技术**，关注的话题包括，*我们该如何生活？或者说，如何品味生活*？作为一名心理从业者，也许是将来的心理咨询师，那么我该如何运用咨询技术帮助来访者？也许是将来的中学心理老师，那么我该如何将心理咨询的内涵融入课堂当中？无论是关于助人，还是助己，我都有很长的一段路要走。

作为一个学生，我一直有着记录笔记的习惯。然而这些笔记最终都流入垃圾桶……或许也是因为这个原因，博客才会存在的吧。总之如果有你需要的内容，那么恭喜你 (。・∀・) ノ

/𝓶𝓰𝔁𝓱𝓴𝓮𝓯𝓪𝓽𝓮.

---

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
        - not:
            - '!file.hasProperty("noteType")'
        - not:
            - noteType == "index"
    order:
      - formula.name
      - formula.类型
      - formula.上次更新
    sort:
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
