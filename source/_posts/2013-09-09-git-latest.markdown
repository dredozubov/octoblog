---
layout: post
title: "git latest - Последние обновленные бранчи в git"
date: 2013-09-09 23:57
comments: true
categories: [git,programming]
keywords: git programming 
description: git latest - последние обновленные бранчи в git

---


Сегодня в интернете проскочил прекрасный алиас для git, позволяющий показать последние обновленные бранчи:

<!--more-->

{% codeblock %}

    git config --global alias.latest "for-each-ref --count=10 --sort=-committerdate --format='%(committerdate:short) %(refname:short)'"

{% endcodeblock %}

Для того чтобы заменить дату человекочитаемым форматом в духе "N days ago", меняем формат на %(commiterdate:relative)

{% codeblock %}

    git config --global alias.latest "for-each-ref --count=10 --sort=-committerdate --format='%(committerdate:relative) %(refname:short)'"

{% endcodeblock %}

git productive!