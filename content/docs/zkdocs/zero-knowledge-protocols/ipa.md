---
weight: 7
bookCollapseSection: false
title: "Inner Product Argument"
summary: "Proving the knowledge of vectors for a public inner product"
needsVariableResetButton: true
references: ["Bootleproofs", "Bulletproofs", "Halo"]
---

# Inner Product Argument

## Overview of the Inner Product Argument

The inner product argument was introduced by Bootle et al. {{< cite Bootleproofs >}}, and refined in Bulletproofs {{< cite Bulletproofs >}} and has the following structure.

{{< hint info >}}
**Goal:**
$\varprover$ convinces $\varverifier$ that they know vectors $\veca$ and $\vecb$ such that they have the inner product $\ip{\veca}{\vecb} = \varz$, and they open the commitment $\varC_P = \ip{\veca}{\vecG} + \ip{\vecb}{\vecH}$.
{{< /hint >}}

 * __Public input:__ a cyclic additive group $\cgroup$ of prime order $\varq$ and $2n$ random $\cgroup$ generators $\vecG = (\varG_1,\dots,\varG_n)$ and $\vecH = (\varH_1,\dots,\varH_n)$; vector commitment $\varC_P \in \cgroup$ and scalar $\varz \in \zq$.

 * __Private input:__ $\varprover$ knows vectors $\veca, \vecb \in \zq^n$ that satisfy the vector commitment $\varC_P = \ip{\veca}{\vecG} + \ip{\vecb}{\vecH}$, and have the inner product $\varz = \ip{\veca}{\vecb}$.

## The Inner Product Argument

In this section, we build the inner product argument over four successive versions. But, first, we introduce vector commitments which underlie all versions.

### Background: Vector Commitments

[Pedersen commitments]({{< ref "./../commitments/pedersen.md" >}}) readily generalize from scalars to vectors of $n$ elements.

As before, start with a finite group $\cgroup$ of prime order $q$, where $q$ is suitably large. Select $n$ random generators $\vecG \coloneqq (\varG_1, \varG_2,\dots,\varG_n)$ of $\cgroup$ such that no discrete log relations are known between them. The parameters of the commitment scheme are $\left(\cgroup,\varq,(\varG_1,\dots,\varG_n)\right)$.

To generate a commitment $\varC \in \cgroup$ to a vector $\vecs = (\vars_1,\dots,\vars_n) \in \zq^n$ compute:
$$
C \coloneqq \ip{\vecs}{\vecG} \coloneqq \vars_1\varG_1 + \vars_2\varG_2 +\cdots+ \vars_n\varG_n
$$
To open the commitment, reveal the committed vector $\vecs$.

{{< hint info >}}
**Upgrading to a hiding commitment.** To achieve hiding in addition to binding, select another group element $\varH$ with unknown discrete log relation, and during commitment, sample a new random value $\sample{\vart}$, and generate the commitment as
$$
C(\vecs, \vart) \coloneqq \ip{\vecs}{\vecG} + t\varH \coloneqq \vars_1\varG_1 + \vars_2\varG_2 +\cdots+ \vars_n\varG_n + \vart\varH
$$
and to open the commitment, reveal the committed vector $\vecs$ and the random value $t$.
{{< /hint >}}

### Version 1: A Simple Construction

Let's start with a simple construction where the prover sends a commitment, and then immediately opens the commitment by sending over the secret vectors $\veca$ and $\vecb$ as the proof.

We will construct the commitment as a combination of commitments to vector $\veca$, vector $\vecb$, and their inner product $z = \ip{\veca}{\vecb}$. First, the prover sends a commitment to the vectors:
$$
\varC_P \coloneqq \ip{\veca}{\vecG} + \ip{\vecb}{\vecH}.
$$
Then, the verifier samples a random group element $\sampleCgroup{\varU}$ and sends it the prover. The prover and the verifier independently compute the full commitment
$$
\varC \coloneqq \ip{\veca}{\vecG} + \ip{\vecb}{\vecH} + \ip{\veca}{\vecb} \cdot \varU.
$$
Finally, the prover reveals the vectors $\veca$ and $\vecb$ and the verifier checks that they open the commitment. The resulting interactive protocol looks like the following:

{{< rawhtml >}}
 $$
 \begin{array}{lcl}
 \uwork{\varprover}{\varverifier}
 \alicework{\varC_P \coloneqq \ip{\veca}{\vecG} + \ip{\vecb}{\vecH}}
 \alicebob{}{\varC_P}{}
 \bobworks{\sampleCgroup{\varU}}
 \bobworks{\varC' \coloneqq \varC_P + \varz \cdot \varU}
 \bobalice{}{\varU}{}
 \alicework{\varC \coloneqq \varC_P + \ip{\veca}{\vecb} \cdot \varU}
 \alicebob{}{\veca, \vecb}{}
 \bobwork{\varC' \equalQ \ip{\veca}{\vecG} + \ip{\vecb}{\vecH} + \ip{\veca}{\vecb} \cdot \varU}
 \end{array}
 $$
{{< /rawhtml >}}

Since this always succeeds if $\varz = \ip{\veca}{\vecb}$, this construction has perfect completeness. Furthermore, since $\varC$ is a binding commitment, it has computational soundness.

{{< hint info >}}
**Comparison with [[BCCGP16]](https://eprint.iacr.org/2016/263).**
Rather than three separate commitments $A$, $B$, and $z$ for the vector $\veca$, vector $\vecb$, and their inner product $\ip{\veca}{\vecb}$, following [[BBBPWM18]](https://eprint.iacr.org/2017/1066.pdf), we have one commitment $\varC$ which combines all three.
{{< /hint >}}

{{< hint info >}}
**Comparison with [[BBBPWM18]](https://eprint.iacr.org/2017/1066.pdf).** Rather than let $\varU$ be a fixed constant, it is sampled randomly by the verifier. As noted in Section 3.1 of [[BGH19]](https://eprint.iacr.org/2019/1021), this prevents the prover from picking $\varC_{\varP}$ conditioned on $\varU$.
{{< /hint >}}

{{< hint warning >}}
**This construction is not hiding.** This construction does not hide the vectors $\veca$ and $\vecb$ since the prover reveals them to the verifier.
{{< /hint >}}

This construction requires the verifier and prover to exchange two group elements ($\varC_{\varP}$ and $\varU$) and two $n$-size vectors ($\veca$ and $\vecb$), giving total communication of size $2n + 2$. This is worse than the $O(1)$ communication required by similar commitment protocols like [KZG commitments]({{< ref "./../commitments/kzg_polynomial_commitment" >}}). Later versions will reduce this to $O(\log n)$.

### Version 2: Using Randomness to Halve Opening Communication

We are going to reduce the communication cost by reducing the size of the opening. In Version 1, we opened the commitments by providing the full-size vectors $\veca$ and $\vecb$. Here, we want to reduce this in half---open the commitment with half-size $\veca'$ and $\vecb'$. The key insight is that, the prover and verifier can use verifier-generated randomness to jointly compute a new commitment $\varC'$ which can be opened with half-size vectors $\veca'$ and $\vecb'$.

First, let us split the vectors $\veca$ and $\vecb$ and generators $\vecG$ and $\vecH$ in half, giving us
$$
\begin{align}
\veca_L &\coloneqq (\vara_1,\dots,\vara_{n/2})&
\veca_R &\coloneqq (\vara_{n/2+1},\dots,\vara_{n})\\\\
\vecb_L &\coloneqq (\varb_1,\dots,\varb_{n/2})&
\vecb_R &\coloneqq (\varb_{n/2+1},\dots,\varb_{n})\\\\
\vecG_L &\coloneqq (\varG_1,\dots,\varG_{\varn/2})&
\vecG_R &\coloneqq (\varG_{\varn/2+1},\dots,\varG_{\varn})\\\\
\vecH_L &\coloneqq (\varH_1,\dots,\varH_{\varn/2})&
\vecH_R &\coloneqq (\varH_{\varn/2+1},\dots,\varH_{\varn})\\\\
\end{align}
$$
Now, let's make use of verifier-provided randomness $\sample{\varu}$, to define new half-size vectors and corresponding half-size generators
$$
\begin{align}
\veca' &\coloneqq \veca_L + \varu^{-1} \cdot \veca_R&
\vecb' &\coloneqq \vecb_L + \varu \cdot \vecb_R\\\\
\vecG' &\coloneqq \vecG_L + \varu \cdot \vecG_R&
\vecH' &\coloneqq \vecH_L + \varu^{-1} \cdot \vecH_R
\end{align}
$$

{{< hint info >}}
**Other combinations.** There are other valid ways to compute the new half-size vectors. For instance, one can choose
$$
\begin{align}
\veca' &\coloneqq \varu \cdot \veca_L + \varu^{-1} \cdot \veca_R&
\vecb' &\coloneqq \varu^{-1} \cdot \vecb_L + \varu \cdot \vecb_R\\\\
\vecG' &\coloneqq \varu^{-1} \cdot \vecG_L + \varu \cdot \vecG_R&
\vecH' &\coloneqq \varu \cdot \vecH_L + \varu^{-1} \cdot \vecH_R
\end{align}
$$
and the same argument works.
{{< /hint >}}

Using these, we can define a new commitment $\varC' \coloneqq \ip{\veca'}{\vecG'} + \ip{\vecb'}{\vecH'} + \ip{\veca'}{\vecb'} \cdot \varU$ which can be opened with the half-size vectors $\veca'$ and $\vecb'$.

Now all that remains is for the verifier and prover to jointly compute the new commitment $\varC'$. They do this in three steps. First, the verifier locally generates some randomness and sends it to the prover. Second, the prover uses the verifier-provided randomness to compute some minimal structured information (the "cross-terms") and sends them to the verifier. Finally, the verifier uses the cross-terms, along with their previously-generated randomness and the original commitment to compute the new commitment.

By the linearity of inner product, we can write the original commitment as
$$
\begin{align}
\varC
&= \ip{\veca}{\vecG} + \ip{\vecb}{\vecH} + \ip{\veca}{\vecb} \cdot \varU\\\\
&= \ip{\veca_L}{\vecG_L} + \ip{\veca_R}{\vecG_R} + \ip{\vecb_L}{\vecH_L} + \ip{\vecb_R}{\vecH_R} + \ip{\veca_L}{\vecb_L} \cdot \varU + \ip{\veca_R}{\vecb_R} \cdot \varU
\end{align}
$$
Notice that this only has "straight-terms" (left terms paired with left terms and right terms paired with right terms), but to compute the new commitment $\varC'$, the verifier also needs the cross-terms defined as:
$$
\begin{align}
\varL &\coloneqq \ip{\veca_R}{\vecG_L} + \ip{\vecb_L}{\vecH_R} + \ip{\veca_R}{\vecb_L} \cdot \varU\\\\
\varR &\coloneqq \ip{\veca_L}{\vecG_R} + \ip{\vecb_R}{\vecH_L} + \ip{\veca_L}{\vecb_R} \cdot \varU
\end{align}
$$
Indeed, these two cross-terms $\varL$ and $\varR$ and the original commitment $\varC$ are sufficient for the verifier to compute the new commitment
$$
\begin{align}
\varC'
&= \ip{\veca'}{\vecG'} + \ip{\vecb'}{\vecH'} + \ip{\veca'}{\vecb'} \cdot \varU\\\\
&= (\veca_L + \varu^{-1} \cdot \veca_R)(\vecG_L + \varu \cdot \vecG_R) + (\vecb_L + \varu \cdot \vecb_R)(\vecH_L + \varu^{-1} \cdot \vecH_R) + \left(\veca_L + \varu^{-1} \cdot \veca_R\right)\left(\vecb_L + \varu \cdot \vecb_R\right) \cdot \varU\\\\
&= (\veca_L + \varu^{-1} \cdot \veca_R) \cdot \vecG_L + (\varu \cdot \veca_L + \veca_R) \cdot \vecG_R + (\vecb_L + \varu \cdot \vecb_R) \cdot \vecH_L + (\varu^{-1} \cdot \vecb_L + \vecb_R) \cdot \vecH_R + \left(\ip{\veca_L}{\vecb_L} + \varu \ip{\veca_L}{\vecb_R} + \varu^{-1} \ip{\veca_R}{\vecb_L} + \ip{\veca_R}{\vecb_R}\right) \cdot \varU\\\\
&= \varu^{-1}(\ip{\veca_R}{\vecG_L} + \ip{\vecb_L}{\vecH_R} + \ip{\veca_R}{\vecb_L} \cdot \varU) + (\ip{\veca_L}{\vecG_L} + \ip{\veca_R}{\vecG_R} + \ip{\vecb_L}{\vecH_L} + \ip{\vecb_R}{\vecH_R} + \ip{\veca_L}{\vecb_L} \cdot \varU + \ip{\veca_R}{\vecb_R} \cdot \varU) + \varu (\ip{\veca_L}{\vecG_R} + \ip{\vecb_R}{\vecH_L} + \ip{\veca_L}{\vecb_R} \cdot \varU)\\\\
&= \varu^{-1} \cdot \varL + \varC + \varu \cdot \varR
\end{align}
$$
Furthermore, at a high-level, this structure where the prover commits to $\varC$ and the verifier randomly generates $\varu$ prevents the prover from cheating and incorrectly computing $\veca'$ and $\vecb'$. See Appendix B of {{< cite Bulletproofs >}} for a formal proof.

Putting it all together, we can define a new protocol as follows:

{{< rawhtml >}}
 $$
 \begin{array}{lcl}
 \uwork{\varprover}{\varverifier}
 \alicework{\varC_P \coloneqq \ip{\veca}{\vecG} + \ip{\vecb}{\vecH}}
 \alicebob{}{\varC_P}{}
 \bobwork{\sampleCgroup{\varU}}
 \bobalice{}{\varU}{}
 \work{\varC \coloneqq \varC_P + \ip{\veca}{\vecb} \cdot \varU}{\varC \coloneqq \varC_P + \varz \cdot \varU}
 \aliceworks{\varL \coloneqq \ip{\veca_R}{\vecG_L} + \ip{\vecb_L}{\vecH_R} + \ip{\veca_R}{\vecb_L} \cdot \varU}
 \aliceworks{\varR \coloneqq \ip{\veca_L}{\vecG_R} + \ip{\vecb_R}{\vecH_L} + \ip{\veca_L}{\vecb_R} \cdot \varU}
 \alicebob{}{\varL, \varR}{}
 \bobwork{\sample{\varu}}
 \bobalice{}{\varu}{}
 \aliceworks{\vecG' \coloneqq \vecG_L + \varu \cdot \vecG_R}
 \aliceworks{\vecH' \coloneqq \vecH_L + \varu^{-1} \cdot \vecH_R}
 \aliceworks{\veca' \coloneqq \veca_L + \varu^{-1} \cdot \veca_R}
 \aliceworks{\vecb' \coloneqq \vecb_L + \varu \cdot \vecb_R}
 \alicebob{}{\veca', \vecb'}{}
 \bobworks{\vecG' \coloneqq \vecG_L + \varu \cdot \vecG_R}
 \bobworks{\vecH' \coloneqq \vecH_L + \varu^{-1} \cdot \vecH_R}
 \bobworks{\varC' \coloneqq \varu^{-1} \cdot \varL + \varC + \varu \cdot \varR}
 \bobworks{\varC' \equalQ \ip{\veca'}{\vecG'} + \ip{\vecb'}{\vecH'} + \ip{\veca'}{\vecb'} \cdot \varU}
 \end{array}
 $$
{{< /rawhtml >}}

{{< hint danger >}}
**$\varu$ must be chosen randomly.** If the prover can pick $\varu$ then they can forge proofs. For instance, they can pick $\varu = 1$ which gives them unconstrained control over $\varC'$, through $\varL$ and $\varR$.
{{< /hint >}}

{{< hint danger >}}
**$\varC$, $\vecG'$, $\vecH'$, and $\varC'$ must be honestly computed by the verifier.** If the verifier accepts these values directly from the prover, then the prover can cheat by picking these values such that they are satisfied by some arbitrary vectors, unrelated to the original commitment; i.e., it would make the statement trivially satisfiable.
{{< /hint >}}

This construction has perfect completeness and, as outline above, computational soundness. See Appendix B of {{< cite Bulletproofs >}} for a formal proof.

This construction requires the prover and verifier to exchange four group elements ($\varC_P$, $\varU$, $\varL$ and $\varR$), a scalar ($\varu$), and two $n/2$-size vectors ($\veca'$ and $\vecb'$), giving total communication of size $n + 5$. This is about half the communication cost of Version 1.

### Version 3: Non-Interactive Using the Fiat-Shamir Transform

Since Version 2 has the three-step commit-challenge-response structure we can apply the [Fiat-Shamir transformation]({{< ref "./../protocol-primitives/fiat-shamir.md" >}}) to make it non-interactive. We can compute the $\varverifier$'s random challenges $\varU \coloneqq \hashtogroup{\vecG, \vecH, \varz, \varC_P}$ and $\varu \coloneqq \hashtofield{\vecG, \vecH, \varz, \varC_P, \varU, \varC, \varL, \varR}$ as hashes of all the public values encountered till that point.

{{< hint info >}}
**Choice of $\HashToField$ and $\HashToGroup$.** For elliptic curves, see `hash_to_field` and `hash_to_curve` in [draft-irtf-cfrg-hash-to-curve-16](https://datatracker.ietf.org/doc/draft-irtf-cfrg-hash-to-curve/16/). Also, see [Hash function choice]({{< ref "./../protocol-primitives/nums.md#hash-function-choice" >}}) for notes on handling tuple inputs.
{{< /hint >}}

{{< rawhtml >}}
 $$
 \begin{array}{lcl}
 \uwork{\varprover}{\varverifier}
 \aliceworks{\varC_P \coloneqq \ip{\veca}{\vecG} + \ip{\vecb}{\vecH}}
 \aliceworks{\varU \coloneqq \hashtogroup{\vecG, \vecH, \varz, \varC_P}}
 \aliceworks{\varC \coloneqq \varC_P + \ip{\veca}{\vecb} \cdot \varU}
 \aliceworks{\varL \coloneqq \ip{\veca_R}{\vecG_L} + \ip{\vecb_L}{\vecH_R} + \ip{\veca_R}{\vecb_L} \cdot \varU}
 \aliceworks{\varR \coloneqq \ip{\veca_L}{\vecG_R} + \ip{\vecb_R}{\vecH_L} + \ip{\veca_L}{\vecb_R} \cdot \varU}
 \aliceworks{\varu \coloneqq \hashtofield{\vecG, \vecH, \varz, \varC_P, \varU, \varC, \varL, \varR}}
 \aliceworks{\vecG' \coloneqq \vecG_L + \varu \cdot \vecG_R}
 \aliceworks{\vecH' \coloneqq \vecH_L + \varu^{-1} \cdot \vecH_R}
 \aliceworks{\veca' \coloneqq \veca_L + \varu^{-1} \cdot \veca_R}
 \aliceworks{\vecb' \coloneqq \vecb_L + \varu \cdot \vecb_R}
 \alicebob{}{\varC_P, \varL, \varR}{}
 \alicebob{}{\veca', \vecb'}{}
 \bobworks{\varU \coloneqq \hashtogroup{\vecG, \vecH, \varz, \varC_P}}
 \bobworks{\varC \coloneqq \varC_P + \varz \cdot \varU}
 \bobworks{\varC' \coloneqq \varu^{-1} \cdot \varL + \varC + \varu \cdot \varR}
 \bobworks{\varu \coloneqq \hashtofield{\vecG, \vecH, \varz, \varC_P, \varU, \varC, \varL, \varR}}
 \bobworks{\vecG' \coloneqq \vecG_L + \varu \cdot \vecG_R}
 \bobworks{\vecH' \coloneqq \vecH_L + \varu^{-1} \cdot \vecH_R}
 \bobworks{\varC' \equalQ \ip{\veca'}{\vecG'} + \ip{\vecb'}{\vecH'} + \ip{\veca'}{\vecb'} \cdot \varU}
 \end{array}
 $$
{{< /rawhtml >}}

This construction has perfect completeness and, as outline above, computational soundness. See Appendix B of {{< cite Bulletproofs >}} for a formal proof.

This construction requires the prover to send three group elements ($\varC_P$, $\varL$, and $\varR$) and two $n/2$-size vectors ($\veca'$ and $\vecb'$), giving total communication of size $n + 3$. This is almost the same as Version 2, but now with only one round of communication instead of three.

### Version 4: Maximum Halving

Versions 2 and 3 halve the size of the vectors to generate a shorter proof. We can repeat this process successively with the halved vectors $\log_2(n)$ times to reduce the size of the vectors to $1$ element.

Let $\varm \coloneqq \log_2(n)$ be the number of times we halve the vectors, and let us use a new variable $\ctx$ to keep track of all the public values for use in the Fiat-Shamir hash.

Also, since the verifier does not need the intermediate halved generators $\vecG_{\varj}$ and $\vecH_{\varj}$ for $\varj < \varm$, it can directly compute the final halved generators $\vecG_{\varm}$ and $\vecH_{\varm}$:
$$
\begin{align}
\vecG_{\varm} &= \varG_1 + \varu_{\varm} \varG_2 + \varu_{\varm-1} \varG_3 + \varu_{\varm-1}\varu_{\varm} \varG_4 + \cdots + \varu_1 \varu_2 \cdots \varu_{\varm} \cdot \varG_n\\\\
\vecH_{\varm} &= \varH_1 + \varu_{\varm}^{-1} \varH_2 + \varu_{\varm-1}^{-1} \varH_3 + \varu_{\varm-1}^{-1}\varu_{\varm}^{-1} \varH_4 + \cdots + \varu_1^{-1} \varu_2^{-1} \cdots \varu_{\varm}^{-1} \cdot \varH_n
\end{align}
$$
and this can be done using [multiscalar multiplication](https://jbootle.github.io/Misc/pippenger.pdf) which is faster than multiplying by scalars one-by-one.

Going a step further, we can make use of the manner in which we compute the $\vecG_{\varm}$ ($\varm$ rounds of multiply the right half by $\varu_i$ and add to the left half), and similarly $\vecH_{\varm}$ to define the polynomials
$$
\begin{align}
\varg(X)
\coloneqq \prod_{i=0}^{m-1} (1 + \varu_{m-i} X^{2^i})
&= 1 + \varu_{\varm} X + \varu_{\varm-1} X^{2} + \varu_{\varm-1}\varu_{\varm} X^{3} + \cdots + \varu_1 \cdots \varu_{\varm} X^{\varn}
\\\\
\varh(X)
\coloneqq \prod_{i=0}^{m-1} (1 + \varu_{m-i}^{-1} X^{2^i})
&= 1 + \varu_{\varm}^{-1} X + \varu_{\varm-1}^{-1} X^{2} + \varu_{\varm-1}^{-1}\varu_{\varm}^{-1} X^{3} + \cdots + \varu_1^{-1} \cdots \varu_{\varm}^{-1} X^{\varn}
\end{align}
$$
and write $\vecG_{\varm} = \ip{\vecg}{\vecG}$ and $\vecH_{\varm} = \ip{\vech}{\vecH}$ where $\vecg$ and $\vech$ are vectors corresponding to the polynomials $\varg$ and $\varh$ respectively.

{{< hint info >}}
**Amortization.** The verifier can leverage the observation that $\varG_{\varm}$ and $\varH_{\varm}$ can be written as polynomial commitments to amortize the verification cost over multiple invocations. Rather than the verifier computing these values themselves for each invocation, at the end of all invocations, they can ask the prover to provide all these values along with a Schnorr-like proof that they were computed correctly. See Section 3.2 of [[BGH19]](https://eprint.iacr.org/2019/1021) for details. Note that the polynomial we use is different from the one in [[BGH19]](https://eprint.iacr.org/2019/1021) since we compute the half-size vectors differently.
{{< /hint >}}

The full protocol looks as follows:

{{< rawhtml >}}
 $$
 \begin{array}{lcl}
 \uwork{\varprover}{\varverifier}
 \aliceworks{\varC_P \coloneqq \ip{\veca}{\vecG} + \ip{\vecb}{\vecH}}
 \aliceworks{\varU \coloneqq \hashtogroup{\vecG, \vecH, \varz, \varC_P}}
 \aliceworks{\varC_0 \coloneqq \varC_P + \ip{\veca}{\vecb} \cdot \varU}
 \aliceworks{\ctx \gets (\vecG, \vecH, \varz, \varC_P, \varU, \varC_0)}
 \aliceworks{\veca_0 \coloneqq \veca;\; \vecb_0 \coloneqq \vecb;\; \vecG_0 \coloneqq \vecG;\; \vecH_0 \coloneqq \vecH}
 \aliceworks{\text{For $i \coloneqq 1$ to $\varm$:}}
 \aliceworks{\indent \varL_i \coloneqq \ip{\veca_{i-1,R}}{\vecG_{i-1,L}} + \ip{\vecb_{i-1,L}}{\vecH_{i-1,R}} + \ip{\veca_{i-1,R}}{\vecb_{i-1,L}} \cdot \varU}
 \aliceworks{\indent \varR_i \coloneqq \ip{\veca_{i-1,L}}{\vecG_{i-1,R}} + \ip{\vecb_{i-1,R}}{\vecH_{i-1,L}} + \ip{\veca_{i-1,L}}{\vecb_{i-1,R}} \cdot \varU}
 \aliceworks{\indent \ctx.\append(\varL_i, \varR_i)}
 \aliceworks{\indent \varu_i \coloneqq \hashtofield{\ctx}}
 \aliceworks{\indent \vecG_i \coloneqq \vecG_{i-1, L} + \varu_i \cdot \vecG_{i-1,R}}
 \aliceworks{\indent \vecH_i \coloneqq \vecH_{i-1,L} + \varu_i^{-1} \cdot \vecH_{i-1,R}}
 \aliceworks{\indent \veca_i \coloneqq \veca_{i-1,L} + \varu_i^{-1} \cdot \veca_{i-1,R}}
 \aliceworks{\indent \vecb_i \coloneqq \vecb_{i-1,L} + \varu_i \cdot \vecb_{i-1,R}}
 \alicebob{}{\varC_P, (\varL_1,\varR_1),\dots,(\varL_{\varm}, \varR_{\varm})}{}
 \alicebob{}{\veca_{\varm}, \vecb_{\varm}}{}
 \bobworks{\varz \mod \varq \gQ 0}
 \bobworks{\varC_P \neq 0 \text{ (point at infinity for EC groups)}}
 \bobworks{\varC_P \inQ \cgroup \text{ (on curve check for EC groups)}}
 \bobworks{\varL_i, \varR_i \neq 0 \text{ (point at infinity for EC groups)}}
 \bobworks{\varL_i, \varR_i \inQ \cgroup \text{ (on curve check for EC groups)}}
 \bobworks{\vara_i \mod \varq \neq 0}
 \bobworks{\varb_i \mod \varq \neq 0}
 \bobwork{\text{ for }i=1,\ldots,m}
 \bobseparator
 \bobworks{\varU \coloneqq \hashtogroup{\vecG, \vecH, \varz, \varC_P}}
 \bobworks{\varC_0 \coloneqq \varC_P + \varz \cdot \varU}
 \bobworks{\ctx \gets (\vecG, \vecH, \varz, \varC_P, \varU, \varC_0)}
 \bobworks{\text{For $i \coloneqq 1$ to $\varm$:}}
 \bobworks{\indent \ctx.\append(\varL_i, \varR_i)}
 \bobworks{\indent \varu_i \coloneqq \hashtofield{\ctx}}
 \bobworks{\indent \varC_{i} \coloneqq \varu_{i}^{-1} \cdot \varL_{i} + \varC_{i-1} + \varu_{i} \cdot \varR_{i}}
 \bobworks{\vecg \coloneqq \varg(X) \coloneqq \prod_{i=0}^{m-1} (1 + \varu_{m-i} X^{2^i})}
 \bobworks{\vech \coloneqq \varh(X) \coloneqq \prod_{i=0}^{m-1} (1 + \varu_{m-i}^{-1} X^{2^i})}
 \bobworks{\vecG_{\varm} \coloneqq \ip{\vecg}{\vecG}}
 \bobworks{\vecH_{\varm} \coloneqq \ip{\vech}{\vecH}}
 \bobworks{\varC_{\varm} \equalQ \ip{\veca_{\varm}}{\vecG_{\varm}} + \ip{\vecb_{\varm}}{\vecH_{\varm}} + \ip{\veca_{\varm}}{\vecb_{\varm}} \cdot \varU}
 \end{array}
 $$
{{< /rawhtml >}}

This construction has perfect completeness and, as outlined for Version 2, computational soundness. See Appendix B of {{< cite Bulletproofs >}} for a formal proof.

This construction requires the prover to send $2m + 1$ group elements ($\varC_P, (\varL_1, \varR_1),\dots,(\varL_{\varm}, \varR_{\varm})$) and two $1$-size vectors ($\veca'$ and $\vecb'$), giving total communication of size $2m + 3 = 2\log_2(n) + 3$.

## Security Assumptions

* **Discrete logarithm:** The security of this protocol relies on the hardness of discrete logarithm in the group $\cgroup$. Specifically, it assumes that there are no known discrete log relations between the random generators. Concretely, for 128-bit security consider elliptic curve groups of size greater than $2^{256}$.

## Security Pitfalls
* **Verifier input validation:** Each of the items above the dotted line for the $\varverifier$ is essential to the security of the protocol. If any of these checks are missing or insufficient it is likely a severe security issue.
* **Weak Fiat-Shamir transform:** When transforming the interactive protocol into a non-interactive protocol with the Fiat-Shamir transform, care needs to be taken to ensure that all parameters are included in the hash. See [Fiat-Shamir transformation]({{< ref "./../protocol-primitives/fiat-shamir.md" >}}).
* **Replay attacks:** This construction does not provide replay protection; i.e., proofs can be replayed. But, replay protection can be achieved by adding more information to the $\Hash$ invocations, see [Preventing replay attacks]({{< ref "./../protocol-primitives/fiat-shamir.md#preventing-replay-attacks" >}}).
* **Malicious verifiers in interactive protocol:** The interactive version of the halving protocol (Version 2) assumes an honest verifier. If the verifier deviates from the protocol, then security may no longer hold. See the warning in Section 2, and [Using HVZKP in the wrong context]({{< ref "./../security-of-zkps/when-to-use-hvzk.md" >}}).
* **Verifier trusting the prover:** All version of this protocol assume that the verifier does not trust the prover beyond the protocol. See the warning in Section 2.


## See also
 - {{< section_entry "docs/zkdocs/commitments/ipa-pcs" >}}