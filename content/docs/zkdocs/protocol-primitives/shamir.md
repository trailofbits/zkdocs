---
weight: 9
bookFlatSection: true
title: "Shamir's Secret Sharing Scheme"
summary: "An overview of Shamir's Secret Sharing scheme and potential security pitfalls."
references: ["shamir"]
---
# Shamir’s Secret Sharing Scheme
[Shamir's Secret Sharing Scheme](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing) is a way of splitting a secret value $S$ into $n$ "pieces" (or "shares") in such a way that any combination of $k$ pieces can be used to recover $S$, but any $k-1$ or fewer pieces provide _no_ information about $S$.

## Overview of Secret Sharing

Shamir's Secret Sharing Scheme relies on a very useful property of polynomials: any degree-$k$ polynomial can be uniquely identified by _any_ set of $k+1$ distinct points. Two points define a line; three points define a parabola. However, with fewer than $k+1$ distinct points, no further information can be gained about the polynomial.

This leads directly to Shamir's approach: encode a secret $S$ into a polynomial over a finite field, and distribute points on the polynomial as shares.

### Splitting $S$

Given a secret $S$, a number of desired shares $n$, and a threshold $k$, splitting $S$ comes down to two steps:
  - encode $S$ into a secret polynomial $f\left(x\right)$ of degree $k-1$ over a finite field $\field{p}$
  - generate distinct shares $\left(x_{i}, f(x_i)\right)$

####  Encoding $S$ in a Polynomial

Shamir proposed a straightforward method for encoding secrets into polynomials: set the constant term of a random polynomial to the secret $S$, and select all other coefficients uniformly at random. For a degree-$(k-1)$  polynomial, there would be $k-1$ random coefficients $r_i$, and the constant coefficient of $S$:
$$f(x) = S + r_1 x + \ldots + r_{k-1} x^{k-1} \enspace.$$

{{< hint info>}}
**Example:** We want to share our secret number 42 with three players so that any two of them can recover it. We define the degree-1 polynomial over $\field{73}$ as
$$f(x) = 42 + 13 \cdot x \enspace,$$
where 13 was randomly sampled over $\field{73}$. We evaluate the polynomial at different points obtaining the shares $(x_i, f(x_i))$. Then, we can share 1 point of the coefficient to the three players, each getting one of $(1, 55), (2, 68), (3, 81)$. Since the polynomial is a line, any two of them could meet and recover the secret value 42!
{{< /hint >}}

This choice has the advantage of relative simplicity; there is no need to use oddball sampling techniques to select the coefficients of $f\left(x\right)$.

##### A Note on Selecting $p$
In the _general_ case, the specific prime $p$ does not matter much. Shamir's original paper proposed using 16-bit primes (the largest of which would be $p=2^{16}-15=65521$) for performance reasons. By limiting intermediate results to 32 bits, multi-precision arithmetic routines could be avoided on any 32-bit processor. Large secrets could be broken into blocks of 16 bits or less and shared with distinct polynomials. One limitation imposed by such a small $p$ is that only about 65000 distinct shares can be generated.

In practice, $p$ should be reasonably large. Breaking $S$ into multiple parts introduces complexity and opportunities for malicious actors. Also, in some verifiable secret sharing schemes, a large $p$ is needed to prevent discrete log attacks.

#### Generating the Shares

Shares $s_{1},s_{2},\ldots,s_{n}$ of $S$ are ordered pairs $\left(x_{i}, f\left(x_{i}\right)\right)$. The $x_{i}$ values can be picked in several ways, but a counter starting in 1 is the most common method.

### Recovering $S$

The most common and direct method of recovering $S$ from the shares $s_{1},\ldots,s_{k}$ is to evaluate $f\left(0\right)=S$ using Lagrange interpolation. It allows computing $f\left(t\right)$ for any $t\in\field{p}$ using the formula below:

$$f\left(t\right) = \displaystyle\sum_{j=1}^{k}{y_{j}\left(\displaystyle\prod_{\begin{smallmatrix}1\leq m\leq k\\\\ m\neq j\end{smallmatrix}}{\frac{t-x_m}{x_j - x_m}}\right)}$$

The $t=0$ case slightly simplifies the product. Each product term can also be precomputed since it only depends on the $x$-coordinate of the shares that can be publicly known.
$$f\left(0\right) = \displaystyle\sum_{j=1}^{k}{y_{j}\left(\displaystyle\prod_{\begin{smallmatrix}1\leq m\leq k\\\\ m\neq j\end{smallmatrix}}{\frac{x_m}{x_m - x_j}}\right)}$$

The Lagrange interpolating polynomial is popular because it's fast enough for several use cases and relatively simple to implement.

## The Zero Share Problem

### How Shares are Generated

As mentioned above, there are several ways to select the $x_{i}$ values during the share-generation phase of the algorithm.

The most common approach, and the one we recommend, is to use a counter, setting $x_{i}=i$ for $i=1,2,\ldots,n$. This has the advantage of simplicity and speed. No user-generated inputs are necessary, and evaluating $f\left(x\right)$ when $x$ is small can be faster than for arbitrary-selected values in the field.

Another approach is to use unique values associated with shareholders, such as userid values from a database, or users' provided values.

Finally, some implementations select $x_{i}$ by selecting them randomly from $\field{p}$.

### What Can Go Wrong?

 * **Zero share problem:** If it is possible to wind up with $x_{i}=0$ for some $i$, the corresponding share is $\left(0, S\right)$, revealing $S$ directly to the $i$-th shareholder. This is not a hypothetical attack; several instances of this problem have been spotted in the wild. Implementations must guarantee that all $x_i$ are modularly nonzero.
 * **Non-unique shares:** The $x_i$ for each value must be modularly unique; when users reconstruct the secret from a share, they need to compute $\frac{x_m}{x_m-x_j}$. If the polynomial is evaluated modulo $q$ and if $x_m\equiv x_j \pmod{q}$, the modular inverse of $(x_m-x_j)$ does not exist and the protocol does not work. Some applications do not check if this modular inverse exists and usually do not handle this gracefully.

### How Things Go Wrong

 - **Counter-based shares:** One of the most common control structures in programming is a loop with a starting index of `i = 0`. The line `for(int i = 0; i < n; i++)` is near-idiomatic in C; `for i in range(n):` is an idiom in Python. It's easy to fall into an old pattern and fail to update these to the correct `for(int i = 1; i <= n; i++)` and `for i in range(1, n + 1):`, respectively. An initial draft of this page _included this very error_ in the "correct" C loop! This common error is enough to destroy the security of the system entirely.

- **Userid-based shares:** Trusting unique user identifiers is problematic, too. A user can have a userid of 0 in plenty of systems. Trusting user-supplied values in security-critical contexts is never a good idea. Even if you compare the userid with 0, you might fail to do it correctly: since the polynomial evaluates over the finite field $\field{p}$, you need to check if it is 0 in $\field{p}$. In the case of the integers modulo a prime number, you must check that the userid is *modularly* different from zero.
Even transforming user inputs can be problematic if you're not careful. For instance, given a user input $n$, a program may try to avoid this issue by taking the $x$-coordinate of $n\cdot G$, where $G$ is a generator of an elliptic curve group. However, if $n$ is 0, the resulting point is the "point at infinity". With some curve parameterizations, the point at infinity is represented as $\left(0,1\right)$.

- **Random shares:** Selecting random values seems safe– in theory, the probability of selecting 0 from $\field{p}$ is $\frac{1}{p}$. But winding up with a zero share is significantly more likely than that. Consider the case where a program generates random numbers by reading from `/dev/random`. For Linux kernels before version 5.6, it is possible for `/dev/random` to block when the "entropy runs out". Since Linux 5.6, `/dev/random` can still block if the PRNG has not been initialized. If a programmer does not check the return value of the `read` function, then depending on how the destination buffer is allocated, the destination buffer can be left in a zeroed state, leading to a zero share.

