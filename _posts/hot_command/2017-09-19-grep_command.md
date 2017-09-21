---
layout: post
title:  grep 命令
date:   2017-09-19 01:08:00 +0800
categories: 常用命令
tag: grep 命令
---

* content
{:toc}



# grep 命令搜索中文  utf-8编码
搜索   "正在启动"

_$ grep -rnP "\xE6\xAD\xA3\xE5\x9C\xA8\xE5\x90\xAF\xE5\x8A\xA8" *_

[中文utf-8编码转换网址](http://www.mytju.com/classcode/tools/encode_utf8.asp)

例如：替换/home下所有文件中的www.admin99.net为admin99.net Bbs.Svn8.Com
　　sed -i "s/www.admin99.net/admin99.net/g" \`grep www.admin99.net -rl /home\`
