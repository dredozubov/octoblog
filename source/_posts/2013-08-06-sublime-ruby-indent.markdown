---
layout: post
title: "Поломанная индентация в Sublime 2 для Ruby"
date: 2013-01-22 17:49
comments: true
categories: sublime ruby
---
<!--more-->

Для того чтобы все инденты стали равны двум пробелам для rb-файлов, открываем какой-нибудь rb-файл и идем в sublime 2 text → preferences → settings(more) → syntax-specific (user) и добавляем туда

{% codeblock lang:json %}

    {
      "tab_size": 2,
      "translate_tabs_to_spaces": true
    }
    
{% endcodeblock %}