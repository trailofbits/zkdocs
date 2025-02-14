---
weight: 1
bookFlatSection: true
title: "A title"
summary: "A summary that ends in a period."
needsVariableResetButton: true
draft: true
references: []
---
# A title


{{< hint info >}}
**Goal:**
$\varprover$ convinces $\varverifier$ that they know
{{< /hint >}}

 * __Public input:__ cyclic group $\cgroup$ of prime order $\varq$ and a generator $\varg\in \cgroup$
 * __Private input:__ $\varprover$ knows a secret


### Interactive protocol

{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\samplezqs{\varr}}
 \alicework{\varu = \varg^\varr}
 \alicebob{}{\varu}{}
 \bobwork{\sample{\varc}}
 \bobalice{}{\varc}{}
 \alicework{\varz = \varr + \varx\cdot \varc}
 \alicebob{}{\varz}{}
 \bobwork{\varg^{\varz} \equalQ \varu \cdot \varh^\varc }
 \end{array}
 $$

{{< /rawhtml >}}
-----

### Non-interactive protocol
We can transform this identification scheme into a non-interactive protocol using the Fiat-Shamir heuristic, where the prover
creates itself the random challenge $c$. To do this, we need to hash together *all* elements of the public set $\\{\varg, \varq, \varh\\}$ as well as the public values of the interaction.
{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varprover}{\varverifier}
 \alicework{\samplezqs{\varr}}
 \alicework{\varu = \varg^\varr}
 \alicework{\varc = \hash{\varg, \varq, \varh, \varu}}
 \alicework{\varz = \varr + \varx\cdot \varc}
 \alicebob{}{\varu, \varc, \varz}{}
 \bobwork{\varc \equalQ \hash{\varg, \varq, \varh, \varu}}
 \bobwork{\varg^\varz \equalQ \varu \cdot \varh ^\varc }
 \end{array}
 $$
{{< /rawhtml >}}



## Security assumptions


## Security pitfalls

