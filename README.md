# Kyre0ee <!-- omit in toc -->

<a href="https://jekyll-themes.com">
  <img src="https://img.shields.io/badge/featured%20on-JekyllThemes-red.svg" height="20" alt="Jekyll Themes Shield" >
</a>

**Kyre0ee的个人博客** is a simple, beautiful, and powerful Jekyll theme for blogs. 

### 配置

 [`_config.yml`](_config.yml) 文件中包含一些变量，可以直接修改。

### 定制头部

可以自定义头部，可以在这里添加任何内容，比如网站图标。创建 `_includes/custom-head.html` 并把内容放入其中.

### 创建主题

可以制作自己的配色方案 `_sass/_variables.scss` 在该文件中使用范围数据属性或类名修改样式表中的css变量.

下面创建的是一个蓝色主题:

```scss
// Example blue theme
[data-theme="blue"] {
  --body-bg: var(--blue);
  --body-color: #fff;
}
```

Then, apply the theme by adding `data-theme="blue"` to the `<html>` element.

### 自定义导航

通过创建 `_data/navigation.yml`文件来配置某些页面的链接。例如,

```yml
- title: Blog
  url: /
- title: About
  url: /about/
```

### 自定义封面图片

您可以通过修改中的变量来设置您自己的封面图片 `cover_image` variable in `_config.yml`, 也可以通过在每个页面上设置该 `cover_image` 变量来为不同的页面设置不同的封面图片。.

如果你发现封面文字颜色和封面背景颜色的对比度不够，你也可以调整这两个变量:

```yml
cover_bg_color: rgb(40, 73, 77)
cover_color: rgb(255, 255, 255)
```

### 自定义社交链接

You can set your social links in `_data/social.yml`. You can custom titles, URLs, and icons (only support [Font Awesome](https://fontawesome.com/) currently), for example:

```yml
- title: Email
  url: mailto://vszhub@gmail.com
  icon: fas fa-envelope
- title: Twitter
  url: https://twitter.com/vszhub
  icon: fab fa-twitter
- title: GitHub
  url: https://github.com/vszhub/not-pure-poole
  icon: fab fa-github
```

### 帖子存档

支持按日期、类别和标签归档帖子。要启用此功能，您应该将以下一些数据放入 `_data/archive.yml`:

```yml
- type: dates
  title: Dates
  url: /dates/
- type: categories
  title: Categories
  url: /categories/
- type: tags
  title: Tags
  url: /tags/
```

之后，这些存档页面的导航将显示在主页顶部。

然后，您可以创建一个类别存档页面，并在该页面上设置以下参数：

```yml
---
layout: archive-taxonomies
type: categories
---
```

Or a tag archive page:

```yml
layout: archive-taxonomies
type: tags
```

Or archive by dates:

```yml
layout: archive-dates
```

### 启用 TOC

如果您想在右侧显示页面的目录，只需toc: true在该页面上进行设置即可。

### 启用 MathJax

如果您想在页面上写数学，只需`math: true`在该页面上设置以启用 MathJax  

### 修改摘要  
默认情况下，post.excerpt 会自动生成文章的摘要，通常是文章开头的一部分。如果你想手动控制摘要长度或内容，可以在文章的 Front Matter（文章开头的 YAML 区域）中添加 excerpt 字段。
---
title: "My Blog Post"
date: 2024-05-29
excerpt: "This is a custom excerpt for my blog post."
---
或者，如果你想截取文章内容的前 N 个字符作为摘要，可以这样做：
<p>{{ post.content | strip_html | truncatewords: 50 }}</p>  
将文章内容截取到前 50 个单词，并去除 HTML 标签。

