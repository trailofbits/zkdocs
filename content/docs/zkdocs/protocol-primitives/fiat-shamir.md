---
weight: 2
bookFlatSection: true
title: "Fiat-Shamir transformation"
references: ["FSWiki", "FS87", "ZKBlog", "BPW12", "RAWiki"]
summary: "Here, we describe what the Fiat-Shamir transformation is, its goals, its pitfalls, and its different versions."
---
# Fiat-Shamir transformation

## Overview

It turns out that in practice, most zero-knowledge proofs tend to have the same three-step structure:
 1. The prover first generates some random value, the commitment, and sends it to the verifier.
 2. The verifier responds with a challenge value generated uniformly at random.
 3. The prover computes the final proof based on both the commitment and challenge.

As you can tell, this structure is interactive, meaning that the prover requires a response from the verifier before they can complete their proof, which is not ideal for most applications. Fortunately, provers can avoid this by using the [Fiat-Shamir heuristic](https://en.wikipedia.org/wiki/Fiat%E2%80%93Shamir_heuristic) (sometimes referred to as the Fiat-Shamir transformation), [developed by Amos Fiat and Adi Shamir](https://link.springer.com/content/pdf/10.1007%2F3-540-47721-7_12.pdf).

The idea behind the Fiat-Shamir transformation is that instead of having the verifier send a random challenge value to the prover, the prover can compute this value themselves by using a random function, such as a cryptographic hash function.

{{< hint info >}}
**Example:** In the [Schnorr protocol]({{< ref "../zero-knowledge-protocols/schnorr.md" >}}), we have an interactive and non-interactive version. In the interactive version, the prover sends their random commitment, $u = g^r$, and the verifier responds with a challenge value, $c$, that they generate uniformly at random. In the non-interactive version, the prover generates $c$ themselves by computing a hash over all of the public values, $c = \hash{g,q,h,u}$. As we will discuss, the hash must include all of these values.
{{< /hint >}}

If you'd like to read more about this, we've written a [blog post describing how they work in more detail](https://blog.trailofbits.com/2021/02/19/serving-up-zero-knowledge-proofs/).

## What can go wrong?

This transformation may seem straightforward, but unfortunately, it tends to be very tricky in practice. In particular, the prover generates the random challenge value using a cryptographic hash function- but what are the inputs? It turns out that if you choose the wrong inputs, it usually means your proof system is broken. To see this, let's look at [Schnorr's protocol]({{< ref "../zero-knowledge-protocols/schnorr.md" >}}) as an example.

{{< hint danger >}}
**Bad example, NEVER DO THIS:** Recall that in the [Schnorr protocol]({{< ref "../zero-knowledge-protocols/schnorr.md" >}}), the prover generates their random commitment, $u = g^r$, computes the challenge value, $c$, and then computes the final proof $z = r + x\cdot c$, where $x$ is the secret value corresponding to their public key, $h = g^x$. Let's say the implementation is incorrect, and the challenge value, $c$, is not computed using the public key, $h$, and instead it's computed as $c = \hash{g,q,u}$. It turns out that malicious provers can now forge proofs by doing the following attack:

 - Set the commitment value, $u$, to be some public key that you do not know the secret key for.
 - Compute the challenge value $c = \hash{g,q,u}$.
 - Set the proof $z$ to be a random value, and then compute your public key to be $h = (\frac{g^z}{u})^{\frac{1}{c}}$
 - Send $u,c,z$ to the verifier.

 The verifier then performs their two checks: $c \equalQ \hash{g,q,u}$ and $g^z \equalQ u\cdot h^c$. The first check will pass as they are both the same, and the second check will pass because of how we constructed $z$, $u$, and $h$.

Since we set $u$ to be some value we do not know the secret key for, we also will not know the secret key for $h$, which means we have forged our proof of knowledge.

**Note:** This attack does not allow you to forge proofs for any public key, this only works for random public keys. This tends not to be problematic when Schnorr proofs as a proof of knowledge in the typical public-key model. However, this can be very problematic when these proofs are used in other scenarios, such as a [cryptographic voting system](https://eprint.iacr.org/2016/771.pdf).
{{< /hint >}}

## Recommendations

### Rule of thumb

The exact inputs required for the hash function will be different for every zero-knowledge proof system. The general rule of thumb to follow is to include all of the public information and all elements in the proof transcript up until that point inside the hash function.

{{< hint info >}}
In the [Schnorr protocol]({{< ref "../zero-knowledge-protocols/schnorr.md" >}}), the public information is the public key, $h$, and the parameters related to the group being used, $g$ and $q$, and the proof transcript also includes the commitment value, $u$, and so we include all of those values in our hash computation.
{{< /hint >}}

{{< hint info >}}
For [short factoring proof protocol]({{< ref "../zero-knowledge-protocols/short-factoring-proofs.md" >}}), it's a bit more complicated. The public information is the number the prover can factor, $N$, so this must be included. What about transcript values? In this scheme, the prover does a bit more work before computing the challenge compared to the [Schnorr protocol]({{< ref "../zero-knowledge-protocols/schnorr.md" >}}). Specifically, they generate a random value ($r$), a series of values ($z_i$) using the [nothing-up-my-sleeve construction]({{< ref "nums.md" >}}), and then they compute $X = \hash{\bunchi{z_i^r \mod N}}$ (this is not the Fiat-Shamir challenge). Since both the $X$ value and the set of $z_i$ values will be known to the verifier (i.e., they are part of the transcript), these must also be included in the hash computation. So, the correct challenge computation is $e = \hash{N, \bunchi{z_i}, X}$.

**Note:** In the short factoring proof and in [Girault's scheme]({{< ref "../zero-knowledge-protocols/girault-identification" >}}), the Fiat-Shamir challenge needs to additionally be of a specific bit-size. So, the correct computation is actually $e = \hashbit{N, \bunchi{z_i}, X}{k}$.
{{< /hint >}}

**Remember:** Always include all public information and all transcript values. When in doubt, consult ZKDocs!

### Preventing replay attacks

Just like digital signatures, zero-knowledge proofs can also be susceptible to [replay attacks](https://en.wikipedia.org/wiki/Replay_attack). As is the case for other replay attacks, the severity of these replay attacks will be highly application and context-specific.

If you believe replay attacks could be severe for your application, you might be able to use the Fiat-Shamir transformation to protect yourself. Specifically, suppose your application has some notion of identity tied to each party (a unique party ID, for example). In that case, you should include the ID of the prover and the verifier inside the Fiat-Shamir hash computation. Then, when the verifier verifies the proof, they should also check that the IDs used in the hash function match the ID of themself and the prover. This will prevent other malicious parties from replaying any proof that they did not produce themselves.
