---
title: Make C string literals const?
url: https://gustedt.wordpress.com/2025/04/06/make-c-string-literals-const/
published: "2025-04-06T19:22:03Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=4413
---

# Make C string literals const?

Martin Uecker has started a new initiative to ensure a better `const` contract for C2y: change the type of string literals to a `const`-qualified base type, much as it is already the case in C++. Compilers support this since a very long time; some of them have this as default, some provide command line switches for that model.

Nevertheless, this would be normative change and might be some burden for existing code. So, before doing this and writing papers, it would be good if we had an idea of the impact of such a change in existing code bases. I would be very grateful if we’d receive feedback from you along the lines of

- You have a project that already uses options such as gcc’s `-Wwrite-strings` (or even a compiler with such a default) to have all string literals `const`-qualified.
- You have a project and you tested it with such options and introducing this change would be easy. (If so does this change expose some qualification bugs in your code base?)
- You have a project and you tested it with such options, but introducing this permanently into your code base would be a real pain.
- Your project does not care about `const` and you wouldn’t even know where to begin. If it were introduced in a future version of C, you probably you would have to use the command line to switch such a feature off.

The goal here is **_not_** to have an opinion poll or similar; I am not convinced that such polls on some random blog like mine have any particular meaning. I really like to have facts first, not opinions:

- If it is open source, please give a pointer to your project. Otherwise, please describe your project e.g if it is a commercial product of your company or employer. But please, do not share confident information that could get you in trouble.
- Report on your experience with these kind of options for `const` qualification.
- Don’t speculate about what could happen, restrict yourself to facts.
