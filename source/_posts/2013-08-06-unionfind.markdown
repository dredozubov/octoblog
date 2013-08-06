---
layout: post
title: "Структуры данных: непересекающиеся множества и Union-Find"
date: 2013-02-07 23:50
comments: true
categories: алгоритмы python unionfind структуры данных
---
C union-find я столкнулся проходя вторую часть стэнфордских [алгоритмических курсов](https://www.coursera.org/course/algo2) на [coursera](https://www.coursera.org).

Так вот - графы. Пойду в объяснении тем же путем, которым объясняли мне, через применение для реализации конечного алгоритма: у нас на входе есть ненаправленный граф и мы ищем его [Минимальное Остовное Дерево](http://ru.wikipedia.org/wiki/%D0%9C%D0%B8%D0%BD%D0%B8%D0%BC%D0%B0%D0%BB%D1%8C%D0%BD%D0%BE%D0%B5_%D0%BE%D1%81%D1%82%D0%BE%D0%B2%D0%BD%D0%BE%D0%B5_%D0%B4%D0%B5%D1%80%D0%B5%D0%B2%D0%BE) [(Minimal Spanning Tree - MST)](http://en.wikipedia.org/wiki/Minimum_spanning_tree). Два самых ходовых алгоритма: алгоритм Прима и алгоритм Крускала. Рассмотрим реализацию последнего.

<!--more-->


На входе имеем взвешенный ненаправленный связный граф G c множеством вершин V и множеством рёбер E. T - посещенные нашим алгоритмом вершины. В самом начале T = ∅. Сортируем рёбра по возрастанию весов. Во время каждого прохода алгоритма мы берем ребро с минимальным весом, при условии, что оно не добавляет цикл в наше дерево(по определению минимального остовного дерева). Добавляем посещенные вершины из V - T в T. Завершаем в тот момент, когда все вершины из V, будут принадлежать нашему остовному дереву.


<!--![](../images/unionfind/unionfind_kruskal1.png)
![](../images/unionfind/unionfind_kruskal2.png)
![](../images/unionfind/unionfind_kruskal3.png)
![](../images/unionfind/unionfind_kruskal4.png)
![](../images/unionfind/unionfind_kruskal5.png)
![](../images/unionfind/unionfind_sets.png)
-->
{% img /images/unionfind/unionfind_kruskal1.png %}
{% img /images/unionfind/unionfind_kruskal2.png %}
{% img /images/unionfind/unionfind_kruskal3.png %}
{% img /images/unionfind/unionfind_kruskal4.png %}
{% img /images/unionfind/unionfind_kruskal5.png %}
{% img /images/unionfind/unionfind_sets.png %}


Проблема возникает на момент проверки графа на ацикличность. Сразу же стоит явно упомянуть о том, что в процессе выполнения алгоритма Крускала мы, как правило, не имеем связного единого связного подграфа - мы выстраиваем множество подграфов из самых дешевых рёбер, которые, в процессе, работы алгоритма соединяются в один. Стало быть, если одна вершина проверяемого на каждом этапе ребра, принадлежит одному множеству, а другая - другому, то текущее ребро соединит их и в подграфе не будет цикла. Если же обе вершины этого ребра принадлежат одному множеству, значит это ребро создаст цикл и его необходимо пропустить. Лобовое решение этой проблемы даст сложность O(n), повышая вычислительную сложность реализации алгоритма Крускала до близкой к квадратичной, O(mn), где m - число ребер графа G, n - число вершин.


Эту задачу и помогает решать union-find, понижая сложность проверки цикличности до постоянного времени 0(1), а сложность всего алгоритма до O(mlogn) - в этом случае узким местом становится препроцессинг(сортировка).


Итак, union-find это структура данных, содержащая непересекающиеся множества и, позволяющая узнать за постоянное время к какому из множеств принадлежит вершина.


Основных методов несколько:


**union(vertice1, vertice2)** - Объединение нескольких множеств в одно. Почему вместо множеств входными аргументами выступают вершины, будет объяснено ниже.


**find(vertice)** - Узнать к какому из множеств относится текущая вершина


**sameset(vertice1, vertice2)** - Добавлен для наглядности и очевидности, зачастую можно встретить реализации, в которых этот метод называется find, на мой взгляд это не очень хорошее имя для этого метода.


**makeset(vertice)** - Добавление еще одного множества из одной вершины vertice.


Итак, как же мы будем находить искомое множество за константное время? Секрет этой структуры данных в том, что каждая вершина каждого подграфа указывает на лидера - одну из вершин подграфа, в который она входит. В каждый момент времени лидер у всех вершин одного подграфа один, но может меняться после операций слияния множеств. Таким образом find и sameset становятся O(1)-операциями. union же несколько сложнее. При слиянии множеств, нужно быть уверенным что, после его завершения, все вершины нового подграфа, входящего в G, будут указывать на одну вершину-лидера. В голову сразу же приходит очевидная оптимизация: изменять лидера необходимо только в вершинах меньшего из двух входных множеств. Как мы узнаем какое из множеств меньше? Очень просто - следует вести отдельные счетчики.


Моя простая реализация на Python:

{% codeblock lang:python %}

    class Unionfind(object):

        def __init__(self, iter_obj):
            self.by_leader = {}
            self.leaders = {}
            self.setscount = 0
            # values key: obj, values value: leader
            if iter_obj:
                # obj items must be unique
                for x in set(iter_obj):
                    self.makeset(x)

        def find(self, obj):
            val = self.leaders.get(obj)
            return self.leaders[val] if val else None

        def union(self, obj1, obj2):
            leader1 = self.leaders[obj1]
            leader2 = self.leaders[obj2]
            if len(self.by_leader[leader1]) > len(self.by_leader[leader2]):
                new_leader = leader1
                to_remove = leader2
            else:
                new_leader = leader2
                to_remove = leader1

            extra = self.by_leader.pop(to_remove)
            self.by_leader[new_leader] = self.by_leader[new_leader] + extra

            for e in extra:
                self.leaders[e] = new_leader

            self.setscount -= 1

        def makeset(self, obj):
            self.leaders[obj] = obj
            self.by_leader[obj] = [obj]
            self.setscount += 1

        def sameset(self, obj1, obj2):
            return self.find(obj1) == self.find(obj2)

        def __unicode__(self):
            return u'Unionfind object: %s' % repr(self.leaders)[:100]

        def __str__(self):
            return self.__unicode__()

{% endcodeblock %}


Для желающих ознакомиться с реализацией на C++ - рекомендую посмотреть на [эту](http://www.boost.org/doc/libs/1_52_0/libs/graph/doc/incremental_components.html) страничку с сайта boost.
