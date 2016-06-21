---
title: Hexo常用命令  
date: 2016-06-17 19:16:03
tags: 命令行
---

1. 安装Hexo  
使用npm进行Hexo安装  

		npm install -g hexo-cli

2. 初始化框架  

		hexo init { yourFolder }
		cd { yourFolder }
		npm install

<!--more-->

3. 新建文章（创建一个Hello World）

		hexo new "Hello World"  
在 { yourFolder }/source/_post 里添加 hello-world.md 文件，之后新建的文章都将存放在此目录下。

4. 转码成对应的网站

		hexo generate
		//可简写成  
		hexo g
		
5. 本机浏览

		hexo server
		//可自定义端口号
		hexo server -p 3000
		//可简写成
		hexo s
		
6. 部署网站（上传至git）

		hexo deploy
		//可简写成
		hexo d
		//步骤4和6可合并成一行命令
		hexo generate -d
部署的配置在{yourFolder}/_config.yml文件里，最后一行

		# Deployment
		## Docs: https://hexo.io/docs/deployment.html
		deploy:
		  type: git
		  repo: https://github.com/remotesc2/remotesc2.github.io.git
		  branch: master
		  
7. 主题  
官方提供一系列[主题](https://hexo.io/themes)可供选择，使用一下命令进行下载

		git clone https://github.com/iissnan/hexo-theme-next themes/next
在{yourFolder}/_config.yml进行主题配置  
		
		# Extensions
		## Plugins: https://hexo.io/plugins/
		## Themes: https://hexo.io/themes/
		theme: next
		  
8. 如果碰到明明修改了文件的代码，然而没有生效
此时需要先进行一次清理，再重新生成

		hexo clean

9. hexo的更新

		npm update -g hexo