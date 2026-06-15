---
title: 'Google Summer of Code: C++ Modernizer Improvements'
url: https://blog.llvm.org/2013/11/google-summer-of-code-c-modernizer.html
published: "2013-11-18T04:12:00Z"
feed: llvm
guid: https://blog.llvm.org/2013/11/google-summer-of-code-c-modernizer.html
---

# Google Summer of Code: C++ Modernizer Improvements

[Google Summer of Code](http://www.google-melange.com/gsoc/homepage/google/gsoc2013) (GSoC) offers students stipends to participate in open source projects during the summer. This year, I was accepted to work on the [Clang C++ Modernizer](http://clang.llvm.org/extra/clang-modernize.html), a project formerly known as the _C++11 Migrator_, driven by a team at Intel. The goals of the tool are to modernize C++ code by using the new features of new C++ standards in order to improve maintainability, readability and compile time and runtime performance. The project was featured in the April blog post “ [Status of the C++11 Migrator](http://blog.llvm.org/2013/04/status-of-c11-migrator.html)” and has been evolving since, both in terms of architecture and features.

This article presents the improvements made to the tool in the last few months, which include my work from this summer for GSoC. For a complete overview of the tool and how to install it, please visit the documentation: [http://clang.llvm.org/extra/clang-modernize.html#getting-started](http://clang.llvm.org/extra/clang-modernize.html#getting-started). For a demonstration of the tool you can take a look at the Going Native 2013 talk given by Chandler Carruth: [The Care and Feeding of C++'s Dragons](http://channel9.msdn.com/Events/GoingNative/2013/The-Care-and-Feeding-of-C-s-Dragons). clang-modernize is featured starting at ~33min.

## Transform all Files That Make up a Translation Unit

A major improvement since the last version is the ability to transform every file that composes a translation unit not only the main source file. This means headers also get transformed if they need to be which makes the modernizer more useful.

To avoid changing files that shouldn’t be changed, e.g. system headers or headers for third-party libraries, there are a few options to control which files should be transformed:

- [-include](http://clang.llvm.org/extra/ModernizerUsage.html#cmdoption-include) Takes a comma-separated list of paths allowed to be transformed. All files within the entire directory tree rooted at each given path are marked as modifiable. For safety, the default behaviour is that no extra files will be transformed.
- [-exclude](http://clang.llvm.org/extra/ModernizerUsage.html#cmdoption-exclude) Takes a comma-separated list of paths forbidden to be transformed. Can be used to prune out subtrees from included directory trees.
- [-include-from](http://clang.llvm.org/extra/ModernizerUsage.html#cmdoption-include-from) and [-exclude-from](http://clang.llvm.org/extra/ModernizerUsage.html#cmdoption-exclude-from) Respectively equivalent to -include and -exclude but takes a filename as argument instead of a comma-separated list of paths. The file should contain one path per line.

Example, assuming a directory hierarchy of:

- src/foo.cpp
- include/foo.h
- lib/third-party.h

to transform both _foo.cpp_ and _foo.h_ but leave _third-party.h_ as is, you can use one of the following commands:

```
clang-modernize -include=include/ src/foo.cpp -- -std=c++11 -I include/ -I lib/
```

```
clang-modernize -include=. -exclude=lib/ src/foo.cpp -- -std=c++11 -I include/ -I lib/
```

## The Transforms

Right now there is a total of 6 transforms, two of which are new:

1. [Add-Override Transform](http://clang.llvm.org/extra/AddOverrideTransform.html)

Adds the ‘override’ specifier to overriden member functions.
2. [Loop Convert Transform](http://clang.llvm.org/extra/LoopConvertTransform.html)

Makes use of for-ranged based loop.
3. [Pass-By-Value Transform](http://clang.llvm.org/extra/PassByValueTransform.html) \[new\]

Replaces const-ref parameters that would benefit from using the pass-by-value idiom.
4. [Replace Auto-Ptr Transform](http://clang.llvm.org/extra/ReplaceAutoPtrTransform.html) \[new\]

Replaces uses of the deprecated `std::auto_ptr` by `std::unique_ptr`.
5. [Use-Auto Transform](http://clang.llvm.org/extra/UseAutoTransform.html)

Makes use of the auto type specifier in variable declarations.
6. [Use-Nullptr Transform](http://clang.llvm.org/extra/UseNullptrTransform.html)

Replaces null literals and macros by nullptr where applicable.

### Improvement to Add-Override

Since the last article in April, the Add-Override Transform has been improved to handle user-defined macros. Some projects, like LLVM, use a macro that expands to the ‘override’ specifier for backward compatibility with non-C++11-compliant compilers. clang-modernize can detect those macros and use them instead of the ‘override’ identifier.

The command line switch to enable this functionality is **-override-macros**.

Example:

`clang-modernize -override-macros foo.cpp`

Before

After

#define LLVM\_OVERRIDE override

struct A {

virtualvoid foo();

};

struct B : A {

virtualvoid foo();

};

#define LLVM\_OVERRIDE override

struct A {

virtualvoid foo();

};

struct B : A {

virtualvoid foo() LLVM\_OVERRIDE;

};

### Improvement to Use-Nullptr

This transform has also been improved to handle user-defined macros that behave like NULL. The user specifies which macros can be replaced by nullptr by using the command line switch [-user-null-macros=<string>](http://clang.llvm.org/extra/ModernizerUsage.html#cmdoption-user-null-macros).

Example:

`clang-modernize -user-null-macros=MY_NULL bar.cpp`

Before

After

#define MY\_NULL 0

voidbar() {

int\*p = MY\_NULL;

}

#define MY\_NULL 0

voidbar() {

int\*p =nullptr;

}

### New Transform: Replace Auto-Ptr

This transform was a result of GSoC work. The transform replaces uses of `std::auto_ptr` by `std::unique_ptr`. It also inserts calls to `std::move()` when needed.

Before

After

#include <memory>

voidsteal(std::auto\_ptr<int> x);

voidfoo(int i) {

std::auto\_ptr<int> p(newint(i));

steal(p);

}

#include <memory>

voidsteal(std::unique\_ptr<int> x);

voidfoo(int i) {

std::unique\_ptr<int> p(newint(i));

steal(std::move(p));

}

### New Transform: Pass-By-Value

Also a product of GSoC this transform makes use of move semantics added in C++11 to avoid a copy for functions that accept types that have move constructors by const reference. By changing to pass-by-value semantics, a copy can be avoided if an rvalue argument is provided. For lvalue arguments, the number of copies remains unchanged.

The transform is currently limited to constructor parameters that are copied into class fields.

Example:

`clang-modernize pass-by-value.cpp`

**Before****After**

#include <string>

classA {

public:

A(const std::string &Copied,

const std::string &ReadOnly)

: Copied(Copied),

ReadOnly(ReadOnly) {}

private:

std::string Copied;

const std::string &ReadOnly;

};

#include <string>

#include <utility>

classA {

public:

A(std::string Copied,

const std::string &ReadOnly)

: Copied(std::move(Copied)),

ReadOnly(ReadOnly) {}

private:

std::string Copied;

const std::string &ReadOnly;

};

`std::move()` is a library function declared in `<utility>`. If need be, this header is added by clang-modernize.

There is a lot of room for improvement in this transform. Other situations that are safe to transform likely exist. Contributions are most welcomed in this area!

## Usability Improvements

We also worked hard on improving the overall usability of the modernizer. Invoking the modernizer now requires fewer arguments since most of the time the arguments can be inferred.

- If no compilation database or flags are provided, -std=c++11 is assumed.
- All transforms are enabled by default.
- Files don’t need to be explicitly listed if a compilation database is provided. The modernizer will get files from the compilation database. Use -include to choose which ones.

Two new features were also added.

1. Automatically reformat code affected by transforms using [LibFormat](http://clang.llvm.org/docs/LibFormat.html).
2. A new command line switch to choose transforms to apply based on compiler support.

### Reformatting Transformed Code

[LibFormat](http://clang.llvm.org/docs/LibFormat.html) is the library used behind the scenes by [clang-format](http://clang.llvm.org/docs/ClangFormat.html), a tool to format C, C++ and Obj-C code. _clang-modernize_ uses this library as well to reformat transformed code. When enabled with -format, the default style is LLVM. The -style option can control the style in a way identical to clang-format.

Example:

**format.cpp**

#include <iostream>

#include <vector>

voidf(const std::vector<int>&my\_container) {

for (std::vector<int>::const\_iterator I = my\_container.begin(),

                                        E = my\_container.end();

       I != E; ++I) {

    std::cout <<\*I << std::endl;

}

}

**Without reformatting**

`$ clang-modernize -use-auto format.cpp`

#include <iostream>

#include <vector>

voidf(const std::vector<int>&my\_container) {

for (auto I = my\_container.begin(),

                                        E = my\_container.end();

       I != E; ++I) {

    std::cout <<\*I << std::endl;

}

}

**With reformatting**

`$ clang-modernize -format -style=LLVM -use-auto format.cpp`

#include <iostream>

#include <vector>

voidf(const std::vector<int>&my\_container) {

for (auto I = my\_container.begin(), E = my\_container.end(); I != E; ++I) {

    std::cout <<\*I << std::endl;

}

}

For more information about this option, take a look at the documentation: [Formatting Command Line Options](http://clang.llvm.org/extra/ModernizerUsage.html#formatting-command-line-options).

### Choosing Transforms based on Compiler Support

Another useful command-line switch is: [-for-compilers](http://clang.llvm.org/extra/ModernizerUsage.html#for-compilers-option). This option enables all transforms the given compilers support.

As an example, imagine that your project dropped a dependency to a “legacy” version of a compiler. You can automagically modernize your code to the new minimum versions of the compilers you want to support:

To support Clang >= 3.1, GCC >= 4.6 and MSVC 11:

```
clang-modernize -format -for-compilers=clang-3.1,gcc-4.6,msvc-11 foo.cpp
```

For more information about this option and to see which transforms are available for each compilers, please read [the documentation](http://clang.llvm.org/extra/ModernizerUsage.html#cmdoption-for-compilers).

## What’s next?

The ability to transform many translation units in parallel will arrive very soon. Think of _clang-modernize -j_ as in make and ninja. Modernization of large code bases will become much faster as a result.

More transforms are coming down the pipe as well as improvements to existing transforms such as the pass-by-value transform.

We will continue fixing bugs and adding new features. Our backlog is publically available: [https://cpp11-migrate.atlassian.net/secure/RapidBoard.jspa?rapidView=1&view=planning](https://cpp11-migrate.atlassian.net/secure/RapidBoard.jspa?rapidView=1&view=planning)

## Get involved!

Interested by the tool? Found a bug? Have an idea of a transform that can be useful to others? The project is Open Source and contributions are most welcomed!

The modernizer has its own bug and project tracker. If you want to file or fix a bug just go to: [https://cpp11-migrate.atlassian.net](https://cpp11-migrate.atlassian.net/)

A few other addresses to keep in mind:

- [Clang C++ Modernizer User’s Manual](http://clang.llvm.org/extra/clang-modernize.html)
- IRC channel: #llvm on irc.oftc.net
- Mailing lists:

  - [cfe-dev](http://lists.cs.uiuc.edu/mailman/listinfo/cfe-dev) for questions and general discussions
  - [cfe-commits](http://lists.cs.uiuc.edu/mailman/listinfo/cfe-commits) for patches

- [Phabricator](http://llvm-reviews.chandlerc.com/) to submit patches

## Final word

Finally I want to thank my mentor Edwin Vane and his team at Intel, Tareq Siraj and Ariel Bernal, for the great support they provided me. Also thanks to the LLVM community and Google Summer of Code team for giving me this opportunity to work on the C++ Modernizer this summer.

\-\- Guillaume Papin
