---
layout: post
title: "Octave под OS X Mountain Lion"
date: 2013-04-30 23:06
comments: true
categories: octave, osx
---
<!--more-->
Собственно проблема возникала, когда я хотел нарисовать какой-либо график:
  

	>> t = [0:0.01:0.99];
	>> hist(sin(3*pi*t),t)
	gnuplot> set terminal aqua enhanced title "Figure 1"  font "*,6"
	                      ^
	         line 0: unknown or ambiguous terminal type; type just 'set terminal' ...


Лечится(при наличии установленного X11) добавлением в octaverc:


	setenv("GNUTERM","X11")


А так можно поменять приглашение в REPL'e:


    PS1(">> ")