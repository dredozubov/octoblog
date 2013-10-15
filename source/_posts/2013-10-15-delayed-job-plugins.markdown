---
layout: post
title: "Расширение delayed_job и метрики для riemann и graphite."
date: 2013-10-15 18:21
comments: true
categories: ruby rails delayed_job

---

В [eviterra](https://eviterra.com) мы используем delayed_job(аналог resque/sidekiq/etc) для асинхронной обработки задач. Давно хотелось как-нибудь мониторить, насколько быстро начинают выполняться задачи. В перспективе это может помочь распределять задачи по воркерам и выставлять приоритеты не только по наитию.

<!--more-->

Итак, что мы имеем: для агрегации данных у нас есть [riemann](http://riemann.io/) и [graphite](http://graphite.wikidot.com/) для их визуализации. В итоге расклад такой: если передавать дату создания таска(джоба) минус дату начала его выполнения, мы получим задержку в текущей очереди. Рисовать это можно либо по очередям, либо по типу джобов - я выбрал второе.

delayed_job предлагает несколько стандартных механизмов для расширения себя:

* Коллбеки, выполняемые в следующем порядке:

    1. before
    2. perform
    3. success/error (в зависимости от успешности выполнения джоба)
    4. after
    5. failure
  
    Очевидное решение - создаем модуль, определяющий нужные нам коллбеки.

  
    {% codeblock lang:ruby %}

        module RiemannMixin
          def before(job)
            job.instance_eval do
              delay_time = (Time.now - @attributes["created_at"]).to_f
              @attributes['handler'] =~ /\/object:(\w+)/
              klass = $1
              # Monitoring.gauge - мой враппер для riemann-client, добавляющий необходимые тэги и прочее
              Monitoring.gauge(
                service: "jobs.startup_time.#{klass.downcase}",
                metric: delay_time
              )
              Rails.logger.info "#{klass} started! Waited for #{delay_time}"
            end
          end
        end


    {% endcodeblock %}

В идеале - используем alias или alias_method для того чтобы обернуть существующие коллбеки нашими и ничего не потерять - для моего случая это не нужно.
Остается лишь include-нуть модуль в нужный джоб. Такой подход хорош для per-job плагинов, т.к. можно использовать или игнорировать плагин в зависимости от класса джоба.

* Глобальные плагины для DJ, выполняемые всеми джобами.

{% codeblock lang:ruby %}

    class RiemannPlugin < Delayed::Plugin
      callbacks do |lifecycle|
        lifecycle.before(:invoke_job) do |job, *args, &block|

          job.instance_eval do
            delay_time = (Time.now - @attributes["created_at"]).to_f
            @attributes['handler'] =~ /\/object:(\w+)/
            klass = $1
            Monitoring.gauge(
              service: "jobs.startup_time.#{klass.downcase}",
              metric: delay_time
            )
            Rails.logger.info "#{klass} started! Waited for #{delay_time}"
          end

          # следующий коллбек в цепочке
          block.call(job, *args) if block.present?
        end
      end
    end


{% endcodeblock %}

т.к. я хочу замерять задержку при выполнении абсолютно всех тасков, мне нравится этот подход. Дополнительные include-ы в таски также не требуются. Впрочем, требуется отдельный инициализатор для rails(если вы юзаете DJ, все шансы что у вас уже он есть).

{% codeblock lang:ruby %}

Delayed::Worker.plugins << RiemannPlugin

{% endcodeblock %}


