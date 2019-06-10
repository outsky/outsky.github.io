---
layout: post
title:  invalid order function for sorting
date:   2014-03-26 00:22:00 +0800
categories: code
tags: lua
---
invalid order function for sorting，这是我在用lua的[table.sort](http://www.lua.org/manual/5.2/manual.html#pdf-table.sort)的时候遇到的错误。  
举例，若我有一个学生的成绩table，我想对它进行排序，规则是：__按成绩由大到小排序，若成绩相等，则年龄小的排在前面__。  
我实现的代码如下：  

{% highlight lua %}
local students = {
    {name="xiaomi", age=13, gender="boy", score=10},
    {name="lilei", age=33, gender="boy", score=10},
    {name="jack", age=23, gender="boy", score=20},
    {name="rose", age=18, gender="girl", score=20},
    {name="lamn", age=9, gender="girl", score=20},
    {name="cpp", age=21, gender="boy", score=38},
    {name="lua", age=12, gender="boy", score=38},
    {name="java", age=13, gender="girl", score=38},
    {name="shell", age=23, gender="girl", score=38},
    {name="linus", age=33, gender="boy", score=10},
}

local function cmp(s1, s2)
    return s1.score>s2.score or s1.age<s2.age
end

table.sort(students, cmp)

for _,v in ipairs(students) do
    print(v.name, v.age, v.gender, v.score)
end
{% endhighlight %}

运行此代码，就会报错：__invalid order function for sorting__  
原因是：cmp函数的逻辑不够严谨，导致cmp(xiaomi, jack)和cmp(jack, xiaomi)的返回值一样都是true，代码从而无法判断哪个更大，所以不是合法的比较函数。

Lua的手册上也提到了：[it must be a function that receives two list elements and returns true when the first element must come before the second in the final order (so that not comp(list[i+1],list[i]) will be true after the sort)](http://www.lua.org/manual/5.2/manual.html#pdf-table.sort)
  
修改如下：

{% highlight lua %}
local function cmp(s1, s2)
    return s1.score>s2.score or (s1.score==s2.score and s1.age<s2.age)
end
{% endhighlight %}

因为，我们定的规则是：首先按成绩排序，只有在成绩相等的时候，才考虑年龄的因素。