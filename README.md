# zengxianming01.github.io

这是一个基于 Jekyll 的个人站点，用来发布 AI 学习、AI 工具实战、项目复盘、阅读笔记和生活思考等内容。

## 本地预览

先确保使用 Homebrew Ruby：

```bash
export PATH="/opt/homebrew/opt/ruby/bin:/opt/homebrew/lib/ruby/gems/4.0.0/bin:$PATH"
```

安装依赖：

```bash
bundle install
```

启动本地服务：

```bash
bundle exec jekyll serve
```

然后打开：

```text
http://127.0.0.1:4000
```

如果 4000 端口被占用，可以换端口：

```bash
bundle exec jekyll serve --port 4001
```

## 新增文章

所有文章放在 `_posts/` 目录下。

文件名必须使用 Jekyll 约定格式：

```text
_posts/YYYY-MM-DD-your-title.md
```

例如：

```text
_posts/2026-06-07-my-ai-note.md
```

文章开头需要写 front matter：

```markdown
---
layout: post
title: "我的第一篇 AI 学习笔记"
date: 2026-06-07
category: ai-learning
tags:
  - AI
  - 学习笔记
  - Agent
excerpt: "这是一篇关于 AI 学习过程的小结。"
---

这里开始写正文。

## 小标题

正文支持 Markdown、代码块、表格、图片链接等。
```

## 分类说明

当前站点支持这些分类：

| category | 页面名称 | 适合内容 |
|---|---|---|
| `ai-learning` | AI 学习 | 模型原理、学习路线、概念拆解、智能体理论 |
| `ai-tools` | AI 工具实战 | Codex、提示词、自动化、工具链、工作流 |
| `project-review` | 项目复盘 | 项目过程、踩坑记录、技术决策、复盘总结 |
| `reading-notes` | 阅读笔记 | 书、文章、论文、访谈整理 |
| `life-notes` | 生活思考 | 注意力、节奏、关系、成长和日常观察 |
| `fragments` | 随笔札记 | 短想法、观察、未完成的问题 |

`category` 必须填写左侧的 slug，例如：

```yaml
category: ai-tools
```

分类配置文件在：

```text
_data/categories.yml
```

如果以后要新增长期栏目，需要同时：

1. 在 `_data/categories.yml` 增加分类配置。
2. 在 `categories/` 下新增对应分类页。
3. 文章 front matter 的 `category` 使用新的 slug。

## 新文章会出现在哪里

新增文章后，Jekyll 会自动把它展示在：

- 首页的“近期内容”
- 对应分类页，例如 `/categories/ai-learning/`
- 归档页 `/archive/`
- 文章详情页，例如 `/ai-learning/2026/06/07/my-ai-note/`

## 使用 HTML 写页面或文章

Jekyll 不只支持 Markdown，也支持 HTML。

### 普通 HTML 页面

如果要创建一个普通页面，可以新建：

```text
pages/about.html
```

内容示例：

```html
---
layout: default
title: 关于我
permalink: /about/
---

<h1>关于我</h1>
<p>这里是 HTML 内容。</p>
```

只要文件顶部有 front matter，Jekyll 就会处理它，并生成：

```text
/about/
```

### HTML 文章

HTML 也可以放进 `_posts/`，作为正式文章发布：

```text
_posts/2026-06-07-my-html-post.html
```

内容示例：

```html
---
layout: post
title: "一篇 HTML 文章"
date: 2026-06-07
category: ai-learning
tags:
  - HTML
excerpt: "这是一篇用 HTML 写的文章。"
---

<h2>正文标题</h2>
<p>这里可以直接写 HTML。</p>
```

这种 HTML 文章会和 Markdown 文章一样，自动出现在：

- 首页“近期内容”
- 对应分类页
- 归档页
- 文章详情页

### Markdown 中混写 HTML

Markdown 文件里也可以直接写 HTML：

```markdown
---
layout: post
title: "Markdown 里写 HTML"
date: 2026-06-07
category: ai-tools
---

这是一段 Markdown。

<div class="custom-box">
  <strong>这里是 HTML 块</strong>
</div>
```

普通文章建议优先使用 Markdown；如果需要复杂排版、特殊组件或手写结构，可以使用 HTML。

## 推荐写作流程

最省心的方法是复制一篇已有文章：

```text
_posts/2026-06-02-rtk-ai-tools.md
```

然后修改：

- 文件名日期和 slug
- `title`
- `date`
- `category`
- `tags`
- `excerpt`
- 正文内容

## 构建检查

写完文章后，运行：

```bash
bundle exec jekyll build
```

如果构建成功，再启动本地服务预览：

```bash
bundle exec jekyll serve
```

## Markdown 注意事项

- 图片可以使用远程链接：

```markdown
![图片说明](https://example.com/image.png)
```

- 本地图片建议放在 `images/` 目录下：

```markdown
![图片说明](/images/example.jpg)
```

- 代码块建议带语言名：

````markdown
```python
print("hello")
```
````

- Obsidian 的双链语法 `[[...]]` 不会被 Jekyll 自动识别，发布前建议改成普通 Markdown 链接或纯文本。

## 常见问题

### 端口被占用

查看 4000 端口：

```bash
lsof -nP -iTCP:4000 -sTCP:LISTEN
```

换端口启动：

```bash
bundle exec jekyll serve --port 4001
```

### Ruby 或 bundle 命令不对

如果系统仍在使用 macOS 自带 Ruby，需要先设置 PATH：

```bash
export PATH="/opt/homebrew/opt/ruby/bin:/opt/homebrew/lib/ruby/gems/4.0.0/bin:$PATH"
```

确认：

```bash
which ruby
ruby -v
which bundle
bundle -v
```
