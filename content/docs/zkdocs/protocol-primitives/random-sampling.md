---
weight: 1
bookFlatSection: true
summary: "In this section, we describe how to uniformly sample from different groups."
---
# Random sampling

In most protocols, it is necessary to sample uniformly from groups like $\zq$ or $\zqs$. Here we will describe how to sample in these specific groups, and a general procedure to sample via rejection sampling.

When available it is advisable to use a cryptographic random number library to securely sample uniformly from a range. For example, to sample uniformly from $\zq$ (or equivalently $\range{q}$) in Python, use the [secrets](https://docs.python.org/3/library/secrets.html) library.

```python
import secrets

def sample_Zq():
    return secrets.randbelow(q)
```

## Sampling from $\zq$
Sampling uniformly from $\zq$ is equivalent to sampling uniformly from the set of representatives, i.e. the range $\range{q}$. We present two secure methods for doing so: wide modular reduction gives samples from an almost-uniform distribution with a (deterministic) logarithmic number of random bits, while rejection sampling gives samples from a truly uniform distribution using a logarithmic number of bits *with high probability*.


### Almost-uniform sampling in $\zq$ using modular reduction
Suppose we only have a cryptographically-secure random bit generator and we would like to sample uniformly from the range $[q]$. As an example we will use the secp256k1 curve order
$$
q = 2^{256} - 432420386565659656852420866394968145599
$$

{{< hint danger >}}
**Never do this:**
```python
def wrong_sampling():
    q = 2**256 - 432420386565659656852420866394968145599
    return randbits(256) % q
```
The `wrong_sampling` function introduces non-negligible *modulo-bias* in the generation of elements, and some elements will be detectably more likely to appear than others. If the random number is used as an ECDSA nonce, this [may be enough to fully compromise the key](https://blog.trailofbits.com/2020/06/11/ecdsa-handle-with-care/).
{{< /hint >}}

Instead, sample at least 512 bits:
```python
from secrets import randbits
def correct_sampling():
    q = 2**256 - 432420386565659656852420866394968145599
    return randbits(512) % q
```

In general, if the overall security level of your system is $\lambda$ bits (e.g., 128), then to safely sample from the range $[q]$ where $2^{n-1} < q < 2^{n}$, it is necessary to reduce
$n + \lambda$ cryptographically random bits mod $q$.

### Modulo Bias
A common approach to sampling from a range $[q]$ is to generate a random bit string $b \in \\{0,1\\}^n$, interpret it as a natural number in the range $[2^n - 1]$, then return $x = b\mod q$. However, this process introduces *modulo bias*. As a simple example where $q = 3, n = 2$, let $b$ be uniformly drawn from $\\{0,1\\}^2 \cong [4]$ and let $x = b \mod 3$. Then
$$
\begin{align*}
&\Pr[x = 0] = \Pr[b = 0 \vee b = 3] = 1/2\\\\
&\Pr[x = 1] = \Pr[b = 1] = 1/4\\\\
&\Pr[x = 2] = \Pr[b = 2] = 1/4\\\\
&\end{align*}
$$

Intuitively, the bias arises because some elements of $\zq$ are double-counted. In order to minimize the modulo bias, we can increase the number of random bits drawn. For $q = 3$ and $n = 8$,
$$\Pr[x = 0] = \frac{86}{256} = \frac{1}{3} + \frac{1}{384}$$

For $n = 256$
$$\Pr[x = 0] = \frac{1}{3} + \frac{1}{3\cdot 2^{255}}$$
No real-world adversary can distinguish the distribution generated in the $n = 256$ case from the uniform distribution on $\mathbb{Z}_3$.

In general, if $q$ is an $n$-bit integer, i.e. $2^{n-1} \leq q < 2^n$ and $X$ is a random variable drawn uniformly from $[2^{n + \lambda}]$ where $\lambda \in \mathbb{N}$ is some security parameter, then for any $a \in \range{q}$

$$
\left|\Pr[X \equiv a \mod q] - \frac{1}{q}\right| \leq \frac{1}{2^{\lambda}}
$$

In practice, if $q$ is $n$ bits then typically $\lambda \leq n$, so it suffices to sample $2n$ bits and reduce mod $q$. For example, the ed25519 nonce generation proceeds by reducing the output of HMAC-SHA512 modulo the 255-bit order of the curve.

#### Statistical Distance

The *statistical distance*, also known as the "total variation distance" of the distribution of a random variable $X$ from that of another random variable $Y$ both with support $[q]$ is defined as
$$
\Delta(X,Y) = \frac{1}{2}\sum_{a = 0}^q |\Pr[X = a] - \Pr[Y = a]|
$$

The statistical distance gives an upper bound on the success probability of any adversary at distinguishing two distributions. For any function $A: [q] \rightarrow \\{0,1\\}$:

$$
|\Pr[A(X) = 1] - \Pr[A(Y) = 1]| \leq \Delta(X, Y)
$$
and in fact if $X_1, \dots X_n$ are identically and independent distributed and similarly for $Y_1, \dots, Y_n$ then

$$
|\Pr[A(X_1, \dots, X_n) = 1] - \Pr[A(Y_1, \dots, Y_n) = 1]| \leq n \cdot \Delta(X_1, Y_1)
$$

Letting $U$ be the uniform distribution over $[q]$, define $S(X) = \Delta(X, U)$. It follows that if $S(X) \leq 2^{-\lambda}$ then no adversary can distinguish $X$ from the uniform distribution with probability more than $1/2$ using fewer than $2^{\lambda - 1}$ samples.

#### Statistical distance of wide modular reduction
Let $X_{n}$ denote random variable resulting from sampling uniformly in $[2^n]$ and reducing modulo $q$. Let $U$ be a random variable uniformly distributed over $q$.
We can compute the statistical distance of $X_n$ from $U_{q}$ explicitly:
Let $2^n = k\cdot q + r$ for $0 \leq r < q$. Then for all $a < r$

$$
\Pr[X = a] = \frac{k + 1}{2^n} = \frac{\frac{2^n - r}{q} + 1}{2^n} = \frac{2^n - r + q}{2^n q} = \frac{1}{q} + \frac{q - r}{2^n q}
$$

and so
 $$
|\Pr[X = a] - 1/q| = \frac{q - r}{2^n q}
 $$
while a similar computation for $a \geq r$ gives
$$
|\Pr[X = a] - 1/q| = \left|\frac{k}{2^n}  - \frac{1}{q}\right| = \frac{r}{2^n q}
$$

and thus in total
$$
S(X) = \frac{1}{2}\sum_{a = 0}^{r-1} \frac{q - r}{2^n q} + \sum_{a = r}^{q-1} \frac{r}{2^n q} = \frac{r}{2} \cdot \frac{q - r}{2^n q} + \frac{q - r}{2}  \cdot\frac{r}{2^n q} = \frac{r(q - r)}{2^n q}
$$

This formula tells us some interesting things about modulo bias:
* When $q$ is a power of 2 less than $2^n$ then $r = 0$, so there is no modulo bias
* In the worst case, when $r = q/2$, $S(X) = q/2^{n+2}$. If we choose "just enough" bits, such that $2^{n-1} < q < 2^n$, then $S(X) \geq 1 / 8$. This is a very substantial advantage, enough to clearly distinguish from uniform after only a few trials.
* If we $\lambda$ is a security parameter and $q$ is an $n$-bit number then by selecting $n + \lambda$ bits we get a statistical distance of at most $\frac{1}{2^{\lambda + 2}}$.

## Rejection sampling
Suppose we know how to sample from a set $S$ and would like to sample from an arbitrary subset $T\subseteq S$.
To do this, we need to check membership in $T$ and perform rejection sampling. In a loop:
 1. sample element $e \in S$.
 2. if $e \in T$, return $e$, otherwise repeat step 1.

On average this requires looping $|S|/|T|$ times. If $|S| \leq 2|T|$, the probability that we will need to sample more than $k$ elements from $S$ is at most $2^{-k}$. Thus as long as $|S|$ is not too much bigger than $|T|$, the rejection sampling algorithm almost always terminates in only a few iterations.

### Uniform sampling in $\zqs$
Elements of $\zqs$ are natural numbers $e$ such that $e \in [q]$ and $\gcd(e, q) = 1$.
To sample these elements uniformly we need to perform rejection sampling.
In a loop,
 1. Sample element $e \in [q]$.
 2. If $\gcd(e, q) = 1$ return $e$, otherwise repeat step 1.


```python
import secrets
import math

def sample_Zqstar():
    while True:
        e = secrets.randbelow(q)
        if math.gcd(e, q) == 1:
            return e
```
{{< hint info >}}

**Note:**
Since $\gcd(0, q)$ is never 1, we could also sample $e$ from the set $\rangeone{q}$.
{{< /hint >}}
