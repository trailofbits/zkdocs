---
weight: 6
bookFlatSection: true
title: "Short factoring proofs"
summary: "Proof of knowledge of the factorization of an integer."
needsVariableResetButton: true
references: ["PS00"]
---
# Short factoring proofs
This system proves knowledge of the factorization of an integer $\varN$. Unlike [Product of two primes]({{< relref "/docs/zkdocs/zero-knowledge-protocols/product-primes/product-of-two-primes.md" >}}), this is not a proof that $\varN$ has two prime factors, only that the prover knows its factorization!
This protocol is not easy to implement securely, so we recommend special attention to sections [Choice of security parameters](#choice-of-security-parameters) and [Security pitfalls](#security-pitfalls).

{{< hint info >}}
**Goal:**
$\varprover$ convinces $\varverifier$ that they know the factorization of $\varN$ without revealing it.
{{< /hint >}}
 * __Public input:__ An integer $\varN$
 * __Private input:__ $\varprover$ knows the factorization of $\varN =\prod_i p_i^{e_i}$
 * __Security parameters:__ The parameters $A, B, \ell, m$, and $K$ all depend on $k$ and the bit-size of $\varN$.



### Vanilla interactive protocol
{{< hint danger >}}
**Note:**
This version assumes that $\varverifier$ is honest. A dishonest one would send $\vare$ with similar size to $B$ and factor $\varN$ using [this attack](#security-pitfalls).
{{< /hint >}}


The following diagram shows one of the protocol's $\ell$ iterations.
{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\sampleRange{\varr}{A}}
 \alicework{\varz_i = \mathsf{gen}_\varz(\zns{\varN}, \{\varN, i\})}
 \alicework{\varx_i = \varz_i^\varr \mod \varN}
 \alicework{\forb}
 \alicebob{}{\bunch{\varx}, \bunch{\varz}}{}
 \bobwork{\sampleRange{\vare}{B}}
 \bobalice{}{\vare}{}
 \alicework{\vary = \varr + (\varN - \varphi(\varN))\cdot \vare \in \naturals}
 \alicebob{}{\vary}{}
 \bobwork{\vary \inQ \range{A}}
 \bobwork{\varN \gQ 1}
 \bobseparator
 \bobwork{\varx_i \equalQ \varz_i^{\vary- \vare\cdot\varN} \mod \varN \forb}
 \end{array}
 $$

{{< /rawhtml >}}

-----

### Improved interactive protocol
{{< hint danger >}}
**Note:**
This version assumes that $\varverifier$ is honest. A dishonest one would send $\vare$ with similar size to $B$ and factor $\varN$ using [this attack](#security-pitfalls).
{{< /hint >}}

{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\sampleRange{\varr}{A}}
 \alicework{\varz_i = \mathsf{gen}_\varz(\zns{\varN}, \{\varN, i\})}
 \alicework{\varX = \hash{\bunchi{\varz_i^\varr \mod \varN}}}
 \alicebob{}{\varX, \bunch{\varz}}{}
 \bobwork{\sampleRange{\vare}{B}}
 \bobalice{}{\vare}{}
 \alicework{\vary = \varr + (\varN - \varphi(\varN))\cdot \vare \in \naturals}
 \alicebob{}{\vary}{}
 \bobwork{\vary \inQ \range{A}}
 \bobwork{\varN \gQ 1}
 \bobseparator
 \bobwork{\varX \equalQ \hash{\bunchi{\varz_i^{\vary- \vare\cdot\varN} \mod \varN}}}
 \end{array}
 $$

{{< /rawhtml >}}

-----

### Non-interactive protocol
We use the Fiat-Shamir heuristic, and the prover creates $\vare$ using a $|B|$-bit sized hash function $\hashbit{\cdot}{|B|}$ and past public values.
{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\sampleRange{\varr}{A}}
 \alicework{\varz_i = \mathsf{gen}_\varz(\zns{\varN}, \{\varN, i\})}
 \alicework{\varX = \hash{\bunchi{\varz_i^\varr \mod \varN}}}
 \alicework{\vare = \hashbit{\varN, \bunch{\varz},\varX}{|B|}}
 \alicework{\vary = \varr + (\varN - \varphi(\varN))\cdot \vare \in \naturals}
 \alicebob{}{\vare, \vary, \varX}{}
 \bobwork{\vary \inQ \range{A}}
 \bobwork{\varN \gQ 1}
 \bobseparator
 \bobwork{\varz_i = \mathsf{gen}_\varz(\zns{\varN}, \{\varN, i\})}
 \bobwork{\vare = \hashbit{\varN, \bunch{\varz},\varX}{|B|}}
 \bobwork{\varX \equalQ \hash{\bunchi{\varz_i^{\vary- \vare\cdot\varN} \mod \varN}}}
 \end{array}
 $$

{{< /rawhtml >}}

## Choice of security parameters
We detail the choice of security parameters based on the bit-size of $\varN$:
 * $k$ -- __the__ security parameter; the cheating probability of the protocol is $2^{-k}$. Choose 80, 128 or 256.
 * $B$ and $\ell$ -- the size of the challenge space and number of iterations; $B$ and $\ell$ should satisfy $\ell\cdot \log B = \theta(k)$, so chose $\ell=1$ and $B=2^k$ in the non-interactive protocol.
 * $A$ -- the size of the commit space; $A$ must be smaller than $\varN$ and satisfy $(\varN - \varphi(\varN) ) \ell B \ll A < \varN$. If you are sure that public moduli are of a fixed size --like $|\varN| = 2048$-- chose $A=2^{|\varN|}$. If the public modulus can have smaller bit-size but never below $|\varN| - 4$, chose $A=2^{|\varN| - 4}$.

## Security pitfalls
- **Verifier input validation:** Each of the items above the dotted line for the $\varverifier$ is essential to the security of the protocol. If any of these checks are missing or insufficient it is likely a severe security issue.
- The two interactive protocols assume an honest verifier. To use this protocol in the context of malicious verifiers, use the non-interactive version. See [Using HVZKP in the wrong context]({{< relref "/docs/zkdocs/security-of-zkps/when-to-use-hvzk" >}}).
- Implementers of this proof system need to carefully choose the security parameters since failing to do it correctly leaks $\varphi(\varN)$:
{{< hint danger >}}
Consider the example where $\varN$ is 2048 bits, the product of two 1024 bit primes. $\varprover$ computes over $\naturals$ the value $$\vary = \varr + (\varN - \varphi(\varN))\cdot \vare$$ where $\sampleRange{\varr}{A}$ and $\sampleRange{\vare}{B}$. If $A$ and $B$ have similar size then $\frac{r}{e}$ will be very small and $\varverifier$ can compute $$-\left\lfloor \frac{\vary}{e} \right \rfloor + \varN = \left\lfloor \frac{r}{e} \right \rfloor + \varphi (\varN) \approx \varphi(\varN) \enspace ,$$ recovering $\varphi(\varN)$.

A variant of this attack happens when $|\frac{r}{e}| < \frac{|\varphi(\varN)|}{4}$: in this case, a quarter of the least significant bits of $\varphi(\varN)$ are unknown, but this still allows an attacker to factor $\varN$ using Coppersmith's attack.
{{< /hint >}}

- __Using high security parameters with small modulus allows proof forgery:__ the security parameters $A=2^{2048}$, $B=2^{128}$ are configured to be used with modulus of size 2048 but the verification routine does not check the size of the modulus. A dishonest prover can send a proof of knowledge for a 1024 modulus and $$\vary = \varN \cdot \vare$$ will still be in the set $\range{A}$ since $\varN\cdot \vare < 2^{1025 + 129} < 2^{2048} = A$.
- __Verifier trusting prover on the non-interactive protocol:__ The $\varverifier$ does not generate the $\varz_i$ values and parses them from the proof without proper validation. The $\varz_i$ values should be generated as described [here](#honest-parameter-generation-mathsfgen_varz). Failing to do so, and not checking that $\varz_i\in\zns{\varN}$, $\varz_i \mod \varN \not\in \\{ -1 , 0, 1\\}$ leads to trivial proofs: when $\varz_i \equiv -1, 0, 1 \mod \varN$, all proofs are valid as long as $y\in \range{A}$.
 - **Weak Fiat-Shamir transformation:** In the non-interactive protocol, it is a common mistake to not include all values in the hash computation $\hashbit{\varN, \bunch{\varz},\varX}{|B|}$. If any of these values are not included in this computation, it could lead to a high severity issue where the prover can forge proofs. See [Fiat-Shamir transformation]({{< relref "docs/zkdocs/protocol-primitives/fiat-shamir" >}}) for more details.
 * __Replay attacks:__ After a non-interactive proof is public, it will always be valid, and anyone could pretend to know how to prove it. To prevent this, consider adding additional information to the generation of the $\varz$ values, such as an ID of the prover and the verifier and a timestamp. The verifier must check the validity of these values when they verify the proof.

## Honest parameter generation $\mathsf{gen}_\varz$
 - Generate the $\varz_i$ values with a standard [nothing-up-my-sleeve construction]({{< relref "/docs/zkdocs/protocol-primitives/nums.md" >}}) in $\zns{\varN}$, use binding parameters $\\{\varN, i\\}$, bitsize of $|\varN|$, and salt $\mathsf{"shortfactoringproofs"}$. To prevent replay attacks consider including, in the binding parameters, ID's unique to the prover and verifier.

## Auxiliary procedures
 - __Hash function $\hashbit{\cdot}{|B|}$:__ this hash function should be domain-separated and have a specific output size of $|B|$-bits. $\mathsf{TupleHash}$ satisfies these restrictions.