
# what's jekyll

jekyll是一个集成在[GitHub Pages](https://docs.github.com/zh/pages)中的静态站点生成器，通过极其简单的语法和规则构建整个静态站点，作为一个想要免费使用GitHub Pages搭建博客的人来说，jekyll是不二之选。

# why jekyll

1. Github Pages 的内置集成，官方推荐使用jekyll作为GitHub Pages的静态站点生成器
2. 不用使用繁杂的Github Actions即可自动触发GitHub Pages的CICD
3. 对比其他生成器，轻量，学习成本非常低，是最有性价比的博客生成器
4. 以markdown为内容载体
5. 丰富的主题样式可供选择，极简配置主题，无需你手动写样式
6. ruby的配置模式以及工程化对程序员友好

# 45分钟指南

## 5分钟下载/安装

1. 安装ruby installer
[RubyInstaller](https://rubyinstaller.org/downloads/)

选择最新版带有DEVKIT的Ruby版本
![image.png](https://s2.loli.net/2024/12/03/sRDB5tZyLjrAh7E.png)
2. 安装完成后，在命令行输入`ridk` , 你可以看到会有ruby install的样式icon显示，此时在命令行输入`ridk install` 并且选择MSYS2 and MINGW development toolchain, 键入3，即下图
![image.png](https://s2.loli.net/2024/12/03/2VzlOoMSvugYIqG.png)

如果出现`错误：无法初始化事务处理 (无法锁定数据库) 错误：无法锁定数据库：Permission denied` 请使用管理员身份运行。

3. 通过gem安装jekyll以及bundler
	命令行输入`gem install jekyll bundler` 

## 3分钟项目搭建

### 2分钟初始化项目

1. 运行

```shell
bundle init
```

此时会生成gem file， 里面会自带一行代码：
```gem
source "https://rubygems.org"
```

2. 这个是指定包管理的依赖所配置的源，由于国外服务器访问速度受限的缘故，在没有科学上网工具的情况下，建议换镜像源：
```gem
source "https://mirrors.tuna.tsinghua.edu.cn/rubygems/"
```

3. 此时我们需要在当前文件(Gemfile)中写入安装主要依赖
```gem
gem "jekyll"
```

> 这里的jekyll是项目本地依赖而不是之前gem install的全局依赖，如果项目中（Gemfile中）有这个依赖优先用项目依赖中的版本


4. 此时命令行运行
```shell
bundle
```
将安装所有Gemfile中的依赖

### 1分钟运行空项目

1. 创建根文件

```html:index.html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Home</title>
  </head>
  <body>
    <h1>Hello World!</h1>
  </body>
</html>
```

2. 打包初始化项目

命令行运行
```shell
jekyll build
```
此时jekyll会将当前根目录以及其所有子目录相关代码打包生成`_site`目录，这里面就是打包后的文件（由于咱们初始化项目看不出来打包痕迹，后面随着内容增加以后，里面的文件会被压缩）

此时目录结构如下
```tree
.
├── _site
│   └── index.html
├── Gemfile
├── Gemfile.lock
└── index.html
```


3. 运行项目

命令行执行
```shell
jekyll serve --livereload --port 3456
```

这里livereload是热更新（即每次修改代码都会自动更新，建议加上）--port指定运行端口3456

访问`http://localhost:3456/
即可生成初始化项目

## 30分钟jekyll实践指南

### 文件组成

由于jekyll是针对markdown格式的静态生成器，所以他能将所有以md/markdown结尾文件为markdown格式的页面视为html文件，可以将markdown文件当作页面处理。由于markdown文件简单易用行，我们可以将除了根文件`index.html`以及后续所说的layouts等相关配置型文件使用html以外，其余页面使用md格式简易处理。

### Front Matter

其被叫做：前置信息，这个是用于写入一些类似于html中的meta数据，来标识整个文件的信息。在jekyll中，每一个yml/markdown/html文件都可以在文件开头以如下形式引入：
```
---
变量名：变量值
---
```

### Layout

前端有个叫做layout的概念，称为布局，意思是页面的结构模板，我们通过引用/继承布局来快速实现页面结构，所以，现在我们在根目录创建`_layouts`文件夹，在里面创建一个default.html文件来表示我们的默认布局，内容如下

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>{{ page.title }}</title>
  </head>
  <body>
    {{ content }}
  </body>
</html>
```

此时目录如下
```text:tree.txt
.
├── _layouts
│   └── default.html
├── _site
│   ├── about.html
│   └── index.html
├── about.md
├── Gemfile
├── Gemfile.lock
└── index.html
```


这里面用双花括号括起来的是jekyll自带的系统变量，如：
1. page
	page变量表示整个页面的信息，page.title会获取页面Front Matter中的title

2. content
	content 变量表示整个页面的内容，比如处在markdown文件中的所有文本


使用layout

这里我们在根目录中创建一个关于的page，使用markdown形式：
```markdown:about.md
---
layout: default
title: About
---

This is XXX. A software developer.
```

访问`http://localhost:3456/about/` 此时我们会看到about页面渲染出来，并且页面tab标签的title也变为了About

### includes

此时我们发现一个问题：我们很难知道整个项目有什么页面，即：没有导航栏支撑，页面全靠用户手动输入特别麻烦，所以，我们使用includes来把整个导航栏加入到layout中

includes被称作重用代码片段，通过将重复的内容抽取到 `_includes` 文件夹中，我们可以在多个页面中轻松插入这些内容，减少重复工作并使代码更加简洁和易于维护。

创建`_includes`文件夹及其在其添加`navigation.html`：
```html:_includes/navigation.html
<nav>
  <a href="/">Home</a>
  <a href="/about">About</a>
</nav>
```

由于是代码片段，`navigation.html`不需要保留完整的html文件格式，只需要相应的标签片段即可
此时目录格式：
```text:tree.txt
.
├── _includes
│   └── navigation.html
├── _layouts
│   └── default.html
├── _site
│   ├── about.html
│   └── index.html
├── about.md
├── Gemfile
├── Gemfile.lock
└── index.html
```

并且在`default.html`layout中增加include语法:
```html:_layouts/default.html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>{{ page.title }}</title>
  </head>
  <body>
    {% include navigation.html %} {{ content }}
  </body>
</html>
```

此处`{% %}` 是专为jekyll自带语法的标识符，`include`将引入_includes目录中的文件作为代码片段使用。

我们访问任意链接（home、about）即可发现页面首部增加了导航栏，点击仍以链接即可跳转

另外我们可以通过jekyll系统变量来设置样式，使导航栏的地址出现聚焦态， 这里修改`navigation.html`中的代码：
```html:_includes/navigation.html
<nav>
  <a href="/" {% if page.url == "/" %}style="color: red;"{% endif %}>
    Home
  </a>
  <a href="/about.html" {% if page.url == "/about.html" %}style="color: red;"{% endif %}>
    About
  </a>
</nav>
```
其中jekyll的if条件判断的语法通过条件判断开始符`{% if xxx %}`以及结束符`{% endif %}` 来进行判断，另外如果需要多个if 判断可以在其中加入`{% elsif xxx %}` 以及`{% else %}` 如下：
```html:_includes/navigation.html
<nav>
  <a href="/"
     {% if page.url == "/" %}
       style="color: red;"
     {% elsif page.url == "/home.html" %}
       style="color: blue;"
     {% else %}
       style="color: black;"
     {% endif %}>
    Home
  </a>
  <a href="/about.html"
     {% if page.url == "/about.html" %}
       style="color: orange;"
     {% elsif page.url == "/info.html" %}
       style="color: green;"
     {% endif %}>
    About
  </a>
</nav>
```


### Data Files

但是我们仍然发现这种配置十分麻烦，需要不断给navigation添加各种各样链接， 我们可以通过数据存储文件(Data Files)的方式来配置文件，这种文件配置信息一般存放在`_data`文件夹下，这种配置信息一般存为yml格式文件，于是我们创建 `_data/navigation.yml`, 如下：

```yaml:_data/navigation.yml
- name: Home
  link: /
- name: About
  link: /about.html
```

`_data`文件夹中的信息是由jekyll系统变量`site.data`访问，如上面的数组化结构即`site.data.navigation`, 此时我们修改原来的navigation文件,`_includes/navigation.html`:
```html:_includes/navigation.html
<nav>
  {% for item in site.data.navigation %}
    <a href="{{ item.link }}" {% if page.url == item.link %}style="color: red;"{% endif %}>
      {{ item.name }}
    </a>
  {% endfor %}
</nav>
```

这里我们使用了jekyll中的for in循环语法，大大简化后期导航栏数据量剧增的逻辑

### Assets

此时我们发现样式编辑挺麻烦的，需要对某个链接的tag加入行内样式。但其实，jekyll自带预编译器sass支持，我们可以将所有样式/图片文件/js脚本这里文件放置于assets目录中，如下:

```text:tree.txt
.
├── assets
│   ├── css
│   ├── images
│   └── js
...
```

jekyll 处理sass默认会找`_sass`对应的文件，我们创建`_sass/main.scss`:
```scss
.current {
  color: green;
}
```

在`assets/css/styles.scss` 我们写入:
```scss
--- 
--- 
@import "main";
```

并将样式引入到default布局中去，`_layouts/default.html`:
```html:_layouts/default.html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>{{ page.title }}</title>
    <link rel="stylesheet" href="/assets/css/styles.css" />
  </head>
  <body>
    {% include navigation.html %} {{ content }}
  </body>
</html>
```

此时我们可以应用我们所写的样式到navigation中，`navigation.html`:
```html:_includes/navigation.html
<nav>
  {% for item in site.data.navigation %}
    <a href="{{ item.link }}"{% if page.url == item.link %} class="current"{% endif %}>{{ item.name }}</a>
  {% endfor %}
</nav>
```

页面的聚焦导航链接的样式即变成了绿色

### Blogging

接下来就到我们的博客编写环节，jekyll默认在`_posts`存放所有博客路径，并且以`日期-文件名.md`的默认格式命名来解析博客内容
1. 我们创造一个博客例子, `_posts/2025-01-05-init.md`:
```markdown:_posts/2025-01-05-init.md
---
layout: default
author: Norman Chen
---

This is a test post.
```

2. 增加一个文章的布局，`_layouts/post.html`, 如下：
```html:_layouts/post.html
---
layout: default
---

<h1>{{ page.title }}</h1>
<p>{{ page.date | date_to_string }} - {{ page.author }}</p>

{{ content }}
```
其中 ｜ 类似于Linux里面的管道运算符，将左侧数据通过右侧方法加工处理。 `date_to_string` 是jekyll自带的日期转字符串的方法。

3. 我们在根目录增加一个blog页面来渲染整个博客页面，`blog.md`:
```markdown:blog.md
---
layout: default
title: Blog
---

<h1>Latest Posts</h1>

<ul>
  {% for post in site.posts %}
    <li>
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>
```

其中jekyll通过`site.posts`获取`_posts`中所有文章内容生成一个数组结构，每一项包含：
	1. url标识每个文章地址
	2. title可以自动从文章名中获取
	3. excerpt则截取markdown中第一段的内容

4. 修改data files里面的blog页面配置：
```yaml:_data/navigation.yml
- name: Home
  link: /
- name: About
  link: /about.html
- name: Blog
  link: /blog.html
```

此时我们自己在多创建几个博客页面，就能在项目中的Blog页面中分别看到对应信息了。

### Collections

以上是所有jekyll基础内容，不过从上述内容我们发现，类似于`_layouts`,`_includes`这些都是jekyll指定好了的系统级集合，如果我们需要自己的集合并在页面中使用则需要自己配置.
我们需要在项目根目录中创建一个项目配置文件,比如创建一个作者集合用于标识文章的不同作者：
```yaml:_config.yml
collections:
  authors:
```

改写了根目录的配置需要重启项目，CTRL+C打断项目进程再执行：
```shell
jekyll serve --livereload --port 3456
```

此时我们需要给这个类型加入作者的信息，比如，创建`_authors/jill.md`：
```markdown:_authors/jill.md
---
short_name: jill
name: Jill Smith
position: Chief Editor
---

Jill is an avid fruit grower based in the south of France.
```
以及`_authors/ted.md`:
```markdown:_authors/ted.md
---
short_name: ted
name: Ted Doe
position: Writer
---

Ted has been eating fruit since he was baby.
```


 我们创建一个新的页面来显示全体作者，`staff.md`:
 ```markdown:staff.md
---
layout: default
title: Staff
---

<h1>Staff</h1>

<ul>
  {% for author in site.authors %}
    <li>
	  <h2><a href="{{ author.url }}">{{ author.name }}</a></h2>
      <h2>{{ author.name }}</h2>
      <h3>{{ author.position }}</h3>
      <p>{{ author.content | markdownify }}</p>
    </li>
  {% endfor %}
</ul>
```

这里面使用了管道运算符markdown格式化来处理author的文本信息
从根目录配置的authors可以直接从site通过`site.authors`访问

修改导航配置数据，`_data/navigation.yml`:
```yaml:_data/navigation.yml
- name: Home
  link: /
- name: About
  link: /about.html
- name: Blog
  link: /blog.html
- name: Staff
  link: /staff.html
```

但此时我们发现配置的staff页面没有我们刚刚设置的内容，这是由于在_config.yml配置的集合字段默认不会给页面中展示，所以我们需要修改配置打开它，`_config.yml`：
```yaml:_config.yml
collections:
  authors:
    output: true
```

重启项目就会发现staff页面里面的数据正常显示了
此时点击author链接即可以跳转到对应的author详情页。


我们再给集合加上layout, `_layouts/author.html`:
```html:_layouts/author.html
---
layout: default
---

<h1>{{ page.name }}</h1>
<h2>{{ page.position }}</h2>

{{ content }}
```

每次这种手动添加layout方式很麻烦，更简洁的做法是在全局yml配置文件设置默认配置，如下：
```yaml:_config.yml
collections:
  authors:
    output: true

defaults:
  - scope:
      path: ''
      type: 'authors'
    values:
      layout: 'author'
  - scope:
      path: ''
      type: 'posts'
    values:
      layout: 'post'
  - scope:
      path: ''
    values:
      layout: 'default'
```

重启以后不在需要在页面上配置layout布局 Front Matter了（所有原先在 Front Matter上吗配置的layout均可以移除）


## 2分钟GitHub Pages项目部署

项目部署非常简单,只需要两步

1. 新建github.io项目仓库
	以用户名.gitlab.io的名称创建GitHub仓库（这里由于我创建过所以无法新建仓库）
	![image.png](https://s2.loli.net/2025/01/05/TpCbrkX5wWy7PKj.png)

2. 指定对应的文件夹并保存
![image.png](https://s2.loli.net/2025/01/05/Nb2tOQREG34uIga.png)

当项目在当前所选分支（此为main分支）上并以当前子目录（从图中看出我是将整个项目放置于子目录的docs文件夹中，这个因个人想法而异，当你的git地址与项目重合后可以选择root作为当前分支子文件夹路径，如下图）
![image.png](https://s2.loli.net/2025/01/05/IvSaex4NUbykTHg.png)


此时选择右侧上方Visit site 按钮即可进入到对应链接。

## 5分钟主题选择

配置博客最有成就感的就是主题配置了

这里使用minima这一主题进行配置：

1. 修改原来的依赖安装
```text:Gemfile
source "https://mirrors.tuna.tsinghua.edu.cn/rubygems/"

# Use github-pages gem which will provide the correct Jekyll version
gem "github-pages", "~> 228"
gem "minima", "~> 2.5.1"
gem "faraday-retry"
gem "kramdown-parser-gfm"
gem "webrick"
gem "wdm", ">= 0.1.0" if Gem.win_platform?

group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.12"
  gem "jekyll-seo-tag", "~> 2.8"
end
```

这里一开始只需新增minima依赖但是其又依赖后面几项才能保证稳定运行，所以均加上

2. 在`_config.yml`中进行主题配置
```yaml:_config.yml
domain: ～～～～替换成你的域名～～～～
url: ～～～～替换成你的链接～～～～
baseurl: ""
title: my blog
email: ～～～～替换成你的邮箱～～～～
description: >- # this means to ignore newlines until "baseurl:"
  for blog posts
github_username: jekyll

# Theme settings
theme: minima
plugins:
  - jekyll-feed
  - jekyll-seo-tag

# Minima specific settings
minima:
  date_format: '%b %-d, %Y'
  social_links:
    github: jekyll

# Build settings
sass:
  sass_dir: _sass
```

3. 在css中引入minima样式
```css
---

---

/* Import styles */
@import "minima";
@import "main";
```

等项目部署完成（不到1分钟）即可访问带有主题的链接