---
weight: 1
bookFlatSection: true
title: "Notation & Definitions"
summary: "Common notation and definitions used in the documentation."
references: ["hoc"]
---
# Notation and Definitions

This page is a glossary for notation and concepts present in the documentation.

## Sets, Groups, and Special Functions
 - $\mathbb{Z}$ is the set of integers, $\\{\ldots, -2, -1, 0, 1, 2, \ldots\\}$.
 - $\naturals$ is the set of integers greater than or equal to 0, $\\{0, 1, 2, \ldots\\}$.
 - $\range{b}$ is the finite set of integers $\\{0, \ldots, b-1\\}$.
 - $\gcd(n, m)$ is the nonnegative [greatest common divisor](https://en.wikipedia.org/wiki/Greatest_common_divisor) of integers $n$ and $m$; when $\gcd(n, m) = 1$, $n$ and $m$ are said to be *coprime*.
 - $\z{n}$ are the integers modulo $n$, a set associated with the equivalence classes of integers $\\{0, 1, \ldots, n-1\\}$.
 - $\zns{n}$ is the multiplicative group of integers modulo $n$: an element $e$ from $\z{n}$ is in $\zns{n}$ iff $\gcd(e, n) = 1$, that is $\zns{n} = \\{e \in \z{n}: \gcd(e, n) = 1\\}$. When $n$ is prime, then $\zns{n} = \\{1, \ldots, n-1\\}$.
 - $\field{p}$ is the finite field of order $p$; when $p$ is a prime number, these are the integers modulo $p$, $\z{p}$; when $p$ is a prime power $q^k$, these are [Galois fields](https://en.wikipedia.org/wiki/Finite_field).
 - $\varphi(n)$ is Euler's [totient function](https://en.wikipedia.org/wiki/Euler%27s_totient_function); for $n\geq 1$, it is the number of integers in $\\{1,\ldots, n\\}$ coprime with $n.$
 - $|S|$ is the *order* of a set $S$, i.e., its number of elements. For example, $|\zns{n}| = \varphi(n)$, and for a prime $n$, $|\zns{n}| = n-1$.

## Vectors
 - $\vec{a} \in \z{q}^n$ is a vector $(a_1,\dots,a_n)$ with $a_i \in \z{q}$ for all $i$.
 - $c \cdot \vec{a}$ denotes the scalar product $(c \cdot a_1,\dots,c \cdot a_n)$.
 - $\ip{\vec{a}}{\vec{b}}$ denotes the inner product $\sum_{i=1}^n a_i \cdot b_i$.

## Number-theory
 - $J(w, n)\in \\{-1, 0, 1\\}$ is the [Jacobi symbol](https://en.wikipedia.org/wiki/Jacobi_symbol) of $w$ modulo $n$, only defined for positive and odd $n$.
 - $J_n$ is the set of elements of $\zns{n}$ with Jacobi symbol $1$.
 - $QR_n$ is the set of quadratic residues modulo $n$, which are elements that have a square-root, i.e., $QR_n = \\{e \in \z{n} : \exists r . r^2 = e \mod n\\}$.

## Sampling
In protocol specifications, we will often need to uniformly sample elements from sets. We will use the following notation:
 - $\sampleGeneric{x}{X}$, where $x$ is uniformly sampled from the set $X$.

Consider reading the section on [Random Sampling]({{< relref "/docs/zkdocs/protocol-primitives/random-sampling" >}}) to learn how to correctly sample a number uniformly  using rejection sampling, avoiding the modulo-bias issue.

## Assertions
We will use assertions in protocol descriptions. When the assertions do not hold, the protocol must abort to avoid leaking secret information.
 - $a \equalQ b$, requires $a=b$, and aborts otherwise
 - $a \gQ b$, requires $a>b$, and aborts otherwise
 - $a \inQ S$, requires that $a$ is in the set $S$, and aborts otherwise.

## Implementations of number-theoretic algorithms
In general, we highly recommend the [Handbook of Applied Cryptography](https://cacr.uwaterloo.ca/hac/), which has detailed descriptions of most algorithms.

## Hash Functions
 - $\hash{\cdot}$ is a cryptographically secure domain-separated hash function.
 - $\hashbit{\cdot}{k}$ is a cryptographically secure domain-separated hash function with specific output-size of $k$-bits.

Find more details on the particular hash functions in [Nothing-up-my-sleeve constructions]({{< relref "/docs/zkdocs/protocol-primitives/nums#hash-function-choice" >}})
