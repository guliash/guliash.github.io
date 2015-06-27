---
layout: post
title:  "Особенности использования TreeSet в Java"
date:   2015-06-27 20:45:00
categories: java
---
Сегодня, решая очередной раунд [CF](http://codeforces.ru/), я столкнулся с интересной [задачей](http://codeforces.com/contest/555/problem/B). Для решения этой задачи можно использовать следующий жадный алгоритм:

1. Отсортируем каждую пару соседних островов по расстоянию между ними в порядке невозрастания
2. Заведём Красно-чёрное дерево, в которое положим длины мостов
3. Будем жадно пытаться каждой паре сопоставить мост который максимально её покрывает

Не будем останавливаться на доказательстве корректности этого алгоритма.

Видно, что расстояния между мостами могут быть числами большими, поэтому нельзя использовать массивы, листы и т.п..
Придётся использовать Set или Map. Для этого нам понадобится или TreeSet или TreeMap, которые представляют реализацию RB-tree в Java.

Ну казалось бы дальше дело техники. Добавим мосты в TreeSet, используя метод `add`. Дальше получаем наиболее подходящий с помощью `floor`. А потом удаляем его с помощью `remove`. Всё просто подумал я. Но у меня не получилось пройти тесты.

Вот код заполнения TreeSet мостами:

{% highlight java %}

class Bridge {
  int pos;
  long len;
  public Bridge(int pos, long len) {
    this.pos = pos;
    this.len = len;
  }
}
....
....
....
Comparator<Bridge> cmp = new Comparator<Bridge>() {
    public int compare(Bridge o1, Bridge o2) {
        if(o1.len < o2.len) {
            return -1;
        }
        if(o1.len > o2.len) {
            return 1;
        }
        return 0;
    }
};
TreeSet<Bridge> treeSet = new TreeSet<Bridge>(cmp);
for(int i = 0; i < m; i++) {
    treeSet.add(bridges[i]);
}
{% endhighlight %}

Итак. В чём проблема? Заглядывая в [javadocs](https://docs.oracle.com/javase/7/docs/api/java/util/TreeSet.html) видим какой-то [natural ordering](https://docs.oracle.com/javase/7/docs/api/java/lang/Comparable.html).  Особенно нас интересует следующее:

>It is strongly recommended (though not required) that natural orderings be consistent with equals

Хм. А что же значит `to be consistent with equals`?

>The natural ordering for a class C is said to be consistent with equals if and only if e1.compareTo(e2) == 0 has the same boolean value as e1.equals(e2) for every e1 and e2 of class C.

Иными словами выше говорится, что это свойство, заключается в том, что если два объекта равны по компаратору, то они также должны быть равны по equals.

>It is strongly recommended (though not required) that natural orderings be consistent with equals. This is so because sorted sets (and sorted maps) without explicit comparators behave "strangely" when they are used with elements (or keys) whose natural ordering is inconsistent with equals. In particular, such a sorted set (or sorted map) violates the general contract for set (or map), which is defined in terms of the equals method.

Во всякого рода HashMap, HashSet, ArrayList и т.д., свойство консистентности компаратора и equals необязательно должно выполняться, ибо удаление и вставка в данных структурах не зависят от компаратора. **НО** в древоидных структурах это свойство необходимо, так как сравнение объектов на равенство там происходит только лишь по компаратору.

Возвращаясь к задаче. Мосты могут иметь одинаковую длину. Таким образом они одинаковы по компаратору, но очевидно не равны по equals. **НО** `TreeSet` смотрит лишь только на **компаратор**. И метод add, видя, что уже есть мост у которого такая же длина, не добавляет новый мост в дерево:

>For example, if one adds two keys a and b such that (!a.equals(b) && a.compareTo(b) == 0) to a sorted set that does not use an explicit comparator, the second add operation returns false (and the size of the sorted set does not increase) because a and b are equivalent from the sorted set's perspective.

Таким образом у нас просто не полностью заполненное дерево и некоторые мосты мы просто не используем.

Хм.. Как тогда решать задачу? Можно использовать `TreeMap<Long, LinkedSet<Integer>>`, где по ключу будут лежать все мосты имеющие такую длину. Сложность решения очевидно из-за этого не пострадает.

Резюмируя. В древоидных структурах данных следует делать так, что одинаковые объекты одинаковы и по компаратору.

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}
