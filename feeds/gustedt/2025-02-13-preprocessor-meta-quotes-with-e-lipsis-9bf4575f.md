---
title: Preprocessor meta-quotes with eĿlipsis
url: https://gustedt.wordpress.com/2025/02/13/preprocessor-meta-quotes-with-ellipsis/
published: "2025-02-13T10:40:05Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=4290
---

# Preprocessor meta-quotes with eĿlipsis

The new revision of [eĿlipsis](https://codeberg.org/gustedt/ellipsis) ( `20250219`) has a lot of cleanups, bugfixes etc, but one thing I’d like to emphasize is a new feature that I’d call meta-quotes in lack for a better idea of a name that implement exemption of tokens from macro replacement. So in C that would interact with translation phase 4.

The idea is to have two new quote characters `⸤` and `⸥` that protect everything between them to be macro expanded.

When, after lexing, the input tokens are filtered by a C-like preprocessor, a nominal (identifier in C speak) that correspond to the name of a macro is expanded. C only has two exceptions from this:

- If the current filtering is the expansion of a macro named `ID`, other occurrences of that same name `ID` are not expanded. So preprocessing has no normal recursion.
- If a macro name `ID` of a functional macro (that is a macro with a parameter list) occurs and that name is not followed by an opening parenthesis (possibly preceded by whitespace) then this token is not expanded, either.

The lack of the possibility to protect other nominals can result in a lot of irritation in contexts where names of identifiers are glued together by means of the token join operator `⨝` ( `##` in C). You’d see things such as the following quite commonly in preprocessor
oriented C headers:

```
#define MYFUNC(F) hurlipurtz ## F

```

The user then expects the following

```
    void* MYFUNC(getone)();

```

to expand to

```
    void* hurlipurtzgetone();

```

But in fact it wouldn’t always. If this code is located in a header and another header is included that would define `hurlipurtz` to expand to `schnurtz`, the above function declaration would read.

```
    void* schnurtzgetone();

```

not at all what was intended. Obviously for the given example this is highly unlikely but the smaller the name components that are used get, the higher the probability of collision. For example a lot of code would use the token `_` to glue together two name components, so whenever some smart colleague thinks they should overload that token with something funny, you are in trouble.

To protect against this effect, as an extension, eĿlipsis adds a third method for identifier protection, the `keep` and `peek` brackets `⸤` and `⸥`. Rewritten with these, the above definition would look

```
#define MYFUNC(F) ⸤hurlipurtz⸥ ## F

```

That is, here the funny brackets ( **BOTTOM LEFT HALF BRACKET** and **BOTTOM RIGHT HALF BRACKET**) protect the nominal `hurlipurtz` to be expanded when `MYFUNC` is used. If you are programming in C and are allergic to Unicode characters you could also use the digraphs `<|` and `|>` that replace them.

These brackets have the following properties

- They don’t interact with tokenization. That is, the input stream is split up into tokens, first, and possible `keep` and `peek` brackets are identified during that process as tokens of their own. They only take their special role in the next phase, filtering, when macros are expanded and directives are interpreted.
- During filtering they apply to arbitrary long sequences of tokens where they protect all nominals that appear in the sequence.
- They can be nested as long `keep` and `peek` brackets nest properly.
- If a construct appears between these brackets that is normally ended by an EOL character, the construct extends to the end of the outermost pair of `keep` and `peek` brackets.

The latter can be in particular be useful for long macro definition that go over multiple lines

```
<|
#define SWAP(NAME, T)
 ⸤void⸥ NAME ## ⸤_swap⸥(T* ⸤a⸥, T* ⸤b⸥) {
    T* ⸤tmp⸥ = ⸤a⸥;
    ⸤a⸥ = ⸤b⸥;
    ⸤b⸥ = ⸤tmp⸥;
 }
|>

```

or equivalently

```
<|
#define SWAP(NAME, T)
 ⸤void NAME ## _swap(T* a, T* b) {⸥
    ⸤T* tmp = a;⸥
    ⸤a = b;⸥
    ⸤b = tmp;⸥
 ⸤}⸥
|>

```

or even

```
<|
#define SWAP(NAME, T)
 <|void NAME ## _swap(T* a, T* b) {
    T* tmp = a;
    a = b;
    b = tmp;
 }|>
|>

```

Such a definition ensures that the text editor (IDE whatever) basically sees a function, and thus code indentation and highlighting should work as expected. The nested used of `keep` and `peek` brackets then also guarantees that the local identifiers `a`, `b` and `tmp` are not accidentally overwritten during macro expansion. So using this macro as `SWAP(fine, well)` would expand to

```
void fine_swap(well* a, well* b) {
    well* tmp = a;
    a = b;
    b = tmp;
}

```

regardless if somebody had defined a macro `tmp` before or not.

This new release of eĿlipsis uses this new feature extensively in a completely revisited implementation of [the `defer` feature](https://gustedt.gitlabpages.inria.fr/ellipsis/extensions.html#defer_desc) that should be quite robust now and that closely follows the bracketed structure of C (for brackets, braces and parenthesis). Namely there is now a feature (in C) to glue callbacks to this syntactic structure of the code called `BLOCKSTATE` that automatically tracks integer values that are associated to different levels. For example a macro invocation

```
__BLOCKSTATE_TST(MINE)

```

would check if a `MINE` value is set for the current level of `{}` (called `BRACE_LEVEL`) of `()` (called `__PARENTHESIS_DEPTH__`) and of `[]` (called `__BRACKET_LEVEL__`) and return that value or `0` if there wasn’t one, yet. Similarly there are macros `__BLOCKSTATE_SET0`, `__BLOCKSTATE_INC` etc to manipulate state that depends on the current block structure of the program.

The `defer` implementation uses this feature for example to keep track if the current level of braces is the body of a `for`-loop or rather a `switch` to deduce if an unwind of deferred blocks for a `continue` statement ends on this level of braces or has to go down on the next level.

To implement all of this, the new meta-quote feature has been really helpful. Otherwise I would easily have been lost in the expansion (or not) of all the name components that are glued together in that code.

Last but not least I’d also like to note that the eĿlipsis project is now to be found at [Codeberg](https://codeberg.org) at

[https://codeberg.org/gustedt/ellipsis/](https://codeberg.org/gustedt/ellipsis/)

Being previously at the gitlab of my employer, INRIA, has not proven to be much helpful. I am very grateful to the Codeberg initiative for providing such an open and free platform for open source development. If you don’t have an account there yet, go and get it.

The online documentation is still at the INRIA site, so the link hasn’t changed:

[https://gustedt.gitlabpages.inria.fr/ellipsis/](https://gustedt.gitlabpages.inria.fr/ellipsis/)
