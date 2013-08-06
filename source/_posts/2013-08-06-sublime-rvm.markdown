---
layout: post
title: "Sublime с нормальной поддержкой rvm"
date: 2013-08-06 21:25
comments: true
categories: ruby sublime rvm
---
Первым делом устанавливаем rvm:

	$ brew install rvm

<!--more-->

Добавляем в загрузку rvm в .zshrc/.bashrc:

{% codeblock lang:sh %}

	# rvm 
	if [ -f $HOME/.rvm/scripts/rvm ]
	then
	  source $HOME/.rvm/scripts/rvm
	fi
	
{% endcodeblock %}


Со скриптом rvm-auto-ruby из стандартной поставки rmv sublime работал следующим образом: использовалась дефолтная версия ruby, игнорировались "попроектные" .rvmrc в вышележащих директориях. В общем, использую другой, найденный в интернете.
Создаем скрипт для корректной проверки .rvmrc(за него спасибо [автору](http://upstre.am/blog/2011/07/sublime-text-2-with-rvm-on-osx/)) в файле ~/bin/rvm_ruby:

{% codeblock lang:ruby %}

	#!/usr/bin/env ruby

	file = File.expand_path(
		ARGV[0] || 
		(STDERR.puts('you must specify a ruby file');
		 exit(-1))
		)
	dir = File.dirname file
	while dir.size > 1
	  if File.exist?(dir + '/.rvmrc')
	    exec("source \"$HOME/.rvm/scripts/rvm\"; cd #{dir}; ruby #{file}")
	  else
	    dir = dir.sub(/\/[^\/]*$/, '')
	  end
	end

	puts "Could not find any .rvmrc above #{file}"
	exit -1

{% endcodeblock %}

Делаем его исполняемым:

	$ chmod +x rvm_ruby


Изменяем Ruby.sublime-build(открыть директорию в finder можно быстро так: sublime-preferences-browse packages) следующим образом для работы через наш новый скрипт:

{% codeblock lang:json %}

	{
	    "cmd": ["/Users/<someninjausername>/bin/rvm_ruby", "$file"],
	    "file_regex": "^(...*?):([0-9]*):?([0-9]*)",
	    "selector": "source.ruby"
	}

{% endcodeblock %}

**Не забудьте изменить юзернейм на свой!**


Проверяем:

	$ ruby -v
	ruby 1.9.3p0 (2011-10-30 revision 33570) [x86_64-darwin12.2.0]
	$ cd ninjapower
	$ ruby -v
	Using: /Users/dr/.rvm/gems/ruby-1.9.2-p290@ninjapower
	$ ls
	shuriken.rb
	$ cat shuriken.rb
	puts "ninjapower!"
	puts RUBY_VERSION%


Открываем shuriken.rb в sublime, нажимаем ⌘ + B:

	Using: /Users/dr/.rvm/gems/ruby-1.9.2-p290@ninjapower
	ninjapower!
	1.9.2
	[Finished in 0.2s]


