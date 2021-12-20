---
bookCollapseSection: false
weight: 3
title: "Number is product of two primes"
needsVariableResetButton: true
summary: "Prove that a number is the product of two primes. We show two proofs of this: one for generic primes, and another, more efficient, when primes are congruent with 3 modulo 4."
---
## Product of primes
One way to prove that some number $\varN$ is the product of two primes $\varN = \varp \varq$ is by showing that:
 - $\varN$ is square-free
 - $\varN$ only has two divisors

If we only showed that $\varN$ is square-free, then it could be of the form $\varN = \varp \varq \varr$.
On the other hand, if we only proved that $\varN$ has two divisors, it could be of the form $\varN = \varp^2\varq$.

When the primes are congruent with $3 \\; \mathsf{mod}\\; 4$, we can show that $\varN$ is the product of two primes much more efficiently with the [Paillier-Blum modulus]({{< relref "/docs/zkdocs/zero-knowledge-protocols/product-primes/paillier_blum_modulus" >}}) proof.

{{<section>}}
