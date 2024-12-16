---
weight: 10
bookFlatSection: true
title: "KZG Polynomial Commitments"
summary: "Kate et. al's Pairing-Based Polynomial Commitments"
needsVariableResetButton: true
references: ["KZG2010", "verkle", "PLONK", "Groth16", "pairing-curves", "pairing-based-crypto",]
---
# KZG Polynomial Commitments

## Overview of Polynomial Commitments
Polynomial commitment schemes allow one party to prove to another the correct evaluation of a polynomial at some set of points, 
without revealing any other information about the polynomial. A generalization of scalar commitment schemes such as [Pedersen commitments]({{< ref "./pedersen.md" >}}), 
polynomial commitments are an important building block in zero-knowledge protocols; once the prover has committed to a polynomial, the verifier can query the value of the polynomial at one or many points
and be assured that the responses are consistent with some predetermined polynomial which was selected without knowledge of the challenge queries.

Kate et. al.'s polynomial commitment scheme {{< cite KZG2010 >}} consists, similarly to [Pedersen commitments]({{< ref "./pedersen.md" >}}), of two phases. 
- During the _commit_ phase, $\varprover$ generates a commitment to some polynomial $\varf(\varx)$ and shares it with $\varverifier$.
- During the _open_ phase, $\varprover$ evaluates $\varf$ at some point $\varx_0$ and sends $\langle \varx_0, \varf(\varx_0), \varw\rangle$, where $\varw$ is a proof (or "witness") such that $\varverifier$ can check the correct evaluation of $\varf$ at $\varx_0$.

The commitment satisfies certain security properties:
- _Hiding_: Given a commitment to an unknown degree-$t$ polynomial $\varf$ and up to $t$ openings $\langle \varx_1, \varf(\varx_1), \varw_1\rangle, \dots, \langle \varx_t, \varf(\varx_t), \varw_t\rangle$, one cannot find the value of $\varf$ at any point not already queried. 
- _Binding_: It is computationally infeasible to construct a commitment $\varC$ and two distinct valid openings $\langle\varx_0, \vary, \varw\rangle, \langle\varx_0, \vary', \varw'\rangle$ at the same point $\varx_0$.

----

## Applications of Polynomial Commitments
Polynomial commitments serve as an important component of several modern noninteractive proof systems.

#### Vector Commitments
Given a vector $\vec{v} = [v_1, v_2, \dots, v_n]$ we can create a succinct commitment to the whole vector $\vec{v}$ by interpolating a degree-$(n-1)$ polynomial $\varf$ such that $\varf(i) = v_i$. This enables provers to commit to a vector of field elements such that openings prove the value at a specific index. Vector commitments can in turn be used as a building block for [Verkle Trees](https://vitalik.eth.limo/general/2021/06/18/verkle.html), a variant of Merkle trees with extremely small proof sizes.

#### zkSNARKs
Polynomial commitment schemes are a core primitive in zkSNARKs such as {{< cite PLONK >}} and {{< cite Groth16 >}}. These general-purpose zero knowledge proofs encode computations as polynomial identity constraints and then use openings of polynomial commitments at random points to verify polynomial equality. The power of polynomial commitments here comes from the fact that two distinct degree-$t$ polynomials over a field of order $q > t$ may intersect in at most $t$ points, and thus agree at a random point with probability at most $\frac{t}{q}$.

---

## The KZG Construction
### Bilinear Pairings
KZG is a pairing-based protocol, meaning it works over a group with an efficiently computable bilinear pairing. 

Let $\cgroup_1$, $\cgroup_2$ and $\cgroup_T$ be abelian groups of prime order $\varq$. $\cgroup_1$ and $\cgroup_2$ will be written additively while $\cgroup_T$  will be multiplicative.
A bilinear pairing is an efficiently computable function $e: \cgroup_1 \times \cgroup_2 \rightarrow \cgroup_T$ such that 

$$
e(a\cdot x, b \cdot y) = e(x, y)^{ab}
$$
for all $x \in \cgroup_1, y \in \cgroup_2$. We generally assume that $e$ is non-degenerate, meaning that $e$ is not the trivial function mapping all pairs to $1_{\cgroup_T}$. 

The KZG commitment scheme was originally presented in the "type-1" setting, where $\cgroup_1 = \cgroup_2$. In practice, and in our presentation here, the scheme is generally implemented in the "type-3" setting, where $\cgroup_1 \neq \cgroup_2$ in the sense that there is no efficiently computable isomorphism between the two groups.

Common choices of pairing-friendly groups are the Barreto-Naehrig {{< cite "BN05" >}} and Barreto-Lynn-Scott curves {{< cite "BLS12-381" >}}.

### Trusted Setup
KZG requires a setup procedure to be carried out by a trusted third party. In practice this setup is often performed as a distributed multiparty computation, such that as long as one participant is honest the setup is secure.

The public parameters are $\cgroup_1$, $\cgroup_2$ and $\cgroup_T$, with generators $g_1 \in \cgroup_1, g_2 \in \cgroup_2$ and 
$e: \cgroup_1 \times \cgroup_2 \rightarrow \cgroup_T$ a nondegenerate bilinear pairing. Let $t \in \mathbb{N}$ also be publicly chosen; $t$ will be the maximum degree of committed polynomials.

The setup procedure then computes 

{{< rawhtml >}}
$$
\begin{align*}
\mathbf{SK} &= \sample{\alpha}\\
\mathbf{PK} &= \langle g_1, \alpha\cdot g_1, \alpha^2 \cdot g_1, \dots, \alpha^t \cdot g_1, g_2, \alpha \cdot g_2\rangle
\end{align*}
$$
{{< /rawhtml >}}
and publishes $\mathbf{PK}$. 

{{< hint info >}}
This pre-generated $\mathbf{PK}$ is sometimes known as a "Structured Reference String"
{{< /hint >}}


{{< hint danger >}}
$\mathbf{SK}$ __must be kept secret and destroyed__.

If a party knows the value $\alpha$ then they can forge openings of a commitment to $\varf$ at $\varx_0$ with putative value $\vary$ by computing

$\varw = (\varf(\alpha) / (\alpha - \varx_0) - \vary) \cdot g_1$
{{< /hint >}}


### Commit
A commitment to a polynomial $\varf$ is simply the group element $\varf(\alpha) \cdot g_1$. Because the prover does not know $\alpha$ and thus cannot compute 
the value $\varf(\alpha)$ directly, the evaluation is carried out in the exponent as follows:

{{< rawhtml >}}
Given $\mathbf{PK}$, to form a commitment to $\varf \in \mathbb{Z}_q[X]$ with $\varf(X) = \sum_{i = 0}^{t} c_i X^i$, compute and publish 
$$\varC = \sum_{i=0}^t c_i \cdot (\alpha^i \cdot g_1)$$
{{< /rawhtml >}}

### Open
For any degree-$t$ polynomial $\varf$, the polynomial $(\varf(X) - \varf(x_0))$ has a root at $x_0$ and thus can be factored as $$\varf(X) - \varf(x_0) = (X - \varx_0) \varf_{\varx_0}(X)$$
for some degree-$(t-1)$ polynomial $\varf_{\varx_0}$. The witness to an opening of $\varf$ at 
$\varx_0$ is then $\varw = \varf_{\varx_0}(\alpha) \cdot g_1$.



More concretely, to open a commitment $\varC$ to polynomial $\varf$ at point $\varx_0$, compute the quotient
$$\varf_{\varx_0}(X) = \frac{\varf(X) - \varf(\varx_0)}{X - \varx_0} = \sum_{i = 0}^{t} c_i X^i$$
and compute the witness as 
$$\varw = \varf_{\varx_0}(\alpha)\cdot g_1 = \sum_{i=0}^t c_i \cdot (\alpha^i \cdot g_1)$$

Reveal $\langle \varx_0, \varf(\varx_0), \varw \rangle$.

### Verify
Verification of an opening of $\varf$ at $\varx_0$ with witness $\varw = \varf_{\varx_0}(\alpha)\cdot g_1$ amounts to checking in the exponent that 
$\varf(\alpha) \equalQ (X- \varx_0)(\alpha) \cdot \varf_{\varx_0}(\alpha) + \varf(\varx_0)$. Since the prover does not know the value of $\alpha$,
we can think of the check as happening at a random point and thus with high probability succeeds only if the polynomial identity 
$\varf(X) \equalQ (X- \varx_0) \cdot \varf_{\varx_0}(X) + \varf(\varx_0)$ holds at all points.

Concretely, to verify an evaluation $\langle \varx_0, \varf(\varx_0), \varw \rangle$ on commitment $\varC$, 
check $$e(\varC, g_2) \equalQ e(\varw, \alpha \cdot g_2 - \varx_0 \cdot g_2) \cdot e(g_1, g_2)^{\varf(\varx_0)}$$

To see that the scheme is _complete_ in the sense that every correctly computed opening passes verification, we compute

{{< rawhtml >}}
$$
\begin{align*}
&e(\varw, \alpha \cdot g_2 - \varx_0 \cdot g_2) \cdot e(g_1, g_2)^{\varf(\varx_0)}\\
&= e(\varf_{\varx_0}(\alpha)\cdot g_1, (\alpha - \varx_0)\cdot g_2) \cdot e(g_1, g_2)^{\varf(\varx_0)}\\
&= e(g_1, g_2)^{\varf_{\varx_0}(\alpha)\cdot (\alpha - \varx_0) + \varf(\varx_0)}\\
&= e(g_1, g_2)^{\frac{\varf(\alpha) - \varf(\varx_0)}{\alpha - \varx_0} \cdot (\alpha - \varx_0) + \varf(\varx_0)}\\
&= e(g_1, g_2)^{\varf(\alpha)}\\
&= e(\varf(\alpha)\cdot g_1, g_2)\\
&= e(\varC, g_2)
\end{align*}
$$
{{< /rawhtml >}}

---

## Hardness Assumptions

### Discrete Logarithm
The _hiding_ property of KZG commitments relies on the hardness of the discrete logarithm in the group $\cgroup_1$.

### $t$-Strong Diffie Hellman 

The _binding_ property requires the $t$-Strong Diffie Hellman ($t$-SDH) assumption: Given $g, \alpha \cdot g, \dots, \alpha^t \cdot g$ it is hard to find a pair $\langle c, \frac{1}{\alpha + c} \cdot g \rangle$. This is a slightly stronger form of the $t$-Polynomial Diffie Hellman assumption that, given $g, \alpha \cdot g, \dots, \alpha^t \cdot g$, it is hard to find $\alpha^{t+1}\cdot g$.

$t$-SDH implies the computational Diffie Hellman assumption, as well as the hardness of the discrete logarithm problem.

---

## KZG Variants
### Indistinguishable and Unconditionally hiding ("Pedersen") variants
The KZG commitment scheme as presented above can be viewed as a generalization of the Discrete Log commitment $\varC(\varx) =  \varx \cdot g$. This scheme is binding and is hiding against a computationally-bounded adversary when $\varx$ is chosen uniformly at random, but lacks _indistinguishability_: if $\varx$ is drawn from a small set known to the adversary then the adversary can distinguish which of the elements was committed. For example, if Alice wants to commit to a single bit $b \in \\{0, 1\\}$ then she will either publish $0 \cdot g = 0$ or $1 \cdot g = g$ and Bob can clearly learn which bit was committed. [Pedersen commitments]({{< ref "./pedersen.md" >}}) solve this problem by adding a random masking value $r$ and an extra random public generator $h$ such that $\varC_r(\varx) = \varx \cdot g + r \cdot h$. This commitment is now _indistinguishable_ and _unconditionally hiding_: for any value $y$ there exists some unique $s$ such that $C_r(x) = x \cdot g + r\cdot h = y \cdot g + s \cdot h = C_s(y)$, so even a computationally-unbounded adversary cannot learn $x$ from $C_r(x)$, if $r$ is unknown.

In a similar vein, let $\varf(\varx)$ be a degree-$t$ polynomial.
* An adversary capable of solving the Discrete Log problem can find the discrete log of $\varC(\varf) = {\varf(\alpha)} \cdot g_1$ and acquire $\langle \alpha, \varf(\alpha)\rangle$. Thus the scheme is only computationally hiding.
* If $\varf(\varx_0)$ is chosen from a small set, say $\\{0,1\\}$, then given $t$ openings an adversary can "guess and check" the value of $\varf(\varx_0)$. Thus the scheme does not possess indistinguishability.

There are two common approaches to resolve these issues:
1. Choose one extra point on $f$ at random: if you want to commit to $t$ "useful" points, instead commit to $t+1$ points, where the last point is chosen uniformly at random. Then, any subset of the $t$ useful points may be revealed without breaking indistinguishability.
2. Use the "Pedersen Variant": let $h_1$ be an agreed-upon random generator of $\cgroup_1$. To commit to a degree-$t$ polynomial $\varf$, choose a random degree-$t$ polynomial $\hat{\varf}$ and publish ${\varf(\alpha)}\cdot g_1 +  {\hat{\varf}(\alpha)} \cdot h_1$. To open at a point $\varx_0$, let 
$$
w = {\frac{\varf(\alpha) - \varf(\varx_0)}{\alpha - \varx_0}}\cdot g_1 + {\frac{\hat{\varf}(\alpha) - \hat{\varf}(\varx_0)}{\alpha - \varx_0}}\cdot h_1
$$
reveal $\varf(\varx_0), \hat{\varf}(\varx_0), wI$

{{< hint danger >}}
**Security note:** $h_1$ must be chosen independently from $g_1$, i.e., no one may know the discrete log of $h_1$ with respect to $g_1$. If Alice knows $s$ such that $h_1 = s \cdot g_1$ then the commitment scheme fails to be binding:

If $h_1 = s \cdot g_1$ then 
$$\begin{align*}
\varC_r(x) &= x \cdot g_1 + r \cdot h_1\\\\
&= x\cdot g_1 +  {(s \cdot r)}\cdot g_1\\\\
&= (x + (y - x) + s(r - \frac{y - x}{s}))\cdot g_1\\\\
&= y\cdot g_1 + (r - \frac{y - x}{s})\cdot h_1\\\\
&= C_{r-\frac{(y - x)}{s}}(y)
\end{align*}
$$
{{< /hint >}}

---

## Security Pitfalls
* __Small-order elements:__ Pairing based cryptography often involves elliptic curve groups of composite order. Any elliptic curve points accepted from untrusted parties must be verified to reside in the proper large prime-order subgroup.
* __Polynomial interpolation:__ If $t+1$ points on a degree-$t$ polynomial are revealed, then all further points can be recovered.
* __Trusted setup:__ If using a trusted setup, $\mathbf{SK}$ must be generated with strong randomness and erased immediately after generating $\mathbf{PK}$.
