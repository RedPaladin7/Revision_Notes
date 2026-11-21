# Genetic Algorithms: a clear study guide

Based on the six Soft Computing Applications slide decks **GA-01 through GA-06**. Repeated introductions have been combined, and calculation errors in the slides have been corrected.

## The whole topic in one minute

An **optimization problem** asks for the best valid choice: the cheapest route, the most valuable set of items, or the best settings for a machine. A **genetic algorithm (GA)** keeps several candidate answers at once. It scores them, gives better answers more chances to become parents, mixes parents to make children, occasionally changes a child at random, and repeats. This often finds a good answer in a huge or awkward search space, but **does not guarantee the global optimum**.

The main loop is:

1. Specify the objective, variables, and constraints.
2. Encode each candidate answer as a **chromosome**; create an initial **population**.
3. Decode and score each chromosome using a **fitness** function.
4. Check whether to stop. Otherwise, **select** parents into a mating pool.
5. Apply **crossover**, then **mutation** (and optionally inversion).
6. Form the next population and return to step 3. Remember the best answer seen so far.

**Memory aid:** representation tells the GA *what an answer looks like*; fitness tells it *what is good*; selection chooses *who reproduces*; crossover and mutation decide *which new answers to try*.

---

## 1. First define the optimization problem

An optimization problem has three parts:

| Part | Question it answers | Example: a delivery route |
| --- | --- | --- |
| Design variables | What can we choose? | The order of cities |
| Objective function | What should improve? | Minimize total distance |
| Constraints | Which choices are allowed? | Visit each city once and return to the start |

Write the general form as **minimize or maximize** $f(x)$, **subject to** the relevant restrictions on $x$. A **feasible** candidate satisfies *all* restrictions. An infeasible candidate is not an answer even if its objective value looks impressive.

For example, minimize $f(x)=x^2$ subject to $x\geq2$. The unconstrained minimum is $x=0$, but it is infeasible. The constrained answer is $x=2$, giving $f=4$.

### Know the problem types

| Distinction | Meaning | Example |
| --- | --- | --- |
| Constrained / unconstrained | Extra restrictions are present / absent | Knapsack has a weight limit |
| Integer / real / mixed | Variables are whole numbers / continuous / both | Number of trucks / engine temperature |
| Linear / nonlinear | Objective and constraints are all linear / at least one is nonlinear | $2x+3y$ / $x^2+y^2$ |
| Single / multiple objectives | Optimize one measure / balance several | Distance alone / distance and cost |

Variable domains, such as $x\geq0$, still matter even when a slide calls a problem “unconstrained.”

Common examples: **TSP** minimizes the cost of a tour; **0–1 knapsack** maximizes the total value of selected items within capacity; **graph coloring** minimizes colors while adjacent nodes differ; **job assignment** allocates jobs to machines; **coin change** can minimize the number of coins; **optimal binary search tree** minimizes expected search cost.

### Why consider a GA?

For a smooth one-variable function, ordinary calculus can find candidates using $f'(x)=0$. If $f''(x^*)>0$, a stationary point is a local minimum; if $f''(x^*)<0$, it is a local maximum. If the second derivative is zero, inspect higher derivatives: the **first nonzero derivative after $f'$** gives a local minimum or maximum when its order is even (positive / negative, respectively); odd order means neither. For example, $x^3$ is flat at zero but has no extremum there. Also check boundaries and constraints. A stationary point can be local rather than global.

Enumerating all possibilities in a large combinatorial problem can be costly. Some objectives are discontinuous or have no useful derivative. GAs search with evaluated candidates and can handle different encodings, including discrete choices. Their tradeoff is approximate, stochastic results that depend on representation and parameter choices.

**Changing the direction of optimization:** minimizing $f$ is equivalent to maximizing $-f$. Maximizing $1/f$ is equivalent to minimizing $f$ only when the relevant $f$ values are strictly positive. This sign change is a simple objective transformation, not the full mathematical notion of optimization duality.

---

## 2. GA vocabulary and complete workflow

| Term | Plain meaning |
| --- | --- |
| Gene | One component of an encoded solution |
| Chromosome / individual | The complete encoded candidate |
| Genotype | The stored encoding |
| Phenotype | The actual decoded solution that the encoding describes |
| Population | The current collection of candidates |
| Fitness | A score used to judge and select candidates |
| Mating pool | Selected parents, often with repeated individuals |
| Generation | One cycle of creating and evaluating candidates |

For example, in a three-item knapsack, genotype `011` means “leave item 1, take items 2 and 3.” Its phenotype is the actual set of items, whose weight and value determine its fitness.

**What happens in one iteration?** Select parents according to fitness; pair them; apply crossover with probability $p_c$; mutate genes with a suitable probability $p_m$; check feasibility or repair children; evaluate them; replace some or all of the population. An optional **inversion** operator reverses or rearranges a segment; it is not essential in every GA. Stop after a target quality, a generation/time/evaluation budget, or a sustained lack of improvement. A plateau alone does not prove optimality.

### Population update strategies

| Strategy | Replacement | Main implication |
| --- | --- | --- |
| Simple / generational GA (SGA) | Replace most or all of the old population with children | Broad turnover; a good parent may disappear without elitism |
| Steady-state GA (SSGA) | Insert one or a few children per update, commonly replacing poor individuals | Gradual change; may stagnate if the population becomes too similar |
| Messy GA | Uses flexible or partially specified chromosomes | Changes the representation itself; only an overview in these slides |

**Elitism** copies one or more of the best candidates unchanged into the next population. Independently, record the **best-so-far** answer, since the final population need not contain the best answer found earlier.

The **generation gap** is $G=p/N$, where $p$ of $N$ individuals are replaced per update: near 1 in a fully generational update, small in steady state.

Parameters to recognize: population size $N$, mating pool size $N_p$, crossover probability $p_c$, mutation probability $p_m$, any inversion rate, and the stopping threshold or budget. There is no single setting that suits every problem.

---

## 3. Encode the solution in a suitable form

| Encoding | Chromosome example | Best fit | Operator caution |
| --- | --- | --- | --- |
| Binary | `0 1 1 0` | Yes/no choices; quantized numbers | A bit flip may change a numerical value sharply |
| Real-valued | `[5.28, -475.36]` | Continuous variables | Mix or perturb numbers, usually within bounds |
| Order / permutation | `[A, C, B, D]` | Routes, schedules, sequences | Every item must still appear exactly once |
| Tree | A tree of blocks and cutting operators | Expressions, hierarchical layouts | Preserve a valid tree after genetic operations |

### Binary encoding: the knapsack worked example

Three items have `(weight, value)` equal to $(10,60)$, $(20,100)$, and $(30,120)$. The capacity is 50. Let bit $b_i=1$ mean item $i$ is selected. We maximize

$$
V(b)=60b_1+100b_2+120b_3
\quad\text{subject to}\quad
10b_1+20b_2+30b_3\leq50,
\quad b_i\in\{0,1\}.
$$

`011` selects items 2 and 3: weight $20+30=50$, value $100+120=220$. `111` has value 280 but weight 60, so it is infeasible. Checking the eight possible bit strings confirms `011` is optimal for this tiny example. For $n$ items there are $2^n$ strings including the empty choice, or $2^n-1$ if at least one item must be taken.

For larger problems, a GA may **reject** invalid candidates, **repair** them, or penalize their fitness. Any such rule must still make the original capacity constraint clear.

### Binary strings representing real numbers

With $L$ bits, an unsigned binary string represents an integer $z$ from 0 to $2^L-1$. To encode a real variable in $[a,b]$, decode it as

$$
x=a+\frac{z}{2^L-1}(b-a).
$$

For `01101`, $L=5$ and $z=13$. In $[1,10]$, this means $x=1+9(13/31)\approx4.774$. The spacing between representable values is $(b-a)/(2^L-1)$; more bits mean finer resolution but longer chromosomes. For multiple variables, reserve a segment for each and decode each using **its own bounds**. A decoded candidate can still violate a constraint such as $x+y\leq10$.

Ordinary binary has a **Hamming cliff**: `0111` and `1000` represent consecutive integers 7 and 8 but differ in four bits. **Gray coding** makes consecutive encoded integers differ in one bit; it can reduce this particular mismatch between genetic and numerical distance.

### Permutations: the traveling salesman problem

`[A,B,E,C,D]` is an order to visit cities; the tour closes by returning from D to A. Its fitness might come from total distance. Using the distance matrix in GA-03, that tour costs $2+5+1+3+6=17$. An unavailable edge has infinite cost, so an apparent tour using one is invalid. A chromosome with a duplicate city or a missing city is also invalid. This is why permutation crossover needs special rules.

### Trees: chip floor planning

A **slicing tree** records how circuit blocks form a chip layout. Leaves are blocks; each internal node combines two regions. With the slides’ convention, `i j V` places region `i` to the **left** of `j`, and `i j H` places `j` **above** `i`. Postorder traversal writes a tree in **postfix / reverse Polish notation**. Example: `1 2 V 3 H` first puts 1 left of 2, then puts 3 above that combined region. A good encoding keeps every block present and produces a legal layout; the objective can include area or wiring cost.

---

## 4. Fitness and parent selection

The **objective** is what the original problem asks for; **fitness** is the score the GA uses to select candidates. For a maximization problem, the objective itself may work as fitness if it is nonnegative. For minimization, transform cost into a score that favors **lower** cost, such as $F=1/(1+c)$ when $c\geq0$, or use rank/tournament selection based directly on lower cost. Do not accidentally reward an infeasible or expensive answer.

### Fitness-proportionate (canonical / roulette-wheel) selection

For nonnegative scores $F_i$ with positive total, the probability of selecting candidate $i$ is

$$
p_i=\frac{F_i}{\sum_jF_j}.
$$

If four candidates have fitness `[1, 2, 3, 4]`, probabilities are `[0.1, 0.2, 0.3, 0.4]`. Their cumulative boundaries are `[0.1, 0.3, 0.6, 1]`. Draw $r$ uniformly from $[0,1)$: for example, $r=0.26$ selects candidate 2. Drawing again can select the **same parent repeatedly**. In a mating pool of size $N_p$, candidate $i$ has expected count $N_p p_i$, not a guaranteed count.

**Why it can fail:** one very high score can take almost all selections; nearly equal scores give almost no selection pressure. Roulette selection probabilities depend on relative score *sizes*, even if the rank order stays the same.

### Rank, tournament, and elitism

- **Rank selection:** sort candidates by fitness and assign probabilities to ranks instead of raw scores. For four candidates, rank weights `4,3,2,1` give probabilities `0.4,0.3,0.2,0.1`. The huge raw lead of one candidate then matters less. The exact rank weighting is a design choice.
- **Tournament selection:** randomly sample $k$ candidates, keep the best of that sample, and repeat until the mating pool is full. For example, from candidates with fitness 2.1 and 4.0, the 4.0 candidate wins. Larger $k$ generally gives stronger preference to good candidates. Resolve equal fitness by a stated tie rule, such as a random choice.
- **Elitism:** copy the best candidate directly to the next population; this is a preservation rule alongside the chosen parent selection method.
- **Boltzmann selection:** use a temperature parameter to control how strongly a fitness difference affects selection; lowering temperature generally favors the better candidates more strongly. The slides mention it without developing an implementation.

**Selection pressure** describes how strongly better candidates are favored. High pressure can rapidly remove diversity; very low pressure can make improvement slow. **Diversity** means the population contains meaningfully different candidates. Good search needs enough pressure to improve and enough diversity to explore. No selection scheme is universally best: behavior depends on tournament size, fitness spread, replacement rules, and other settings.

---

## 5. Crossover: combine parents without breaking the encoding

Select two parents, then apply crossover with probability $p_c$. If it does not happen, copy the parents (perhaps subject to later mutation). A **cut point** lies *between* genes; for a chromosome of length $L$, a nontrivial single cut is at position 1 through $L-1$.

### Binary crossover

Suppose `P1 = 0110|0010` and `P2 = 1010|1100`. **One-point crossover** swaps the suffixes:

```text
child 1 = 0110|1100
child 2 = 1010|0010
```

**Two-point crossover** swaps a segment between two cuts; **multi-point** alternates parental segments at several cuts. **Uniform crossover** makes a parental choice at each position. For example, with `P1=1100`, `P2=0011`, and mask `1010` (1 takes from P1, 0 from P2), child 1 is `1001` and child 2 is `0110`.

Other named binary methods in the slides:

| Method | Essence |
| --- | --- |
| Half-uniform (HUX) | Swap half of the positions at which the parents differ; if that number is odd, an implementation must specify rounding |
| Shuffle | Apply the same random permutation to both parents, do a one-point crossover, then undo the permutation to restore original gene positions |
| Matrix | Lay genes out in a matrix and exchange corresponding regions |
| Three-parent | If P1 and P2 agree at a position, use their bit; otherwise use P3’s bit |

Single-point crossover tends to keep nearby genes together, making the position of a gene matter. Uniform crossover is less tied to position but may disrupt useful groups of genes. Even a valid binary child may decode to an **infeasible** solution, so check the original constraints.

### Real-valued crossover

For real parents $P_1=15.65$, $P_2=18.83$, **linear crossover** creates children using $C=\alpha P_1+\beta P_2$. For instance, $\alpha=\beta=0.5$ gives the midpoint $C=17.24$. Weights summing to 1 give an affine mixture; weights outside $[0,1]$ can produce a child outside the parent interval. Check variable bounds afterward.

**Blend crossover (BLX-$\alpha$)** can sample from the expanded interval

$$
[P_{\min}-\alpha d,\;P_{\max}+\alpha d],\qquad d=|P_2-P_1|.
$$

For the same parents and $\alpha=0.5$, the interval is `[14.06, 20.42]`. Clip or otherwise handle children that exceed the actual variable bounds. The slides also describe $\gamma=(1+2\alpha)r-\alpha$, $r\in[0,1]$, and $C_1=(1-\gamma)P_1+\gamma P_2$; drawing $r$ uniformly gives a child in this expanded interval.

**Simulated binary crossover (SBX)** samples a nonnegative spread factor $\beta$. It makes two children centered on the parent midpoint, separated by $\beta |P_2-P_1|$:

$$
C_{1,2}=\tfrac12\bigl(P_1+P_2\mp\beta|P_2-P_1|\bigr).
$$

$\beta<1$ makes children closer together than the parents; $\beta>1$ spreads them farther apart. A distribution parameter controls how often each happens. In the slides, $\beta=1.0772$ yields about `15.53` and `18.95`. “Simulated binary” describes the offspring distribution, **not** a conversion of real parents into binary strings.

### Permutation crossover: preserve each item exactly once

A plain bit-string cut can duplicate or omit cities. With `P1 = [A,B,C,D,E]` and `P2 = [C,E,B,A,D]`, a **one-point order crossover** after two genes keeps `[A,B]` from P1. Scan P2 left to right, skipping A and B: `[C,E,D]`. The child is `[A,B,C,E,D]`, a valid permutation.

| Method | What it preserves |
| --- | --- |
| One-point order | Prefix from one parent; relative order of remaining items from the other |
| Two-point order | A middle segment from one parent; fill remaining slots with unused items from the other, in order |
| Partially mapped (PMX) | A copied segment plus position mappings to resolve duplicate values |
| Position-based | Specific positions from one parent; fill empty positions in the other parent’s order |
| Precedence-preserving (PPX) | Repeatedly take the first remaining item from the chosen parent and remove that item from both parents |
| Edge recombination | Favor parent-to-parent city adjacencies using an edge table |

For **edge recombination**, create each city’s neighbor list from both parents, choose a starting city, then prefer an unvisited neighbor with the fewest remaining choices; break ties by a stated rule. On a graph with unavailable connections, verify every chosen edge and the closing edge. Valid permutation alone does not ensure a feasible TSP route.

---

## 6. Mutation, inversion, and stopping

**Mutation** introduces occasional new variation after crossover. It helps the search try answers that recombination alone may not produce. It does not guarantee an escape from a local optimum.

- **Binary bit flip:** each chosen bit changes `0↔1`. A mask such as `0010` applied to `1011` gives `1001` (bitwise XOR). Mutation rates are often small; a useful way to think about $p_m$ is the expected number of flipped bits $L p_m$ per $L$-bit chromosome.
- **Swap:** exchange two positions. For a permutation `[A,B,C,D]`, swapping B and D gives `[A,D,C,B]` while remaining valid.
- **Segment reversal / inversion:** reverse a chosen section. `[A,B,C,D,E]` with positions 2–4 reversed becomes `[A,D,C,B,E]`. Reversing a single bit does nothing; reversal mainly makes sense for several positions or an ordered chromosome.
- **Real random mutation:** the slides use $x'=x+(r-0.5)\Delta$, $r\in[0,1]$. With $x=15.6$, $r=0.7$, and $\Delta=2.5$, the result is **16.1**. This formula changes the value by at most $\Delta/2$; clip or repair it when required by bounds.
- **Real polynomial mutation:** the slides use a random $r$, exponent $q>0$, and $x'=x+\delta\Delta$, where

  $$
  \delta=\begin{cases}
  (2r)^{1/(q+1)}-1,&r<0.5,\\
  1-[2(1-r)]^{1/(q+1)},&r\geq0.5.
  \end{cases}
  $$

  For $x=15.6$, $r=0.7$, $q=2$, $\Delta=1.2$, $\delta\approx0.1566$, giving $x'\approx15.788$. Larger positive $q$ generally favors smaller changes. Bounds still need handling.

Common stopping rules: required quality reached, a fixed generation count, time/evaluation budget reached, or no improvement for a chosen number of generations. Stopping because scores have leveled off is a practical decision, **not** proof that the best possible answer was found.

---

## 7. Fitness scaling: control how strongly scores affect selection

Fitness scaling changes the **scores used for selection**, not what the original objective or feasible set means. With roulette selection, `[1, 2, 100]` lets one candidate dominate; `[100, 101, 102]` gives almost equal probabilities even though one is better. Scaling can temper the first situation or accentuate the second.

| Approach | Basic idea | Care needed |
| --- | --- | --- |
| Linear | $F_i'=aF_i+b$, usually $a>0$ | Keep selection scores nonnegative and their order correct |
| Sigma | Use the population mean $\bar F$ and standard deviation $\sigma$ as a reference | State whether high or low original values are good |
| Power law | $F_i'=F_i^k$ for nonnegative $F_i$ | $k<1$ compresses ratios; $k>1$ enlarges them |

For example, with fitness `[100, 101, 102]`, subtracting 99 gives `[1, 2, 3]`: candidate 3 now has a larger selection advantage, while ranking stays the same. With `[1, 2, 100]`, taking square roots gives `[1, 1.414, 10]`, reducing its dominance. These are illustrations, not universal settings.

**Direction matters for sigma scaling.** The slides show a *cost minimization* version: let $w=\bar c+S\sigma$, then set $F_i'=\max(0,w-c_i)$, where smaller cost $c_i$ gives larger selection score. If the raw quantity is a *fitness to maximize*, one possible analogous threshold is $F_i'=\max(0,F_i-(\bar F-S\sigma))$. Do not use the cost formula on a maximize-fitness problem: it would reward the worst candidates. The constant $S$ controls the cutoff and must be chosen for the problem.

**Practical rule:** after any scaling, check that feasible better candidates still score at least as well as feasible worse ones, that scores intended for roulette are nonnegative, and that their sum is positive.

---

## 8. Quick exam checklist

1. **Formulate:** state variables, optimize what, and list every constraint.
2. **Choose encoding:** bits for selection, numbers for real parameters, permutations for ordering, trees for hierarchy.
3. **Decode before scoring:** state exactly what a chromosome means and test feasibility.
4. **Select correctly:** for roulette, divide each nonnegative fitness by the total and use cumulative intervals; for tournament, choose the best within each random group.
5. **Apply the right crossover:** binary segment swap for bit strings, numerical mixing for real values, duplicate-safe methods for permutations.
6. **Mutate sparingly, handle bounds, and keep the best-so-far answer.**
7. **Know the limitation:** a GA returns a strong candidate, not a guaranteed optimum in general.

### Corrections to keep in mind when revising from the slides

- A polynomial of degree $m$ has at most **$m-1$** stationary points over the reals, since its derivative has degree $m-1$; the slides’ “$m$ candidate points” claim is too strong.
- For a stationary point, classify using the **first nonzero higher derivative**: even order can be a minimum or maximum; odd order is neither. Some text in GA-01 reverses or muddles this test.
- The BLX expanded interval ends at $P_{\max}+\alpha d$. GA-05 prints a minus sign for that upper endpoint. In its linear crossover example, $-0.5(15.65)+1.5(18.83)=20.42$, despite one printed value of `20.24`.
- GA-06’s worked polynomial mutation result uses **addition**: $15.6+0.1566(1.2)\approx15.788$. Its printed intermediate line incorrectly shows multiplication.
- GA-06’s sigma formula favors **lower raw cost**. For a maximize-fitness GA, reverse the direction as explained above.

**Slide map:** GA-01: optimization and calculus; GA-02: GA workflow and update strategies; GA-03: representations; GA-04: fitness and selection; GA-05: crossover; GA-06: mutation, stopping, and scaling.
