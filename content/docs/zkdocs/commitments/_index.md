---
weight: 9
bookFlatSection: true
title: "Commitment Schemes"
summary: "An overview of the commitment schemes and their applications"
references: ["Blum81"]
---

# Commitment Schemes Generally

Commitment schemes were introduced by Blum in 1981. Blum posed the following problem:

> Alice and Bob want to flip a coin by telephone. (They have just divorced, live in different cities, want to decide who gets the car.) Bob would not like to tell Alice HEADS and hear Alice (at the other end of the line) say "Here goes... I'm flipping the coin.... You lost!"

Blum's solution was a _commitment scheme_: a way for Bob to send his call of HEADS or TAILS ahead of time, but in a way that makes it impossible to change his mind later, and not to _reveal_ his choice until Alice has flipped the coin. Once Alice has flipped the coin, Bob can reveal his call to Alice to see who won the coin toss.

A commitment scheme has two phases:

  - A _commit_ phase, in which the committer generates the commitment value and shares it with others, and

  - An _open_ phase, when the committer reveals the committed value, and the commitment is verified by others

It's very important that Bob can't cheat Alice by revealing a fake call. It's also very important that Alice can't gain any information about Bob's call from his commitment. These are the two major features of a good commitment scheme:

  - _Hiding_: a commitment doesn't reveal anything about the committed value

  - _Binding_: it is not feasible to find two or more distinct values with the same commitment

Informally, you can think of hiding as what keeps Alice from cheating (since she can't get any information about Bob's call to use against him), and binding as what keeps Bob from cheating (since he can't "change his mind" and send a valid opening for $call'\neq call$ if Alice announces a coin toss result he doesn't like).

Lots of cryptographic protocols rely on this sort of exchange. For instance, this type of behavior is used in threshold signature schemes to allow a group of parties to securely generate an unbiased, uniformly random signing key.

To see an example of how this might be done using cryptography, consider the following process. Bob randomly selects $\sampleGeneric{call}{\{\coinheads,\cointails\}}$ as his call. Then he selects a 256-bit random value $r$ and computes, $c=\hmac{r}{call}$, then sends $c$ to Alice. Alice flips her coin, and announces either $\coinheads$ or $\cointails$. Bob then sends Alice $\left(r,call\right)$. Alice verifies that $c=\hmac{r}{call}$. If the two match, she accepts Bob's call as valid; otherwise, she rejects Bob's call as invalid.

We call $c$ Bob's _commitment_ to his call, and we call $\left(r,call\right)$ the _opening_ of $c$.

In protocol terms, we have:

{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varalice}{\varbob}
 \bobwork{\sampleGeneric{call}{\{\coinheads, \cointails\}}}
 \bobwork{\sampleGeneric{r}{\{0, 1\}^{256}}}
 \bobwork{c_{A}=\hmac{r}{call}}
 \bobalice{}{c_{A}}{}
 \alicework{\sampleGeneric{toss}{\{\coinheads, \cointails\}}}
 \alicebob{}{toss}{}
 \bobalice{}{\left(call,r\right)}{}
 \alicework{\hmac{r}{call}\equalQ c_{A} }
 \end{array}
 $$
{{< /rawhtml >}}

It's worth noting that simply setting $C=\hash{call}$ is problematic as a commitment mechanism. In our example, it would be trivial for Alice to compute $c_{\coinheads}=\hash{\coinheads}$ and $c_{\cointails}=\hash{\cointails}$ and compare them to Bob's value for $C$.

Similarly, in our $\mathsf{HMAC}$ scheme, if Bob doesn't select a fresh $r$ for each commitment, Alice can detect when Bob commits to the same value in different contexts.

Of course, the $\mathsf{HMAC}$ scheme isn't limited to simple $\coinheads/\cointails$ calls; it can be used for an arbitrary message $m$: $c=\hmac{r}{m}$.

This scheme will be both _hiding_ and _binding_ due to the properties of the $\mathsf{HMAC}$ function. Without $r$, Alice has no way of checking if $c$ commits to $\coinheads$ or $\cointails$, and it's computationally infeasible for Bob to find $r'$ such that $c=\hmac{r'}{call'}$ (where $call'\neq call$).

It's possible to create a commitment scheme that is hiding but not binding. Suppose the commitment $c\_{A}$ above only included the lowest 16 bits of the HMAC output. It would take a fraction of a second for Bob to find $r'\neq r$ such that $c=\hmac{r'}{call'}$.

Conversely, it's possible to have a commitment scheme that is binding but not hiding. The easiest example would be for Bob to simply send $call$ to Alice, as in the original, insecure coin flip protocol. Bob can't change his mind after sending $c$, but nothing is hidden from Alice.

Some specialized commitment schemes provide features beyond simple binding and hiding. The KZG polynomial commitment scheme, for instance, enables Bob to commit to a polynomial $f$ while allowing him to prove that a pair $\left(x, y=f\left(x\right)\right)$ represents a correct evaluation of $f$.

{{<section>}}
