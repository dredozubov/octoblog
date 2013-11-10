---
layout: post
title: "установка gem-а pg на OS X 10.9 Mavericks"
date: 2013-11-10 20:30
comments: true
categories: ruby rails gem install pg postgresql bundler osx maverics

---
После обновления OS X до 10.9 Mavericks у меня возникли сложности с установкой gem'а pg. Кратко пересказывая сообщение об ошибке - проблема заключалась в отсутствии залоговочного файла из libpq(уже не могу найти точное сообщение из .gem/ruby/2.0.0/gems/pg-0.15.1/ext/gem_make.out, за его отсутствием).


<!--more-->

{% codeblock lang:bash %}
brew install libpqxx
==> Downloading http://pqxx.org/download/software/libpqxx/libpqxx-4.0.tar.gz
Already downloaded: /Library/Caches/Homebrew/libpqxx-4.0.tar.gz
==> Patching
patching file tools/maketemporary
patching file tools/splitconfig
patching file configure
==> ./configure --prefix=/usr/local/Cellar/libpqxx/4.0 --enable-shared
==> make install
libtool: compile:  clang++ -DHAVE_CONFIG_H -I../include -I../include -I/usr/local/Cellar/postgresql/9.3.1/include -g -O2 -MT nontransaction.lo -MD -MP -MF .deps/nontransaction.Tpo -c nontransaction.cxx  -fno-common -DPIC -o .libs/nontransaction.o
mv -f .deps/field.Tpo .deps/field.Plo
libtool: compile:  clang++ -DHAVE_CONFIG_H -I../include -I../include -I/usr/local/Cellar/postgresql/9.3.1/include -g -O2 -MT nontransaction.lo -MD -MP -MF .deps/nontransaction.Tpo -c nontransaction.cxx -o nontransaction.o >/dev/null 2>&1
mv -f .deps/nontransaction.Tpo .deps/nontransaction.Plo
make: *** [install-recursive] Error 1
{% endcodeblock %}

Впрочем, есть еще один обходной способ - можно использовать pg_config для конфигурации гема(требует установленного postgresql):

{% codeblock lang:bash %}
gem install pg --with-pg-config=/usr/local/Cellar/postgresql/9.3.1/bin/pg_config
{% endcodeblock %}

или

{% codeblock lang:bash %}
bundle config build.pg --with-pg-config=/usr/local/Cellar/postgresql/9.3.1/bin/pg_config
gem install pg
{% endcodeblock %}

Путь к pg_config может отличаться, locate/which вам в помощь.