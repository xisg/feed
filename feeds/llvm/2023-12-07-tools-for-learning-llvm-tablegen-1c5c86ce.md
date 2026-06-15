---
title: Tools for Learning LLVM TableGen
url: https://blog.llvm.org/posts/2023-12-07-tools-for-learning-llvm-tablegen/
published: "2023-12-07T00:00:00Z"
feed: llvm
guid: https://blog.llvm.org/posts/2023-12-07-tools-for-learning-llvm-tablegen/
---

# Tools for Learning LLVM TableGen

[TableGen](https://github.com/llvm/llvm-project/tree/main/llvm/utils/TableGen) is a language used within the LLVM project for generating a variety of files,when manual maintenance would be very difficult.

For example, it is used to define all of the instructions that can be used on aparticular architecture. The information is defined in TableGen and we canproduce many things based on that single source file. C++ code, documentation,command line options, and so on.

TableGen has been in existence since [before](https://github.com/llvm/llvm-project/commit/a6240f6b1a34f9238cbe8bc8c9b6376257236b0a) the first official release of LLVM, over 20 years ago.

Today in the [LLVM project repository](https://github.com/llvm/llvm-project) there areover a thousand TableGen source files totalling over 500,000 lines of code.Making it the 5th most popular language in the repository.

LanguagefilesblankcommentcodeC++2964295854218701015544445C/C++ Header118443168064998451486165C1053525990016035941011269Assembly106944780351222315820236TableGen13129411283616580289

(Counted from [this commit](https://github.com/llvm/llvm-project/commit/ba24b814f2a20a136f0a7a0b492b6ad8a62114c6),rest of table omitted)

With projects such as MLIR [embracing TableGen](https://mlir.llvm.org/docs/DefiningDialects/Operations/),it is only going to grow. So if you are contributing to LLVM, you will encounterit at some point.

Which might be a problem as TableGen only exists within LLVM. Unlike a languagesuch as C++, TableGen does not have a large array of resources.

So, as well as joining a new project, you also need to learn a newDomain Specific Language (DSL). You did not come to LLVM to learn a DSL, youprobably came here to write a compiler.

I cannot say when this problem might be solved, but the situation is not asbleak as it appears. There have been big improvements in TableGen toolsrecently, which means you can put more of your energy into the goals thatbrought you to LLVM in the first place.

# A Brief Introduction to TableGen

Imagine you wanted to represent the registers of an architecture. I am going touse Arm’s AArch64 in particular here.

You could describe them in TableGen as:

```bash
$ cat register.tdclass Register<int _size, string _alias=""> { int size = _size; string alias = _alias;}// 64 bit general purpose registers are X<N>.def X0: Register<8> {}// Some have special alternate names.def X29: Register<8, "frame pointer"> {}// Some registers omitted...
```

By default, the TableGen compiler `llvm-tblgen` creates “records” - which areshown below.

```bash
$ ./bin/llvm-tblgen register.td------------- Classes -----------------class Register<int Register:_size = ?, string Register:_alias = ""> { int size = Register:_size; string alias = Register:_alias;}------------- Defs -----------------def X0 { // Register int size = 8; string alias = "";}def X29 { // Register int size = 8; string alias = "frame pointer";}
```

This is the intermediate representation (IR) of the TableGen compiler, similarto LLVM’s “LLVM IR”.

When using LLVM you would select a “target” which is the processor architectureyou want to generate instructions for. TableGen’s equivalent is a “backend”.These backends do not generate instructions, but instead output a format forthat backend’s specific use case.

For example, there is a backend that generates C++ code for [searching](https://godbolt.org/z/5c696j1f9) data tables. Other examples areC header files and [reStructuredText](https://docutils.sourceforge.io/rst.html) documentation.

```
 TableGen source | +--llvm-tblgen----------------|------------------------+ | v | | +----- Expanded records ----+ | | | | | | v v | | +-------------------------+ +-------------------+ | | | --gen-searchable-tables | | Other backends... | | | +-------------------------+ +-------------------+ | | | | | +--------------|---------------------------|-----------+ v v .inc file with C++ code Other output formats... for table searching.
```

The main compiler is `llvm-tblgen`, but there are others specific tosub-projects of LLVM. For example `clang-tblgen` and `lldb-tblgen`. The onlydifference is the backends included in each one, the language is the same.

You might take your register definitions and produce C++ code to initialise themin some kind of bootloader. Perhaps you also document it and produce a diagramof the process. With enough backends, you could do all that from the sameTableGen source code.

You would write these backends either in C++ within the TableGen compiler,or as an external backend using the compiler’s [JSON output](https://godbolt.org/z/vre845e77) ( `--dump-json`). So you can useany language with a JSON parser (such as [Python](https://github.com/llvm/llvm-project/blob/main/llvm/utils/TableGen/jupyter/sql_query_backend.ipynb)).

# There is TableGen and There Are Things Built With TableGen

This is more a mindset than a tool. It is summed up best by a quote from the [documentation](https://llvm.org/docs/TableGen/index.html#tablegen-deficiencies):

> Despite being very generic, TableGen has some deficiencies that have beenpointed out numerous times. The common theme is that, while TableGen allowsyou to build domain specific languages, the final languages that you createlack the power of other DSLs, which in turn increase considerably the size andcomplexity of TableGen files.
>
> At the same time, TableGen allows you to create virtually any meaning of thebasic concepts via custom-made backends, which can pervert the original designand make it very hard for newcomers to understand the evil TableGen file.”

This means that you will be tackling TableGen, and things built with TableGen.Which are often more complicated than the language.

It is like learning C++ and struggling to use [Boost](https://www.boost.org/).Someone might say to you, “Boost is not required, why not remove it and saveyourself the hassle?”. As someone new to C++, you might not be aware of theboundary between the two of them.

Of course this does not help you too much if the project you want to contributeto uses Boost. You are stuck dealing with both. In LLVM terms, the TableGenlanguage and the backends that consume it are a package deal.

I mention this so that you can draw a distinction between not understandingone or the other. Knowing which one is confusing you is a big advantageto finding help.

For any task there are probably one or two “things built with TableGen” that youneed to understand and even then, not entirely.

Do not think that your TableGen journey must end with understanding all the waysit is used. That is possible, but it is not required, and hardly anyone learnseverything. Instead put your energy into the things that really interest you.

# Compiler Explorer

Of course we have TableGen in Compiler Explorer! Is a language even real if it isnot in Compiler Explorer?

(Of course it is, but if your favourite language is not there, Compiler Explorerhas [excellent documentation](https://github.com/compiler-explorer/compiler-explorer/blob/main/docs/AddingALanguage.md) and friendly maintainers)

Compiler Explorer is a whole bunch of different versions of compilers fordifferent languages and different architectures that you can access with just abrowser tab.

It is an incredible tool for learning, teaching, triaging, optimising and [many more](https://www.youtube.com/watch?v=O5sEug_iaf4) things. I will not go intodetail about it here, just a few things about TableGen’s inclusion.

The obvious thing is that `llvm-tblgen` does not emit instructions (though ahypothetical backend could) so there is no option to compile to binary orexecute code.

By default, records are printed as plain text. You can choose a backend by adding acompiler option, or by opening the “Overrides” menu and selecting an “Action”.

It is important to note that TableGen backends have very specific expectations ofwhat will be in the source code. As if you had a C++ compiler thatwould not compile for Arm unless it saw `arm_is_cool` somewhere in thesource code.

In the LLVM repository all the required classes are set up for you, but inCompiler Explorer they are not. So, if you would like to experiment with anexisting backend, I suggest you provide stub implementations of the classes, orcopy some from the LLVM project repository. You can also use standard includesfrom `include/llvm/*.td`.

It is not possible at this time to develop a backend within Compiler Explorer,but you can select the JSON backend and copy that JSON to give to local scripts.

Multi-file projects (“IDE mode”) also work as expected, so, if you would like,you can have your own [include files](https://godbolt.org/z/4qhdoaMjE).

Finally, remember that you can share Compiler Explorer examples. If you areasking or answering questions about TableGen, always include a Compiler Explorerlink if you can!

# Jupyter Notebooks

[Jupyter](https://jupyter.org/) creates interactive notebooks. A notebook is asingle document which contains text, code and the results of running that code.This enables you to edit the code and rerun it to update the results in thenotebook.

This is great for taking notes or building up large examples from small chunksof code. You can export the document as a notebook that anyone can edit, orin noninteractive formats such as PDF or Markdown.

TableGen can be used in notebooks by using the TableGen Jupyter Kernel.Installation instructions are available [here](https://github.com/llvm/llvm-project/tree/main/llvm/utils/TableGen/jupyter) and you can watch me talk more about it [here](https://www.youtube.com/watch?v=Gf0FUiY2TRo).

**Note:** There is also an [MLIR kernel](https://github.com/llvm/llvm-project/tree/main/mlir/utils/jupyter) for Jupyter, along with many others.

We have aimed to give the same experience as other languages, so I will focusnot on how to use a notebook, but instead on what we have been able to make withthem.

## TableGen Tutorial Notebook

This notebook is an introduction to TableGen. You can read it on [GitHub](https://github.com/llvm/llvm-project/blob/main/llvm/utils/TableGen/jupyter/tablegen_tutorial_part_1.ipynb),or [download](https://raw.githubusercontent.com/llvm/llvm-project/main/llvm/utils/TableGen/jupyter/tablegen_tutorial_part_1.ipynb) it and read it in Jupyter.

When using Jupyter, you can edit the document to add your own examples or expandthe ones that you find interesting.

## “How to Write a TableGen Backend” Notebook

This notebook uses Python instead of TableGen, and it shows you how to write abackend.

The 2021 EU LLVM Developer’s Meeting talk [“How to write a TableGen backend”](https://www.youtube.com/watch?v=UP-LBRbvI_U) by Min-Yih Hsu is the basis for this. The [notebook](https://github.com/llvm/llvm-project/blob/main/llvm/utils/TableGen/jupyter/sql_query_backend.ipynb) is in fact a Python port of Min’s own [C++](https://github.com/mshockwave/SQLGen) implementation.

It shows you how to take the JSON output of `llvm-tblgen` and process it withPython to create SQL queries.

What is unique here is we now have the same content in multiple media forms andmultiple programming languages. Choose the ones that suit you best.

Referring back to “There is TableGen and There are Things Built With TableGen”, the tutorial notebook is TableGen. The writing a backend notebook is “ThingsBuilt With TableGen”.

## Limitations

The major limitation of the notebooks is that we have no output filtering. Thismeans if you do `include “llvm/Target/Target.td"` you will get about 320,000lines of output (before you have added any of your own code). This is more thana default notebook accepts from a kernel and when I removed that limit, thebrowser tab crashed.

This is not a problem in most cases and the possible solutions have bigtrade-offs, so we are not going to rush a fix. If it does affect you, please add yourfeedback to the [tracking issue](https://github.com/llvm/llvm-project/issues/72856).

# TableGen Language Server

The MLIR project has implemented a server for the [Language Server Protocol](https://microsoft.github.io/language-server-protocol/)(LSP). Which supports TableGen and [2 other languages](https://mlir.llvm.org/docs/Tools/MLIRLSP/) used within MLIR.

The language server protocol provides information to compatible editors aboutthe structure of a language and project. For example, where are the includedfiles? Where is the definition of a particular type?

If you have used a LSP compatible editor (such as Visual Studio Code), you haveprobably used a language server without knowing. “Go To Definition” is themost common feature they provide.

The Language Server Protocol allows you to open a project, go to the code youwant to change and jump from there directly to the other relevant parts of therepository. With 500,000+ lines of TableGen in the LLVM project, that is a lot ofcode you get to ignore!

# Setup

You will need a copy of the server binary `tblgen-lsp-server`. Which you can getfrom the [release package](https://github.com/llvm/llvm-project/releases) for yourplatform, or you can build it yourself.

This is how to build it yourself:

```bash
$ cmake -G Ninja <path-to>/llvm-project/llvm -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_PROJECTS="mlir"$ ninja tblgen-lsp-server
```

Having run those commands, `tblgen-lsp-server` is found in `<build-dir>/bin/`.

The server reads a compilation database file `tablegen_compile_commands.yml`,which is made for you when you configure LLVM using CMake.

This serves a similar purpose to the `compile_commands.json` file generated when using `CMAKE_EXPORT_COMPILE_COMMANDS`, but the two files are not related.

As long as your checkout of llvm-project includes [this commit](https://github.com/llvm/llvm-project/commit/c4afeccdd235a282d200c450e06a730504a66a08) the compilation database includes TableGen files from all enabledprojects (prior to that commit it was MLIR only).

For example this configure command includes information about TableGen files from theLLVM, Clang, MLIR and LLDB subprojects:

```bash
$ cmake -G Ninja <path-to>/llvm-project/llvm -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_PROJECTS="clang;llvm;lldb;mlir"
```

This also applies to `-DLLVM_TARGETS_TO_BUILD=`. Enabling only one target meansthat the compilation database only has files relevant to that target.

**Note:** You do not need to build a project to include its TableGen files inthe compilation database. Configuring is all that is needed.

Next, configure the LSP client for your editor.

If you are using Visual Studio Code, install the MLIR [extension](https://marketplace.visualstudio.com/items?itemName=llvm-vs-code-extensions.vscode-mlir). Then follow the setup instructions [here](https://mlir.llvm.org/docs/Tools/MLIRLSP/#td---tablegen-files) to tellthe extension where the server and compilation database are.

If you are using a different editor, refer to its documentation to learn how toset up a language server. Setting the path to the compilation database may requirethe use of the server’s command line options. Run `tblgen-lsp-server --help` tosee all available options.

## Example

This example assumes you have configured LLVM with the `AArch64` target enabled.(It is enabled by default)

- Open the file `llvm/lib/Target/AArch64/AArch64.td`.
- Put your cursor on a use of the `SubtargetFeature` type.
- In the menu bar, select “Go” then “Go to Definition”.
- This takes you to `llvm/include/llvm/Target/Target.td`, where `SubtargetFeature` is defined.

## Limitations

The language server highlights an anti-pattern in the way some LLVM targetssuch as AArch64 use TableGen.

You may find yourself in a file that uses a class but does not define it orinclude any files which define it. This is because this file is intended to beincluded in another file, which does include a definition of that class.

```
example.td: class Example {}uses_example.td: def example: Example {}main.td: include "example.td" include "uses_example.td"
```

The example above shows this anti-pattern:

- The file `example.td` defines the class `Example`.
- `uses_example.td` uses the class `Example`, but does not include `example.td`.
- `main.td` includes both `example.td` and `uses_example.td`.
- `main.td` is the file that is compiled.
- When you are in `uses_example.td`, the language server does not know where `Example` is defined,
- When you are in `main.td`, the language server does know where `Example` isdefined.

Perhaps we can address this by improving the language server, or reorganisingthe includes so we do not have files that appear to be isolated.

# Dump

What about `printf`? The best debugging tool of them all.

TableGen’s equivalent is [dump](https://llvm.org/docs/TableGen/ProgRef.html#dump-print-messages-to-stderr),and its companion `repr`.

```
def op;class A { string A = "some text"; dag X =(op op);}def a : A;dump "The Value of a is: \n" # !repr(a);
```

`dump` prints to `stderr`:

```
<source>:8:1: note: The Value of a is:a {// A string A = "some text"; dag X = (op op);}dump "The Value of a is: \n" # !repr(a);^
```

This was added [recently](https://github.com/llvm/llvm-project/commit/411c4edeef076bd2e01b104fe095ba381600a3d3).So you will need a recent build, or a released version 18.0 or newer (which is unreleasedat time of writing).

Of course you can try this [on Compiler Explorer](https://godbolt.org/z/Ta6jb19hr) right now!

# Assertions

An assertion checks that a condition is true at a specific point in yourprogram. An assertion consists of:

- The keyword `assert`.
- A condition (usually a call to one of the [bang operators](https://llvm.org/docs/TableGen/ProgRef.html#bang-operators)).
- A message.

If the condition is false, a compiler error is generated with the message youprovided.

For example, the code below checks that you have not tried to make a registerwith a size that is less than 0.

```
class Register<int _size> { assert !gt(_size, 0), "Register size must be > 0, not " # _size # "." ; int size = _size;}def X0: Register<8> {}def X1: Register<-8> {}
```

( [Try this on Compiler Explorer](https://godbolt.org/z/e4GzvhEeh))

The register `X0` has `_size=8`, so the condition `!gt(_size, 0)` (which wouldbe `_size > 0` in C syntax) is true and therefore no error is generated.

The register `X1` has `_size=-8`, so the condition is false and an error isgenerated. The compiler output is shown below:

```
<source>:2:11: error: assertion failed assert !gt(_size, 0), ^note: Register size must be > 0, not -8.
```

While learning new code it is helpful to add your own assertions to check yourassumptions. In addition, adding assertions to code written to be used by otherpeople is a good way to stop them using it incorrectly. Unlike documentation,you cannot miss an assertion error.

# Find In Files

This is last because in an ideal world it would be the last option, but it isoften not the least of the options. Grep, ack, Find In Files, whatever you call it,searching text is unreasonably effective if you have a little knowledge of thelanguage syntax.

Why should I mention such an obvious idea? Well, obvious is subjective, andthere is a special situation that makes it more effective than usual.

In the LLVM project repository we have the vast majority of TableGen code in use today.Would you like to know how to use a particular feature? It is all there,somewhere in 500,000+ lines of source code. You would be surprised by what asimple query can find despite that.

Think about the thing you are trying to find. What do you think its sourcecode would look like? If it is a class would it have template arguments or notand so would there be a `<` after the name? If it is an error message, what partswould be constant and what parts would be inserted into a template message?

`Expected end of line` is likely to be a static string so you can search for themessage itself. In contrast, `class Foo has no attribute Bar` is more likely tobe created by substituting in the name of the class and attribute. So a goodsearch term for this would be `has no attribute`.

There are also tests for the compiler, most of which are in [this folder](https://github.com/llvm/llvm-project/tree/main/llvm/test/TableGen).This folder contains minimal examples for the language features. Try narrowingyour search to this location.

# Conclusion

Learning TableGen does not have to be scary. Do not think that because it is anisolated DSL that it does not have what you have come to expect from yourfavourite languages.

Keep in mind that TableGen is also a tool, not a goal in itself. If you canachieve your goals with a limited but accurate understanding of TableGen and itsbackends, that is great. Learn as much as you want or need.

In addition to the tools, there is an active community ready to answer yourquestions on [Discord](https://discord.com/invite/xS7Z362) or the [forums](https://discourse.llvm.org/).

If you find problems or want to contribute improvements please do so. Open aGitHub [Issue](https://github.com/llvm/llvm-project/issues) or [Pull Request](https://llvm.org/docs/Contributing.html).

Look at the other languages you use. Do they have these tools? Should they? Theymight be the difference between frustration and your new favourite language.

# Acknowledgements

Thank you to Andrzej Warzyński, Francesco Petrogalli, Min-Yih Hsu and Sally Neale (Arm) for reviewing this article.
