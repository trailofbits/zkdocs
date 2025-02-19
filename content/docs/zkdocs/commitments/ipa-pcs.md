---
weight: 11
bookFlatSection: true
title: "IPA Polynomial Commitment"
summary: "An overview of the Inner Product Argument polynomial commitment scheme"
needsVariableResetButton: true
references: ["Bootleproofs", "Bulletproofs", "Halo", "ProofCarryingData", "Halo2"]
---

# IPA Polynomial Commitment

## Overview of IPA Polynomial Commitment

The inner product argument was introduced by Bootle et al. {{< cite Bootleproofs >}}, and refined in Bulletproofs {{< cite Bulletproofs >}}. This protocol was specialized to polynomial commitments in {{< cite Halo >}} then further studied and refined in {{< cite ProofCarryingData >}}.

{{< hint info >}}
**Goal:**
$\varprover$ convinces $\varverifier$ that they know a secret polynomial $\varf$ such that $\varf(\varx) = \varv$, for evaluation point $\varx$ and evaluation $\varv$; and this satisfies the polynomial commitment $\varC_{\varP}$.
{{< /hint >}}

 * __Public input:__ a cyclic additive group $\cgroup$ of prime order $\varq$ and $n+1$ random $\cgroup$ generators $\vecG = (\varG_1,\dots,\varG_n)$ and $\varH$; polynomial commitment $\varC_{\varP} \in \cgroup$, scalar $\varx \in \zq$, and scalar $\varv \in \zq$.

 * __Private input:__ $\varprover$ knows secret polynomial $\varf$ that satisfies the polynomial commitment $\varC_P$ and $\varf(\varx) = \varv$.

**Comparison with [KZG Polynomial Commitments]({{< ref "./kzg_polynomial_commitment" >}}).**
IPA PCS requires fewer assumptions but is less efficient. IPA does not require a trusted setup and only assumes discrete log, whereas KZG requires a trusted setup and assumes pairings. But, IPA proofs are $O(\log \varn)$ and the verifier needs to do $O(n)$ work, whereas KZG proofs are $O(1)$ and the verifier only needs to do $O(1)$ work. That said, the verification time for IPA proofs can be amortized over multiple runs.

## Polynomial Commitment from IPA

In this section, we build polynomial commitments from IPA over two successive versions.

### Version 1: A Simple, Non-Hiding Construction

The most straightforward way to do polynomial commitments is to interpret the polynomial $f(\varX) = \vara_1 + \vara_2 \varX + \vara_3 \varX^2 + \cdots + \vara_{\varn} \varX^{\varn-1}$ as a vector of coefficients $(a_1,\dots,a_{\varn})$ and fix the other vector to be powers of the evaluation point $\vecb = (1, \varx, \varx^2, \dots, \varx^{\varn-1})$. Then their inner product
$$
\ip{\veca}{\vecb} = \vara_1 + \vara_2 \varx + \vara_3 \varx^2 + \cdots + \vara_{\varn} \varx^{\varn-1}
$$
is the evaluation $f(x)$.

Since $\vecb$ is now public, the prover no longer has to commit to it. So, the verifier can compute the reduced value $\vecb_{\varm}$ similar to how it computes $\vecG_{\varm}$ in the Inner Product Argument using the polynomial $\varg(\varX) \coloneqq \prod_{i=0}^{\varm-1} (1 + \varu_{\varm-i} \varX^{2^i})$:
$$
\begin{align}
\vecG_{\varm} &= \varG_1 + \varu_{\varm} \varG_2 + \cdots + \varu_1 \varu_2 \cdots \varu_{\varm} \cdot \varG_n = \ip{\vecg}{\vecG}\\\\
\vecb_{\varm} &= 1 + \varu_{\varm} \varx + \cdots + \varu_1 \varu_2 \cdots \varu_{\varm} \cdot \varx^{\varn-1} = \ip{\vecg}{\vecb}\\\\
\end{align}
$$

The full protocol, adapted from Version 4 in [Inner Product Argument]({{< ref "./../zero-knowledge-protocols/ipa/_index.md" >}}) is as follows:

{{< rawhtml >}}
 $$
 \begin{array}{lcl}
 \uwork{\varprover}{\varverifier}
 \aliceworks{\veca \coloneqq (\vara_1,\dots,\vara_{\varn})}
 \dupwork{\vecb \coloneqq (1, \varx, \varx^2, \dots, \varx^{\varn-1})}
 \aliceworks{\varC_P \coloneqq \ip{\veca}{\vecG}}
 \aliceworks{\varU \coloneqq \hashtogroup{\vecG, \varx, \varv, \varC_P}}
 \aliceworks{\varC_0 \coloneqq \varC_P + \ip{\veca}{\vecb} \cdot \varU}
 \aliceworks{\ctx \gets (\vecG, \varx, \varv, \varC_P, \varU, \varC_0)}
 \aliceworks{\veca_0 \coloneqq \veca;\; \vecb_0 \coloneqq \vecb;\; \vecG_0 \coloneqq \vecG}
 \aliceworks{\text{For $i \coloneqq 1$ to $\varm$:}}
 \aliceworks{\indent \varL_i \coloneqq \ip{\veca_{i-1,R}}{\vecG_{i-1,L}} + \ip{\veca_{i-1,R}}{\vecb_{i-1,L}} \cdot \varU}
 \aliceworks{\indent \varR_i \coloneqq \ip{\veca_{i-1,L}}{\vecG_{i-1,R}} + \ip{\veca_{i-1,L}}{\vecb_{i-1,R}} \cdot \varU}
 \aliceworks{\indent \ctx.\append(\varL_i, \varR_i)}
 \aliceworks{\indent \varu_i \coloneqq \hashtofield{\ctx}}
 \aliceworks{\indent \vecG_i \coloneqq \vecG_{i-1, L} + \varu_i \cdot \vecG_{i-1,R}}
 \aliceworks{\indent \veca_i \coloneqq \veca_{i-1,L} + \varu_i^{-1} \cdot \veca_{i-1,R}}
 \aliceworks{\indent \vecb_i \coloneqq \vecb_{i-1,L} + \varu_i \cdot \vecb_{i-1,R}}
 \alicebob{}{\varC_P, (\varL_1,\varR_1),\dots,(\varL_{\varm}, \varR_{\varm})}{}
 \alicebob{}{\veca_{\varm}}{}
 \bobworks{\varU \coloneqq \hashtogroup{\vecG, \varx, \varv, \varC_P}}
 \bobworks{\varC_0 \coloneqq \varC_P + \varv \cdot \varU}
 \bobworks{\ctx \gets (\vecG, \varx, \varv, \varC_P, \varU, \varC_0)}
 \bobworks{\text{For $i \coloneqq 1$ to $\varm$:}}
 \bobworks{\indent \ctx.\append(\varL_i, \varR_i)}
 \bobworks{\indent \varu_i \coloneqq \hashtofield{\ctx}}
 \bobworks{\indent \varC_{i} \coloneqq \varu_{i}^{-1} \cdot \varL_{i} + \varC_{i-1} + \varu_{i} \cdot \varR_{i}}
 \bobworks{\vecg \coloneqq \varg(X) \coloneqq \prod_{i=0}^{m-1} (1 + \varu_{m-i} X^{2^i})}
 \bobworks{\vecG_{\varm} \coloneqq \ip{\vecg}{\vecG}}
 \bobworks{\vecb_{\varm} \coloneqq \ip{\vecg}{\vecb}}
 \bobworks{\varC_{\varm} \equalQ \ip{\veca_{\varm}}{\vecG_{\varm}} + \ip{\veca_{\varm}}{\vecb_{\varm}} \cdot \varU}
 \end{array}
 $$
{{< /rawhtml >}}

{{< hint warning >}}
**This construction is not hiding.** This construction reveals $\veca_{\varm}$ which in turns leaks at least one bit of information about $\veca = \varf$.
{{< /hint >}}

{{< hint info >}}
**Amortization.** The computation of $\vecG_{\varm}$ and $\vecb_{\varm}$ (the only linear time computations the verifier needs to do) can be amortized over multiple runs, see "Amortization" in [Inner Product Argument]({{< ref "./../zero-knowledge-protocols/ipa/_index.md" >}}) for details.
{{< /hint >}}

### Version 2: Hiding Construction

Version 1 did not hide the polynomial $\varf$ and thus was not zero-knowledge. This version remedies that by using hiding commitments and an intermediate random-looking polynomial.

First, we compute a hiding commitment $\varC_P$ to the polynomial $\varf$. Let $\veca$ and $\vecb$ be the coefficients of $\varf$ and powers of the evaluation point $\varx$, respectively, as before. The prover samples a random blinding factor $\sample{\vart}$ and computes
$$
\varC_P \coloneqq \ip{\veca}{\vecG} + \vart \cdot \varH
$$

Second, we compute a random-looking polynomial. Sample a random polynomial $p_r(\varX) \coloneqq \vars_1 + \vars_2 \varX + \vars_3 \varX^2 + \cdots + \vars_{\varn} \varX^{\varn-1}$ of the same degree as $\varf$ by picking random coefficients $\vars_1,\dots,\vars_{\varn}$. Construct $\bar{\varf} \coloneqq p_r - p_r(\varx)$ so that $\bar{\varf}$ looks random and has a root at the evaluation point $\varx$; that is, $\bar{\varf}(\varx) = 0$.

Third, we compute a hiding commitment to the random-looking polynomial. Let $\bar{\veca} \coloneqq (\varw_{1},\dots,\varw_{\varn})$ be the coefficients of $\bar{\varf}$. The prover samples a random blinding factor $\sample{\bar{\vart}}$ and computes the commitment
$$
\bar{\varC}_P \coloneqq \ip{\bar{\veca}}{\vecG} + \bar{\vart} \cdot \varH
$$

Fourth, we blind the polynomial $\varf$ using the random-looking polynomial $\bar{\varf}$. After receiving these commitments, the verifier samples some randomness $\varalpha$ and the prover uses it to compute the blinded polynomial $\varf' \coloneqq \varf + \varalpha \cdot \bar{\varf}$. Let $\vecc = (\varc_1,\dots,\varc_n)$ be the coefficients of $\varf'$. The prover computes the corresponding randomness $\vart' \coloneqq \vart + \varalpha \cdot \bar{\vart}$ and uses it to compute a non-hiding commitment to $\varf'$ as $\varC_P' \coloneqq \varC_P + \varalpha \cdot \bar{\varC}_P - \vart' \cdot \varH = \ip{\vecc}{\vecG}$.

Now we continue with the halving like Version 1, except using the blinded polynomial $\varf'$ in place of the original polynomial $\varf$. The full protocol is below.

{{< rawhtml >}}
 $$
 \begin{array}{lcl}
 \uwork{\varprover}{\varverifier}
 \aliceworks{\veca \coloneqq (\vara_1,\dots,\vara_{\varn})}
 \dupwork{\vecb \coloneqq (1, \varx, \varx^2, \dots, \varx^{\varn-1})}
 \aliceworks{\sample{\vart}}
 \aliceworks{\varC_P \coloneqq \ip{\veca}{\vecG} + \vart \cdot \varH}
 \aliceworks{\sampleGeneric{\varp_r}{\zq^{\varn}}}
 \aliceworks{\bar{\varf} \coloneqq \varp_r - \varp_r(\varx)}
 \aliceworks{\bar{\veca} \coloneqq (\varw_1,\dots,\varw_n) = \bar{\varf}}
 \aliceworks{\sample{\bar{\vart}}}
 \aliceworks{\bar{\varC}_P \coloneqq \ip{\bar{\veca}}{\vecG} + \bar{\vart} \cdot \varH}
 \aliceworks{\ctx \gets (\vecG, \varH, \varx, \varv, \varC_P, \bar{\varC}_P)}
 \aliceworks{\varalpha \coloneqq \hashtofield{\ctx}}
 \aliceworks{\vecc \coloneqq \veca + \alpha \cdot \bar{\veca} = \varf'}
 \aliceworks{\vart' \coloneqq \vart + \varalpha \cdot \bar{\vart}}
 \aliceworks{\varC_P' \coloneqq \varC_P + \varalpha \cdot \bar{\varC}_P - \vart' \cdot \varH = \ip{\vecc}{\vecG}}
 \aliceworks{\ctx.\append(\varC_P')}
 \aliceworks{\varU \coloneqq \hashtogroup{\ctx}}
 \aliceworks{\varC_0 \coloneqq \varC_P + \ip{\veca}{\vecb} \cdot \varU}
 \aliceworks{\ctx.\append(\varC_0)}
 \aliceworks{\vecc_0 \coloneqq \vecc;\; \vecb_0 \coloneqq \vecb;\; \vecG_0 \coloneqq \vecG}
 \aliceworks{\text{For $i \coloneqq 1$ to $\varm$:}}
 \aliceworks{\indent \varL_i \coloneqq \ip{\veca_{i-1,R}}{\vecG_{i-1,L}} + \ip{\veca_{i-1,R}}{\vecb_{i-1,L}} \cdot \varU}
 \aliceworks{\indent \varR_i \coloneqq \ip{\veca_{i-1,L}}{\vecG_{i-1,R}} + \ip{\veca_{i-1,L}}{\vecb_{i-1,R}} \cdot \varU}
 \aliceworks{\indent \ctx.\append(\varL_i, \varR_i)}
 \aliceworks{\indent \varu_i \coloneqq \hashtofield{\ctx}}
 \aliceworks{\indent \vecG_i \coloneqq \vecG_{i-1, L} + \varu_i \cdot \vecG_{i-1,R}}
 \aliceworks{\indent \vecc_i \coloneqq \vecc_{i-1,L} + \varu_i^{-1} \cdot \vecc_{i-1,R}}
 \aliceworks{\indent \vecb_i \coloneqq \vecb_{i-1,L} + \varu_i \cdot \vecb_{i-1,R}}
 \alicebob{}{\varC_P, (\varL_1,\varR_1),\dots,(\varL_{\varm}, \varR_{\varm})}{}
 \alicebob{}{\bar{\varC}_P, \veca_{\varm}, \vart'}{}
 \bobworks{\varv \mod \varq \gQ 0}
 \bobworks{\varx \mod \varq \gQ 0}
 \bobworks{\vart' \mod \varq \gQ 0}
 \bobworks{\varC_P \neq 0 \text{ (point at infinity for EC groups)}}
 \bobworks{\varC_P \inQ \cgroup \text{ (on curve check for EC groups)}}
 \bobworks{\bar{\varC}_P \neq 0 \text{ (point at infinity for EC groups)}}
 \bobworks{\bar{\varC}_P \inQ \cgroup \text{ (on curve check for EC groups)}}
 \bobworks{\varL_i, \varR_i \neq 0 \text{ (point at infinity for EC groups)}}
 \bobworks{\varL_i, \varR_i \inQ \cgroup \text{ (on curve check for EC groups)}}
 \bobworks{\vara_i \mod \varq \neq 0}
 \bobwork{\text{ for }i=1,\ldots,m}
 \bobseparator
 \bobworks{\ctx \gets (\vecG, \varH, \varx, \varv, \varC_P, \bar{\varC}_P)}
 \bobworks{\varalpha \coloneqq \hashtofield{\ctx}}
 \bobworks{\varC_P' \coloneqq \varC_P + \varalpha \cdot \bar{\varC}_P - \vart' \cdot \varH}
 \bobworks{\ctx.\append(\varC_P')}
 \bobworks{\varU \coloneqq \hashtogroup{\ctx}}
 \bobworks{\varC_0 \coloneqq \varC' + \varv \cdot \varU}
 \bobworks{\ctx.\append(\varC_0)}
 \bobworks{\text{For $i \coloneqq 1$ to $\varm$:}}
 \bobworks{\indent \ctx.\append(\varL_i, \varR_i)}
 \bobworks{\indent \varu_i \coloneqq \hashtofield{\ctx}}
 \bobworks{\indent \varC_{i} \coloneqq \varu_{i}^{-1} \cdot \varL_{i} + \varC_{i-1} + \varu_{i} \cdot \varR_{i}}
 \bobworks{\vecg \coloneqq \varg(X) \coloneqq \prod_{i=0}^{m-1} (1 + \varu_{m-i} X^{2^i})}
 \bobworks{\vecG_{\varm} \coloneqq \ip{\vecg}{\vecG}}
 \bobworks{\vecb_{\varm} \coloneqq \ip{\vecg}{\vecb}}
 \bobworks{\varC_{\varm} \equalQ \ip{\veca_{\varm}}{\vecG_{\varm}} + \ip{\veca_{\varm}}{\vecb_{\varm}} \cdot \varU}
 \end{array}
 $$
{{< /rawhtml >}}

{{< hint info >}}
**Comparison with [[BCMS20]](https://eprint.iacr.org/2020/499).** Rather than compute $\varU \coloneqq \xi_0 \cdot \varH$ using a challenge field element $\xi_0$, we directly ask for the challenge group element $\varU$. These approaches are equivalent.
{{< /hint >}}

{{< hint info >}}
**Comparison with [[BGH19]](https://eprint.iacr.org/2019/1021).** Rather than run halving with the original polynomial and blind the outputs, $\varL_i$, $\varR_i$, and $\veca_{\varm}$, we instead run halving with a blinded polynomial $\bar{\varf}$.
{{< /hint >}}

This construction has perfect completeness and, as outlined above, computational soundness. See Appendix A.3 of {{< cite ProofCarryingData >}} for a proof.

## Security Assumptions

* **Discrete logarithm:** The security of this protocol relies on the hardness of discrete logarithm in the group $\cgroup$. Specifically, it assumes that there are no known discrete log relations between the random generators. Concretely, for 128-bit security consider elliptic curve groups of size greater than $2^{256}$.

## Security Pitfalls
* **Verifier input validation:** Each of the items above the dotted line for the $\varverifier$ is essential to the security of the protocol. If any of these checks are missing or insufficient it is likely a severe security issue.
* **Weak Fiat-Shamir transform:** When transforming the interactive protocol into a non-interactive protocol with the Fiat-Shamir transform, care needs to be taken to ensure that all parameters are included in the hash. See [Fiat-Shamir transformation]({{< ref "./../protocol-primitives/fiat-shamir.md" >}}).
* **Replay attacks:** This construction does not provide replay protection; i.e., proofs can be replayed. But, replay protection can be achieved by adding more information to the $\Hash$ invocations, see [Preventing replay attacks]({{< ref "./../protocol-primitives/fiat-shamir.md#preventing-replay-attacks" >}}).
* **Verifier trusting the prover:** All versions of this protocol assume that the verifier does not trust the prover beyond the protocol. See the warning in Section 2 of [Inner Product Argument]({{< ref "./../zero-knowledge-protocols/ipa/_index.md" >}}).

## See also
 - {{< section_entry "docs/zkdocs/zero-knowledge-protocols/ipa" >}}
