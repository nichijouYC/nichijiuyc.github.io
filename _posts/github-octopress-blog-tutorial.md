---
layout: post
title: "GitHub_Octopress_Blog_Tutorial"
date: 2015-07-21 11:32:16 +0800
comments: true
categories: 
---
## 使用Octopress在GitHub上搭建个人Blog

[Octopress首页](http://octopress.org/)

## 步骤
<!-- more -->
- 安装Git([地址](http://git-scm.com/download/)),配置Git与GitHub相连
- 安装Ruby，版本1.9.3以上
    - [Rbenv安装Ruby教程(Linux or OS X)](http://git-scm.com/download/)
    - [RubyInstaller安装Ruby(Win)](http://rubyinstaller.org/)
- 安装ExecJS(Linux)([地址](https://github.com/sstephenson/execjs))，或者安装Devkit(Win)([地址](http://rubyinstaller.org/downloads/)) 由于本机使用的是Win7，所以以下步骤为安装DevKit后的步骤。Linux或OS X系统的设置步骤见[Octopress官网教程](http://octopress.org/docs/setup/)
- 解压DevKit后（注意：解压目录中必须没有中文和空格），在命令行输入以下命令进行安装
    1. cd 进入DevKit目录
    2. `ruby dk.rb init`
    3. `ruby dk.rb install`
- 用Git安装Octopress，在Ruby目录下安装
    1. cd 进入Ruby目录
    2. `git clone git://github.com/imathis/octopress.git octopress`
- 安装依赖工具bundle。由于国内网络原因，可能直接安装会出现错误，建议替换淘宝源可解决([地址](http://ruby.taobao.org/))
    1. cd 进入Octopress目录
    2. `gem install bundle`
    3. `bundle install`（可能提示缺少某些依赖包，按照提示安装缺少的依赖包即可）
- 安装Octopress默认主题
    1. `rake install`
- 支持中文编码（配置环境变量）
    1. `LANG=zh_CN.UTF-8`
    2. `LC_ALL=zh_CN.UTF-8`
- 修改Blog名，作者等配置
    1. 修改`octopress/_config.yml`文件的`url`，`title`，`subtitle`，`author`等值
- 部署和预览（发布新博客或者修改配置后）
    1. `rake generate`
    2. `rake preview`
    3. 本地访问地址：[http://localhost:4000/](http://localhost:4000/)
- 写博客
    1. `rake new_post[“title”]`
    2. 修改`octopress/source/_posts`下的md文件
    3. 重新发布和预览
- 部署在Github上
    1. 在GitHub上新建一个名为`user-id.github.io`的版本库（user-id为你的GitHub用户名）。GitHub会自动为其生成网站地址`user-id.github.io`
    2. 将Octopress目录中的`public`文件夹上传至GitHub版本库中
    3. 访问`user-id.github.io`即可看到生成的博客页面
    4. 如果对页面有修改，先在本地进行部署和预览，然后将`public`文件夹上传即可
- 加快Blog访问速度
    - 由于HTML中的某些资源(字体和脚本)有由google提供的，所以国内访问起来很慢。使用国内的替代品可以解决([360](http://libs.useso.com/))
    - 将`source/_includes/head.html`和`source/_include/custom/head.html`中的`googleapis.com`改成`useso.com` 即可
    - 若不使用Google Analyze的话，可以将`source/_includes/head.html`中的包含google_analytics页面注释掉
- 阅读更多(read on)
    - 正常情况下，Octopress首页会显示每篇blog的全部内容，这样会使得页面很长，不易浏览和阅读早期的blog
    - 我们可以在每篇blog中添加`read on`按钮，使得在首页只显示本篇blog的部分内容，点击`read on`按钮可以进入本blog页面显示全部内容
    - 完成这个效果，只要编辑blog的md文件，在你想要部分显示内容的后面插入`<!-- more -->`即可
- 插入代码，并设置语法高亮
    - 行内代码如`cd ..`,使用两个反引号``包含代码片段
    - 多行代码块，可以使用下图方式插入代码并且实现语法高亮
    {% codeblock %}
{ % codeblock [title] [lang:language] [url] [link text] %}
python code
{ % endcodeblock %}
    {% endcodeblock %}
注意在实际使用中去掉`{ %`中的空格
    - 注意多行代码块的这个语法在普通markdown中不适用。在普通markdown语法中，可以通过```` python`来插入代码块并且设置语法高亮
- 修正列表排版
    - 默认情况下，所有文字的排头会对齐，但是博客内容中的列表头会超出文章的主体区域
    - 修改`octopress/sass/custom/_layout.scss`文件，将`indented-lists: true`去掉注释即可
