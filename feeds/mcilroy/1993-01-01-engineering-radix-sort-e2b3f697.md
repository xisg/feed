---
title: "Engineering Radix Sort"
url: http://www.usenix.org/publications/compsystems/1993/win_mcilroy.pdf
published: "1993-01-01T00:00:00Z"
feed: mcilroy
guid: http://www.usenix.org/publications/compsystems/1993/win_mcilroy.pdf
---

# Engineering Radix Sort

Engineering Radix Sort

Peter M. Mcllroy and Keith Bostic

ABSTRACT Radix sorting methods have excellent asymptotic performance on string data, for which comparison is not a unit-time operation. Attractive for use in large byte-addressable memories, these methods have nevertheless long been eclipsed by more easily prograÍrmed algorithms. Three ways to sort strings by bytes left to right-a stable list sort, a stable two-array sort, and an in-place "American flag" sor¿-are illustrated with practical C programs. For heavy-duty sorting, all three perform comparably, usually running at least twice as fast as a good quicksort. We recommend American flag sort for general use.

@ Computing Systems, Vol. 6 . No. 1 . Winter 1993

University of California at Berkeley;

and M. Douglas Mcllroy

AT&T Bell Laboratories

1. Introduction

For sorting strings you can't beat radix ss¡f-s¡ so the theory says. The idea is simple. Deal the strings into piles by their first letters. One pile gets all the empty strings. The next gets all the strings that begin with A-; another gets B- strings, and so on. Split these piles recursively on second and further letters until the strings end. Vy'hen there are no more piles to split, pick up all the piles in order. The strings are sorted. In theory radix sort is perfectly efficient. It looks at just enough letters in each string to distinguish it from all the rest. There is no way to inspect fewer letters and still be sure that the strings are properly sorted. But this theory doesn't tell the whole story: it's hard to keep track of the piles. Our main concern is bookkeeping, which can make or break radix sorting as a practical method. The paper may be read as a thorough answer to excercises posed in Knuth chapters 5.2 and 5.2.5, where the general plan is laid out.[l] Knuth also describes the other classical sorting methods that we refer to: radix exchange, quicksort, insertion sort, Shell sort, and little-endian radix sort.

I.I. Radix Exchange

For a binary alphabet, radix sorting specializes to the simple method of radix exchange.t'l Split the strings into three piles: the empty strings, those that begin with 0, and those that begin with 1. For classical radix exchange assume further that the strings are all the same length. Then there is no pile for empty strings and splitting can be done as in quicksort, with a bit test instead of quicksort's comparison to decide which pile a string belongs in. Program 1.1 sorts the part of array A that runs from Allo]to Alhi - 1]. All the strings in this range have the same å-bit prefix, say

Peter M. Mcllroy, Keith Bostic and M. Douglas Mcllroy

RadixExchange(,A', 1o, hi, b) ifhi-lo<1thel return if b > length(Atlol) then return mid : = Split (,{, 1o, hi, b) RadixExchange (A', fo, mid, b+1) RadixExchange(4, mid, hi, b+1)

x-. The function split moves strings with prefix -r0- to the beginning of the array, from A [0] through Almid - 1], and strings with prefix ¡ 1- to the end, from A lmidl throtgh Alhi - Il. To sort an n-element array, call

RadixExchange (,{, 0, n, 0)

When strings can have different lengths, a full three-way split is needed, as in Program I.2.l3l The pile of finished strings, with value x, say , begins at Allo]; the ¡ 0- pile begins at Alí01; the x 1 -pile begins at AliIl.

RadixExchange(,{, lo, hi, b) ifhi-1o<1 then return (i0, i1) :: Split3 (A', 1o, hi, b) RadixExchange(4, i0, il, b+1) RadixExchange(A', i1, hi, b+1)

Three-way splitting is the famous problem of the Dutch national flag: separate three mixed colors into bands like the red, white and blue of the flag.l+] For us, the three colors are Ø (no bit), 0 and 1. A recipe for splitting is given in Figure 1.1 and Program 1.3. The index l0 points to the beginning of the 0- pile, i I points just beyond the end of the 0- pile, and i2 points to the beginning of the 1- pile. The notation A[i].å denotes the åth bit, counted from 0, in string A[i]. When split3 finishes, il points to the beginning of the 1- pile as desired. The test for Ø is figurative; it stands for a test for end of string.

Program 1.1.

Program 1.2.

Engineering Radix Sort 7

A il i2 hi Ø l0! 0 ,| I

A r0 lo

A lo i0 iL i2 hi

Ø 0 ,l I

to to it ûn hi

Ø o [:*¡l ? l?l I

Figure 1.1 How split3 works. The four parts of the array hold strings known to have ended (Ø), strings known to have 0 in the selected position, unknown strings, and strings known to have I there. Repeatedly look at the selected position of the first unknown string-the shaded box. Update according to the matching diagram.

Program 1.3. splitS(s', lo, hi, b¡ = (i0, i1, i2) := (1o, 1o, hi) while i2 > i1 do case A[il].b of Ø: (Ati01 , Atill , i0, i1) := (Atitl, tiol , io+1, i1+1) 0: i1::i1f1 1: (Ati1l, Ati2-11 , i2) :: (Ati2-11 , Ati11 , iz-L) return (i0, i1)

L2. Quiclcsort to the Fore

After enjoying a brief popularity, radix exchange was upstaged by quicksort.lsl Not only was radix exchange usually slower than quicksort, it was not good for programming in Fortran or Algol, which hid the bits that it depends on. Quicksort became known as simple and nearly unbeatable; radix sorting disappeared from textbooks.

8 Peter M. Mcllroy, Keith Bostic and M. Douglas Mcllroy

Nevertheless, radix exchange cannot be bettered in respect to amount of data looked at. Quicksort doesn't even come close. Quicksort on average performs O (log n) comparisons per string and in-

spects Ct(log n) bits per comparison. By this measure the expected running time for quicksort is 0(n log2 n), for radix exchange only O(n log n). Worse, quicksort can "go quadratic," and take time A@2 bgn) on unfortunate inputs. This is not just an abstract possibility. Production quicksort routines have gone quadratic on perfectly reasonable inputs.[6J The theoretical advantages of radix exchange are usually swamped by the cost of bit picking. Quicksort is nimbler in several ways.

. Quicksort can use machine instructions to compare whole words or bytes instead of bits. . Quicksort splits adaptively. Because it picks splitting values from the data, quicksort can be expected to get roughly 50-50 splits even on skewed data. Such splits are necessary to realize minimal expected times in either quicksort or radix exchange. . Quicksort can sort anything, not just strings. Change the comparison routine and it is ready to handle different data. Because radix sort intrinsically assumes string data on a finite alphabet, it requires one to make the data fit the routine, not vice versa. For example, to sort dates with quicksort, one might provide code to parse ordinary notation (e.9. February Il,1732) as each key is looked at, while with radix sort one would preconvert all dates to a canonical form (e.g. I732O2II).

In other ways, quicksort and radix exchange are quite alike. They both sort in place, using little extra space. Both need a recursion stack, which we expect to grow to size O (log n).In either method, if the strings are long or have different lengths, it is well to address strings through uniform descriptors and to sort by rearranging small descriptors instead of big strings. The wisdom that blesses quicksort dates from the era of small memories. With bigger machines, the difference between nlog n and n log2 n becomes more significant. And with bigger machines we can afford more space. Thus the wisdom deserves to be reexamined.

Engineering Radix Sort 9

2. List-Based Sort

For most modern machines, the 8-bit byte is a natural radix, which should overcome the bit-picking slowness of radix exchange. A byte radix makes for 256- or 257 -way splitting, depending on how the ends of strings are determined. This raises the problem of managing space for so many piles of unknown size at each level of recursion. An array of linked lists is an obvious data structure. Dealing to the piles is easy; just index into the array. Picking up the sorted piles and getting them hooked together into a single list is a bit tricky, but takes little code. Program 2.1 does the job. It is written in C rather than pseudocode,

because the troubles with radix sort are in implementation, not in conception. The input variables are a linked list of null-terminated strings. b the offset of the byte to split on; the strings agree in all earlier bytes. sequel a sorted linked list of strings that compare greater than the strings in list a.

Three in-line functions are coded as macros:

ended (a, b) tells whether byte position b is just beyond the null byte at the end of string a. append (s, a) appends list s to the last element of non-empty list ø. deal (a, p) removes the first string from list a and deals it to pile p.

Program 2.1 has four parts. First, if the list is empty, then the result of sorting it together with sequel is sequel. Next, at "pile finished," if the last byte seen in the strings in list d was null (0), they cannot be sorted further. Put them in front of the sequel and return the combined list. At "split," all strings have a bth byte. Clear all the piles and then deal the strings out according to byte b of each string. Finally, at "recur on each pile," sort the piles from last to first. At each stage append to the sorted current pile the sorted list accumulated from all following piles. Program 2.1 works-slowly. Empty piles are the root of the trouble. Except possibly at the first level or two of recursion, most piles will be empty. The cost of clearing and recursively "sorting" as many

10 Peter M. Mcllroy, Keith Bostic and M. Douglas Mcllroy

Program 2.1. Simple list-based sort.

typedef struct list { struct list *next; rmsigned char *data; ) list; list *rsort(list *a, int b, list *sequel)

{ #def ine ended(a, b) b>O && a->datalb-11:o #define append(s, a) tnp=¿t while(tmp->next) tmp:f,¡p->next; tmp->next:s #define deal (a, p) tmp : "-ttt.*t, a->next : p' p = a, a : tmp list xPile [256] , *tmp; int i; if 1a: o¡ return sequel; if(ended(a, b)) { 7x pile finished */ append(sequel, a); return a;

) for 1i : 256; --i >- 0; ) /* split */ nile¡i¡ : 6' while (a) deal (a, pile [a->data tb] I ) ; for 1i = 256|, --i >= o; ) /* recur on each pile *,2 sequel : rsort(piletil, b+l, sequel); return sequel; Ì

as 255 empty piles for each byte of data is overwhelming. Some easy improvements will speed things up. l2.l Always deal into the same array and clear only the occupied piles between deals, meanwhile stackirig the occupied piles out of the way. I2.2 Manage the stack directly. Since the number of occupied piles is unpredictable, and probably small except at the first level or two of recursion, much space can be saved. The piles

may be stacked in first-to-last order so they will pop off in last-to-first order just as in Program 2.1. I2.3 Don't try to split singleton piles. I2.4 Optimize judiciously: eliminate redundant computation; replace subscripts by pointers. 12.5 Avoid looking at empty piles.

Engíneering Radíx Sort 11

r'J Program 2.2. lmproved list-based sort. See Program 2.1 for some macros.

Iist* rsort (list* a)

(D

tDH F

{#define singleton(a) a-)next == 0 #define push(a, b) sp->sa : a, (sp++)->sb : b #define pop (a, b) a : (--sp) -)sâ, b : sp->sb #define stackempty0 (sp <: stack) struct { Iist *sa; int sb; } stacktslzEl, *sp: stack; static list *pi]e [256] ; list *tmp, xsequel : 0; int i, b; if (a) push (a, 0) ; while(!stackemptyO) { pop (a, b) ; if (singleton(a) I I ended(a, b) ) 1 7* pile f inished */ append(sequel, a); sequel : a; continue'

ô ä

^

(D

tËo

a) Þ Èz ;o

0a l9Øzô

)while (a) /* deal (a, pile [a->data [b] I ) ; f or (i : 0; i<256; i++) /'É if (pile til ) { push(pi1elil , b+1); pilelil - 0; I Ìreturn sequel;

split */

stack the pieces */

Program 2.2 implements improvements l2.I-I2.3. Here the statecarrying parameters b and sequel become hidden, as they should be. On typical aþabetic data Program2.2 runs about 15 times as fast

as Program 2.1-not a bad return from such simple optimizations, but not yet good enough. Most of the time for Program 2.2 is still wasted scanning empty piles. Thus we turn to improvementI2.5, avoiding looking at empty piles, which can be done in many ways. For textual keys, such as names or decimal numbers, the piles are likely to be bunched. Single-case letters span only 26 of the 256 piles, digits only 10. To exploit bunching, we use a simple pile-span heuristic: keep track of the range of occupied piles. The finished 0- pile is an expected outlier and is kept track of separately. Thken together, improvements 12.l-I2.5 speed up Program 2.1 by a factor of 100 on typical inputs. The result, Program A in the appendix, is a creditable routine. It usually sorts arrays of 10,000 to 100,000 keys twice as fast as do competitive quicksorts. None of our programs so far sorts stably. Because piles are built by pushing records on the front of lists, the order of equal-keyed records is reversed at each deal. To stabilize the sort we can reverse each backwards pile as we append to it. Alternatively we can maintain the piles in forward order by keeping track of head and tail of each pile. Program A does the latter. Sorting times differ negligibly among forward/reverse and stable/unstable versions.

3. Two-Array Sort

Suppose the strings come in an arÍay as for radix exchange. In basic radix exchange, the two piles live in known positions against the bottom and top of the array. For larger radixes, the positions of the piles can be calculated in an extra pass that tallies how many strings belong in each pile. Knowing the sizes of the piles, we don't need linked lists. Program 3.1 gets the strings home by moving them as a block to the auxiliary afiay ta, and then moving each element back to its proper place. The upper ends of the places are precomputed in array pile as shown in Figure 4. 1. (This "backward" choice is for harmony with the programs in section 4.) Elements are moved stably; equal elements retain the order they had in the input. As in Program 2.2, the

Engíneering Radix Sort 13

À Program 3.1. Simple two-array sort. typedef unsigned char *string; void rsort (string xa, int n)

¡úo tl z ; a) ão

{ #def ine push (a, n, b) sp->sa : a, sp->sn : n, (sp#) ->sb : b #define pop (a, n, b) a : (--sp) -)Sâ, n : sp->sn, b : sp->sb #define stackernpty0 (sp <: stack) #define splittable (c) c > 0 && count [c] > 1 struct { string *s¿' int sn, sb; } stacklsrzEl, *sp : stack; string xPile [256] , *ak, 't'ta; static int cou¡:t [256] ; int i, b; ta : ma1loc (n*sizeof (string) ) ;

X (Þ

ld Ø

a) az ;oÉ 0aIt 2ô ä

push 1a, n, 0) ; while (!staekemptyO) { pop (4, n, b) ; for(i = n; --i >= 0; ) count ta til [bl ]+l; for (ak : a, i : 0; í < 256; i++¡ { if (splittable (i) )

Ìfor(i

for (i

)free (ta) ;

/* tal-l-y */

/* find places

push (ak, count Ii], b+f) ; Pilelil : ak += countlil; count [i] - 0;

*/

move to temp

-n; --i>:0; ) /* tatil : alil i :n; --i>-0; ) /* r,--pile lta til tbl I : ta [i] ;

move home */

stack is managed explicitly; the stack has a third field to hold the length of each subarray. Program 3.1 is amenable to most of the improvements listed in section 2; they appear in Program B. In addition, the piles are independent and need not be handled in order. Nor is it necessary to record the places of empty piles. These observations are embodied in the "find places" step of Program B. As we observed in the introduction, radix sorting is most advantageous for large arrays. When the piles get small, we may profitably divert to a simple low-overhead comparison-based sorting method

such as insertion sort or Shell sort. Diversion thresholds between 10 and 50 work well; the exact value is not critical. Program B in the appendix is such a hybrid two-array sort. It is competitive with list-based sort; which of the two methods wins depends on what computer, compiler, and test data one measures. For library purposes, an array interface is more natural than a list interface. But two-array sort dilutes that advantage by using O(n) working space and dynamic storage allocation. Our next variant overcomes this drawback.

4. American Flag Sort

Instead of copying data to an auxiliary array and back, we can permute the data in place. The central problem, a nice exercise in practical algorithmics, is to rearrange into ascending order an array of n integer values in the range 0 to m - 1. Here m is a value of moderate size, fixed in our case at 256, andn is arbitrary. Special cases are the partition step of quicksort (m : 2) and the Dutch national flag problem (m : -3). By analogy with the latter, we call the general problem

the American flag problem. (The many stripes are understood to be labeled distinctly, as if with the names of the several states in the original American union.) American flag sort differs from the two-array sort mainly in its final phase. The effect of the "move to temp" and "move home" phases of Program 3.1 is attained by the "permute home" phase shown in Program 4.1 and Figures 4.1-4.4. This phase fills piles from the top, making room by cyclically displacing elements from pile to pile.x

*A similar algorithm in Knuth chapter 5.2, exercise 13, does without the array count, bnt involves more case analysis and visits O(n) elements more than once. The speed and simplicity of Program 4. I justify the cost of the extra array.

Engineering Radix Sort 15

Program 4.1. In-place permutation to substitute in Program 3.1.

#def ine swap (p, 9, string t, int k,

for(k:0; k<n; ) { r : alkl; for(;;) {c : rlbl ; if (--piIe Ic] <: a*k) break; swap (xpi1e Ic] , T, t) Ìalkl : r; k +: count [c] ; count[c] : 0;

Let alkf be the first element of the first pile not yet known to be completely in place. Displace this element out of line to r (Figure 4.2). Let c be the number of the pile the displaced element belongs to. Find in the c- pile the next unfilled location, just below pílelc] (Figure 4.3). This location is the home of the displaced element. Swap the displaced element home after updating pilelcl to account for it. Repeat the operation of Figure 4.3 on the newly displaced element, following a cycle of the permutation until finally the home of the displaced element is where the cycle started, at alk]. Move the displaced element to ølk].Its pile, the current c- pile, is now filled (Figure a.a). Skip to the beginning of the next pile by incrementing ft. (Values in the count array must be retained from the "find places" phase.) Clear the count of the just-completed pile, and begin another permutation cycle. It is easy to check that the code works right when a * k : pilelc] initially, that is, when the pile is already in place. When all piles but one are in place, the last pile must necessarily be in place, too. Progrum 4.2, otherwise a condensed Program 4.1, exploits this fact. Program 4.2 and Program B form the the basis of Program C in the appendix.

16 Peter M. Mcllroy, Keith Bostic and M. Douglas Mcllroy

T:P' P:9' Q:r r) t; c;

Figure 4.2

Figure 4.3

/* Figiure 4.4 */

Figure 4.1. Array ¿ before permuting home.

Figure 4.2. After first displacement. Arrow shows completed action.

Figure 4.3. During displacement cycle. The bth byte of the string pointed to by r is c. Arrows show actions to do, except no swap happens in last iteration.

Program 4.2. lmproved in-place permutation.

cmax : /* index of last occupied pile *7' n -: countlcmax]; count lcnax] - O; for (ak : a; ak < a*n; ak +: count [c] , count [c] : 0) { r : *ak; while(--pile[s : r[b]l > ak) swap (xpile [c] , T, t) ; *ak : T; Ì

pile[255]=a+n

r

r

a+k + count

Figure 4.4. Last move.

T7

Engineering Radix Sort

4.1. Stack Growth

In the programs so far, the stack potentially grows linearly with running time. We can bound the growth logarithmically by arranging to split the largest pile at each level last-a trick well known from quicksort. This biggest-pile-last strategy is easy to install in array-based sorts, but inconvenient for list-based, where stack order matters and pile sizes are not automatically available. Even with a logarithmic bound, the stack can be sizable. In the worst case, a split produces no 0- piles, 253 little (size 2) piles, and two big piles of equal size for the rest of the data. One of the big piles is immediately popped from the stack and split similarþ. For n : 1,000,000, the worst-case stack has 2800 entries (33,000 bytes on a 32-bit machine). By contrast, a stack of about 254logrru n entries (only 630 when n : I,000,000) suffices for uniform data. Still smaller stacks work for realistic data. Instead of a worst-case stack, we may allocate a short stack, say enough for two levels of full 256way splitting, and call the routine recursively in the rare case of overflow. For completeness, both stacking tactics are shown in program C, though they will almost surely never be needed in practice. Stack control adds one-third to the executable code, but only about one percent to the running time.

4.2 . Tricks for Tallying

The pile-span heuristic for coping with empty piles presupposes a not unfavorably distributed aþabet. Other ways to avoid looking at empty piles may be found in the literature. One is to keep a list of occupied piles. Each time a string goes into an empty pile, record that pile in the list. After the deal, sort the list of occupied piles by pile number. If the list is too long, ignore it and scan all the piles.tTJ A version of Program C with this strategy instead of pile-span ran slightly faster on adverse data, but slower on reasonable data. Alternatively, an occupancy tree may be superimposed on the array of piles. Then the amount of work to locate occupied piles will diminish with diminishing occupancy. The best of several tree-tallying schemes that we have tried is quite insensitive to the distribution of

18 Peter M. Mcllroy, Keith Bostic and M. Douglas Mcllroy

strings and beats pile-span decisively on adverse data, but normally runs about one-third to one-half slower than pile-span. Noting that only the identity and not the order of the piles matters in splitting, Paige and Tarjan propose to scan piles in one combined pass after all splits are done.[8] Their method favors large radixes; it runs faster with radix 64K than with radix 256, Unfortunately, overhead-from 4n to 8n extra words of memory-swamps the theoretical

advantage. Little-endian (last-letter first) sorting mitigates the problem of scanning empty piles. In little-endian sorts the number of splits is equal to the number of letters in the longest key, whereas in bigendian sorts like ours the number of splits typically exceeds the number of keys. Aho, Hopcroft, and Ullman show how to eliminate pile scanning at each deal of a little-endian sort by using a O ("total size") presort of all letters from all keys to predict what piles will occur.le] A little-endian radix sort, however, must visit all letters of all keys instead ofjust the letters of distinguishing prefixes. In practice, exotic tricks for tallying are rendered moot by diverting to an alternate algorithm for small r. For it is only when n is small that the time to scan piles is noticeable in comparison to the time to deal and permute. Nevertheless, we still like the extremely cheap pile-span heuristic as a supplemental strategy, for it can improve running times as much as IOVo beyond diversion alone.

5. Perþrmance

The merits of the several programs must be judged on preponderant, not decisive, evidence. In theory, all have the same worst-case asymptotic running time, O(S), where .S is the size of the data measured in bytes. None is clearly dominant in practical terms, either. The relative behaviors vary with data, hardware, and compiler. In assessing performance, we shall consider only large data sets, where radix sorting is most attractive. Just how attractive is indicated by comparison with quicksort. The tested quicksort program, which compares strings in line, chooses a random splitting element, and diverts to a simple sort for small arrays, was specialized from a model by Bentley and Mcllroy. The routine is not best possible, but probably

Engineering Radix Sort I9

within l/3 of the ultimate speed for C code. (We recoiled from adapting their fastest model, which would require 23 in-line string comparisons.) Figure 5.1 shows the variation with size for 15 tests of each of four routines on one computer for two kinds of random key: (1) strings of 8 random decimal digits and (2) strings of random bytes, exponentially distributed in length with mean 9. The range of this experiment is too narrow to reveal quicksort's nlog2 n depafture from linearity, or to fully smooth quantizing effects. (Across this range the expected

Timeln psec/key

10000 20000 50000 100000 Number of keys, n

Figure 5.1. Leasrsquares fits to sorting time per key versus log n for a DEC VAX 8550. Representative t I cr eÍÍor bars are shown; other curves fit comparably.

Peter M. Mcllroy, Keith Bostic and M. Douglas Mcllroy

QS quicksort LB list-based (Program A) TA two-array (Program B) AF American-flag (Program C)

¡¿¡dsm bytes ---- randomdigits

TA-----

Machine Compiler

DEC Vax 8850

MIPS 6280

Sun Sparcstation

Cray XMP

Table 5.1. Machines and compilers tested.

length of comparisons varies by only one digit for the decimal data

and half a byte for the byte data.) Nevertheless the figure clearly

shows that the radix sorts run markedly faster than quicksort. This

observation is robust. No comparable generalization can be drawn about the relative performance of the three radix sorts. As the figure shows, the rank order depends on the kind of data. Other experiments show similar variation with hardware. The sensitivity of the pile-span heuristic to key distribution shows up in Figure 5.1 as a large variation of slope for list-based sort. In the other two routines, diversion almost completely damps the variation. The sensitivity is even greater on sparse random data. In the extreme case of random keys strings containing just two byte values, the listbased Program A took about 5 times as long to sort keys with distant byte values as with adjacent values. The hybrid Programs B and C varied by a factor of 1.2 or less. Quicksort was unaffected. Realistic sorting problems are usually far from random. We sorted the 73,000 words of the Merriam-Webster Collegiate Dictionary, Tth edition, using the machines and compilers listed in Thble 5. 1. The word list consists mainly of two interleaved alphabetical lists, of capitalized and uncapitalized words. We sorted, into strict ASCII order, three input configurations: (1) as is, (2) two copies of the list concatenated, and (3) ordered by reversed spelling, which mixes the data well. The running times of programs A, B, and C were usually within

a factor of 1.2 of each other, with no clear winner. American-flag sort

lcc gcc gcc -O

Fraser and Hansonro Gnu same, with optimization

MIPS same, with optimization Fraser and Hanson

cc cc -O4 lcc

lcc gcc -O

Fraser and Hanson Gnu

Cray

scc

Engineering Radix Sort 2l

won consistently on the MIPS, two-array sort on the Cray, and listbased on the Vax, a result roughly consonant with the degree of pipelining on the several machines. The list-based program was the most erratic. It lost consistently on the MIPS, and decisively-by a factor of 1.6-on some Cray and MIPS runs. As in Figure 5.1, quicksort fell far behind the radix sorts, usually by a factor of two or more.

6. Discussion

Our programs synopsize experiments that we have made jointly and severally over the past few years. Bostic wrote a two-array radix sort similar to Program B for the Berkeley BSD library, based in part on a routine by Dan Bernstein. P. Mcllroy adapted that routine for use in the BSD version of the Posix standardll sort utility.12 P. Mcllroy also conceived American flag sort as a replacement for the two-array library routine. Independently, D. Mcllroy wrote a Posix utility around a list-based radix sort, and installed it on research systems at AT&T. Both the Berkeley and the AT&T utilities typically run twice as fast overall as the venerable quicksort-based programsl3 that they replace. Although radix sorts have unbeatable asymptotic performance, they present problems for practical implementation: (1) managing scattered piles of unpredictable size and (2) handling complex keys. W'e have shown that the piles can be handled comfortably. Our utilities cope with complex keys by preconverting them into strings. Although it costs memory roughly proportional to the volume of keys, this strategy is simple and effective for sorting records after the fashion of the proposed Posix standard. List-based radix sort is faster than pure array-based radix sorts. The speed disparity is overcome by hybrid routines that divert from radix sorting to simple comparison-based sorting for small arrays. The natural array-argument interface makes them attractive for library purposes. Both list-based and two-array sorts entail O(n) space overhead. That overhead shrinks to O(log n) in American flag sort, which, like quicksort, trades off stability for space efficiency. We recommend American flag sort as an all-round algorithm for sorting strings. We have profited, even to the wording of our title, from the advice and exemplary style of Jon Bentley.

Peter M. Mcllroy, Keith Bostic and M. Douglas Mcllroy 22

Appendix

Program A. Søble list-based sort. typedef strucÈ list { struct list *next,' unsigned char *data,' l list,' list *rsort (list *a)

t #define push(a, t, b) sp->sa - ar sP-)st: tr (sP++)-)sb = b #define pop(a, t, b) a = (--sp)-)sa, t = 5p-)st' b: sp->sb #define stackernptyO (sp <= stack) #define singleton (a) (a->next :: Q) #define ended(a, b) b>0 && a-)datatb-Ll=:O struct { list *sa, *st,'int sb; } stackISIZE], *sp = stacki static list *pile [256] . *tail 12561; list *atail, *sequel = 0,' int b, c, cmin, nc : 0,' if (a ee ! singleton (a) ) Push (a, 0. 0) ,' while(!stackenPtYO) { pop(a, atai1, b); if(singleton(a) | | ended(a, b)) { /* pile finished */ atail-)next : sequel,' sequel = a,' continuei

) cmin : 255,' /* sPlit */ for(;a;a-a->next) { c - a->datalbl; if(Pileicl =: 0) { tsai1[c] =PileIc]:a; if (c :: 0) continue,. if(c < cmin) cmin = c; nc++,' ) else taillcl - tail[c]-)next : a; Ì if(pifet0l) { / * stack the pieces */ push (pile [0] , tail- [0] , b+1) ,' tail [0]->next = PiIe[0] = 0,'

)for(c = cmin; nc > 0t c++) if (pile Ic] ) i Push (Pile Ic ] , tail Ic] ' b+L) ; tail[c]->next : Pilelcl = 0; nc--;

) i relurn sequeli

Engineering Radix Sort 23

Program B. Hybrid two-anay sort. lypedef unsigned char *strinq,' void sinrplesort (string*, int, inÈ) i void rsort (strj.ng *a, int n)

{ #define push(a, n, i) sp->sa = a, sp->sr = i, (sp++)->si: i *define pop(a, n, i) a: (--sp)->sa, n = sp->6n, i = sp->si #define stackempty$ (sp <= stack) strucÈ { string *sa; int sn, si; } stackISIZE], *sp = stack; string *pileÍ2561, *aí, *ak, *ta; static int count 12561; int b, c, cmin, *cp, nc = 0; ta = malloc (n*sizeof (string) ) ,' push (a' n, 0) ; while(!s¿ackemptyO) { pop(a, n' b); if(n < THRESHOLD) I /* divert */ simplesort (a, n, b) ,' continue;

) cmin = 255,. /* LaLLy */ for (ak - a*ni --ak >= a; ) { c : (*ak) [b]; if(+*counllcl == 1 && c > 0) { if (c < cmin) cmin = c,: nc++,'

) pilelo] - ak = a + countlo]; ,/* find places */ counttol : 0; for(cp = count+cmin,' nc > 0,' cp++' nc--) { while (*cP == 0) cP++" if 1*"n t t, push(ak, *cp' b+1); Pile[cP-count] : ak += *cP; *cP : o"

) for(ak = ta*n, ai = a+n; ak ) ta,' ) /* move to temp */ *--ak = *--ai; for(ak = ta*n,' ak-- ) ta,' ) /* move home */ *--pile [ (*ak) [b]l = *"¡,'

) f¡ee (t.a) ,'

24 Peþr M. Mcllroy, Keith Bostic and M. Douglas Mcllroy

)

Program C. Hybrid American flag sort; optional stack control in fine print.

enum { SIZE = 5L0, THRESHOLD = 16 i; typedef unsigned char *string; typedef struct { string *sa,' int sn, si; } stack_t; void simplesort (string*' int, int),' static void rsorta(strj-ng *a, int n ,int b)

{ *define push(a, n, i) sp-)sa: a, sp->sn = n, (sp++)->si = i *define pop(a, n, i) a: (--sp)->sa, n = sp->sn' i = sp->si *define stackemptyQ (sp <= stack) #define swap (p, q' r) x = p, P : Q' g = r stack_t stack[SIZE], *sp = stack, stmp, *oldsp, *biqsp; stri-ng *pile 12561 , *ak, *an, r, t,' static int count 1256) , cmin, nc,' int *cp, c, cmax,/*' b = O*/;

push (a, n, b) ; while(!stackemPtYO) { pop(a, n, b); if(n < THRESHOLD) { /* diver! */ simplesort (a, n ' b) ,' continue,'

Ì an=a+n,' if (nc == 0) ( ,/* untallied? */ cmin = 255,. /* LaLLy x/ for(ak = a; ak < an,' ) { c : (*ak++) [b]; if (++count [c] =: I && c > 0) { if (c < cmin) cmin = c,' nc++; Ì Ì if (sp+nc > stack+sizE) I / * stâck overflow i,/ rsorta (4, n, b) ; cont inue;

I oldsp = bigsp = sp, c = 2, ,/* logarithmic stack */ piletol = ak = a+count[cmax-0¡; /* fínd places */ for(cp : counÈ*cmin,' nc ) 0,' cp**, nc--) { while (*cP == 0) cP++; if(*cp > 1) { if(*cp > c) c = *cP, bj.gsP = sP; push(ak' *cP, b+1);

Ì

Ì

) Pile[cmax : cp-count] : ak += *cP;

Engíneering Radix Sort 25

swap(roldsp, *bigsp, stmpli an -- countlcmax] ì /* permute home */ count[cmax] - 0,' for(ak : a,' ak ( an; ak += count[c], count[c] = 0) { r : *ak,' while(--pile[c = rtb] I > ak) swap(*pi1eIc] , r, t),' *ak : r,' I /* here nc = countL...l : a */

Ì l void rsort (string ia, 1nt n) { rsorta (å, n, 0); }

7. Addendum

While this paper was in press, another radix sort appeared, recursive like Program 3.1, with diversion and in-place permutation. [. J. Davis, A fast radix sort, Computer J. 35 (1992) 636-642.1Although wasteful of storage, that program can be easily modified to run as fast

as Program C, which stands as a good benchmark for radix sorting. IVe are grateful to Peter McCauley for critical reading of our progfams.

26 Peter M. Mcllroy, Keith Bostic and M. Douglas Mcllroy

References

[1] Knuth, D 8., The Art of Computer Programming, 3, Sorting and Searching, Addison Wesley (1973).

[2] Hildebrandt, P. and Isbitz, H., "Radix exchange-an internal sorting method for digital computers," JACM 6, pp. 156-163 (1959).

[3] Bentley, J. L., Programming Pearls, Addison-Wesley (1986). Solution 10.6.

[4] Dijkstra, E. W., A Discipline of Programming, Prentice-Hall (1976).

[5] Hoare, C. A. R., "Quicksort," Computer Journal5, pp. 10-15 (1962).

[6] Bentley, J. L., "The trouble with qsort," UnixReview L0(2), pp. 85-93 (Feb. 1992).

[7] McCauley, P. 8., Sorting method and apparatøs, U.S. Patent 4,809,158 (1989).

[8] Paige, R. and Tärjan, R. E., "Three partition refinement algorithms," SIAM J. Comput. 16, pp. 973-989 (1987).

[9] Aho, A. V., Hopcroft, J. E., and Ullman, J. D., The Design and AnaIysis of Computer Algorithrøs, Addison-Wesley (1974).

[10] Fraser, C. W. and Hanson, D. R., "A retargetable compiler for ANSI C," ACM SIGPLAN Notices 26(10), pp. 29-43 (October l99I).

[1 1] IEEE, Draft Standard for Information Technology-Operating System Interface (POSX) Part 2: Shell and Utilities, Vol. P1003.2 /Dll (1991). [2] Mcllroy, P. M., "Intermediate files and external radix sort," submitted for publication.

[13] Linderman, J. P., "Theory and practice in the construction of a working sort routine," AT&T Bell Laboratories Technical Journal 63, pp. 1827-1844 (1984).

[submitted Sept. 3, L992; accepted Oct. 3, 1992]

Permission to copy without fee all or part of this material is granted provided that the copies are not made or distributed for direct commercial advantage, the Computing Systems copyright notice and its date appear, and notice is given that copying is by permission of the Regents of the University of California. To copy otherwise, or to republish, requires a fee and/or specific permission. See inside front cover for details.

Engineeríng Radix Sort 21
