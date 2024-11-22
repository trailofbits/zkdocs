---
weight: 3
title: "Two prime divisors"
needsVariableResetButton: true
summary: "Proves that a number has two prime divisors."
references: ["hoc", "jacobi", "AP18", "GRSB19"]
---
## Number has exactly two prime divisors

To prove that a number is the product of two distinct primes, one step is showing that the number only has two prime divisors, i.e., showing that $\varN$ is not of the form $\varN = pqr$.

This protocol allows to prove in zero-knowledge --without revealing its factors-- that a number is in the set $$L_{ppp} = \\{\varN \in \naturals : \varN \text{ is odd and has exactly two distinct prime divisors}\\}.$$


{{< hint info >}}
**Goal:**
$\varprover$ convinces $\varverifier$ that $\varN$ only has two distinct prime divisors without revealing its factorization.
{{< /hint >}}

 * __Public input:__ $\varN\in\naturals$
 * __Private input:__ $\varprover$ knows the factorization of $\varN = \varp\varq$
 * __Security parameters:__ $\kappa, m = \ceil{\kappa \cdot 32 \cdot \ln(2)}$

The protocol works because an honest prover can calculate square-roots of arbitrary numbers in $\z{\varN}$.

Both parties have to agree on the security parameter $\kappa$ prior to the execution of the protocol.

## Interactive protocol
{{< hint danger >}}
**Security note:** The protocol is zero-knowledge (does not reveal the factorization of $\varN$) only when the verifier is honest and generates each $\rhovar_i$ randomly. If the verifier chooses these values maliciously they can recover the factorization of $\varN$. If your attacker model takes this into consideration, use the non-interactive version. More details on [Using HVZKP in the wrong context](../../../security-of-zkps/when-to-use-hvzk).
{{< /hint >}}
{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \bobwork{\sampleSet{\rhovar_i}{J_\varN}, \text{ for }i=1,\ldots,m}
 \bobalice{}{\{\rhovar_i\}_{i=1}^m}{}
 \alicework{\sigmavar_i = \begin{cases}
  \sqrt{\rhovar_i} \mod \varN &\text{ if }\rhovar_i \in QR_\varN \\
  0 &\text{ otherwise}
\end{cases}
}
 \alicework{\text{ for }i=1,\ldots,m}
 \alicebob{}{\{\sigmavar_i\}_{i=1}^m}{}
 \bobwork{\varN \gQ 1}
 \bobwork{\varN \mod 2 \equalQ 1}
 \bobwork{\mathsf{isPrime}(\varN) \equalQ false}
 \bobwork{\mathsf{isPrimePower}(\varN) \equalQ false}
 \bobwork{\gcd(\varN, \Pi_\alpha) \equalQ 1}
 \bobseparator
 \bobwork{|\{\sigmavar_i \text{ for }i=1,\ldots,m| \sigmavar_i \neq 0 \mod \varN\}| \gQ 3m/8}
 \bobwork{\text{if } \sigmavar_i \neq 0 \mod \varN \text{ then }\rhovar_i \equalQ \sigmavar_i^2 \mod \varN}
 \end{array}
 $$
{{< /rawhtml >}}

 In this protocol we need to uniformly sample from the $J_\varN$, the set of integers modulo $\varN$ with Jacobi symbol $1$. The prover also needs to check membership of the $QR_\varN$, the set of quadratic residues modulo $\varN$. Both computations, the Jacobi symbol and quadratic residuosity, can be done [efficiently](#auxiliary-algorithms).


-----

## Non-interactive protocol
The non-interactive version of the protocol replaces the verifier-side sampling of $\rhovar_i$ and generates them using the $\mathsf{gen}_\rhovar$ [function](../square-freeness/#auxiliary-functions---mathsfgen_rhovar).
The participants only exchange one message and this proof can be used in the context of malicious verifiers.
{{< rawhtml >}}
  $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\rhovar_i = \mathsf{gen}_\rhovar(J_\varN, \{\varN,i, F\})}
 \alicework{\sigmavar_i = \begin{cases}
  \sqrt{\rhovar_i} \mod \varN &\text{ if }\rhovar_i \in QR_\varN \\
  0 &\text{ otherwise}
\end{cases}
}
 \alicework{\text{ for }i=1,\ldots,m}
 \alicebob{}{\bunchi{\sigmavar_i}, F}{}
 \bobwork{\varN \gQ 1}
 \bobwork{\varN \mod 2 \equalQ 1}
 \bobwork{\mathsf{isPrime}(\varN) \equalQ false}
 \bobwork{\mathsf{isPrimePower}(\varN) \equalQ false}
 \bobwork{\gcd(\varN, \Pi_\alpha) \equalQ 1}
 \bobseparator
 \bobwork{\rhovar_i = \mathsf{gen}_\rhovar(J_\varN, \{\varN,i, F\})}
 \bobwork{|\{\sigmavar_i \text{ for }i=1,\ldots,m| \sigmavar_i \neq 0 \mod \varN\}| \gQ 3m/8}
 \bobwork{\text{if } \sigmavar_i \neq 0 \mod \varN \text{ then }\rhovar_i \equalQ \sigmavar_i^2 \mod \varN}
 \end{array}
 $$
{{< /rawhtml >}}

## Security pitfalls
 * **Verifier input validation:** Each of the items above the dotted line for the $\varverifier$ is essential to the security of the protocol. If any of these checks are missing or insufficient it is likely a severe security issue.
 * __Using the interactive protocol in a malicious verifier context:__ high severity issue which allows factoring $\varN$; see [Using HVZKP in the wrong context]({{< relref "/docs/zkdocs/security-of-zkps/when-to-use-hvzk" >}}).
 * **Sampling $\rhovar_i$ values uniformly and not using honest generation:** high severity issue, allows $\varprover$ to forge proofs by sampling $\sampleSet{\sigmavar_i}{\z\varN}$ and setting $\rhovar_i = \sigmavar_i^2\mod \varN$.
 * **Failing to include a fresh value in $\mathsf{gen}_\rhovar$:** potential high severity issue since this means that all proofs for the same $\varN$ will have the same $\rhovar_i$; if the prover uses a probabilistic algorithm to compute square-roots, he can leak two different square-roots of the same value, allowing an attacker to factor $\varN$ using the same attack as [Using HVZKP in the wrong context]({{< relref "/docs/zkdocs/security-of-zkps/when-to-use-hvzk#the-case-of-the-two-prime-divisor-proof" >}}). If the square-root algorithm always returns the same square-root for a given $\rhovar_i$ this is not an issue.
 * __Verifier trusting prover on the non-interactive protocol:__
   * $\varverifier$ uses $\rhovar_i$ values provided by $\varprover$ instead of generating them: this is a high severity issue since the prover can trivially forge proofs (e.g., by sending $\rhovar_i=1, \sigmavar_i=1$).
   * $\varverifier$ does not validate $\sigmavar_i$ as valid values modulo $\varN$ (between 0 and $\varN -1)$: this allows replaying the same proof with *different* values with $\sigmavar_i + k\varN$.
   * $\varverifier$ does not compare the number of received $\sigmavar_i$ with $m$: this can lead to the prover bypassing proof verification by sending an empty list, or fewer than $m$ elements.
 * __Replay attacks:__ After a non-interactive proof is public, it will always be valid and anyone could pretend to know how to prove it. To prevent this, consider adding additional information to the hash function such as an ID of both the prover and the verifier and a timestamp.


## Choice of security parameters $\kappa$
The protocol uses the security parameter $\kappa$ to determine the number of exchanged elements.

For different $\kappa$ values, we obtain
 - $m(\kappa=64) = 1420$
 - $m(\kappa=128) = 2840$

## Honest parameter generation $\mathsf{gen}_\rhovar$
- Generate the $\rhovar_i$ values with a standard [nothing-up-my-sleeve construction]({{< relref "/docs/zkdocs/protocol-primitives/nums.md" >}}) in $J_{\varN}$, use binding parameters $\\{\varN, i, F\\}$, bitsize of $|\varN|$, and salt $\mathsf{"twoprimedivisorsproof"}$. To prevent replay attacks consider including, in the binding parameters, ID's unique to the prover and verifier.

## Auxiliary algorithms
- To calculate the [Jacobi symbol](https://en.wikipedia.org/wiki/Jacobi_symbol) use Algorithm 2.149 of the [Handbook of Applied Cryptography](https://cacr.uwaterloo.ca/hac/).


- To calculate square-roots modulo $\varN$ use Algorithm 3.44 of the [Handbook of Applied Cryptography](https://cacr.uwaterloo.ca/hac/).

- To implement $\mathsf{isPrimePower}$, a simple algorithm is [here](https://cstheory.stackexchange.com/questions/2077/how-to-check-if-a-number-is-a-perfect-power-in-polynomial-time) and also explained on page 3 of the [AKS paper](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.532.7639&rep=rep1&type=pdf). Note 3.6 of the [Handbook of Applied Cryptography](https://cacr.uwaterloo.ca/hac/about/chap3.pdf) also has a description.

- Most libraries will have a primality testing routine for the $\mathsf{isPrime}$ function.

