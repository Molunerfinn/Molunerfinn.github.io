---
title: 网站更新日志-5
tags:
  - 更新日志
id: 254
categories:
  - WordPress日常
  - 日志
date: 2015-06-18 01:44:58
---

##说在前头
这个主题原本是Moyu的[EverBox](http://demo.20theme.com/everbox-cn/ "http://demo.20theme.com/everbox-cn/")，后来经过修改，二次开发后我将其命名为Mofinn，并会一直更新。
Mofinn的Github地址在[这里](https://github.com/Molunerfinn/Mofinn "https://github.com/Molunerfinn/Mofinn")
目前已经更新至v1.0PROA6

* * *

> 6.18更新日志

##更新说明
更新了日历；增加了回复评论能够E-mail通知评论者的功能。

###更新效果
- 日历下方左右各更新一个按钮，用于查看上一个/下一个有文章更新的月份。
- 稍微修改了日历头部的颜色显示效果。现在鼠标滑动日历头部将能看到颜色变化的效果。
- 增加了回复评论能够E-mail通知评论者的功能。目前功能还算完善。一个评论下最多能重叠4层回复。

###不足以及有待改进之处

首先仍然是手机端的导航菜单栏的问题还没有解决。其次考虑将E-mail回复评论者的功能的代码单独做成一个PHP，然后在Functions里调用它就行，让Functions里保持代码一致性。

* * *

> 以上是主题文件更新
>   以下是非主题文件更新

将WP-SUPER-CACHE插件删去。删去插件后发现并没有影响网站的访问速度。原因是今天在刷新wscache的时候出错，让网站白屏了。让我困惑不已。进后台将其删去后一切正常。至此考虑将不会再装回。