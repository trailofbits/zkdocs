---
weight: 1
bookFlatSection: true
title: "Variants of Schnorr's identification protocol"
summary: "Common variants of Schnorr's protocol."
needsVariableResetButton: true
references: ["schnorr_rfc"]
---
# Variants of Schnorr's identification protocol
All variants of Schnorr's protocol prove the knowledge of an $\varx$ such that $\varh = \varg^\varx$ for a public $\varh$.

The differences are tradeoffs between the size of the proof statements and a more costly proof verification, and smaller communication overhead.

{{< hint info >}}
**Goal:**
$\varprover$ convinces $\varverifier$ that they know $\varx$ such that $\varh = \varg^\varx$
{{< /hint >}}

 * __Public input:__ cyclic group $\cgroup$ of prime order $\varq$, a $\cgroup$ generator $\varg$ and $\varh\in \cgroup$.
 * __Private input:__ $\varprover$ knows secret $\varx\in\zq$ such that $\varh = \varg^\varx$.

We present all variants in their non-interactive version.

## Non-interactive protocols

### Original
{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\sampleNs{\varr}{\varq}}
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

-----
### Slim Original
In this variant, the prover does not send the value of $\varc$ which has to be computed by the verifier.
{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\sampleNs{\varr}{\varq}}
 \alicework{\varu = \varg^\varr}
 \alicework{\varc = \hash{\varg, \varq, \varh, \varu}}
 \alicework{\varz = \varr + \varx\cdot \varc}
 \alicebob{}{\varu, \varz}{}
 \bobwork{\schnorrvalidate(\varu, \varh)}
 \bobwork{\varz \neq 0 \mod \varq}
 \bobseparator
 \bobwork{\bar{\varc} = \hash{\varg, \varq, \varh, \varu}}
 \bobwork{\varg^\varz \equalQ \varu \cdot \varh ^\overline{\varc} }
 \end{array}
 $$
{{< /rawhtml >}}

-----

### Subtract
In this variant, the prover uses a subtraction to compute the value of $\varz$.

{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\sampleNs{\varr}{\varq}}
 \alicework{\varu = \varg^\varr}
 \alicework{\varc = \hash{\varg, \varq, \varh, \varu}}
 \alicework{\varz = \varr - \varx\cdot \varc}
 \alicebob{}{\varu, \varc, \varz}{}
 \bobwork{\schnorrvalidate(\varu, \varh)}
 \bobwork{\varz \neq 0 \mod \varq}
 \bobseparator
 \bobwork{\varc \equalQ \hash{\varg, \varq, \varh, \varu}}
 \bobwork{\varg^\varz \cdot \varh ^\varc \equalQ \varu  }
 \end{array}
 $$
{{< /rawhtml >}}

-----

### Subtract + Derive
In this variant, the prover uses a subtraction to compute the value of $\varz$ and does not send $\varu$, which the verifier derives from the values they know.

{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\sampleNs{\varr}{\varq}}
 \alicework{\varu = \varg^\varr}
 \alicework{\varc = \hash{\varg, \varq, \varh, \varu}}
 \alicework{\varz = \varr - \varx\cdot \varc}
 \alicebob{}{\varc, \varz}{}
 \bobwork{\schnorrvalidate(\varh)}
 \bobwork{\varz \neq 0 \mod \varq}
 \bobseparator
 \bobwork{\bar{\varu} = \varg^\varz \cdot \varh^\varc}
 \bobwork{\bar{\varc} = \hash{\varg, \varq, \varh, \bar{\varu}}}
 \bobwork{\varc \equalQ \bar{\varc} }
 \end{array}
 $$
{{< /rawhtml >}}


where $\schnorrvalidate(\varh)$ aborts if any of the following conditions are not met:
 * $\varh \neq 0 \text{ (point at infinity for EC groups)}$ and
 * $\varh \inQ \cgroup\text{ (on curve check for EC groups)}$. If called with two arguments, $\schnorrvalidate(\varu, \varh)$ will abort if any of the following conditions are not met by either argument.


## Security pitfalls
These variants suffer from the same pitfalls as the original Schnorr scheme, with some adjustments when $\varz$ is computed with a subtraction:
 * **Verifier input validation:** Each of the items above the dotted line for the $\varverifier$ is essential to the security of the protocol. If any of these checks are missing or insufficient it is likely a severe security issue.
 * __Verifier trusts prover:__
   * $\varverifier$ uses $g$ and $q$ provided in the proof instead of using publicly known values.
   * When the $varprover$ sends $\varc$, if the $\varverifier$ assumes that the hash $\varc$ is correctly computed and does not compute it themself. Both are high severity issues since $\varprover$ can forge proofs.
 * __Weak Fiat-Shamir transformation:__ It is a common issue that some parameters are missing on the hash computation $\hash{\varg, \varq, \varh, \varu}$:
   * $\varh$ or $\varu$ missing: high severity issue. Read [Fiat-Shamir transformation]({{< ref "../protocol-primitives/fiat-shamir.md" >}}) for more details.
   * $\varg$ or $\varq$ missing: usually no issue, but it might be one if the Verifier uses these parameters directly from the proof structure. This way, the prover can provide bad generators or orders to forge the proof.
 * __Weak randomness:__ Bad randomness may cause the secret $\varx$ to leak. If $\varr$ is reused twice with two different non-interactive challenges then:
  $$ \frac{\varz - \varz'}{\varc-\varc'} = \frac{\varr -\varr + \varx\cdot(\varc - \varc')}{\varc-\varc'} = \varx  \enspace,$$
  or when $\varz$ is computed with a subtraction:
  $$ \frac{\varz - \varz'}{\varc'-\varc} = \frac{\varr -\varr + \varx\cdot(\varc' - \varc)}{\varc'-\varc} = \varx  \enspace.$$
 * __Replay attacks:__ After a non-interactive proof is public, it will always be valid and anyone could pretend to know the secret value. To prevent this, consider adding the ID of both the prover and the verifier inside of the Fiat-Shamir hash computation.
