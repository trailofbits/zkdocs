---
title: Introduction
type: docs
---

# ZKDocs

## Zero-knowledge protocols

ZKDocs provides comprehensive, detailed, and interactive documentation on zero-knowledge proof systems and related primitives.

At [Trail of Bits](https://www.trailofbits.com/), we audit many implementations of non-standardized cryptographic protocols and often find the same issues. As we discovered more instances of these bugs, we wanted to find a way to prevent them in the future. Unfortunately, for these protocols, the burden is on the developers to figure out all of the low-level implementation details and security pitfalls.

We hope that ZKDocs can fill in this gap and benefit the larger cryptography community.

{{< columns >}}

### Comprehensive
We aim to be both self-contained and comprehensive in the topics related to zero-knowledge proof systems, from descriptions of simple systems like [Schnorr’s identification protocol]({{< relref "/docs/zkdocs/zero-knowledge-protocols/schnorr" >}}), to complex proof systems like [Paillier-Blum modulus]({{< relref "/docs/zkdocs/zero-knowledge-protocols/product-primes/paillier_blum_modulus" >}}). We also cover cryptographic primitives such as: [random sampling]({{< relref "/docs/zkdocs/protocol-primitives/random-sampling" >}}), [Fiat-Shamir transformation]({{< relref "/docs/zkdocs/protocol-primitives/fiat-shamir" >}}), and [Shamir's Secret Sharing]({{< relref "/docs/zkdocs/protocol-primitives/shamir" >}}).

<--->

### Detailed
We describe each protocol in great detail, including all necessary setup, sanity-checks, auxiliary algorithms, further references, and potential security pitfalls with their associated severity.

{{< /columns >}}


### Interactive

The protocol descriptions are interactive, letting you modify variable names. This allows you to match the variable names in ZKdocs' specification to the variable names in your code, making it easier to find bugs and missing assertions.

Interactivity features:
 - Click on $\varX$ to highlight the variable across the document. Try it!
 - Type or paste with $\varX$ highlighted to edit $\varX$'s name. Press `Enter` or `Escape` to stop editing.
 - Press the {{< rawhtml >}}<button onclick="resetVariableNames()">Reset variables names</button>{{< /rawhtml>}} button to reset the names of all variables on the current page (variable names are independent across different pages)


{{< box >}}
![Basic interactivity usage](/figs/demo.gif)
{{< /box >}}


### Contribute
We will continue to add more proof systems like Range proofs, STARK, and Bulletproofs.

Feel free to [open issues](https://github.com/trailofbits/zkdocs/issues) and suggest new protocols that you would like to see added in [ZKDocs](https://github.com/trailofbits/zkdocs).