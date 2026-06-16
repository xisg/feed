---
title: webkittens! lexical scoping is in danger! — wingolog
url: https://wingolog.org/archives/2011/12/02/webkittens-lexical-scoping-is-in-danger
published: "2011-12-02T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2011/12/02/webkittens-lexical-scoping-is-in-danger
---

# webkittens! lexical scoping is in danger! — wingolog

## [webkittens! lexical scoping is in danger!](/archives/2011/12/02/webkittens-lexical-scoping-is-in-danger)

2 December 2011 5:36 PM

- [webkit](/tags/webkit)
- [webkitgtk](/tags/webkitgtk)
- [jsc](/tags/jsc)
- [javascript](/tags/javascript)
- [ecmascript](/tags/ecmascript)
- [igalia](/tags/igalia)

![](//wingolog.org/pub/webkittens.jpg)

The [GTK+ WebKittens](https://live.gnome.org/Hackfests/WebKitGTK2011) are on the loose here in Coruña. There's folks here from Red Hat, Motorola, Collabora, and of course Igalia. It's good times; beyond the obvious platitudes of "um, the web is important and stuff" it's good to be in a group that is creating the web experience of millions of users.

My part in that is very small, adding support for [block-scoped `let` and `const`](https://bugs.webkit.org/show_bug.cgi?id=31813) to [JavaScriptCore](//wingolog.org/archives/2011/10/28/javascriptcore-the-webkit-js-implementation).

I've made some progress, but it could be going more smoothly. I have made the parser do the right thing for `const`, correctly raising errors for duplicate bindings, including nested `var` declarations that get hoisted. The parser is fine: it maintains an environment like you would expect. But the AST assumes that all locals get hoisted to function scope, so there's no provision for e.g. two distinct local variables with the same name. So there is still some work to do on the AST, and it's a thicket of templates.

Hopefully I'll end up with a prototype by the end of the hackfest (Sunday). Sooner if I find that sword of omens, which I seem to have misplaced. Sight beyond sight!

## related articles

- [inside javascriptcore's low-level interpreter](/archives/2012/06/27/inside-javascriptcores-low-level-interpreter)
- [JavaScriptCore, the WebKit JS implementation](/archives/2011/10/28/javascriptcore-the-webkit-js-implementation)
- [javascript weakmaps should be iterable](/archives/2024/08/19/javascript-weakmaps-should-be-iterable)
- [heap object representation in spidermonkey](/archives/2018/10/11/heap-object-representation-in-spidermonkey)
- [state of js implementations, 2014 edition](/archives/2014/12/09/state-of-js-implementations-2014-edition)
- [generators in firefox now twenty-two times faster](/archives/2014/11/14/generators-in-firefox-now-twenty-two-times-faster)

Comments are closed.
