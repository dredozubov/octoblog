---
layout: post
title: "coffeescript-unicode-tilt"
date: 2013-05-14 23:05
comments: true
categories: ruby coffeescript tilt sinatra
---
Столкнулся на днях со следующей проблемой - при попытке скомпилить coffeescript с кириллицей ломался рубишный [tilt](https://github.com/rtomayko/tilt). 

<!-- more -->

Encoding::UndefinedConversionError - "\xD0" from ASCII-8BIT to UTF-8 и далее здоровенный трейсбек. Лечится это вот таким манки-патчем:
    
    module Tilt
	  class CoffeeScriptTemplate
	    def prepare
	      @data.force_encoding Encoding.default_external
	      if !options.key?(:bare) and !options.key?(:no_wrap)
	        options[:bare] = self.class.default_bare
	      end
	    end
	  end
	end


[Источник.](http://stackoverflow.com/questions/10828668/padrino-sass-coffee-encodingundefinedconversionerror-from-ascii-8bit-to)