---
title: Clang ♥ bash -- better auto completion is coming to bash
url: https://blog.llvm.org/2017/09/clang-bash-better-auto-completion-is.html
published: "2017-09-19T23:37:00Z"
feed: llvm
guid: https://blog.llvm.org/2017/09/clang-bash-better-auto-completion-is.html
---

# Clang ♥ bash -- better auto completion is coming to bash

[![](https://4.bp.blogspot.com/-01JogPSFCBM/WcKRj79NXNI/AAAAAAAANZA/X1Mr4wf_cnI57LGLoglYPn_XZWE_q-6egCLcBGAs/s640/out.gif)](https://4.bp.blogspot.com/-01JogPSFCBM/WcKRj79NXNI/AAAAAAAANZA/X1Mr4wf_cnI57LGLoglYPn_XZWE_q-6egCLcBGAs/s1600/out.gif)

Compilers are complex pieces of software and have a multitude of command-line options to fine tune parameters. Clang is no exception: it has 447 command-line options. It’s nearly impossible to memorize all these options and their correct spellings, that's where shell completion can be very handy. When you type in the first few characters of a flag and hit tab, it will autocomplete the rest for you.

Background

However, such a autocompletion feature is not available yet, as there's no easy way to get a complete list of the options Clang supports. For example, bash doesn’t have any autocompletion support for Clang, and despite some shells like zsh having a script for command-line autocompletion, they use hard coded lists of command-line options, and are not automatically updated when a new option is added to Clang. These shells also can’t autocomplete arguments which some flags take (-std=\[tab\] for instance).

This is the problem we were working to solve during this year’s Google Summer of Code. We’re adding a feature to Clang so that we can implement a complete, exact command-line option completion which is highly portable for any shell. To start with, we'll provide a completion script for bash which uses this feature.

Implementation

Clang now has a new command line option called --autocomplete. This flag receives the incomplete user input from the shell and then queries the internal data structures of the current Clang binary, and returns a list of possible completions. With this API, we can always get an accurate list of options and values any time, on any newer versions of Clang.

We built an autocompletion using this in bash for the first implementation. You can find its source code [here](https://github.com/llvm-mirror/clang/blob/master/utils/bash-autocomplete.sh). Also, [here](https://github.com/Teemperor/clang-autocomplete-qt-demo/blob/master/clangcompleter.cpp#L29) is the sample for Qt text entry autocompletion to give an example how to use this API from an UI application as seen below:

![final.gif](https://lh6.googleusercontent.com/ZMCiIg-nvoKUCVI3yrIxajonOBqEL1QiievBUEFeL3zNp-E6Ua6na5u6cenXMteX4XjqH2t_ve584cTA4Hw6MNczz6UtTOr_MpnejkI61fEhbsuhmIkccA7cUgmyIvyR4dni98dX)

You can always complete one flag at a time. So if you want to use the API, you have to select the flag that the user is currently typing. Then just pass this flag to the --autocomplete flag in the selected clang binary. So in the case below all flags start with \`-tr\` are displayed with their descriptions behind them (separated from the flag with a tab character).

[![](https://2.bp.blogspot.com/-VRKNRM79brs/WcKM_j0JZdI/AAAAAAAANYg/0JosgCr4lo0VphdvQMqGSTXBwtqzGIOqwCLcBGAs/s640/tr.png)](https://2.bp.blogspot.com/-VRKNRM79brs/WcKM_j0JZdI/AAAAAAAANYg/0JosgCr4lo0VphdvQMqGSTXBwtqzGIOqwCLcBGAs/s1600/tr.png)

The API also supports completing the values of flags. If you have a flag for which value completion is supported, you can also provide an incomplete value behind the flag separated by a comma to get completion for this:

[![](https://2.bp.blogspot.com/-bu1RSxe2PjQ/WcKNLOcnCqI/AAAAAAAANYk/xu41ZdQbPKcIoR1VpjLcI_055YCWv_lsgCLcBGAs/s640/stdlib.png)](https://2.bp.blogspot.com/-bu1RSxe2PjQ/WcKNLOcnCqI/AAAAAAAANYk/xu41ZdQbPKcIoR1VpjLcI_055YCWv_lsgCLcBGAs/s1600/stdlib.png)

If you provide nothing after the comma, the list of the all possible values for this flag is displayed.

[![](https://4.bp.blogspot.com/-7CZRQ3DJUMA/WcKNQ_GeYgI/AAAAAAAANYo/YsnMXn5EcUIjPrO-y5nEVj6GvuEaRZ9jACLcBGAs/s640/meabi.png)](https://4.bp.blogspot.com/-7CZRQ3DJUMA/WcKNQ_GeYgI/AAAAAAAANYo/YsnMXn5EcUIjPrO-y5nEVj6GvuEaRZ9jACLcBGAs/s1600/meabi.png)

How to get it
This feature is available for use now with LLVM/clang 5.0 and we’ll also be adding this feature to the standard bash completion package. Make sure you have the latest clang version on your machine, and source [this script](https://github.com/llvm-mirror/clang/blob/master/utils/bash-autocomplete.sh). If want to make the change permanent, just source it from your .bashrc and enjoy typing your clang invocations!
