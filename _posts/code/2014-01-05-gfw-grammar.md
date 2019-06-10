---
layout: post
title:  GFM语法格式收集整理
date:   2014-01-05 17:01:00 +0800
categories: code
tags: web
---
[Github Flavored Markdown](https://help.github.com/articles/github-flavored-markdown)（GFM），是github改进过的[Markdown](http://daringfireball.net/projects/markdown/)，使得我们可以用易读简洁的纯文本方式编写html，非常方便使用。  
本文对GFM的各种语法格式进行收集整理，以便日后查阅。  


*   [Line Breaks](#p)
*   [Headers](#header)
*   [Blockquotes](#blockquote)
*   [Lists](#list)
*   [Code Blocks](#precode)
*   [Horizontal Rules](#hr)
*   [Links](#link)
*   [Emphasis](#em)
*   [Images](#img)
*   [Automatic Links](#autolink)
*   [Backslash Escapes](#backslash)
* * *

<h3 id="p">Line Breaks</h3>

End a line with two or more spaces, then type return.

<h3 id="header">Headers</h3>

```markdown
This is an H1
=============
This is an H2
-------------
```

This is an H1
=============
This is an H2
-------------
- - -

```markdown
# This is an H1
## This is an H2
###### This is an H6
```
# This is an H1
## This is an H2
###### This is an H6

- - -

<h3 id="blockquote">Blockquotes</h3>

```markdown
> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
> consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
> Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
> 
> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
> id sem consectetuer libero luctus adipiscing.
```

> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
> consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
> Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
> 
> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
> id sem consectetuer libero luctus adipiscing.
- - -

```markdown
> This is the first level of quoting.
>
> > This is nested blockquote.
>
> Back to the first level.
```

> This is the first level of quoting.
>
> > This is nested blockquote.
>
> Back to the first level.
- - -

contain other Markdown elements:

```markdown
> ## This is a header.
> 
> 1.   This is the first list item.
> 2.   This is the second list item.
> 
> Here's some example code:
> 
>     return shell_exec("echo $input | $markdown_script");
```

> ## This is a header.
> 
> 1.   This is the first list item.
> 2.   This is the second list item.
> 
> Here's some example code:
> 
>     return shell_exec("echo $input | $markdown_script");

- - -

<h3 id="list">Lists</h3>

```markdown
*   Red
*   Green
*   Blue

+   Red
+   Green
+   Blue

-   Red
-   Green
-   Blue
```

-   Red
-   Green
-   Blue
- - -

```markdown
1.  Bird
2.  McHale
3.  Parish
```

1.  Bird
2.  McHale
3.  Parish
- - -

```markdown
1.  This is a list item with two paragraphs. Lorem ipsum dolor
	sit amet, consectetuer adipiscing elit. Aliquam hendrerit
	mi posuere lectus.

	Vestibulum enim wisi, viverra nec, fringilla in, laoreet
	vitae, risus. Donec sit amet nisl. Aliquam semper ipsum
	sit amet velit.

2.  Suspendisse id sem consectetuer libero luctus adipiscing.
```

1.  This is a list item with two paragraphs. Lorem ipsum dolor
	sit amet, consectetuer adipiscing elit. Aliquam hendrerit
	mi posuere lectus.

	Vestibulum enim wisi, viverra nec, fringilla in, laoreet
	vitae, risus. Donec sit amet nisl. Aliquam semper ipsum
	sit amet velit.

2.  Suspendisse id sem consectetuer libero luctus adipiscing.
- - -

```markdown
*   A list item with a blockquote:

	> This is a blockquote
	> inside a list item.
```

*   A list item with a blockquote:

	> This is a blockquote
	> inside a list item.
- - -

To put a code block within a list item, the code block needs
to be indented *twice* -- 8 spaces or two tabs:

```markdown
*   A list item with a code block:

		std::cout << "Hello world!" << std::endl;
```

*   A list item with a code block:

		std::cout << "Hello world!" << std::endl;
- - -

```markdown
1986. What a great season.
```

1986. What a great season.
- - -

```markdown
1986\. What a great season.
```

1986\. What a great season.

- - -

<h3 id="precode">Code Blocks</h3>

````markdown
```javascript
function test() {
  console.log("notice the blank line before this function?");
}
```
````

```javascript
function test() {
  console.log("notice the blank line before this function?");
}
```

- - -

<h3 id="hr">Horizontal Rules</h3>

```markdown

* * *

***

*****

- - -

---------------------------------------
```

- - -

<h3 id="link">Links</h3>

```markdown
This is [an example](http://example.com/ "Title") inline link.
```

This is [an example](http://example.com/ "Title") inline link.
- - -

```markdown
[This link](http://example.net/) has no title attribute.
```

[This link](http://example.net/) has no title attribute.
- - -

```markdown
I get 10 times more traffic from [Google] [1] than from
[Yahoo] [2] or [MSN] [3].

[1]: http://google.com/        "Google"
[2]: http://search.yahoo.com/  "Yahoo Search"
[3]: http://search.msn.com/    "MSN Search"
```

I get 10 times more traffic from [Google] [1] than from
[Yahoo] [2] or [MSN] [3].

[1]: http://google.com/        "Google"
[2]: http://search.yahoo.com/  "Yahoo Search"
[3]: http://search.msn.com/    "MSN Search"
- - -
      
```markdown
I get 10 times more traffic from [Google](http://google.com/ "Google")
than from [Yahoo](http://search.yahoo.com/ "Yahoo Search") or
[MSN](http://search.msn.com/ "MSN Search").
```

I get 10 times more traffic from [Google](http://google.com/ "Google")
than from [Yahoo](http://search.yahoo.com/ "Yahoo Search") or
[MSN](http://search.msn.com/ "MSN Search").

- - -

<h3 id="em">Emphasis</h3>

```markdown
~~Mistaken text.~~

*single asterisks*

_single underscores_

**double asterisks**

__double underscores__

un*frigging*believable

\*this text is surrounded by literal asterisks\*
```

~~Mistaken text.~~

*single asterisks*

_single underscores_

**double asterisks**

__double underscores__

un*frigging*believable

\*this text is surrounded by literal asterisks\*

- - -

<h3 id="img">Images</h3>

```markdown
![Alt text](https://help.github.com/assets/help/invertocat-1264358d01ae6ae7a728f3c909f51c83.png)

![Alt text](https://help.github.com/assets/help/invertocat-1264358d01ae6ae7a728f3c909f51c83.png "Optional title")

![Alt text][github]
```
![Alt text](https://help.github.com/assets/help/invertocat-1264358d01ae6ae7a728f3c909f51c83.png)

![Alt text](https://help.github.com/assets/help/invertocat-1264358d01ae6ae7a728f3c909f51c83.png "Optional title")

![Alt text][github]

Where "github" is the name of a defined image reference. Image references
are defined using syntax identical to link references:

```markdown
[github]: https://help.github.com/assets/help/invertocat-1264358d01ae6ae7a728f3c909f51c83.png  "Optional title attribute"
```

[github]: https://help.github.com/assets/help/invertocat-1264358d01ae6ae7a728f3c909f51c83.png  "Optional title attribute"

As of this writing, Markdown has no syntax for specifying the
dimensions of an image; if this is important to you, you can simply
use regular HTML `<img>` tags.

- - -

<h3 id="autolink">Automatic Links</h3>

```markdown
<http://example.com/>  
<address@example.com>
```

<http://example.com/>  
<address@example.com>

- - -

<h3 id="backslash">Backslash Escapes</h3>

```markdown
*literal asterisks*  
\*literal asterisks\*
```

*literal asterisks*  
\*literal asterisks\*

- - -

Markdown provides backslash escapes for the following characters:

    \   backslash
    `   backtick
    *   asterisk
    _   underscore
    {}  curly braces
    []  square brackets
    ()  parentheses
    #   hash mark
	+	plus sign
	-	minus sign (hyphen)
    .   dot
    !   exclamation mark

