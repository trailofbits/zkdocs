---
weight: 12
bookFlatSection: true
title: "Alternative versions of Shamir's Secret Sharing scheme"
summary: "Secret sharing alternatives which do not hide the secret in the constant term of the polynomial."
---
# Alternative Secret Sharing Solutions

There are multiple ways to prevent inadvertently sharing the secret value by evaluating the polynomial at zero. In [Shamir Secret Sharing](../shamir), we described defense strategies to solve that problem in classical Shamir's Secret Sharing scheme. Here, we describe alternative implementations of Shamir's scheme, which ensure that evaluating $f\left(0\right)$ does not reveal $S$.

{{<hint warning>}}
These alternative schemes are certainly harder to implement (and to review), and their added complexity might not be worth the ability to mistakenly leak $f(0)$, since they can still leak the secret value by leaking other points of the polynomial.
{{</hint>}}

### Transforming the Inputs Using a Nonzero Function

Suppose we are working over $\z{p}$, where $p=2^{255}-19$. Consider the multiplicative group $\zns{p}$ of integers less than $p$  and relatively prime to $p$. We can define $h\left(m\right)=2^{m}\pmod{p}$. Since $0\notin \zns{p}$, we know that $h\left(m\right)\neq 0$ for any integer $m$. Since $2$ is a generator of $\zns{p}$, we know that $h\left(m\right)$ has maximal order in $\zns{p}$.

During share generation, given an integer input $x$, define $x'=h\left(x\right)$. Then, generate the corresponding share according to $\left(x',f\left(x'\right)\right)$. Because $x'\neq 0$, counters and external inputs can be used without the risk of generating a zero share.

This technique is compatible with the secret reconstruction using Lagrange interpolation: as far as the Lagrange interpolating polynomial goes, a share formed from the transformed values works the same as any other shares.

### Avoiding the $y$-intercept

It is possible to encode the secret $S$ into a polynomial such that $f\left(t\right)=S$ for a value $t\neq 0$. The process is only a little more complicated than the standard Shamir scheme:

  - Select a non-zero $t\in\field{p}$
  - Select a random degree-$(k-1)$ polynomial $g\left(x\right)\in\field{p}\left[x\right]$
  - Compute $S'=S-g\left(t\right)$
  - Set the final polynomial to $f\left(x\right)=g\left(x\right)+S'$

We now have $f\left(t\right)=g\left(t\right)+S'=g\left(t\right)+S-g\left(t\right)=S$. The Lagrange interpolating polynomial can compute $f\left(x\right)$ at $x=t$  just as easily as at $x=0$.

If shares are generated sequentially, selecting a value of $t$ larger than any reasonable number of shares $n$ (e.g., $t\gg 2^{64}$) can prevent creating an insecure share of the form $\left(t,f\left(t\right)\right)$.

If shares are generated randomly, it's still possible to wind up with an insecure share by randomly selecting $t$. However, generating this value at random is significantly less likely than accidentally "generating" a zero share from a blocked read.

### Moving Away from the Constant Term

There is no requirement that $S$ be encoded into $f\left(x\right)$ through the manipulation of the constant coefficient. It's possible to encode $S$ into, say, the linear coefficient of $f\left(x\right)$. Recovering the linear coefficient is straightforward, once you realize that the linear term of $f\left(x\right)$ is the constant term of $f'\left(x \right)$, the derivative of $f\left(x\right)$.

The process for generating a polynomial becomes:

  - Generate a random, degree-$(k-3)$ polynomial $g\left(x\right)$
  - Select a random $c\in\field{p}$
  - Set $f\left(x\right)=g\left(x\right)x^2+Sx+c$

The Lagrange interpolating polynomial can be adapted to evaluate derivatives of polynomials; the equation used to compute the first derivative is below. The equation is slightly more complicated than before but certainly within the capability of a good programmer.

$$f'\left(t\right)=\displaystyle\sum_{j=1}^{k}{y_{j}\left(\displaystyle\sum_{\begin{smallmatrix}i=1\\\\ i\neq j\end{smallmatrix}}^{k}{\left[\frac{1}{x_{j}-x_{i}}\displaystyle\prod_{\begin{smallmatrix}m=1\\\\ m\neq i,j\end{smallmatrix}}^{k}{\frac{t-x_{m}}{x_{j}-x_{m}}}\right]}\right)}$$

Evaluating $f'\left(0\right)$ gives the linear coefficient of $f\left(x\right)$, recovering $S$.

### Requiring Recovery of the Full Polynomial

Finally, it's possible to encode the secret across _all_ coefficients of $f\left(x\right)$. Take $g\left(x\right)=\displaystyle\sum_{i=0}^{k-1}{a_{i}x^{i}}$ , where each $a_{i}$ is chosen uniformly at random (except that $a_{k-1}\neq 0$), then we can set $M=\displaystyle\sum_{i=0}^{k-1}{a_{i}}$  and $R=S - M$.  We then select a random $0\leq i\leq k-1$ and set $f\left(x\right)=g\left(x\right)+Rx^i$. Then the sum of the coefficients is equal to $S$.

Going back to the original Lagrange interpolating polynomial equation, if $t$ is replaced with a symbol $x$, the Lagrange interpolating polynomial recovers $f\left(x\right)$ exactly. $S$ can then be recovered from the reconstructed polynomial. Symbolic computations are more expensive and slightly more complex, but for, say, $k<30$, the algorithm works just fine.
