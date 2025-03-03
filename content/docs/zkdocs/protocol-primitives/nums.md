---
weight: 4
bookFlatSection: true
title: "Nothing-up-my-sleeve constructions"
summary: "Generic, honest, and deterministic method to sample elements."
references: ["shake"]
---
# Nothing-up-my-sleeve constructions

Protocols often require using public but random group elements or alternative generators. If these were simply randomly sampled, other participants could be suspicious that the party did not honestly generate them.

To generate the elements you require:
- **membership:** know how to check membership in the sample space
- **bit-size:** know the bit-size of the elements
- **binding parameters:** the protocol's public values other binding parameters like an index
- **salt**

{{< hint info >}}
**Example:** In the [Short factoring proofs]({{< relref "/docs/zkdocs/zero-knowledge-protocols/short-factoring-proofs#non-interactive-protocol" >}}) non-interactive protocol, we need to generate values $\varz_i \in \zns{\varN}$. Our ingredients are:
- **membership:** to check if an element $e$ is in $\zns{\varN}$, we check that $\gcd(e, \varN) = 1$
- **bit-size:** the element bit-size will be $|\varN|$
- **binding parameters:** $\varN$ and $i$
- **salt:** the salt can be $\mathsf{"shortfactoringproof"}$
{{< /hint >}}

The construction is generic: start a counter and sample an element with an extendable-output hash function over the binding parameters and the counter; if the generated element is in the sample space, we return it; otherwise, we increment the counter and sample again.

```python
DS = '|'

# Rejection sampling using a counter and binding parameters of the protocol
def gen_element(sample_space_predicate, bitsize, param_list, salt):
  counter = 0
  while True:

    # elements to hash
    elements = param_list + [salt, counter]

    # add each element's length
    to_hash = [len(s) + s for s in elements]

    # domain separated elements
    to_hash = DS + DS.join(to_hash)

    # generate bitsize element with extendable-output hash function
    e = hash_xof(to_hash, bitsize)

    # check that e is in sample_space
    if sample_space_predicate(e):
      return e

    # increment counter
    counter += 1
```

## Hash function choice
The hash function used here must be an extendable-output function (XOF). These hash functions can generate an output of arbitrary length. We must use these functions since the elements we are sampling can have an arbitrary number of bits, and failing to achieve that size can cause security issues. An example is $\mathsf{SHAKE256}$, which provides 256-bit security against collision attacks when at least 512 bits are sampled.

An alternative is using $\mathsf{TupleHash}$ which does not require manually domain-separating the elements to be hashed.

