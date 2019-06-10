---
layout: post
title:  GCC内置原子内存存取函数
date:   2014-09-06 22:03:00 +0800
categories: code
tags: gcc
---
#__sync_fetch_and_op系列

```c
type __sync_fetch_and_add (type *ptr, type value, ...)
type __sync_fetch_and_sub (type *ptr, type value, ...)
type __sync_fetch_and_or (type *ptr, type value, ...)
type __sync_fetch_and_and (type *ptr, type value, ...)
type __sync_fetch_and_xor (type *ptr, type value, ...)
type __sync_fetch_and_nand (type *ptr, type value, ...)
```

这些函数进行的运算（op）可以从函数名中看出：

- 将`*ptr`与`value`做运算
- 返回运算**前**的`*ptr`

代码解释如下：

```c
{ tmp = *ptr; *ptr op= value; return tmp; }
{ tmp = *ptr; *ptr = ~tmp & value; return tmp; }   // nand
```

----------

#__sync_op_and_fetch系列

```c
type __sync_add_and_fetch (type *ptr, type value, ...)
type __sync_sub_and_fetch (type *ptr, type value, ...)
type __sync_or_and_fetch (type *ptr, type value, ...)
type __sync_and_and_fetch (type *ptr, type value, ...)
type __sync_xor_and_fetch (type *ptr, type value, ...)
type __sync_nand_and_fetch (type *ptr, type value, ...)
```

这些函数进行的运算（op）可以从函数名中看出：

- 将`*ptr`与`value`做运算
- 返回运算**后**的`*ptr`

代码解释如下：

```c
{ *ptr op= value; return *ptr; }
{ *ptr = ~*ptr & value; return *ptr; }   // nand
```

----------

#__sync_x_compare_and_swap系列

```c
bool __sync_bool_compare_and_swap (type *ptr, type oldval, type newval, ...)
type __sync_val_compare_and_swap (type *ptr, type oldval, type newval, ...)
```

这些函数对比`*ptr`和`oldval`，如果它们相等，就将`newval`赋值给`*ptr`。  
返回值：

- bool版本：true(`*ptr`被赋新值)
- type版本：返回操作前的`*ptr`

代码解释如下：

```c
{
	if(*ptr == oldval) {
		*ptr = newval;
		return true;
	}
	return false;
}

{
	type tmp = *ptr;
	if(*ptr == oldval)
		*ptr = newval;
	return tmp;
}
```

----------

#type __sync_lock_test_and_set (type *ptr, type value, ...)

将`value`赋值给`*ptr`，返回赋值前的`*ptr`

代码解释如下：

```c
{ type tmp=*ptr; *ptr=value; return tmp; }
```

----------

#void __sync_lock_release (type *ptr, ...)

释放`__sync_lock_test_and_set`申请的锁，通常意味着将`*ptr`赋值为0。

参考资料：[http://gcc.gnu.org/onlinedocs/gcc-4.1.2/gcc/Atomic-Builtins.html](http://gcc.gnu.org/onlinedocs/gcc-4.1.2/gcc/Atomic-Builtins.html)