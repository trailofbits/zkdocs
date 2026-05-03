---
weight: 2
bookFlatSection: true
title: "Number is square-free"
summary: "Proves that a number is square-free."
needsVariableResetButton: true
references: ["GRSB19"]
---
# Number is square-free

To prove that a number is the product of two distinct primes, the first step is showing that the number is free of squares, i.e., that $\varN$ is
not of the form $\varN = p^2 r$.

This proof shows in zero-knowledge --without revealing the factors of the number-- that an element belongs to
the set $$\sqfree = \\{\varN \in \naturals : \text{there is no prime }p \text{ such that }p^2\|N \\}.$$

The protocol uses the fact that an honest prover can calculate $\varN$-th roots of arbitrary numbers in $\z{\varN}$.

{{< hint info >}}
**Goal:**
$\varprover$ convinces $\varverifier$ that $\varN$ is square-free without revealing its factorization.
{{< /hint >}}

 * __Public input:__ $\varN\in\naturals$
 * __Private input:__ $\varprover$ knows the factorization of $\varN = \varp\varq$
 * __Security parameters:__ $\alpha, \kappa, m = \ceil{\kappa/ \log_2 \alpha}$

The protocol uses the constant $\displaystyle\Pi_\alpha = \prod_\underset{p\text{ prime}}{p<\alpha} p$, the product of the primes smaller than $\alpha$.

We assume that both parties agree with the security parameters $\alpha, \kappa$ and $m$ prior to the execution of the protocol.

## Interactive protocol (HVZK)
{{< hint danger >}}
**Security note:** The protocol is zero-knowledge (does not reveal the factorization of $\varN$) only when the verifier is honest and generates each $\rhovar_i$ randomly. If the verifier chooses these values maliciously they can recover the factorization of $\varN$. If your attacker model takes this into consideration, use the non-interactive version. More details on [Using HVZK in the wrong context](../../../security-of-zkps/when-to-use-hvzk).
{{< /hint >}}

{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \bobwork{\sampleN{\rhovar_i}{\varN}, \text{ for }i=1,\ldots,m}
 \bobalice{}{\{\rhovar_i\}_{i=1}^m}{}
 \alicework{\sigmavar_i = (\rhovar_i)^{\varN^{-1}\mod \varphi(\varN)} \mod \varN,}
 \alicework{\text{ for }i=1,\ldots,m}
 \alicebob{}{\{\sigmavar_i\}_{i=1}^m}{}
 \bobwork{\varN \gQ 1}
 \bobwork{\gcd(\varN, \Pi_\alpha) \equalQ 1}
 \bobwork{\sigmavar_i \mod \varN \gQ 0,}
 \bobseparator
 \bobwork{\rhovar_i \equalQ \sigmavar_i^\varN \mod \varN,}
 \bobwork{\text{ for }i=1,\ldots,m}
 \end{array}
 $$
{{< /rawhtml >}}


-----

## Non-interactive protocol
The non-interactive version of the protocol replaces the verifier-side sampling of $\rhovar_i$ elements and generates them using the Fiat-Shamir transformation.
The participants only exchange one message, and there is no risk of leaking the factorization of $\varN$ using maliciously chosen $\rhovar_i$ values.
{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\rhovar_i = \mathsf{gen}_\rhovar(\z{\varN}, \{\varN,i\})}
 \alicework{\sigmavar_i = (\rhovar_i)^{\varN^{-1}\mod \varphi(\varN)} \mod \varN,}
 \alicework{\text{ for }i=1,\ldots,m}
 \alicebob{}{\{\sigmavar_i\}_{i=1}^m}{}
 \bobwork{\varN \gQ 1}
 \bobwork{\gcd(\varN, \Pi_\alpha) \equalQ 1}
 \bobwork{\sigmavar_i \mod \varN \gQ 0 ,}
 \bobseparator
 \bobwork{\rhovar_i = \mathsf{gen}_\rhovar(\z{\varN}, \{\varN,i\})}
 \bobwork{\rhovar_i \equalQ \sigmavar_i^\varN \mod \varN,}
 \bobwork{\text{ for }i=1,\ldots,m}
 \end{array}
 $$
{{< /rawhtml >}}

## Security pitfalls
 * **Verifier input validation:** Each of the items above the dotted line for the $\varverifier$ is essential to the security of the protocol. If any of these checks are missing or insufficient it is likely a severe security issue.
 * __Using the interactive protocol in a malicious verifier context:__ high severity issue which allows factoring $\varN$; see [Using HVZK in the wrong context]({{< relref "/docs/zkdocs/security-of-zkps/when-to-use-hvzk" >}}).
 * **Sampling $\rhovar_i$ values uniformly and not using honest generation:** high severity issue that allows $\varprover$ to forge proofs by sampling $\sampleSet{\sigmavar_i}{\z\varN}$ and setting $\rhovar_i = \sigmavar_i^\varN\mod \varN$.
 * __Verifier trusting prover on the non-interactive protocol:__
   * $\varverifier$ uses $\rhovar_i$ values provided by $\varprover$ instead of generating them: this is a high severity issue since the prover can trivially forge proofs (e.g., by sending $\rhovar_i=1, \sigmavar_i=1$).
   * $\varverifier$ does not validate $\sigmavar_i$ as valid values modulo $\varN$ (between 0 and $\varN -1$): this allows replaying the same proof with *different* values with $\sigmavar_i + k\varN$.
   * $\varverifier$ does not compare the number of received $\sigmavar_i$ with $m$: this can lead to the prover bypassing proof verification by sending an empty list, or fewer than $m$ elements.
 * __Replay attacks:__ After a non-interactive proof is public, it will always be valid and anyone could pretend to know how to prove the original statement. To prevent this, consider adding additional information to the generation of $\rhovar_i$ values such as an ID of both the prover and the verifier, and a timestamp. The verifier must check the validity of these values when they verify the proof.


## Auxiliary functions - $\mathsf{gen}_\rhovar$
- Generate the $\rhovar_i$ values with a standard [nothing-up-my-sleeve construction]({{< relref "/docs/zkdocs/protocol-primitives/nums.md" >}}) in $\z{\varN}$, use binding parameters $\\{\varN, i\\}$, bitsize of $|\varN|$, and salt $\mathsf{"squarefreeproof"}$. To prevent replay attacks consider including, in the binding parameters, ID's unique to the prover and verifier.



## Choice of security parameters $\alpha$, $\kappa$

The security of the protocol depends $\alpha$ and $\kappa$ which define the number of witnesses that $\varprover$ has to give $\varverifier$. The protocol achieves a statistical soundness error of $2^{-\kappa}$, which means $\varprover$ cannot cheat except with probability $2^{-\kappa}$.

Parameter $\alpha$ is tunable and controls proving and verification time. Higher $\alpha$ values will generally decrease proving time and verification time, until some limit is reached and the verification time starts increasing.

Recommended values for the security parameters are:
 - $\kappa = 128$
 - $\alpha = 65537, 319567$ can be tried and the best performing one chosen.

These values yield $m$ values of
 - $m(\kappa=128, \alpha=65537) = 8$ and
 - $m(\kappa=128, \alpha=319567) = 7$.


