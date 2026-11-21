# Computing Mathematics (WIX1001)

> Discrete maths is the maths of *separate, countable things*: sets, true/false statements, connections. That's exactly the kind of thing computers work with.

**Topics:** [Sets](#1-sets) · [Propositional Logic](#2-propositional-logic) · [Predicate Logic](#3-predicate-logic) · [Proof](#4-proof) · [Functions](#5-functions) · [Relations](#6-relations) · [Matrices](#7-matrices) · [Counting](#8-counting) · [Graph Theory](#9-graph-theory) · [Trees](#10-trees)

---

## 1. Sets

### What is a set?
A **set** is a collection of *distinct* objects where order doesn't matter.

> 🎒 **Analogy:** A set is like a bag of unique stickers. Putting the same sticker in twice doesn't give you two, and shaking the bag doesn't change what's inside.

```
A = {1, 2, 3}
{1, 2, 3} = {3, 1, 2} = {1, 1, 2, 3}   ← all the same set
```

| Term | Meaning | Example |
|---|---|---|
| Member `x ∈ A` | x is in A | `2 ∈ {1,2,3}` |
| Subset `A ⊆ B` | every element of A is in B | `{1,2} ⊆ {1,2,3}` |
| Proper subset `A ⊂ B` | subset, but A ≠ B | `{1} ⊂ {1,2}` |
| Universe `U` | everything we're talking about | all students in UM |
| Empty set `∅` or `{}` | no elements | the set of dogs that are cats |
| Power set `P(A)` | set of **all** subsets of A | `P({a,b}) = {∅,{a},{b},{a,b}}` |
| Order `|A|` | how many elements | `|{a,b,c}| = 3` |

**Power set rule:** if `|A| = n`, then `|P(A)| = 2ⁿ`. Why? For each element you make a yes/no choice: in the subset or not.

### Set operations (draw Venn diagrams!)

| Operation | Symbol | Meaning | `A={1,2,3}`, `B={3,4}` |
|---|---|---|---|
| Union | `A ∪ B` | in A **or** B | `{1,2,3,4}` |
| Intersection | `A ∩ B` | in A **and** B | `{3}` |
| Difference | `A − B` | in A but **not** B | `{1,2}` |
| Complement | `Aᶜ` or `A'` | everything in U **not** in A | depends on U |

> 🍕 **Analogy:** A = people who like pizza, B = people who like burgers. `A ∩ B` is the people who'd be happy at either restaurant; `A − B` is pizza-only people.

**Counting elements (inclusion–exclusion):**
```
|A ∪ B| = |A| + |B| − |A ∩ B|
```
We subtract the overlap because it was counted twice.

*Example:* 30 students take Java, 25 take Python, 10 take both. How many take at least one? `30 + 25 − 10 = 45`.

**Useful set identities** (look just like logic laws later):
- De Morgan: `(A ∪ B)ᶜ = Aᶜ ∩ Bᶜ` and `(A ∩ B)ᶜ = Aᶜ ∪ Bᶜ`
- Distributive: `A ∩ (B ∪ C) = (A ∩ B) ∪ (A ∩ C)`

### Cartesian product
`A × B` = all ordered pairs `(a, b)` with a from A and b from B.
```
A = {1,2}, B = {x,y}
A × B = {(1,x), (1,y), (2,x), (2,y)}      |A × B| = |A|·|B| = 4
```
> 👕 **Analogy:** 3 shirts × 2 pants = 6 outfits.

---

## 2. Propositional Logic

### Proposition
A **proposition** is a statement that is either **true or false** (not both).

| Proposition? | Why |
|---|---|
| "Kuala Lumpur is in Malaysia." ✅ | true |
| "2 + 2 = 5" ✅ | false, but still a proposition |
| "Close the door." ❌ | a command |
| "x + 1 = 3" ❌ | depends on x |
| "What time is it?" ❌ | a question |

### Connectives

| Name | Symbol | Read as | True when… |
|---|---|---|---|
| Negation | `¬p` | not p | p is false |
| Conjunction | `p ∧ q` | p and q | **both** true |
| Disjunction | `p ∨ q` | p or q | **at least one** true |
| Conditional | `p → q` | if p then q | false **only** when p true and q false |
| Biconditional | `p ↔ q` | p if and only if q | p and q have the same value |

**Truth table:**
```
p q | ¬p  p∧q  p∨q  p→q  p↔q
T T |  F   T    T    T    T
T F |  F   F    T    F    F
F T |  T   F    T    T    F
F F |  T   F    F    T    T
```

> 🤝 **Understanding `p → q`, think of a promise:** "If you pass the exam (p), I'll buy you dinner (q)."
> - You pass, I buy dinner → promise kept (T)
> - You pass, no dinner → **I lied** (F)
> - You fail, I buy dinner anyway → no promise broken (T)
> - You fail, no dinner → no promise broken (T)

**Precedence:** `¬` → `∧` → `∨` → `→` → `↔` (like × before + in arithmetic).

### Converse, inverse, contrapositive
Starting from `p → q`:

| Name | Form | Same meaning as original? |
|---|---|---|
| Converse | `q → p` | ❌ no |
| Inverse | `¬p → ¬q` | ❌ no |
| **Contrapositive** | `¬q → ¬p` | ✅ **yes, always equivalent** |

*Example:* "If it's raining, the ground is wet." Contrapositive: "If the ground is not wet, it's not raining." (true). Converse: "If the ground is wet, it's raining." (not necessarily, someone may have watered the plants).

### Tautology, contradiction, equivalence
- **Tautology:** always true, e.g. `p ∨ ¬p` ("it's raining or it's not")
- **Contradiction:** always false, e.g. `p ∧ ¬p`
- **Logically equivalent** `p ≡ q`: same truth table. Key one: `p → q ≡ ¬p ∨ q`

**Laws of logic** (same shapes as set identities):
- De Morgan: `¬(p ∧ q) ≡ ¬p ∨ ¬q`, `¬(p ∨ q) ≡ ¬p ∧ ¬q`
- Double negation: `¬¬p ≡ p`
- Distributive, commutative, associative, identity, domination, idempotent, absorption

💻 **In code:**
```java
// De Morgan in action: these two conditions are identical
if (!(age >= 18 && hasID)) { deny(); }
if (age < 18 || !hasID)   { deny(); }
```

### Arguments and rules of inference
An **argument** = premises → conclusion. It's **valid** if whenever all premises are true, the conclusion must be true.

| Rule | Form | Everyday example |
|---|---|---|
| Modus Ponens | `p → q`, `p` ⊢ `q` | If it rains, I bring an umbrella. It rains. ∴ I bring an umbrella. |
| Modus Tollens | `p → q`, `¬q` ⊢ `¬p` | If it rains, the road is wet. Road isn't wet. ∴ it didn't rain. |
| Hypothetical Syllogism | `p → q`, `q → r` ⊢ `p → r` | Study → pass, pass → graduate ∴ study → graduate |
| Disjunctive Syllogism | `p ∨ q`, `¬p` ⊢ `q` | Tea or coffee. Not tea. ∴ coffee. |
| Addition | `p` ⊢ `p ∨ q` | |
| Simplification | `p ∧ q` ⊢ `p` | |
| Conjunction | `p`, `q` ⊢ `p ∧ q` | |

⚠️ **Common invalid arguments:** *affirming the consequent* (`p → q`, `q` ⊢ `p`) and *denying the antecedent* (`p → q`, `¬p` ⊢ `¬q`).

---

## 3. Predicate Logic

Propositional logic can't say "**all** students" or "**some** number". Predicate logic adds variables and quantifiers.

- **Predicate** `P(x)`: a statement with a variable, e.g. `P(x): x > 3`. It becomes a proposition once you plug in x: `P(5)` is true.
- **Domain:** the set of values x can take.

| Quantifier | Symbol | Meaning | True when… | False when… |
|---|---|---|---|---|
| Universal | `∀x P(x)` | for all x | P(x) true for **every** x | you find **one counterexample** |
| Existential | `∃x P(x)` | there exists x | P(x) true for **at least one** x | P(x) false for every x |

> 🏫 **Analogy:** `∀x` is like a teacher saying "**everyone** submitted homework"; one missing student breaks it. `∃x` is "**someone** brought a pen"; you only need to find one.

💻 **In code:**
```java
boolean all  = list.stream().allMatch(x -> x > 0);   // ∀x (x > 0)
boolean some = list.stream().anyMatch(x -> x > 0);   // ∃x (x > 0)
```

### Negating quantifiers (De Morgan for quantifiers)
```
¬∀x P(x) ≡ ∃x ¬P(x)      "Not everyone passed" = "someone failed"
¬∃x P(x) ≡ ∀x ¬P(x)      "Nobody cheated"     = "everyone did not cheat"
```

### Nested quantifiers: order matters!
Domain: people. `L(x,y)`: x loves y.
- `∀x ∃y L(x,y)`: everyone loves *somebody* (maybe different people).
- `∃y ∀x L(x,y)`: there is *one person* whom everybody loves.

Think of nested quantifiers as nested loops:
```java
// ∀x ∃y P(x,y)
for (x : domain) { boolean found = false; for (y : domain) if (P(x,y)) found = true; if (!found) return false; }
return true;
```

### Rules of inference with quantifiers
| Rule | Idea |
|---|---|
| Universal instantiation | `∀x P(x)` ⊢ `P(c)`: true for all, so true for this one |
| Universal generalization | `P(c)` for an *arbitrary* c ⊢ `∀x P(x)` |
| Existential instantiation | `∃x P(x)` ⊢ `P(c)` for *some* c (give it a name) |
| Existential generalization | `P(c)` ⊢ `∃x P(x)` |

*Classic:* All humans are mortal. Socrates is human. ∴ Socrates is mortal. (Universal instantiation + modus ponens.)

---

## 4. Proof

A **proof** is a step-by-step argument showing a statement must be true.

| Method | To prove `p → q`… | When to use |
|---|---|---|
| **Direct** | Assume p, reason forward to q | Default first try |
| **Contrapositive** | Assume ¬q, reason to ¬p | When "not q" gives you more to work with |
| **Contradiction** | Assume the statement is false, derive something impossible | Statements like "√2 is irrational", "infinitely many primes" |
| **Cases** | Split into all possible cases, prove each | Even/odd, positive/negative |
| **Equivalence** `p ↔ q` | Prove `p → q` **and** `q → p` | "if and only if" statements |

**Direct proof example:** *If n is odd, then n² is odd.*
> Let n = 2k + 1. Then n² = 4k² + 4k + 1 = 2(2k² + 2k) + 1, which is odd. ∎

**Contrapositive example:** *If n² is even, then n is even.*
> Contrapositive: if n is odd, n² is odd, which we just proved. ∎

**Contradiction example:** *√2 is irrational.*
> Suppose √2 = a/b in lowest terms. Then 2b² = a², so a is even, a = 2c. Then 2b² = 4c² → b² = 2c² → b is even. But then a and b are both even, which contradicts "lowest terms". ∎

> 🕵️ **Analogy:** Proof by contradiction is like a detective saying "Suppose the butler did it. Then he'd have to be in two places at once, which is impossible. So the butler didn't do it."

---

## 5. Functions

A **function** `f: A → B` assigns **each** element of A to **exactly one** element of B.

> 🥤 **Analogy:** A vending machine. Each button (input) gives exactly one drink (output). Two buttons may give the same drink, but one button never gives two different drinks.

- **Domain** = A (all allowed inputs)
- **Codomain** = B (where outputs are allowed to land)
- **Range** = the outputs actually produced (⊆ codomain)

### Types of functions

| Type | Rule | Analogy |
|---|---|---|
| **Injective** (one-to-one) | different inputs → different outputs | each student has a unique matric number |
| **Surjective** (onto) | every element of B gets hit | every seat in the cinema is taken |
| **Bijective** | both injective **and** surjective | perfect pairing: every student gets exactly one seat, no empty seats |

- **Test injective:** assume `f(a) = f(b)` and show `a = b`. To show it's NOT injective, find two inputs with the same output.
- **Test surjective:** for any y in B, solve `f(x) = y` and check x is in A.

*Example:* `f(x) = 2x + 1` on ℝ → ℝ is bijective. `g(x) = x²` on ℝ → ℝ is neither (g(2) = g(−2), and nothing maps to −1).

### Inverse and composition
- **Inverse** `f⁻¹` exists only if f is **bijective**. To find it, write `y = f(x)`, solve for x, then swap the names.
  `f(x) = 2x + 1` → `x = (y − 1)/2` → `f⁻¹(x) = (x − 1)/2`
- **Composition** `(f ∘ g)(x) = f(g(x))`: do g **first**, then f.

💻 **In code:**
```java
Function<Integer,Integer> g = x -> x + 1;
Function<Integer,Integer> f = x -> x * 2;
f.compose(g).apply(3);   // f(g(3)) = f(4) = 8
f.andThen(g).apply(3);   // g(f(3)) = g(6) = 7
```

### Common functions
- **Floor** `⌊x⌋`: round down. `⌊3.7⌋ = 3`, `⌊−1.2⌋ = −2`
- **Ceiling** `⌈x⌉`: round up. `⌈3.2⌉ = 4`
- **Piecewise:** different rules for different inputs (like `if/else`)
- **Multivariable:** more than one input, e.g. `f(x, y) = x + y`

💡 *Floor in real life:* how many full boxes of 12 from 50 eggs? `⌊50/12⌋ = 4`. How many boxes needed? `⌈50/12⌉ = 5`.

---

## 6. Relations

A **relation** R from A to B is any subset of `A × B`: a set of pairs saying "who is related to whom". Functions are a special kind of relation.

> 🎓 **Analogy:** Students and courses. `(Ali, Java)` in R means Ali takes Java. Unlike a function, a student can take many courses.

**Types by "how many":** one-to-one, one-to-many, many-to-one, many-to-many. (You'll see these again in Databases!)

**Representations:**
- List of pairs: `R = {(1,2), (2,3), (1,3)}`
- **Matrix:** put a 1 at row a, column b if `(a,b) ∈ R`
- **Directed graph (digraph):** an arrow from a to b

**Inverse:** `R⁻¹ = {(b,a) | (a,b) ∈ R}` (flip each pair).
**Composite:** `S ∘ R`: (a,c) if there's a b with `(a,b) ∈ R` and `(b,c) ∈ S`. ("friend of a friend")

### Properties of a relation on a set A

| Property | Rule | Example | Non-example |
|---|---|---|---|
| **Reflexive** | every `(a,a)` is in R | `≤`, "same birthday as" | `<` |
| **Symmetric** | `(a,b)` → `(b,a)` | "is sibling of" | "is parent of" |
| **Antisymmetric** | `(a,b)` and `(b,a)` → `a = b` | `≤` | "is sibling of" |
| **Transitive** | `(a,b)` and `(b,c)` → `(a,c)` | `<`, "is ancestor of" | "is parent of" |

**Equivalence relation** = reflexive + symmetric + transitive. It splits the set into groups (equivalence classes).
*Example:* "has the same remainder mod 3" groups integers into {…,0,3,6,…}, {…,1,4,7,…}, {…,2,5,8,…}.

> 🎨 **Analogy:** "same colour shirt as" sorts a crowd into colour groups.

---

## 7. Matrices

A **matrix** is a rectangular grid of numbers with `m` rows and `n` columns (`m × n`).

> 📊 **Analogy:** a spreadsheet. Images are matrices of pixels, game graphics use matrices to rotate and scale, and graphs can be stored as adjacency matrices.

### Operations
- **Add/subtract:** same size only, element by element.
- **Scalar multiply:** multiply every entry by the number.
- **Matrix multiply** `A (m×n) · B (n×p) = C (m×p)`: inner sizes must match. Each entry is **row of A · column of B**.

```
[1 2] [5 6]   [1·5+2·7  1·6+2·8]   [19 22]
[3 4] [7 8] = [3·5+4·7  3·6+4·8] = [43 50]
```
⚠️ In general `AB ≠ BA` (order matters!).

```java
int[][] multiply(int[][] A, int[][] B) {
    int m = A.length, n = B.length, p = B[0].length;
    int[][] C = new int[m][p];
    for (int i = 0; i < m; i++)
        for (int j = 0; j < p; j++)
            for (int k = 0; k < n; k++)
                C[i][j] += A[i][k] * B[k][j];
    return C;
}
```

### Special matrices
| Name | Description |
|---|---|
| Square | rows = columns |
| Identity `I` | 1s on the diagonal, 0 elsewhere; `AI = IA = A` (like multiplying by 1) |
| Diagonal | non-zero only on the diagonal |
| Symmetric | `A = Aᵀ` (mirror across the diagonal) |
| Upper/Lower triangular | zeros below/above the diagonal |
| Row/column vector | 1 × n or n × 1 |

**Transpose** `Aᵀ`: rows become columns. Property: `(AB)ᵀ = BᵀAᵀ`.

### Determinant
A single number that tells you whether a square matrix is invertible (`det ≠ 0`).
```
2×2:  |a b|
      |c d| = ad − bc
```
3×3: expand along a row using cofactors (or use Sarrus' rule):
```
det = a(ei − fh) − b(di − fg) + c(dh − eg)   for [[a,b,c],[d,e,f],[g,h,i]]
```

### Inverse
`A⁻¹` is the matrix such that `A·A⁻¹ = I`. For 2×2:
```
A⁻¹ = 1/(ad − bc) · [ d  −b]
                    [−c   a]
```
For 3×3, use the adjoint method (`A⁻¹ = adj(A)/det(A)`) or Gauss–Jordan elimination `[A | I] → [I | A⁻¹]`.

### Solving systems of linear equations
```
2x + y = 5          [2 1][x]   [5]
x  − y = 1    →     [1 −1][y] = [1]    →  X = A⁻¹B  → x = 2, y = 1
```
Other methods: Cramer's rule (`x = det(Aₓ)/det(A)`), Gaussian elimination.

---

## 8. Counting

### Multiplication principle
If one task has `m` ways and another has `n` ways, doing both has `m × n` ways.
*Example:* a password of 2 letters then 3 digits: `26² × 10³ = 676,000`.

### Factorial
`n! = n × (n−1) × … × 1`, and `0! = 1`. It counts the ways to arrange n objects in a line.

### Permutations vs combinations

| | Order matters? | Formula | Example |
|---|---|---|---|
| **Permutation** `P(n,r)` | ✅ yes | `n! / (n−r)!` | Choose president, VP, secretary from 10: `P(10,3) = 720` |
| **Combination** `C(n,r)` | ❌ no | `n! / (r!(n−r)!)` | Choose a 3-person committee from 10: `C(10,3) = 120` |

> 🔐 **Analogy:** A "combination lock" is really a *permutation* lock, since 1-2-3 ≠ 3-2-1! A fruit salad is a true combination: apple+banana = banana+apple.

**Permutations with repeated objects:** arrangements of "BANANA" = `6! / (3! · 2!) = 60` (divide out the identical A's and N's).

```java
long fact(int n) { return n <= 1 ? 1 : n * fact(n - 1); }
long P(int n, int r) { return fact(n) / fact(n - r); }
long C(int n, int r) { return fact(n) / (fact(r) * fact(n - r)); }
```

### Pigeonhole principle
- **Form 1:** if `n + 1` pigeons go into `n` holes, some hole has ≥ 2 pigeons.
- **Form 2/3 (generalised):** if `N` items go into `k` boxes, some box has at least `⌈N/k⌉` items.

*Examples:*
- Among 13 people, two share a birth month.
- Among 367 people, two share a birthday.
- 100 students, 12 months → some month has ≥ `⌈100/12⌉ = 9` birthdays.

> 🧦 **Analogy:** grab socks in the dark from a drawer with red and blue socks. Take 3 and you're guaranteed a matching pair.

---

## 9. Graph Theory

A **graph** `G = (V, E)` is a set of **vertices** (dots) connected by **edges** (lines).

> 🗺️ **Analogy:** cities (vertices) and roads (edges). Social networks (people and friendships), the internet (routers and cables), Google Maps: all graphs.

### Vocabulary
| Term | Meaning |
|---|---|
| Adjacent | two vertices joined by an edge |
| Degree `deg(v)` | number of edges touching v (a loop counts 2) |
| Isolated vertex | degree 0 |
| Path | a sequence of edges from one vertex to another |
| Cycle | a path that starts and ends at the same vertex |
| Connected | there's a path between every pair of vertices |
| Component | a connected "piece" of the graph |
| Subgraph | part of a graph (subset of vertices and edges) |

**Handshake theorem:** `Σ deg(v) = 2|E|` (every edge has 2 ends). So the number of odd-degree vertices is always even.

### Types of graphs
| Type | Description |
|---|---|
| Simple | no loops, no multiple edges |
| Directed (digraph) | edges have arrows (one-way streets) |
| Weighted | edges have numbers (distance, cost, time) |
| Complete `Kₙ` | every vertex connects to every other; `n(n−1)/2` edges |
| Bipartite | vertices split into two groups; edges only go *between* groups (e.g. students ↔ courses) |
| Complete bipartite `Kₘ,ₙ` | every vertex in group 1 connects to every vertex in group 2 |

### Euler vs Hamilton

| | Visits every… | Condition |
|---|---|---|
| **Euler cycle** | **edge** exactly once, ends at start | connected and **every vertex has even degree** |
| **Euler path** | every edge once, may end elsewhere | exactly **0 or 2** odd-degree vertices |
| **Hamilton cycle** | **vertex** exactly once, returns to start | no simple rule (hard problem!) |

> ✏️ **Euler = Edges** (draw the picture without lifting your pen). **Hamilton = vertices** (visit every city once).
> The **Travelling Salesman Problem** = the cheapest Hamilton cycle in a weighted graph.

### Representing graphs
```
   A — B
   |   |
   C — D

Adjacency matrix:        Adjacency list:
    A B C D              A: B, C
A [ 0 1 1 0 ]            B: A, D
B [ 1 0 0 1 ]            C: A, D
C [ 1 0 0 1 ]            D: B, C
D [ 0 1 1 0 ]
```
Matrix = quick "is there an edge?" lookup; list = saves memory for sparse graphs.

**Distance matrix:** entry (i, j) = length of the shortest path between i and j.

### Dijkstra's algorithm (shortest path in a weighted graph, no negative weights)
1. Set distance to the start = 0, all others = ∞.
2. Pick the unvisited vertex with the smallest distance; mark it visited.
3. **Relax** its neighbours: if `dist[u] + w(u,v) < dist[v]`, update `dist[v]`.
4. Repeat until all vertices are visited.

```java
int[] dijkstra(int[][] g, int src) {
    int n = g.length; int[] dist = new int[n]; boolean[] done = new boolean[n];
    Arrays.fill(dist, Integer.MAX_VALUE); dist[src] = 0;
    for (int it = 0; it < n; it++) {
        int u = -1;
        for (int i = 0; i < n; i++) if (!done[i] && (u == -1 || dist[i] < dist[u])) u = i;
        done[u] = true;
        for (int v = 0; v < n; v++)
            if (g[u][v] > 0 && dist[u] + g[u][v] < dist[v]) dist[v] = dist[u] + g[u][v];
    }
    return dist;
}
```
> 🚗 **Analogy:** water spreading out from the start through pipes of different lengths. It reaches the nearest junctions first.

---

## 10. Trees

A **tree** is a connected graph with **no cycles**. A tree with `n` vertices has exactly `n − 1` edges.

> 🌳 **Analogy:** a family tree, or the folders on your computer (one root folder, subfolders inside, files as leaves).

### Rooted tree vocabulary
| Term | Meaning |
|---|---|
| Root | the top vertex |
| Parent / child | directly above / below |
| Siblings | same parent |
| Leaf | no children |
| Internal vertex | has children |
| Level / depth | distance from the root |
| Height | maximum level |
| Ancestor / descendant | anywhere above / below on the path |

### m-ary trees
- **m-ary:** every internal vertex has at most m children (binary: m = 2).
- **Full m-ary:** every internal vertex has **exactly** m children.
- A full m-ary tree with `i` internal vertices has `n = m·i + 1` vertices and `l = (m−1)·i + 1` leaves.

*Example:* a full 3-ary tree with 4 internal vertices has `3·4 + 1 = 13` vertices and `2·4 + 1 = 9` leaves.

**Applications:** binary search trees, decision trees, Huffman coding, expression trees, file systems, game trees.

### Spanning trees
A **spanning tree** of a graph includes **all vertices** and just enough edges to stay connected with no cycles.
> 🔌 **Analogy:** connecting every house to the electricity network using the fewest cables.

### BFS vs DFS
| | **Breadth-First Search** | **Depth-First Search** |
|---|---|---|
| Idea | explore level by level (all neighbours first) | go as deep as possible, then backtrack |
| Uses | **queue** | **stack** / recursion |
| Analogy | ripples in a pond | exploring a maze, keeping one hand on the wall |
| Good for | shortest path in an *unweighted* graph | detecting cycles, topological sort, puzzles |

```java
void bfs(List<List<Integer>> adj, int s) {
    boolean[] seen = new boolean[adj.size()];
    Queue<Integer> q = new LinkedList<>(); q.add(s); seen[s] = true;
    while (!q.isEmpty()) {
        int u = q.poll(); System.out.print(u + " ");
        for (int v : adj.get(u)) if (!seen[v]) { seen[v] = true; q.add(v); }
    }
}
void dfs(List<List<Integer>> adj, int u, boolean[] seen) {
    seen[u] = true; System.out.print(u + " ");
    for (int v : adj.get(u)) if (!seen[v]) dfs(adj, v, seen);
}
```

**Shortest path with BFS:** in an unweighted graph, the level at which BFS first reaches a vertex = its shortest distance from the start.

---

### ✅ Quick self-check
1. How many subsets does a 5-element set have? *(32)*
2. What's the contrapositive of "If I study, I pass"? *(If I don't pass, I didn't study.)*
3. Negate `∀x (x > 0)`. *(∃x (x ≤ 0))*
4. Is `f(x) = x³` on ℝ bijective? *(Yes)*
5. How many ways to choose 2 toppings from 6? *(C(6,2) = 15)*
6. Can you draw K₄ without lifting your pen and return to start? *(No, every vertex has degree 3, which is odd)*
