# Multi-objective optimization and MOEAs: study guide

Based on the Soft Computing Applications decks **MOOP-07**, **MOOP-08**, and **MOOP GA-09**. Repeated definitions are merged, and a few misleading slide statements are corrected at the end.

## The idea in one minute

A **multi-objective optimization problem (MOOP)** asks you to improve at least two measures at once. For example, choose a car with both low cost and high comfort. Usually one choice improves one measure but worsens another. There may be **several equally defensible tradeoffs**, rather than one answer that wins on everything.

A **multi-objective evolutionary algorithm (MOEA)** evolves a population of choices. It uses familiar GA operations, such as selection, crossover, and mutation, but must solve two extra problems:

1. **Preference:** how do we compare answers when objectives conflict?
2. **Diversity:** how do we retain answers from different parts of the tradeoff range?

The essential rule is **Pareto dominance**: one solution beats another only if it is *no worse in every objective* and *strictly better in at least one*.

---

## 1. Formulate the problem and understand the tradeoff

Let $x=(x_1,\ldots,x_n)$ be a choice in the **decision space**. It is **feasible** when it satisfies all constraints. Its **objective vector** $f(x)=(f_1(x),\ldots,f_m(x))$ belongs to the **objective space**. We want to improve $m\geq2$ objective values over the feasible choices.

For example, a design choice $x$ might determine the price and pollution of a product. We minimize $f_1(x)=\text{price}$ and $f_2(x)=\text{pollution}$. The design itself is a point in decision space; its `(price, pollution)` pair is a point in objective space. **One objective point can represent more than one design.**

If one design is cheaper but more polluting than another, neither wins outright. We must keep both until someone supplies a preference, such as a budget or a willingness to pay for cleaner production.

### Running example: minimize cost and pollution

The following six feasible solutions will also explain dominance, sorting, ranking, and crowding. **Smaller is better in both columns.**

| Solution | Cost $f_1$ | Pollution $f_2$ | First observation |
| --- | ---: | ---: | --- |
| A | 4 | 9 | Cheapest |
| B | 6 | 6 | Balanced choice |
| C | 8 | 4 | Cleaner, more costly |
| D | 10 | 2 | Cleanest |
| E | 7 | 8 | B is cheaper **and** cleaner |
| F | 9 | 9 | Several solutions are at least as good |

For example, B **dominates** E: $6<7$ and $6<8$. A and D **do not** dominate one another: A is cheaper, while D is cleaner. Consequently, A, B, C, and D are worthwhile tradeoffs in this set; E and F can be ruled out.

### Ideal, utopian, and nadir vectors

For **minimization** problems:

| Vector | How to construct it | Running example |
| --- | --- | --- |
| **Ideal** $z^*$ | Take the best feasible value for *each objective separately* | $(4,2)$: A's cost and D's pollution |
| **Utopian** $z^{**}$ | Choose a value strictly better than ideal in each component | $(3,1)$, if we subtract 1 from each ideal component |
| **Nadir** $z^{\mathrm{nad}}$ | Take the worst value for each objective **among Pareto-optimal solutions** | $(10,9)$ among A, B, C, D |

The ideal pair $(4,2)$ is not achieved by any of these six choices. An ideal point **can** be feasible if one choice attains all separate objective minima. A utopian point, defined to beat every separate minimum, cannot be feasible. The nadir is based on the true Pareto set, **not** on the worst feasible answer: F is feasible but does not define the nadir here.

Ideal and nadir values can put objectives on comparable scales. For an objective $f_i$ that is **minimized**:

$$
\widehat f_i(x)=\frac{f_i(x)-z_i^*}{z_i^{\mathrm{nad}}-z_i^*},
\qquad z_i^{\mathrm{nad}}>z_i^*.
$$

B's normalized cost is $(6-4)/(10-4)=1/3$; its normalized pollution is $(6-2)/(9-2)=4/7$. A value of 0 is ideal for that objective; 1 reaches its nadir value. If the denominator is zero, omit that constant objective or give it a defined special treatment. **Being closer to the ideal by one distance measure does not automatically make a solution preferable**: the choice of measure expresses a preference.

---

## 2. Dominance, non-dominated sets, and the Pareto front

For two **minimization** objectives, $x$ dominates $y$, written $x\prec y$, when

$$
f_1(x)\leq f_1(y),\quad f_2(x)\leq f_2(y),
\quad\text{and at least one inequality is strict.}
$$

For **maximization**, reverse the relevant inequalities. In a mixed problem such as *minimize cost, maximize comfort*, $x$ dominates $y$ if its cost is no greater and its comfort is no lower, with at least one strict improvement. You can also convert every maximize objective to a minimize objective by negating it.

For a quick mixed-direction check, `(cost 5, comfort 8)` dominates `(cost 6, comfort 7)`. It is incomparable with `(cost 4, comfort 6)`: one car is more comfortable, the other is cheaper.

**How to compare two solutions:** check every objective in its correct direction. If one solution improves one objective but worsens another, the pair is **incomparable**. “$x$ does not dominate $y$” does **not** imply that $y$ dominates $x$. Equal objective vectors do not strictly dominate one another.

| Term | Meaning |
| --- | --- |
| Non-dominated **within a set $P$** | No other member of $P$ dominates it |
| Pareto-optimal **over all feasible solutions** | No feasible solution anywhere dominates it |
| Pareto-optimal set | All Pareto-optimal solutions in **decision space** |
| Pareto front | Their objective vectors in **objective space** |

Our six-solution sample has first non-dominated layer $\{A,B,C,D\}$. If these are *all* feasible solutions, that layer is the true Pareto-optimal set; if other feasible solutions exist, it is only a current-population approximation. A Pareto front may consist of one point if one feasible answer dominates all others. The concept still applies.

Strict Pareto dominance is **irreflexive** ($x$ does not dominate itself), **asymmetric** (if $x$ dominates $y$, $y$ cannot dominate $x$), and **transitive** (if $x$ dominates $y$ and $y$ dominates $z$, then $x$ dominates $z$). It is a **strict partial order**. The slides incorrectly say it is not antisymmetric: asymmetry actually implies antisymmetry in the formal logical sense.

---

## 3. Choose *when* preferences enter the search

| Approach | Preference enters | Typical output | Main use |
| --- | --- | --- | --- |
| **A priori** | Before optimization | One preferred solution per run | You already know priorities or tradeoff weights |
| **A posteriori** | After generating alternatives | Several candidate tradeoffs, then a choice | You want to inspect the available compromises first |

These labels answer *when* preferences are used. A different distinction is whether the **selection rule uses Pareto dominance**. Lexicographic ordering and weighted sum are non-Pareto approaches; VEGA selects by individual objectives and is also non-Pareto. MOGA, NPGA, NSGA, and NSGA-II use Pareto comparisons.

### A priori: lexicographic ordering

Rank objectives by importance, such as (1) minimize cost, then (2) minimize pollution. First find the smallest feasible cost $c^*$. Next, minimize pollution **subject to cost remaining exactly $c^*$**. Repeat through all objectives.

In our example, A has the smallest cost, 4. It wins even though it has the highest pollution, 9. Lower-priority objectives can break a tie in cost but cannot compensate for a worse cost. Results can change substantially when priorities change. With numerical computations, a specified tolerance may replace exact equality if the application allows it.

### A priori: weighted sum / SOEA

For two objectives being minimized, make one scalar objective:

$$
F(x)=w_1\widehat f_1(x)+w_2\widehat f_2(x),\qquad
w_i\geq0,\quad w_1+w_2=1.
$$

Then use a conventional single-objective optimizer or GA. We normalize here because a difference of 1 dollar and a difference of 1 tonne should not be treated as automatically equivalent. **Weights depend on units and scaling**; a weight of 0.7 does not by itself mean that an objective is “70% important.”

In our example, equal weights on the normalized columns give A: $0.5$, B: $\tfrac12(1/3+4/7)\approx0.452$, C: $\tfrac12(2/3+2/7)\approx0.476$, and D: $0.5$. The lowest score is **B** for these weights. Other weights may pick another tradeoff. Positive weights and *exact* global optimization yield a Pareto-optimal solution, but an approximate GA may settle on a worse one. Repeating the search with many weights can reveal several tradeoffs, yet weighted sums can miss Pareto points on a **nonconvex** part of the attainable front.

**Another simple strategy: one objective as a constraint.** Minimize cost while requiring pollution $\leq6$: among the sample, B is the cheapest feasible choice. Tighten the limit to $\leq4$, and C is the cheapest; at $\leq2$, D is the only choice. The pollution limit is a preference supplied before each run. Varying it can reveal tradeoffs that a weighted sum misses.

### A posteriori: VEGA (vector evaluated GA)

With $k$ objectives and a mating pool of size $M$, select roughly **$M/k$ parents using objective 1 alone**, $M/k$ using objective 2 alone, and so on. Shuffle the selected parents, then use ordinary GA crossover, mutation, and population replacement. For $M=12$ and two objectives, you might select six parents for low cost and six for low pollution.

**Strength:** easy to build on a simple GA because parent selection is the key change. **Weakness:** selection can favor specialists at the extremes and lose balanced solutions. It does not itself test whether every selected solution is Pareto non-dominated. Any non-dominance claim about the *current sample* is not a guarantee about the entire feasible space.

---

## 4. Pareto-based MOEAs: what distinguishes each method?

All four methods use domination information, but they turn it into parent selection differently.

| Method | Who gets preference? | How it keeps different tradeoffs | Elitist in its basic form? |
| --- | --- | --- | --- |
| **MOGA** | Fewer population members dominate an individual | Adjust/average fitness by dominance rank; sharing variants may help | No explicit elitist parent–child merge |
| **NPGA** | Tournament candidate not dominated by a random comparison set | Niche/sharing tie breaker | No explicit elitist merge |
| **NSGA** | Earlier non-dominated front | Fitness sharing within a front | No |
| **NSGA-II** | Earlier front, then less crowded points | Crowding distance; retains best from parents and children | **Yes** |

**Sharing / niching:** if many candidates occupy the same small region of objective space, lower their selection advantage so less populated regions also survive. A **niche count** measures how crowded a candidate's neighborhood is, often by summing a distance-based sharing function. A larger niche count means more nearby solutions. Traditional sharing needs a radius or sharing parameter, which can be awkward to tune. NSGA-II instead uses crowding distance.

### MOGA: rank by how many solutions dominate you

For an individual $x_i$ in a population $P$:

$$
r_{\text{MOGA}}(x_i)=1+\#\{x_j\in P: x_j\prec x_i\}.
$$

In our example, A, B, C, D have rank **1**, E is dominated by B and has rank **2**, and F is dominated by A, B, C, E and has rank **5**. MOGA then maps the ranks to selection fitness, often assigning equal fitness to solutions with the same rank.

**Important distinction:** MOGA rank is **1 plus the number of dominators**, not the front number. Several solutions on the same later front can have different MOGA ranks.

### NPGA: dominance tournaments plus a niche tie breaker

1. Randomly choose two candidates and a separate comparison set from the current population.
2. Check whether any comparison-set member dominates each candidate.
3. If **only one** candidate is dominated by the comparison set, pick the **other** candidate.
4. If both or neither are dominated, prefer the candidate in the **less crowded niche**.

The comparison-set size and sharing radius affect the behavior. NPGA examines a sample for each tournament, so “undominated in this tournament” is weaker than “Pareto-optimal over all feasible solutions.”

### NSGA: sort into fronts and share fitness

Repeatedly remove the non-dominated solutions: the first removal is **front 1**; find the non-dominated set of what remains for **front 2**; continue. Our example gives:

| Front | Members | Reason |
| --- | --- | --- |
| 1 | A, B, C, D | None is dominated in the original sample |
| 2 | E | B dominated E, but B has been removed |
| 3 | F | Remaining E dominates F |

NSGA gives everyone within a front the same initial **dummy fitness**, higher for better fronts. It then divides or otherwise adjusts scores based on niche density, selects parents, and reproduces using a GA. Original NSGA does not automatically preserve the best parents in the next generation; its sharing parameter needs tuning. The older sorting approach is often described as roughly $O(MN^3)$ for $M$ objectives and population size $N$.

### NSGA-II: retain good fronts and spread across the front

This is the most important procedure to learn:

1. Start with $N$ parents $P$ and generate $N$ children $Q$ through selection, crossover, and mutation.
2. Combine them into $R=P\cup Q$, containing **$2N$ individuals**. Keep duplicate individuals as separate population entries if the implementation does.
3. Sort $R$ into non-dominated fronts $F_1,F_2,\ldots$.
4. Fill the next population with complete fronts in order, stopping before exceeding $N$ individuals. **An exactly fitting front is included in full.**
5. If the next front does not fit, calculate its crowding distances and take the members with the **largest** distances to fill the remaining slots.
6. Generate the next children from the chosen population, usually using **crowded tournament selection**, and repeat.

Parents and children compete together, providing **elitism**: strong parents have a chance to survive even if the new children are weak. NSGA-II's usual fast sorting method compares pairs and tracks, for each solution, its **domination count** (how many dominate it) and its **dominated set** (which solutions it dominates). Zero-count solutions make front 1; remove their effect on the counts to reveal front 2, and so on. The standard sorting work is $O(MN^2)$, with crowding calculations typically $O(MN\log N)$.

### Crowding distance: one calculation to understand

**Crowding distance** estimates how isolated a solution is *among members of the same front*. For each objective, sort that front by the objective's value. Give boundary solutions an infinite distance. For an interior solution $i$, add the normalized difference between its two neighboring values:

$$
d_i=\sum_{k=1}^{m}
\frac{f_k(i+1)-f_k(i-1)}{f_k^{\max}-f_k^{\min}}.
$$

Here $(i-1)$ and $(i+1)$ mean neighbors **after sorting separately by objective $k$**; they need not be the same two individuals for every objective. A constant objective contributes zero rather than a division by zero. Larger $d_i$ means the point occupies a less crowded part of the front.

For front 1 of our example, A `(4,9)` and D `(10,2)` are boundary points and receive $\infty$. For B `(6,6)`, its cost neighbors are A and C, giving $(8-4)/(10-4)=4/6$. Its pollution neighbors are C and A, giving $(9-4)/(9-2)=5/7$. Thus $d_B\approx1.381$. For C `(8,4)`, $d_C=(10-6)/6+(6-2)/7\approx1.238$. If only three places are available for A, B, C, D, the crowding rule retains **A, D, and B**. Endpoints survive, and B fills a relatively larger gap than C.

In **crowded tournament selection**, first prefer a lower front number; if candidates share a front, prefer the one with larger crowding distance. Crowding distance spreads the sample but does **not** say a sparse solution is intrinsically better for every decision-maker.

---

## 5. Fast revision: what to write in an exam

| If the question asks... | Start your answer with... |
| --- | --- |
| What is a MOOP? | Multiple objectives, decision variables, feasibility constraints, and conflicting tradeoffs |
| Does $x$ dominate $y$? | Check **all** objective directions; no worse in every one, better in at least one |
| Non-dominated vs. Pareto-optimal? | Non-dominated **in the tested set** vs. non-dominated **over the entire feasible set** |
| Ideal vs. nadir? | Best of each objective over feasible points vs. worst of each objective **on the Pareto set** |
| A priori vs. a posteriori? | Preferences before the search vs. inspect alternatives before choosing |
| MOGA vs. NSGA rank? | $1+$ number of dominators vs. sequential non-dominated **front number** |
| NPGA? | Two-candidate dominance tournament against a comparison set; sharing resolves ties |
| NSGA-II? | Parents + children $\to$ fronts $\to$ complete fronts $\to$ crowding for the partial front |

### Slide corrections worth remembering

- An ideal point **can** be feasible, and the Pareto set **can** contain only one solution. Some MOOP-07 slides imply otherwise.
- Strict dominance is a **strict partial order**. It is irreflexive and asymmetric; the statement in MOOP-07 that it is “not antisymmetric” is formally incorrect.
- In MOOP GA-09, distinguish the set of solutions that **dominate** $x_i$ from the set that $x_i$ **dominates**. Fast sorting needs both, in opposite roles.
- An NPGA tournament must choose the candidate that **is not** dominated by the comparison set when only one candidate is dominated; a pseudocode line in MOOP GA-09 reverses this choice.
- For NSGA-II, include a front whenever it **fits exactly**. The strict “$<$” condition on one slide misses that case. Its crowding comparison applies across fronts for parent tournaments, and within the last partial front for survivor selection.

**Slide map:** MOOP-07 covers terminology and Pareto dominance; MOOP-08 covers lexicographic ordering, weighted sums, and VEGA; MOOP GA-09 covers MOGA, NPGA, NSGA, and NSGA-II.
