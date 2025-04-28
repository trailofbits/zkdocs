---
weight: 1
bookFlatSection: true
title: "Schnorr's identification protocol"
summary: "The zero-knowledge proof for a discrete-logarithm in a prime modulo."
needsVariableResetButton: true
references: ["Sch91", "schnorr-lecture", "shake", "safe-primes"]
---
# Schnorr's identification protocol
  Schnorr's identification protocol is the simplest example of a zero-knowledge protocol. With it,
  $\varprover$ can convince $\varverifier$ that they know the discrete logarithm $\varx$ of some value
  $\varh = \varg^\varx$, without revealing $\varx$.

{{< hint info >}}
**Goal:**
$\varprover$ convinces $\varverifier$ that they know $\varx$ such that $\varh = \varg^\varx$.
{{< /hint >}}

 * __Public input:__ cyclic group $\cgroup$ of prime order $\varq$, a $\cgroup$ generator $\varg$ and $\varh\in \cgroup$.
 * __Private input:__ $\varprover$ knows secret $\varx\in\zq$ such that $\varh = \varg^\varx$.


{{< tabs "uniqueid" >}}
{{< tab "Multiplicative notation" >}}
### Interactive protocol
The Schnorr interactive identification protocol

{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\samplezqs{\varr}}
 \alicework{\varu = \varg^\varr}
 \alicebob{}{\varu}{}
 \bobwork{\schnorrvalidate(\varu, \varh)}
 \bobseparator
 \bobwork{\sample{\varc}}
 \bobalice{}{\varc}{}
 \alicework{\varz = \varr + \varx\cdot \varc}
 \alicebob{}{\varz}{}
 \bobwork{\varz \neq 0 \mod \varq}
 \bobseparator
 \bobwork{\varg^{\varz} \equalQ \varu \cdot \varh^\varc }
 \end{array}
 $$

{{< /rawhtml >}}
-----

### Non-interactive protocol
We can transform this identification scheme into a non-interactive protocol using the Fiat-Shamir heuristic. Here, the prover creates the random challenge $c$ hashing *all* public values $\\{\varg, \varq, \varh, \varu\\}$.
{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\samplezqs{\varr}}
 \alicework{\varu = \varg^\varr}
 \alicework{\varc = \hash{\varg, \varq, \varh, \varu}}
 \alicework{\varz = \varr + \varx\cdot \varc}
 \alicebob{}{\varu, \varc, \varz}{}
 \bobwork{\schnorrvalidate(\varu, \varh)}
 \bobwork{\varz \neq 0 \mod \varq}
 \bobseparator
 \bobwork{\varc \equalQ \hash{\varg, \varq, \varh, \varu}}
 \bobwork{\varg^\varz \equalQ \varu \cdot \varh ^\varc }
 \end{array}
 $$
{{< /rawhtml >}}
{{< /tab >}}


{{< tab "Additive notation" >}}

### Interactive protocol
The Schnorr interactive identification protocol


{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\samplezqs{\varr}}
 \alicework{\varu = \varr\cdot\varg}
 \alicebob{}{\varu}{}
 \bobwork{\schnorrvalidate(\varu, \varh)}
 \bobseparator
 \bobwork{\sample{\varc}}
 \bobalice{}{\varc}{}
 \alicework{\varz = \varr + \varx\cdot \varc}
 \alicebob{}{\varz}{}
 \bobwork{\varz \neq 0 \mod \varq}
 \bobseparator
 \bobwork{\varz \cdot \varg \equalQ \varu + \varc\cdot\varh}
 \end{array}
 $$



{{< /rawhtml >}}

-----

### Non-interactive protocol
We can transform this identification scheme into a non-interactive protocol using the Fiat-Shamir heuristic. Here, the prover creates the random challenge $c$ hashing *all* public values $\\{\varg, \varq, \varh, \varu\\}$.
{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\samplezqs{\varr}}
 \alicework{\varu = \varr\cdot \varg}
 \alicework{\varc = \hash{\varg, \varq, \varh, \varu}}
 \alicework{\varz = \varr + \varx\cdot \varc}
 \alicebob{}{\varu, \varc, \varz}{}
 \bobwork{\schnorrvalidate(\varu, \varh)}
 \bobwork{\varz \neq 0 \mod \varq}
 \bobseparator
 \bobwork{\varc \equalQ \hash{\varg, \varq, \varh, \varu}}
 \bobwork{\varz \cdot \varg \equalQ \varu + \varc\cdot\varh}
 \end{array}
 $$
{{< /rawhtml >}}
{{< /tab >}}
{{< /tabs >}}


where $\schnorrvalidate(\varu, \varh)$ aborts if any of the following conditions are not met:
 * $\varu, \varh \neq 0 \text{ (point at infinity for EC groups)}$ and
 * $\varu, \varh \inQ \cgroup\text{ (on curve check for EC groups)}$.

## Security pitfalls
 * **Verifier input validation:** Each of the items above the dotted line for the $\varverifier$ is essential to the security of the protocol. If any of these checks are missing or insufficient it is likely a severe security issue.
 * __Verifier trusts prover:__ On the verification check, the verifier uses $g$ and $q$ provided with the proof instead of using publicly known values. On the NI version, the verifier assumes that the hash $\varc$ is correctly computed and does not compute it themself. Both are high severity issues since $\varprover$ can forge proofs.
 * __Weak Fiat-Shamir transformation:__ In the non-interactive protocol, it is a common occurrence that some parameters are missing on the hash computation $\hash{\varg, \varq, \varh, \varu}$:
   * $\varh$ or $\varu$ missing: high severity issue. Read [Fiat-Shamir transformation]({{< ref "../protocol-primitives/fiat-shamir.md" >}}) for more details.
   * $\varg$ or $\varq$ missing: usually no issue, but it might be one if the Verifier uses these parameters directly from the proof structure. This way, the prover can provide bad generators or orders to forge the proof.
 * __Weak randomness:__ Bad randomness may cause the secret $\varx$ to leak. If $\varr$ is reused twice with two different interactive challenges, or different data on the non-interactive version then
  $$ \frac{\varz - \varz'}{\varc-\varc'} = \frac{\varr -\varr + \varx\cdot(\varc - \varc')}{\varc-\varc'} = \varx  $$
 * __Replay attacks:__ After a non-interactive proof is public, it will always be valid and anyone could pretend to know the secret value. To prevent this, consider adding the ID of both the prover and the verifier inside of the Fiat-Shamir hash computation.


## Security assumptions
 * __Hash function:__ The hash function should be either [TupleHash](https://www.nist.gov/publications/sha-3-derived-functions-cshake-kmac-tuplehash-and-parallelhash) or SHA-256 where each input is domain separated with a unique string together with the length of each element.
 * __Hardness of the discrete logarithm:__ The order of the cyclic group $\cgroup$ should be at least $\varq>2^{K}$ where $K=256$, for a generic group $\cgroup$. If $\cgroup$ is a (prime-order) subgroup of $\zps$, then $p$ should be greater than $2^{\kappa}$ for $\kappa=3072$ to avoid [subexponential attacks](https://en.wikipedia.org/wiki/Index_calculus_algorithm) based on the extra structure of $\zps$. Note that this requires $p - 1 = q\cdot r$ with some potentially composite number $r$. Refer to table 2 (pp. 54-55) of [the NIST recommendations](https://doi.org/10.6028/NIST.SP.800-57pt1r5) for an overview of different bit security levels for finite field discrete logarithms.

