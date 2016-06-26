---
layout: post
title: Universal Links(通用链接)
date: 2016-02-26 22:32:24.000000000 +09:00
---
#引言
&#160; &#160; &#160; &#160;本文主要讲述iOS9新加入的一项技术-Universal Links，首先介绍什么是Universal Links，它的作用是什么。然后用介绍如何使用Universal Links。最后指出在使用Universal Links需要注意的问题，以及我在项目中碰到的坑。

#什么是Universal Links
&#160; &#160; &#160; &#160;做app开发的同学肯定碰到过这样的需求：在h5中点击一个链接或者按钮可以打开app，这个在Safari里直接使用url schema就可以搞定，很简单。那么问题来了，如果这个h5页面是在微信浏览器里打开的，怎么办？大家都知道，微信浏览器是把url schema屏蔽掉了的，也就是说在微信浏览器里h5是不能直接打开我们的app的(腾讯自家的app除外)。既然urlschema不能用，那有别的解决方案吗？答案就是Universal Links。

&#160; &#160; &#160; &#160;Universal Links是苹果在iOS9之后推出一项新的链接技术，它能够通过传统的http链接来启动app，使用相同的链接来打开网页和app。使用这个唯一的链接，可以调到对应的网页或者是app对应的页面，试想一下，如果你手机上没有安装app，点击这个链接，就打开对应h5页面，如果安装了app，就进到app相应的页面，最重要的是这个功能微信是没有禁掉的，也就是可以解决上面的需求(仅限于iOS9以上的系统)。

#如何使用Universal Links
......

#使用Universal Links需要注意的问题
......























