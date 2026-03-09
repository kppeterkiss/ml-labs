## Full numeric Variable Elimination example

I’ll use the **student network** and compute:

```python
P(G)
```


because it shows the mechanics clearly without too much evidence bookkeeping.

---

# 1. The network and CPDs

We have variables:

- `D` = Difficulty: Easy, Hard
- `I` = Intelligence: Dumb, Intelligent
- `G` = Grade: A, B, C
- `L` = Letter: Bad, Good
- `S` = SAT: Bad, Good

Structure:

```plain text
D -> G <- I
G -> L
I -> S
```


Factorization:

```python
P(D, I, G, L, S) = P(D) P(I) P(G | D, I) P(L | G) P(S | I)
```


From your notebook, the CPDs are:

## Priors

```python
P(D=Easy) = 0.6
P(D=Hard) = 0.4
```


```python
P(I=Dumb) = 0.7
P(I=Intelligent) = 0.3
```


---

## Grade CPD

I’ll rewrite the table in a more readable form.

### For `I=Dumb, D=Easy`
```python
P(G=A | I=Dumb, D=Easy) = 0.3
P(G=B | I=Dumb, D=Easy) = 0.4
P(G=C | I=Dumb, D=Easy) = 0.3
```


### For `I=Dumb, D=Hard`
```python
P(G=A | I=Dumb, D=Hard) = 0.05
P(G=B | I=Dumb, D=Hard) = 0.25
P(G=C | I=Dumb, D=Hard) = 0.7
```


### For `I=Intelligent, D=Easy`
```python
P(G=A | I=Intelligent, D=Easy) = 0.9
P(G=B | I=Intelligent, D=Easy) = 0.08
P(G=C | I=Intelligent, D=Easy) = 0.02
```


### For `I=Intelligent, D=Hard`
```python
P(G=A | I=Intelligent, D=Hard) = 0.5
P(G=B | I=Intelligent, D=Hard) = 0.3
P(G=C | I=Intelligent, D=Hard) = 0.2
```


---

## Other CPDs

### Letter given grade

```python
P(L=Bad | G=A) = 0.1
P(L=Good | G=A) = 0.9

P(L=Bad | G=B) = 0.4
P(L=Good | G=B) = 0.6

P(L=Bad | G=C) = 0.99
P(L=Good | G=C) = 0.01
```


### SAT given intelligence

```python
P(S=Bad | I=Dumb) = 0.95
P(S=Good | I=Dumb) = 0.05

P(S=Bad | I=Intelligent) = 0.2
P(S=Good | I=Intelligent) = 0.8
```


---

# 2. The query

We want:

```python
P(G)
```


So we need to eliminate:

```python
D, I, L, S
```


and keep:

```python
G
```


---

# 3. Start from the full expression

```python
P(G) = Σ_D Σ_I Σ_L Σ_S P(D) P(I) P(G | D, I) P(L | G) P(S | I)
```


Now VE says: don’t compute all combinations at once. Eliminate variables one by one.

Let’s use this elimination order:

```python
L -> S -> D -> I
```


This is a good choice because `L` and `S` are leaves.

---

# 4. Eliminate `L`

The only factor involving `L` is:

```python
P(L | G)
```


So we compute:

```python
φ1(G) = Σ_L P(L | G)
```


Let’s do it numerically.

For `G=A`:

```python
0.1 + 0.9 = 1
```


For `G=B`:

```python
0.4 + 0.6 = 1
```


For `G=C`:

```python
0.99 + 0.01 = 1
```


So:

```python
φ1(G) = 1
```


for every value of `G`.

So `L` disappears.

---

# 5. Eliminate `S`

The only factor involving `S` is:

```python
P(S | I)
```


Compute:

```python
φ2(I) = Σ_S P(S | I)
```


For `I=Dumb`:

```python
0.95 + 0.05 = 1
```


For `I=Intelligent`:

```python
0.2 + 0.8 = 1
```


So:

```python
φ2(I) = 1
```


for every value of `I`.

So `S` also disappears.

---

# 6. What remains?

Now the problem is reduced to:

```python
P(G) = Σ_D Σ_I P(D) P(I) P(G | D, I)
```


That is already much smaller.

---

# 7. Eliminate `D`

Now combine the factors containing `D`:

- `P(D)`
- `P(G | D, I)`

Define:

```python
ψ(I, G) = Σ_D P(D) P(G | D, I)
```


We compute this for each combination of `I` and `G`.

---

## Case 1: `I = Dumb`

### For `G = A`

```python
ψ(I=Dumb, G=A)
= P(D=Easy)P(G=A | Dumb, Easy) + P(D=Hard)P(G=A | Dumb, Hard)
= 0.6 * 0.3 + 0.4 * 0.05
= 0.18 + 0.02
= 0.20
```


### For `G = B`

```python
ψ(Dumb, B)
= 0.6 * 0.4 + 0.4 * 0.25
= 0.24 + 0.10
= 0.34
```


### For `G = C`

```python
ψ(Dumb, C)
= 0.6 * 0.3 + 0.4 * 0.7
= 0.18 + 0.28
= 0.46
```


So for `I=Dumb`:

```python
ψ(Dumb, G) = [0.20, 0.34, 0.46]
```


---

## Case 2: `I = Intelligent`

### For `G = A`

```python
ψ(Intelligent, A)
= 0.6 * 0.9 + 0.4 * 0.5
= 0.54 + 0.20
= 0.74
```


### For `G = B`

```python
ψ(Intelligent, B)
= 0.6 * 0.08 + 0.4 * 0.3
= 0.048 + 0.12
= 0.168
```


### For `G = C`

```python
ψ(Intelligent, C)
= 0.6 * 0.02 + 0.4 * 0.2
= 0.012 + 0.08
= 0.092
```


So for `I=Intelligent`:

```python
ψ(Intelligent, G) = [0.74, 0.168, 0.092]
```


---

# 8. The factor after eliminating `D`

Now we have a new factor `ψ(I, G)`:

| I | G=A | G=B | G=C |
|---|---:|---:|---:|
| Dumb | 0.20 | 0.34 | 0.46 |
| Intelligent | 0.74 | 0.168 | 0.092 |

This is not yet `P(G)`. We still need to eliminate `I`.

---

# 9. Eliminate `I`

Now combine:

- `P(I)`
- `ψ(I, G)`

and sum out `I`.

So:

```python
P(G) = Σ_I P(I) ψ(I, G)
```


Recall:

```python
P(I=Dumb) = 0.7
P(I=Intelligent) = 0.3
```


Now calculate each grade value.

---

## For `G = A`

```python
P(G=A)
= 0.7 * 0.20 + 0.3 * 0.74
= 0.14 + 0.222
= 0.362
```


---

## For `G = B`

```python
P(G=B)
= 0.7 * 0.34 + 0.3 * 0.168
= 0.238 + 0.0504
= 0.2884
```


---

## For `G = C`

```python
P(G=C)
= 0.7 * 0.46 + 0.3 * 0.092
= 0.322 + 0.0276
= 0.3496
```


---

# 10. Final answer

So:

```python
P(G=A) = 0.3620
P(G=B) = 0.2884
P(G=C) = 0.3496
```


That matches your notebook output exactly:

```python
G(A) = 0.3620
G(B) = 0.2884
G(C) = 0.3496
```


Nice — the numbers line up.

---

# 11. What just happened conceptually?

We did not enumerate the full joint over all five variables.

Instead:

- `L` was removed locally
- `S` was removed locally
- `D` was folded into a smaller factor with `I, G`
- `I` was folded into the final answer

So VE used the structure to avoid unnecessary work.

---

# 12. Why `L` and `S` disappeared so easily

This is a beautiful point.

Because `L` and `S` were leaf variables and we had no evidence about them, their CPDs summed to 1:

```python
Σ_L P(L | G) = 1
Σ_S P(S | I) = 1
```


So they contributed nothing to the marginal over `G`.

This is exactly the kind of efficiency VE exploits.

---

# 13. Compare with brute force

Brute force would sum over all combinations of:

- 2 values of `D`
- 2 values of `I`
- 2 values of `L`
- 2 values of `S`

That’s:

```python
2 * 2 * 2 * 2 = 16
```


terms for each value of `G`, so 48 terms total.

VE reduced it to a sequence of small local computations.

In larger networks, this difference becomes massive.

---

# 14. A second numeric example with evidence

Let’s also briefly do:

```python
P(G | D=Easy, I=Intelligent)
```


This one is short and helps intuition.

Since `D` and `I` are given, we plug them into:

```python
P(G | D, I)
```


From the table:

```python
P(G=A | Easy, Intelligent) = 0.9
P(G=B | Easy, Intelligent) = 0.08
P(G=C | Easy, Intelligent) = 0.02
```


That’s the result immediately:

```python
P(G | D=Easy, I=Intelligent) = [0.9, 0.08, 0.02]
```


Again, this matches your notebook output.

Why was it so easy?  
Because once the parents of `G` are known, `G` is fully determined probabilistically by its CPD.

---

# 15. The VE pattern hidden in that evidence example

If we write it formally:

```python
P(G | D=Easy, I=Intelligent)
∝ P(D=Easy) P(I=Intelligent) P(G | D=Easy, I=Intelligent) Σ_L P(L | G) Σ_S P(S | I=Intelligent)
```


Now:

```python
Σ_L P(L | G) = 1
Σ_S P(S | I=Intelligent) = 1
```


and the terms `P(D=Easy)` and `P(I=Intelligent)` are constants w.r.t. `G`, so after normalization you just recover:

```python
P(G | D=Easy, I=Intelligent)
```


---

# 16. The main lesson from the numeric example

## VE is just:
- local multiplication
- local summation
- repeated carefully

Nothing mystical happened.

The “magic” is only that the graph tells us how to keep the computations local.

---

# 17. A compact summary table of the first example

## Query
```python
P(G)
```


## Start with factors
```python
P(D), P(I), P(G|D,I), P(L|G), P(S|I)
```


## Eliminate `L`
```python
Σ_L P(L|G) = 1
```


## Eliminate `S`
```python
Σ_S P(S|I) = 1
```


## Eliminate `D`
```python
ψ(I,G) = Σ_D P(D)P(G|D,I)
```


Result:

| I | A | B | C |
|---|---:|---:|---:|
| Dumb | 0.20 | 0.34 | 0.46 |
| Intelligent | 0.74 | 0.168 | 0.092 |

## Eliminate `I`
```python
P(G) = Σ_I P(I) ψ(I,G)
```


Final:

| G | Probability |
|---|---:|
| A | 0.3620 |
| B | 0.2884 |
| C | 0.3496 |

---

# 18. One very useful intuition

You can interpret the result as a **mixture** over intelligence and difficulty.

For instance:

- if the student is dumb, grades tend to be worse
- if intelligent, grades tend to be better
- difficulty also shifts the grade
- `P(G)` averages over both hidden causes using their priors

That is exactly what the sums are doing.

---

# 19. If you want to remember the math structure

For hidden variables `H`:

```python
P(Query) = Σ_H ∏ factors
```


VE just chooses an order to carry out the sums efficiently.

---

