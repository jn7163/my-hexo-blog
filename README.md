[![][my_pic]][my_url]
[my_pic]: /source/images/default_avatar.jpg
[my_url]: http://lixingcong.github.io


# Usage

## 安装git

[github for windows](https://windows.github.com)

或者使用绿色版git

## 安装node.js

[nodejs.org](http://nodejs.org)


## 创建你的博客文件夹

### 若是全新的仓库：

	mkdir my_hexo_blog.git

### 若是克隆回来的仓库

	git clone https://github.com/lixingcong/my_hexo_blog.git


## 安装hexo并初始化

打开git-bash

	cd my_hexo_blog.git
	npm install -g hexo-cli
	npm install hexo --save
	hexo init（如果是git clone目录，请不要初始化，防止删掉_config.yml配置文件）

## 安装插件
可以用一句命令安装：

	npm install hexo-generator-index hexo-generator-archive hexo-generator-category hexo-generator-tag hexo-server hexo-deployer-git hexo-deployer-heroku hexo-deployer-rsync hexo-deployer-openshift hexo-renderer-marked@0.2 hexo-renderer-stylus@0.2 hexo-generator-feed@1 hexo-generator-sitemap@1 --save

## 开启本地预览

	hexo s

浏览器打开localhost:4000是否正常运行！！

按ctrl_c退出

## 修改配置文件

这里不详述，自己爬文搜索怎么修改配置，毕竟太难总结，我只给出文件位置

hexo总配置文件：my_hexo_blog\_config.yml

主题文件\theme\xxxx\_config.yml

## 使用Haroopress(推荐)或者MarkdownPad书写你的博客

[Haroopress for Linux/Win](http://pad.haroopress.com/)

[MarkdownPad for Windows](http://markdownpad.com/)

[Markdown语法学习](https://www.zybuluo.com/AntLog/note/63228)

日志位置在\source\_posts\，格式为md文件

建议保存为英文名字的文件，有时候浏览器中显示地址很难看

## 生成静态博客

	hexo g

## 打开本地预览

	hexo s

## 提交到Git仓库

注意只需要提交\public目录下所有文件

	mv public D:\lixingcong.github.io
	cd d:\lixingcong.github.io

### 若是新仓库，需要新建仓库

参考官方帮助：

添加说明文件

首先新建一个txt文档，改名为README.md，然后添加一行注释

	echo # test my hexo blog >> README.md

初始化仓库

	git init

添加所有文件，跟踪所有文件

	git add *

添加commit

	git commit -m "version 1.0"

添加到远程仓库

	git remote add origin https://github.com/lixingcong/lixingcong.github.io
	git push -u origin master

### 若是已创建仓库

查看修改状态
	
	git status

注意将红色文件添加上

可以全部添加跟踪，并且commit

	git commit -a -m "version 1.0"

推送到远处

	git push

# Attention

在文件系统为NTFS驱动器下，用windows资源管理器中对my_hexo_blog目录进行拷贝或者zip打包，会出现“文件名太长”的提示。无法进行拷贝某些文件。

![](/source/images/readme/error_ntfs.png)

此时在Git_bash里面进行cp即可

	cp -R my_hexo_blog/ ./Copy_hexo_blog

同理，在bash用tar打包应该没问题，我没试过，大家有空试一下。

# 更新日志

参见[Changelog.md](/ChangeLog.md)
