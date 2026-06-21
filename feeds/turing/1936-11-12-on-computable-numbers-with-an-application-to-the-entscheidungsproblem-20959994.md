---
title:
  "On Computable Numbers, with an Application to the
  Entscheidungsproblem"
url: https://www.cs.virginia.edu/~robins/Turing_Paper_1936.pdf
published: "1936-11-12T00:00:00Z"
feed: turing
guid: https://www.cs.virginia.edu/~robins/Turing_Paper_1936.pdf
---

# On Computable Numbers, with an Application to the Entscheidungsproblem

_Proceedings of the London Mathematical Society, Series 2, Vol. 42
(1936–37), pp. 230–265, with corrections from Vol. 43 (1937), pp.
544–546._

The "computable" numbers may be described briefly as the real
numbers whose expressions as a decimal are calculable by finite
means. Although the subject of this paper is ostensibly the
computable numbers, it is almost equally easy to define and
investigate computable functions of an integral variable or a real
or computable variable, computable predicates, and so forth. The
fundamental problems involved are, however, the same in each case,
and I have chosen the computable numbers for explicit treatment as
involving the least cumbrous technique. I hope shortly to give an
account of the relations of the computable numbers, functions, and
so forth to one another. This will include a development of the
theory of functions of a real variable expressed in terms of
computable numbers. According to my definition, a number is
computable if its decimal can be written down by a machine.

In §§ 9, 10 I give some arguments with the intention of showing
that the computable numbers include all numbers which could
naturally be regarded as computable. In particular, I show that
certain large classes of numbers are computable. They include, for
instance, the real parts of all algebraic numbers, the real parts
of the zeros of the Bessel functions, the numbers $\pi$, $e$, etc.
The computable numbers do not, however, include all definable
numbers, and an example is given of a definable number which is
not computable.

Although the class of computable numbers is so great, and in many
ways similar to the class of real numbers, it is nevertheless
enumerable. In §8 I examine certain arguments which would seem to
prove the contrary. By the correct application of one of these
arguments, conclusions are reached which are superficially similar
to those of Gödel. These results have valuable applications. In
particular, it is shown (§11) that the Hilbertian
Entscheidungsproblem can have no solution.

In a recent paper Alonzo Church has introduced an idea of
"effective calculability", which is equivalent to my
"computability", but is very differently defined. Church also
reaches similar conclusions about the Entscheidungsproblem. The
proof of equivalence between "computability" and "effective
calculability" is outlined in an appendix to the present paper.

## 1. Computing machines

We have said that the computable numbers are those whose decimals
are calculable by finite means. This requires rather more explicit
definition. No real attempt will be made to justify the
definitions given until we reach §9. For the present I shall only
say that the justification lies in the fact that the human memory
is necessarily limited.

We may compare a man in the process of computing a real number to
a machine which is only capable of a finite number of conditions
$q_1, q_2, \ldots, q_R$ which will be called "$m$-configurations".
The machine is supplied with a "tape" (the analogue of paper)
running through it, and divided into sections (called "squares")
each capable of bearing a "symbol". At any moment there is just
one square, say the $r$-th, bearing the symbol $\mathfrak{S}(r)$
which is "in the machine". We may call this square the "scanned
square". The symbol on the scanned square may be called the
"scanned symbol". The "scanned symbol" is the only one of which
the machine is, so to speak, "directly aware". However, by
altering its $m$-configuration the machine can effectively
remember some of the symbols which it has "seen" (scanned)
previously. The possible behaviour of the machine at any moment is
determined by the $m$-configuration $q_n$ and the scanned symbol
$\mathfrak{S}(r)$. This pair $q_n,\,\mathfrak{S}(r)$ will be
called the "configuration": thus the configuration determines the
possible behaviour of the machine. In some of the configurations
in which the scanned square is blank (i.e. bears no symbol) the
machine writes down a new symbol on the scanned square: in other
configurations it erases the scanned symbol. The machine may also
change the square which is being scanned, but only by shifting it
one place to right or left. In addition to any of these operations
the $m$-configuration may be changed. Some of the symbols written
down will form the sequence of figures which is the decimal of the
real number which is being computed. The others are just rough
notes to "assist the memory". It will only be these rough notes
which will be liable to erasure.

It is my contention that these operations include all those which
are used in the computation of a number. The defence of this
contention will be easier when the theory of the machines is
familiar to the reader. In the next section I therefore proceed
with the development of the theory and assume that it is
understood what is meant by "machine", "tape", "scanned", etc.

## 2. Definitions

**Automatic machines.** If at each stage the motion of a machine
(in the sense of §1) is completely determined by the
configuration, we shall call the machine an "automatic machine"
(or $a$-machine). For some purposes we might use machines (choice
machines or $c$-machines) whose motion is only partially
determined by the configuration (hence the use of the word
"possible" in §1). When such a machine reaches one of these
ambiguous configurations, it cannot go on until some arbitrary
choice has been made by an external operator. This would be the
case if we were using machines to deal with axiomatic systems. In
this paper I deal only with automatic machines, and will therefore
often omit the prefix $a$-.

**Computing machines.** If an $a$-machine prints two kinds of
symbols, of which the first kind (called figures) consists
entirely of $0$ and $1$ (the others being called symbols of the
second kind), then the machine will be called a computing machine.
If the machine is supplied with a blank tape and set in motion,
starting from the correct initial $m$-configuration, the
subsequence of the symbols printed by it which are of the first
kind will be called the sequence computed by the machine. The real
number whose expression as a binary decimal is obtained by
prefacing this sequence by a decimal point is called the number
computed by the machine.

At any stage of the motion of the machine, the number of the
scanned square, the complete sequence of all symbols on the tape,
and the $m$-configuration will be said to describe the complete
configuration at that stage. The changes of the machine and tape
between successive complete configurations will be called the
moves of the machine.

**Circular and circle-free machines.** If a computing machine
never writes down more than a finite number of symbols of the
first kind it will be called circular. Otherwise it is said to be
circle-free.

A machine will be circular if it reaches a configuration from
which there is no possible move, or if it goes on moving, and
possibly printing symbols of the second kind, but cannot print any
more symbols of the first kind. The significance of the term
"circular" will be explained in §8.

**Computable sequences and numbers.** A sequence is said to be
computable if it can be computed by a circle-free machine. A
number is computable if it differs by an integer from the number
computed by a circle-free machine.

We shall avoid confusion by speaking more often of computable
sequences than of computable numbers.

## 3. Examples of computing machines

**I.** A machine can be constructed to compute the sequence
$010101\ldots$. The machine is to have the four $m$-configurations
$\mathfrak{b}, \mathfrak{c}, \mathfrak{e}, \mathfrak{k}$ and is
capable of printing $0$ and $1$. The behaviour of the machine is
described in the following table in which $R$ means "the machine
moves so that it scans the square immediately on the right of the
one it was scanning previously". Similarly for $L$. $E$ means the
scanned symbol is "erased" and $P$ stands for "prints". This table
(and all succeeding tables of the same kind) is to be understood
to mean that for a configuration described in the first two
columns the operations in the third column are carried out
successively, and the machine then goes over into the
$m$-configuration described in the last column. When the second
column is left blank, it is understood that the behaviour of the
third and fourth columns applies for any symbol and for no symbol.
The machine starts in the $m$-configuration $\mathfrak{b}$ with a
blank tape.

|  $m$-config.   | symbol | operations | final $m$-config. |
| :------------: | :----: | :--------: | :---------------: |
| $\mathfrak{b}$ |  None  |  $P0,\ R$  |  $\mathfrak{c}$   |
| $\mathfrak{c}$ |  None  |    $R$     |  $\mathfrak{e}$   |
| $\mathfrak{e}$ |  None  |  $P1,\ R$  |  $\mathfrak{k}$   |
| $\mathfrak{k}$ |  None  |    $R$     |  $\mathfrak{b}$   |

If (contrary to the description in §1) we allow the letters $L$,
$R$ to appear more than once in the operations column we can
simplify the table considerably.

|  $m$-config.   | symbol |  operations  | final $m$-config. |
| :------------: | :----: | :----------: | :---------------: |
| $\mathfrak{b}$ |  None  |     $P0$     |  $\mathfrak{b}$   |
|                |  $0$   | $R,\ R,\ P1$ |  $\mathfrak{b}$   |
|                |  $1$   | $R,\ R,\ P0$ |  $\mathfrak{b}$   |

**II.** As a slightly more difficult example we can construct a
machine to compute the sequence $001011011101111011111\ldots$. The
machine is to be capable of five $m$-configurations, viz.
$\mathfrak{o}, \mathfrak{q}, \mathfrak{p}, \mathfrak{f}, \mathfrak{b}$,
and of printing $\partial, x, 0, 1$. The first three symbols on
the tape will be $\partial\,\partial\,0$; the other figures follow
on alternate squares. On the intermediate squares we never print
anything but $x$. These letters serve to "keep the place" for us
and are erased when we have finished with them. We also arrange
that in the sequence of figures on alternate squares there shall
be no blanks. (Here $\partial$ denotes the symbol Turing writes as
a turned "e", the schwa.)

|  $m$-config.   |      symbol      |                        operations                         | final $m$-config. |
| :------------: | :--------------: | :-------------------------------------------------------: | :---------------: |
| $\mathfrak{b}$ |                  | $P\partial,\ R,\ P\partial,\ R,\ P0,\ R,\ R,\ P0,\ L,\ L$ |  $\mathfrak{o}$   |
| $\mathfrak{o}$ |       $1$        |                   $R,\ Px,\ L,\ L,\ L$                    |  $\mathfrak{o}$   |
|                |       $0$        |                                                           |  $\mathfrak{q}$   |
| $\mathfrak{q}$ | Any ($0$ or $1$) |                          $R,\ R$                          |  $\mathfrak{q}$   |
|                |       None       |                         $P1,\ L$                          |  $\mathfrak{p}$   |
| $\mathfrak{p}$ |       $x$        |                          $E,\ R$                          |  $\mathfrak{q}$   |
|                |    $\partial$    |                            $R$                            |  $\mathfrak{f}$   |
|                |       None       |                          $L,\ L$                          |  $\mathfrak{p}$   |
| $\mathfrak{f}$ |       Any        |                          $R,\ R$                          |  $\mathfrak{f}$   |
|                |       None       |                       $P0,\ L,\ L$                        |  $\mathfrak{o}$   |

To illustrate the working of this machine a table of the first few
complete configurations may be written out, each by writing down
the sequence of symbols on the tape with the $m$-configuration
written below the scanned symbol, the successive complete
configurations being separated by colons. This same trace can also
be written in a form (which Turing labels (C)) in which a space is
made on the left of the scanned symbol and the $m$-configuration
is written in this space; this form is less easy to follow but is
used later for theoretical purposes.

The convention of writing the figures only on alternate squares is
very useful: I shall always make use of it. I shall call the one
sequence of alternate squares $F$-squares and the other sequence
$E$-squares. The symbols on $E$-squares will be liable to erasure.
The symbols on $F$-squares form a continuous sequence. There are
no blanks until the end is reached. There is no need to have more
than one $E$-square between each pair of $F$-squares: an apparent
need of more $E$-squares can be satisfied by having a sufficiently
rich variety of symbols capable of being printed on $E$-squares.
If a symbol $\beta$ is on an $F$-square $S$ and a symbol $\alpha$
is on the $E$-square next on the right of $S$, then $S$ and
$\beta$ will be said to be marked with $\alpha$. The process of
printing this $\alpha$ will be called marking $\beta$ (or $S$)
with $\alpha$.

## 4. Abbreviated tables

There are certain types of process used by nearly all machines,
and these, in some machines, are used in many connections. These
processes include copying down sequences of symbols, comparing
sequences, erasing all symbols of a given form, etc. Where such
processes are concerned we can abbreviate the tables for the
$m$-configurations considerably by the use of "skeleton tables".
In skeleton tables there appear capital German (Fraktur) letters
and small Greek letters. These are of the nature of "variables".
By replacing each capital German letter throughout by an
$m$-configuration and each small Greek letter by a symbol, we
obtain the table for an $m$-configuration. The skeleton tables are
to be regarded as nothing but abbreviations: they are not
essential.

> _The skeleton tables themselves are dense multi-column tables of
> Fraktur/Greek variables and are not reproduced here (see the
> original, pp. 235–239); their meanings are described below._

Turing builds up a small library of these "$m$-functions". For
example, $\mathfrak{f}(\mathfrak{C},\mathfrak{B},\alpha)$ is the
function which, from its starting configuration, makes the machine
find the symbol of form $\alpha$ which is farthest to the left
(the "first $\alpha$"); the $m$-configuration then becomes
$\mathfrak{C}$, or, if there is no $\alpha$, becomes
$\mathfrak{B}$. Other $m$-functions defined this way include:
$\mathfrak{e}(\mathfrak{C},\mathfrak{B},\alpha)$, which finds the
first $\alpha$, erases it, and goes to $\mathfrak{C}$ (and
$\mathfrak{e}(\mathfrak{B},\alpha)$, which erases all letters
$\alpha$); $\mathfrak{pe}(\mathfrak{C},\beta)$, which prints
$\beta$ at the end of the sequence of symbols and goes to
$\mathfrak{C}$; $\mathfrak{l}(\mathfrak{C})$ and
$\mathfrak{r}(\mathfrak{C})$, which move one square left or right;
$\mathfrak{c}(\mathfrak{C},\mathfrak{B},\alpha)$ and
$\mathfrak{cc}(\mathfrak{C},\mathfrak{B},\alpha)$, which copy
(down at the end) the symbol(s) marked $\alpha$;
$\mathfrak{re}(\mathfrak{C},\mathfrak{B},\alpha,\beta)$, which
replaces the first $\alpha$ by $\beta$; and $\mathfrak{cp}$ /
$\mathfrak{cpe}$, which compare the first symbol marked $\alpha$
with the first marked $\beta$, branching according as they are
alike or not (the $\mathfrak{cpe}$ variant also erasing the
compared symbols). These are composed by repeated substitution to
build the complete tables of the machines that follow.

## 5. Enumeration of computable sequences

A computable sequence $\gamma$ is determined by a description of a
machine which computes $\gamma$. Thus the sequence
$001011011101111\ldots$ is determined by the table on p. 234, and,
in fact, any computable sequence is capable of being described in
terms of such a table.

It will be useful to put these tables into a kind of standard
form. Suppose the table is given in the same form as the first
example (machine I); that is, the entry in the operations column
is always of one of the forms $E$; $E,R$; $E,L$; $P\alpha$;
$P\alpha,R$; $P\alpha,L$; $R$; $L$; or no entry at all. The table
can always be put into this form by introducing more
$m$-configurations. Now let us give numbers to the
$m$-configurations, calling them $q_1, \ldots, q_R$, as in §1. The
initial $m$-configuration is always to be called $q_1$. We also
give numbers to the symbols $S_1, \ldots, S_m$ and, in particular,
blank $= S_0$, $0 = S_1$, $1 = S_2$. The lines of the table are
now of one of the forms

$$
\begin{aligned}
q_i\ \ S_j\ \ PS_k,\,L\ \ q_m & \qquad (N_1)\\
q_i\ \ S_j\ \ PS_k,\,R\ \ q_m & \qquad (N_2)\\
q_i\ \ S_j\ \ PS_k\ \ q_m & \qquad (N_3)
\end{aligned}
$$

(a line giving operation $E,R$ is written as $PS_0,R$, and one
giving no change of symbol as $PS_j,\ldots$). From each line of
form $(N_1)$ we form an expression $q_i\,S_j\,S_k\,L\,q_m$; from
each line of form $(N_2)$ an expression $q_i\,S_j\,S_k\,R\,q_m$;
and from each line of form $(N_3)$ an expression
$q_i\,S_j\,S_k\,N\,q_m$. We write down all the expressions so
formed and separate them by semicolons; this gives a complete
description of the machine. In this description we replace $q_i$
by the letter "$D$" followed by "$A$" repeated $i$ times, and
$S_j$ by "$D$" followed by "$C$" repeated $j$ times. This new
description is the **standard description** (S.D); it is made up
entirely from the letters $A$, $C$, $D$, $L$, $R$, $N$, and from
";".

If finally we replace "$A$" by $1$, "$C$" by $2$, "$D$" by $3$,
"$L$" by $4$, "$R$" by $5$, "$N$" by $6$, and ";" by $7$, we have
a description of the machine as an Arabic numeral. The integer
represented by this numeral is a **description number** (D.N) of
the machine. The D.N determines the S.D and the structure of the
machine uniquely. The machine whose D.N is $n$ may be described as
$\mathcal{M}(n)$.

To each computable sequence there corresponds at least one
description number, while to no description number does there
correspond more than one computable sequence. The computable
sequences and numbers are therefore enumerable.

For the machine I of §3, renaming the $m$-configurations gives the
table

$$
\begin{aligned}
q_1\ \ S_0\ \ PS_1,R\ \ q_2;\quad q_2\ \ S_0\ \ PS_0,R\ \ q_3;\\
q_3\ \ S_0\ \ PS_2,R\ \ q_4;\quad q_4\ \ S_0\ \ PS_0,R\ \ q_1.
\end{aligned}
$$

The standard description is
$$\mathrm{DADDCRDAA;\ DAADDRDAAA;\ DAAADDCCRDAAAA;\ DAAAADDRDA;}$$
and a description number is
$$31332531173113353111731113322531111731111335317.$$

A number which is a description number of a circle-free machine
will be called a **satisfactory number**. In §8 it is shown that
there can be no general process for determining whether a given
number is satisfactory or not.

## 6. The universal computing machine

It is possible to invent a single machine which can be used to
compute any computable sequence. If this machine $\mathcal{U}$ is
supplied with a tape on the beginning of which is written the S.D
of some computing machine $\mathcal{M}$, then $\mathcal{U}$ will
compute the same sequence as $\mathcal{M}$. In this section I
explain in outline the behaviour of the machine; the next section
is devoted to giving the complete table for $\mathcal{U}$.

Suppose we have a machine $\mathcal{M}'$ which writes down on the
$F$-squares the successive complete configurations of
$\mathcal{M}$, expressed (as in the form (C)) with the symbols
replaced by their standard descriptions — each $m$-configuration
by "$D$" followed by the appropriate number of "$A$"s, and each
symbol by "$D$" followed by the appropriate number of "$C$"s, so
that in particular "$0$" is replaced by "$DC$", "$1$" by "$DCC$",
and blanks by "$D$". These substitutions are made after the
complete configurations are put together; doing the substitution
first would require replacing all the blanks, so the complete
configuration would not be a finite sequence of symbols. For
example, in machine II of §3 the sequence (C) becomes

$$\mathrm{DA:\ DCCCDCCCDAADCDDC:\ DCCCDCCCDAAADCDDC:\ \ldots} \qquad (C_1)$$

(the sequence of symbols on the $F$-squares). It is not difficult
to see that if $\mathcal{M}$ can be constructed then so can
$\mathcal{M}'$: the rules of operation (the S.D) of $\mathcal{M}$
are written within $\mathcal{M}'$ itself, and each step is carried
out by referring to these rules. Regarding these rules as capable
of being taken out and exchanged for others gives something very
akin to the universal machine. One thing is lacking —
$\mathcal{M}'$ prints no figures; we correct this by printing,
between each successive pair of complete configurations, the
figures which appear in the new configuration but not in the old,
so that $(C_1)$ becomes a sequence $(C_2)$ beginning
$\mathrm{DDA:0:0:\ldots}$. The sequences of letters between the
colons may be used as standard descriptions of the complete
configurations; when the letters are replaced by figures, as in
§5, we obtain a numerical description number of the complete
configuration.

## 7. Detailed description of the universal machine

Turing gives a complete table for the universal machine
$\mathcal{U}$. Its $m$-configurations are all those occurring in
the first and last columns of the table, together with all those
that arise when the abbreviated $m$-functions are written out in
full.

> _The full table for $\mathcal{U}$ — built from subsidiary
> $m$-functions such as $\mathfrak{con}$ (mark off a
> configuration), $\mathfrak{anf}$, $\mathfrak{kom}$,
> $\mathfrak{sim}$ (find the instruction whose left-hand side
> matches the current configuration), $\mathfrak{mk}$ (mark the
> configuration), $\mathfrak{sh}$, and $\mathfrak{inst}$ (carry
> out the marked instruction) — spans several dense pages (pp.
> 240–246) and is not reproduced here._

When $\mathcal{U}$ is ready to start, the tape bears the symbol
$\partial$ on an $F$-square and again $\partial$ on the next
$E$-square; after this, on $F$-squares only, comes the S.D of the
machine $\mathcal{M}$, followed by a double colon "::" (a single
symbol, on an $F$-square). The S.D consists of a number of
instructions, separated by semicolons. Each instruction consists
of five consecutive parts:

- (i) "$D$" followed by a sequence of letters "$A$" — the relevant
  $m$-configuration;
- (ii) "$D$" followed by a sequence of letters "$C$" — the scanned
  symbol;
- (iii) "$D$" followed by another sequence of letters "$C$" — the
  symbol into which the scanned symbol is to be changed;
- (iv) "$L$", "$R$", or "$N$" — whether the machine moves left,
  right, or not at all;
- (v) "$D$" followed by a sequence of letters "$A$" — the final
  $m$-configuration.

$\mathcal{U}$ is to be capable of printing $A$, $C$, $D$, $0$,
$1$, $u$, $v$, $w$, $x$, $y$, $z$; the S.D itself is formed from
";", $A$, $C$, $D$, $L$, $R$, $N$.

## 8. Application of the diagonal process

It may be thought that arguments which prove that the real numbers
are not enumerable would also prove that the computable numbers
and sequences cannot be enumerable. It might, for instance, be
thought that the limit of a sequence of computable numbers must be
computable. This is clearly only true if the sequence of
computable numbers is defined by some rule.

Or we might apply the diagonal process. "If the computable
sequences are enumerable, let $\alpha_n$ be the $n$-th computable
sequence, and let $\phi_n(m)$ be the $m$-th figure in $\alpha_n$.
Let $\beta$ be the sequence with $1 - \phi_n(n)$ as its $n$-th
figure. Since $\beta$ is computable, there exists a number $K$
such that $1 - \phi_n(n) = \phi_K(n)$ for all $n$. Putting
$n = K$, we have $1 = 2\phi_K(K)$, i.e. $1$ is even. This is
impossible. The computable sequences are therefore not
enumerable."

The fallacy in this argument lies in the assumption that $\beta$
is computable. It would be true if we could enumerate the
computable sequences by finite means, but the problem of
enumerating computable sequences is equivalent to the problem of
finding out whether a given number is the D.N of a circle-free
machine, and we have no general process for doing this in a finite
number of steps. In fact, by applying the diagonal process
argument correctly, we can show that there cannot be any such
general process.

The simplest and most direct proof of this is by showing that, if
this general process exists, then there is a machine which
computes $\beta$. This proof, although perfectly sound, has the
disadvantage that it may leave the reader with a feeling that
"there must be something wrong". The proof which I shall give has
not this disadvantage, and gives a certain insight into the
significance of the idea "circle-free". It depends not on
constructing $\beta$, but on constructing $\beta'$, whose $n$-th
figure is $\phi_n(n)$.

Let us suppose that there is such a process; that is to say, that
we can invent a machine $\mathcal{D}$ which, when supplied with
the S.D of any computing machine $\mathcal{M}$, will test this S.D
and, if $\mathcal{M}$ is circular, will mark the S.D with the
symbol "$u$", and if it is circle-free will mark it with "$s$". By
combining the machines $\mathcal{D}$ and $\mathcal{U}$ we could
construct a machine $\mathcal{H}$ to compute the sequence
$\beta'$. The machine $\mathcal{D}$ may require a tape; we may
suppose that it uses the $E$-squares beyond all symbols on
$F$-squares, and that when it has reached its verdict all the
rough work done by $\mathcal{D}$ is erased.

The machine $\mathcal{H}$ has its motion divided into sections. In
the first $N-1$ sections, among other things, the integers
$1, 2, \ldots, N-1$ have been written down and tested by the
machine $\mathcal{D}$. A certain number, say $R(N-1)$, of them
have been found to be the D.N's of circle-free machines. In the
$N$-th section the machine $\mathcal{D}$ tests the number $N$. If
$N$ is satisfactory, i.e. if it is the D.N of a circle-free
machine, then $R(N) = 1 + R(N-1)$ and the first $R(N)$ figures of
the sequence of which a D.N is $N$ are calculated. The $R(N)$-th
figure of this sequence is written down as one of the figures of
the sequence $\beta'$ computed by $\mathcal{H}$. If $N$ is not
satisfactory, then $R(N) = R(N-1)$ and the machine goes on to the
$(N+1)$-th section of its motion.

From the construction of $\mathcal{H}$ we can see that
$\mathcal{H}$ is circle-free. Each section of the motion of
$\mathcal{H}$ comes to an end after a finite number of steps. For,
by our assumption about $\mathcal{D}$, the decision as to whether
$N$ is satisfactory is reached in a finite number of steps. If $N$
is not satisfactory, the $N$-th section is finished. If $N$ is
satisfactory, then the machine $\mathcal{M}(N)$ whose D.N is $N$
is circle-free, and therefore its $R(N)$-th figure can be
calculated in a finite number of steps. When this figure has been
calculated and written down as the $R(N)$-th figure of $\beta'$,
the $N$-th section is finished. Hence $\mathcal{H}$ is
circle-free.

Now let $K$ be the D.N of $\mathcal{H}$. What does $\mathcal{H}$
do in the $K$-th section of its motion? It must test whether $K$
is satisfactory, giving a verdict "$s$" or "$u$". Since $K$ is the
D.N of $\mathcal{H}$ and since $\mathcal{H}$ is circle-free, the
verdict cannot be "$u$". On the other hand the verdict cannot be
"$s$". For if it were, then in the $K$-th section of its motion
$\mathcal{H}$ would be bound to compute the first
$R(K-1)+1 = R(K)$ figures of the sequence computed by the machine
with $K$ as its D.N and to write down the $R(K)$-th as a figure of
the sequence computed by $\mathcal{H}$. The computation of the
first $R(K)-1$ figures would be carried out all right, but the
instructions for calculating the $R(K)$-th would amount to
"calculate the first $R(K)$ figures computed by $\mathcal{H}$ and
write down the $R(K)$-th". This $R(K)$-th figure would never be
found. I.e., $\mathcal{H}$ is circular, contrary both to what we
have found in the last paragraph and to the verdict "$s$". Thus
both verdicts are impossible and we conclude that there can be no
machine $\mathcal{D}$.

We can show further that there can be no machine $\mathcal{E}$
which, when supplied with the S.D of an arbitrary machine
$\mathcal{M}$, will determine whether $\mathcal{M}$ ever prints a
given symbol ($0$ say). The argument combines, for each
$\mathcal{M}$, a family of machines
$\mathcal{M}_1, \mathcal{M}_2, \ldots$ (where $\mathcal{M}_k$
prints as $\mathcal{M}$ does but with its first $k$ printed $0$'s
replaced by a fresh symbol) to reduce "does $\mathcal{M}$ print
$0$ infinitely often?" — and hence "is $\mathcal{M}$ circle-free?"
— to the supposed machine $\mathcal{E}$; since no machine can
decide circle-freedom, there can be no machine $\mathcal{E}$.

The expression "there is a general process for determining …" has
been used throughout this section as equivalent to "there is a
machine which will determine …". This usage can be justified if
and only if we can justify our definition of "computable". For
each of these "general process" problems can be expressed as a
problem concerning a general process for determining whether a
given integer $n$ has a property $G(n)$ [e.g. $G(n)$ might mean
"$n$ is satisfactory" or "$n$ is the Gödel representation of a
provable formula"], and this is equivalent to computing a number
whose $n$-th figure is $1$ if $G(n)$ is true and $0$ if it is
false.

## 9. The extent of the computable numbers

No attempt has yet been made to show that the "computable" numbers
include all numbers which would naturally be regarded as
computable. All arguments which can be given are bound to be,
fundamentally, appeals to intuition, and for this reason rather
unsatisfactory mathematically. The real question at issue is "What
are the possible processes which can be carried out in computing a
number?" The arguments which I shall use are of three kinds:
**(a)** a direct appeal to intuition; **(b)** a proof of the
equivalence of two definitions (in case the new definition has a
greater intuitive appeal); **(c)** giving examples of large
classes of numbers which are computable.

Once it is granted that computable numbers are all "computable",
several other propositions of the same character follow. In
particular, it follows that if there is a general process for
determining whether a formula of the Hilbert function calculus is
provable, then the determination can be carried out by a machine.

**I. [Type (a)].** This argument is only an elaboration of the
ideas of §1. Computing is normally done by writing certain symbols
on paper. We may suppose this paper is divided into squares like a
child's arithmetic book. The two-dimensional character of paper is
no essential of computation, so I assume the computation is
carried out on one-dimensional paper, i.e. on a tape divided into
squares. I shall also suppose that the number of symbols which may
be printed is finite. If we were to allow an infinity of symbols,
then there would be symbols differing to an arbitrarily small
extent. The effect of this restriction is not very serious, since
it is always possible to use sequences of symbols in place of
single symbols — thus an Arabic numeral such as $17$ or
$999999999999999$ is normally treated as a single symbol. The
difference is that a compound symbol, if too lengthy, cannot be
observed at one glance; we cannot tell at a glance whether
$9999999999999999$ and $999999999999999$ are the same.

The behaviour of the computer at any moment is determined by the
symbols which he is observing and his "state of mind" at that
moment. We may suppose that there is a bound $B$ to the number of
symbols or squares which the computer can observe at one moment;
if he wishes to observe more, he must use successive observations.
We also suppose that the number of states of mind which need be
taken into account is finite, for reasons of the same character as
those which restrict the number of symbols. The simple operations
must therefore include: changes of the symbol on one of the
observed squares; and changes of one of the observed squares to
another square within $L$ squares of a previously observed one.
The possible behaviour is, at each stage, determined by the state
of mind and the observed symbols — which is just what a computing
machine does. Turing concludes that the operations of a (human)
computer can be carried out by a machine, so that every number
computable in the intuitive sense is computable by a machine.

**II. [Type (b)]** sketches the equivalence of computability with
definability in (a restricted form of) the Hilbert functional
calculus, and the appendix relates computability to Church's
$\lambda$-definability and effective calculability.

## 10. Examples of large classes of numbers which are computable

It is useful to begin with definitions of a computable function of
an integral variable and of a computable variable. There are many
equivalent ways of defining a computable function of an integral
variable; the simplest is, possibly, as follows. If $\mathfrak{O}$
is a computable sequence in which $0$ appears infinitely often,
and $n$ is an integer, let $W(\mathfrak{O},n)$ be the number of
figures $1$ between the $n$-th and the $(n+1)$-th figure $0$ in
$\mathfrak{O}$. Then $\psi(n)$ is computable if, for all $n$ and
some $\mathfrak{O}$, $\psi(n) = W(\mathfrak{O},n)$. An equivalent
definition phrases this in terms of a contradiction-free axiom
system from which the value of the function is provable for each
argument.

We cannot define general computable functions of a real variable,
since there is no general method of describing a real number, but
we can define a computable function of a computable variable. If
$n$ is satisfactory, let $\gamma_n$ be the number computed by
$\mathcal{M}(n)$, and let

$$\alpha_n = \tan\!\Big(\pi\big(\gamma_n - \tfrac{1}{2}\big)\Big),$$

unless $\gamma_n = 0$ or $\gamma_n = 1$, in either of which cases
$\alpha_n = 0$. Then, as $n$ runs through the satisfactory
numbers, $\alpha_n$ runs through the computable numbers. With
these definitions Turing states (proving some, omitting others) a
series of closure theorems, including:

- (i) A computable function of a computable function (of an
  integral or computable variable) is computable.
- (ii) Any function of an integral variable defined recursively in
  terms of computable functions is computable; and corresponding
  theorems for computable-valued functions.
- (iii)–(v) The sum, and in general any suitably uniformly
  convergent limit, of a computable sequence of computable numbers
  is computable; consequently $e$, $\pi$, the real parts of all
  algebraic numbers, the real parts of the zeros of the Bessel
  functions, and similar large classes of numbers are computable.

## 11. Application to the Entscheidungsproblem

The results of §8 can be used to show that the Hilbert
Entscheidungsproblem can have no solution. (For the formulation of
this problem see Hilbert and Ackermann's _Grundzüge der
Theoretischen Logik_, Berlin, 1931, ch. 3.)

I propose to show that there can be no general process for
determining whether a given formula $\mathfrak{U}$ of the
functional calculus $K$ is provable — i.e. that there can be no
machine which, supplied with any one $\mathfrak{U}$ of these
formulae, will eventually say whether $\mathfrak{U}$ is provable.

It should be remarked that what I shall prove is quite different
from the well-known results of Gödel. Gödel has shown that (in the
formalism of _Principia Mathematica_) there are propositions
$\mathfrak{U}$ such that neither $\mathfrak{U}$ nor
$-\mathfrak{U}$ is provable; as a consequence, no proof of
consistency of _Principia Mathematica_ (or of $K$) can be given
within that formalism. On the other hand, I shall show that there
is no general method which tells whether a given formula
$\mathfrak{U}$ is provable in $K$, or, what comes to the same,
whether the system consisting of $K$ with $-\mathfrak{U}$ adjoined
as an extra axiom is consistent.

If the negation of what Gödel has shown had been proved, i.e. if,
for each $\mathfrak{U}$, either $\mathfrak{U}$ or $-\mathfrak{U}$
is provable, then we should have an immediate solution of the
Entscheidungsproblem: for we could invent a machine $\mathcal{K}$
which proves consecutively all provable formulae, and sooner or
later $\mathcal{K}$ would reach either $\mathfrak{U}$ or
$-\mathfrak{U}$.

Corresponding to each computing machine $\mathcal{M}$ we construct
a formula $\mathrm{Un}(\mathcal{M})$, and we show that, if there
is a general method for determining whether
$\mathrm{Un}(\mathcal{M})$ is provable, then there is a general
method for determining whether $\mathcal{M}$ ever prints $0$. The
interpretations of the propositional functions involved are:

- $R_{S_0}(x,y)$: "in the complete configuration $x$ (of
  $\mathcal{M}$) the symbol on the square $y$ is $S_0$";
- $I(x,y)$: "in the complete configuration $x$ the square $y$ is
  scanned";
- $K_{q_m}(x)$: "in the complete configuration $x$ the
  $m$-configuration is $q_m$";
- $F(x,y)$: "$y$ is the immediate successor of $x$".

A single instruction $q_i S_j S_k L q_o$ of $\mathcal{M}$ is
rendered by an abbreviation $\mathrm{Inst}\{q_i S_j S_k L q_o\}$
standing for the formula

$$
(x,y,x',y')\;\Big\{ \big(R_{S_j}(x,y)\ \&\ I(x,y)\ \&\ K_{q_i}(x)\ \&\ F(x,x')\ \&\ F(y',y)\big) \rightarrow
$$

$$
\big( I(x',y')\ \&\ R_{S_k}(x',y)\ \&\ K_{q_o}(x')\ \&\ (z)\,[\,F(y',z)\ \lor\ (R_{S_j}(x',z) \leftrightarrow R_{S_k}(x',z))\,] \big) \Big\},
$$

with similar abbreviations for instructions ending in $R\,q_o$ or
$N\,q_o$. Assembling these instructions (together with axioms
describing the behaviour of $R$, $I$, $K$, $F$ and the initial
configuration) into a single formula $\mathrm{Un}(\mathcal{M})$,
Turing shows that $\mathrm{Un}(\mathcal{M})$ is provable in $K$ if
and only if $\mathcal{M}$ at some time prints $0$. Since (by §8)
there is no general process to decide whether $\mathcal{M}$ ever
prints $0$, there can be no general process to decide provability
in $K$. **Hence the Entscheidungsproblem is unsolvable.**

---

## Notes

1. Gödel, "Über formal unentscheidbare Sätze der Principia
   Mathematica und verwandter Systeme, I", _Monatshefte Math.
   Phys._, 38 (1931), 173–198.
2. Alonzo Church, "An unsolvable problem of elementary number
   theory", _American J. of Math._, 58 (1936), 345–363.
3. Alonzo Church, "A note on the Entscheidungsproblem", _J. of
   Symbolic Logic_, 1 (1936), 40–41.
4. Cf. Hobson, _Theory of functions of a real variable_ (2nd ed.,
   1921), 87, 88.
5. If we regard a symbol as literally printed on a square, the
   symbol may be defined as a (measurable) set of points in the
   unit square; one can then define the "distance" between two
   symbols, and with this topology the symbols form a
   conditionally compact space.
6. "The functional calculus" is used throughout to mean the
   restricted Hilbert functional calculus.
7. The required machine is most naturally constructed first as a
   choice machine (§2); each proof is determined by a sequence of
   binary choices $i_1, i_2, \ldots, i_n$, and the corresponding
   integer completely determines the proof.

_(Editorial note: the original uses High German blackletter for
$m$-configurations and $m$-functions, rendered here with Fraktur —
$\mathfrak{a}, \mathfrak{b}, \ldots$. The capitals $C$, $E$, $S$
are notoriously hard to distinguish in the original typeface.)_
