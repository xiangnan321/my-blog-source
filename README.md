使用 Hexo + GitHub Pages 实现的免费托管博客。

### 创建新帖子

``` bash
$ hexo new <title>  #hexo new "My New Post"
```

### 创建草稿

``` bash
$ hexo new draft <title> # hexo new draft "未完成"
```

### 发布草稿

``` bash
$ hexo publish <filename>   # hexo publish _drafts/未完成.md
```

### 本地启动测试

``` bash
$ hexo clean && hexo server  # 清理缓存并启动本地服务器
```

### 本地实时预览

``` bash
$ hexo server --draft  # 实时渲染草稿
```

### 安装部署插件

``` bash
$ npm install hexo-deployer-git --save  # 安装 Git 部署工具
```

### 生成静态文件并部署

``` bash
$ hexo clean && hexo generate --deploy  # 清理 → 生成HTML → 推送至GitHub
```

### 部署更新

``` bash
$ hexo clean && hexo g -d  # 一键清理+生成+部署
```

### 本地测试

``` bash
$ hexo clean   # 清除旧缓存（避免修改未生效）
$ hexo g       # 重新生成静态文件
$ hexo s       # 启动本地服务器预览（http://localhost:4000）
```

## Front Matter（文章元数据）
在每篇文章的 Markdown 文件头部，用 --- 包裹的 YAML 区域定义文章属性：
``` markdown
---
title: 文章标题          # 【必填】支持HTML标签（如<br>）
date: 2024-01-01       # 【必填】格式 YYYY-MM-DD HH:mm:ss
updated: 2024-01-02    # 更新时间（可选）
categories: 
  - 分类1              # 多级分类用列表
  - 分类2
tags: [标签1, 标签2]    # 标签可用数组或列表
cover: /images/cover.jpg  # 封面图路径（相对source目录）
toc: true              # 是否显示目录（需主题支持）
mathjax: true          # 启用数学公式（如LaTeX）
comments: false        # 关闭当前文章评论
sticky: 100            # 置顶优先级（数值越大越靠前）
---
```