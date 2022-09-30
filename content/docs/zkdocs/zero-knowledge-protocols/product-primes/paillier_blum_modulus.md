---
weight: 6
bookFlatSection: true
title: "Paillier-Blum Modulus"
needsVariableResetButton: true
summary: "An efficient proof that shows a number is the product of two primes congruent with 3 mod 4."
references: ["gg21", "hoc"]
---
# Paillier-Blum Modulus
This protocol allows the prover to show that a public modulus is the product of two $3 \\; \mathsf{mod}\\; 4$ prime numbers. Since *safe-primes* -- primes of the form $p = 2q + 1$ where $q$ is also prime -- are congruent with $3\\; \mathsf{mod}\\; 4$, this proof system can be used in those cases. Similar to the proof for [Product of two primes](../product-of-two-primes), this system also uses the proof of [Square freeness](../square-freeness). This proof system is also much more efficient than the one for the [Product of two primes](../product-of-two-primes).


{{< hint info >}}
**Goal:**
$\varprover$ convinces $\varverifier$ that $\varN$ is the product of two primes congruent with $3\\; \mathsf{mod}\\; 4$, and that $\gcd(\varN, \varphi(\varN)) = 1$, without revealing its factorization.
{{< /hint >}}

 * __Public input:__ modulus $\varN$
 * __Private input:__ $\varprover$ knows $p,q$ such that $\varN = p q$. These are primes congruent with $3\\; \mathsf{mod}\\; 4$.
 * __Security parameters:__ $m$ to achieve a cheating probability of $2^{-m}$

## Interactive protocol
{{< hint danger >}}
**Security note:** The protocol is zero-knowledge (does not reveal the factorization of $\varN$) only when the verifier is honest and randomly generates each $\vary_i$. If the verifier chooses these values maliciously, they can recover the factorization of $\varN$. If your attacker model considers this, use the non-interactive version. More details on [Using HVZKP in the wrong context](../../../security-of-zkps/when-to-use-hvzk).
{{< /hint >}}

{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\sampleN{\varw}{\varN} \text{ with }J(\varw, \varN) = -1}
 \alicebob{}{\varw}{}
 \bobwork{\sampleNs{\vary_i}{\varN} }
 \bobalice{}{\bunch{\vary}}{}
 \alicework{\varz_i = \vary_i^{\varN^{-1} \mod \varphi ( \varN )}}
 \alicework{\varx_i = \sqrt[4]{\vary_i'} \mod \varN}
 \alicework{\text{where } \vary_i' = (-1)^{\vara_i} \varw^{\varb_i} \vary_i \in QR_\varN}
 \alicework{\text{and } \vara_i, \varb_i \in \{0, 1\}\enspace\;\;}
 \alicework{\text{ for }i=1,\ldots,m}
 \alicebob{}{\bunchi{(\varx_i, \vara_i, \varb_i), \varz_i}}{}
 \bobwork{\varN \gQ 1}
 \bobwork{\varN \mod 2 \equalQ 1}
 \bobwork{\mathsf{isPrime}(\varN) \equalQ false}
 \bobwork{ \varz_i ^\varN \equalQ \vary_i \mod \varN  }
 \bobwork{\vara_i, \varb_i \inQ \{0, 1\}  }
 \bobwork{ \varx_i ^4 \equalQ (-1)^{\vara_i} \varw^{\varb_i} \vary_i \mod \varN  }
 \bobwork{\text{ for }i=1,\ldots,m}
 \end{array}
 $$
{{< /rawhtml >}}

-----

## Non-interactive protocol
The non-interactive version of the protocol replaces the verifier-side sampling of $\vary_i$ and generates them using the $\mathsf{gen}_\vary$ [function](#honest-parameter-generation-mathsfgen_vary).
The participants only exchange one message and this proof can be used in the context of malicious verifiers.

{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\sampleN{\varw}{\varN} \text{ with }J(\varw, \varN) = -1}
 \alicework{\vary_i = \mathsf{gen}_\vary(\zns{\varN}, \{\varN, \varw, i\})}
 \alicework{\varz_i = \vary_i^{\varN^{-1} \mod \varphi ( \varN )}}
 \alicework{\varx_i = \sqrt[4]{\vary_i'} \mod \varN}
 \alicework{\text{where } \vary_i' = (-1)^{\vara_i} \varw^{\varb_i} \vary_i \in QR_\varN}
 \alicework{\text{and } \vara_i, \varb_i \in \{0, 1\}\enspace\;\;}
 \alicework{\text{ for }i=1,\ldots,m}
 \alicebob{}{\varw, \bunchi{(\varx_i, \vara_i, \varb_i), \varz_i}}{}
 \bobwork{\varN \gQ 1}
 \bobwork{\varN \mod 2 \equalQ 1}
 \bobwork{\mathsf{isPrime}(\varN) \equalQ false}
 \bobwork{J(\varw, \varN) \equalQ -1}
 \bobwork{\vary_i = \mathsf{gen}_\vary(\zns{\varN}, \{\varN, \varw, i\})}
 \bobwork{ \varz_i ^\varN \equalQ \vary_i \mod \varN  }
 \bobwork{\vara_i, \varb_i \inQ \{0, 1\}  }
 \bobwork{ \varx_i ^4 \equalQ (-1)^{\vara_i} \varw^{\varb_i} \vary_i \mod \varN  }
 \bobwork{\text{ for }i=1,\ldots,m}
 \end{array}
 $$
{{< /rawhtml >}}

## Security pitfalls
 * __Using the interactive protocol in a malicious verifier context:__ high severity issue which allows factoring the modulus $\varN$; see [Using HVZKP in the wrong context]({{< relref "/docs/zkdocs/security-of-zkps/when-to-use-hvzk" >}}).
 * **Not checking that $\varN \mod 2 \equalQ 1$ and $\varN \gQ 1$ before computing the Jacobi symbol $J(\varw, \varN)$**: the Jacobi symbol is only defined when $\varN$ is an odd positive integer. Failing to verify this might lead to a program panic, as in the [Go library](https://cs.opensource.google/go/go/+/refs/tags/go1.17.5:src/math/big/int.go;l=838).
 * __Verifier trusting prover on the non-interactive protocol:__
   - $\varverifier$ does not check that $J(\varw, \varN) = -1$: high severity issue since this allows $\varprover$ to submit a proof forgery with $\varw = 0$ and $\varx_i = 0$.
   * $\varverifier$ uses $\vary_i$ values provided by $\varprover$ instead of generating them: high severity issue since the prover can trivially forge proofs (e.g., by sending $\rhovar_i=1, \sigmavar_i=1$).
   * $\varverifier$ does not validate $\varw, \varz_i, \varx_i$ as valid values modulo $\varN$ (between 0 and $\varN -1)$: this allows replaying the *same* proof with *different* values with $\varz_i + k\varN$.
   * $\varverifier$ does not compare the number of received tuples $(\varx_i, \vara_i, \varb_i), \varz_i$ with $m$: this can lead to the prover bypassing proof verification by sending an empty list, or fewer than $m$ elements.
 * **Reusing the same $\varw$ for further proofs of $\varN$:** reusing the same $\varw$ value means that for a fixed $\varN$, the $\vary_i$ values will be the same; if the algorithm that computes the fourth-root is probabilistic, it can reveal different fourth-roots for the same value $\vary_i'$, allowing an attacker to factor $\varN$. Solve this in one of two ways: either add a fresh value to the parameters of $\mathsf{gen}_\vary$ and share it with $\varverifier$, or ensure that you deterministically compute the fourth-roots.
 * __Replay attacks:__ After a non-interactive proof is public, it will always be valid, and anyone could pretend to know how to prove the original statement. To prevent this, consider adding additional information to the generation of $\vary_i$ values, such as an ID of the prover and the verifier and a timestamp. The verifier must check the validity of these values when they verify the proof.

## Choice of security parameters
To achieve a safe security level, choose:
 - $|\varN| = 2048$
 - $m = 80$.

## Honest parameter generation $\mathsf{gen}_\vary$
 - Generate the $\vary_i$ values with a standard [nothing-up-my-sleeve construction](../../../protocol-primitives/nums) in $\zns{\varN}$, use binding parameters $\\{\varN, \varw, i\\}$, bitsize of $|\varN|$, and salt $\mathsf{"paillierblumproof"}$. To prevent replay attacks consider including, in the binding parameters, ID's unique to the prover and verifier.


## Auxiliary algorithms
 - The function $J(\varw, \varN)$ is the [Jacobi symbol](https://en.wikipedia.org/wiki/Jacobi_symbol) of $\varw$ modulo $\varN$. To implement it use Algorithm 2.149 of the [Handbook of Applied Cryptography](https://cacr.uwaterloo.ca/hac/).
 - Compute modular fourth-roots using [[HOC] - Fact 2.160]: if $x$ is a quadratic residue modulo $\varN = p q$ (and $p,q$ are $3\mod 4$) then $\sqrt[2]{x} = x^{(\varphi + 4)/8} \mod \varN$, where $\varphi = (p-1)\cdot(q-1)$. Applying this computation twice yields the modular fourth-root.
 - The prover has to check if an element is a quadratic residue modulo $\varN$. To do this, use [[HOC] - Fact 2.137]: $x$ is a quadratic residue modulo $\varN = pq$ if and only if it is a quadratic residue modulo $p$ and modulo $q$. A number $x$ is a quadratic residue modulo a prime $p$, if $x^{(p-1)/2} = 1$. Thus, $x$ is a quadratic residue modulo $\varN = pq$ if and only if $x^{(p-1)/2} = 1$ and $x^{(q-1)/2} = 1$.
 - Most libraries will have a primality testing routine for the $\mathsf{isPrime}$ function.



[HOC]: https://cacr.uwaterloo.ca/hac/
