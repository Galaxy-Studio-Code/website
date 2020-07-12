---
layout: page
title: About
description: 星际工作室
keywords: 星际工作室
comments: true
menu: 关于
permalink: /about/
---

# 星际工作室

![img](https://raw.githubusercontent.com/Galaxy-Studio-Code/galaxy-studio-code.github.io/master/image1.png)

星际工作室成立于2020年，致力于开发软件。目前星际工作室共14名成员，包括技术部4人，测试部3人，营销部2人，售后部2人，人事部3人。相信未来的星际工作室会越来越强大！若您有好的素材、想法（皆与编程有关），欢迎您与我们交流！

**星光不负赶路人，星际伴你前行！**

**注意事项：** 本工作室开发的软件仅限Windows及Linux系统使用，苹果系列（Unix除外）系统无法使用，敬请谅解！本工作室开发的所有软件均由星际工作室版权所有，请勿抄袭源代码，违者后果自负！本工作室开发的所有软件未经本工作室允许，不得用于商业用途。如有非星际工作室成员给您发来星际工作室的软件，请立即联系我们，并告知其来源，谢谢！若您在使用我们的软件时遇到困难或发现我们软件有漏洞，请尽快反馈给我们，很抱歉给您带来不便。

**欢迎您随时光临星际工作室！**

## 联系

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}
{% if site.url contains 'mazhuang.org' %}
<li>
</li>
{% endif %}
</ul>


## Skill Keywords

{% for skill in site.data.skills %}
### {{ skill.name }}
<div class="btn-inline">
{% for keyword in skill.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
