【博客项目说明】

这是一个基于 Hugo + PaperMod 主题的个人博客项目。
- 内容目录：`content/blog/<slug>/index.md`（Page Bundle 模式，每篇文章一个目录）
- 主题：PaperMod，已在 `hugo.toml` 启用 `unsafe = true`，允许 Markdown 中嵌入 HTML 与 shortcode
- 构建产物：`public/`，不要手工修改


【文章资源放置（Page Bundle）】

每篇文章必须采用 Page Bundle 结构，所有相关资源（图片、附件、绘图原文件）放在与 `index.md` 同级目录：

```
content/blog/<slug>/
├── index.md        # 正文
├── xxx.png         # 图片资源
├── xxx.svg         # 矢量图
└── attachments/    # 可选，存放 PDF、源工程等大附件
```

引用一律使用相对路径（不带 `/`、不带子目录前缀），例如 `![](xxx.png)`。
不要使用 `static/`、`assets/` 存放文章资源。


【图片插入机制 - 强制规范】

设计目标：在 **IDE Markdown 预览、Hugo 站点、GitHub** 三处都能正常渲染，所见即所得。
所以 **禁止** 使用 Hugo shortcode（`{{< figure >}}`、`{{< img >}}` 等），因为它们只在 `hugo server` 时被渲染，IDE 预览和 GitHub 上都会变成纯文本。

1. 标准写法（必须）：

   ```markdown
   ![alt 文本：客观描述图片内容，用于无障碍与 SEO](xxx.png)
   *图 N：渲染在图片下方的图注，给读者看，可包含图号与解释*
   ```

   - 图片下面紧跟一行斜体（用 `*...*` 包裹），中间 **不要空行**，让多数 MD 渲染器把它视为图注

2. 字段说明：
   - `alt`：必填，简短客观，不要写"如下图所示"这种无信息量内容；典型长度 1 句话
   - 图注：强烈建议，统一格式 `图 N：<说明>`，N 在同一篇文章内顺序编号
   - 图片路径：同目录直接写文件名 `xxx.png`，不要写 `/blog/xxx/xxx.png`

3. 自适应布局：
   - 浏览器对 `<img>` 默认是按原图尺寸渲染，但 PaperMod 主题已对正文 `<img>` 应用 `max-width:100%`，移动端会自动缩放
   - 不要在文章里写死像素宽度
   - 极个别需要缩小或并排的图，统一去 `assets/css/extended/custom.css` 里加自定义 class，不要在文章里堆内联样式或 HTML

4. alt vs 图注 的区别（务必区分）：
   - `alt`：图片加载失败或屏幕阅读器使用，给机器看
   - 图注（斜体行）：渲染在图下方，给读者看，可包含图号、解释、引用来源

5. 退化方案：临时贴图或不需要图注时，可只写 `![alt](xxx.png)`，但 `alt` 始终必填。

6. 反例（禁止使用）：

   ```text
   {{< figure src="xxx.png" caption="..." >}}   ← shortcode，IDE 预览失效
   <img src="xxx.png" width="600">              ← 写死像素，移动端会溢出
   ![](xxx.png)                                  ← 缺 alt
   ![如下图所示](xxx.png)                        ← alt 无信息量
   ```


【图片命名规范】

- 全小写 + 连字符（kebab-case）：`hnsw-overview.png`
- 语义前缀：`<topic>-<what>.png`，便于 IDE 排序与检索
  - 示例：`hnsw-build.png`、`hnsw-search.png`、`hnsw-mem-layout.png`
- 不要使用中文文件名、空格、大写


【图片格式与体积】

| 类型 | 推荐格式 | 说明 |
|------|---------|------|
| 截图 | PNG | 文字清晰，无损 |
| 照片 | JPG | 压缩率高 |
| 流程图/示意图 | SVG（首选）/ PNG | SVG 矢量、无损缩放，体积小 |
| 动图 | WebP / GIF | 优先 WebP |

体积要求：
- 单张图片建议 < 500 KB；超过请用 tinypng / squoosh 压缩
- SVG 优先（特别是手绘示意图，可以用 Excalidraw / draw.io 导出）


【Markdown 文章前置规范（frontmatter）】

每篇 `index.md` 顶部 frontmatter 至少包含：

```yaml
---
title: 文章标题
date: 2026-05-31T10:00:00+08:00
tags: [标签 1, 标签 2]
series: [系列名]            # 可选，归属系列
featured: true              # 是否首页推荐
description: "一句话摘要，会显示在列表与社交分享卡片"
draft: true                 # 完成前保持 true，发布时改 false
ShowToc: true               # 必须开启，展示文章目录
TocOpen: true               # 目录默认展开
---
```

`ShowToc` 和 `TocOpen` 为必填字段。全局配置 `ShowToc = false`，必须在每篇文章的 frontmatter 中单独开启。


【URL 兼容性】

Hugo 中 `xxx.md` 与 `xxx/index.md` 生成的 URL 完全相同（都是 `/blog/xxx/`），改造为 Page Bundle 不会破坏现有链接和 SEO。
