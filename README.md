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