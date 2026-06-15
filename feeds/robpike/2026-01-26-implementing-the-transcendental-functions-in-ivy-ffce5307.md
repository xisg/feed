---
title: Implementing the transcendental functions in Ivy
url: https://commandcenter.blogspot.com/2026/01/implementing-transcendental-functions.html
published: "2026-01-26T07:07:00Z"
feed: robpike
guid: tag:blogger.com,1999:blog-6983287.post-5728563089568741872
---

# Implementing the transcendental functions in Ivy

Towards the end of 2014, in need of pleasant distraction, I began writing, in Go, my second pseudo-APL, called Ivy. Over the years, Ivy's capabilities expanded to the point that it's now being used to do things such as to design [floating-point printing algorithms](https://research.swtch.com/fp-knuth). That was unexpected, doubly so because among the set of array languages founded by APL, Ivy remains one of the weakest.

Also unexpected was how the arrival of high-precision floating point in Go's libraries presented me with some mathematical challenges. Computing functions such as sine and cosine when there are more bits than the standard algorithms provide required polishing up some mathematics that had rusted over in my brain. In this article, I'll show the results and the sometimes surprising paths that led to them.

First I need to talk a bit more about Ivy itself.

For obvious reasons, Ivy avoids APL's idiosyncratic character set and uses ASCII tokens and words, just as in my previous, Lisp-written pseudo-APL back in 1979. To give it a different flavor and create a weak but valid justification for it, from the start Ivy used exact arithmetic, colloquially big ints and exact rationals. This made it unusual and a little bit interesting, and also meant it could be used to perform calculations that were clumsy or infeasible in traditional languages, particularly cryptographic work. The factorial of 100, in Ivy (input is indented):

> !100      93326215443944152681699238856266700490715968264381621468592963895217599993229915608941463976156518286253697920827223758251185210916864000000000000000000000000

Or perhaps more surprising:

> 0.5
>
> 1/2

Ivy also became, due to the work of Hana Kim and David Crawshaw, the first mobile application written in Go. I take no credit for that. The iOS version is long gone and the Android version is very old, but that situation might change before long.

For those who don't know the history and peculiarities of APL, it's worth doing your own research; that education is not the purpose of this article. But to make the examples comprehensible to those sadly unfamiliar with it, there are a few details of its model one should understand to make sense of the Ivy examples below.

First, every operator, whether built-in or user-programmed, takes either one argument or two. If one, the operator precedes the argument; if two, the arguments are left and right of the operator. In APL, these are called monadic and dyadic operators. Because Ivy was written based on my fading memory of APL, rather than references, in Ivy they are called unary and binary. Some operators work in either mode according to the surrounding expression; consider

> 1 - 2  # Binary: subtraction
>
> -1
>
>     \- 0.5  # Unary: negation
>
> -1/2

The second detail is that values can be scalars or they can be multidimensional vectors or matrices. And the operators handle these in a way that feels deeply natural, which is part of why APL happened:

> 4 5 6 - 2
>
> 2 3 4
>
>     4 5 6 - 1 2 3
>
> 3 3 3

Compare those expressions to the equivalent in almost any other style of language to see why APL has its fans.

Finally, because the structure of APL expressions is so simple with only two syntactic forms of operators (unary and binary), operator precedence simply does not exist. Instead, expressions are evaluated from right to left, always (modulo parentheses). Thus

3 \* 4 + 5

evaluates to 27 because it groups as 3 \* (4+5): every operand is maximal length, traveling left, until an operator arrives. In most languages, the usual precedence rules would instead group this as (3\*4) + 5.

Enough preamble.

Ivy was fun to write, which was its real purpose, and there were some technical challenges, in particular how to handle the fluid typing, with things changing from integers to fractions, scalars to vectors, vectors to matrices, and so on. I gave a [talk](https://www.youtube.com/watch?v=PXoG0WX0r_E) about this problem very early in Ivy's development, if you are interested. A lot has changed since then but the fundamental ideas persist.

Before long, it bothered me that I couldn't calculate square roots because Ivy did not handle irrational numbers, only exact integers and rationals. But Robert Griesemer, who had written the math/big library that Ivy used, decided to tackle high-precision floating point. This was important not just for Ivy, but for Go itself. The rules for [constants in Go](https://go.dev/blog/constants) require that they can be represented to high precision in the source code, much more so than the usual 64-bit variables we are used to. That property allowed constant expressions involving floating-point numbers to be evaluated at compile time without losing precision.

At the time, the Go toolchain was being [converted](https://go.dev/talks/2014/c2go.slide#1) from the original C implementation to Go, and the constant evaluation code in the original compiler had some problems. Robert decided, with his usual wisdom and tenacity, to build a strong high-precision floating-point library that the new compiler could use. And Ivy would benefit.

By mid-2015, Ivy supported floats with lots of bits. Happiness ensued:

> sqrt 2
>
> 1.41421356237
>
> )format '%.50f'
>
> sqrt 2
>
> 1.41421356237309504880168872420969807856967187537695

Time to build in some helpful constants:

> pi
>
> 3.14159265358979323846264338327950288419716939937511
>
> e
>
> 2.71828182845904523536028747135266249775724709369996

Internally these constants are stored with 3000 digits and are converted on the fly to the appropriate precision, which is 256 bits by default but can be set much bigger:

> )prec 10000       # Set the mantissa size in bits
>
> )format '%.300f'  # Set the format for printing values
>
> pi
>
> 3.141592653589793238462643383279502884197169399375105820974944592307816406286208998628034825342117067982148086513282306647093844609550582231725359408128481117450284102701938521105559644622948954930381964428810975665933446128475648233786783165271201909145648566923460348610454326648213393607260249141274

Now things get harder.

Square root is easy to calculate to arbitrary precision. Newton's method converges very quickly and we can even use a trick to speed it up: create the initial value by halving the exponent of the floating-point argument.

But what about sine, cosine, exponential, logarithm? These transcendental functions are problematic computationally. Arctangent is a nightmare.

When I looked around for algorithms to compute these functions, it was easy to find clever ways to compute accurate values with 64-bit floating-point (float64) inputs, but that was it. I had to get creative.

What follows is how I took on programming high-precision transcendental functions and their relatives. It was a lot of fun and I uncovered some fascinating things along the way.

_Disclaimer_: Before going further, I must admit that I am not an expert in numerical computation. I am not an expert in analysis. I am not an expert in calculus. I have just enough knowledge to be dangerous. In the story that follows, if I say something that's clearly wrong, I apologize in advance. If you see a way I could have done better, please let me know. I'd love to lift the game. I know for a fact that my implementations are imperfect and can get ratty in the last few bits. And I cannot state anything definitive about their error characteristics. But they were fun to puzzle out and implement.

In short, here follows the ramblings of a confessed amateur traversing unfamiliar ground.

The easiest transcendental functions to provide are sine, cosine, and exponential. These three are in fact closely related, and can all be computed well using their Taylor series, which converge quickly even for large values. Here for instance is the Taylor series (actually a Maclaurin series, since we start at 0) for exponential (thanks to Wikipedia—which, by the way, I support financially and hope you do too—for the screen grab of these equations):

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiWHgu70DgZsV6u1Hu9KpffcilaCmUaVFDQexaYUO8_HpCO67u2Xdrh0gAyiTWAJKSeolKaDx_R26A4Q-RZLT2nIOuplSznZE-lhthY1uvEEuo7M-Ird-IXhXHYXG1KlTVIIBuapaE01A5ayg9mVdpvPQztwNR354dvYbnUofNaZ5QMIathie1FIA/w200-h82/exp.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiWHgu70DgZsV6u1Hu9KpffcilaCmUaVFDQexaYUO8_HpCO67u2Xdrh0gAyiTWAJKSeolKaDx_R26A4Q-RZLT2nIOuplSznZE-lhthY1uvEEuo7M-Ird-IXhXHYXG1KlTVIIBuapaE01A5ayg9mVdpvPQztwNR354dvYbnUofNaZ5QMIathie1FIA/s582/exp.png)

Those factorials in the denominator make this an effective way to compute the exponential. And the sine and cosine series are closely related, taking alternate (sine odd, cosine even) terms from the exponential, with alternating signs:

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhCyCkfqCzNQxiom3x3RzUhIFUZHlz0_3ZJ_cjJmsCOpGW_YQKiP_Q7tcZgl8Df72O4k4cPCpDPR3QjoE8nr18ueue3AxNz_7NbNrMhamguSiSaaWJTt9atmxoKfVvkbVtnQWzb4sNd1BzsXPj3G8-NXxXKPLZ5zHl8X6WR7hSaZVx_y-zu79nVHQ/w200-h79/sin.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhCyCkfqCzNQxiom3x3RzUhIFUZHlz0_3ZJ_cjJmsCOpGW_YQKiP_Q7tcZgl8Df72O4k4cPCpDPR3QjoE8nr18ueue3AxNz_7NbNrMhamguSiSaaWJTt9atmxoKfVvkbVtnQWzb4sNd1BzsXPj3G8-NXxXKPLZ5zHl8X6WR7hSaZVx_y-zu79nVHQ/s596/sin.png)

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhzk8XLRGr8XC1BuLKLnJF4m4bYSUm1sVtp1xTu4EqjkDMHSFXTU078KVgtUSu7twWm70f7Mr8nWu-RfGFmBnJJwe-anfdCwoh4iZl4TAgdnG6zLaUJk0qbNdJiBZ3TGWaVHoH1oZwXvSu8Idb0AemPt2nmd0qCghvqal6gidN-l4ZtpBSsaq77cw/w200-h79/cos.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhzk8XLRGr8XC1BuLKLnJF4m4bYSUm1sVtp1xTu4EqjkDMHSFXTU078KVgtUSu7twWm70f7Mr8nWu-RfGFmBnJJwe-anfdCwoh4iZl4TAgdnG6zLaUJk0qbNdJiBZ3TGWaVHoH1oZwXvSu8Idb0AemPt2nmd0qCghvqal6gidN-l4ZtpBSsaq77cw/s594/cos.png)

Euler's famous identity

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjBJ-6AubV2aG3ANSV17ZudJ-XZ8l7aMuHJS54ZX0dUXhFoWnCPoSjihEvyDMN-kaphPiZ5FP6MXsgAACgtnPxbc7ZLj9capnPbOfilBQVorTkJ4l-7k1GBp-n3YYVqWYym7yVK0x7eihVyfhCNcR6tCTrjuUi_SJw2OT3qT6i-8erPpbjwdih5Ig/s1600/euler.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjBJ-6AubV2aG3ANSV17ZudJ-XZ8l7aMuHJS54ZX0dUXhFoWnCPoSjihEvyDMN-kaphPiZ5FP6MXsgAACgtnPxbc7ZLj9capnPbOfilBQVorTkJ4l-7k1GBp-n3YYVqWYym7yVK0x7eihVyfhCNcR6tCTrjuUi_SJw2OT3qT6i-8erPpbjwdih5Ig/s78/euler.png)

might help explain why these three functions are related. (Or not, depending on your sensibilities.)

The challenge with sine and cosine is not the calculation itself, but the process of argument reduction. Because these functions are periodic, and because small values of the argument do better in a Taylor series, it is a good idea to "reduce" or unwind the argument into the range of 0 to 2𝛑. For very large values, this reduction is hard to do precisely, and I suspect my simplistic code isn't great.

For tangent, rather than fight the much clumsier Taylor series, I just computed sin( _x_) and cos( _x_) and returned their quotient, with care.

That process of taking existing implementations and using identities to build a new function was central to how Ivy eventually supported complex arguments to these functions as well as hyperbolic variants of them all. For example, for complex tangent, we can use this identity:

tan( _x_ + _yi_) = (sin(2 _x_) \+ _i_ sinh(2 _y_))/(cos(2 _x_) \+ cosh(2 _y_))

Here sinh is the hyperbolic sine, which is easy to compute given the exponential function or by just taking the odd terms of the Taylor series for exponential, while cosh takes the even terms. See the pattern?

Next up we have the inverse functions arcsine (asin) and arccosine (acos). They aren't too bad but their Taylor series have poor convergence when the argument is near ±1. So we grub around for helpful identities and come up with two:

asin( _x_) = atan(x/sqrt(1 **-** _x_ ²))

acos( _x_) = π/2 - asin( _x_)

But that means we need arctangent (atan), and its Taylor series has awful convergence for _x_ near 1, whose result should be the entirely reasonable π/4 or 45 degrees. An experiment showed that using the Taylor series directly at 1.00001 took over a million iterations to converge to a good value. We need better.

I did some more exploration and in one of my old schoolbooks I found this identity:

atan( _x_) = atan( _y_) \+ atan(( _x_ **-** _y_)/(1+ _xy_))

In this identity, _y_ is a free variable! That allows us to do better. We can choose _y_ to have helpful properties. Now:

tan(π/8) = √2-1

or equivalently,

atan(√2-1) = π/8

so we can choose _y_ to be √2-1, which gives us

atan( _x_) = π/8 + atan(( _x_ **-** _y_)/(1+ _xy_))   # With _y_ = √2-1

so all we need to compute now is one arctangent, that of ( _x_ **-** _y_)/(1+ _xy_). The only concern is when that expression nears one. So there are several steps.

First, atan( **-** _x_) is **-** atan( _x_), so we take care of the sign up front to give us a non-negative argument.

Next if \|1 **-** _x_ \|< 0.5, we can use the identity above by recurring to calculate atan(( _x_ **-** _y_)/(1+ _xy_)) with _y_ =√2 **-** 1\. It converges well.

If _x_ is less than 1 the standard series works well:

atan( _x_) = _x_ \- _x_ ³/3 + _x_ ⁵/5 - _x_ ⁷/7 + ...

For values above 1, we use this alternate series:

atan( _x_) = +π/2 - 1/ _x_ \+ 1/3 _x_ ³ -1/5 _x_ ⁵ \+ 1/7 _x_ ⁷ \- ...

Figuring all that out was fun, and the algorithm is nice and fast. (Values troublesomely close to 1 are picked off by the identity above.)

So now we have all the arc-trig functions, and in turn we can do the arc-hyperbolics as well using various identities.

With trigonometry out of the way, it's time to face logarithms and arbitrary powers. As with arctangent, the Taylor series for logarithm has issues. And we need logarithms to calculate arbitrary powers: using Ivy's notation for power,

x \*\* y

is the same as

e \*\* y\*log x

That's the exponential function, which we have implemented, but this necessary rewrite relies on the logarithm of _x_.

(If the power is an integer, of course, we can just use exact multiplication, with a clever old algorithm that deserves another viewing:

> \# pow returns x\*\*exp where exp is an integer.
>
> op x pow exp =
>
> z = 1
>
> :while exp > 0
>
> :if 1 == exp&1
>
> z = z\*x
>
> :end
>
> x = x\*x
>
> exp = exp>>1
>
> :end
>
> z
>
> .5 pow 3
>
> 1/8

That loop is logarithmic (ha!) in multiplications. It is an exercise for the reader to see how it works. If the exponent is negative, we just invert the result.)

The Taylor series for log( _x_) isn't great but if we set _x_ =1 **-** _y_ we can use this series:

log (1- _y_) = **-** _y_ **-** _y_ ²/2 **-** _y_ ³/3 ...

It converges slowly, but it does converge. If _x_ is big, the convergence is very slow but that's easy to address. Since log(1/ _x_) is **-** log( _x_), and 1/ _x_ is small when _x_ is large, for large _x_ we take the inverse and recur.

There's more we can do to help. We are dealing with floating-point numbers, which have the form (for positive numbers):

> mantissa \* 2\*\*exponent

and the mantissa is always a value in the close/open range \[0,0.5), so its series will converge well. Thus we rewrite the calculation as:

> log(mantissa \* 2\*\*exponent) = log(mantissa) + log(2)\*exponent

Now we have a good mantissa we can use in the series and a working exponential function. All that's missing is an accurate value for log(2).

When I was doing this work, I spent a long but unsuccessful time looking for a 3,000-digit accurate value for log(2). Eventually I decided to use Ivy itself, or at least the innards, to compute it. In fact I computed 10,000 digits to be sure, and was able to spot-check some of the digits using [https://numberworld.org/digits/Log(2)/](https://numberworld.org/digits/Log(2)/), which had sample sparse digits, so I was confident in the result.

The calculation itself was fun. I don't remember exactly how I did it, but looking at it now one way would be to compute

log(0.5) = **-** log 2

with the series above. With _x_ =0.5, 1 **-** _x_ also equals 0.5 so

log(2) = **-** log (0.5) = **-**( **-** 0.5 **-** 0.5²/2 **-** 0.5³/3...)

and that converges quickly. The computation didn't take long, even for 10,000 digits.

I then saved the result as a constant inside Ivy, and had all that was needed for _x_\*\*y.

Finally, if we need the logarithm of a negative or complex number, that's easy given what we've already built:

log( _x_) = log(\| _x_ \|) \\* phase( _x_)

To compute the phase, we use the arctangent calculation we worked out before.

And with that, Ivy had square roots, trigonometry, hyperbolic trigonometry, logarithms and powers, all for the whole complex plane. That was it for quite a while.

But come late 2025, I decided to fill one gap that was bothering me: The gamma function 𝚪( _z_), which is defined by the integral

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhac237ZNwsf_PmghFsMPruXy25u-awy_6GI8U8WKv-71z-bPTqgZ-lzdYB9SZZAH84reJm8UMl-mBXrTfU2wP0Kh1XTWM7arsT5XaXtSi3vGTmqdlnde9DrdO6yZ2YvHZ7BaiyZzx3JoaOKP8CBz83RFIHad-sH42wT2BEskhSQBKIg1welLNCuw/s320/gamma.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhac237ZNwsf_PmghFsMPruXy25u-awy_6GI8U8WKv-71z-bPTqgZ-lzdYB9SZZAH84reJm8UMl-mBXrTfU2wP0Kh1XTWM7arsT5XaXtSi3vGTmqdlnde9DrdO6yZ2YvHZ7BaiyZzx3JoaOKP8CBz83RFIHad-sH42wT2BEskhSQBKIg1welLNCuw/s642/gamma.png)

This function has many fascinating properties, but there are three that matter here. First is that if you play with that integral by parts you can discover that it has a recurrence relation

𝚪( _z_ +1) = _z_ 𝚪( _z_)

which is a property shared by the factorial function:

_z+_ 1! = _z_! _z_

Because of this, the gamma function is considered one way to generalize factorial to non-integers, in fact to the whole complex plane except for non-positive integers, which are its poles. For real( _z_) < 0, we can (and do) use the identity

𝚪( _z_) = 𝛑/(sin( _z_ 𝛑)\*𝚪(1- _z_))

It works out that for integer values,

> 𝚪( _z_ +1) = _z_!

which means, if we so choose, we can define factorial over the complex plane using

> z! = 𝚪( _z_ +1)

APL and Ivy both do this.

The second property, besides being easy to evaluate for integer arguments because of the relation with factorial, is that it's also easy to evaluate at half-integers because

𝚪( **½**) = √π

and we can use the recurrence relation to get exact values for all positive half-integers (or at least, as exact as our value of π).

The third property is less felicitous: there is no easy way to compute 𝚪( _z_) for arbitrary _z_ to high precision. Unlike the functions discussed above, there is no Taylor series or other trick, no known mechanism for computing the exact value for all arguments. The best we can do is an approximation, perhaps by interpolation between the values that we do know the exact value for.

With that dispiriting setup, we now begin our quest: How best to calculate 𝚪( _z_) in Ivy?

The first thing I tried was to use a method called the Lanczos approximation, which was described by Cornelius Lanczos in 1964. This gives a reasonable approximate value for 𝚪( _z_) for a standard floating-point argument. It gives 10-12 decimal digits of accuracy, and a float64 has about 16, float32 about 7. For Ivy we want to do better, but when I uncovered Lanczos's approximation it seemed a good place to start.

Here is the formula:

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiMXxtHOpoD7XmIax2lXlf9iqsKhbRzoalm0VNsninpyHu28IylxXLvEBpmzqRcapOU237oWlcJ1au_pCNsI0F1ATxX1jjOvPp18VdsYGWS6OQh02A7KX9SNbnF7vt1p2YmrAIvN251sux_k2Qh3N29LjerA9IzYpWHcuG0tZCtbUCo79mxDPaiBA/s320/lanczos.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiMXxtHOpoD7XmIax2lXlf9iqsKhbRzoalm0VNsninpyHu28IylxXLvEBpmzqRcapOU237oWlcJ1au_pCNsI0F1ATxX1jjOvPp18VdsYGWS6OQh02A7KX9SNbnF7vt1p2YmrAIvN251sux_k2Qh3N29LjerA9IzYpWHcuG0tZCtbUCo79mxDPaiBA/s808/lanczos.png)

with

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiIJ5YU0DBseGMfjL-RPDzo-0WXJUWzaaNhwCu-saJQqWBq503IBLNVYBWG5wFy7WJg5jL0Nufp7jms0zwx6Y7-MHIaz_fioigGsQyAHMLX_Q_r1vBeKSvS0qjKtkDwFshX4G42W_9fXDbf2qCf1NoFG8Uls-RrTIZ5AqJK_L2zABfU_EizZ4LnCA/w200-h66/lanczosA.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiIJ5YU0DBseGMfjL-RPDzo-0WXJUWzaaNhwCu-saJQqWBq503IBLNVYBWG5wFy7WJg5jL0Nufp7jms0zwx6Y7-MHIaz_fioigGsQyAHMLX_Q_r1vBeKSvS0qjKtkDwFshX4G42W_9fXDbf2qCf1NoFG8Uls-RrTIZ5AqJK_L2zABfU_EizZ4LnCA/s416/lanczosA.png)

Those _c_ coefficients are the hard part: they are intricate to calculate. Moreover, because this is an approximation, not a series, we can't just compute a few terms and stop. We need to use all the terms of the approximation, which means deciding a priori how many terms to use. Plus, adding more terms does little to improve accuracy, so something like 10 terms is probably enough, given we're not going to be precise anyway. And as we'll see, the number of terms itself goes into the calculation of the _c_ coefficients.

The [Wikipedia page](https://en.wikipedia.org/wiki/Lanczos_approximation) for the approximation is a good starting point for grasping all this. It has some sample coefficients and a Python implementation that's easy to relate to the explanation, but using it to calculate the coefficients isn't easy.

I found an article by Paul Godfrey very helpful ( [Lanczos Implementation of the Gamma Function](https://www.numericana.com/answer/info/godfrey.htm)). It turns the cryptic calculation in the Wikipedia article into a comprehensible burst of matrix algebra. Also, as a step along the way, he gives some sample coefficients for N=11. I used them to implement the approximation using Wikipedia's Python code as a guide, and it worked well. Or as well as it could.

However, if I were going to use those coefficients, it seemed dishonest not to calculate them myself, and Godfrey explains, if not exactly shows, how to do that.

He reduces the calculation of _c_ coefficients to a multiplication of three matrices, _D_, _B_, _C_, and a vector _F_. I set about calculating them using Ivy itself. I do not claim to understand why the process works, but I am able to execute it.

_D_ is straightforward. In fact its elements are found in my ex-Bell-Labs colleague Neil Sloane's Online Encyclopedia of Integer Sequences as number [A002457](https://oeis.org/A002457). The terms are a( _n_) = (2 _n_ +1)!/ _n_!², starting with _n_ =1\. Here's the Ivy code to do it (All of these calculations can be found in the [Ivy GitHub repo](https://github.com/robpike/ivy) in the file testdata/lanczos). We define the operator to calculate the _n_ th element:

> op A2457 n = (!1+2\*n)/(!n)\*\*2

The operator is called A2457 for obvious reasons, and it returns the _n_ th element of the sequence. Then _D_ is just a diagonal matrix of those terms, all but the first negated, which can be computed by

> N = 6 # The matrix dimension, a small value for illustration here.
>
> op diag v =
>
> d = count v
>
> d d rho flatten v @, d rho 0

The @ decorator iterates its operator, here a comma "raveling" or joining operator, over the elements of its left argument applied to the entire right. The diag operator then builds a vector by spreading out the elements of v with d zeros (d rho 0) between each, then reformats that into a d by d matrix (d d rho ...). That last expression is the return value: operators evaluate to the last expression executed in the body. Here's an example:

> diag 1 2 3 4 5 6
>
> 1 0 0 0 0 0
>
> 0 2 0 0 0 0
>
> 0 0 3 0 0 0
>
> 0 0 0 4 0 0
>
> 0 0 0 0 5 0
>
> 0 0 0 0 0 6

For the real work, we pass a vector created by calling A2457 for each of 0 through 4 (-1+iota N-1), negating that, and adding an extra 1 at the beginning. Here's what we get:

> D = diag 1, -A2457@ -1+iota N-1

> D
>
> 1    0    0    0    0    0
>
> 0   -1    0    0    0    0
>
> 0    0   -6    0    0    0
>
> 0    0    0  -30    0    0
>
> 0    0    0    0 -140    0
>
> 0    0    0    0    0 -630

_B_ is created from a table of binomial coefficients. This turned out to be the most intricate of the matrices to compute, and I won't go through it all here, but briefly: I built Pascal's triangle, put an extra column on it, and rotated it. That built the coefficients. Then I took alternate rows and did a little sign fiddling. The result is an upper triangle matrix with alternating +1 and -1 down the diagonal.

> B
>
> 1   1   1   1   1   1
>
> 0  -1   2  -3   4  -5
>
> 0   0   1  -4  10 -20
>
> 0   0   0  -1   6 -21
>
> 0   0   0   0   1  -8
>
> 0   0   0   0   0  -1

Again, if you want to see the calculation, which I'm sure could be done much more concisely, look in the repo.

Now the fun one. The third matrix, _C_, is built from the coefficients of successive Chebyshev polynomials of the first kind, called Tₙ( _x_). They are polynomials in _x_, but we only want the coefficients. That is, _x_ doesn't matter for the value but its presence is necessary to build the polynomials. We can compute the polynomials using these recurrence relations:

T₀( _x_) = 1

T₁( _x_) = _x_

Tₙ₊₁( _x_) = 2xTₙ( _x_) \- Tₙ₋₁( _x_)

That's easy to code in Ivy. We treat a polynomial as just a vector of coefficients. Thus, as polynomials, 1 is represented as the vector

1

while _x_ is

0 1

and _x_ ² is

0 0 1

and so on. To keep things simple, we always carry around a vector of all the coefficients, even if most are zero. In Ivy we can use the take operator to do this:

> > 6 take 1
>
> 1 0 0 0 0 0

gives us a vector of length 6, with 1 as the first element. Similarly, we get the polynomial _x_ by

> 6 take 0 1
>
> 0 1 0 0 0 0

The timesx operator shows why this approach is good. To multiply a polynomial by _x_, we just push it to the right and add a zero constant at the beginning. But first we should define the size of our polynomials, tsize, which needs to be twice as big as N because, as with the binomials, we will only keep alternate sets of coefficients.

> tsize = -1+2\*N
>
> op timesx a = tsize take 0, a

That is, put a zero at the beginning of the argument vector and 'take' the first tsize elements of that to be returned.

> timesx tsize take 0 1 # makes x²
>
> 0 0 1 0 0 0 0 0 0 0 0

And of course to add two polynomials we just add the vectors.

Now we can define our operator T to create the Chebyshev polynomial coefficients.

> op T n =
>
> n == 0: tsize take 1
>
> n == 1: tsize take 0 1
>
> (2 \* (timesx T n-1)) - (T n-2) # Compare to the recurrence relation.

That T operator is typical Ivy code. The colon operator is a kind of switch/return: If the left operand is true, return the right operand. The code is concise and easy to understand, but can be slow if N is much bigger. (I ended up memoizing the implementation but that's not important here. Again, see the repo for details.) The rest is just recursion with the tools we have built.

To create the actual _C_ matrix, we generate the coefficients and select alternate columns by doubling the argument to T. Finally, for reasons beyond my ken, the first element of the data is hand-patched to 1/2. And then we turn the vector of data into a square N by N matrix:

> op gen x = (tsize rho 1 0) sel T 2\*x
>
> data =  flatten gen@ -1+iota tsize
>
> data\[1\] = 1/2
>
> C=N N rho data
>
> C
>
> 1/2   0    0    0     0   0
>
>  -1   2    0    0     0   0
>
> 1  -8    8    0     0   0
>
>  -1  18  -48   32     0   0
>
> 1 -32  160 -256   128   0
>
>  -1  50 -400 1120 -1280 512

Phew. Now we just need _F_, and it's much easier. We need gamma for half-integers, which we know how to do, and then we can calculate some terms as shown in the article.

> op gammaHalf n = # n is guaranteed here to be an integer plus 1/2.
>
> n = n - 0.5
>
> (sqrt pi) \* (!2\*n) / (!n)\*4\*\*n
>
> op fn z = ((sqrt 2) / pi) \* (gammaHalf(z+.5)) \* (\*\*(z+g+.5)) \* (z+g+0.5)\*\*-(z+0.5)
>
> F = N 1 rho  fn@ -1+iota N  # N 1 rho ... makes a vertical vector
>
> F
>
> 33.8578185471
>
> 7.56807943935
>
> 3.69514993967
>
> 2.34118388428
>
> 1.69093653732
>
> 1.31989578103

This code will seem like arcana to someone unaccustomed to array languages like APL and Ivy, but Ivy was actually a great tool for this calculation. To be honest, I started to write all this in Go but never finished because it was too cumbersome. The calculation of the C matrix alone took a page of code, plus some clever formatting. In Ivy it's just a few lines and is much more fun.

Now that we have all the pieces, we can do the matrix multiplication that Godfrey devised:

(D+.\*B+.\*C)+.\*F

Those are inner products: +.\* means multiply element-wise and then add them up (right to left, remember), standard matrix multiplication. We want the output to have high(ish) precision so print out lots of digits:

> > '%.22g' text (D+.\*B+.\*C)+.\*F
>
> 0.9999999981828222336458
>
> -24.7158058035104436273
>
> -19.21127815952716945532
>
> -2.463474009260883343571
>
> -0.009635981162850649533387
>
> 3.228095448247356928485e-05

Using this method, I was able to calculate the coefficients exactly as they appear in Godfrey's article with size N=11. With these coefficients and the code modeled on the Python from Wikipedia, the result was 10 to 12 digits of gamma function.

I was thrilled at first, but the imprecision of the Lanczos approximation, for all its magic, nagged at me. Ivy is supposed to do better than 10 digits, but that's all we can pull out of Lanczos.

After a few days I went back to the web and found another approximation, one by John L. Spouge from 1994, three decades after Lanczos. It has the property that, although it's still only an approximation, the accuracy can be bounded and also tuned at the expense of more computation. That means more terms, but the terms are easy to calculate and the coefficients are much easier to calculate than with Lanczos, so much so that Ivy ends up computing them on demand. We just need a lot more of them.

Even better, I found a paper from 2021 by Matthew F. Causley that refined the approximation further to allow one to decide where the approximation can be most accurate. That paper is [The Gamma Function via Interpolation](https://arxiv.org/pdf/2104.00697v1) by Matthew F. Causley, 2021.

Let's start with Spouge's original approximation, which looks like this:

[![](https://blogger.googleusercontent.com/img/a/AVvXsEhC_juuD0R9zibPdKIb8WttQl9EhtKockMNaiKiMY3xxdDsueOTdFt48BE1XSrhnnvJDsxGovaE-2yeqwe7R_aeVKCtu6JS9zjzUu0FHm5Fv7Hi3hj4DdiPfOoDkyDYpOgSAq5lYvINw_AAY8Oz1CsmK31yhV0DGHBTSkfV84UniCZ3cYMNthTjmA)](https://blogger.googleusercontent.com/img/a/AVvXsEhC_juuD0R9zibPdKIb8WttQl9EhtKockMNaiKiMY3xxdDsueOTdFt48BE1XSrhnnvJDsxGovaE-2yeqwe7R_aeVKCtu6JS9zjzUu0FHm5Fv7Hi3hj4DdiPfOoDkyDYpOgSAq5lYvINw_AAY8Oz1CsmK31yhV0DGHBTSkfV84UniCZ3cYMNthTjmA)

where

[![](https://blogger.googleusercontent.com/img/a/AVvXsEjecqfHe1_g1yo-rHv1bjUG3MCkP-Muqb-NpXtl3NlI_Cu99lQ-apIwpX4VxsFpA8gXoy_qHG0T6YzE3h2KMEezEeGqV1bGdD8vVUUXNulCpR22UjGUDmG4GZQIt3LGmxT7Fy0n4wXAwyK2kwQ7eNL_UY7dCY6NXS-h0f5rG1fMNIr_3Dy9Ih6pGA)](https://blogger.googleusercontent.com/img/a/AVvXsEjecqfHe1_g1yo-rHv1bjUG3MCkP-Muqb-NpXtl3NlI_Cu99lQ-apIwpX4VxsFpA8gXoy_qHG0T6YzE3h2KMEezEeGqV1bGdD8vVUUXNulCpR22UjGUDmG4GZQIt3LGmxT7Fy0n4wXAwyK2kwQ7eNL_UY7dCY6NXS-h0f5rG1fMNIr_3Dy9Ih6pGA)

Notice that the variable _a_ is not only the number of terms, it's also part of the calculations and the coefficients themselves. As in Lanczos, this is not a convergent series but an interpolated approximation. Once you decide on the value of _a_, you need to use all those terms. (I'll show you some of the coefficients later and you may be surprised at their value.)

The calculation may be long, but it's straightforward given all the groundwork we've laid in Ivy already and those _c_ coefficients are easy to compute.

What Causley adds to this is to separate the number of terms, which he calls N, and the variable based on N that goes into the calculations themselves, which he calls _r_. In regular Spouge, _a_ = _r_ =N, but Causley splits them. Then, by brute force if necessary, we look for the value of _r_ that gives us the best approximation at some value of _z_, called _zbar_, of our choosing. In Causley's paper, he shows his calculated N and optimal _r_ values for a few integer values of _zbar_. The values in his table are not precise enough for our purposes, however. We need to compute our own.

Here it makes sense to calculate more terms than with Lanczos. Empirically I was able to get about half a digit for every term, and therefore decided to go with N=100, which gives about 50 digits or more, at least near _zbar_.

The calculation is much simpler than with Lanczos. Given N and _r_, here's how to calculate the vector of coefficients, _c_ ₙ:

> > op C n =
>
> > t = (1 -1)\[n&1\]/!n
>
> > u = e\*\*r-n
>
> > v = (r-n)\*\*n+.5
>
> > t\*u\*v
>
> > c = N 1 rho (C@ iota N)

Once we have those coefficients, the gamma function itself is just the formula:

> > op gamma z =
>
> > p = (z+r)\*\*z-.5
>
> > q = \*\*-(z+r)
>
> > n = 0
>
> > sum = cinf
>
> > :while n <= N-1
>
> > sum = sum+c\[n\]/(z+n)
>
> > n = n+1
>
> > :end
>
> > p\*q\*sum

The variable cinf is straight from the paper. It appears to be a fixed value.

> cinf = 2.5066 # Fixed by the algorithm; see Causley.

I ended up with N=100 and _r_ =126.69 for _zbar_ =6\. Let's see how accurate we are.

> )format %.70f # The number of digits for a 256-bit mantissa, not counting the 8 left of the decimal sign.
>
> gamma 12
>
> 39916799.9999999999999999999999999999999999999999083377203094662100418136867266

The correct value is !11, which is of course an integer (and computed as such in Ivy):

> !11
>
> 39916800

They agree to 48 places, a huge step up from Lanczos and a difference unlikely to be noticed in normal work.

Have a gander at the first few of our cₙ coefficients with N=11 and _r_ =126.69:

> )format %.12e
>
> c
>
> 1.180698687310e+56
>
> -5.437816144514e+57
>
> 1.232332820715e+59
>
> -1.831910616577e+60
>
> 2.009185300215e+61
>
> -1.733868504749e+62
>
> 1.226116937711e+63
>
> ...

They have many more digits than that, of course, but check out the exponents. We are working with numbers of enormous size; it's remarkable that they almost completely cancel out and give us our gamma function with such accuracy. This is indeed no Taylor series.

I chose N because I wanted good accuracy. I chose _r_ because it gave me excellent accuracy at gamma 6 and thereabouts, well over 50 digits. To find my value of _r_, I just iterated, computing _C_ and then gamma using values of _r_ until the error between gamma 6 and !5 was as small as I could get. It's explained in the repo in the file testdata/gamma.

But there remains one mystery. In Causley's paper, _r_ is closer to N, in fact always less than one away. So why did I end up with such a different value? The result I get with those parameters is excellent, so I am not complaining. I am just puzzled.

The optimization method is outlined in Causley's paper but not thorougly enough for me to recreate it, so I did something simpleminded: I just varied _r_ until it gave good results at _zbar_ =6\. Perhaps that's naive. Or perhaps there is a difference in our arithmetical representation. Causley says he uses variable-precision arithmetic but does not quantify that. I used 256-bit mantissas. It's possible that the highest-precision part of the calculations is exquisitely sensitive to the size of the mantissa. I don't know, and would like to know, what's going on here. But there is no question that my values work well. Moreover, if I instead choose _r_ =N, as in the original paper by Spouge, the answer has 38 good digits, which is less than in my version but also much more than a 64-bit float could deliver and perhaps more than Causley's technique could capture. (I do not mean to impugn Causley here! I just don't know exactly how he computed his values.)

However that may play out, there may be only one person who cares about the accuracy of the gamma function in Ivy. And that person thoroughly enjoyed exploring the topic and building a high-resolution, if necessarily imperfect, implementation.

Thanks for reading. If you want to try Ivy,

go install robpike.io/ivy@latest
