---
title: "The Tiniest C Sort Function"
url: https://www.cs.dartmouth.edu/~doug/tinysort.html
published: "2008-01-01T00:00:00Z"
feed: mcilroy
guid: https://www.cs.dartmouth.edu/~doug/tinysort.html
---

# The Tiniest C Sort Function

### The tiniest C sort function?

I wondered how small a sort program one can write in C.
The best result to date, after investigating several
distinctly different algorithms, tweaking a lot, and
getting major help from others,
is a 61-byte function at step 23. Can you beat it?
Or tie it, with subexponential running time?
s(a,n)int*a;{n-->1?s(a,n),s(a+1,n),n=*a,*a=a[1],a[n>*a]=n:0;}

GCC warning: some of the programs below need option
-std=c99.

HTML warning.
To preserve verbatim C source, naked
> symbols have been
left in function definitions instead of the proper HTML
escape sequence. Internet Explorer,
Safari, and Firefox tolerate the lapse.

My original intuition was that something really short would be really cryptic; and much of the code below is. But the current winner, while unconventional in using a conditional expression
instead of if, is very clean, with only one
deep-C trick—the conditional subscript.
The code is "oblivious", doing comparisons in the same order on every
input of a given length; and
its running time is extraordinary!

##### Comparison flaw

A technical flaw (pointed out by Bill Mckeeman, who credits
Steve Johnson) afflicts many of the following functions, in
which pointer b (or c)
can take on the value a-1.
The C standard does not guarantee the result of comparisons
of pointers p related to an array a of length n
for p outside the range [a,a+n].
Code that suffers from this defect is marked CF.

One example of what could go wrong is that if
the array address a
happens to be 0, then
a−1 may compare high to a.
However, the situation of an array being allocated
at address 0 won't happen in implementations of C
that take address 0 to be the null pointer.
Trouble could also arise for pointers represented
as base plus (nonnegative) offset.

In some cases the flaw only affects empty arrays
and can be dispelled by the classic maneuver of
declaring it a feature, not a bug: change
the precondition to require n>0.

#### Selection sort

.
-
It all began with a vanilla selection sort, the
precondition for which is a!=NULL&&n>=0.
The only trickery here is routine: eliminating curly brackets
by using commas instead of semicolons.
The length is 93 bytes,
excluding newlines. (CF)
void sort(int*a,int n){int*b,*c,t;for(b=a+n;--b>a;)
for(c=b;--c>=a;)if(*c>*b)t=*b,*b=*c,*c=t;}

-
Ditch a separate declaration by using C++/C99
loop initialization;
and can variable t by reusing
n.
(87 bytes, CF)
void sort(int*a,int n){for(int*b=a+n,*c;--b>a;)
for(c=b;c-->a;)if(*c>*b)n=*b,*b=*c,*c=n;}

-
Save the calculation of b by
changing signature to sort(first,last+1).
(83 bytes, CF)
void sort(int*a,int*b){for(int*c,t;--b>a;)
for(c=b;c-->a;)if(*c>*b)t=*b,*b=*c,*c=t;}

-
Peter McIlroy moved the declarations of
c and
t.
(82 bytes, CF)
void sort(int*a,int*b){while(--b>a)
for(int*c=b,t;c-->a;)if(*c>*b)t=*b,*b=*c,*c=t;}

-
Now something wild: put both
loop tests in one for. (81 bytes, CF)

Some bootless work gets done. The inner loop test
comes first, and fails the first time through.
Also the
if fails the first time
it's executed in each iteration of the outer loop.
void sort(int*a,int*b){for(int*c=a,t;c-->a
||(c=--b)>a;)if(*c>*b)t=*b,*b=*c,*c=t;}

-
Peter takes out a common subexpression, *b,
by embedding an assignment in a comparison.
(80 bytes, CF)
void sort(int*a,int*b){for(int*c=a,t;c-->a
||(c=--b)>a;)if(*c>(t=*b))*b=*c,*c=t;}

#### Weed out inversions one by one

-
Do away with one loop. Sweep the whole array, looking
for an inversion. If one is found, reverse it and
bump the decreasing loop index
back to the top. (79 bytes, CF)

This turns O(N2) behavior into O(N3).
But speed isn't our objective, and O(N2) is shabby
itself.

void sort(int*a,int*b){for(int*c=b,t;--c>a;)
if(t=c[-1],t>*c)c[-1]=*c,*c=t,c=b;}

-
Shorten the name. (76 bytes, CF)
void s(int*a,int*b){for(int*c=b,t;--c>a;)if(t=c[-1],t>*c)c[-1]=*c,*c=t,c=b;}

-
Convert from first,last+1 to first,last to get rid of
a decrement operator.
Use decrement and subscript [1] instead of
two subscripts [-1].
(72 bytes, CF)
void s(int*a,int*b){for(int*c=b,t;c>a;)if(t=*c--,*c>t)c[1]=*c,*c=t,c=b;}

-
Use obsolescent (pre-void) syntax. (68 bytes)
s(a,b)int*a,*b;{for(int*c=b,t;c>a;)if(t=*c--,*c>t)c[1]=*c,*c=t,c=b;}

-
Live dangerously. Same as step 10, but
with modern parameter declarations.
(67 bytes, CF)

Actually I wrote 11 before 10, but erroneously assumed I had
to revert to ancient syntax in order to have an
int function that doesn't
return a value. In fact
this is legal; the standard
merely forbids use of the missing result.
s(int*a,int*b){for(int*c=b,t;c>a;)if(t=*c--,*c>t)c[1]=*c,*c=t,c=b;}

-
Jeremy Yallop replaced the if with a conditional
expression. (66 bytes, CF)

The if had no else, so the
second branch (:0) in the conditional
is useless code. Still the trick saves a byte.

The nesting of commas illustrates an asymmetry of the two
branches of a conditional: only the first
can contain commas.

s(int*a,int*b){for(int*c=b,t;c>a;)t=*c--,*c>t?c[1]=*c,*c=t,c=b:0;}

-
Then he shed a comma by moving all but the first comma-clause from the body of the
for into the "increment"
clause.
(65 bytes, CF)
s(int*a,int*b){for(int*c=b,t;c>a;*c>t?c[1]=*c,*c=t,c=b:0)t=*c--;}

#### Cheating

-
Grzegorz Lukasik observed that at least
one compiler (gcc) will infer
type specifiers in global object declarations.
(64 bytes, CF)

Error: allowed in the early days of C, this behavior
does not comply with current standards. If the code
compiles, though, it is likely to work.
*c,t;s(int*a,int*b){for(c=b;c>a;*c>t?c[1]=*c,*c=t,c=b:0)t=*c--;}

#### Recursion

-
In this approach
the first recursive call sorts all but the
first element of the array. Then if the first
element isn't smallest, it's swapped with the
smallest and the rest of the array is sorted again.
The signature is s(first,last+1). (68 bytes)

With two recursive calls in the body, the code
may look as if its running time is
O(2N), but more detailed analysis
reveals it's O(N3), the
same as for the previous program.

Flaw: one element is accessed even if the array is empty.

s(int*a,int*b){int t=*a;b>++a?s(a,b),t>*a?a[-1]=*a,*a=t,s(a,b):0:0;}

-
Several suggestions from a

blog discussion can be combined
to make a little progress.
First, change the signature to
s(first,length,dummy).
(68 bytes)

This code doesn't do all its own work,
as it depends on the caller to allocate temporary
storage for it by supplying a dummy argument.

Flaws: accesses one element beyond end of array and
goes wild on empty arrays.
s(a,n,t)int*a;{t=*a++;--n?s(a,n,0),t>*a?a[-1]=*a,*a=t,s(a,n,0):0:0;}

-
Then suppress the increment of a to
facilitate a remarkable conditional swap in place of
a conditional expression.
Change the guard, --n ⇒ n--, to tame one flaw.
(67 bytes)

Big effect of a "small" change:
eliminating the conditional expression causes the running time
to explode to
O(2N).

Flaw: accesses one element beyond end of array.
s(a,n,t)int*a;{t=*a;n--?s(a+1,n,0),*a=a[1],a[t>*a]=t,s(a+1,n,0):0;}

-
Get rid of the dummy-argument zeroes by putting useful code in their place.
(62 bytes)

The trick was pioneered by a blog commenter who
put the first recursive call in the dummy argument of
the second:

s(a,n,t)int*a;{n--&&s(a,n,a[t>(*a=1[s(a+1,n),a])]=t=*a);}

Alas, this valiant offering overreaches and stumbles into
severely undefined behaviors: calling with the wrong number of arguments,
order of side-effects in an expresssion.
As the code happened to compile for me,
it propagated a[n] throughout the array.

Flaw (below): accesses one element beyond end of nonempty array.
s(a,n,t)int*a;{n--?s(a+1,n,t=*a),s(a+1,n,a[t>(*a=a[1])]=t):0;}

-
Correct the flaw by strengthening the guard.

Error, also in previous step: two assignments to a[0] can occur within the last function
argument. As no sequence point intervenes, the behavior is undefined.
Though the program has been observed to work as intended, it is not
required to.
(64 bytes)
s(a,n,t)int*a;{n-->1?s(a+1,n,t=*a),s(a+1,n,a[t>(*a=a[1])]=t):0;}

#### Hybrid

-
Ben Carter saw past the ingrained habit of swapping adjacent elements,
and abolished subscripts by swapping data via existing pointers.
Make the last element nonminimal. Bring the minimal element to the
front by sorting all but the last, then sort all but the first.
The signature is s(first,last)
and the running time is O(2N).
(70 bytes, CF)
s(int*a,int*b){int t;if(b>a)t=*a,t>*b?*a=*b,*b=t:0,s(a++,b-1),s(a,b);}

-
Replace tail recursion with iteration and
split the conditional-swap sequence between the increment clause
and the body of the for statement. (64 bytes, CF)
s(int*a,int*b){for(int t;b>a;t>*b?*a=*b,*b=t:0,s(a++,b-1))t=*a;}

-
Displace the useless 0 with the recursive call. (62 bytes, CF)

Due to exponential running time, the cost of this change (which
causes the tests to be reexecuted after a swap)
is less than that of lengthening
the array by 1.
s(int*a,int*b){for(int t;b>a;t>*b?*a=*b,*b=t:s(a++,b-1))t=*a;}

-
Returning to step 20,
Ben put the sorts before the swap, which allows t
to be eliminated. The two recursive calls order
the array unless the minimal element is last, in which case the
minimal element ends up in second place and gets swapped
home with conditional-swap code from step 17.
The signature is
s(first,length).
(61 bytes)
s(a,n)int*a;{n-->1?s(a,n),s(a+1,n),n=*a,*a=a[1],a[n>*a]=n:0;}

#### More cheating

-
Silas Poulson exploited a gcc extension to drop the
only keyword, int. (56 bytes)

Error: as in step 10, this nonstandard program is likely to work
if a compiler accepts it. Current (2020) versions of gcc reject it by default.
s(*a,n){n-->1?s(a,n),s(a+1,n),n=*a,*a=a[1],a[n>*a]=n:0;}

-
Michael Dunphy noted that the undefined-behavior swap code from step 19
can be applied and Alex Cole roped n into the undefinedness as well.
(53 bytes)
s(*a,n){n-->1?s(a,n),s(a+1,n),a[n>(*a=a[1])]=n=*a:0;}

Originally written March 2008. "Recursion" section soon shortened, and corrected
with regard to asymptotic running time.

January 2010, steps 12,13,14,16,17 added.

October 2010, steps 18 and 19 added.

November 2010, steps 20-23 and special treatment of "Comparison flaw" added.

April 2017, step 24 added and some words tweaked.

June 2020, step 19 critique (belatedly) added; step 23 intro corrected;

May 2021, delete 2 stray symbols in commentary; clarify a subhead
step 25 added; typography made more consistent.
