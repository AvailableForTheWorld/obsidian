
# what's jekyll

jekyll是一个集成在[GitHub Pages](https://docs.github.com/zh/pages)中的静态站点生成器，通过极其简单的语法和规则构建整个静态站点，作为一个想要免费使用GitHub Pages搭建博客的人来说，jekyll是不二之选。

# why jekyll

1. Github Pages 的内置集成，官方推荐使用jekyll作为GitHub Pages的静态站点生成器
2. 不用使用繁杂的Github Actions即可自动触发GitHub Pages的CICD
3. 对比其他生成器，轻量，学习成本非常低，是最有性价比的博客生成器
4. 以markdown为内容载体
5. 丰富的主题样式可供选择，极简配置主题，无需你手动写样式
6. ruby的配置模式以及工程化对程序员友好

# 30分钟指南

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

## 5分钟项目搭建

