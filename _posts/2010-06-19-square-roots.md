---
title: "Square Roots"
date: "2010-06-19"
categories:
  - "math"
tags:
  - "abacus"
  - "algorithms"
  - "computing"
  - "integer"
  - "roots"
use_math: true
---

Here's a computational question: how do I tell if an integer is a perfect square?

Possibly the most straightforward way is to compute an approximation (e.g. via Newton's method) to the square root, truncate, then square and compare.

This seems somehow unsatisfying to me. I want an exact answer, and I would like to use only integer operations to get there.

Let's forget about computers for a minute. It turns out people have been computing integer square roots for a very long time---via an abacus. There are several abacus-based algorithms for computing a square root, but they essentially boil down to this observation:

$ (a+b)^2 = a^2 +2ab + b^2$.

That's an unassuming little formula, but consider what happens when we start thinking about numbers positionally. In real life, we typically write an integer as a string of base-$ k$ digits. We will denote this string-notation using curly brackets; i.e. $ \\{1234\\}\_{k} = 1 \\times 10^3 + 2 \\times 10^2 + 3 \\times 10^1 + 4 \\times 10^0$

To simplify things, when we talk about base 10, we will drop the subscript $k$.

Now think about squaring a two-digit base-10 integer:

$ \{ab\}^2 = (10a + b)^2 = 100a^2 + 20ab + b^2$

A rough approximation, then, would be to guess at the first digit, $ a$, and then choose an appropriate $ b$ from the above formula. By iterating this process over successively accurate choices for $ a$, we eventually reach a solution.

Another way to think of this iteration is to consider:

$ (A+B+C)^2 = (A+B)^2 + 2(A+B)C + C^2$

Setting $ A=100a, B=10b, C=c$, we quickly see that

$ \\{abc\\}^2 =100 \\{ab\\}^2+ 20\\{ab\\}c + c^2$

And this pattern works for any length digit string.

This leads to the [abacus algorithm][1]. Did you know you could do square roots on an abacus? The education system really failed me here.

[1]: http://webhome.idirect.com/~totton/soroban/KojimaSq/
