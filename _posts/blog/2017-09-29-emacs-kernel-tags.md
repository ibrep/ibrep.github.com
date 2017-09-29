---
layout: post
title:  "使用TAGS帮助阅读源码"
date:   2017-09-29 07:21:25 +0800
categories: blog
---
# 生成tags文件
为Emacs生成tags文件要用大写的TAGS：

	make TAGS

为Vim生成tags文件用小写的tags：

	make tags
	
# Emacs中的tags操作
选择tags文件：

	M-x visit-tags-table

查找tag：

	M-x find-tag    (M-.)
	
继续查找下一个tag：

	C-u M-.

返回：

	M-*

# Vim中的tags操作
加载tags文件：

	:set tags=../../ctags
	
跳转到指定的标签：

	:tag start_kernel
	
跳转到光标下单词的标签：

	CTRL-]
	
跳转回上一个标签：

	CTRL-O
	
如果有多个相同的标签，那么：

列出所有标签供选择

	:tselect

跳到第一个标签

	:tfirst 或者 :trewind 

跳到最后一个标签 
	
	:tlast 

跳到下一个匹配的标签
	
	:tnext 

跳到前一个匹配的标签 

	:tprevious 或者 :tNext 

使用预览窗格：

| 命令         | 短命令      |      描述            |
|--------------|-------------|----------------------|
| :ptag [tag]  | :pta        | 在预览窗格中打开标签 |
| CTRL-W }	   |             | 打开光标下的标签     |
| :ptnext	   | :ptn        | 跳到下一个匹配的标签 |
| :ptprevious  | :ptp        | 跳到上一个匹配的标签 |
| :pclose      | :pc         | 关闭预览窗格         |
| CTRL-W z	   |             | 关闭预览窗格         |



	
	
