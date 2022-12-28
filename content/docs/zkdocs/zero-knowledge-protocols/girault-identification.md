---
weight: 5
bookFlatSection: true
title: "Girault's identification protocol"
summary: "A statistical zero-knowledge proof for discrete-logarithm in a composite modulo."
needsVariableResetButton: true
references: ["girault", "giraultref"]
---
# Girault's identification protocol
This scheme is a zero-knowledge proof for a discrete logarithm, like [Schnorr's]({{< ref "schnorr.md" >}}) [protocol]({{< ref "schnorr-variants.md" >}}), but over a composite modulus instead of a prime modulus.


{{< hint info >}}
**Goal:**
$\varprover$ convinces $\varverifier$ that they know $\varx$ such that $\varh = \varg^{-\varx} \mod \varN$
{{< /hint >}}

 * __Public input:__ $\varh, \varN$  and a high order generator $\varg\in \zns{\varN}$
 * __Private input:__ $\varprover$ knows the secret $\varx\in \range{S}$
 * __Security parameters:__ The parameters $k, k', S$ and $R = 2^{k+k' + |S|}$.

### Interactive protocol
{{< hint danger >}}
**Security note:**
The interactive identification protocol assumes an honest verifier and should not be used in the context of malicious verifiers. A malicious verifier can send $\vare = R$ and recover the secret $\varx$ by dividing $\varz$ by $\vare$.
{{< /hint >}}

{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\varh = \varg^{-\varx} \mod \varN}
 \alicework{\sampleRange{\varr}{R}}
 \alicework{\varu = \varg^\varr \mod \varN}
 \alicebob{}{\varu}{}
 \bobwork{\sampleRange{\vare}{2^k}}
 \bobalice{}{\vare}{}
 \alicework{\varz = \varr + \varx\cdot \vare \in \naturals}
 \alicebob{}{\varz}{}
 \bobwork{\varu \equalQ \varg^{\varz} \cdot \varh^\vare \mod \varN}
 \end{array}
 $$

{{< /rawhtml >}}

-----

### Non-interactive protocol
We obtain a non-interactive protocol using the Fiat-Shamir heuristic, where the prover
creates the random $k$-bit challenge $\vare$ using domain-separated hash function over the $\\{\varg, \varN, \varh, \varu\\}$ parameters.
{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\varh = \varg^{-\varx} \mod \varN}
 \alicework{\sampleRange{\varr}{R}}
 \alicework{\varu = \varg^\varr \mod \varN}
 \alicework{\vare = \hashbit{\varg, \varN, \varh, \varu}{k}}
 \alicework{\varz = \varr + \varx\cdot \vare \in \naturals}
 \alicebob{}{\varu, \vare, \varz}{}
 \bobwork{\vare \equalQ \hashbit{\varg, \varN, \varh, \varu}{k}}
 \bobwork{\varu \equalQ \varg^{\varz} \cdot \varh^\vare \mod \varN}
 \end{array}
 $$
{{< /rawhtml >}}

## Security pitfalls
 * **Parameter choice:** Implementers must pay special attention to the choice of parameter values, in particular the relation between $2^k$ and $R$. If these were of similar size, since $\varz$ is computed over the naturals, $\varx$ would be approximately $\lfloor \varz/\vare\rfloor$.
 * __Using the interactive protocol in a malicious verifier context:__ high severity issue which allows recovering the secret $\varx$; see [Using HVZKP in the wrong context]({{< relref "/docs/zkdocs/security-of-zkps/when-to-use-hvzk" >}}).
 * __Verifier trusting prover on the non-interactive protocol:__
   * $\varverifier$ uses a $\varg$ value provided by $\varprover$ instead of using the standard generator: this is a high severity issue since the prover can trivially forge proofs (e.g., by sending $\varu=0, \varg=0$).
   * $\varverifier$ does not validate $\varu,\varh$ as valid elements of $\zns{\varN}$ (between 1 and $\varN-1$ and with $\gcd(k, \varN) = 1$): this allows replaying the *same* proof with *different* values adding multiples of $\varN$.
 * __Replay attacks:__ After a non-interactive proof is public, it will always be valid, and anyone could pretend to know how to prove the original statement. To prevent this, consider adding additional information to the computation of the hash function: values such as an ID unique to the prover and verifier, and a timestamp. The verifier must use these values and check their validity to verify the proof.


## Choice of parameter values
 - $|\varN| = 2048$
 - $S = 2^{256}$
 - $k,k' = 128$

## Auxiliary procedures
 - * __Hash function $\hashbit{\cdot}{k}$:__ this hash function should be domain-separated and have a specific output size of $k$-bits. Using $\mathsf{TupleHash}$ satisfies these restrictions.



