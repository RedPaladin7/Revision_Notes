# Database Interview — Weekly Revision Guide

Synthesized from `sample.md`. You already read the long notes. Use this **every week** as a timed recitation pass. Every critical fact is here; examples are compressed, not dropped.

**How to use (one full pass per week, ~40 min/day):**

| Day | Cover | Recite until you can say it without looking |
|-----|--------|---------------------------------------------|
| **Mon** | ACID, tx states, 4 anomalies, isolation + MVCC | ACID nuance, 4 anomalies, isolation table + defaults |
| **Tue** | Locking, 2PL, serializability | Compatibility matrix, 2PL vs deadlocks, precedence graph |
| **Wed** | Indexing | B vs B+, clustered, left-prefix, when optimizer ignores |
| **Thu** | Normalization | 1NF→BCNF, lossless test, why not always BCNF |
| **Fri** | SQL deep | Exec order, windows, NOT IN vs NOT EXISTS, NULLs |
| **Sat** | Deadlocks + WAL/ARIES | Coffman, Wait-Die/Wound-Wait, WAL rules, ARIES 3 phases |
| **Sun** | Query processing + storage + cheat sheet | Join algos, EXPLAIN flags, buffer pool, row vs column |

Each day: **Recite** the boxed facts → **Scan** the section → **Answer** the self-quiz at the end of that day. Sunday: run the full cheat sheet out loud.

---

# DAY 1 — Transactions, Anomalies, Isolation

## Transaction

A **unit of work**: a group of operations treated as one indivisible thing. Canonical: bank transfer — debit A, credit B. If 1 succeeds and 2 fails, money vanishes. Either both happen, or neither.

```sql
BEGIN;
  UPDATE accounts SET balance = balance - 500 WHERE id = 'A';
  UPDATE accounts SET balance = balance + 500 WHERE id = 'B';
COMMIT;
```

## ACID — one by one

| | Meaning | Intuition | Whose job |
|---|---|---|---|
| **A** Atomicity | All or nothing. Outside world never sees a partial state. | Git commit — no half-committed history | **DB only** |
| **C** Consistency | DB moves from one **valid** state to another. Constraints, FKs, checks, business invariants hold before and after. | Schema `balance >= 0` cannot be left negative | **Shared:** DB enforces declared constraints; **app** writes correct business logic |
| **I** Isolation | Concurrent txs should not interfere. Each behaves as if it is the only one running. | Two buyers, last item — only one succeeds | **DB only** (degrees exist — isolation levels) |
| **D** Durability | Once committed, it stays committed even if crash is 1 ms later. Non-volatile storage. | “Order confirmed” email must survive a crash | **DB only** — enforced by **WAL** (Day 6) |

**CAP “C” ≠ ACID “C”.** Know this if asked. ACID C = validity of data/constraints. CAP C = all nodes see the same data at the same time.

**Interview line:** “Consistency is the only ACID property that is partially the application’s responsibility. The DB enforces declared constraints; business-logic correctness is on the developer.”

## COMMIT / ROLLBACK / SAVEPOINT

- **COMMIT** — finalize; permanent and visible
- **ROLLBACK** — undo all since BEGIN (or since a SAVEPOINT)
- **SAVEPOINT** — named intermediate point; partial rollback

```sql
BEGIN;
  UPDATE orders SET status = 'shipped' WHERE id = 42;
  SAVEPOINT sp1;
  INSERT INTO shipments ...  -- fails
  ROLLBACK TO sp1;           -- undo only the insert; orders update still pending
COMMIT;                      -- orders update persists
```

**Engine difference:** PostgreSQL — a failed statement puts the **transaction** in an error state; you **must ROLLBACK**. MySQL is more lenient and auto-handles some errors. Know your DB.

## Transaction states

```
Active ──last stmt──▶ Partially Committed ──commit──▶ Committed ──▶ Terminated
   │                                                              ▲
   └──error──▶ Failed ──rollback──▶ Aborted ──────────────────────┘
```

After **Aborted**:
1. **Restart** if failure was non-deterministic (deadlock)
2. **Kill** if deterministic (logic error)

## The four concurrency problems (know cold)

Mental model: **Lost Update** = two writers clobber. **Dirty Read** = uncommitted garbage. **Non-Repeatable Read** = same *row*, different value. **Phantom** = same *query*, different *set of rows*. If you blank, draw a two-column timeline (T1 | T2).

### 1. Lost Update — two writers, last writer wins

```
T1: READ X=100          T2: READ X=100
T1: WRITE X=150 (+50)   T2: WRITE X=120 (+20)
Final X=120  ← T1 silently erased
```

Real world: two people edit the same Wikipedia paragraph.

### 2. Dirty Read — uncommitted data that later vanishes

```
T1: WRITE X=200 (uncommitted)
T2: READ X=200
T1: ROLLBACK  (X back to 100)
T2 continues with 200, which never officially existed
```

“Dirty” = modified but not validated/committed.

### 3. Non-Repeatable Read — same row, committed change between reads

```
T1: READ X → 100
T2: WRITE X=200, COMMIT
T1: READ X → 200
```

**vs dirty:** dirty = uncommitted. NRR = committed, but changed between your reads.

### 4. Phantom Read — set of rows changes

```
T1: SELECT * FROM orders WHERE amount > 100  → 5 rows
T2: INSERT order amount=500, COMMIT
T1: same SELECT → 6 rows  (phantom)
```

NRR = existing *row* value changed. Phantom = *set of rows* changed (insert/delete matching the predicate). Real world: sum orders twice; a new order lands between sums.

## Isolation levels (SQL standard: four)

Full isolation (Serializable) is expensive (more waiting). Tunable spectrum.

| Level | Dirty | NRR | Phantom | Prevents | Allows | Notes |
|---|---|---|---|---|---|---|
| **Read Uncommitted** | possible | possible | possible | Nothing | Everything including lost updates | Approximate analytics only. Extremely rare. |
| **Read Committed** | **prevented** | possible | possible | Dirty reads | NRR, phantoms | **Default: PostgreSQL, Oracle, SQL Server** |
| **Repeatable Read** | prevented | **prevented** | possible* | Dirty + NRR | Phantoms (theory) | **Default: MySQL InnoDB.** *InnoDB also prevents phantoms via **gap locks** |
| **Serializable** | prevented | prevented | **prevented** | All anomalies including lost updates | Nothing bad | Highest |

**Read Committed implementation (MVCC):** each **statement** sees a snapshot of committed data as of when *that statement* started. Different statements in the same tx can see different data.

**Repeatable Read implementation (MVCC):** snapshot taken at **transaction start**. Every read in the tx sees the same snapshot.

**Serializable:**
- **PostgreSQL:** **SSI** (Serializable Snapshot Isolation) — detect and **abort** txs that would create a serialization anomaly; not heavy traditional locking
- **MySQL:** **range locks / gap locks** to block phantom inserts

## MVCC — why modern DBs don’t lock for reads

**Multi-Version Concurrency Control:** multiple versions of each row. Readers see the version for their snapshot. Writers create new versions.

**Result:** Readers never block writers. Writers never block readers. Only **write-write** conflicts need locking.

PostgreSQL, MySQL InnoDB, Oracle — all MVCC. Reads don’t block even at high isolation.

**Interview:** PG default = Read Committed. MySQL = Repeatable Read. RC vs RR in MVCC: RC = **fresh snapshot per statement**; RR = **one snapshot for the whole transaction**.

### Day 1 self-quiz
1. Which ACID property is shared with the application? How is CAP C different?
2. PG vs MySQL after a failed statement inside a tx?
3. Draw dirty vs NRR vs phantom vs lost update.
4. Isolation defaults: PG, Oracle, SQL Server, MySQL.
5. How does PG implement Serializable vs MySQL?
6. What does MVCC buy you? What still needs locks?

---

# DAY 2 — Locking, 2PL, Serializability

## Why locks

Before accessing data, acquire a lock. Conflicting lock → wait. Most common isolation mechanism.

## Shared vs Exclusive

| | S held | X held |
|---|---|---|
| Request **S** (read) | compatible | wait |
| Request **X** (write) | wait | wait |

- **S-lock:** “I’m reading. Others can read. No one writes.” Multiple S on same data OK.
- **X-lock:** only ONE holder. Blocks all readers and writers.

**S + S = fine. Anything + X = conflict.**

## Two-Phase Locking (2PL)

Protocol that guarantees **conflict serializability**.

- **Growing:** acquire locks, **cannot release any**
- **Shrinking:** release locks, **cannot acquire any**
- First lock release → shrinking **permanently**
- **Lock point** = moment of maximum locks held

```
Locks: 0 → 1 → 3 → 5 → 5 → 3 → 1 → 0
           [growing]    [shrinking]
                    ^ lock point
```

**Why serializable:** lock points create a natural ordering. Following 2PL cannot produce a cycle in the conflict graph.

**2PL does NOT prevent deadlocks.** It prevents serializability violations. Deadlocks are separate: two txs each hold a lock the other needs. 2PL **increases** deadlock risk because locks are held longer.

## Strict 2PL vs Rigorous 2PL

**Standard 2PL → cascading aborts:**

T1 writes X, **releases X-lock before commit** → T2 reads T1’s X → T1 aborts → T2 must abort → anyone who read T2 aborts → cascade.

**Strict 2PL:** hold **ALL exclusive (write) locks** until COMMIT or ROLLBACK. Prevents dirty reads: you cannot read T1’s writes until T1 commits (and releases).

**Rigorous 2PL:** hold **ALL locks (S and X)** until commit. Most real DBs (PG, MySQL) behave like this: commit → all locks released at once.

## Serializability — core

A **schedule** = interleaved sequence of reads/writes from concurrent txs.

**Serializable** = outcome equivalent to **some** serial execution (one tx fully, then the next). They need not actually run sequentially — result must be *as if* they did.

## Conflict serializability

Two ops **conflict** iff:
1. Different transactions
2. Same data item
3. At least one is a **WRITE**

Pairs: **RW**, **WR**, **WW**. Two reads **never** conflict.

**Conflict equivalent:** same conflicting operations in the same relative order.

**Conflict serializable:** conflict equivalent to some serial schedule.

## Precedence graph (interview algorithm — do mechanically)

1. One node per transaction
2. For each conflicting pair (Ti op1 **before** Tj op2): edge **Ti → Tj**
3. **No cycle → conflict serializable.** Cycle → not.
4. If acyclic, **topological sort** = equivalent serial order

Do not eyeball. One missed edge = wrong answer.

### Example — NOT serializable

Schedule: `R1(X), R2(Y), W1(Y), W2(X)`

- R1(X) before W2(X) → T1 → T2
- R2(Y) before W1(Y) → T2 → T1
- Cycle → **not** conflict serializable

### Example — serializable

Schedule: `R1(X), W1(X), R2(X), W2(X)`

- W1 before R2 → T1 → T2
- W1 before W2 → T1 → T2
- No reverse edge → equivalent serial order **T1 then T2**

## View serializability (weaker — “did everyone see the same thing?”)

Conflict serializability looks at **every conflicting pair** and their **order**. View serializability is looser. It only asks:

> If we ran these transactions one after another (some serial order), would each transaction **read the same values**, and would the database **end with the same final values**?

If yes, the interleaved schedule is **view serializable**. The transactions *viewed* the world the same way they would have in that serial order.

**Three checks** (all must match some serial schedule S'):

1. **Initial reads.** If T1 read X when nobody had written X yet (the original value), T1 must also read that original X in S'.
2. **Who did I read from?** If T1 read an X that T2 wrote, then in S' T1 must still read T2’s X — not someone else’s, not the original.
3. **Final writes.** Whoever wrote the **last** value of X in the interleaved schedule must also be the last writer of X in S'. (That is the value that “sticks.”)

Think of it as: **same reads + same final result**, not “same order of every write.”

**Why is this weaker than conflict serializability?** Because of **blind writes** — a transaction writes X **without reading it first**. It just overwrites. Then the *order* of those overwrites can differ from a conflict-serializable order, but as long as **the same transaction writes last**, and nobody read the intermediate values, the *view* is the same.

Tiny example of the idea:

```
T1: W(X)     -- blind write, never reads X
T2: W(X)     -- also blind
T3: W(X)     -- last writer; this is the final X everyone will see later
```

If T3 is always the last writer, the “final X” is T3’s value either way. Intermediate overwrites that nobody read don’t change what anyone *viewed*. Conflict serializability still records every WW pair and their order; view serializability ignores those pairs when nobody read the in-between values. (Simple all-blind-write schedules are often *also* conflict serializable — the extra view-only schedules show up in slightly richer blind-write examples. For interviews, the distinction and the three checks matter more than inventing a cycle.)

**View serializable ⊃ Conflict serializable** — every conflict-serializable schedule is view serializable. The extra ones are the rare blind-write cases.

**Why databases ignore view serializability:** checking it is **NP-hard**. Conflict serializability is just “draw the graph, look for a cycle” — **polynomial**. Engines therefore test **conflict** serializability. In interviews: explain the three view conditions, then say engines use the precedence graph.

### Day 2 self-quiz
1. Draw the S/X compatibility matrix.
2. Growing vs shrinking. What is the lock point?
3. Does 2PL prevent deadlocks? Why does it increase deadlock risk?
4. Strict vs rigorous 2PL. Which do real DBs use? How do cascading aborts happen?
5. Three conditions for conflict. Run the precedence-graph algorithm on both examples.
6. View serializability in three checks. What is a blind write? Why do engines test conflict, not view?

---

# DAY 3 — Indexing

## Why indexes

No index → `WHERE email = ...` on 50M rows = **full table scan**. Index = separate pre-organized structure mapping values → physical locations. Trade-off: **extra storage + slower writes → much faster reads**. Textbook index: book unchanged; index is a navigation structure.

## What an index actually looks like

The **table** (heap, or clustered primary) stores full rows, in whatever physical order they live on disk. The **index** is a *second* structure: sorted copies of **one (or a few) columns** + a **pointer** to the real row.

Heap table `users` (pages store whole rows, **not** sorted by email):

```
Page 1:  [id=3, email=carol@x.com, name=Carol, city=Delhi]
Page 2:  [id=1, email=alice@x.com, name=Alice, city=Pune]
Page 3:  [id=2, email=bob@x.com,   name=Bob,   city=Delhi]
```

Non-clustered index on `email` — leaves are **sorted by email**, each entry is `(key → row pointer)`:

```
email         pointer
------------  ----------------
alice@x.com → Page 2, slot 1
bob@x.com   → Page 3, slot 1
carol@x.com → Page 1, slot 1
```

`WHERE email = 'bob@x.com'`: walk the B+ tree (few pages) → pointer → fetch **one** table page. You never scan Page 1 and Page 2.

A clustered index on `id` **is** the table. Leaves hold full rows, sorted by `id`:

```
id  email        name   city    ← these ARE the rows, in id order
1   alice@x.com  Alice  Pune
2   bob@x.com    Bob    Delhi
3   carol@x.com  Carol  Delhi
```

No separate “go fetch the row” step for a PK lookup — you are already on the row.

## B-Tree vs B+ Tree (almost every DB interview)

| | B-Tree | B+ Tree |
|---|---|---|
| Data / row pointers | Internal **and** leaf | **Leaves only** |
| Internal nodes | Keys + data | Keys only (navigation) → **higher fanout** |
| Early terminate | Yes, if key at internal | Always go to leaf |
| Same-level links | No — range queries awkward | **Leaves doubly linked** → range = sequential leaf scan |
| Height | Deeper for same N | Shallower |

**Why B+ dominates:**
- **Fanout:** more keys per internal node → shallower tree → fewer disk I/Os. `Height ≈ log_B(N)`. B=1000, N=1e9 → height ≈ **3** = 3 disk reads for any row in 1 billion.
- **Range:** `WHERE age BETWEEN 20 AND 30` → first leaf age=20, walk linked list until age>30. No backtracking.
- **Sequential, cache-friendly** leaf scans.

**Every major RDBMS uses B+ trees for indexes:** PostgreSQL, MySQL InnoDB, SQL Server, Oracle.

## Clustered vs Non-Clustered

**Clustered:** table rows **physically stored in the sorted order of the index key**. Only **one** per table (a pile of paper can only be sorted one way). Lookup: traverse B+ → you **are at the data**. Done.

- **MySQL InnoDB:** primary key **IS** the clustered index
- **PostgreSQL:** heap storage, **no true clustered index**; you can `CLUSTER` a table (one-shot reorder, not maintained)

Intuition: dictionary — word and definition physically together.

**Non-clustered:** a **separate** structure. Each entry is the key + a **pointer** to the real row. Many per table. Pointer = physical row ID (**PostgreSQL**) or the clustered PK value (**InnoDB**). Two-step: traverse index → **bookmark lookup / key lookup** to fetch the row.

Intuition: library card catalog — sorted cards, then a walk to the shelf.

### Why “many rows → I/O adds up” is a *non-clustered* problem

For **one** row (`WHERE email = 'bob@x.com'`), clustered and non-clustered are close. Non-clustered pays one extra hop (index → table). That hop is cheap.

The pain starts when the index matches **many** rows, e.g. `WHERE city = 'Delhi'` returning 50,000 people.

**Non-clustered on `city`:** the index is sorted by city, so the 50,000 Delhi *keys* sit together in the index. Their **table rows do not**. Alice might be on page 2, Carol on page 1, someone else on page 9000. Each bookmark lookup is a **random** jump to a different heap page. 50,000 jumps → tens of thousands of random I/Os. Random I/O is the expensive kind.

**Clustered on `city`:** Delhi rows are **physically next to each other** on disk. Find the first Delhi leaf, then read the next pages sequentially until city changes. 50,000 rows might be ~200 consecutive pages. Sequential I/O + prefetch. That is why clustered (or a covering non-clustered index that never visits the heap) is so much faster for **range / many-row** lookups.

| | 1 row | 50,000 rows matching the key |
|---|---|---|
| Non-clustered | index + 1 random heap fetch | index + **tens of thousands of random heap fetches** |
| Clustered on that key | already on the row | **sequential** scan of neighboring table pages |

### InnoDB secondary indexes (critical)

Secondary indexes store the **PK value**, not a physical RID.

1. Traverse secondary B+ → get PK
2. Traverse primary (clustered) B+ → get row

**Two tree traversals.** Small PK matters — PK is embedded in **every** secondary index.

## Composite indexes — Left-Prefix Rule

Index `(A, B, C)` usable for: **A**, **A+B**, **A+B+C**. **Not** B alone, C alone, B+C.

Phone book (last, first): all “Kumars” yes; all “Abhinavs” no — scattered.

**Column order:**
- **Equality before range** — a range “breaks” the index for later columns
- **Higher selectivity first** (more distinct values / more filtering)

`WHERE status = 'active' AND created_at > '2024-01-01'` → `(status, created_at)` not the reverse.

## Covering indexes

Index **covers** a query if **all** columns needed (filter **and** return) live in the index. Table never touched. Smaller, more cacheable.

```sql
SELECT first_name, last_name FROM users WHERE email = '...';
CREATE INDEX idx_cov ON users (email, first_name, last_name);
```

MySQL EXPLAIN: `Using index`. PostgreSQL: `Index Only Scan`.

## When an index IS useful

- High cardinality: `email`, `user_id`, `order_number`
- Columns in WHERE, JOIN ON, ORDER BY, GROUP BY, frequently filtered
- Small-range / high-selectivity queries
- Covering-index cases

## High cardinality vs low cardinality

**Cardinality** = how many distinct values the column has.

- **High:** `email` — millions of distinct values, each value matches ~1 row. Index says “here is the one page.” Tiny I/O vs scanning the whole table. **This is what indexes are for.**
- **Low:** `is_active` (true/false), `gender`, `status` with 5 values. `WHERE is_active = true` might match **80% of the table**. The index has a huge list of pointers, and following them is random I/O across almost every page. A full table scan reads each page **once, in order**, and checks `is_active` in memory. Cheaper.

**Selectivity** = fraction of rows that match. High selectivity (few rows match) → index. Low selectivity (lots of rows match) → sequential scan.

## Why a function on an indexed column breaks the index

The B+ tree is sorted by the **raw stored values**: `2024-03-15`, `2024-06-01`, `2025-01-10`, …

`WHERE YEAR(created_at) = 2024` asks a different question: “rows whose *year extracted from the date* is 2024.” The tree is **not** sorted by `YEAR(...)`. The engine cannot seek to “2024” in that tree. It would have to compute `YEAR` on **every** row — which is a full scan, so it often ignores the index entirely.

Same idea: `WHERE LOWER(email) = 'a@x.com'`, `WHERE age + 1 > 26`, `WHERE CAST(user_id AS text) = '123'`. You wrapped the column; the sorted order no longer matches the predicate.

**Fix:** put the column **bare** on one side, and move the computation to the constant:

```sql
-- bad: function on the column
WHERE YEAR(created_at) = 2024

-- good: range on the raw column — index can seek to 2024-01-01 and scan until 2025
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01'
```

(You *can* build a **function/expression index** on `YEAR(created_at)` if you really need that predicate. That is a *different* tree, sorted by the function’s output.)

## When an index IS NOT useful

- **Low cardinality** — see above
- **Small tables:** traversing an index vs scanning 100 rows isn’t worth it
- **Leading wildcard:** `LIKE '%kumar'` cannot use B+ (unknown leading bytes). `LIKE 'kumar%'` **can**
- **Functions on indexed columns** — see above
- **Implicit type conversion:** `WHERE user_id = '123'` when `user_id` is INT — conversion often kills index use
- **Write-heavy, read-rare columns:** maintain the index on every INSERT/UPDATE/DELETE, maybe never read it

## Index scan vs full table scan

Your intuition is **right**: sequential vs random is the reason. One correction on “discard.”

Heap stored in insertion order, 2 words per page:

```
Page 1: elephant, ball
Page 2: deer, apple
Page 3: cat
```

**Full table scan** (`SELECT *` or a filter that matches most rows): read Page 1, then 2, then 3. Sequential. Disk/OS prefetch the next page while you process this one. Each page is read **once**. You do see words you might not need, but you never jump around.

**Index on the word (sorted):** `apple → Page 2`, `ball → Page 1`, `cat → Page 3`, `deer → Page 2`, `elephant → Page 1`.

To fetch rows *through* that index (e.g. “give me everything in alphabetical order,” or a low-selectivity filter):

1. Index says apple → **jump to Page 2**, take apple (deer happens to sit next to it on that page)
2. Index says ball → **jump to Page 1**, take ball
3. Index says cat → **jump to Page 3**
4. Index says deer → **jump to Page 2 again**
5. Index says elephant → **jump to Page 1 again**

Random order, pages **re-read**, no prefetch. That is slower than the three sequential reads — even though the index is “using an index.”

You don’t really “fetch apple+cat then discard cat” unless apple and cat shared an **index leaf** *and* you then followed both pointers. The waste is the **heap jumps**, not extra words on the index page.

- **Index scan:** great for **few** rows. At roughly **>10–20%** of a large table, random heap I/O loses to one sequential pass. Optimizer uses statistics. `WHERE status='active'` with 80% active often **ignores** your index.

**“Why did the optimizer ignore my index?” (90% of cases):**
1. Low selectivity — too many rows
2. Stale stats — `ANALYZE` (PG) / `ANALYZE TABLE` (MySQL)
3. Function on the column
4. Implicit type conversion

### Day 3 self-quiz
1. Sketch a heap page vs an email index (key → pointer). Why B+ over B? Height with B=1000, N=1e9.
2. One-row lookup: clustered vs non-clustered. 50k-row lookup: why clustered (or covering) is much faster.
3. Left-prefix: which predicates use (A,B,C)? InnoDB secondary = two tree walks. Why small PK?
4. High vs low cardinality. Why `YEAR(created_at)` breaks an index. Sargable rewrite.
5. Elephant/ball/deer example: why sequential heap scan beats following an index for most of the table. Crossover %.
6. Four reasons the optimizer ignores an index. `LIKE '%x'` vs `'x%'`. `id = '123'`.

---

# DAY 4 — Normalization

## Functional dependencies

**A → B:** knowing A uniquely determines B.

- `student_id → student_name` yes
- `student_name → student_id` no (duplicate names)
- `(order_id, product_id) → quantity` yes

**Superkey:** attribute set whose **closure = all attributes**. Uniquely identifies a row.

**Candidate key:** **minimal** superkey — drop any attribute, no longer a superkey.

**Closure X⁺:** all attributes derivable from X via given FDs. If A→B, B→C then `{A}⁺ = {A,B,C}`.

## Why normalize — three anomalies

Redundancy causes:

- **Insertion:** cannot record a fact without another (can’t add a department without an employee)
- **Update:** change one fact in many rows; miss one → inconsistent (dept city on every employee)
- **Deletion:** delete last employee → department info gone

Fix: decompose so each fact is stored **once**.

## 1NF

All attributes **atomic**. No multi-valued attributes, repeating groups, arrays in cells.

Violation: `Student(id, name, courses)` with `'Math, Physics, CS'`.

Fix: one row per course, or `StudentCourse(student_id, course)`.

Baseline: relational DBs technically enforce this.

## 2NF

**1NF + no partial dependency.** Every non-key attribute depends on the **entire** PK, not part of it.

**Only relevant when PK is composite.**

Violation: `OrderItem(order_id, product_id, quantity, product_name)` PK `(order_id, product_id)`. `product_name` depends only on `product_id`. Rename product → update every OrderItem row.

Fix: `OrderItem(order_id, product_id, quantity)` + `Product(product_id, product_name)`.

## 3NF

**2NF + no transitive dependency.** Non-key attributes must not depend on other non-key attributes.

Violation: `Employee(emp_id, dept_id, dept_name)`. `emp_id → dept_id`, `dept_id → dept_name`. Same `dept_name` on every employee in the dept.

Fix: `Employee(emp_id, dept_id)` + `Department(dept_id, dept_name)`.

**Formal 3NF:** for every non-trivial FD X → Y, **either** X is a superkey **or** Y is a **prime attribute** (part of some candidate key).

## BCNF — in plain language

**3NF said:** for every rule “X determines Y”, either X is a key of the whole row, **or** Y is *part of some key* (a “prime” attribute). That second door is the exception.

**BCNF closes that door.** Rule: if X determines Y, **X must be a key of the whole table**. No exceptions.

Even simpler: **whoever is allowed to decide another column’s value must uniquely identify the row.** You should not have a “side rule” where a non-key column dictates another column.

**The classic 3NF-but-not-BCNF situation:**

```
StudentAdvisor(student, advisor, major)
Meaning:
  - A given student in a given major has one advisor
    (student, major) → advisor
  - Each advisor works in only one major
    advisor → major

Keys of this table: (student, major)  and  (student, advisor)
```

Walk through `advisor → major`:
- Knowing the advisor tells you the major. Fine, that’s true in real life.
- Is `advisor` a **key of the whole row**? No. One advisor has many students. Advisor does not tell you *which student*.
- **BCNF:** fail. A non-key (`advisor`) determines something (`major`).
- **3NF:** pass, because `major` is part of a key `(student, major)` — the exception 3NF allows.

What goes wrong in practice: `major` is stored once per (student, advisor) pair. If an advisor’s specialty is recorded differently on two rows, the table disagrees with itself. BCNF would split this so `advisor → major` lives in a table where `advisor` **is** the key: `Advisor(advisor, major)` plus `Advises(student, advisor)`.

**Tradeoff:** that split is cleaner, but the original FD `(student, major) → advisor` now spans two tables. The database cannot check it with a single-table constraint. Sometimes you **cannot** have BCNF **and** keep every FD checkable **and** keep a lossless join. Then people stop at 3NF. Real schemas often stop at 3NF (or even 2NF) because extra joins cost.

## Lossless vs lossy decomposition — with real rows

**Decompose** = split one table into two (or more). You must be able to **join them back and get exactly the original rows**.

- **Lossless:** `R1 JOIN R2 = R`. No extra rows, no missing rows.
- **Lossy:** the join invents **spurious** rows that were never in R. Information is “lost” in the sense that you can no longer tell which combinations were real.

**The test:** split of R into R1 and R2 is lossless **iff** the **common columns** form a key of **at least one** of the pieces:

`(R1 ∩ R2) → R1`  **or**  `(R1 ∩ R2) → R2`

In English: the overlap must uniquely identify rows in R1 or in R2. Then the join cannot mix the wrong partners.

### Lossless example

```
R(A, B, C)     FD: A → B

A  B   C
1  10  5
2  20  5
```

Split into **R1(A, B)** and **R2(A, C)**. Common column = `{A}`. `A → B`, so A is a key of R1. Test passes.

```
R1          R2
A  B        A  C
1  10       1  5
2  20       2  5

R1 JOIN R2 on A:
A  B   C
1  10  5
2  20  5     ← exactly original R
```

Each A matches one B, so the join cannot invent a fake (A, B, C).

### Lossy example (same R, bad split)

Split into **R1(A, C)** and **R2(B, C)**. Common = `{C}`. C determines nothing. Test fails.

```
R1          R2
A  C        B   C
1  5        10  5
2  5        20  5

R1 JOIN R2 on C=5: every A pairs with every B
A  B   C
1  10  5     ← original
1  20  5     ← FAKE (A=1 never had B=20)
2  10  5     ← FAKE
2  20  5     ← original
```

Two real rows became four. You cannot tell which (A, B) pairs existed. That is **lossy**.

Same idea in words: both rows share C=5; after the split, “5” no longer remembers *which* B belonged to *which* A. The join cross-products them.

## Dependency preservation — in plain language

An FD is a **rule the database should enforce** (e.g. `advisor → major`: one advisor, one major).

After you split tables, **dependency preservation** means: each of those rules can still be checked by looking at **one table alone**, with no join.

- `Advisor(advisor, major)` with `advisor` as PK → the DB enforces `advisor → major` automatically. **Preserved.**
- `(student, major) → advisor` after the BCNF split: checking the rule needs a **join**. The engine will not run that join on every insert. The rule can be violated unless the **application** checks it. **Not preserved.**

Why it matters: if a constraint requires a join, it will quietly stop being a database constraint.

**3NF** always has a decomposition that is **both lossless and dependency-preserving** (3NF synthesis algorithm). **BCNF** always can be lossless; it does **not** always preserve every FD.

**Why not always BCNF?** (1) You may lose the ability to enforce some FDs inside the DB. (2) More tables → more joins, and joins cost. OLAP often **deliberately denormalizes** (star schema, wide tables) to avoid joins on analytical queries.

### Day 4 self-quiz
1. Superkey vs candidate key vs closure.
2. Three anomalies with one-line examples.
3. 1NF / 2NF / 3NF / BCNF rules. When does 2NF even apply?
4. Formal 3NF vs BCNF. Recite StudentAdvisor.
5. Lossless test. Give lossless and lossy splits of R(A,B,C), A→B.
6. Why not always BCNF? Does 3NF always preserve FDs?

---

# DAY 5 — SQL Deep

## JOINs

```sql
-- INNER JOIN: only rows with matches in both tables
SELECT u.name, o.amount
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN: all left rows + matching right rows (NULL where no match)
SELECT u.name, o.amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
-- Users with no orders still appear; o.amount = NULL

-- FULL OUTER JOIN: all rows from both; NULLs where no match on either side
-- MySQL has no direct FULL OUTER — UNION of LEFT and RIGHT instead
SELECT u.name, o.amount
FROM users u
FULL OUTER JOIN orders o ON u.id = o.user_id;

-- CROSS JOIN: cartesian product (every left row × every right row)
SELECT * FROM colors CROSS JOIN sizes;  -- generate all combinations

-- SELF JOIN: join a table to itself
SELECT e.name, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

- **INNER:** only matching rows both sides
- **LEFT:** all left + matching right; NULL on right if no match
- **FULL OUTER:** all rows both sides. **MySQL:** emulate with `UNION` of LEFT and RIGHT
- **CROSS:** cartesian product
- **SELF:** table joined to itself (employee ↔ manager)

## Logical execution order (critical)

```
FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT
```

**WHERE vs HAVING:** WHERE filters **rows before** grouping. HAVING filters **groups after** aggregation. Cannot `WHERE COUNT(*) > 5` — that is HAVING.

```sql
SELECT dept, COUNT(*) AS emp_count, AVG(salary) AS avg_sal
FROM employees
WHERE hire_date > '2020-01-01'     -- filter rows BEFORE grouping
GROUP BY dept
HAVING COUNT(*) > 5                -- filter groups AFTER aggregating
ORDER BY avg_sal DESC;
```

**SELECT restriction:** with GROUP BY, every SELECT column must be in GROUP BY **or** in an aggregate. **PostgreSQL is strict.** **MySQL relaxed mode** picks an **arbitrary** value for ungrouped columns — dangerous.

## Subqueries

**Uncorrelated:** runs **once**; outer query uses that result. Independent of any outer row.

```sql
SELECT name FROM employees
WHERE dept_id IN (SELECT id FROM departments WHERE location = 'NYC');
```

**Correlated:** the inner query **mentions a column from the outer row**, so it re-runs **once per outer row**. Can be slow on large tables.

```sql
-- employees earning above their own department's average
SELECT name, salary FROM employees e
WHERE salary > (
    SELECT AVG(salary) FROM employees
    WHERE dept = e.dept          -- refers to outer alias e
);
```

If there are 10,000 employees, that average may be computed ~10,000 times (once per person, even though there are only a handful of departments).

**Rewrite as one scan + a window** (compute each dept average once, then filter):

```sql
SELECT name, salary
FROM (
  SELECT
    name,
    salary,
    AVG(salary) OVER (PARTITION BY dept) AS dept_avg
  FROM employees
) t
WHERE salary > dept_avg;
```

`PARTITION BY dept` is “GROUP BY that does not collapse rows.” Every employee keeps their own row, plus a `dept_avg` column. One pass over `employees`, then a simple filter. Same idea as JOIN against a grouped subquery:

```sql
SELECT e.name, e.salary
FROM employees e
JOIN (
  SELECT dept, AVG(salary) AS dept_avg
  FROM employees
  GROUP BY dept
) d ON e.dept = d.dept
WHERE e.salary > d.dept_avg;
```

Rule of thumb: correlated subquery on a hot path → try window or JOIN+GROUP BY.

## CTEs — Common Table Expressions

**CTE = Common Table Expression.** The `WITH ... AS (...)` clause. You give a **name** to a query result and then use that name like a temporary table **in the same statement**. It is not stored for later sessions; it lives only while this query runs.

Why it exists: nested subqueries get unreadable (`SELECT ... FROM (SELECT ... FROM (SELECT ...))`). A CTE is the same idea with a label.

```sql
WITH
  active_users AS (
    SELECT id, name FROM users
    WHERE last_login > NOW() - INTERVAL '30 days'
  ),
  user_order_totals AS (
    SELECT user_id, SUM(amount) AS total
    FROM orders
    GROUP BY user_id
  )
SELECT au.name, uot.total
FROM active_users au
JOIN user_order_totals uot ON au.id = uot.user_id;
```

Read it top to bottom: “first, the set of active users; second, each user’s order total; finally, join those two.” You can chain CTEs — later ones can read earlier ones.

**Recursive CTE:** the name refers to **itself**, so you can walk a tree or graph (org chart, bill of materials). Two parts glued with `UNION ALL`:

1. **Anchor** — starting rows (the CEO: no manager). Runs once.
2. **Recursive step** — “given people we already found, find their direct reports.” Repeats until no new rows.

```sql
WITH RECURSIVE org_tree AS (
  -- Anchor
  SELECT id, name, manager_id, 0 AS depth
  FROM employees
  WHERE manager_id IS NULL

  UNION ALL

  -- Recursive: employees whose manager is already in org_tree
  SELECT e.id, e.name, e.manager_id, ot.depth + 1
  FROM employees e
  JOIN org_tree ot ON e.manager_id = ot.id
)
SELECT * FROM org_tree ORDER BY depth;
```

**CTE vs subquery:** a CTE is usually computed **once** and can be referenced many times in the outer query. An inline subquery is written where it is used (and conceptually re-evaluated there). Modern optimizers may **inline** CTEs anyway and rewrite them; you still write CTEs for readability.

## Window functions

Do **not** collapse rows (unlike GROUP BY). One output row per input row.

```sql
SELECT
  name,
  dept,
  salary,
  RANK() OVER (PARTITION BY dept ORDER BY salary DESC) AS dept_rank,
  AVG(salary) OVER (PARTITION BY dept) AS dept_avg,
  SUM(salary) OVER (
    ORDER BY hire_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS running_total,
  LAG(salary, 1) OVER (ORDER BY hire_date) AS prev_salary
FROM employees;
```

**Ranking:**
- `ROW_NUMBER()` — unique, no ties (1,2,3,4)
- `RANK()` — ties same rank, **gap** (1,2,2,4)
- `DENSE_RANK()` — ties same rank, **no gap** (1,2,2,3)
- `NTILE(n)` — n equal buckets

**Frames:**
- `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` — running total
- `ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING` — 5-row sliding window
- `RANGE BETWEEN INTERVAL '7' DAY PRECEDING AND CURRENT ROW` — 7-day window

**ROWS** = physical rows. **RANGE** = logical value ranges (ties differ).

**Must-know problems:**
1. **Second highest salary per dept:** `RANK() OVER (PARTITION BY dept ORDER BY salary DESC)` then `WHERE rank = 2`
2. **Running total:** `SUM(amount) OVER (ORDER BY date ROWS UNBOUNDED PRECEDING)`
3. **MoM growth:** `LAG(revenue, 1) OVER (ORDER BY month)` then % change
4. **Dedup keep latest per user:** `ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC)` then `WHERE rn = 1`

## EXISTS / NOT EXISTS — “is there at least one?”

`EXISTS` does not build a list. For each outer row it asks: **does even one matching inner row exist?** First hit → `TRUE`, stop. That is why `SELECT 1` is conventional — nobody uses the selected value.

```sql
-- users who placed at least one order
SELECT name FROM users u
WHERE EXISTS (
  SELECT 1 FROM orders o WHERE o.user_id = u.id
);

-- users who never ordered
SELECT name FROM users u
WHERE NOT EXISTS (
  SELECT 1 FROM orders o WHERE o.user_id = u.id
);
```

Picture: standing at a user, peek into `orders` until you see one row with that `user_id`. You do not fetch all their orders.

**`IN` vs `EXISTS`:** `IN` is “is my id in this list?” `EXISTS` is “can I find a matching row?” For a simple semi-join they often plan the same. The trap is **`NOT IN` + NULL**.

**NOT IN vs NOT EXISTS with NULLs (landmine):**

`NOT IN (list)` is FALSE/UNKNOWN for a row if **any** list element is NULL. Reason: `id = NULL` is not TRUE, it is UNKNOWN; `NOT UNKNOWN` is not TRUE; if the list contains a NULL, the whole `NOT IN` never returns TRUE.

```sql
-- if orders.user_id can be NULL:
SELECT name FROM users
WHERE id NOT IN (SELECT user_id FROM orders);
-- ANY NULL user_id in orders → this query returns ZERO rows  (NULL poison)

-- NOT EXISTS: a NULL user_id in orders does not match u.id, so it is ignored
SELECT name FROM users u
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);
```

**Always prefer NOT EXISTS over NOT IN when the subquery can return NULLs.**

## UNION / INTERSECT / EXCEPT

Same column count, compatible types. Output column names come from the **first** SELECT.

```sql
-- UNION: both lists, duplicates removed
SELECT city FROM customers
UNION
SELECT city FROM suppliers;

-- UNION ALL: both lists, keep duplicates (faster — no sort/dedup)
SELECT city FROM customers
UNION ALL
SELECT city FROM suppliers;

-- INTERSECT: cities that appear in BOTH
SELECT city FROM customers
INTERSECT
SELECT city FROM suppliers;

-- EXCEPT (Oracle MINUS): in the first list, not in the second
SELECT city FROM customers
EXCEPT
SELECT city FROM suppliers;
```

- **UNION** — combine, **dedup**
- **UNION ALL** — keep dupes, **faster**
- **INTERSECT** — in both
- **EXCEPT** — in first, not second

## NULL landmines

NULL is **unknown**, not 0, not `''`, not false. Poisons expressions.

| Expr | Result |
|---|---|
| `NULL = NULL` | NULL (not TRUE) |
| `NULL = 5` | NULL |
| `NULL + 5` | NULL |
| `NULL OR TRUE` | TRUE (TRUE wins) |
| `NULL AND FALSE` | FALSE (FALSE wins) |
| `NULL OR FALSE` | NULL |
| `NULL AND TRUE` | NULL |

Correct: `IS NULL` / `IS NOT NULL`. **Never** `WHERE col = NULL` (always 0 rows).

**Aggregates:**
- `COUNT(*)` — all rows including NULL
- `COUNT(col)` — non-NULL only
- `SUM`, `AVG`, `MAX`, `MIN` — **ignore NULLs**

**COALESCE(a,b,c)** — first non-NULL. `COALESCE(nickname, first_name, 'Anonymous')`.

**NULLIF(a,b)** — NULL if a=b. `total / NULLIF(count, 0)` avoids divide-by-zero (returns NULL instead of error).

## Query optimization basics

Optimizer picks cheapest plan. Help it:

1. No functions on indexed columns
2. Filter early — most selective WHERE first (narrow faster)
3. Avoid `SELECT *` — enables covering indexes
4. **Sargable** predicates (below)
5. Prefer JOINs over correlated subqueries on hot paths
6. `EXPLAIN` / `EXPLAIN ANALYZE` — actual vs estimated rows; big mismatch → stale stats

Look for: Seq Scan on large tables (usually want Index Scan), nested loops on large tables without indexes, high actual vs estimated row mismatch.

### Sargable — can the index *seek*?

**Sargable** = **S**earch **ARG**ument **ABLE**. A predicate is sargable if the engine can use it as a **start/stop key** in a B+ tree: jump to the first matching leaf, scan a range, stop.

That requires the **indexed column to be bare** on one side, compared to a constant (or to something that does not depend on that column).

**Sargable (index can seek):**

```sql
WHERE age > 25
WHERE email = 'a@x.com'
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01'
WHERE name LIKE 'kumar%'          -- trailing wildcard: leading bytes known
WHERE id BETWEEN 100 AND 200
```

**Not sargable (tree is sorted by `col`, not by `f(col)`):**

```sql
WHERE age + 1 > 26                -- expression on the column
WHERE YEAR(created_at) = 2024     -- function on the column
WHERE LOWER(email) = 'a@x.com'
WHERE name LIKE '%kumar'          -- leading wildcard: no known prefix
WHERE CAST(user_id AS text) = '123'
```

`age + 1 > 26` is *logically* `age > 25`, but the optimizer typically will not algebraically rearrange it. The tree is ordered by `age`, not by `age+1`, so it cannot binary-search. Rewrite it yourself so the column stands alone.

### Day 5 self-quiz
1. Recite execution order. WHERE vs HAVING. PG vs MySQL GROUP BY. One SQL example per join type.
2. Uncorrelated vs correlated. Rewrite “above dept average” as window *and* as JOIN+GROUP BY. What does CTE stand for?
3. Recursive CTE: anchor vs recursive step. ROW_NUMBER vs RANK vs DENSE_RANK. Four classic window problems.
4. EXISTS in one sentence. Why NOT EXISTS over NOT IN.
5. UNION vs UNION ALL vs INTERSECT vs EXCEPT (what each returns). NULL truth table. COUNT(*) vs COUNT(col).
6. What does sargable mean? Give two sargable and two non-sargable predicates. Six optimizer-help rules.

---

# DAY 6 — Deadlocks + WAL / ARIES

## Deadlock

Circular wait: each tx holds a lock the other needs; none can proceed.

```
T1 holds A, waits for B
T2 holds B, waits for A
```

## Coffman conditions (all four required)

Prevention = remove **at least one**.

1. **Mutual exclusion** — at least one resource held non-sharable (X-lock)
2. **Hold and wait** — holds one resource, waits for another
3. **No preemption** — lock cannot be forcibly taken; holder releases voluntarily
4. **Circular wait** — T1 waits T2 waits … Tn waits T1

## Detection — Wait-For Graph

Node = active tx. Edge **Ti → Tj** = Ti waits for a lock **Tj holds**. Cycle = deadlock.

Victim: usually **youngest** or **least work done** (minimize wasted work), abort it, others proceed.

PostgreSQL and MySQL InnoDB: **automatic** deadlock detection + victim selection.

## Prevention — timestamps (Wait-Die and Wound-Wait)

Give every transaction a **timestamp when it starts**. Smaller timestamp = **older** (started first). When Ti wants a lock that Tj already holds, compare ages and decide: **wait** or **abort**. That decision is designed so a **wait-for cycle can never form**.

Aborted transactions **restart with their original timestamp**. They do not get a fresh “young” time. So a tx that keeps dying still ages relative to newcomers and eventually becomes old enough to win. That prevents **starvation**.

Use one story for both protocols:

```
T1 started first  (old,  ts = 10)
T2 started later  (young, ts = 20)
```

### Wait-Die — “old waits, young dies”

Ti wants a resource Tj holds.

- If Ti is **older** than Tj → Ti **waits**.
- If Ti is **younger** than Tj → Ti **dies** (abort + restart later).

“Seniors are patient. Juniors give up rather than make a senior wait behind them.”

| Who wants the lock? | Who holds it? | Wait-Die does |
|---|---|---|
| T1 old | T2 young | T1 **waits** for T2 to finish |
| T2 young | T1 old | T2 **dies** and restarts later |

Why no deadlock: a young tx **never waits for an older one**. A cycle would need A waiting for B **and** B waiting for A. One of those two is younger and would have died instead of waiting.

### Wound-Wait — “old wounds, young waits”

Same setup, opposite personality.

- If Ti is **older** than Tj → Ti **wounds** Tj: force the young holder to abort, Ti takes the lock.
- If Ti is **younger** than Tj → Ti **waits**.

“Seniors are aggressive. Juniors stand aside.”

| Who wants the lock? | Who holds it? | Wound-Wait does |
|---|---|---|
| T1 old | T2 young | T1 **kills T2** and takes the lock |
| T2 young | T1 old | T2 **waits** for T1 |

Why no deadlock: an old tx **never waits for a younger one** (it preempts). Again, a cycle cannot close.

**Which is “nicer”?** Wait-Die only kills the **newcomer** (who has done less work). Wound-Wait may kill a young tx that already did a lot, because an old tx showed up. Both are correct; Wait-Die is the one people usually quote first.

(There is no protocol named “wound-die”. The pair is **Wait-Die** and **Wound-Wait**.)

## Prevention — lock ordering (app-level, simplest)

Always acquire locks in the **same global order**. Money transfer: always lock `min(id)` first, then `max(id)`. Cycle requires opposite acquisition order — this forbids it.

## Avoidance — Banker's Algorithm

Before granting, check system stays in a **safe state** (all txs can eventually complete). If not, wait.

Theoretically elegant. **Practically useless in DBs** — requires knowing **all** resources a tx will ever need **upfront**, which is impossible.

Production: **Detection + Kill** (PG, MySQL) or **Wait-Die / Wound-Wait**, not avoidance.

**Interview — prevent deadlocks in app code:** consistent global lock order (lower ID first). Keep txs **short**. **No I/O or user interaction inside a transaction.**

---

## WAL — Write-Ahead Logging (recovery), in order

### 0. Pieces of the machine (define these first)

The database is **not** one file that is always up to date. Three places matter:

| Name | Where | What it holds |
|---|---|---|
| **Data files** | Disk | The official pages of tables/indexes. Only updated when a page is **flushed**. |
| **Buffer pool** | RAM | Copies of hot pages. Almost all reads/writes happen here first. Fast, **volatile** — a crash wipes it. |
| **WAL / log** | Disk (separate sequential file) | A diary: “tx 7 changed page 42, field X, from 100 to 150.” Append-only. The **source of truth after a crash**. |

Other words:

- **Page** — unit of I/O (8KB PG / 16KB InnoDB). You never write “one integer” to disk; you write a whole page.
- **Dirty page** — the RAM copy was modified and is **newer** than the copy in the data file.
- **Flush** — write a dirty page from RAM to the data file.
- **LSN (Log Sequence Number)** — integer that only goes up. Each log record gets the next LSN. Each page stores “the LSN of the last log record that changed me.” That is how recovery knows whether a page is stale.
- **COMMIT** — the user is told “this is permanent.” After this, a crash must **not** lose the work.

### 1. What goes wrong without a log

You `UPDATE accounts SET balance = 150 WHERE id = 1` (was 100) and then `COMMIT`.

- The change almost certainly happened **only in RAM**.
- The data file may still say 100.
- Crash. RAM gone. Data file still 100. The user already got “committed.” Durability is broken.

The opposite failure: the buffer pool was full, so it **flushed** the page (150) to the data file **before** the transaction committed, then crashed. Data file says 150, but the tx never committed. Atomicity is broken — a partial tx leaked onto disk.

We need a diary that survives the crash, so we can **redo** committed work and **undo** uncommitted work.

### 2. The WAL rule (one sentence, then two sub-rules)

**Write the diary entry to disk *before* you write the matching data page to disk.** The log is always ahead of the data files. After a crash, the log is what you trust.

**Undo rule** (protects atomicity): before a **dirty page** is flushed to the data file, the log record that describes that change must already be on disk. Otherwise you would have 150 on disk with no note that it used to be 100 — you could not undo.

**Redo rule** (protects durability): before you tell the user **COMMIT succeeded**, **all** of that transaction’s log records (including the COMMIT record) must be on disk. The data pages do **not** need to be on disk yet. If we crash, we still have the recipe to replay.

Intuition: a surgeon writes the plan down *before* cutting. If something fails, the notes say what to reverse.

### 3. What a log record looks like

The log is a sequence of records, each with a unique LSN:

```
LSN | Txn | Type     | Page | Offset | Old (before) | New (after) | PrevLSN
----|-----|----------|------|--------|--------------|-------------|--------
 41 | 7   | BEGIN    |      |        |              |             |
 42 | 7   | UPDATE   |  88  |  20    | 100          | 150         | 41
 43 | 7   | COMMIT   |      |        |              |             | 42
```

- **Type:** BEGIN, UPDATE, COMMIT, ABORT, or **CLR** (Compensation Log Record — “I just undid something”; used so a crash *during recovery* does not undo twice).
- **Old / before-image:** value to put back on **undo**.
- **New / after-image:** value to put on the page on **redo**.
- **PrevLSN:** previous record of **this same transaction**. Follow this chain backwards to undo a tx.

ARIES keeps **one** log with both images, not two separate undo/redo logs.

### 4. Normal life of one UPDATE + COMMIT (the actual order)

1. `BEGIN` — append a BEGIN record to the log **in RAM** (log buffer).
2. `UPDATE` — change the row **in the buffer pool**. Page 88 is now **dirty**. Also append an UPDATE log record (old=100, new=150) to the log buffer. **Do not** wait to write page 88 to the data file.
3. If RAM is tight and page 88 must be flushed: **first** fsync the log through LSN 42, **then** write page 88 to the data file (undo rule). Often this never happens before commit.
4. `COMMIT` — append COMMIT, then **fsync the log** through that COMMIT (redo rule). **Then** return success to the client. Page 88 may still live only in RAM.
5. Later (checkpoint or eviction): flush page 88 to the data file. Harmless — the log already has the story.

Log writes are a **sequential append** (fast). Data-page writes are random (slow). WAL exists so COMMIT only waits on the fast sequential fsync, not on flushing every dirty table page.

### 5. After a crash — two jobs

- **Redo:** every tx whose COMMIT is in the log must end up reflected in the data files (even if those pages never flushed).
- **Undo:** every tx with no COMMIT/ABORT in the log must leave **no** trace on the data files (even if some of its dirty pages *did* flush).

### 6. ARIES — three phases (PostgreSQL, SQL Server, DB2, …)

Start from the **last checkpoint**, not from the beginning of time.

**Phase 1 — Analysis** (scan log **forward** from checkpoint)

Rebuild two lists by reading what happened since the checkpoint:

- **Transaction table:** which txs were still **active** (started, never committed/aborted).
- **Dirty page table:** which pages were dirty, and the **earliest LSN** that dirtied each one.

You now know *who* might need undo and *from where* redo must start.

**Phase 2 — Redo** (scan log **forward** from the **oldest dirty-page LSN**)

Replay **every** UPDATE, including those of txs that never committed. Goal: make the data files look **exactly like RAM looked at crash time** — including uncommitted dirt.

Why redo uncommitted work? Because some of those pages were flushed and some were not. Repeating *all* logged changes from a known point is the only way to get a consistent crash-time picture. Undo (next) will clean the uncommitted ones. Skip a page if its on-disk **page LSN** is already ≥ this log record (that flush already contains this change).

**Phase 3 — Undo** (walk **backwards** along each loser tx’s PrevLSN chain)

“Loser” = still active at crash (from the transaction table). For each of their UPDATE records, put the **old** value back. For every undo, write a **CLR** (“I reversed LSN 42”). If you crash *during* this phase, redo will see the CLR and will **not** undo the same change again.

When undo of a tx is done, log an ABORT. Losers are gone; winners’ work is on disk. Database may open for traffic.

### 7. Checkpoints — so you don’t replay the entire log

Without checkpoints, Analysis/Redo start at LSN 1. After weeks of traffic that is hours of recovery.

A **checkpoint** periodically:

1. Notes “recovery may start here.”
2. Flushes some/all dirty pages to the data file so redo has less to do.
3. Writes which txs are active and which pages are still dirty.

**Fuzzy checkpoint** (what real engines use — the database **keeps running**):

1. Write `BEGIN CHECKPOINT` to the log.
2. Flush dirty pages **gradually** in the background. New transactions continue.
3. Write `END CHECKPOINT` with the dirty-page table and active-tx list **as they were at step 1**.
4. Recovery starts from **`BEGIN CHECKPOINT`**, not from END (work during the fuzzy window is after BEGIN, so you must not skip it).

This **bounds** recovery time without freezing the database for a huge flush.

### 8. Two interview questions

**“COMMIT succeeded, crash before data pages flushed — is the data lost?”**  
No. The redo rule already put the log (including COMMIT) on disk. Analysis finds a committed tx; Redo replays the after-images. That **is** durability.

**“Undo log vs redo log?”**  
Undo = before-images (roll back losers). Redo = after-images (replay winners). ARIES stores **both in one WAL**.

### Day 6 self-quiz
1. Four Coffman conditions. How does lock ordering remove circular wait?
2. Wait-Die vs Wound-Wait. Same story (T1 old, T2 young), both directions. Why original timestamp?
3. Why is Banker's Algorithm unused in DBs? What do PG/MySQL do?
4. Name the three storage pieces. Recite undo rule vs redo rule. Recite the 5-step UPDATE+COMMIT order.
5. Recite log fields. What is a CLR? Why redo *uncommitted* changes in ARIES phase 2?
6. Fuzzy checkpoint: recover from BEGIN or END? Why?
7. Commit, crash, pages not flushed — is data lost? Why?

---

# DAY 7 — Query Processing + Storage + Full Recitation

## SQL-to-result pipeline

```
SQL Text
  → Parser (syntax, parse tree / AST)          -- SELCT fails here
  → Semantic Analyzer (names, types)           -- type mismatch may fail here (or implicit cast)
  → Logical plan (relational algebra)
  → Optimizer (rewrite + physical operators)
  → Physical plan (join algos, access methods)
  → Executor → Result
```

## Logical plan — relational algebra

| Symbol | Meaning |
|---|---|
| σ sigma | Selection — WHERE |
| π pi | Projection — SELECT columns |
| ⋈ | Join |
| γ | Aggregation — GROUP BY + aggs |
| δ | Dedup — DISTINCT |
| τ | Sort — ORDER BY |

**Rewrite rules:** push selections down (filter early, fewer rows into joins); push projections down (drop columns early); **reorder joins** (huge cost driver).

## Cost-based optimization

Estimates cost from **statistics:**
- Row count per table
- Distinct values / cardinality per column
- **Histograms** (e.g. 80% `status='active'`)
- Column correlation
- Index stats

**Cardinality estimation** cascades: bad estimate at step 1 poisons every downstream estimate.

Update stats: `ANALYZE users;` (PG), `ANALYZE TABLE users;` (MySQL).

**Join ordering:** N tables → **N!** orderings. 10 tables = 3,628,800. Optimizer uses **dynamic programming** to prune; good not always optimal. For **>8–10 tables**, switch to **greedy heuristics** (“always join smallest result first”).

### Example: a naive query, and what the optimizer does

You write this (logically correct, physically dumb if executed as written):

```sql
SELECT o.id, c.name
FROM orders o          -- 100 million rows
JOIN customers c       -- 1 million rows
  ON o.cust_id = c.id
JOIN nations n         -- 200 rows
  ON c.nation_id = n.id
WHERE n.name = 'France'
  AND YEAR(o.created_at) = 2024
  AND o.amount > 100;
```

**If executed left-to-right, no rewrites:**

1. Join 100M orders × 1M customers first → a huge intermediate.
2. Join that to nations.
3. Then filter France, year, amount. Most of the work was already wasted.

**What a cost-based optimizer actually does (logical rewrites + physical choices):**

1. **Push selections down.** `n.name = 'France'` is applied to `nations` *before* any join → ~1 row. `o.amount > 100` is applied while scanning/seeking `orders`, not after the join.
2. **Reorder joins.** Start with the tiny France row → join `customers` (maybe 10k French customers) → join `orders` for those customers. Intermediate sizes: 1 → 10k → a few million, not 100M × 1M.
3. **Pick physical operators.** Tiny France × customers: nested-loop or index NLJ on `customers.nation_id`. Then hash or index join into `orders`.
4. **It often cannot fix** `YEAR(o.created_at) = 2024`. That is not sargable, so `created_at`’s index may be ignored. **You** rewrite to `o.created_at >= '2024-01-01' AND o.created_at < '2025-01-01'` so the optimizer can seek.

The optimizer is not magic: it needs **statistics** (`ANALYZE`) to know that France is 1 nation and `amount > 100` is selective. Stale stats → it may still pick the giant join first.

## Join algorithms — in plain language

Joining is “match rows of R to rows of S on a key.” There are a few ways to find those matches. Think of **users ⋊ orders** on `user_id`.

### Nested Loop Join (NLJ) — “for each outer row, scan the inner”

Like two nested `for` loops:

```
for each user:
    for each order:
        if order.user_id == user.id: emit pair
```

Cost **O(|users| × |orders|)** — quadratic. Fine if one side is tiny (3 rows). Terrible if both are large (you re-read the whole inner table for every outer row).

**Index Nested Loop:** same idea, but the inner lookup uses an **index** instead of a full scan:

```
for each user:                          -- say 50 users from WHERE city='Goa'
    index-lookup orders where user_id = that user
```

Cost O(|outer| × cost of one index lookup). Best when **outer is small** and **inner has an index on the join key**. Classic **OLTP** (“this one user’s orders”).

### Hash Join — “build a dictionary, then probe it”

Only for **equality** joins (`ON a.id = b.a_id`), not for `<` / `BETWEEN`.

1. **Build:** take the **smaller** table, put it in a hash map keyed by the join column. `id → user row`.
2. **Probe:** scan the **larger** table. For each order, `hashmap.get(order.user_id)`. Hit → emit.

You read each table **once**. Cost **O(|R| + |S|)** — linear. This is the workhorse of **OLAP** (two big tables, equality, no useful index).

**Memory:** the hash table must fit in RAM. If it doesn’t, **Grace Hash Join**: hash-partition **both** tables into files on disk (bucket 0, bucket 1, …). Then hash-join bucket 0 of R with bucket 0 of S, etc. Still linear I/O but about a **3×** constant (write partitions, then read them).

Picture: you don’t search a phone book for every order. You dump all users into a keyed dictionary, then each order is one lookup.

### Sort-Merge Join — “sort both lists, walk two fingers”

1. Sort R by join key, sort S by join key (skip a sort if that side is already ordered — clustered index).
2. Walk two pointers through the sorted lists, like merging two sorted arrays. Equal keys → emit. Advance the smaller side.

Cost **O(|R| log |R| + |S| log |S|)** for the sorts; **O(|R| + |S|)** if already sorted.

Best when: data is **already sorted** on the join key; or the join is an **inequality / range** (`o.date BETWEEN w.start AND w.end`) that hash cannot do; or you needed the sort anyway (`ORDER BY` that key).

Picture: two attendance sheets already in roll-number order. Slide a finger down each; you never jump around.

### Selection / Projection

**Selection (WHERE):** if the column is indexed and few rows match → index scan. Otherwise sequential scan and test the predicate on each row.

**Projection (SELECT cols):** drop unused columns **during** the scan, not as a later extra pass.

| Algorithm | Use when | Plain English |
|---|---|---|
| Nested Loop | Tiny outer, inner maybe scanned | Nested for-loops |
| Index NLJ | Small outer + indexed inner, OLTP | For each outer, jump via index |
| Hash Join | Large tables, equality, no index, OLAP | Dictionary then probe |
| Sort-Merge | Pre-sorted data, or range/inequality | Two-finger merge |

## EXPLAIN — reading plans

Plans are **trees, executed bottom-up**. Leaves = table access. Inner nodes = ops.

```
HashAggregate  (cost=... rows=10000) (actual rows=9823 time=145ms)
  Hash Left Join  ...
    -> Seq Scan on orders
    -> Hash
         -> Index Scan on users using users_pkey
```

**Red flags:**
- `Seq Scan` on a large table when you expected index → stats, index existence, selectivity
- Estimated vs actual rows wildly different → stale stats, `ANALYZE`
- Nested loop over large tables **without** index
- `Sort` on a large dataset → index on sort column may help

**Slow query playbook (do not jump to “add an index”):**
1. `EXPLAIN ANALYZE`
2. Find expensive step (largest cost or actual time)
3. Check estimate vs actual (stale stats?)
4. Index on join/filter columns?
5. Rewrite: drop extra subqueries, push filters, reorder joins

---

## Pages — atomic unit of I/O

Smallest disk I/O unit. Typically **4KB**, **8KB (PostgreSQL default)**, **16KB (MySQL InnoDB default)**. Read one byte = read whole page. Disks are block devices; aligning to block size minimizes wasted I/O and enables prefetch.

**Layout:** header (page LSN, free-space ptr, slot count, checksum) → slot array growing **down** from top `[(offset, len), ...]` → free space in the middle → records packed **up** from the bottom. Variable-length records, slot array at predictable offsets.

**Page LSN** = most recent log record that modified this page — critical for WAL.

## Records

**Fixed-length:** field 3 always at byte 40. Fast, wasteful for VARCHAR.

**Variable-length:** null bitmap + fixed fields + pointers to variable fields + variable data.

**TOAST (PostgreSQL):** values **>~2KB** stored in a separate TOAST table; main row keeps a pointer. Transparent to SQL.

## Disk vs memory (why the whole architecture exists)

| Storage | Random latency | Sequential |
|---|---|---|
| DRAM | ~100 ns | Very high |
| NVMe SSD | ~70–100 μs | High |
| SATA SSD | ~100–200 μs | Medium |
| HDD 7200 RPM | ~5–10 ms | Low |

HDD random ≈ **50,000×** slower than DRAM. NVMe random ≈ **700×** slower than DRAM.

Architecture minimizes **disk I/Os**, especially **random** I/Os: B+ height, buffer pool, prefer sequential scans over many random fetches, use indexes carefully (too many random lookups worse than seq scan).

## Buffer pool

Fixed-size in-memory cache of disk pages. Most critical performance component.

Need page P → hash `page_id → frame` → **hit** use it / **miss** pick victim, write victim if dirty, read P from disk.

**Hit rate:** most important metric. Aim **>99% in OLTP**. 1% miss on busy OLTP can destroy performance.

## Replacement policies

- **LRU:** evict least recently accessed. Good temporal locality.
- **Clock (LRU approx):** reference bit; access sets 1; hand: if 1 set 0 and skip; if 0 evict. Cheaper than full LRU. **PostgreSQL uses clock.**
- **LRU-K:** timestamp of the **K-th** most recent access; evict oldest K-th. **MySQL InnoDB = LRU-2.**

**Why LRU-K:** sequential scans **kill plain LRU** — one full table scan evicts all hot pages for pages never reused. Longer history stops scan pollution.

## Buffer manager jobs

1. **Pin** pages in use (no eviction); unpin when done
2. **Dirty tracking** — flush dirty before eviction
3. **WAL coordination** — dirty page to disk **only after** its log record is on disk (undo rule)
4. **Prefetch / read-ahead** for sequential scans

## Row vs column vs hybrid

**Row (NSM — N-ary Storage Model):** all columns of a row contiguous.

- **OLTP:** point lookup `id=123` → one page, whole row; insert/update/delete in one place
- **OLAP bad:** `AVG(salary)` reads all 10 columns; 90% wasted I/O

**Column (DSM):** each column stored separately, values contiguous.

- **OLAP:** `AVG(salary)` reads **only** salary
- **Compression 5–10×:** similar values adjacent (RLE on `dept`); fewer I/Os
- **Vectorized execution:** 10k salaries as an array + **SIMD**
- **OLTP bad:** insert touches **every** column file; point lookup reassembles row from many files (write amplification)

**PAX (Partition Attributes Across):** pages = row groups (~10k rows); **within** group, columnar. **Parquet**, ORC, Iceberg. Row-group locality + column compression + vectorized access.

| Use case | Model | Systems |
|---|---|---|
| OLTP (lookups, row mutations) | Row | PostgreSQL, MySQL, SQL Server |
| OLAP (aggs, few cols, many rows) | Column | Redshift, BigQuery, Snowflake, ClickHouse |
| Mixed HTAP | Hybrid | TiDB, SingleStore, SAP HANA |
| Data-lake files | PAX | Parquet, ORC |

**Why column for analytics (3):** (1) read only needed columns — less I/O (2) better compression — similar values adjacent (3) vectorized SIMD + better CPU cache. **Why not for OLTP:** row mutate touches every column file.

### Day 7 self-quiz
1. Recite the SQL pipeline. Six algebra operators. Three rewrite rules.
2. Walk the France-orders example: what the naive plan does vs pushdown + join reorder. What the optimizer *cannot* fix.
3. NLJ vs Index NLJ vs Hash vs Grace vs Sort-Merge — in one sentence each, plus when/cost. Equality vs inequality.
4. Slow-query 5-step playbook. EXPLAIN red flags.
5. Page sizes PG vs InnoDB. Slot-array layout. TOAST threshold.
6. Latency table. Buffer hit-rate target. PG vs InnoDB eviction. Why LRU-K.
7. NSM vs DSM vs PAX. Three reasons column for OLAP.

---

# SUNDAY CLOSE — Full cheat sheet (recite out loud)

**Deadlock necessary conditions:** Mutual exclusion, hold-and-wait, no preemption, circular wait.

**2PL does NOT prevent deadlocks.** It prevents serializability violations. It holds locks longer → more deadlocks. Strict 2PL holds X locks until commit (no dirty reads / no cascading aborts). Rigorous holds all locks until commit (what PG/MySQL do).

**Dirty read → Read Committed prevents it.**
**Non-repeatable read → Repeatable Read prevents it.**
**Phantom read → Serializable prevents it.** (MySQL RR also via gap locks.)

**Defaults:** PG/Oracle/SQL Server = **Read Committed**. MySQL = **Repeatable Read**. PG Serializable = **SSI**. MySQL Serializable = **gap/range locks**. MVCC: RC = snapshot **per statement**; RR = snapshot **per transaction**. Readers don’t block writers.

**ACID C** is shared with the app. **CAP C** is different. A, I, D are DB-only. Durability = WAL.

**Conflict serializability:** precedence graph, no cycle. Conflicts = different tx, same item, ≥1 write (RW/WR/WW). View ⊃ conflict; view is NP-hard; engines use conflict. Blind writes are the gap.

**B+ vs B:** data only in leaves + linked leaves = range queries + higher fanout. Height ≈ log_B(N). Clustered: one per table, data **is** the index (InnoDB PK). Non-clustered: pointer (PG RID / InnoDB PK → second tree walk). Left-prefix: (A,B,C) → A, AB, ABC only. Covering: `Using index` / `Index Only Scan`. Optimizer ignore: selectivity, stale stats, function, type conversion. Crossover ~10–20%.

**2NF:** no partial dependency (composite PK). **3NF:** no transitive; X superkey **or** Y prime. **BCNF:** every determinant is a superkey. 3NF always lossless + dep-preserving; BCNF may not preserve FDs. Lossless iff intersection keys at least one side.

**SQL order:** FROM JOIN WHERE GROUP BY HAVING SELECT DISTINCT ORDER BY LIMIT. NOT EXISTS over NOT IN (NULL poison). COUNT(*) vs COUNT(col). `NULL=NULL` is NULL. Windows don’t collapse rows.

**WAL:** Log *before* data pages. Three pieces: data files, buffer pool, WAL. Undo rule = log on disk before dirty page flush. Redo rule = log on disk before COMMIT returns. UPDATE happens in RAM; COMMIT fsyncs the log, not the table. ARIES: Analysis (forward) → Redo all to crash state (even uncommitted) → Undo losers + CLRs. Fuzzy checkpoint: recover from **BEGIN CHECKPOINT**.

**Joins:** Hash = large, equality, OLAP. Sort-merge = pre-sorted or inequality. NLJ = small outer + indexed inner. Grace hash if hash table > memory (~3×).

**Column storage** = read less, compress 5–10×, vectorized SIMD. Row storage = OLTP full-row access. PAX = Parquet/ORC. Buffer pool >99% OLTP. PG Clock, InnoDB LRU-2. Pages: PG 8KB, InnoDB 16KB.

**App deadlocks:** lock min(id) first; short txs; no I/O inside txs. Detection = wait-for graph. Wait-Die / Wound-Wait keep original timestamp.

---

# Interview lines (say these verbatim)

1. **ACID C:** “Only ACID property that’s partially the application’s job. DB enforces declared constraints; business logic is on us. CAP C is a different C.”
2. **Four anomalies:** Lost update = two writers clobber. Dirty = uncommitted garbage. NRR = same row, different committed value. Phantom = same predicate, different row set.
3. **Isolation defaults + MVCC:** “PG Read Committed: new snapshot per statement. MySQL Repeatable Read: one snapshot at tx start. PG Serializable is SSI; MySQL uses gap locks. MVCC: readers never block writers.”
4. **2PL vs deadlock:** “2PL gives conflict serializability via the lock point. It does **not** prevent deadlocks — it makes them more likely because locks are held longer.”
5. **Serializability method:** List conflicts → draw precedence graph → cycle? → if not, topo-sort serial order. Never eyeball.
6. **Ignored index:** Selectivity / stale `ANALYZE` / function on column / implicit cast. Then maybe covering vs 10–20% crossover.
7. **InnoDB PK:** “Secondary indexes store the PK, so every secondary lookup is two B+ walks. Keep the PK small.”
8. **Why not always BCNF:** “May not preserve FDs, so the DB can’t enforce some constraints. Joins cost. OLAP often denormalizes on purpose.”
9. **NOT IN:** “If the subquery can return NULL, NOT IN can return no rows. Prefer NOT EXISTS.”
10. **Slow query:** EXPLAIN ANALYZE → expensive node → estimate vs actual → index existence → rewrite. Don’t lead with ‘add an index’.
11. **Commit then crash:** “Not lost. Redo rule put the log on disk. ARIES redo replays it. That’s durability.”
12. **Column vs row:** “Analytics: less I/O, compression, SIMD. OLTP: a row write would touch every column file.”
13. **App deadlocks:** “Global lock order, short transactions, no I/O or UI inside the transaction.”
14. **Wait-Die vs Wound-Wait:** Wait-Die = old waits, young dies. Wound-Wait = old kills the young holder, young waits. Restart with **original** timestamp so they don’t starve. There is no “wound-die.”

---

# Weekly 15-minute emergency pass (if you only have one sitting)

Recite in this order, eyes closed:

1. ACID (C is shared; CAP ≠ ACID) → COMMIT/ROLLBACK/SAVEPOINT → PG must rollback on error
2. Lost / Dirty / NRR / Phantom
3. Isolation table + defaults + SSI vs gap locks + MVCC snapshot timing
4. S/X matrix → 2PL growing/shrinking → strict/rigorous → 2PL ≠ no deadlock
5. Conflict def → precedence graph → view ⊃ conflict, NP-hard
6. B+ leaves + linked + log_B(N) → clustered/InnoDB two walks → left-prefix → covering → 4 ignore reasons
7. 1NF atomic → 2NF partial → 3NF transitive/prime exception → BCNF superkey → lossless test → 3NF preserves FDs
8. SQL order → windows ranks → NOT IN NULL → NULL table → sargable
9. Coffman → WFG → Wait-Die/Wound-Wait → lock min(id)
10. Data file / buffer pool / WAL → undo+redo rules → UPDATE in RAM, COMMIT fsyncs log → ARIES A/R/U → CLR → fuzzy BEGIN CHECKPOINT
11. NLJ / Hash / Grace / Sort-Merge → EXPLAIN red flags
12. Page 8 vs 16KB → buffer >99% → Clock vs LRU-2 → row/column/PAX
