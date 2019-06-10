---
layout: post
title:  boost::bind源码剖析
date:   2014-01-19 20:57:00 +0800
categories: code
tags: cpp
---

* [原理](#a)
* [实现分析](#b)
	+ [1. 函数信息保存](#b1)
    + [2. 参数](#b2)
    	- [2.1. 参数占位符](#b21)
        - [2.2. 多参数支持](#b22)
        - [2.3. 参数的保存](#b23)
        - [2.4. 参数访问](#b24)
        - [2.5. 抽象类实例](#b25)
    + [3. 函数](#b3)
    	- [3.1. 函数调用](#b31)
        - [3.2. const成员函数支持](#b32)
* [结束](#c)

<h1 id="a">原理</h1>
要调用一个函数, 只需两块信息: 函数指针func, 参数列表arg_list. 有了这两块信息, 就可以调用此函数了: func(arg_list)  
boost::bind将这两块信息保存至类实例中, 然后重载()运算符来模拟函数调用.  
C++中的函数有两大类: 非类成员函数, 类成员函数. 而类成员函数的调用和非类成员函数的调用是有区别的, 因为类成员函数需要通过类实例来调用. 所以, bind类成员函数的时候, 还需要将类实例一并记录下来.

可以通过一段粗糙的代码来观察实现原理, 此代码实现绑定带一个参数的类成员函数.

{% highlight cpp %}
template <typename R, typename T, typename A1>
class bind_t {
        typedef R (T::*F)(A1);

    public:
        bind_t(T* c, F f, A1 a1) : _f(f), _a1(a1) {}

        R operator()() { return (_c->*_f)(_a1); }

    private:
        T* _c;      // class instance
        F _f;       // member function
        A1 _a1;     // argument
};

template <typename R, typename T, typename A1>
bind_t<R,T,A1> bind(R (T::*f)(A1), T* c, A1 a1) {
    return bind_t<R,T,A1>(c, f, a1);
}
{% endhighlight %}

此代码提供bind接口函数, 将类成员函数指针,类实例,参数封装到bind_t中, 然后通过bind_t的重载操作符()调用函数.  
此实现有如下问题:  

* bind_t只支持1个参数
* 必须在绑定的时候指定参数, 而不能在调用的时候指定
* 只能传递类实例的指针
* 没有考虑const成员函数的情况

这些问题boost::bind都做了处理.  

<h1 id="b">实现分析</h1>
<h2 id="b1">1. 函数信息保存</h2>
函数的信息保存在类bind_t中:

{% highlight cpp %}
template<class R, class F, class L>
class bind_t {
    // ...
    private:
        F f_;
        L l_;
};
{% endhighlight %}

模板化函数返回值类型(R), 函数指针类型(F), 参数列表(L).  
从定义中并未看出类成员函数和非类成员函数的区别, 的确, 它们都是用这个类来保存信息的.  
boost::bind绑定成员函数的时候, 同样需要将类实例传递给它, 但它有个规则: 必须把类实例放在函数指针的后面, 其他参数的前面. 这样, 它就可以将类实例和其他参数一起放到参数列表(L)中, 通过取参数列表的第一个来获得类实例.  
类函数和非类函数的bind接口声明对比如下:

{% highlight cpp %}
// 函数原型: R (*f)(A1)
// bind(f, a1);
template<class R, class F, class A1>
    _bi::bind_t<R, F, typename _bi::list_av_1<A1>::type>
    BOOST_BIND(F f, A1 a1)
{ ... }

// 函数原型: R (T::*f)(A2)
// bind(&T::f, a1, a2);
// 其中, a1为类实例
template<class R, class T,
    class B1,
    class A1, class A2>
    _bi::bind_t<R, _mfi::BOOST_BIND_MF_NAME(mf1)<R, T, B1>, typename _bi::list_av_2<A1, A2>::type>
    BOOST_BIND(R (BOOST_BIND_MF_CC T::*f) (B1), A1 a1, A2 a2)
{ ... }
{% endhighlight %}

<h2 id="b2">2. 参数</h2>
<h3 id="b21">2.1. 参数占位符</h3>
参数占位符的作用是: 绑定函数的时候, 不具体指定某参数的值, 仅用占位符来代替, 等到具体调用的时候再指定.  
比如:

{% highlight cpp %}
int f(int, char, bool);
bind_t bf = bind(f, 1, _1, false);
bf('z'); // f(1, 'z', false);
{% endhighlight %}

占位符的定义如下:

{% highlight cpp %}
boost::arg<1> _1;
boost::arg<2> _2;
boost::arg<3> _3;
boost::arg<4> _4;
boost::arg<5> _5;
boost::arg<6> _6;
boost::arg<7> _7;
boost::arg<8> _8;
boost::arg<9> _9;
{% endhighlight %}

<h3 id="b22">2.2. 多参数支持</h3>
bind_t模板化参数列表, 将多参数的支持推给模板类型(L). L是list_n([0-9]), 参数就是保存在这里的. n代表它保存的参数的个数, 比如list_3保存了3个参数.  
list_n还负责调用被绑定的函数, 见bind_t重载操作符():  

{% highlight cpp %}
template<class A1> result_type operator()(A1 & a1)
{
    list1<A1 &> a(a1);
    BOOST_BIND_RETURN l_(type<result_type>(), f_, a, 0);
}
{% endhighlight %}

为什么要让list\_n调用f\_呢? 为什么不在bind\_t::operator()(...)中直接调用f\_呢?  
因为: 在bind\_t::operator()()中, 没法知道f\_所需的参数的个数, 而l\_知道, 因为f\_所需的参数在构造bind\_t的时候都保存在l\_中了, 当然, 可能包含占位符.  
所以, 多参数的支持是通过list_n来实现的, boost::bind实现了list_0~list_9, 所以, 对于非成员函数, boost::bind最多支持9个参数; 而对于成员函数, 最多支持8个(类实例要占一个)

<h3 id="b23">2.3. 参数的保存</h3>
因为存在占位符, 所以不能简单的将参数直接保存到list_n中.  
boost::bind对真正参数做了一层封装:

{% highlight cpp %}
template<class T> class value
{
public:
    value(T const & t): t_(t) {}

    T & get() { return t_; }
    T const & get() const { return t_; }

    bool operator==(value const & rhs) const
    {
        return t_ == rhs.t_;
    }

private:
    T t_;
};
{% endhighlight %}

专门定义value类来保存真正的参数的目的是: 为了实现list_n中的operator[], 来区分占位符(arg<n>).  
在构造bind_t的时候, 根据参数的类型(是否为占位符)来决定list_n的参数的类型: 若不是占位符, 则参数类型为value<T>:

{% highlight cpp %}
// bind(F, A1)
template<class R, class F, class A1>
    _bi::bind_t<R, F, typename _bi::list_av_1<A1>::type>
    BOOST_BIND(F f, A1 a1)
{
    typedef typename _bi::list_av_1<A1>::type list_type;
    return _bi::bind_t<R, F, list_type> (f, list_type(a1));
}
{% endhighlight %}

list_av_n通过add_value来决定list_n的参数类型:

{% highlight cpp %}
template<class A1> struct list_av_1
{
    typedef typename add_value<A1>::type B1;
    typedef list1<B1> type;
};
{% endhighlight %}

add_value通过模板类add_value_2来判断参数类型:

{% highlight cpp %}
template<class T> struct add_value
{
    typedef typename add_value_2< T, boost::is_placeholder< T >::value >::type type;
};
{% endhighlight %}

is_placeholder判断类型T是否是placeholder, 若不是, 则is_placeholder::value=0.  
add_value_2特化非占位符类, 实现了对占位符与非占位符的区分:

{% highlight cpp %}
template< class T, int I > struct add_value_2
{
    typedef boost::arg<I> type;
};

template< class T > struct add_value_2< T, 0 >
{
    typedef _bi::value< T > type;
};
{% endhighlight %}

<h3 id="b24">2.4. 参数访问</h3>
list_n通过重载operator[]来实现对参数的访问:

{% highlight cpp %}
template<class T> T & operator[] (_bi::value<T> & v) const { return v.get(); }
A1 operator[] (boost::arg<1>) const { return base_type::a1_; }
{% endhighlight %}

因为要区分占位符, 所以分别为占位符和真正的参数重载了operator[].  
list_n可以看作是继承实现的(其实它另外定义了storage[n]类专门用来存放, 而storage[n]是继承实现的), 所以list_4的前3个参数都保存在list_3中.  
调用函数的时候, 把占位符替换成真正的参数的秘密就在这里了.  
以2.1中的例子分析:  
bind(f, 1, _1, false)会将参数保存至list_3中, 然后存放至bind_t;  
bf('z')首先将参数'z'存放至list_1, 然后调用list_3的operator()  
bind_t::operator()如下:

{% highlight cpp %}
// l_: list_3, { 1, _1, false }
// a : list_1, { 'z' }
template<class A1> result_type operator()(A1 & a1)
{
    list1<A1 &> a(a1);
    BOOST_BIND_RETURN l_(type<result_type>(), f_, a, 0);
}
{% endhighlight %}

list_3::operator()如下:

{% highlight cpp %}
// a : list_1, { 'z' }
// base_type : storage3, { 1, _1, false }
template<class R, class F, class A> R operator()(type<R>, F & f, A & a, long)
{
    return unwrapper<F>::unwrap(f, 0)(a[base_type::a1_], a[base_type::a2_], a[base_type::a3_]);
}
{% endhighlight %}

至此, list_1::operator[]会根据list_3中的值的类型, 返回对应的值: 若为占位符则返回list_1中的对应值, 否则直接返回参数中的值.

<h3 id="b25">2.5. 抽象类实例</h3>
抽象类的实例可能是指针, 也可能不是. 而指针的是与否影响了调用的语法:  
pointer:    (o->\*f)(...)  
non-pointer:(o.\*f)(...)  
为了支持多种情况, boost::bind模板化类名和实例类型:

{% highlight cpp %}
// T : 类名
// A1: 实例类型(可能是T*, T 等)
template<class R, class T,
    class B1,
    class A1, class A2>
    _bi::bind_t<R, _mfi::BOOST_BIND_MF_NAME(mf1)<R, T, B1>, typename _bi::list_av_2<A1, A2>::type>
    BOOST_BIND(R (BOOST_BIND_MF_CC T::*f) (B1), A1 a1, A2 a2)
{
    typedef _mfi::BOOST_BIND_MF_NAME(mf1)<R, T, B1> F;
    typedef typename _bi::list_av_2<A1, A2>::type list_type;
    return _bi::bind_t<R, F, list_type>(F(f), list_type(a1, a2));
}
{% endhighlight %}

上述代码所绑定的函数原型为: R (T::*)(B1), bind的原型为bind_t<...> bind(F, A1, A2), 既然A2是被绑定函数的参数, 为什么要用两个模板变量B1,A2呢?  
因为A2有可能是占位符.  

<h2 id="b3">3. 函数</h2>
<h3 id="b31">3.1. 函数调用</h3>
成员函数与非成员函数的调用是有区别的, 因为成员函数的调用需要类实例.  
上面分析过, boost::bind并不区分参数和类实例, 统一将它们放入list_n, 唯一的规则是: 类实例要放在list_n的首位.  
举例分析:  
non_member_func(123, false);  
o->member_func(123, false);  
non_member_func: list_2 = { 123, false }  
member_func: list_3 = { o, 123, false }  
函数的调用是通过list_n来触发的, 为了保持格式统一, boost::bind对成员函数进行了封装, 并重载了operator()来模拟函数调用:

{% highlight cpp %}
template<class R, class T, class A1, class A2 BOOST_MEM_FN_CLASS_F> class BOOST_MEM_FN_NAME(mf2)
{
public:
    explicit BOOST_MEM_FN_NAME(mf2)(F f): f_(f) {}
    R operator()(T * p, A1 a1, A2 a2) const
    {
        BOOST_MEM_FN_RETURN (p->*f_)(a1, a2);
    }
    // ...
    
private:
    F f_;
};
{% endhighlight %}

这样, 就可以在list_n对函数调用统一处理了:

{% highlight cpp %}
template<class R, class F, class A> R operator()(type<R>, F const & f, A & a, long) const
{
    return unwrapper<F const>::unwrap(f, 0)(a[base_type::a1_], a[base_type::a2_]);
}
{% endhighlight %}

unwrapper根据F的类型, 返回对应的函数:

{% highlight cpp %}
template<class F> struct unwrapper
{
    // 非成员函数
    static inline F & unwrap( F & f, long )
    {
        return f;
    }

    // 成员函数
    template<class R, class T> static inline _mfi::dm<R, T> unwrap( R T::* pm, int )
    {
        return _mfi::dm<R, T>( pm );
    }
};
{% endhighlight %}

_mfi::dm根据pm的类型, 选择正确的调用方法:

{% highlight cpp %}
namespace _mfi
{
template<class R, class T> class dm
{
    public:
        explicit dm(F f): f_(f) {}

        R & operator()(T * p) const
        {
            return (p->*f_);
        }

        R & operator()(T & t) const
        {
            return (t.*f_);
        }
    
    private:
        typedef R (T::*F);
        F f_;
};
}
{% endhighlight %}

<h3 id="b32">3.2. const成员函数支持</h3>
对于const成员函数的支持, boost::bind的解决办法是重载:

{% highlight cpp %}
// 非const
template<class R, class T,
    class B1,
    class A1, class A2>
    _bi::bind_t<R, _mfi::BOOST_BIND_MF_NAME(mf1)<R, T, B1>, typename _bi::list_av_2<A1, A2>::type>
    BOOST_BIND(R (BOOST_BIND_MF_CC T::*f) (B1), A1 a1, A2 a2)
{
    typedef _mfi::BOOST_BIND_MF_NAME(mf1)<R, T, B1> F;
    typedef typename _bi::list_av_2<A1, A2>::type list_type;
    return _bi::bind_t<R, F, list_type>(F(f), list_type(a1, a2));
}

// const
template<class R, class T,
    class B1,
    class A1, class A2>
    _bi::bind_t<R, _mfi::BOOST_BIND_MF_NAME(cmf1)<R, T, B1>, typename _bi::list_av_2<A1, A2>::type>
    BOOST_BIND(R (BOOST_BIND_MF_CC T::*f) (B1) const, A1 a1, A2 a2)
{
    typedef _mfi::BOOST_BIND_MF_NAME(cmf1)<R, T, B1> F;
    typedef typename _bi::list_av_2<A1, A2>::type list_type;
    return _bi::bind_t<R, F, list_type>(F(f), list_type(a1, a2));
}
{% endhighlight %}

<h1 id="c">结束</h1>
根据boost::bind的原理, 我模仿着实现了一个简化版: [outsky/bind](https://github.com/outsky/bind)
