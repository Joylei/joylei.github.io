+++
title = "zola 简单使用说明"
date=2020-09-04

[taxonomies]
categories=["Misc"]
tags=["blog", "static", "zola", "rust"]
+++

[zola](https://www.getzola.org/)是用`pure rust`写的静态建站工具，类似的工具有用`go`写的`hugo`。

# 安装zola

可以直接下载二进制文件。也可以使用各系统对应的安装工具安装。

MacOS

```shell
brew install zola
```

alpine linux:

```
apk install zola
```

Windows
使用`Scoop`或者`Chocolatey`安装。

# 使用

## 建立新站点

```shell
zola init mysite
```

这里`mysite`是你的站点名字，会创建对应的目录。

## 目录结构

建好后的目录结构

```text
mysite
- content
- public
- sass
- static
- templates
- themes
- config.toml
```

`content`里存放markdown文件
`public`里存放编译好的文件
`sass` 存放自定义的sass文件
`static`存放额外的静态资源
`templates`存放自定义的模板文件
`themes`存放下载的主题
`config.toml`站点的配置文件

## 下载主题

官方已经有一些主题，可以到[这里](https://www.getzola.org/themes/)查看。

下载下来放在`themes`目录就可以了。每个主题都有使用说明，按照说明操作。

## 内容相关概念

### front matter

在markdown文件中定义的元数据区域，以`+++`来标识起始和结束。类似于：

```text
+++
title = "Something new"
date = 2017-09-25

[taxonomies]
categories = ["Prog"]
tags = ["rust", "async"]
+++
```

### Section

`content`目录下的任何一个目录，只要它下面有`_index.md`文件，那么这个目录就是一个`section`。
`_index.md`就是元数据文件。
`section`需要`section.html`模板来渲染内容，如果在主题的`templates`目录没有这个文件，按照`index.html`修改一个。
不同的地方就是`index.html`中使用`paginator`变量，在`section.html`中使用`section`变量，它们的属性是一样的。

`section`相关元数据定义

```toml
title=""
description=""
sorted_by="none"
template="section.html"

# render为true时会显示section的页面，false时不显示
render=true
# transparent为true时内容对父section可见，false时不可见
transparent=false
```

更多内容查看[`section`定义页面](https://www.getzola.org/documentation/content/section/)。

### Page

`page`就是一个个页面。
如果一个目录下有`index.md`，那么这个目录就会渲染成一个页面。例如`content\life\my-note\index.md`，会形成URL类似于`http://base_url/life/my-note`。
如果文件的名字不是`index.md`，而是`something.md`，那么形成的URL类似于`http://base_url/life/something`。

`page`的主要`front matter`属性：

```toml
title = "Something new"
date = 2017-09-25

[taxonomies]
categories = ["Prog"]
tags = ["rust", "zola"]
```

每个页面基本上只设置上述属性就够了，更多查看[这里](https://www.getzola.org/documentation/content/page/#front-matter)。

### 静态资源

静态资源可以在markdown文件中用相对路径引用，`zola`在build时会把资源复制到`public`目录对应的位置。

### 语法高亮

需要在`config.toml`启用，并配置一个主题。

```toml
highlight_code = true

highlight_theme = "material-dark"

```

更多语法高亮主题看[这里](https://www.getzola.org/documentation/getting-started/configuration/#syntax-highlighting)。

### Taxonomies

在config.toml中配置分类和标签。

``` rust
taxonomies = [
    {name = "categories", rss = true},
    {name = "tags", rss = true},
]
```

### Feed

在 config.toml中配置：

```toml
generate_feed = true
```

### 链接与引用

锚点

`zola`会为内容中的标题生成唯一锚点。如：

```
# Something exciting! <- something-exciting
## Example code <- example-code

# Something else <- something-else
## Example code <- example-code-1
```

也可以手动指定锚点:

```
# Something manual! {#manual}
```

内部链接引用
`content`目录下的页面可以直接以`@/`开头引用。例如：
`content/pages/about.md` 这样引用 `[my link](@/pages/about.md)`。

## Serve

本地`serve`预览：

```shell
zola serve --interface 0.0.0.0 --port 2000
```

## Build

构建静态页面：

```shell
zola build
```

## Deploy to github

使用github actions部署。
简单的来说`github`允许选择哪个分支来发布。
你可以把所有代码上传到某个分支，然后使用[`zola-deploy-action`](https://github.com/shalzz/zola-deploy-action)来自动构建和发布。

这里示例的是用`code`分支存放所有代码，用`master`分支做`Github Pages`。

- 在`Github`创建一个`Personal Access Token`: <https://github.com/settings/tokens>，记住这个`token`的值
- 把这个`token`的值添加到项目的设置的`Secrets`里，并且名字为`TOKEN`
- 添加`github action`：在``code`分支创建`.github/workflows/zola.yml`，内容为

```text
on:
  push:
    branches:
      - code
  pull_request:
jobs:
  build:
    runs-on: ubuntu-latest
    if: github.ref != 'refs/heads/code'
    steps:
      - name: 'Checkout'
        uses: actions/checkout@main
      - name: 'Build only'
        uses: shalzz/zola-deploy-action@v0.10.1
        env:
          BUILD_DIR: .
          TOKEN: ${{ secrets.TOKEN }}
          BUILD_ONLY: true
  build_and_deploy:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/code'
    steps:
      - name: 'Checkout'
        uses: actions/checkout@main
      - name: 'Build and deploy'
        uses: shalzz/zola-deploy-action@v0.10.1
        env:
          PAGES_BRANCH: master
          BUILD_DIR: .
          TOKEN: ${{ secrets.TOKEN }}
```

- 当你对远程仓库执行`pull`或者`push`代码到`code`分支时就会自动运行`workflow`

## More

到[官网](https://www.getzola.org/)去探索更多内容吧。
