---
title: A diagram of C23 basic types
url: https://gustedt.wordpress.com/2025/03/29/a-diagram-of-c23-basic-types/
published: "2025-03-29T14:43:41Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=4399
---

# A diagram of C23 basic types

This week on the C committee mailing list we had a discussion about how C’s types are organized into different categories. At the end I came up with a diagram with that organization. It basically translates the section “ **6.2.5 Types**” of the C23 standard into a graph of inclusions.

[![](https://gustedt.wordpress.com/wp-content/uploads/2025/03/scalars.png?w=840)](https://gustedt.wordpress.com/wp-content/uploads/2025/03/scalars.png)

Sorry, it is probably a little bit too wide for your phone, but by opening it on a computer screen or a laptop in a separate window and using the zooming feature of your browser you should be able to discover its parts. Annotations give the paragraph number in that clause of the C23 standard.

There is some minor color coding:

- Special cases of additions to certain categories are noted with a red arrow
- Inclusions that are not mentioned explicitly in the standard but that follow from the extent of the underlying set of types are dashed blue.
- Type correspondence where a type or type category is used to define another are dotted lines.
- Narrow integer types that are promoted (usually to `int`) are in dark green.
