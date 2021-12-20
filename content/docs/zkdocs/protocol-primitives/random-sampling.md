---
weight: 1
bookFlatSection: true
summary: "In this section, we describe how to uniformly sample from different groups using rejection sampling."
---
# Random sampling

In most protocols, it is required to sample uniformly from groups like $\zq$ or $\zqs$. Here we will describe how to sample in these specific groups, and a general procedure to sample via rejection sampling.

## Uniform sampling in $\zq$
Since we identify the elements of $\zq$ with the set of integers $\range{q}$, to uniformly sample
we only need to sample from the set $\range{q}$.

We should always use a suitable cryptographic random number library to sample uniformly. For instance,
in python, we can use [secrets](https://docs.python.org/3/library/secrets.html).

```python
import secrets

def sample_Zq():
    return secrets.randbelow(q)
```
{{< hint danger >}}
**Never do  this:** When you only have access to sampling elements of a fixed bit-size but want to sample over $\z{1337}$ do not do this:
```python
def wrong_sampling():
    return sample_32_bits() % 1337
```
The `wrong_sampling` function introduces *modulo-bias* in the generation of elements, and some elements will be more likely to appear than others. In these cases you need to use [rejection sampling](#sampling-from-any-set-via-rejection-sampling).
{{< /hint >}}



## Rejection sampling
Suppose we know how to sample from a set $S$ and would like to sample from a subset $T\subseteq S$.
To do this, we need to check membership in $T$ and perform rejection sampling. In a loop:
 1. sample element $e \in S$.
 2. if $e \in T$, return $e$, otherwise repeat step 1.



This is how we must perform sampling in $\zq$ if we cannot use a library like `secrets`.
In this case, rejection sampling would look like this:
```python
def correct_sampling():
    while True:
        e = sample_32_bits()
        if e >= 0 and e < 1337:
            return e
```

Depending on the set we can sample from, this method can take longer, but in practice, we should always be able to sample from spaces only slightly larger than our target space.



## Uniform sampling in $\zqs$
Sampling in $\zqs$ requires an extra step. The elements of $\zqs$ are the elements invertible modulo $q$ and for $e$ to be invertible means that $\gcd(e, q) = 1$.
So, to obtain these elements we need to perform rejection sampling. In a loop,
 1. sample element $e \in \zq$ using [the previous section](#uniform-sampling-in-zq).
 2. if $\gcd(e, q) = 1$ return $e$, otherwise repeat step 1.


```python
import secrets
import math

def sample_Zqstar():
    while True:
        e = sample_Zq()
        if math.gcd(e, q) == 1:
            return e
```

{{< hint info >}}
**Note:**
Since $\gcd(0, q)$ is never 1, we can sample $e$ from the set $\rangeone{q}$.
{{< /hint >}}
