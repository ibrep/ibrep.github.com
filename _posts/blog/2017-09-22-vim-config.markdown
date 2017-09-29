---
layout: post
title:  "Vim常用配置"
date:   2017-09-22 19:49:25 +0800
categories: blog 
---
注释用双引号开头：

	" This is a comment line

非兼容模式：

	set nocp

显示行号：

	set nu

搜索不区分大小写：

	set ignorecase

自动折行显示： 

	set linebreak

语法高量： 

	syntax on

自动缩进： 

	set autoindent

按文件类型缩进：

	filetype plugin indent on

函数参数对齐：

	set cinoptions+=(0 或者
	set cino+=(0

