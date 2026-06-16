---
title: Detect empty macro arguments
url: https://gustedt.wordpress.com/2010/06/08/detect-empty-macro-arguments/
published: "2010-06-08T11:58:39Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=37
---

# Detect empty macro arguments

The macro `NARG2` that we introduced in [post](http://wp.me/pWCc9-x) still has a major disadvantage, it will not be able to detect an empty argument list. This is due to a fundamental difference between C and its preprocessor. For C a parenthesis ` ()` is empty and contains no argument. For the preprocessor it contains just one argument, and this argument is the empty token.

So in fact `NARG2` is cheating. It doesn’t count the number of arguments that it receives, but returns the number of commas plus one. In particular, even if it receives an empty argument list it will return `1`. The following two macros better expresses this property:

```
#define _ARG16(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, ...) _15
#define HAS_COMMA(...) _ARG16(__VA_ARGS__, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0)

```

As before, these are constrained to a maximum number of arguments (here 16), but you will easily work out how to extend it to larger maximum values.

Now, when we want to write a macro that detects an empty argument we will be using the feature that a function macro that is not followed by an open parenthesis will be left alone. This we will do with the following macro that just transforms the following parenthesis and contents into a comma.

```
#define _TRIGGER_PARENTHESIS_(...) ,

```

The idea is to put the arguments that we want to test between the macro and its parenthesis, such that the macro only triggers if the arguments are empty:

```
_TRIGGER_PARENTHESIS_ __VA_ARGS__ (/*empty*/)

```

To do so we, must first check for some corner cases where special properties of the argument might mimic the behavior that we want to test.

```
#define ISEMPTY(...)                                                    \
_ISEMPTY(                                                               \
          /* test if there is just one argument, eventually an empty    \
             one */                                                     \
          HAS_COMMA(__VA_ARGS__),                                       \
          /* test if _TRIGGER_PARENTHESIS_ together with the argument   \
             adds a comma */                                            \
          HAS_COMMA(_TRIGGER_PARENTHESIS_ __VA_ARGS__),                 \
          /* test if the argument together with a parenthesis           \
             adds a comma */                                            \
          HAS_COMMA(__VA_ARGS__ (/*empty*/)),                           \
          /* test if placing it between _TRIGGER_PARENTHESIS_ and the   \
             parenthesis adds a comma */                                \
          HAS_COMMA(_TRIGGER_PARENTHESIS_ __VA_ARGS__ (/*empty*/))      \
          )

```

Here we distinguish four different cases, of which the last is just the main idea as exposed above. The first helps to exclude the trivial case, that ` __VA_ARGS__` already contains a comma by itself. The two others test if the two possible combinations of `_TRIGGER_PARENTHESIS_`, ` __VA_ARGS__` and `(/*empty*/)` also already trigger a comma to their output.

Now the outcome of this will be calling the macro ` _ISEMPTY` with four different 0-1-values according to different cases that `__VA_ARGS__` presents. In particular, the case that `__VA_ARGS__` is empty corresponds exactly to the outcome `_ISEMPTY(0, 0, 0, 1)`. All other outcomes will indicate that it was non-empty. We will detect this case with the following helper macro.

```
#define _IS_EMPTY_CASE_0001 ,

```

and we leave all the other 15 cases undefined. Now with

```
#define PASTE5(_0, _1, _2, _3, _4) _0 ## _1 ## _2 ## _3 ## _4
#define _ISEMPTY(_0, _1, _2, _3) HAS_COMMA(PASTE5(_IS_EMPTY_CASE_, _0, _1, _2, _3))

```

we will exactly detect the case we are interested in.

As a test here comes all of that together in a block. This is not a reasonable C program but just something to run through the preprocessor to test the validity of the approach.

```
#define _ARG16(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, ...) _15
#define HAS_COMMA(...) _ARG16(__VA_ARGS__, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0)
#define _TRIGGER_PARENTHESIS_(...) ,

#define ISEMPTY(...)                                                    \
_ISEMPTY(                                                               \
          /* test if there is just one argument, eventually an empty    \
             one */                                                     \
          HAS_COMMA(__VA_ARGS__),                                       \
          /* test if _TRIGGER_PARENTHESIS_ together with the argument   \
             adds a comma */                                            \
          HAS_COMMA(_TRIGGER_PARENTHESIS_ __VA_ARGS__),                 \
          /* test if the argument together with a parenthesis           \
             adds a comma */                                            \
          HAS_COMMA(__VA_ARGS__ (/*empty*/)),                           \
          /* test if placing it between _TRIGGER_PARENTHESIS_ and the   \
             parenthesis adds a comma */                                \
          HAS_COMMA(_TRIGGER_PARENTHESIS_ __VA_ARGS__ (/*empty*/))      \
          )

#define PASTE5(_0, _1, _2, _3, _4) _0 ## _1 ## _2 ## _3 ## _4
#define _ISEMPTY(_0, _1, _2, _3) HAS_COMMA(PASTE5(_IS_EMPTY_CASE_, _0, _1, _2, _3))
#define _IS_EMPTY_CASE_0001 ,

#define EATER0(...)
#define EATER1(...) ,
#define EATER2(...) (/*empty*/)
#define EATER3(...) (/*empty*/),
#define EATER4(...) EATER1
#define EATER5(...) EATER2
#define MAC0() ()
#define MAC1(x) ()
#define MACV(...) ()
#define MAC2(x,y) whatever
ISEMPTY()
ISEMPTY(/*comment*/)
ISEMPTY(a)
ISEMPTY(a, b)
ISEMPTY(a, b, c)
ISEMPTY(a, b, c, d)
ISEMPTY(a, b, c, d, e)
ISEMPTY((void))
ISEMPTY((void), b, c, d)
ISEMPTY(_TRIGGER_PARENTHESIS_)
ISEMPTY(EATER0)
ISEMPTY(EATER1)
ISEMPTY(EATER2)
ISEMPTY(EATER3)
ISEMPTY(EATER4)
ISEMPTY(MAC0)
ISEMPTY(MAC1)
ISEMPTY(MACV)
/* This one will fail because MAC2 is not called correctly */
ISEMPTY(MAC2)

```

***Edit:*** What is presented here is a version with minor improvement to capture the case of `MAC0`. Notice the restriction on the argument of `ISEMPTY` that is apparent with `MAC2`. In fact `ISEMPTY` should work when it is called with macros as argument that expect `0`, `1` or a variable list of arguments. If called with a macro `X` as an argument that itself expects more than one argument (such as `MAC2`) the expansion leads to an invalid use of that macro `X`.
