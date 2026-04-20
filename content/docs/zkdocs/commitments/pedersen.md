---
weight: 9
bookFlatSection: true
title: "Pedersen Commitments"
summary: "An overview of the Pedersen commitment scheme and its applications"
references: ["Pedersen91", "ANSIX962"]
---

# Pedersen Commitments

Pedersen commitments were originally described as part of Pedersen's verifiable secret-sharing scheme in 1992. Section 3 of the Pedersen paper notes that the commitment scheme is "very similar" to a scheme proposed by Bos, Chaum, and Purdy in an unpublished set of notes proposing a voting system.

Pedersen commitments are simple. Committing to a value requires only two modular exponentiations and a multiplication. Because a random value is required for each commitment, commitments are non-deterministic. And, because Pedersen commitments rely heavily on group structure, they inherit a large number of useful properties that allow for useful operations and proofs for the committed values.

Pedersen's original proposal described the commitment in terms of order-$q$ subgroups of $\zns{p}$, but we will describe it in more generic terms, as it is applicable to any group $G$ where the discrete logarithm problem is believed to be hard (such as elliptic curve groups).


## Definition

Start with a finite group $G$ of prime order $q$, where $q$ is suitably large. For instance, we can use an order-$q$ subgroup of the multiplicative group $\zps$ where $q$ divides $p$, or an elliptic curve group or subgroup. Select two generators $g,h$ of $G$ such that $\log\_{g}\left(h\right)$ is not known. The parameters of the commitment scheme are $\left(G,q,g,h\right)$.

To generate a commitment $c\in G$ to a secret integer $s$, where $0 < s < q$, randomly sample $\sampleN{t}{\varq}$ and compute:

$$
    c = C\_{g,h}\left(s,t\right)=g^{s}h^{t}
$$

(When $g,h$ are clear from context, we can simply write $C\left(s,t\right)$.)

To open the commitment, simply reveal $s$ and $t$.

### Intuition as to Why Pedersen Commitments are Binding

Suppose Alice has sent Bob a commitment $c=C\left(s,t\right)$ to a value $s$. She now wants to cheat by sending Bob an opening for $c$ that corresponds to $s'\neq s$. In other words, she wants to break the binding property of Pedersen commitments.

That means Alice has to find $t'$ such that $g^{s'}h^{t'}=c$. Since $g^{s'}$ and $c$ are already fixed, Alice is left to solve $h^{t'}=cg^{-s'}$. In other words, she needs to compute $t'=\log\_{h}\left(cg^{-s'}\right)$. Since the discrete logarithm problem is supposed to be hard in $G$, Alice will have a difficult time finding $t'$.

Before committing to a value, Alice can also look for distinct pairs $\left(s,t\right),\left(s',t'\right)$ such that $C\left(s,t\right)=C\left(s',t'\right)$, without regard for the specific values of $s$ and $s'$. This is _also_ equivalent to computing a discrete logarithm in $G$. As the Pedersen paper points out: given $C\left(s,t\right)=C\left(s',t'\right)$, where $s'\neq s$, then $t'\neq t$, and it is possible to compute the discrete logarithm of $h$ with respect to $g$:

$$
    \log\_{g}\left(h\right)=\frac{s-s'}{t'-t}\pmod{q}
$$

To summarize, breaking the binding property of Pedersen commitments is equivalent to solving the discrete log problem.

### Intuition as to Why Pedersen Commitments are Hiding

Suppose Alice has sent Bob a commitment $c=C\left(s,t\right)$ to a value $s$. Bob wants to cheat by figuring out $s$, or at least something _about_ $s$, from $c$.

Remember that $t$ is sampled uniformly at random, and so every value of $t$ is equally likely to be chosen, which means every value of $h^{t}$ is equally likely. Since $h$ is a generator of $G$, that means that $t$ "selects" every element of $G$ uniformly at random.

If Bob is able to gain any advantage in guessing the value of $s$ or $g^{s}$ from $c$, then he necessarily gains an advantage in guessing $t$ or $h^{t}$. However, $t$ is selected uniformly at random, so $h^{t}$ is a uniform random element of $G$. You can't gain information about a uniform random distribution.


### The Additive Property

Given two commitments, $c\_{1}=C\left(s\_{1},t\_{2}\right)=g^{s\_{1}}h^{t\_{1}}$ and $c\_{2}=C\left(s\_{2}, t\_{2}\right)=g^{s\_{2}}h^{t\_{2}}$, the product of $c\_{1}$ and $c\_{2}$ is $g^{s\_{1}+s\_{2}}h^{t\_{1}+t\_{2}} = C\left(s\_{1}+s\_{2},t\_{1}+t\_{2}\right)$. That is, the product of any two Pedersen commitments is a commitment to the sum of the committed values.

This also allows for scalar multiplication through exponentiation; $C\left(x,y\right)^{n}=C\left(nx,ny\right)$.

This additive property can be incredibly useful. It allows for a variety of operations to be performed on committed values before revealing them. Systems like Bulletproofs, for instance, exploit this property of the Pedersen scheme to commit to entire _vectors_ of values at once, aggregating the commitments into a single value.

## Useful Algorithms and Properties of Pedersen Commitments

### Proof of Knowledge of Secret

Given a commitment $A=C\_{g,h}\left(x,y\right)=g^{x}h^{y}$, Alice can demonstrate to Bob that she knows $x$ and $y$ without revealing either $x$ or $y$. This is a useful protocol for protecting against the additive properties of Pedersen commitments being used maliciously. The protocol works as follows:

{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varalice}{\varbob}
 \alicework{\sampleN{t_{1},t_{2}}{\varq}}
 \alicework{T = g^{t_{1}}h^{t_{2}}}
 \alicebob{}{T}{}
 \bobwork{\sampleN{k}{\varq}}
 \bobalice{}{k}{}
 \alicework{s_{1}=x \cdot k+t_{1},s_{2}=y \cdot k+t_{2}}
 \alicebob{}{s_{1},s_{2}}{}
 \bobwork{g^{s_{1}}h^{s_{2}} \equalQ A^{k}T }
 \end{array}
 $$
{{< /rawhtml >}}

This works because:

$$
    g^{s\_{1}}h^{s\_{2}}=g^{xk+t\_{1}}h^{yk+t\_{2}}=g^{xk}h^{yk}g^{t\_{1}}h^{t\_{2}}=\left(g^{x}h^{y}\right)^{k}T=A^{k}T
$$

This is secure because, in order to trick Bob, Alice needs to break the discrete log problem.

A straightforward application of the Fiat-Shamir transform can turn this into a non-interactive proof. If Alice selects $k=\hash{g\Vert h\Vert A\Vert T}$, where the output of $\hash{}$ has the same bit length as $q$, she can proceed as though $k$ were selected uniformly at random. Bob can compute $k$ in the same way, and verify the rest of the proof from there.

### An Easy Proof of Equal Commitments

Given two commitments to a secret $s$ using the same set of generators, call them $c\_{1}=C\left(s,t\_{1}\right)$ and $c\_{2}=C\left(s,t\_{2}\right)$, where $t\_{2}\neq t\_{1}$, Alice can easily show that both $c_{1}$ and $c_{2}$ commit to the same value. The simplest way, given by Pedersen, is for Alice to reveal $r=t\_{1}-t\_{2}\pmod{q}$ to Bob.

This works because Bob can check that

$$
    c\_{1}c\_{2}^{-1}=g^{s}h^{t\_{1}}g^{-s}h^{-t\_{2}}=g^{s-s}h^{t\_{1}-t\_{2}}=h^{t\_{1}-t\_{2}}=h^{r}
$$

### Proof of Equal Commitments with Different Binding Generators

Suppose Alice has $c\_{1}=C\_{g\_{1},h}\left(s,t\_{1}\right)$ and $c\_{2}=C\_{g\_{2},h}\left(s,t\_{2}\right)$, possibly with $g\_{1}\neq g\_{2}$. Alice can prove to Bob that $c\_{1}$ and $c\_{2}$ commit to the same value.

Alice starts by selecting random integers $\sampleN{r\_{1},r\_{2},r\_{3}}{\varq}$, then computing $c\_{3}=C\_{g\_{1},h}\left(r\_{1},r\_{2}\right)=g\_{1}^{r\_{1}}h^{r\_{2}}$ and $c\_{4}=C\_{g\_{2},h}\left(r\_{1},r\_{3}\right)=g\_{2}^{r\_{1}}h^{r\_{3}}$. She sends $c\_{3},c\_{4}$ to Bob.

Bob selects a random challenge value $\sampleN{k}{\varq}$ and sends it to Alice.

Alice computes $z\_{1}=ks+r\_{1}$, $z\_{2}=kt\_{1}+r\_{2}$, and $z\_{3}=kt\_{2}+r\_{3}$ in $\mathbb{Z}\_{q}$ and sends $z\_{1},z\_{2},z\_{3}$ to Bob.

Bob then checks that $c\_{3}c_{1}^{k}=C\_{g\_{1},h}\left(z\_{1},z\_{2}\right)$ and $c\_{4}c\_{2}^{k}=C\_{g\_{2},h}\left(z\_{1},z\_{3}\right)$. If the condition holds, Bob accepts the proof as valid.

This works because:

$$
    c\_{3}c\_{1}^{k}=\left(g\_{1}^{r\_{1}}h^{r\_{2}}\right)\left(g\_{1}^{s}h^{t\_{1}}\right)^{k}=g\_{1}^{r\_{1}}h^{r\_{2}}g\_{1}^{ks}h^{kt\_{1}}=g\_{1}^{ks+r\_{1}}h^{kt\_{1}+r\_{2}}=g\_{1}^{z\_{1}}h^{z\_{2}}=C\_{g\_{1},h}\left(z\_{1},z\_{2}\right)
$$

and 

$$
    c\_{4}c^{k}\_{2}=\left(g\_{2}^{r\_{1}}h^{r\_{3}}\right)\left(g\_{2}^{s}h^{t\_{2}}\right)^{k}=g\_{2}^{r\_{1}}h^{r\_{3}}g\_{2}^{ks}h^{kt\_{2}}=g\_{2}^{ks+r\_{1}}h^{kt\_{2}+r\_{3}}=g\_{2}^{z\_{1}}h^{z\_{3}}=C\_{g\_{2},h}\left(z\_{1},z\_{3}\right)
$$

#### In diagram form:
{{< rawhtml >}}
 $$
 \begin{array}{c}
 \work{\varalice}{\varbob}
 \alicework{\sampleN{r_{1},r_{2},r_{3}}{\varq}}
 \alicework{c_{3}=C_{g_{1},h}\left(r_{1},r_{2}\right),c_{4}=C_{g_{2},h}\left(r_{1},r_{3}\right)}
 \alicebob{}{c_{3},c_{4}}{}
 \bobwork{\sampleN{k}{\varq}}
 \bobalice{}{k}{}
 \alicework{z_{1}=ks+r_{1}\\ z_{2}=kt_{1}+r_{2}\\ z_{3}=kt_{2}+r_{3}}
 \alicebob{}{z_{1},z_{2},z_{3}}{}
 \bobwork{c_{3}c_{1}^{k}\equalQ C\left(z_{1},z_{2}\right)}
 \bobwork{c_{4}c_{2}^{k}\equalQ C\left(z_{1},z_{3}\right)}
 \end{array}
 $$
{{< /rawhtml >}}

Again, this can be made non-interactive through the use of a Fiat-Shamir transform. Alice can use $k=\hash{g\_{1}\Vert g\_{2}\Vert h\Vert c\_{1}\Vert c\_{2}\Vert c\_{3}\Vert c\_{4}}$, again selecting a hash so that $k$ and $q$ have the same bit length.

While this construction is more complicated, it has the benefit of allowing for commitments to be made when $g\_{1}\neq g\_{2}$.

### Proof of Squared Commitments

We can use the equality of commitments proof above to commit to both a secret $s$ and its square.

Suppose Alice has a random $t\_{1}$ and corresponding commitment $c\_{1}=C\_{g,h}\left(s,t\_{1}\right)=g^{s}h^{t\_{1}}$. Alice can select a random $t\_{2}$ and compute $c\_{2}=C\_{c\_{1},h}\left(s,t\_{2}\right)=\left(c\_{1}\right)^{s}h^{t\_{2}}$


Note that $c\_{2}=C\_{c\_{1},h}\left(s,t\_{2}\right)=\left(c\_{1}\right)^{s}h^{t\_{2}}=\left(g^{s}h^{t\_{1}}\right)^{s}h^{t\_{2}}=g^{s^{2}}h^{st\_{1}+t\_{2}}=C\_{g,h}\left(s^{2},st\_{1}+t\_{2}\right)$, so $c\_{2}$ is a commitment to $s^{2}$ in base $g$.

Once $c\_{1}$ has been generated, Alice can easily generate $c\_{2}$, provide both values to Bob, and use the equality proof above to demonstrate that $c\_{2}$ commits to the square of the value committed to by $c\_{1}$.

## Security Considerations

### Generator Selection

Suppose that Alice knows $l=\log\_{g}\left(h\right)$ and has a commitment $c=C\left(s,t\right)=g^{s}h^{t}$.

Then Alice can rewrite the commitment as $c=g^{s}h^{t}$ as $c=g^{s}\left(g^{l}\right)^{t}=g^{s}g^{tl}=g^{s+tl}$. Let $s'=s+l$ and $t'=t-1$. She can the fraudulently "open" the commitment $c$ by sending $\left(s',t'\right)$. This will appear as a valid opening to Bob, since:

$$
    g^{s'}h^{t'}=g^{s+1}\left(g^{l}\right)^{t - 1}=g^{s+l}g^{l\left(t-1\right)}=g^{s+l}g^{tl-l}=g^{s+tl}=g^{s+tl}=c
$$

If the discrete logarithm of $h$ with respect to $g$ is known to Alice (or if an insecure group is chosen that makes computing the discrete log easy), _the binding property is lost_.

This is why it is important to ensure that generators $g$ and $h$ are selected independently. Ideally, $g$ and $h$ should be chosen independently from one another using a nothing-up-my-sleeve construction. For instance, $g$ and $h$ can be selected using an agreed-upon PRF and algorithm D.3.1 from the ANSI X9.62 standard (“Finding a Point on an Elliptic Curve”).

### Random Number Generator Issues

In selecting the hiding value $\sampleN{t}{\varq}$, it is important that $t$ be truly uniform.

Suppose there is an attack on the process used to generate $t$, and we have a set of commitments $c\_{1}, c\_{2},\ldots,c\_{n}$ to secrets $s\_{1},s\_{2},\ldots,s\_{n}$, not necessarily distinct. An attacker can then attack each $h^{t\_{i}}$ and recover the corresponding values for $g^{s\_{i}}$. From this, the attacker can quickly identify repeated commitments to the same value. In cases where commitments are suspected of having a simple linear relationship (e.g. $s\_{i}=k\cdot s\_{j}$ or $s\_{i}=s\_{j} + k$ for some _known_ value of $k$), these relationships can be recovered as well by examining $g^{s\_{i}}g^{k}$ and $\left(g^{s\_{i}}\right)^{k}$.

