---
weight: 5
title: "Product of two primes"
needsVariableResetButton: true
summary: "Proves that a number is the product of two distinct primes: in parallel, run the square-freeness proof together with the two-prime-divisors proof."
references: ["GRSB19", "jacobi", "AP18", "hoc"]
---
## Number is the product of two primes

To show that a number is the product of two distinct primes, we make use of the previous protocols to show that:
 - $\varN$ is [square-free](../square-freeness)
 - $\varN$ [only has two divisors](../two-prime-divisors)

Formally, we show that $\varN$ is in the set $$L_{pp} = \\{N \in \naturals | N \text{ is odd and is the product of two distinct primes}\\} = L_{ppp} \cap \sqfree.$$
For this, we run the protocols from [Square-free](../square-freeness) and [Two prime divisors](../two-prime-divisors) in parallel for the same $N$.

{{< hint info >}}
**Goal:**
$\varprover$ convinces $\varverifier$ that $\varN$ is the product of two distinct primes without revealing its factorization.
{{< /hint >}}

 * __Public input:__ $\varN\in\naturals$
 * __Private input:__ $\varprover$ knows the factorization of $\varN = \varp\varq$
 * __Security parameters:__ $\alpha, \kappa, m_1 = \ceil{\kappa/ \log_2 \alpha}$, $m_2  = \ceil{\kappa \cdot 32 \cdot \ln(2)}$

We only present the non-interactive version of this protocol, suitable to use in the context of malicious verifiers.

## Non-interactive protocol

{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\rhovar_i = \mathsf{gen}_\rhovar(\z{\varN}, \{\varN,i\})}
 \alicework{\sigmavar_i = (\rhovar_i)^{\varN^{-1}\mod \varphi(\varN)} \mod \varN,}
 \alicework{\text{ for }i=1,\ldots,m_1}
 \aliceseparator
  \alicework{\thetavar_i = \mathsf{gen}_\rhovar(J_\varN, \{\varN, m_1 + i, F\})}
 \alicework{\muvar_i = \begin{cases}
  \sqrt{\thetavar_i} \mod \varN &\text{ if }\thetavar_i \in QR_\varN \\
  0 &\text{ otherwise}
\end{cases}
}
 \alicework{\text{ for }i=1,\ldots,m_2}
 \alicebob{}{\{\sigmavar_i\}_{i=1}^{m_1}, \{\muvar_i\}_{i=1}^{m_2}, F}{}
 \bobwork{\varN \gQ 1}
 \bobwork{\varN \mod 2 \equalQ 1}
 \bobwork{\mathsf{isPrime}(\varN) \equalQ false}
 \bobwork{\mathsf{isPrimePower}(\varN) \equalQ false}
 \bobwork{\gcd(\varN, \Pi_\alpha) \equalQ 1}
 \bobwork{\sigmavar_i \gQ 0,}
 \bobwork{\rhovar_i = \mathsf{gen}_\rhovar(\z{\varN}, \{\varN,i\})}
 \bobwork{\rhovar_i \equalQ \sigmavar_i^\varN \mod \varN,}
 \bobwork{\text{ for }i=1,\ldots,m_1}
\bobseparator
 \bobwork{\thetavar_i = \mathsf{gen}_\rhovar(J_\varN, \{\varN, m_1 + i, F\})}
 \bobwork{|\{\muvar_i \text{ for }i=1,\ldots,m| \muvar_i \neq 0\}| \gQ 3m_2/8}
 \bobwork{\text{if } \muvar_i \neq 0 \text{ then }\thetavar_i \equalQ \muvar_i^2 \mod \varN}
 \bobwork{\text{ for }i=1,\ldots,m_2}
 \end{array}
 $$
{{< /rawhtml >}}

## Security parameters
 - $\kappa = 128$
 - $m_1(\kappa=128, \alpha=65537) = 8$
 - $m_2(\kappa=128) = 2840$

## Security pitfalls
This protocol suffers from the same pitfalls as the [Square-free](../square-freeness#security-pitfalls) and [Two prime divisors](../two-prime-divisors#security-pitfalls) protocols. We include them here for clarity:
 * **Sampling the $\rhovar_i$ or $\thetavar_i$ values uniformly and not using honest generation:** high severity issue that allows $\varprover$ to forge proofs by sampling $\sampleSet{\sigmavar_i,\muvar_i}{\z\varN}$, and setting $\rhovar_i = \sigmavar_i^\varN\mod \varN$ and $\thetavar_i = \muvar_i^2\mod \varN$ .
 * **Failing to include a fresh value in $\mathsf{gen}_\rhovar$ to generate $\thetavar_i$:** potential high severity issue since this means that all proofs for the same $\varN$ will have the same $\thetavar_i$ values; if the prover uses a probabilistic algorithm to compute square-roots, he can leak two different square-roots of the same value, allowing an attacker to factor $\varN$ using the same attack as [Using HVZKP in the wrong context]({{< relref "/docs/zkdocs/security-of-zkps/when-to-use-hvzk#the-case-of-the-two-prime-divisor-proof" >}}). If the square-root algorithm always returns the same square-root for a given $\rhovar_i$ this is not an issue.
 * __Verifier trusting prover on the non-interactive protocol:__
   * $\varverifier$ uses $\rhovar_i$ or $\thetavar_i$ values provided by $\varprover$ instead of generating them: this is a high severity issue since the prover can trivially forge proofs (e.g., by sending $\rhovar_i=1, \sigmavar_i=1$ and $\thetavar_i = 1, \muvar_i=1$).
   * $\varverifier$ does not validate $\sigmavar_i$, $\muvar_i$ as valid values modulo $\varN$ (between 0 and $\varN -1)$: this allows replaying the same proof with *different* values with $\sigmavar_i + k\varN$, $\muvar_i + k'\varN$.
   * $\varverifier$ does not compare the number of received $\sigmavar_i$ with $m_1$ and $\muvar_i$ with $m_2$: this can lead to the prover bypassing proof verification by sending an empty list, or fewer than $m_1$ or $m_2$ elements.
 * __Replay attacks:__ After a non-interactive proof is public, it will always be valid and anyone could pretend to know how to prove the original statement. To prevent this, consider adding additional information to the generation of $\rhovar_i$ and $\thetavar_i$ values such as an ID of both the prover and the verifier, and a timestamp. The verifier must check the validity of these values when they verify the proof.


## Performance considerations
For the desired security level of $\kappa=128$, we need to generate 2840 $\muvar_i$ elements. Because of this, this proof system can be very computationally expensive. If this dramatically affects the performance of your protocol, and your $\varN$ is the product of two primes congruent with $3\mod 4$, we recommend using an alternative proof system, the [Paillier-Blum modulus](../paillier_blum_modulus) proof.

## Honest parameter generation $\mathsf{gen}_\rhovar$
- Generate the $\rhovar_i$ and $\thetavar_i$ values with a standard [nothing-up-my-sleeve construction]({{< relref "/docs/zkdocs/protocol-primitives/nums.md" >}}):
  - $\rhovar_i$ in $\z{\varN}$, use binding parameters $\\{\varN, i\\}$, bitsize of $|\varN|$, and salt $\mathsf{"productoftwoprimesproof"}$.
  - $\thetavar_i$ in $J_{\varN}$, use binding parameters $\\{\varN, i, F\\}$, bitsize of $|\varN|$, and salt $\mathsf{"productoftwoprimesproof"}$. $F$ should be a fresh value unique to the current proof.
- To prevent replay attacks consider including, in the binding parameters, ID's unique to the prover and verifier.

## Auxiliary algorithms
- To calculate the [Jacobi symbol](https://en.wikipedia.org/wiki/Jacobi_symbol) use Algorithm 2.149 of the [Handbook of Applied Cryptography](https://cacr.uwaterloo.ca/hac/).

- To calculate square-roots modulo $\varN$ use Algorithm 3.44 of the [Handbook of Applied Cryptography](https://cacr.uwaterloo.ca/hac/).

- To implement $\mathsf{isPrimePower}$, a simple algorithm is [here](https://cstheory.stackexchange.com/questions/2077/how-to-check-if-a-number-is-a-perfect-power-in-polynomial-time) and also explained on page 3 of the [AKS paper](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.532.7639&rep=rep1&type=pdf). Note 3.6 of the [Handbook of Applied Cryptography](https://cacr.uwaterloo.ca/hac/about/chap3.pdf) also has a description.

- Most libraries will have a primality testing routine for the $\mathsf{isPrime}$ function.