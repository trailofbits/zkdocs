---
weight: 5
bookFlatSection: true
title: "Using HVZKP in the wrong context"
needsVariableResetButton: true
summary: "Potential attacks when honest verifier zero-knowledge proofs are used in the context of a malicious verifier."
references: ["hoc", "gg21"]
---
## Using HVZKP in the wrong context
Honest verifier zero-knowledge proofs (HVZKP) assume --yes, you guessed it-- an honest verifier! This means that in the presence of malicious verifiers, non-interactive protocols should always be used. These also exchange fewer messages between prover and verifier.

A malicious verifier can employ different attacks depending on the proof system. Here, we will present attacks for the [Short factoring proofs](../../zero-knowledge-protocols/short-factoring-proofs) and the [Two prime divisors proof](../../zero-knowledge-protocols/product-primes/two-prime-divisors).

## The case of the Short-Factoring-Proofs
Recall that in [Short factoring proofs](../../zero-knowledge-protocols/short-factoring-proofs) the prover shows that they know $\varphi(\varN)$ in the style of [Girault's scheme](../../zero-knowledge-protocols/girault-identification).
{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\sampleRange{\varr}{A}}
 \alicework{\varx_i = \varz_i^\varr \mod \varN}
 \alicework{\forb}
 \alicebob{}{\bunch{\varx}}{}
 \bobwork{\sampleRange{\vare}{B}}
 \bobalice{}{\vare}{}
 \alicework{\vary = \varr + (\varN - \varphi(\varN))\cdot \vare \in \naturals}
 \alicebob{}{\vary}{}
 \bobwork{\vary \inQ \range{A}}
 \bobwork{\varx_i \equalQ \varz_i^{\vary- \vare\cdot\varN} \mod \varN \forb}
 \end{array}
 $$
{{< /rawhtml >}}

After the initial commit, the verifier responds with a challenge $e$ supposedly sampled from $\range{B}$. However, being malicious, the verifier choses $\vare=A$, the maximum value that $\varr$ can be. So, that after receiving $\vary = \varr + (\varN - \varphi(\varN))\cdot \vare$, they can compute $\varN - \vary//\vare$ which will reveal $\varphi(\varN)$.


## The case of the Two-Prime-Divisor proof
In the [Two prime divisors proof](../../zero-knowledge-protocols/product-primes/two-prime-divisors), the prover has no way of checking if the verifier is trying to attack them. Recall the beginning of the protocol that the verifier chooses $\rhovar_i$ values:
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
\end{array}
 $$
{{< /rawhtml >}}

Then, the verifier computes the square-roots of these values! It is known that factoring and computing modular square-roots are equivalent [[HOC] - Fact 3.46].

An attacker can:
-  select random numbers $r_i$
-  send their square $r^2_i \mod \varN$ to the prover,

The prover will compute their square roots, $\sigma_i$ which can be different than $\pm \varr_i$ since there are four different square-roots modulo $\varN = p q$. When $\sigma_i\neq \pm \varr_i$, computing $\gcd (\varN, \sigma_i - r_i)$ will reveal one of the factors of $\varN$. This is because

$\begin{align*}
\sigma_i^2 &\equiv r_i^2 \mod \varN \\\\
 (\sigma_i^2 - r_i^2) &\equiv 0 \mod \varN \\\\
 (\sigma_i - r_i)(\sigma_i + r_i) &\equiv 0 \mod \varN
\end{align*}$





[HOC]: https://cacr.uwaterloo.ca/hac/


