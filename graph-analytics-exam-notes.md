# Graph Analytics for Big Data — Exam Notes

These notes cover the course PDFs in this folder: what analytics is, what a graph is, why graphs are used, the V’s of big data, and the four analytic families from the module (paths, connectivity, communities, centrality).

The module’s goal is not only to compute a number. It is to **pick the right family of techniques** for a question. Use the decision table first, then the section that matches.

## Which technique answers which question

| If the question is about… | Use | Typical output |
| --- | --- | --- |
| The best route from A to B, with or without stops and bans | Path analytics (Dijkstra and its variants) | A path and its total weight |
| Whether the network falls apart when a few nodes or edges are removed | Connectivity, separating sets, algebraic connectivity, weighted spectral distribution | A robustness score |
| Whether two graphs have a similar shape | Degree histograms and a distance between them | A small distance means “similar” |
| Who hangs out with whom, and how tight the group is | Community analytics (density, cliques, k-cores, modularity, Louvain) | A cluster, plus a quality score |
| Who the important nodes are | Centrality. The *kind* of importance decides which measure | A score per node |
| Whom to remove so that a disease or message stops spreading | Key Player Problem 1, fragmentation | A small set whose removal breaks the graph |
| Whom to recruit so that a message reaches as many people as possible | Key Player Problem 2, distance-weighted reach | A small set close to everyone else |
| How a group’s membership changes over time | Autocorrelation and stationarity | A churn score |

---

## 1. What analytics is

**Analytics** is the discovery and communication of meaningful patterns, using four ingredients together:

1. **Mathematical properties** of the data.
2. **Computing**, to access and manipulate the data.
3. **Domain knowledge**, so the result can be interpreted.
4. **Statistical models**, to infer or predict.

Four broad purposes show up on exams:

- Describe a data set from its mathematical properties.
- Answer a specific question that needs more than one data set.
- Build a model that predicts how some variables behave.
- Detect something that emerges, and explain what produced it.

**Graph analytics** is analytics whose data is either stored as a graph already, or can faithfully be modeled as one.

A chart of sales over time is not graph analytics. A graph of who replied to whom is.

---

## 2. What a graph is — and what it is not

### A chart is the graph of a function

A bar chart or a time-series plot is a **function**: each input (a product category, a date) maps to one output (a count, a sales total). That is “the graph of a function” in school mathematics. This course does not mean that.

Graph analytics is based on **graph theory**.

### Where the idea came from

In 1736 Leonhard Euler studied Königsberg (now Kaliningrad). Seven bridges crossed the Pregel river. The question was whether a walk could cross **each bridge exactly once**. Euler solved it by ignoring the shapes of the islands and keeping only **places** and **links**. That abstraction is a graph.

### Two definitions you must be able to write

**Mathematical definition.** A graph is a pair \(G = (V, E)\).

- \(V\) is the set of **vertices** (also called nodes).
- \(E\) is the set of **edges**.
- Each edge names a pair of vertices. The course writes both styles: \(E = \{e_1, e_2, e_3, e_4\}\) and \(E = \{(v_1, v_5), (v_1, v_3), (v_6, v_3), (v_4, v_5)\}\).

**Computer-science definition.** A graph is an **abstract data type**:

1. A data structure that stores the mathematical graph.
2. Operations on it. The ones named in the course are `Add_edge`, `Add_vertex`, and `Get_neighbor`.

Mathematicians (Euler) care about the sets. Computer scientists (the course cites Dijkstra) care about the representation and the operations.

### Two representations

**Adjacency matrix.** Rows and columns are vertices. A 1 in row \(i\), column \(j\) means an edge from \(i\) to \(j\).

- `Add_edge(v3, v2)` writes a 1 in that cell.
- `Get_neighbor(v3)` scans **row v3 and column v3**. Every cell that holds a 1 is a neighbor. The row finds edges that leave v3. The column finds edges that enter v3.

**Pointer structure (used inside some graph databases).** Separate objects for nodes, edges, and connectors, with “next edge” and “next vertex” links. You only need to know that this exists and why: matrices waste space when almost all entries are 0, which is the usual case for big graphs.

---

## 3. The extended graph: types, attributes, weights

A bare pair \((V, E)\) is not enough for real data. A tweet graph has users, tweets, URLs, and media, and those are not interchangeable.

The **extended graph model** has:

| Piece | Meaning |
| --- | --- |
| \(V\), \(E\) | Vertices and edges |
| \(T_N\), and \(f\) assigning a type to each vertex | Node labels, such as Tweet, User, URL, Media |
| \(T_E\), and \(g\) assigning a type to each edge | Edge labels, such as follows, retweets, mentions |
| \(A_N\), \(A_E\) | Attributes on nodes and on edges |
| \(\mathrm{dom}(A_N[i])\), \(\mathrm{dom}(A_E[i])\) | The allowed values of the \(i\)-th attribute |

**Node schema** = the attributes and their values. A tweet node might have `text = "We've just posted a sneak preview…"`.

**Edge schema** = attributes on edges. A biological edge might have `interactionType`, whose domain is {physical, genetic, biochemical, …}.

### Why edges have weights

A **weight** is an edge attribute with a numeric meaning. The same mathematics then serves many domains:

| Domain | What the weight means |
| --- | --- |
| Road network | Distance, or travel time |
| Personal network | Strength of the relationship |
| Biological network | Likelihood that the interaction is real |
| Knowledge network | Certainty of the information |

“Shortest path” always means **least total weight**, not necessarily fewest edges. The course repeats this because the two answers differ.

### Loop and multigraph

- A **loop** is an edge whose two ends are the same vertex.
- A **multigraph** has more than one edge between the same pair of vertices. The edges are not duplicates. Each one carries different information (two different interaction types, two different roads).

---

## 4. Why these problems become graphs

Anything whose meaning lives in **relationships** is a graph. Four case studies are used throughout the course. The Alzheimer’s–colorectal cancer example comes back later as a path question, so learn it once.

### Example 1 — Social media

A tweet is already a small graph.

**Nodes:** user, tweet, URL, hashtag, media.

**Edges:** user creates tweet, tweet replies to tweet, tweet retweets tweet, user mentions user, tweet contains hashtag, user follows user.

A Facebook fragment in the slides has the same idea: a user, a post, its text, its media, a comment, a response to that comment, a location, and a tagged person. Edge labels on that fragment include **Contains**, **Tagged_in**, and **Refers_to**.

Questions a behavioral psychologist actually asks:

- Do players of violent games talk in a more confrontational way?
- Can we tell how addicted a user is to a game?

Graph jobs that fall out of those questions: pull out **conversation threads**, find **groups that interact**, find **influencers** inside a community.

### Example 2 — Biology

Nodes are genes, mRNA, proteins, phenotypes, cells, anatomical parts, diseases.

Edges are interactions and associations. They come from three places:

- Experiments: gene–protein links, gene–gene interactions, cell–cell signaling, gene–phenotype–disease links.
- Human knowledge: anatomy, the taxonomy of the animal kingdom.
- Computation: bioinformatics algorithms and literature mining.

**Data integration makes the graph bigger.** More sources, more volume. The payoff is the ability to **connect dots that no single data set contains**, including an indirect association between diseases such as colorectal cancer and Alzheimer’s disease. That is exploratory analysis: you do not yet know the path, you ask whether one exists.

### Example 3 — Human information networks

One person-graph built by combining public professional data (LinkedIn), public personal data (Facebook, Google+), calendars, contacts, and co-activities. Records are organized by **time and events**.

Further layers: financial and business transactions, performance, fitness habits, location.

Questions:

- **Matchmaking:** which candidate fits which job?
- **Topical influence:** who has the greatest reach for task X?
- **Situation assessment and threat detection.**

### Example 4 — Smart cities

Several networks share the same geography: transport (many modes), water and sewage, power, broadband, and machine-to-machine networks.

Each network has two layers you must not mix up:

- The **physical infrastructure**.
- The **commodity that flows** on it: material, energy, or people.

Planners build an analytical traffic model and a congestion model. Graph questions: where to put smart hubs, what the congestion pattern will be, how demand response should work, and which route is energy-optimal.

---

## 5. Graphs and the four V’s

The course uses four V’s. **Valence** is the one that is special to graphs. Do not replace it with “veracity” unless a question asks for the generic big-data list.

| V | Plain meaning | What it does to graph analytics |
| --- | --- | --- |
| **Volume** | Size: how many nodes and edges | Memory runs out. Algorithms get expensive. Time from data to answer becomes too long. |
| **Velocity** | How fast the graph grows in size **and** complexity | Updates arrive as a **stream of edges**. The stream does not fit in memory, so a metric has to be maintained with limited memory. |
| **Variety** | Different types and sources; the data is non-uniform | Graphs are built by integration, and different kinds of graphs mean different things. |
| **Valence** | How connected and interdependent the elements are | High valence is exactly what analytics exploits. Valence tends to **rise over time**. |

### Volume, worked as an exam story

Question: is there a simple path from “Alzheimer’s disease” to “colorectal cancer”? The course calls this a hard decision problem on the integrated biological graph. The follow-up is worse: **how many** simple paths are there? The number of simple paths can grow **exponentially with the number of nodes**, so you cannot list them on a big graph.

Keep one distinction clear, because both wordings appear:

- “Can I get from A to B at all?” is **reachability**. A search answers it without listing every route.
- “List or count every simple path” blows up with volume. A **simple path** repeats no vertex.

### Velocity

Social media is the running example: the stream of updates **is** a stream of edges. Metrics the course names: shortest distance between two nodes, and the count of strongly connected groups (a Facebook group is the intuition). The exam point is the constraint, not a specific streaming formula: a never-ending stream does not fit in memory.

### Variety has two parts

**Part 1 — graphs are usually integrated.** Sources named in the slides, in the order they are introduced: relational tables, XML/JSON, documents, and data that is already a graph.

**Part 2 — the same drawing can mean different things.** Kinds named: social networks, citation networks, interaction networks, the Semantic Web / Linked Data, and ontologies. An edge in a citation network (“cites”) is not an edge in a social network (“friends”). Analytics that ignores the meaning will be formally correct and scientifically useless. That is why domain knowledge is one of the four ingredients of analytics.

### Valence

Borrowed from chemistry, where valence is about how an atom bonds. In this course:

> Valence is the degree of connectedness or interdependence. Higher valence means the data elements are strongly related. Analytics exploits that relatedness.

Over time, valence usually goes **up**:

- Pieces of the graph get **denser**.
- The **average distance** between pairs of nodes goes **down**.

---

## 6. Path analytics

### 6.1 Walks and the restrictions on them

A **walk** is an alternating sequence of vertices and edges. You may repeat both.

Restrictions, from weakest to strongest:

| Name | What may not repeat | Course detail |
| --- | --- | --- |
| Walk | Nothing | The general object |
| **Trail** | Edges | Vertices may repeat. The walk \(H \to F \to G \to C \to F \to E \to C \to F \to J\) is **not** a trail, because it reuses edges at F. |
| **Path** | Vertices | A walk with no repeated vertex, except that the first and last may be the same. |
| **Cycle** | — | A path of length \(n \ge 3\) whose start and end are the same vertex. |
| Acyclic graph | — | A graph with no cycles. |

Length is the number of edges in the sequence, unless a question says “weight”, in which case length is the sum of edge weights.

**Reachability.** Node \(u\) is reachable from node \(v\) when a directed path runs from \(v\) to \(u\). In the course graph, **A is reachable from I**, and **I is not reachable from A**. Direction matters. “From” names the start of the path.

### 6.2 Distance and diameter

The **shortest-hop distance** from \(u\) to \(v\) is the smallest number of edges in a directed path from \(u\) to \(v\). It ignores weights. No path means the distance is \(\infty\).

The course builds the full **shortest-hop distance matrix** (rows = from, columns = to). The diagonal is 0.

**Diameter** is the maximum pairwise distance. In the worked matrix the largest **finite** entry is **4**, the distance from I to A, and the slide marks 4 as the largest value. Cells that are \(\infty\) mean that pair has no directed path.

If a question shows you that matrix:

- Diameter they want you to read off: **4**.
- The graph is **not strongly connected**, because some entries are infinite. Nothing reaches H, and A reaches almost nobody.

### 6.3 The basic path question

> What is the best path from node 1 to node 2?

“Best” has four parts. Exams ask you to list them.

1. An **objective**: the function to optimize (least weight, least time).
2. Nodes or edges you **must** traverse (a waypoint).
3. Nodes or edges you must **avoid**.
4. Other **preferences**.

Google Maps is the intuition: the shortest route changes with weather, traffic, and road condition, because those change the weights. The graph does not have to change shape for the best path to change.

The course’s concrete problem: least-weight path from **I to B**, **avoiding E**, and **forced to pass through J**.

They simplify in two steps before running an algorithm:

- Drop the banned vertex and work on the **relevant subgraph**.
- **Split** a waypoint constraint into two ordinary shortest paths: I to J, then J to B.

### 6.4 Dijkstra’s algorithm

Dijkstra finds a least-weight path from one source to the other nodes. The course points at it as the standard method and walks it from **I** toward **B**.

**State kept for every vertex \(v\):**

- \(d[v]\): best distance from the source found so far. \(d[\text{source}] = 0\), everyone else starts at \(\infty\).
- \(\mathrm{pred}[v]\): the vertex you came from to achieve \(d[v]\).
- \(\mathrm{done}[v]\): whether \(v\) is finished.

**One step:**

1. Among vertices that are not done, pick the one with the smallest \(d\). Mark it done. Call it \(u\).
2. For each edge \(u \to w\) of weight \(c\): if \(d[u] + c < d[w]\), set \(d[w] = d[u] + c\) and \(\mathrm{pred}[w] = u\). This is **relaxation**.

**What happens on the course graph, source I:**

| Step | Vertex finished | Updates worth remembering |
| --- | --- | --- |
| 1 | I, distance 0 | \(d[F] = 5\) via I. \(d[J] = 15\) via I. |
| 2 | F, distance 5 | \(d[J]\) improves to \(5 + 5 = 10\) via F. \(d[G] = 15\). \(d[E] = 25\). |
| 3 | J, distance 10 | Nothing useful improves. |
| 4 | G, distance 15 | \(d[D] = 25\), \(d[C] = 35\). E is already 25 via F, so G does not beat that. |
| 5 | E, distance 25 | \(d[B] = 30\) via E. \(d[C]\) improves to 30 via E. **B is reached.** |

**Reconstruct the path by walking predecessors backward from the target.**

\[
B \leftarrow E \leftarrow F \leftarrow I
\]

so the path is \(I \to F \to E \to B\), total weight \(5 + 20 + 5 = 30\).

**H stays at \(\infty\).** H has edges leaving it (toward E and I) and none that a search from I can arrive on. The slide calls H a root that is unreachable from I. Dijkstra does not invent a path that is not there.

**Cost.** Worst-case cost is proportional to

\[
(\text{number of edges}) \times \log(\text{number of nodes}).
\]

For 1 million nodes and 10 million edges the course says this is on the order of 14 million operations and already far too high. That is the motivation for the two variants below. Plain Dijkstra also assumes **weights are not negative**. A negative modified weight is called out later as illegal.

### 6.5 Bidirectional Dijkstra

Search from **both ends**.

- Forward frontier: ordinary Dijkstra from the source.
- Reverse frontier: Dijkstra from the target on the graph with every edge flipped.

Alternate. Expand one frontier, then the other. Stop when a frontier touches a vertex the other frontier has already reached, and **check that the sum of the two weighted distances is the minimum**. Meeting is not enough by itself. You must confirm no cheaper combination exists.

The course’s walk from I toward A:

- Forward: \(I \to F\) has length 5, then \(I \to F \to G\) has length 15. J is ignored on that expansion.
- Reverse: \(A \leftarrow D\) has length 5, then \(A \leftarrow D \leftarrow G\) has length 15.
- The frontiers meet at G. Both sides contribute 15.

**Least weight, not least hop.** A 2-edge path such as \(F \to C \to A\) can weigh **more** than a 3-edge path such as \(F \to G \to D \to A\). Count the sum of weights from both frontiers. Never stop just because a path has fewer edges.

### 6.6 Goal-directed Dijkstra

Bidirectional search is still blind. Goal-directed search **makes edges that point toward the target look cheaper**, so the search stops exploring the wrong side of the graph.

Use a **potential** \(\lambda(u)\), a guess of how close \(u\) is to the target. Replace every weight by

\[
w'(u, v) = w(u, v) - \lambda(u) + \lambda(v).
\]

\(w'\) **must not be negative**, or Dijkstra’s guarantee dies.

**How to choose \(\lambda\).** On a geometric graph such as a road network, use distances to **landmark** nodes. If the target B is itself a landmark and the known distances are \(\mathrm{Dist}(B, G) = 80\), \(\mathrm{Dist}(B, F) = 20\), \(\mathrm{Dist}(B, E) = 15\), then

\[
\begin{align*}
w'(F \to G) &= 10 - 20 + 80 = 70, \\
w'(F \to E) &= 20 - 20 + 15 = 15.
\end{align*}
\]

F→E now looks much cheaper than F→G, so the search prefers the edge that actually heads toward B. That is the whole point of the example: the unmodified algorithm walked F→G and would have finished sooner had it been willing to try F→E.

### 6.7 Path-analytics comparison

| Method | Idea | When it earns its place | Watch out for |
| --- | --- | --- | --- |
| Dijkstra | Grow the finished set from the source | One source, non-negative weights, graph fits in memory | Cost about \(\|E\| \log \|V\|\). Unreachable nodes stay at \(\infty\). |
| Bidirectional | Grow from source and target | Source and target are both known | Optimize **weight**, not hop count. Confirm the meeting node is the cheapest. |
| Goal-directed | Reweight edges with a potential | You can estimate distance to the target (maps, landmarks) | \(w'\) must stay non-negative. |
| With constraints | Delete banned vertices; split on waypoints | “Avoid E, pass through J” | Do this **before** searching, on the relevant subgraph. |

---

## 7. Connectivity analytics

The questions in this family:

- How robust is the graph? How many removals does it take to break it?
- How similar is the structure of \(G_1\) to the structure of \(G_2\)?
- Which numbers describe a graph’s shape well enough to compare graphs?

Finding connected components at **big-data scale** is postponed to the next module. The definitions are in this one, and those are what get examined.

### 7.1 Connected, strongly, weakly

The course’s baseline sentence:

> A graph is **connected** when, for every pair of vertices \((u, v)\), there is a directed path from \(u\) to \(v\) **or** a directed path from \(v\) to \(u\).

A picture of several separate blobs is labeled **not connected**. Each blob is a **component**.

Then the distinction that the quiz actually grades:

| Term | Definition |
| --- | --- |
| **Strongly connected** | There is a directed path from every vertex to every other vertex. Both directions. |
| **Weakly connected** | The graph becomes connected after you **forget directions** and treat every edge as undirected. |

**Quiz answer: the running example (nodes A–J) is weakly connected, not strongly.** You can walk between any pair if arrows do not matter. You cannot walk from every node to every other node along the arrows. H is a source nobody can enter. A is a sink that reaches nobody else.

### 7.2 How a connected graph is broken

Two parallel definitions. Do not mix “nodes” and “edges” in the answer.

| | Vertices | Edges |
| --- | --- | --- |
| The set you remove | **Separating set** \(S \subset V\). After deleting \(S\), more than one component remains. | **Disconnecting set** \(H \subset E\). After deleting \(H\), more than one component remains. |
| The number | **Connectivity** \(k(G)\): size of the smallest separating set. | **Edge-connectivity**: size of the smallest disconnecting set. |

A graph with connectivity 1 has a **cut vertex**: one node whose removal splits it. A graph with edge-connectivity 1 has a **bridge**.

### 7.3 Robustness

**Robustness:** if \(v\) was reachable from \(u\), it should stay reachable after an **attack**. An attack is a removed node or a removed edge.

On the course graph, removing **F** is the interesting attack, because F sits on many routes: C→G, C→J, C→E, J→E, I→E all break. The slide then points at **C** as the next target. A network that falls apart after one or two well-chosen deletions is not robust.

Two ways the course measures this, and they answer different questions:

| Measure | Question it answers |
| --- | --- |
| **Algebraic connectivity** | How well connected is the network, as a single number? |
| **Weighted spectral distribution (WSD)** | How much did the structure change when we attacked it? |

### 7.4 The normalized Laplacian

Build a symmetric matrix \(L\), one row and column per vertex.

\[
L(u, v) =
\begin{cases}
1 & \text{if } u = v \text{ and } d(v) \ne 0 \\
-\dfrac{1}{\sqrt{d(u)\, d(v)}} & \text{if an edge }(u, v)\text{ exists} \\
0 & \text{otherwise}
\end{cases}
\]

\(d(u)\) is the degree of \(u\). Diagonal entries are 1 for every non-isolated vertex. An edge contributes a negative off-diagonal entry that is larger in magnitude when both ends have small degree.

You are told to treat the next step as a black box that software computes: solve

\[
Lv = \lambda v
\]

for the **eigenvalues** \(\lambda\). There is one eigenvalue per vertex.

**Facts that get asked:**

- The number of eigenvalues equal to **0** equals the number of **components**. One 0 means the graph is connected.
- **Algebraic connectivity** is the **second-smallest** eigenvalue. In the 7-node example it is **0.4110**, not the 0.
- Larger algebraic connectivity means the graph is harder to pull apart.

The example graph has a hub (node 3, degree 5) linked to nodes 1, 2, 4, 5, and 6, plus a triangle among 3, 4, 5 and a triangle among 4, 5, 7. The slide counts **two 3-cycles and one 4-cycle**. Eigenvalues reported:

| | \(\lambda\) | \(1 - \lambda\) | \((1 - \lambda)^3\) |
| --- | --- | --- | --- |
| 1 | 1.8615 | −0.8615 | −0.6394 |
| 2 | 1.3942 | −0.3942 | −0.0612 |
| 3 | 1.3333 | −0.3333 | −0.0370 |
| 4 | 1.0000 | 0 | 0 |
| 5 | 1.0000 | 0 | 0 |
| 6 | **0.4110** | 0.5890 | 0.2043 |
| 7 | **0** | 1 | 1 |

Sorted, the eigenvalues are \(0,\ 0.4110,\ 1,\ 1,\ 1.3333,\ 1.3942,\ 1.8615\). Second-smallest = 0.4110 = algebraic connectivity. Exactly one zero, so one component.

### 7.5 Weighted spectral distribution

\[
W(G, N) = \sum_i (1 - \lambda_i)^N
\]

\(N\) chooses which local structures you are sensitive to. The course uses \(N = 3\) **because the graph has 3-cycles**, and gets

\[
W(G, 3) = \sum_{i=1}^{7} (1 - \lambda_i)^3 = 0.4667.
\]

Try other values of \(N\) when the structures you care about have a different size. A 4-cycle story would push you toward a different \(N\).

**After an attack,** recompute \(W\) and take the difference:

\[
\Delta W(G, N) = W_{\text{before}} - W_{\text{after}} \quad \text{(the change in } W\text{)}.
\]

The slide draws \(\Delta W\) going **up** as robustness goes **down**. A large change in the spectrum means the attack changed the structure a lot, so the network was fragile.

You are not expected to compute eigenvalues by hand. You are expected to know what 0 means, which eigenvalue is algebraic connectivity, why the exponent is 3 in the example, and what a large \(\Delta W\) says about robustness.

### 7.6 Degree

**Degree** of a node = number of edges touching it.

On a **directed** graph, split it:

- **Indegree**: edges coming in.
- **Outdegree**: edges going out.
- **Degree** = indegree + outdegree.

Worked line from the slides: indegree of G is 3, outdegree of G is 3, degree of G is 6.

Degree table used to build the histogram (13 nodes):

| Node | A | B | C | D | E | F | G | H | I | J | K | L | M |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Degree | 4 | 1 | 2 | 2 | 3 | 3 | 6 | 3 | 2 | 4 | 1 | 4 | 2 |

**Degree histogram:** for each degree, how many nodes have it.

| Degree | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Count | 0 | 2 | 4 | 3 | 3 | 0 | 1 |

That histogram is a **feature vector** for the whole graph. Two graphs can be compared by the distance between their vectors.

- **Euclidean distance** is the one the course names, along with “many other” histogram distances and statistical distances between distributions.
- Worked pair: \(G_1 = (0, 2, 4, 3, 3, 0, 1)\) and \(G_2 = (0, 3, 4, 3, 2, 1, 1)\) for degrees 0 through 6. They differ by 1 in the bins for degrees 1, 4, and 5. The slide reports the distance as **3** and calls the graphs quite similar.
- The sum of squared bin-wise differences is \(1 + 1 + 1 = 3\). If a question asks you to *compute* ordinary Euclidean distance, that is \(\sqrt{1^2 + 1^2 + 1^2} = \sqrt{3}\). Either way the conclusion in the course is “quite similar”. Say which formula you used.

**Joint degree histogram.** Bin nodes by the pair (indegree, outdegree), not by degree alone. The slide’s labels:

- **Talkers** — high out-degree, they send a lot.
- **Listeners** — high in-degree, they receive a lot.
- **Communicators** — high on both.

Two graphs with the same degree histogram can still differ here. One may be full of broadcasters, the other full of people who both speak and listen.

### 7.7 Degree distribution and power-law graphs

The **degree distribution** \(P(k)\) is the probability that a vertex has \(k\) neighbors. It is the histogram turned into probabilities.

**Power law, also called scale-free:**

\[
P(k) \sim k^{-\alpha}
\]

for some positive \(\alpha\). A few vertices have huge degree. Most have very small degree. On a log-log plot this is a straight line falling to the right.

**Log-normal:**

\[
P(k) = \frac{1}{\sqrt{2\pi}\,\sigma\, k}\, e^{-(\ln k - \mu)^2 / (2\sigma^2)}
\]

Power-law graphs are common: the Internet, social networks, protein–protein interaction networks, airline networks, financial networks.

Structural features the course lists:

- A few **hubs**, often essentially one dominant hub, with very high degree.
- A large number of nodes tied to hubs, directly or through a short chain.
- A **dense, low-degree** region around each hub.

**Why “scale-free” matters for robustness. This paragraph is a standard exam answer.**

- **Random failure is tolerated.** Almost every node has tiny degree, so a random deletion almost never hits a hub. If one hub does fail, other hubs usually keep the graph in one piece.
- **Targeted attack is devastating.** Delete a few major hubs and the graph falls into isolated pieces.

Hubs are both the strength and the weakness. Connect this back to valence and to the “whom do we immunize?” question later: in a scale-free contact network, immunizing hubs (a targeted attack on the disease’s graph) does far more than immunizing random people.

---

## 8. Community analytics

### 8.1 What a community is

Entities interact inside groups, and those interactions form clusters.

> A **community** is a dense subgraph whose vertices are more connected **to each other** than **to the rest of the graph**.

Running picture: vertices are researchers, edges are collaborations, a community is a group that publishes together.

Three families of questions:

| Family | Questions |
| --- | --- |
| **Static** | What are the communities at time \(T\)? Who is in one? How tightly knit is it? |
| **Predictive** | Will it grow? Will these nodes still form a community later? Are dominant roles appearing? |
| **Temporal** | How did it form? Which communities are stable? Which strong communities were transient, and why did they appear or dissolve? |

### 8.2 Detecting one with densities

Let \(C\) be a connected subgraph of \(G\), with \(n_C\) vertices, inside a graph of \(n\) vertices.

- **Internal degree** of a vertex: neighbors inside \(C\).
- **External degree** of a vertex: neighbors outside \(C\).
- **Internal / external degree of the cluster**: the sum of those, over vertices of \(C\). The internal sum counts every internal edge twice.

Densities use edge counts, so they are easier to compare across groups of different sizes:

\[
\delta_{\mathrm{int}} = \frac{\text{number of internal edges in } C}{n_C(n_C - 1)/2}
\]

\[
\delta_{\mathrm{ext}} = \frac{\text{number of edges leaving } C}{n_C\,(n - n_C)}
\]

The denominator of \(\delta_{\mathrm{int}}\) is the most edges \(C\) could possibly have. The denominator of \(\delta_{\mathrm{ext}}\) is the most edges that could run from \(C\) to the outside.

> \(C\) looks like a community when \(\delta_{\mathrm{int}}\) is **high** and \(\delta_{\mathrm{ext}}\) is **low**.

### 8.3 Clique, near-clique, core

These are local definitions. They look at a subgraph and its neighborhood, not at a global score.

| Object | Definition | Exam point |
| --- | --- | --- |
| **Clique** | Every two distinct vertices are adjacent. The “perfect” community. | Finding the **largest** clique is computationally hard. Finding cliques of a **fixed size \(k\)** is the easier ask. A clique on 5 vertices is drawn as the illustration. |
| **\(n\)-clique** | A maximal subgraph in which every pair of members has distance **at most \(n\)**. | \(n = 1\) is an ordinary clique. The distance is measured in the **whole graph**, so the path may walk through non-members. The slide marks this with a sad face: the members may not actually be close **inside** the group. |
| **\(n\)-clan** | An \(n\)-clique whose **geodesic distance inside the subgraph** is also at most \(n\). | Fixes the sad face. Members must be able to reach each other in \(\le n\) hops **without leaving the group**. |
| **\(k\)-core** | A maximal subgraph in which **every** vertex has at least \(k\) neighbors inside the subgraph. | This is a minimum-degree condition, not a “everyone knows everyone” condition. |

**How the cores nest.** A 3-core sits inside a 2-core, which sits inside a 1-core. Raising \(k\) peels off vertices that do not have enough neighbors inside. In the picture, A–B–C–D form the 3-core (each has 3 neighbors inside that block). Dangling vertices such as a node hanging off by a single edge die as soon as \(k\) reaches 2.

**\(n\)-clique versus \(n\)-clan, the way an exam asks it.**

Suppose Holly and Jenny are both in a proposed group, and the only path of length 2 between them goes through Sam, who is not in the group.

- They can still sit in a **2-clique**, because distance 2 exists in the full graph.
- They cannot sit in a **2-clan** unless some path of length \(\le 2\) uses only members.

The course quiz lists candidate groups from a named social graph (John, Gary, Mike, Jenny, Holly, Pam, Pat, Harry, Don, Carol, Paul, Ann, …) and asks which of them are 2-clans. The method above is the whole answer: check pairwise distance in the full graph, then check pairwise distance again using only the induced subgraph.

### 8.4 Modularity

Density judges one cluster. **Modularity** judges a **partition of the whole graph**.

> Modularity is the fraction of edges that fall inside the given groups, minus the fraction you would expect if the same degrees were wired at random.

The value lies in \([-1/2,\ 1)\). Higher means a cleaner community structure. The LinkedIn example later lands near 0.54, which is a solid but not perfect partition.

\[
Q = \frac{1}{2m} \sum_{ij} \bigl(A_{ij} - P_{ij}\bigr)\, \delta(C_i, C_j)
\]

| Symbol | Meaning |
| --- | --- |
| \(m\) | Number of edges |
| \(A\) | Adjacency matrix. \(A_{ij} = 1\) if the edge exists. |
| \(P_{ij}\) | Expected connection between \(i\) and \(j\) under the random model |
| \(C_i\) | Cluster of vertex \(i\) |
| \(\delta(C_i, C_j)\) | 1 if \(i\) and \(j\) are in the same cluster, otherwise 0 |

Only same-cluster pairs contribute, because of \(\delta\). A cluster helps \(Q\) when it contains **more internal edges than the random model expects**.

The random model keeps the degrees and throws the edges away. A vertex of degree \(k_i\) “owns” \(k_i\) edge-ends out of \(2m\) edge-ends in total. The probability that one random edge lands with one end on \(i\) and the other on \(j\) is

\[
p_{ij} = \frac{k_i}{2m} \cdot \frac{k_j}{2m} = \frac{k_i k_j}{4m^2}.
\]

There are \(2m\) edge-ends being paired, and the expected number of edges between \(i\) and \(j\) is

\[
P_{ij} = \frac{k_i k_j}{2m}.
\]

That is the \(P_{ij}\) inside the sum. Memorize \(P_{ij} = k_i k_j / (2m)\). The \(4m^2\) form is the probability the course uses to derive it.

**Maximizing \(Q\) exactly is too hard.** Every practical algorithm, including Louvain, is an approximation. Say that explicitly if you are asked whether the result is optimal.

### 8.5 Louvain method

Louvain is the modularity method the course demonstrates (the slides defer the animation to a video and then show the result).

**Phase 1 — move nodes.**

1. Put every vertex in its own community.
2. Visit each vertex. Try moving it into the community of each neighbor. Keep the move that gives the **largest positive gain** in \(Q\). If no move helps, leave it.
3. Repeat until a full pass moves nobody.

**Phase 2 — aggregate.**

1. Collapse each community into one super-vertex.
2. An edge between super-vertices gets a weight equal to the number (or total weight) of edges between those communities. Edges inside a community become a self-loop.
3. Run Phase 1 again on this much smaller graph.

Stop when aggregation no longer improves \(Q\). The result is a **hierarchy**: the last level is the coarsest partition, and intermediate levels are finer communities.

**Worked result from the slides.** On AG’s LinkedIn graph (315 nodes), resolution 1.0:

- One level finds **7** communities, \(Q \approx 0.5378\).
- The multi-level run finds **6** communities, \(Q \approx 0.5376\).

Resolution 1.0 is the setting printed on the figure. You will not be asked to recompute \(Q\) for 315 nodes. You may be asked what Louvain optimizes (modularity), why it aggregates (so it can see bigger communities), and that it is approximate.

### 8.6 How a community evolves

Take the membership of a community at a reference time \(t_0\), and again at later times. The course writes \(A(t_0)\) for the set of members.

**Autocorrelation** measures overlap of the membership. It is a Jaccard index:

\[
C(t) = \frac{\lvert A(t_0) \cap A(t_0 + t) \rvert}{\lvert A(t_0) \cup A(t_0 + t) \rvert}
\]

Numerator: nodes present at both times (the overlap). Denominator: nodes present at either time (the joint graph’s membership). \(C = 1\) means identical membership. \(C = 0\) means no one in common.

**Stationarity** is the average autocorrelation between successive snapshots:

\[
\zeta = \frac{\sum_{t = t_0}^{t_{\max} - 1} C(t,\ t + 1)}{t_{\max} - t_0 - 1}
\]

\[
1 - \zeta = \text{the average fraction of members who change in one step.}
\]

High \(\zeta\) (near 1) means a **stationary** community: the same people stay. Low \(\zeta\) means high churn.

The three pictures:

| Picture | What is going on |
| --- | --- |
| Small graph, not many changes | Size and membership stay put. Stationary. |
| Small graph, many changes | The group is small but the members are replaced. Non-stationary. Size alone would have fooled you. |
| Large graph, many changes | The community grows and then turns over. Stacked bands show members who are old, members who will leave at the next step, and members who just arrived. Non-stationary. |

Exam trap: **stable size is not stationarity.** Stationarity is about *who* remains.

---

## 9. Centrality analytics

### 9.1 The question

Not all nodes are equally important. What “important” means depends on the domain:

| Domain | The important node |
| --- | --- |
| Social network | An influencer |
| Transport | A junction station |
| Biology | A housekeeping gene |
| Computers | A central server |

**Centrality** scores one **node** (or one edge) from its position.

**Centralization** scores the **network**: how uneven the node scores are.

\[
\text{Centralization} = \frac{\sum_i \bigl(c_{\max} - c(v_i)\bigr)}{c_{\max}}
\]

A star, with one hub and many leaves, is **more centralized**. A mesh, where every node looks similar, is **less centralized**. If every node has the same centrality, every term in the sum is 0 and centralization is 0.

### 9.2 The measures

| Measure | Formula / rule | A high score means | Do not use it when |
| --- | --- | --- | --- |
| **Degree** | \((\text{number of edges on the node}) / (N - 1)\) | The node is a hub: many direct ties | Direct ties are a bad proxy for influence, for example when only ties to *important* nodes matter |
| **Group degree** | \((\text{edges into the group}) / (\text{number of non-members})\) | The group, treated as one super-node, touches a large share of outsiders. The course picture scores **1**: every outsider has an edge in. | You need paths of length greater than 1 |
| **Closeness** | Raw value: sum of shortest-path distances to all other nodes, possibly divided by \(N - 1\). The **score** runs the other way: low raw sum, high score. | The node is a few hops from everyone. In a shortest-path flow, it hears information early and can influence many nodes directly and indirectly. The course figure says **F has the highest closeness score**. | The thing being spread does **not** travel by shortest paths. The slide asks “how about gossip?” for this reason. |
| **Betweenness** | For each pair of other nodes, the fraction of their shortest paths that pass through \(i\), summed up. Standard writing: \(\sum_{s \ne i \ne t} \sigma_{st}(i) / \sigma_{st}\) | A large fraction of shortest-path traffic is forced through this node. It is a broker or a bottleneck. | Rumors, infections, and any flow that does not choose shortest paths. |
| **Eigenvector** | A node is important when its **neighbors** are important. \(C_E(v_i) \propto \sum_{v_j \text{ neighbor of } i} A_{ij}\, C_E(v_j)\). In matrix form \(x \propto Ax\), i.e. \(\lambda x = Ax\). | The node sits next to other important nodes. “It is not what you know, it is who you know.” | You need a simple count of ties, or the graph is directed and dangling nodes break the plain version. Use PageRank then. |
| **Katz** | Named in the list, not derived. | A walk-based variant of eigenvector centrality. | The course does not ask you to compute it. |
| **PageRank** | Eigenvector centrality plus random jumps. See below. | A random surfer spends a lot of time at this node. | The graph is undirected and “who knows whom” is the whole question. Eigenvector is enough. |

**Degree versus eigenvector, from the figure.** Two nodes can both have degree 2. Degree centrality gives them the same score. Eigenvector centrality does not: a degree-2 node whose neighbors are leaves scores around 0.11, while a degree-2 node wired into the important middle of the graph scores much higher. The measure is called **local** because a node’s score is computed from its neighbors’ scores — but those depend on their neighbors, so importance propagates.

**Betweenness, the small figure.** Nodes B–C–A–E–D–C, with B stuck on the side. The paths listed are BCA, BCAE, BCD, BCDE, CAE, CDE, AED. Scores printed on the figure: **B = 0** (a leaf, on no path between others), **C = 3.5** (most paths go through C), **A = 1**, **D = 1**, **E = 0.5**. You can be asked who is highest and why. You are rarely asked to recompute 3.5 from scratch.

### 9.3 PageRank

PageRank is the classical graph algorithm of web search (Page and Brin). It is eigenvector centrality with an escape hatch, so a page with no out-links cannot trap the walker.

**Random surfer.** Standing at \(v\), the surfer:

- follows a random outgoing edge with probability \(\alpha\), or
- jumps to a completely arbitrary node with probability \(1 - \alpha\).

\(\alpha\) is the damping factor. In the worked example \(\alpha = 0.85\), so the jump probability is \(0.15\).

**Meaning.** A random surfer is found most often at nodes that many nodes point to, directly or along a chain. PageRank is the **stationary distribution** of that walk: the probability of being at each node after the walk has been going forever.

**Power iteration.** Store the scores in an \(n\)-vector \(PR\). Let \(M\) be the transition matrix of the surfer model.

\[
PR(t + 1) = M \cdot PR(t)
\]

Repeat until \(\lvert PR(t + 1) - PR(t) \rvert < \varepsilon\).

**Worked graph.** Three nodes. A points to B and to C. B points to A. C points to A. So A has out-degree 2, and both B and C have out-degree 1. A is the page everyone cites.

The course writes the update **without** dividing the jump term by \(N\):

\[
\begin{align*}
P(A) &= (1 - \alpha) + \alpha \left(\frac{P(B)}{1} + \frac{P(C)}{1}\right) \\
P(B) &= (1 - \alpha) + \alpha \left(\frac{P(A)}{2}\right) \\
P(C) &= (1 - \alpha) + \alpha \left(\frac{P(A)}{2}\right)
\end{align*}
\]

Start at 0. The slide itself asks whether 0 is a bad start. It is legal; it just wastes the first iteration. With \(\alpha = 0.85\):

| Iteration | P(A) | P(B) | P(C) |
| --- | --- | --- | --- |
| 1 | \(0.15 + 0.85 \cdot 0 = 0.15\) | \(0.15 + 0.85 \cdot (0.15 / 2) = 0.21\) | 0.21 |
| 2 | \(0.15 + 0.85 \cdot (0.21 + 0.21) = 0.51\) | \(0.15 + 0.85 \cdot (0.51 / 2) = 0.37\) | 0.37 |
| 3 | \(0.15 + 0.85 \cdot (0.37 \cdot 2) = 0.78\) | \(0.15 + 0.85 \cdot (0.78 / 2) \approx 0.48\) | 0.48 |
| 20 | **1.46** | **0.77** | **0.77** |

A stays ahead because both of the other pages point at it. B and C stay equal because the graph is symmetric between them. The scores sum to 3, the number of nodes, because this version adds a full \((1 - \alpha)\) at every node instead of \((1 - \alpha)/N\). If you have seen the other form in a textbook, both are the same ranking up to a constant scale. Use the form printed above when the question is about this module. One line on the slide writes \(0.87\) in iteration 3; the arithmetic that produces 0.48 uses \(0.78\).

### 9.4 The two key-player problems

Centrality scores nodes **one at a time**. Key-player problems pick a **set** of \(k\) nodes for a job. The best set is not “the \(k\) nodes with the highest individual centrality”, because those \(k\) nodes often cover the same neighborhood.

**Problem 1 — disruption.**

> Which \(k\) nodes should we remove to damage communication as much as possible?

Stories: immunize a subpopulation to slow an infection; remove brokers from a network. In the Krebs 2002 terrorist network, removing A, B, and C breaks the graph into **7 components** and gives fragmentation \(F = 0.59\).

Suppose the removal leaves distances \(d_{ij}\) (infinite when \(i\) and \(j\) are disconnected; then \(1/d_{ij} = 0\)).

\[
F = 1 - \frac{2 \sum_{i < j} 1/d_{ij}}{n(n - 1)}
\]

Read the fraction as “how close pairs still are, on average”. \(F\) near 0 means pairs are still close (every pair at distance 1 gives \(F = 0\)). \(F\) near 1 means the graph has been shattered. You want the set whose removal **maximizes** \(F\).

**Problem 2 — reach.**

> Which set \(S\) of size \(k\) reaches the most people within \(k\) hops?

Story: in a community of 1000, find 5 people who can persuade the rest to vote. In the same terrorist network, targeting A, C, and D reaches **100%** of the nodes. Notice A, C, D rather than A, B, C. The best set for spreading is not the best set for disrupting.

\(d_{Sj}\) is the distance from node \(j\) to the nearest member of \(S\).

\[
{}^{D}\!R = 1 - \frac{\sum_j 1/d_{Sj}}{n}
\]

The subtracted term is large when every node is close to \(S\), because \(1/d\) is large for small \(d\) and is 0 when \(j\) cannot be reached. The goal stated in words on the slide is the one to optimize: **reach as many nodes as possible, in as few hops as possible, from a small set.**

| | Key Player 1 | Key Player 2 |
| --- | --- | --- |
| Verb | Remove, immunize, disrupt | Recruit, persuade, broadcast |
| Metric | Fragmentation \(F\) | Distance-weighted reach |
| Good set | Deletion shatters the graph | The set sits close to everyone |
| Krebs example | Remove A, B, C → 7 components, \(F = 0.59\) | Use A, C, D → 100% of the network reached |

---

## 10. Comparisons that get asked directly

### Chart vs graph

| | Chart | Graph in this course |
| --- | --- | --- |
| What it is | A function, input mapped to a number | A set of vertices plus a set of edges |
| Example | Sales by month, counts by product category | Who replies to whom, which gene interacts with which protein |
| Field | Ordinary plotting | Graph theory, from Euler, 1736 |

### Walk, trail, path, cycle

| | Repeats vertices? | Repeats edges? | Closed? |
| --- | --- | --- | --- |
| Walk | Allowed | Allowed | Not required |
| Trail | Allowed | No | Not required |
| Path | No, except possibly the two ends | No | Not required |
| Cycle | Only the shared start/end | No | Yes, and length at least 3 |

### Strong vs weak

| | Follow the arrows? | The A–J example |
| --- | --- | --- |
| Strong | Yes, and in **both** directions for every pair | Fails. H is unreachable. A reaches nobody. |
| Weak | No. Ignore arrows, then ask if it is in one piece. | Passes. |

### Vertex-connectivity vs edge-connectivity

| | Delete | Name of the set | Name of the minimum size |
| --- | --- | --- | --- |
| Vertex | Nodes | Separating set | Connectivity \(k(G)\) |
| Edge | Edges | Disconnecting set | Edge-connectivity |

### The four V’s, one sentence each

- **Volume** — too many nodes and edges, so algorithms and memory fail. Path *counts* explode.
- **Velocity** — edges arrive as a stream that will not fit in memory.
- **Variety** — mixed sources, and mixed meanings (social vs citation vs interaction vs ontology).
- **Valence** — how interdependent the nodes are. It usually increases: denser locally, shorter distances globally.

### Power-law failure vs attack

| Hit the network how? | What happens | Why |
| --- | --- | --- |
| Random nodes fail | Usually stays connected | The victim is almost never a hub |
| A few hubs are removed | Falls into pieces | The remaining low-degree nodes were only connected through the hubs |

### Community objects

| | Every pair adjacent? | Allows distance \(> 1\)? | Path may leave the set? | Condition |
| --- | --- | --- | --- | --- |
| Clique | Yes | No | No | Perfect, and the largest one is hard |
| \(n\)-clique | Only if \(n = 1\) | Up to \(n\) | **Yes** | Maximal |
| \(n\)-clan | Only if \(n = 1\) | Up to \(n\), **inside the set** | **No** | An \(n\)-clique with internal geodesics \(\le n\) |
| \(k\)-core | Not required | — | — | Every member has \(\ge k\) neighbors **in the core** |

### Centrality, one line each

- **Degree** — how many ties.
- **Closeness** — how near everyone else is.
- **Betweenness** — how much shortest-path traffic you carry.
- **Eigenvector** — how important your neighbors are.
- **PageRank** — eigenvector, plus a random jump so sinks do not trap the walk.
- **Centralization** — how unequal those scores are across the network. Not a node score.
- **Group degree** — degree of a whole set, divided by the number of outsiders.

### Dijkstra and its two upgrades

| | Searches from | Uses a hint about the target? | Extra requirement |
| --- | --- | --- | --- |
| Dijkstra | Source only | No | Non-negative weights |
| Bidirectional | Source and target | The target’s identity only | Check least **weight** at the meeting node |
| Goal-directed | Source, biased toward target | Yes, a potential \(\lambda\) | \(w' = w - \lambda(u) + \lambda(v) \ge 0\) |

---

## 11. Formula sheet

\[
\begin{align*}
&\text{Degree centrality} && \frac{\text{edges on } v}{N - 1} \\
&\text{Group degree centrality} && \frac{\text{edges into the group}}{\text{number of non-members}} \\
&\text{Centralization} && \frac{\sum_i (c_{\max} - c(v_i))}{c_{\max}} \\
&\text{Intra-cluster density} && \delta_{\mathrm{int}} = \frac{\text{internal edges}}{n_C(n_C - 1)/2} \\
&\text{Inter-cluster density} && \delta_{\mathrm{ext}} = \frac{\text{edges leaving } C}{n_C(n - n_C)} \\
&\text{Modularity} && Q = \frac{1}{2m}\sum_{ij}(A_{ij} - P_{ij})\,\delta(C_i, C_j), \quad P_{ij} = \frac{k_i k_j}{2m} \\
&\text{Normalized Laplacian} && L_{uu} = 1,\quad L_{uv} = -1/\sqrt{d(u)d(v)}\ \text{if edge exists} \\
&\text{Algebraic connectivity} && \text{second-smallest eigenvalue of } L \\
&\text{Components} && \text{count of eigenvalues equal to } 0 \\
&\text{WSD} && W(G, N) = \sum_i (1 - \lambda_i)^N \\
&\text{Power law} && P(k) \sim k^{-\alpha} \\
&\text{Log-normal} && P(k) = \frac{1}{\sqrt{2\pi}\,\sigma\, k}\exp\!\bigl(-(\ln k - \mu)^2/(2\sigma^2)\bigr) \\
&\text{Goal-directed weight} && w'(u, v) = w(u, v) - \lambda(u) + \lambda(v) \\
&\text{Dijkstra cost} && \text{proportional to } |E|\log|V| \\
&\text{Autocorrelation} && C(t) = \frac{|A(t_0) \cap A(t_0+t)|}{|A(t_0) \cup A(t_0+t)|} \\
&\text{Stationarity} && \zeta = \text{average of successive } C(t, t+1),\quad 1 - \zeta = \text{average churn} \\
&\text{Fragmentation} && F = 1 - \frac{2\sum_{i<j} 1/d_{ij}}{n(n-1)} \\
&\text{Distance-weighted reach} && {}^{D}\!R = 1 - \frac{\sum_j 1/d_{Sj}}{n} \\
&\text{Eigenvector} && \lambda x = Ax \\
&\text{PageRank step} && PR \leftarrow M\cdot PR,\ \text{stop when the change is below }\varepsilon
\end{align*}
\]

Numbers worth memorizing from the worked examples:

- Least-weight path I→F→E→B has weight **30**. H is unreachable from I.
- Diameter read from the hop matrix: **4** (I to A).
- Algebraic connectivity of the 7-node graph: **0.4110**. One zero eigenvalue. \(W(G, 3) = 0.4667\).
- Louvain on the 315-node LinkedIn graph, resolution 1: **Q ≈ 0.54**, **6 or 7** communities.
- PageRank after 20 iterations on the 3-node graph: **A = 1.46, B = 0.77, C = 0.77**, with \(\alpha = 0.85\).
- Krebs network: remove A, B, C → **7 components, F = 0.59**. Reach with A, C, D → **100%**.
- Two degree histograms in the similarity example differ by a reported distance of **3** and are called quite similar.

---

## 12. Practice questions

Answers are underneath each question. Cover them and try first.

**1.** Define analytics, and define graph analytics.

Analytics discovers and communicates meaningful patterns, using mathematical properties, computing, domain knowledge, and statistical models. Graph analytics is that activity on data that is a graph, or that can be modeled as one.

**2.** A slide shows a bar chart of product category against units sold and asks “is this a graph?” What do you answer?

No. It is a chart of a function, from category to total. A graph in this course is a set of vertices and a set of edges.

**3.** Give both definitions of a graph.

Mathematical: \(G = (V, E)\), vertices and edges, each edge a pair of vertices. Computer science: an abstract data type with a representation plus operations such as `Add_edge`, `Add_vertex`, and `Get_neighbor`.

**4.** In an adjacency matrix, what does `Add_edge(v3, v2)` change, and how does `Get_neighbor(v3)` work?

It writes a 1 at row v3, column v2. `Get_neighbor` scans that row and that column and returns every vertex whose cell is 1.

**5.** List the extra ingredients in the extended graph model, beyond \(V\) and \(E\).

Node types and a type assignment, edge types and a type assignment, node attributes, edge attributes, and the domain of each attribute.

**6.** Give four meanings a weight can have.

Distance on a road, strength of a personal tie, likelihood of a biological interaction, certainty of a piece of knowledge.

**7.** What is a loop? What is a multigraph, and why would you allow one?

A loop is an edge from a vertex to itself. A multigraph has several edges between the same pair, because each edge carries different information.

**8.** Name the four V’s and one analytic consequence of each.

Volume: algorithms and memory stop coping; the number of simple paths is exponential. Velocity: updates are an edge stream that does not fit in memory. Variety: graphs are integrated from relational, XML/JSON, document, and graph sources, and a social edge does not mean the same thing as a citation edge. Valence: higher connectedness is what analytics uses, and it tends to increase, with denser regions and shorter average distances.

**9.** Define walk, trail, path, and cycle.

A walk is an alternating sequence of vertices and edges. A trail repeats no edge. A path repeats no vertex, except possibly the first and last. A cycle is a path of length at least 3 that starts and ends at the same vertex.

**10.** The course graph has \(\infty\) in its distance matrix, and 4 is circled. What do you report as the diameter, and is the graph strongly connected?

The largest finite shortest-hop distance is 4, from I to A, which is the diameter the slide marks. The graph is not strongly connected, because some pairs have no directed path. It is weakly connected.

**11.** Dijkstra from I finishes B at distance 30 and leaves H at infinity. Explain both facts, and write the path.

Relaxation through F and then E gives \(d[B] = 30\) with predecessors B←E←F←I, so the path is I→F→E→B with weights 5, 20, and 5. No directed path from I reaches H, so H stays at its initial infinity.

**12.** Why can a 2-hop path lose to a 3-hop path in bidirectional Dijkstra?

The objective is least weight, not fewest edges. \(F \to C \to A\) can weigh more than \(F \to G \to D \to A\).

**13.** Write the goal-directed weight and the condition on it. Why did F→E become preferable to F→G in the example?

\(w'(u,v) = w(u,v) - \lambda(u) + \lambda(v)\), and \(w'\) must be non-negative. With landmark distances to B, \(w'(F \to E) = 15\) and \(w'(F \to G) = 70\), so the search is pushed toward the target.

**14.** Define separating set, connectivity, disconnecting set, and edge-connectivity.

A separating set is a vertex set whose deletion leaves more than one component. Connectivity is the size of the smallest one. A disconnecting set is an edge set with the same effect. Edge-connectivity is the size of the smallest one.

**15.** What does a zero eigenvalue of the normalized Laplacian tell you? Which eigenvalue is the algebraic connectivity in the example?

The number of zero eigenvalues equals the number of components. Algebraic connectivity is the second-smallest eigenvalue, 0.4110 in the example.

**16.** Why does the example use \((1 - \lambda)^3\), and what does a large \(\Delta W\) mean?

The exponent matches the 3-cycles in the graph (there are two of them). \(W(G, 3) = 0.4667\). A large change in \(W\) after an attack means the structure changed a lot, so robustness is low.

**17.** State the power-law form and the robustness claim.

\(P(k) \sim k^{-\alpha}\). The network tolerates random failures because hubs are rare, and it breaks if the hubs are removed on purpose.

**18.** When is a subgraph a community, in the density definition?

When internal density is high and external density is low. Write both formulas if asked: internal edges over \(n_C(n_C-1)/2\), and leaving edges over \(n_C(n - n_C)\).

**19.** Why is an \(n\)-clique sometimes a bad community, and what does an \(n\)-clan add? How is a \(k\)-core different from both?

An \(n\)-clique only requires distance \(\le n\) in the full graph, so the connecting path may leave the set. An \(n\)-clan is an \(n\)-clique whose distance **inside the set** is also \(\le n\). A \(k\)-core does not require short paths between everyone. It requires every member to have at least \(k\) neighbors inside the core. Finding a largest clique is hard; a fixed size \(k\) is the easier version.

**20.** Write \(Q\), say what \(P_{ij}\) is, and say what Louvain does and does not guarantee.

\(Q = \frac{1}{2m}\sum_{ij}(A_{ij} - P_{ij})\delta(C_i, C_j)\) with \(P_{ij} = k_i k_j/(2m)\), the expected edges under a random model that preserves degrees. Range \([-1/2, 1)\). Louvain greedily moves nodes to improve \(Q\), then collapses communities and repeats. It approximates the maximum. It does not guarantee the best partition. On the 315-node example, \(Q \approx 0.54\) with 6 or 7 communities.

**21.** A community stays the same size for ten weeks, but only a third of the members are the same from one week to the next. Is it stationary?

No. Stationarity is about membership overlap, not size. Autocorrelation \(C\) would be low, \(\zeta\) would be low, and \(1 - \zeta\) would show that a large fraction of members change at each step.

**22.** For each task, name the centrality measure and say why the others are weaker. (a) Find a junction that most shortest routes use. (b) Find who will hear a shortest-path broadcast first. (c) Find who has the most contacts. (d) Find who is tied to other important people. (e) Rank web pages.

(a) Betweenness. (b) Closeness. (c) Degree. (d) Eigenvector. (e) PageRank. Betweenness is the wrong tool for a rumor or an infection, because those do not travel by shortest paths. Closeness has the same blind spot for gossip.

**23.** Distinguish centrality and centralization. What does a star score, relative to a mesh?

Centrality is a node score. Centralization is a network score: \(\sum(c_{\max} - c(v_i)) / c_{\max}\). A star is more centralized. A regular mesh is less centralized, and a network where every score is equal has centralization 0.

**24.** State the two key-player problems. Which Krebs set goes with which?

Problem 1 removes a set to maximize disruption, measured by fragmentation. A, B, C leave 7 components and \(F = 0.59\). Problem 2 picks a set that reaches as many people as possible in few hops. A, C, D reach 100%. The sets differ, so “top three centrality nodes” is not an acceptable shortcut.

**25.** Match the question to the family. (a) Energy-optimal routing in a city. (b) Will this power grid split if two substations fail? (c) Which research groups exist in a co-authorship graph? (d) Are these two friendship graphs structurally alike? (e) Whom should a campaign brief so the message reaches a 1000-person community?

(a) Path analytics, a least-weight path. (b) Connectivity: separating sets, algebraic connectivity, \(\Delta W\). (c) Community analytics: modularity or Louvain, or a \(k\)-core if you only want a dense nucleus. (d) Degree histograms and a vector distance. (e) Key Player Problem 2.

**26.** In the PageRank example, why does A end above B and C, and why are B and C equal?

Both B and C point to A, and A splits its own out-links between them. The graph is symmetric between B and C, so their updates are identical. After 20 iterations the scores are 1.46, 0.77, and 0.77.

**27.** A biological graph was integrated from three databases and a pile of papers. Which V’s just increased, and what kind of question does the integration newly allow?

Volume and variety both go up, because there is more data and it came from different kinds of sources. Valence can go up too, because entities that used to be disconnected are now linked. The new question is an indirect one that no single source contained, such as a path from one disease to another.

---

## 13. Short checklist the night before

- A chart is a function. A graph is vertices and edges. Euler, Königsberg, 1736.
- Extended model: types, type assignments, attributes, domains, weights.
- Every path is a trail, and every trail is a walk. A cycle is a closed path of length at least 3.
- Reachability respects direction. Diameter in the example is 4. The example is weakly, not strongly, connected.
- Dijkstra: relax, then read predecessors. I→F→E→B, weight 30. Cost \(\lvert E\rvert \log\lvert V\rvert\). No negative weights.
- Bidirectional: least weight, not least hop. Goal-directed: \(w - \lambda(u) + \lambda(v) \ge 0\).
- Separating set is vertices. Disconnecting set is edges.
- Zeros of the Laplacian count components. Algebraic connectivity is the second-smallest eigenvalue. Large \(\Delta W\) means fragile. Exponent \(N\) follows the cycle length you care about.
- Power law: random failure is fine, hub attack is fatal.
- Community: \(\delta_{\mathrm{int}}\) high, \(\delta_{\mathrm{ext}}\) low. Clique is perfect and hard. \(n\)-clan fixes the “path leaves the group” flaw of the \(n\)-clique. \(k\)-core is a minimum internal degree.
- Modularity compares against \(k_i k_j / (2m)\). Louvain approximates it by moving, then collapsing.
- Stationarity is churn of **members**, not stability of size.
- Degree, closeness, betweenness, eigenvector, PageRank: use the decision table. Centralization describes the network.
- Key player 1 breaks things (A, B, C). Key player 2 reaches things (A, C, D).
