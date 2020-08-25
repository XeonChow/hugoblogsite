---
title: "关于本站评论系统 Valine 的说明"
date: 2020-05-06T13:14:44+08:00
slug: valine-guide
tags: ["blog"]
indent: false
dropCap: false
toc: false
mathjax: false
---

随着从 Hexo 迁移到 Hugo，本站的评论系统也从 [Disqus](https://disqus.com/) 改为 [Valine](https://valine.js.org/)。

<!--more-->

## Why Valine

之所以选择 Valine，是因为它是一款无后端的评论系统，这意味着不同于 Disqus 和 [utterances](https://github.com/utterance/utterances) 等其他评论系统，Valine ..无需任何注册和登陆操作..即可进行评论，这可以大大减少用户进行评论的成本，提高博客的互动率。

## 如何评论

使用 Valine 进行评论十分方便，你只需要填入以下内容：

|    昵称    |       邮箱       |                 网址                  |
| :--------: | :--------------: | :-----------------------------------: |
| 可任意填写 | 填写个人有效邮箱 | 填写个人有效网址，博客或 SNS 主页均可 |

甚至，你可以不填写以上任何一项个人信息，进行匿名评论。但..强烈建议填入个人有效邮箱..，这样当你的评论被回复时，你将及时收到一封提醒邮件。

由于 Valine 的特性，你将无法自行删除自己的评论，如果有需要，可以给我[发送邮件](mailto:enitxeon@gmail.com)进行删除。你也无需担心隐私被泄露，网页的评论区不会显示你的邮箱，并且你完全可以选择匿名评论。

Valine 支持 Markdown，Markdown 是一种非常流行且方便的轻量级标记语言，如果你还不知道如何使用 Markdown，[这里](https://sspai.com/post/25137)是一份简单的语法参考。你也可以无视 Markdown 的特性，直接撰写评论即可。

Valine 使用的是 [Gravatar](http://cn.gravatar.com/) 作为评论列表头像，登录或注册 [Gravatar](http://cn.gravatar.com/)，然后修改自己的头像，评论的时候，留下在 Gravatar 注册时所使用的邮箱即可。

## 关于邮件提醒

由于 [Valine 官方的邮件提醒功能已经关闭](https://valine.js.org/notify.html)，本站采用 [赵俊](https://github.com/zhaojun1998) 大大开发的 [Valine 邮件提醒扩展应用](https://github.com/zhaojun1998/Valine-Admin)，详细配置教程可参考 [赵俊的博客](http://www.zhaojun.im/hexo-valine-admin/)。

该应用可以实现：

* 当你评论文章时，站长（也就是我）会收到提醒邮件，提醒邮件长这样：
  <img src="/valine-guide/comment.png" width="500"  title="新评论提醒邮件">
* 当有人回复你的评论时，如果你填入了有效的邮件地址，则你也会收到如下提醒邮件：
  <img src="/valine-guide/reply.png" width="500"  title="新回复提醒邮件">

邮件的服务器是我自己的 Hotmail 邮箱，理论上直接回复提醒邮件，我也能收到，但依然建议你点击邮件中的链接，回到博客评论区进行回复操作。不清楚微软对于第三方应用的自动邮件发送数量是否有限制，但基于本站几乎无人浏览的现实，我想短期内不会达到该限制。

由于系统不会进行个人身份验证，你可以填写虚假的个人邮箱地址、或填写别人的邮箱地址进行评论，但..强烈不建议这样做..，这不仅会对别人造成骚扰，也毫无意义。

如果你发现你的评论被回复、但没有收到邮件提醒，请检查是否填写了正确的个人邮箱地址，以及检查一下邮箱的垃圾箱。如果依然没有收到回复提醒邮件，可以给我[发送邮件](mailto:enitxeon@gmail.com)进行反馈。

最后，本站欢迎所有真实的评论，你的评论我也会及时回复。

希望大家多多回复，大家一起愉快地交流吧~