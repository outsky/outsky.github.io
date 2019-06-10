---
layout: post
title: 信用卡分期的真实年利率
date: 2016-11-30 16:44:00 +0800
categories: life
tags: credit
---

今天，收到招行信用卡的短信，推荐我现金分期，月利息0.38%

![](/res/img/20161130/1.png)

算了一下，年利率只有4.56%，想想还真有点小激动

点进去看了一下，具体如图：

![](/res/img/20161130/2.png)

但仔细一想，不对：银行的利息是按本金收的，但我每个月都要还一部分本金，它还是按最初的本金收取利息。利息肯定比他宣传的要高，具体高多少呢？我决定写个程序来算一下：


{% highlight lua %}
function compute(borrow, monthes, paymonthly)
    print(string.format("# %d (%.2f x%d)", borrow, paymonthly, monthes))
    delta = borrow/monthes
    borrowavg = (delta+borrow)/2
    interest = (paymonthly-delta)*monthes
    annualrate = interest/(borrowavg/12*monthes)
    print(string.format("annual rate: %.2f%%, total interest: %.2f\n", annualrate*100, interest))
end

compute(50000, 3, 16904.17)
compute(50000, 6, 8533.33)
compute(50000, 10, 5187.50)
compute(50000, 12, 4354.17)
compute(50000, 18, 2965.28)
compute(50000, 24, 2270.83)
{% endhighlight %}

Output:

<pre>
# 50000 (16904.17 x3)
annual rate: 8.55%, total interest: 712.51

# 50000 (8533.33 x6)
annual rate: 8.23%, total interest: 1199.98

# 50000 (5187.50 x10)
annual rate: 8.18%, total interest: 1875.00

# 50000 (4354.17 x12)
annual rate: 8.31%, total interest: 2250.04

# 50000 (2965.28 x18)
annual rate: 8.53%, total interest: 3375.04

# 50000 (2270.83 x24)
annual rate: 8.64%, total interest: 4499.92
</pre>


由此可见，实际年利率在8.5%左右，比宣传的利率要高，但也不算很黑了。

-------

假设：

B为借款总额（元）, M为借贷月数（月）, P为每期还款额（元）, 则，计算平均年化利率的公式可简化为：

![](/res/img/20161130/3.gif)

--------

我写了一个方便计算贷款真实利率的微信小程序：

![](/res/img/20161130/LoanRateCalc.jpg)
