# Production GenAI Interview — Weekly Revision Guide

Synthesized from `Production_GenAI_Systems_Guide.md`. You already read the long notes. Use this **every week** as a timed recitation pass. Every critical fact is here; examples are compressed, not dropped.

**How to use (one full pass per week, ~40 min/day):**

| Day | Cover | Recite until you can say it without looking |
|-----|--------|---------------------------------------------|
| **Mon** | RAG pipelines + retrieval | Two pipelines, chunk tradeoff, hybrid + rerank, 6 failures |
| **Tue** | Fine-tuning + LoRA | RAG vs FT diagnostic, PEFT, lifecycle, RLHF/DPO |
| **Wed** | Evals | Task/trial/grader, pass@k vs pass^k, capability vs regression |
| **Thu** | Observability | Evals vs obs, percentiles, version tags, 6-way quality drop |
| **Fri** | Caching | 4 layers, exact vs semantic, invalidation, scoped keys |
| **Sat** | Feedback + guardrails | Feedback ≠ truth, loop to evals, model proposes / app decides |
| **Sun** | Request path + maturity + cheat sheet | 7 capabilities, 0–4 maturity, full recitation |

Each day: **Recite** the boxed facts → **Scan** the section → **Answer** the self-quiz at the end of that day. Sunday: run the full cheat sheet out loud.

**Interview line to open any GenAI system talk:** “The model is a component, not a system. Accuracy, freshness, safety, cost, and improvement are built **around** it.”

---

# DAY 1 — RAG

## Why RAG exists

Three hard limits of every LLM:

1. **Knowledge cutoff** — does not know yesterday’s price or this morning’s policy
2. **No private data** — never saw your wiki, tickets, or account history
3. **Finite context** — you cannot paste the whole corpus; extra junk also **hurts** answers

RAG fetches the 3–4 passages that answer *this* question and stuffs them in just before generation. It turns a training problem into a **search problem** — more debuggable.

**RAG changes what the model *sees*. It does not touch weights.**

## Two pipelines (different clocks, different bugs)

**Indexing (offline, on document change):**

```
Documents → Parse → Chunk → Embed → Vector DB
```

A parse/chunk bug **silently poisons** the store. You only notice when someone asks that section.

**Query (online, every request):**

```
Query → Embed → Retrieve top-K → Rerank → Context → LLM → Answer
```

A retrieval/rerank bug shows up **immediately** on every affected query. Easier to catch.

## Parsing

PDFs, HTML, Word, Confluence, sheets, transcripts → clean structured text. **Preserve** headings, lists, tables as metadata. Flattening a table into a number soup poisons every downstream chunk. Most underestimated RAG failure source.

## Chunking — the real tradeoff

| Size | Win | Loss |
|---|---|---|
| Small (100–200 tok) | Precise; less noise | Splits an idea; loses context |
| Large (800–1500 tok) | Keeps nuance | Dilute embedding; more noise per hit |
| Overlap 10–20% | Idea not cut at the boundary | More storage, duplicate hits |

**Default that works:** chunk on **document structure** (section / paragraph), modest overlap, attach the **parent heading** as metadata so a small chunk still has a title.

## Embeddings + vector DB

Embedding = vector (hundreds–thousands of dims) such that **meaning-neighbors sit close**. “Refund for damaged goods” ≈ “order arrived broken” even with almost no shared words.

Vector DB: store vector + text + metadata. Query = “nearest neighbors.” Under the hood: **ANN** (HNSW, IVF) — a little recall for a lot of speed. Not brute force.

## Hybrid search

Pure vectors miss **exact strings**: SKUs, error codes, citations, weird acronyms. Hybrid = **semantic + BM25 (keyword)**, merge the two lists. `Error 0x8007045D` must not vanish because it embeds as mush.

## Reranking

Over-fetch top **20–50** with cheap ANN. Then a **cross-encoder** reads **query + candidate together**, scores true relevance, keep final **3–8**.

Bi-encoder = two independent vectors, fast. Cross-encoder = joint read, slower, much more accurate. That is why you two-stage.

## Metadata filters — *before* similarity

Scope by type, dept, geo, product, **ACL**, version. EU customer must not retrieve a US-only policy. Current price must not retrieve last year’s sheet.

Access control in the filter is a **security** concern, not a quality nicety.

## Six failure modes (diagnose stage by stage)

| Failure | Looks like | Fix |
|---|---|---|
| Missed document | Answer exists, never in top-K | Chunking, hybrid, raise K, fix filters |
| Wrong rank | Right doc retrieved, outranked | Rerank; better embedder |
| Stale | Old policy/price | Re-index on change; surface `last_updated` |
| Bad chunk | Cut mid-sentence; qualifier two sentences later | Structure + overlap |
| Too much context | Right passage buried; model dilutes | Fewer, better chunks; aggressive rerank |
| Hallucinate *with* context | Passage is there; model still invents | “Answer only from context”; citation check / output guardrail |

Never treat “wrong answer” as one blob. Walk the stages.

## Worked query — tent return

User: *“Can I return a tent I bought two months ago if I only used it once?”*

1. Parser kept the **return-window table**
2. Chunker split on headings → “Outdoor Equipment” = 90-day + like-new, heading attached
3. Hybrid: semantic loves “tent” (might pull a spec); keyword keeps “return”
4. Top-20 includes tent-care and warranty junk
5. Reranker promotes the 90-day policy, drops warranty registration
6. Metadata already scoped to the customer’s **region**
7. Top-3 + “answer only from context, state the day-count and condition”

Any one silent miss (split window from condition, no reranker, wrong-country filter) → wrong answer while every other stage “worked.”

## Advanced patterns

| Pattern | One sentence |
|---|---|
| **Query rewrite** | LLM *before* retrieval: expand abbreviations, resolve pronouns, paraphrase |
| **HyDE** | Draft a hypothetical answer, **embed that** — closer to document style than the raw question |
| **Multi-hop** | Retrieve, use result to form the next query (purchase date, then warranty term) |
| **Agentic RAG** | Model decides: enough? retrieve again? tool? ask user? More paths → evals + traces mandatory |
| **Graph RAG** | Entities + edges for hierarchies / deps / citations. Vector chunks are bad at structural multi-hop |

### Day 1 self-quiz
1. Three reasons LLMs need RAG. What does RAG change vs not change?
2. Draw both pipelines. Which bug is silent? Which is immediate?
3. Chunk size table + the structure+overlap+heading default.
4. Why hybrid? Why a cross-encoder after ANN? When do metadata filters run?
5. Recite the six failures and one fix each.
6. Walk the tent query. HyDE vs query rewrite in one line each.

---

# DAY 2 — Fine-Tuning

## What it is

Take a pretrained model. **Nudge weights** on a small curated task set. General capability stays; default behavior, tone, and format shift.

RAG = what information is available **now**. Fine-tune = how the model **behaves**, even with an empty prompt.

## When to fine-tune

- Classification / extraction onto a **fixed** label or schema
- Rigid output: JSON, house citation style, report template
- Brand voice / support persona / terse internal tone (else the system prompt gets huge)
- Domain jargon where the base model is *subtly* wrong
- One narrow high-volume job: a **smaller** FT model can beat a giant general model on cost + latency

## RAG vs fine-tuning

| | RAG | Fine-tune |
|---|---|---|
| What changes | Context at answer time | Weights / default behavior |
| Freshness | Update documents | Retrain |
| Private facts | Good | Expensive to keep current |
| Style / format | Weak (prompt only) | Strong |
| Classify / extract | Prompt every time | Usually cheaper long-term |
| Cost | Retrieval + longer prompts | Upfront train + versioning |
| Explain | Show the document | Buried in weights |

**Complementary.** Many systems: FT for structure/tone, RAG for facts.

**The diagnostic:** *“If I updated a document tomorrow, should the answer change immediately?”*  
**Yes → RAG. No (you want tone/structure/skill) → fine-tune.**

**Most common mistake:** fine-tuning because the model “doesn’t know” a price or policy. That is a freshness problem. Retrain-every-change is a treadmill.

## LoRA / PEFT

Full FT updates every parameter → huge GPU, a whole model copy per variant.

**LoRA:** freeze the base. Train tiny **adapter** matrices (often **<1%** of params).

- Cheap train, small files (MB not GB)
- Many adapters, **one** frozen base, swap at inference
- Default outside frontier labs for format / tone / narrow classify

Cannot always match full FT on **deep, broad** behavior shifts. For most product work, close enough.

## Lifecycle

```
Define target behavior
  → collect / curate examples
  → clean + dedup
  → train / val / held-out test
  → train
  → eval held-out + regression suite
  → deploy behind a version tag
  → monitor
```

**Data quality is the bottleneck**, not GPUs. A few hundred **real**, consistent, production-like examples beat tens of thousands of noisy synthetic ones.

Data sins: annotators disagree; examples easier than prod; silent dupes → overfit a handful of patterns.

## Preference tuning

**SFT** = input → one gold output.

**RLHF / DPO** = pairs: “A is better than B.” Used by labs for helpful/harmless. Same idea for tone when “correct” is a degree, not a string.

## Version the adapter like code

Tag: **training-data snapshot + base model + hyperparameters**. No tag → you cannot roll back a quality drop.

### Day 2 self-quiz
1. RAG vs FT in one sentence each. Recite the diagnostic question.
2. Five good FT use cases. The treadmill mistake.
3. LoRA: what is frozen, what is trained, why <1%, when it loses to full FT.
4. Recite the lifecycle. Why data > compute. Three data sins.
5. SFT vs DPO/RLHF. What three things must be on the version tag?

---

# DAY 3 — Evals

## Why not unit tests

1. Same prompt → different phrasing
2. Many valid answers (“summarize this contract”)
3. Multi-step agents: many places to go right or wrong

Evals define a **successful outcome** and a **grader** (code, model, or human). Not string equality.

## Vocabulary (say these exactly)

```
Task → Trial → Transcript → Outcome → Grader
```

| Term | Meaning |
|---|---|
| **Task** | Inputs + explicit success criteria |
| **Trial** | One run of that task |
| **Transcript** | The story: prompts, outputs, tools, retrieved docs |
| **Outcome** | Resulting **world state** (ticket created? sub cancelled?) |
| **Grader** | Code / model / human that judges the criteria |
| **Harness** | Runs task, captures transcript, applies grader |
| **Suite** | A set of tasks for one capability |

Transcript can say “I cancelled the subscription.” Outcome: still active. Grade the **outcome**.

## Three graders — complementary

| | Code | LLM-as-judge | Human |
|---|---|---|---|
| Use | Exact, regex, DB state, schema, API | Rubric, tone, faithfulness, pairwise | Calibrate the judge; hard subjective |
| Win | Fast, cheap, reproducible, CI | Nuance at scale | Gold standard |
| Loss | Cannot score a “good summary” | Inconsistent, length/confidence bias, needs its own eval | Expensive |

**Cheapest honest grader.** Code first. Model when subjective. Human to **spot-check the judge**, not every trial.

## Capability vs regression

| | Capability | Regression |
|---|---|---|
| Question | What *can* it do? | Did a used-to-work thing **break**? |
| Difficulty | Hard on purpose; headroom | Should already pass |
| Pass rate | Often << 100% | ≈ 100%; any drop is a signal |
| When | Deciding if a change is better | **Every deploy** |

All-hard suite: 2-point drop is noise. All-easy suite: stuck at 100%, no “are we better?”

## pass@k vs pass^k

Same system, opposite questions.

| | Meaning | Question |
|---|---|---|
| **pass@k** | ≥1 of *k* trials succeeds | Can it **eventually**? |
| **pass^k** | **All** *k* succeed | Is it **reliable**? |

Worked — 80% single-trial, k=5:

```
pass@5 = 1 − (0.2)^5  ≈ 99.68%
pass^5 = (0.8)^5      ≈ 32.77%
```

Recite the gap. Unattended agents need **pass^k**. “Eventually” is not production.

## A good task

**Clear, solvable, reproducible, representative, easy to grade.**

Start **20–50 realistic tasks mined from real failures**, not invented puzzles.

**Test both directions:** should do X **and** should **not** do X (no unauthorized refund; say “I don’t know” when retrieval is empty). Positive-only suites reward over-triggering.

**Grade outcomes, not trajectories.** Many valid tool paths. Don’t fail a better route.

## First suite — 7 steps

1. Mine prod transcripts, escalations, thumbs-down
2. Write success in **one sentence** before picking a grader
3. Cheapest honest grader
4. 20–50 tasks; should + should-not
5. Run on **current prod** → baseline
6. CI gate, same as unit tests
7. Prune tasks that no longer discriminate; add new failures

## Noise

A few-point swing on a few dozen tasks at temp>0 is often **sampling noise**. Run each task **multiple times and average**. Don’t ship a “2% win” that is weather.

### Day 3 self-quiz
1. Three reasons unit tests fail for LLMs. Transcript vs outcome.
2. Recite the Task→…→Grader chain. When each grader type wins.
3. Capability vs regression: pass-rate targets. Why you need both.
4. Compute pass@5 and pass^5 at p=0.8. Who needs pass^k?
5. Five properties of a good task. Why should-not cases.
6. Recite the 7-step first suite. Why a 2-point bump may be noise.

---

# DAY 4 — Observability

## Evals vs observability

| | Evals | Observability |
|---|---|---|
| Data | Curated tasks | **Live** traffic |
| When | Before / at deploy | Continuously |
| Question | *Did it work?* | *Why* this real request? |

## Trace the stages

```
Request → query processing → embed → retrieve → rerank → LLM → response
```

Capture **every** stage.

## What to log

**Performance:** p50 / p95 / **p99**, throughput.

**Cost:** in/out tokens, $ per request, cut by feature / customer / model version.

**Quality:** prod-sampled evals, user feedback, task-success rate.

**RAG:** retrieved doc IDs, similarity scores, **context size**.

**Also:** errors per stage; **model + prompt + index version**; every tool call + result.

## Percentiles, not averages

p50 = 800 ms, p99 = 9 s → **1 in 100** users is near-timeout. They churn. The average still looks fine.

Alert on the **tail**.

## Version tags are as important as metrics

Every trace: **model version, prompt version, retrieval/index version**. No tags → you see “worse,” you cannot name the change. Include **silent provider-side** model updates.

## Worked: quality 90% → 75% in a week

Check, in order, the **signature** of each:

1. Retrieval? Doc IDs + scores before vs after
2. Prompt? Version timeline
3. Model? Including provider silent updates
4. Stale data? `last_updated` vs the drop
5. Latency? p95/p99 → timeouts / truncated completions
6. Tool/API? Error rates

Without traces, all six look like “the bot got worse.”

## OpenTelemetry

Don’t invent a log format. OTel + GenAI span attrs (prompt id, tokens, retrieved IDs, model). Same APM as the DB and queues. Portable across LLM vendors.

## Alerting

- Trends vs a **rolling baseline**, not a forever-fixed number
- **Page** on down/errors. **Ticket** on quality drift — do not wake someone at 3am for a 3-point eval dip
- Alert p95/p99, not the mean
- Alert should name the next action: run regression suite, diff version tags

### Day 4 self-quiz
1. Evals vs observability in one sentence each.
2. Recite the 7-stage trace. Four signal families + RAG extras.
3. Why p99 over average. Give the 800 ms / 9 s line.
4. Three version tags. Why provider silent updates matter.
5. Recite the 6-way quality-drop checklist.
6. OTel why. Page vs ticket. Rolling baseline.

---

# DAY 5 — Caching

## Why

LLM calls are the expensive thing: **money per token**, hundreds of ms to seconds. A DB read is milliseconds. Reuse embeddings, retrievals, answers, tool results.

## Four layers

| Layer | Avoids | Hits when |
|---|---|---|
| **Embedding** | Re-embed unchanged docs/queries | Stable corpus; repeated queries |
| **Retrieval** | Re-run ANN | FAQ-ish repetition |
| **Response** | Re-call the LLM | Common / templated traffic |
| **Tool / API** | Re-hit a slow dependency | Data changes slower than it is asked |

Check **cheap → expensive**. A response miss can still hit a warm embed cache.

## Exact vs semantic

**Exact:** hash query + params. Safe if context unchanged. Hit rate = truly identical asks.

**Semantic:** embed the new question, reuse if close to a cached one. “What’s your return policy?” ≈ “Can I send this back?”

**False positive:** “return policy for **electronics**” vs “for **perishables**.” Loose threshold → confident **wrong** answer. Worse than a miss.

Track **semantic false-positive rate** as hard as hit rate.

## Invalidation is the real problem

Not “can we cache?” — **when is this entry a lie?**

| Strategy | When |
|---|---|
| **TTL** | Seconds for prices, days for a static FAQ |
| **Event-driven** | Doc/price/record update **purges** the key |
| **Version in the key** | Prompt/model/index bump must not serve old answers |
| **Scope in the key** | Permissions, locale, account tier |

**Scoped keys are security.** Cache a personalized answer, serve it to the next similar question from another user = **data leak**. Same ACL discipline as the source data.

## Request path

```
Request
  → response cache? HIT return
  → embed cache
  → retrieval cache
  → else full retrieve + rerank + LLM
  → write back each layer with its TTL
```

## Measure per layer

Hit rate · $ saved · latency saved · **staleness incidents** · semantic FP rate.

A cache you don’t measure you cannot tune. A stale cache is more expensive than the latency it saved.

### Day 5 self-quiz
1. Four cache layers and what each skips.
2. Exact vs semantic. Recite a false-positive pair.
3. Four invalidation strategies. Why keys must include ACL/locale/tier.
4. Draw the hit/miss path. Which metric is unique to semantic cache?
5. “Freshness is designed, not free” — say it in your own words.

---

# DAY 6 — Feedback + Guardrails

## Why offline evals are not enough

Evals test questions **you thought of**. Prod tests the ones you didn’t.

## Five signals

| Signal | Type |
|---|---|
| Thumbs / stars | Explicit |
| User **edits** the answer | Explicit |
| Immediate **retry** / rephrase | Implicit |
| Escalate to a human | Implicit |
| **Abandon** the workflow | Implicit |

## Feedback is not ground truth

A thumbs-down can mean: fact wrong · retrieval wrong · incomplete · **UX** · user asked out of scope.

“40 thumbs-down this week” is a **lead list**, not a KPI. Treating it as a KPI → you fine-tune when the real bug was a **missing document**.

Feedback is **sparse** (most users never click) and **biased to egregious** failures (people ignore mediocre). Low feedback volume ≠ healthy.

## Explicit vs implicit

| | Explicit | Implicit |
|---|---|---|
| Volume | Low | Every session |
| Clarity of intent | High (reason still fuzzy) | Low |
| Bias | Extremes of love/hate | Whatever you instrumented |

Track **both**. Lots of 👍 plus high silent abandon on one flow = two different truths.

## The loop (this is the job)

```
Feedback → failure analysis → root cause → fix
        → NEW REGRESSION EVAL → never again
```

Fixing the ticket helps one user. **The eval protects everyone later.** That conversion is the whole discipline.

## Capture that people actually use

One click, not a form. Optional reason **after**. Show that feedback changed something. Capture **in the moment**, not a Friday survey.

---

## Why guardrails

The model will try to comply — including malicious, unauthorized, or out-of-scope asks. Boundaries the model **cannot** be trusted to keep live **outside** it.

```
User → Input guardrails → LLM → Output guardrails → App
```

## Input (before the model)

Prompt-injection detect (user text **and retrieved** text) · PII redact · validate size/shape · **authn** · **authz**.

RAG chunks are user-influenceable. Inject in a doc.

## Output (before the user or any side effect)

Schema validate · PII / sensitive leak · policy / brand · toxicity (on top of the model’s own safety).

## The line you must say

**The model proposes. The application decides.**

₹50,000 refund proposal → hardcoded max auto-refund ₹10,000 → else **human**. The model cannot talk its way around a rule that is not in the prompt.

High-stakes / irreversible actions: deterministic policy + HITL. Never “the model said it was OK.”

## Defense in depth

No single check is perfect. Layer. One miss should still die on the next fence.

## Over-block vs under-block

| | Looks like | Cost |
|---|---|---|
| **Under** | Injection or PII gets through | Direct harm |
| **Over** | Real refund flagged “suspicious” | Users rage → team **turns the rail off** → now *less* safe |

Measure rails on **should-block and should-allow** sets. Same as evals.

### Day 6 self-quiz
1. Five feedback sources. Why a thumbs-down is ambiguous (5 reasons).
2. Sparse + severity-biased. Explicit vs implicit table.
3. Recite the feedback→eval loop. Why the last arrow is the job.
4. Input vs output rails. Why retrieved text is in scope.
5. “Model proposes, app decides” + the refund threshold story.
6. Over-block’s second-order failure. Should-block **and** should-allow.

---

# DAY 7 — How It Fits + Maturity

## Request path

```
USER
  → input guardrails
  → LLM  (maybe: retrieve? which tool?)
  → RAG + application data → assembled context
  → LLM synthesizes
  → output guardrails
  → USER
```

The LLM shows up **twice**: plan/tools, then write. Rails wrap **both** ends, not just the final paragraph.

Caching is **cross-cutting**, not a stage: embed cache inside retrieval, retrieval cache before the vector DB, response cache around the last LLM call. Bolt on incrementally.

## Improvement path

```
response → feedback ∥ observability
        → evals
        → failure analysis
        → RAG fix  and/or  FT fix  and/or  rail/app fix
        → better system
```

Often the bug is not the model.

## The production loop

```
BUILD → EVAL → DEPLOY → OBSERVE → FEEDBACK
     → ANALYZE → NEW EVALS → IMPROVE → BUILD
```

A demo stops at BUILD. Production is this loop. The model is often **not** the part that changes most.

**Failures → feedback → evals → improvements → fewer failures.** That chain *is* the discipline.

## Maturity 0–4

| Level | You have | You’re proving |
|---|---|---|
| **0 Prototype** | Prompt → model | It can do the thing once |
| **1 Grounded** | Basic RAG, smoke tests | Answers are current and private-data-aware |
| **2 Measured** | Capability + regression evals; latency/cost/errors | You know if a change helped |
| **3 Guarded** | I/O rails, business rules, caching | Safe and affordable at scale |
| **4 Self-improving** | Feedback→evals, full versioned traces, deliberate FT | Quality rises without heroics |

Not strictly sequential (legal may force rails at level 1). Use it as a **diagnostic**: if quality is mysterious, name the missing level before buying a bigger model.

### Day 7 self-quiz
1. Recite the request path. Where does the LLM appear twice?
2. Where the three cache layers sit. Why caching is not a stage.
3. Improvement path. Name a failure that is not a model bug.
4. Recite the 8-step production loop.
5. Maturity 0–4 in one line each. What do you do if the system “feels fragile”?

---

# SUNDAY CLOSE — Full cheat sheet (recite out loud)

**Model is a component.** Production is rails, retrieval, evals, traces, cache, feedback around a probabilistic box.

**RAG:** cutoff + private + finite context. Index = parse→chunk→embed→store (silent poison). Query = embed→K→rerank→LLM (fails in your face). Chunk on structure + 10–20% overlap + parent heading. Hybrid = vector + BM25. Rerank = cross-encoder, keep 3–8. Filters **before** ANN, including ACL. Six fails: miss, wrong rank, stale, bad chunk, too much context, hallucinate-with-context. HyDE embeds a fake answer. Multi-hop iterates. Agentic RAG needs traces.

**FT vs RAG:** document tomorrow → change now? yes RAG, no FT. LoRA freezes base, <1% adapters, swap many tasks. Data quality is the bottleneck. Version data+base+hparams. Don’t FT a price list.

**Evals:** task/trial/transcript/outcome/grader. Grade world state. Code / judge / human-calibrate. Capability hard, regression ≈100%, both. pass@k eventually, pass^k always; 80% → 99.7% vs 33% at k=5. 20–50 real failures, should + should-not. CI gate. Average multiple trials.

**Obs:** live *why*. p50/p95/p99 not mean. Tag model/prompt/index. 6-way drop: retrieval, prompt, model, stale, latency, tool. OTel. Page outages, ticket drift.

**Cache:** embed / retrieve / response / tool. Exact safe, semantic FP dangerous. Invalidate: TTL, event, version key, **scope key = security**. Measure hit, $, latency, staleness, FP.

**Feedback:** not labels. Sparse, severe-biased. Explicit + implicit. Every real fail → new regression eval.

**Rails:** input (injection incl. retrieved, PII, authz) / output (schema, leak, policy). Model proposes, app decides. HITL over a hardcoded threshold. Defense in depth. Measure over-block.

**Maturity:** 0 demo → 1 RAG → 2 evals → 3 rails+cache → 4 closed loop.

---

# Interview lines (say these verbatim)

1. **Component:** “The model is a component, not a system. Freshness, safety, cost, and improvement are built around it.”
2. **RAG vs FT:** “If a document change tomorrow should change the answer tomorrow, that is RAG. Fine-tune is tone, format, and stable skill.”
3. **Two pipelines:** “Index bugs poison silently. Query bugs fail on every request. I debug RAG stage by stage, not as ‘the answer was wrong.’”
4. **Hybrid + rerank:** “Vectors miss SKUs and error codes — that’s BM25. ANN over-fetches; a cross-encoder jointly scores query and doc and we keep a handful.”
5. **LoRA:** “Freeze the base, train adapters under 1% of weights, swap them per task. Full FT only if I need a deep behavior shift.”
6. **Outcome vs transcript:** “The log can say the subscription was cancelled. The outcome is whether it actually is. I grade the world.”
7. **pass@k:** “pass@5 at 80% looks like 99.7%. pass^5 is 33%. Unattended workflows need the second number.”
8. **Capability vs regression:** “Hard tasks tell me if we’re better. Easy tasks that must stay at 100% tell me if we broke something. I ship both.”
9. **p99:** “Averages hide the user who is about to churn. I alert on p95/p99 and I tag every trace with model, prompt, and index version.”
10. **Cache security:** “The hard question is invalidation. Keys include ACL, locale, and version. A personalized cache hit for the wrong user is a leak.”
11. **Feedback loop:** “A thumbs-down is a lead, not a label. The job is: root-cause it, fix it, and add a regression eval so it cannot silently return.”
12. **Guardrails:** “The model proposes. The application decides. A ₹50k refund suggestion dies on a ₹10k deterministic cap, then a human — not on the model’s confidence.”

---

# Weekly 15-minute emergency pass (if you only have one sitting)

Recite in this order, eyes closed:

1. Model = component → 7 capabilities in one line each
2. Two RAG pipelines → chunk default → hybrid + cross-encoder → filters first → 6 failures
3. Tent-query walk → HyDE / multi-hop / agentic
4. FT diagnostic question → LoRA → data bottleneck → version tag
5. Task/trial/transcript/outcome/grader → code vs judge vs human
6. Capability vs regression → pass@5 vs pass^5 at 80%
7. Evals vs obs → p99 → three version tags → 6-way quality drop
8. Four cache layers → exact vs semantic FP → TTL/event/version/scope
9. Feedback ≠ truth → explicit+implicit → fail becomes eval
10. Input/output rails → retrieved injection → model proposes, app decides → over-block
11. Request path (LLM twice) → production loop → maturity 0–4
