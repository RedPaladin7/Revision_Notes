# DSA Solving Guide

How to recognize a LeetCode problem and which boilerplate to reach for. Not a problem list. Each section is: **signal → intuition → template → traps**.

C++ is the dialect. Snippets assume the usual headers (`vector`, `string`, `array`, `queue`, `stack`, `deque`, `unordered_map`, `unordered_set`, `map`, `algorithm`, `numeric`, `functional`) and LeetCode's `ListNode` / `TreeNode`. `priority_queue` lives in `<queue>`.

**How to use a problem:**

1. Write the brute force in one sentence and its complexity.
2. Read `n`. That number picks the legal complexity (table below). If brute force is already inside the budget, code it.
3. Name the bottleneck ("I rescan the left part every time", "I try every subset").
4. Match that bottleneck to the recognition map, then copy the template and change only the predicate.
5. Before typing, say the **invariant** out loud: what is true about the window, the stack, or `dp[i]` after each step.

| `n` up to | You can afford | Typical tool |
|---|---|---|
| 10–12 | `n!` or `2^n * n` | permutations, TSP bitmask |
| 20–22 | `2^n * n` | subset enumeration, bitmask DP |
| 100 | `n^3` | interval DP, Floyd |
| 500–5_000 | `n^2` | 1D/2D DP, LIS `O(n^2)` |
| 10^5 | `n log n` | sort, heap, binary search, balanced tree |
| 10^6 | `n` or `n log n` with a tiny log | two pointers, sliding window, prefix, DSU, linear DP |
| 10^8+ | `n` or better | math, bucket, single pass |

If the constraint and your algorithm disagree, you picked the wrong pattern. Do not micro-optimize the wrong one.

---

## Recognition map

Read the ask, then the data shape.

| You notice | Pattern |
|---|---|
| Need a partner, duplicate, or frequency in unsorted data | hash map / set |
| Many range-sum queries, or "subarray sum equals K" | prefix sum |
| Sorted array, pair/triplet with a target, palindrome from both ends | two pointers, opposite ends |
| Partition, remove duplicates in place, fast/slow on a list | two pointers, same direction |
| Contiguous subarray/substring, longest/shortest under a constraint | sliding window |
| "Exactly K" window counts | `atMost(K) - atMost(K-1)` |
| Next greater/smaller, histogram, stock span, valid brackets | monotonic stack |
| Max/min in every window of size K | monotonic deque |
| Sorted search, first true in a monotonic predicate, "minimize the maximum" | binary search |
| Cycle, middle, reverse, merge lists | linked list + dummy node |
| Path, height, LCA, "return something to parent" | tree DFS |
| Level by level, shortest path in an unweighted graph | BFS |
| "All subsets / permutations / combinations", constraint satisfaction | backtracking |
| Kth, top K, merge K sorted, running median | heap |
| Overlapping ranges, meetings, "merge if overlap" | sort intervals |
| "As many as possible" with a local swap argument | greedy |
| Prerequisites, course schedule, alien order | topological sort |
| Redundant edge, "are they connected", dynamic connectivity | union-find |
| Cheapest path, non-negative weights | Dijkstra |
| Optimal value built from smaller indexes or a grid | DP |
| Grid, two strings, knapsack | 2D DP |
| Best score on a subarray `i..j` | interval DP |
| `n ≤ 20` and subsets are the state | bitmask DP |

**Count vs optimize vs decide** changes the recurrence, not the pattern:

- number of ways → **add** the options
- min / max cost → **min / max** the options
- yes / no → **or** the options (or a boolean)

---

## 1. Arrays and hashing

**Signal.** Unsorted data, and you need existence, frequency, a complement, or a group. A nested scan is `O(n^2)` only because you keep asking "have I seen this before?"

**Intuition.** Trade memory for a second pass. The map is an index of everything to the left of `i`, so each element is processed once.

### Pattern A — complement (Two Sum)

While walking left to right, the map holds values already seen. The partner of `x` is `target - x`. Check before insert so a value is not paired with itself.

```cpp
vector<int> twoSum(vector<int>& nums, int target) {
    unordered_map<int, int> seen; // value -> index
    for (int i = 0; i < (int)nums.size(); i++) {
        int need = target - nums[i];
        if (seen.count(need)) return {seen[need], i};
        seen[nums[i]] = i;
    }
    return {};
}
```

Same shape: "two numbers that differ by K", "check if a pair sums to 0".

### Pattern B — frequency or canonical key (group anagrams, valid anagram)

An anagram is the same multiset of characters. The key is either the sorted string or a length-26 count tuple. Group by that key.

```cpp
vector<vector<string>> groupAnagrams(vector<string>& strs) {
    map<array<int, 26>, vector<string>> groups;
    for (auto& s : strs) {
        array<int, 26> key{};
        for (char c : s) key[c - 'a']++;
        groups[key].push_back(s);
    }
    vector<vector<string>> ans;
    for (auto& [key, group] : groups) ans.push_back(group);
    return ans;
}
```

Use a count key when strings are long and the alphabet is small. Use `sorted(s)` when the alphabet is large or the code has to stay obvious.

### Pattern C — prefix as the key (subarray sum equals K)

This is the most important hash pattern after Two Sum. If `prefix[r] - prefix[l] = k`, then `prefix[l] = prefix[r] - k`. Store how many times each prefix has occurred.

```cpp
int subarraySum(vector<int>& nums, int k) {
    int count = 0, prefix = 0;
    unordered_map<int, int> seen{{0, 1}}; // empty prefix
    for (int x : nums) {
        prefix += x;
        count += seen[prefix - k];
        seen[prefix]++;
    }
    return count;
}
```

The `seen[0] = 1` is the whole trick. Without it, a subarray that starts at index 0 is invisible.

Same shape: continuous array (prefix of 0s and 1s, treat 0 as -1, find longest prefix that repeats), "binary subarrays with sum K", number of subarrays divisible by P (`prefix % P` repeats).

### Pattern D — in-place mark, values are the index

When the array is a permutation of `1..n` (missing number, find duplicate, first missing positive), the index *is* the hash. Cyclic sort: each number `x` belongs at index `x` or `x - 1`.

```cpp
void cyclicSortPlace(vector<int>& nums) {
    int n = nums.size(), i = 0;
    while (i < n) {
        int correct = nums[i] - 1; // if values are 1..n
        if (correct >= 0 && correct < n && nums[i] != nums[correct])
            swap(nums[i], nums[correct]);
        else
            i++;
    }
}
```

After placement, the first index whose value is not `i + 1` is the answer. Do not increment `i` when you swap — the swapped-in value has not been placed yet.

**Traps.**

- Inserting into the map before the lookup pairs an element with itself.
- Forgetting `seen[0] = 1` on prefix problems.
- Using a set when you need the index or the count. A set answers "seen?". A map answers "where?" or "how many?".
- Sorting to find a pair is fine at `O(n log n)` and uses no extra memory. Hashing is the linear option.

---

## 2. Prefix sums and difference arrays

**Signal.** Many queries of the form "sum from L to R", or many range updates and one final array. Recomputing the range each time is the bottleneck.

**Intuition.** `prefix[i]` = sum of everything strictly before `i` (or up to `i` — pick one and stick to it). Any range is then one subtraction: `prefix[r+1] - prefix[l]`.

```cpp
vector<long long> buildPrefix(vector<int>& nums) {
    vector<long long> prefix(nums.size() + 1);
    for (int i = 0; i < (int)nums.size(); i++)
        prefix[i + 1] = prefix[i] + nums[i];
    return prefix;
}

long long rangeSum(vector<long long>& prefix, int l, int r) { // inclusive
    return prefix[r + 1] - prefix[l];
}
```

**2D prefix.** `pref[r+1][c+1]` includes the rectangle `(0,0)` through `(r,c)`. A sub-rectangle is inclusion-exclusion: big rectangle minus the part above minus the part left plus the corner you subtracted twice.

```cpp
vector<vector<int>> buildPrefix2d(vector<vector<int>>& grid) {
    int rows = grid.size(), cols = grid[0].size();
    vector<vector<int>> p(rows + 1, vector<int>(cols + 1));
    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++)
            p[r + 1][c + 1] = grid[r][c] + p[r][c + 1] + p[r + 1][c] - p[r][c];
    return p;
}
```

**Difference array.** You must add `+v` on `[L, R]` many times, then read the final array once. Do not touch every index of every update.

```cpp
vector<int> rangeAdd(vector<int>& nums, vector<array<int, 3>>& updates) {
    int n = nums.size();
    vector<int> diff(n + 1);
    for (auto [l, r, v] : updates) {
        diff[l] += v;
        diff[r + 1] -= v;
    }
    int running = 0;
    for (int i = 0; i < n; i++) {
        running += diff[i];
        nums[i] += running;
    }
    return nums;
}
```

The `+v` at `L` starts the effect. The `-v` at `R+1` ends it. The prefix of `diff` is the total added at each index.

**Product except self** is a prefix of products from the left times a prefix from the right. Two passes, no division.

---

## 3. Two pointers

**Signal.** A pair or a region inside a **sorted** array, a palindrome, or an in-place partition. Brute force checks every pair. Sorting (or an already sorted input) lets one pointer replace the inner loop.

**Intuition.** Each pointer moves only in one direction, so together they do `O(n)` work. The move is forced: if the pair is too small, only raising the left end helps; if too big, only lowering the right end helps. That implication is only true when the array is sorted.

### Opposite ends

```cpp
vector<int> twoSumSorted(vector<int>& nums, int target) { // nums sorted
    int lo = 0, hi = (int)nums.size() - 1;
    while (lo < hi) {
        int s = nums[lo] + nums[hi];
        if (s == target) return {lo, hi};
        if (s < target) lo++;
        else hi--;
    }
    return {};
}
```

**3Sum.** Sort. Fix `i`. Run the opposite-ends scan on `i+1 .. n-1` looking for `-nums[i]`. Skip duplicates at all three positions if the problem wants unique triplets.

```cpp
vector<vector<int>> threeSum(vector<int>& nums) {
    sort(nums.begin(), nums.end());
    vector<vector<int>> res;
    int n = nums.size();
    for (int i = 0; i < n; i++) {
        if (i && nums[i] == nums[i - 1]) continue;
        int lo = i + 1, hi = n - 1;
        while (lo < hi) {
            int s = nums[i] + nums[lo] + nums[hi];
            if (s == 0) {
                res.push_back({nums[i], nums[lo], nums[hi]});
                lo++;
                hi--;
                while (lo < hi && nums[lo] == nums[lo - 1]) lo++;
                while (lo < hi && nums[hi] == nums[hi + 1]) hi--;
            } else if (s < 0) {
                lo++;
            } else {
                hi--;
            }
        }
    }
    return res;
}
```

Container With Most Water is the same loop, but the move is "discard the shorter side" because width shrinks either way and height is limited by the min.

### Same direction

Both pointers start on the left. `write` lags behind `read`. Used for remove-duplicates, move-zeroes, sort colors (Dutch flag: three pointers `lo / mid / hi`).

```cpp
int removeDuplicatesSorted(vector<int>& nums) {
    if (nums.empty()) return 0;
    int write = 1;
    for (int read = 1; read < (int)nums.size(); read++) {
        if (nums[read] != nums[write - 1])
            nums[write++] = nums[read];
    }
    return write;
}
```

**Dutch flag** (sort 0, 1, 2). `mid` scans. `0` swaps with `lo` and both advance. `2` swaps with `hi` and only `hi` retreats (`mid` must re-read the swapped value). `1` just advances `mid`.

**When this is the wrong tool.** Unsorted + existence of one pair → hash map, which is linear and does not need an ordering. Two pointers after a sort is `O(n log n)` and wins when you need the pair in order, need uniqueness via a sort, or cannot use extra memory.

**Trap.** Moving the wrong pointer. State the implication: "sum too small ⇒ left must increase". If you cannot say that sentence, the array is not sorted in the way you think and the algorithm is wrong.

---

## 4. Sliding window

**Signal.** Contiguous subarray or substring. The question is longest, shortest, or a count, under a constraint you can update in `O(1)` when one element enters or leaves (sum, distinct count, char frequency, "at most K zeros").

**Intuition.** The window is always a valid candidate, or it is one step away from becoming valid. The right edge only grows. The left edge only grows. Each index enters once and leaves once, so the scan is `O(n)` even though it looks nested.

### Variable window — shrink while invalid

```cpp
int longestWithAtMostKDistinct(string& s, int k) {
    unordered_map<char, int> freq;
    int left = 0, best = 0;
    for (int right = 0; right < (int)s.size(); right++) {
        freq[s[right]]++;
        while ((int)freq.size() > k) {
            if (--freq[s[left]] == 0) freq.erase(s[left]);
            left++;
        }
        best = max(best, right - left + 1);
    }
    return best;
}
```

Swap the `while` condition for the problem's definition of "illegal":

| Ask | Shrink while |
|---|---|
| longest substring without repeating chars | `freq[s[right]] > 1` after adding |
| longest with at most K zeros (flip to 1s) | zeros inside window `> k` |
| minimum size subarray with sum `≥ target` | track the min **before** shrinking, shrink while `sum >= target` |
| smallest window containing all of `t` | shrink while every required char is still satisfied |

Minimum window is the inverted shape: expand until valid, then shrink to the shortest valid, then expand again.

### Fixed window of size K

Add `s[right]`, and once `right >= k`, remove `s[right - k]` before scoring. Or score first and then remove. Pick one order and keep it.

```cpp
int maxSumWindowK(vector<int>& nums, int k) {
    int window = 0;
    for (int i = 0; i < k; i++) window += nums[i];
    int best = window;
    for (int right = k; right < (int)nums.size(); right++) {
        window += nums[right] - nums[right - k];
        best = max(best, window);
    }
    return best;
}
```

### Exactly K

Windows are not monotonic in "exactly K": growing can go from valid to invalid to valid. Do not write that `while`. Use:

```text
count(exactly K) = count(atMost K) - count(atMost K - 1)
```

`atMost` *is* monotonic, so the template above applies. This is the boilerplate for "subarrays with K different integers" and "binary subarrays with sum K" (sum K on a 0/1 array is "exactly K ones").

**Traps.**

- Forgetting to erase a key when its count hits 0. `freq.size()` then lies.
- Using a window on a **non-contiguous** subsequence. Windows only work when taking an element forces you to take everything between the ends.
- Sliding a window of variable size for a problem that is actually Kadane (no constraint other than contiguity and max sum). Kadane is DP, section 16.

---

## 5. Stacks, monotonic stacks, monotonic deques

**Signal.** Nested structure (brackets, function calls, histograms), or "the nearest larger/smaller element to the left or right".

**Intuition.** A stack remembers unfinished work in the order you must finish it (LIFO). A **monotonic** stack keeps indexes whose values are sorted. When a new value breaks the order, every index it resolves is one whose answer is this new value — those indexes will never find a closer answer later.

### Matching

Push opens. On a close, the stack top must be the matching open. A leftover stack or an early empty pop means invalid.

### Next greater element

Stack stores indexes. Values on the stack stay **decreasing** from bottom to top. When `nums[i]` is greater than the top, it is the next greater for that index.

```cpp
vector<int> nextGreater(vector<int>& nums) {
    int n = nums.size();
    vector<int> ans(n, -1);
    stack<int> st; // indexes, nums[st] strictly decreasing
    for (int i = 0; i < n; i++) {
        while (!st.empty() && nums[st.top()] < nums[i]) {
            ans[st.top()] = nums[i]; // or i, if the problem wants the index
            st.pop();
        }
        st.push(i);
    }
    return ans;
}
```

Flip the comparison for next smaller. Walk right to left for "previous" greater, or keep this walk and treat the stack itself as the previous-greater structure (the top, after popping smaller items, is the previous greater).

Circular array (next greater in a loop): iterate `i` from `0` to `2n - 1` and use index `i % n`. Push only while `i < n` so each index is pushed once.

### Daily temperatures / stock span

Same loop. `ans[popped] = i - popped`.

### Largest rectangle in histogram

For each bar, the width it can extend is bounded by the nearest shorter bar on each side. Maintain an **increasing** stack of heights. When bar `i` is shorter than the top, the top's right bound is `i` and its left bound is the new top.

```cpp
int largestRectangle(vector<int>& heights) {
    stack<int> st;
    st.push(-1); // sentinel: no bar to the left
    int best = 0;
    heights.push_back(0); // flush the stack at the end
    for (int i = 0; i < (int)heights.size(); i++) {
        while (st.top() != -1 && heights[st.top()] > heights[i]) {
            int height = heights[st.top()];
            st.pop();
            int width = i - st.top() - 1;
            best = max(best, height * width);
        }
        st.push(i);
    }
    heights.pop_back();
    return best;
}
```

Maximal rectangle in a binary matrix: for each row, build a histogram of consecutive 1s ending at that row, then run the histogram routine. `O(rows * cols)`.

### Sliding window maximum — monotonic deque

Need the max of every window of size K. A deque stores indexes in **decreasing** value order. The front is the max of the current window. Evict from the front when it falls out of the window; evict from the back when the new value is larger (those indexes can never be the max again).

```cpp
vector<int> maxSlidingWindow(vector<int>& nums, int k) {
    deque<int> dq; // indexes, values decreasing
    vector<int> ans;
    for (int i = 0; i < (int)nums.size(); i++) {
        while (!dq.empty() && dq.front() <= i - k) dq.pop_front();
        while (!dq.empty() && nums[dq.back()] <= nums[i]) dq.pop_back();
        dq.push_back(i);
        if (i >= k - 1) ans.push_back(nums[dq.front()]);
    }
    return ans;
}
```

**Traps.**

- Storing values instead of indexes. You need the index to compute distance and to know whether it is still inside the window.
- Strict vs non-strict comparison. Duplicates: decide whether the left copy or the right copy should survive, and use `<` or `<=` on purpose.
- Off-by-one width in the histogram: `i - st.top() - 1`, because both bounds are exclusive.

---

## 6. Binary search

**Signal.** A sorted array, or any yes/no question where once the answer becomes "yes" it stays "yes" (or the reverse). The second case is **binary search on the answer**, and the array you search does not have to exist.

**Intuition.** Half the search space is impossible after one probe. That requires monotonicity. If both sides could still hold the answer, you cannot discard either, and this is the wrong tool.

### Lower bound on a sorted array

`lo` ends as the first index where `a[i] >= target`. If every element is smaller, `lo == n`.

```cpp
int lowerBound(vector<int>& a, int target) {
    int lo = 0, hi = a.size(); // search space is [lo, hi)
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (a[mid] < target) lo = mid + 1;
        else hi = mid;
    }
    return lo;
}
```

`upper_bound` (first index with `a[i] > target`) uses `<=` instead of `<`.

Memorize **one** template. The closed variant `while lo <= hi` with `hi = mid - 1` is equivalent and easy to get wrong by one. The half-open form above returns the insertion point directly.

Rotated sorted array: compare `a[mid]` to `a[lo]` (or `a[hi]`) to decide which half is sorted, then check whether `target` sits inside that sorted half.

### Binary search on the answer

Use this when the problem says "minimum possible maximum", "maximum possible minimum", "smallest capacity that still works", "koko eating bananas", "split array largest sum", "ship within D days".

Template:

1. `lo`, `hi` = smallest and largest values the answer could ever be.
2. `feasible(mid)` is monotonic. If capacity `mid` works, every larger capacity works.
3. If `feasible(mid)`, try to go smaller (`hi = mid`), else you need more (`lo = mid + 1`).

```cpp
long long minAnswer(long long lo, long long hi) {
    while (lo < hi) {
        long long mid = lo + (hi - lo) / 2;
        if (feasible(mid)) hi = mid; // feasible is your monotonic predicate
        else lo = mid + 1;
    }
    return lo;
}
```

`feasible` is usually a greedy scan or a sliding window, not another clever algorithm. Example: "can I ship all packages in `days` if the ship holds `mid`?" Walk the array, open a new day whenever the current day would overflow.

**Finding a peak** (a value greater than both neighbors) also discards half: if `a[mid] < a[mid + 1]`, a peak exists on the right, because the slope is still climbing and the array is finite.

**Traps.**

- Searching a non-monotonic predicate. Test `feasible` on a tiny example and confirm the answers flip from no to yes exactly once.
- `mid = (lo + hi) / 2` with `hi = mid` when `lo` and `hi` are adjacent can infinite-loop **if** you also set `lo = mid` on the other branch. In the template above the failing branch does `lo = mid + 1`, so it terminates. Do not "simplify" that `+ 1` away.
- Integer overflow on `(lo + hi)`. Use `lo + (hi - lo) / 2`.

---

## 7. Linked lists

**Signal.** A singly linked list. You cannot index, you cannot walk backward, and you often should not count the length first.

**Intuition.** Keep a `prev` pointer yourself if you need to look back. Use a **dummy head** whenever the real head might be deleted or replaced, so the empty-list and delete-head cases are not special.

```cpp
ListNode* reverse(ListNode* head) {
    ListNode* prev = nullptr;
    ListNode* cur = head;
    while (cur) {
        ListNode* nxt = cur->next;
        cur->next = prev;
        prev = cur;
        cur = nxt;
    }
    return prev;
}
```

Save `nxt` before you overwrite `cur.next`. That is the entire algorithm.

### Fast and slow

- **Cycle:** move `slow` one step, `fast` two. If they meet, there is a cycle. If `fast` hits `nullptr`, there is not.
- **Middle:** same speeds. When `fast` reaches the end, `slow` is at the middle (the second middle if the length is even — check which one the problem wants).
- **Start of the cycle:** after they meet, reset one pointer to `head` and walk both one step at a time. The next meeting point is the entrance. (Floyd: the entrance is equidistant from `head` and from the meeting point, along the cycle.)

### Merge two sorted lists

Dummy head. Each step, attach the smaller node and advance that list. Attach the leftover tail at the end.

### Common recipes

| Problem shape | Recipe |
|---|---|
| remove nth from end | dummy + fast leads by `n + 1`, then move both; slow sits just before the victim |
| palindrome list | find middle, reverse the second half, compare, reverse back if you must restore |
| reorder list | find middle, reverse second half, zip the two halves |
| add two numbers | dummy, carry, build a new list; lists may already be least-significant digit first |
| reverse nodes in k-group | count k, reverse that segment with the plain reverse, stitch `prev_tail` to the new head |
| copy list with random pointer | interleave copied nodes (`A -> A' -> B -> B'`), wire randoms, then unweave. Or a map from old node to new node |

**Traps.**

- Losing the rest of the list by overwriting `.next` before saving it.
- Returning `head` after a reversal or a head deletion. Return `dummy.next` or `prev`.
- Cycle detection that marks nodes by value. Values are not unique. Mark by node identity, or use Floyd when you cannot mutate and cannot use a set.

---

## 8. Trees

**Signal.** Hierarchical data, paths from root to leaf, or a relation between a node and its subtree. If the input is a graph with possible cycles, this section is the wrong one; go to graphs and bring a visited set.

**Intuition.** A tree DFS returns a small summary of the subtree. The parent combines the summaries of its children. Decide up front whether the answer is **returned** upward or **accumulated** in a global.

### The one traversal you rewrite every time

```cpp
int dfs(TreeNode* node) {
    if (!node) return baseCase;
    int left = dfs(node->left);
    int right = dfs(node->right);
    return combine(node, left, right);
}
```

What changes per problem is `base_case` and `combine`:

| Ask | Return upward | Extra |
|---|---|---|
| max depth | `1 + max(left, right)` | base `0` |
| balanced? | height, or `-1` if unbalanced | a subtree is unbalanced if `abs(L-R) > 1` or either child returned `-1` |
| diameter | height | global `max(global, left + right)` — diameter through this node does **not** include the node count twice; edges are `left + right` |
| path sum exists | `bool` | `node.val == target` at a leaf, or either child with `target - node.val` |
| max path sum (any node to any node) | max **downward** path starting here: `node.val + max(0, left, right)` | global also considers `node.val + max(0,left) + max(0,right)`, the path that bends |
| LCA | the node, if this subtree contains `p` or `q` | if both children return non-null, this node is the LCA |
| house robber on a tree | pair `(rob_this, skip_this)` | `rob = node.val + left.skip + right.skip`; `skip = max(left) + max(right)` |

### BST

Inorder is sorted. That single fact generates the rest:

- Validate: pass an allowed `(lo, hi)` range down. A node must sit strictly inside it. Checking only against the parent misses a left grandchild that is larger than the grandparent.
- Kth smallest: inorder, stop at the kth visit. Or augment each node with subtree size if the tree is built once and queried many times.
- Lowest common ancestor in a BST: if both values are smaller, go left; both larger, go right; otherwise this node splits them and is the LCA.
- Build from preorder: the next value inside `(lo, hi)` is the root; recurse left, then right, consuming a shared index.
- Two-sum in a BST: inorder into an array, then opposite-ends. Or a BST iterator (stack of the left spine) from both ends.

### Level order

BFS with a queue. Capture `len(queue)` at the start of each iteration; that many pops are one level. Right-side view is the last node of each level.

### Serialize / deserialize

Preorder with an explicit null marker (`"x"`). Deserialize with a shared index: read a token, if null return `nullptr`, else build the node and fill left then right. The null markers are what make the shape unique.

**Traps.**

- Diameter and max-path-sum: the value you **return** is not the value you **record**. You return a single downward chain, because the parent can only continue one side. You record the bend in a global.
- Using a too-small sentinel as the empty-path base when node values can be negative, then adding to it. Do not drop a negative child with `max(0, child)` unless the problem allows skipping nodes (it does in max path sum; it does not in "path from root to leaf").
- BST validation with only parent comparisons.

---

## 9. Tries

**Signal.** Many prefix queries: autocomplete, "does any word start with", word search on a board where you walk a dictionary, replacing words with roots.

**Intuition.** Share the common prefix so you do not rescan it per word. Each edge is one character. A node is a word-end or it is not; the path from the root is the string.

```cpp
struct TrieNode {
    TrieNode* children[26]{};
    string word; // non-empty means a word ends here
};

class Trie {
    TrieNode root;
    TrieNode* walk(const string& s) {
        TrieNode* node = &root;
        for (char ch : s) {
            int i = ch - 'a';
            if (!node->children[i]) return nullptr;
            node = node->children[i];
        }
        return node;
    }
public:
    void insert(const string& word) {
        TrieNode* node = &root;
        for (char ch : word) {
            int i = ch - 'a';
            if (!node->children[i]) node->children[i] = new TrieNode();
            node = node->children[i];
        }
        node->word = word;
    }
    bool search(const string& word) {
        TrieNode* node = walk(word);
        return node && !node->word.empty();
    }
    bool startsWith(const string& prefix) {
        return walk(prefix) != nullptr;
    }
};
```

**Word search II.** Insert every word. DFS the board, moving through trie children. When `node.word` is set, record it and clear `node.word` so you do not emit duplicates. Mark the cell visited for the duration of the DFS branch, then unmark.

A trie beats a set of words when many words share prefixes or you must stop a board DFS as soon as the current path is not a prefix of anything. If you only check whole words, a hash set is simpler and usually enough.

---

## 10. Heaps

**Signal.** "Kth", "top K", "merge K sorted lists", "rebalance by always taking the current smallest", running median, or Dijkstra.

**Intuition.** You do not need full order. You need repeated access to the current min or max. A heap gives you that in `O(log n)` per update. `priority_queue` is a **max**-heap. For a min-heap, pass `greater<int>` (or push `-value`).

### Top K / Kth largest

Keep a min-heap of size K holding the K largest seen so far. The root is the smallest of those, which is the Kth largest. Anything smaller than the root cannot enter.

```cpp
int kthLargest(vector<int>& nums, int k) {
    priority_queue<int, vector<int>, greater<int>> heap; // min-heap of size k
    for (int i = 0; i < k; i++) heap.push(nums[i]);
    for (int i = k; i < (int)nums.size(); i++) {
        if (nums[i] > heap.top()) {
            heap.pop();
            heap.push(nums[i]);
        }
    }
    return heap.top();
}
```

Pop-then-push is one rebalance. If `k` is tiny and `n` is huge, this is `O(n log k)`, better than sorting.

Quickselect is the average `O(n)` alternative for a one-shot Kth element, with worst case `O(n^2)` unless you randomize the pivot. Prefer it when you need the element once and can mutate the array. Prefer the heap when the stream keeps growing.

### Merge K sorted lists

Push `(value, list_id, node)` for each list head. Pop the smallest, append it to the output, push its successor. `O(N log K)`.

Put a tie-break in the tuple, `(value, listId)`, and keep the node pointer in a side array. `priority_queue` will not order `ListNode*`, and two equal values need a second field or the comparison is unspecified.

### Two heaps — running median

- `small`: max-heap of the lower half (default `priority_queue<int>`)
- `large`: min-heap of the upper half (`priority_queue<int, vector<int>, greater<int>>`)

Keep sizes equal, or `small` one bigger. The median is the top of `small`, or the average of both tops. Rebalance by moving the root across after every insert.

### Other heap tells

| Problem | What the heap stores |
|---|---|
| task scheduler | counts of remaining tasks; always run the most frequent that is not cooling down. Often a heap plus a cooldown queue |
| reorganize string | most frequent char that is not the one just used |
| meeting rooms II | min-heap of end times; if the earliest end is `≤` the new start, reuse that room |
| ugly numbers / super ugly | heap of candidates plus a set so you do not push duplicates |
| k closest points | max-heap of size K by distance, same as Kth largest |

**Traps.**

- A heap is not sorted. Never scan it looking for "the" element other than the root; that scan is a lie about the complexity.
- Forgetting the tie-break field and crashing on equal priorities.
- Using a heap to repeatedly "find min and delete arbitrary index". A heap cannot delete an interior element efficiently unless you add lazy deletion (push updates, ignore stale entries when they reach the root — the Dijkstra pattern).

---

## 11. Backtracking

**Signal.** "Return **all** combinations / permutations / partitions / placements" that satisfy a constraint. Or a yes/no puzzle (sudoku, N-queens) where you must try a placement and undo it.

**Intuition.** DFS over the decision tree. Each call is one partial solution. You append a choice, recurse, then pop it (**undo**) so the next choice sees a clean state. Prune the moment the partial solution can never be completed.

```cpp
void backtrack(int start, vector<int>& path) {
    if (isSolution(path)) {
        res.push_back(path); // copies; path will be mutated
        return;
    }
    for (int i = start; i < n; i++) {
        if (!canTake(i, path)) continue;
        path.push_back(choices[i]);
        backtrack(nextStart(i), path);
        path.pop_back();
    }
}
```

The only real difference between the classic problems is `start` and whether you skip duplicates:

| Problem | Loop start | Next start | Extra state |
|---|---|---|---|
| subsets | `start` | `i + 1` | none |
| subsets with duplicates | sort, `start` | `i + 1` | skip `i > start and a[i] == a[i-1]` |
| combinations of K | `start` | `i + 1` | stop when `len(path) == k` |
| combination sum (reuse allowed) | `start` | `i` (same index) | stop when sum exceeds target |
| combination sum (no reuse) | `start` | `i + 1` | same |
| permutations | `0` | `0` | a `used[]` array, or swap `path[start]` with `path[i]` |
| permutations with duplicates | sort, `0` | `0` | skip a duplicate if the previous identical value is not used |
| palindrome partition | `start` is the cut | `i + 1` where `s[start:i+1]` is a palindrome | the path is a list of pieces |
| letter combinations of a phone number | index in the digits | `index + 1` | the loop is over the letters of this digit, not over a shared array |
| generate parentheses | none | none | two counters, `open` and `close`; push `(` if `open < n`, push `)` if `close < open` |
| N-queens | row | `row + 1` | columns, diagonals, anti-diagonals as sets. Diagonal key is `row - col`, anti is `row + col` |
| sudoku | next empty cell | that cell | row/col/box sets; box index is `(r // 3) * 3 + c // 3` |

**Choose / not-choose form** (subsets only) is equivalent and sometimes cleaner:

```cpp
void subsets(int i, vector<int>& path) {
    if (i == n) {
        res.push_back(path);
        return;
    }
    subsets(i + 1, path);          // skip nums[i]
    path.push_back(nums[i]);
    subsets(i + 1, path);          // take nums[i]
    path.pop_back();
}
```

**Word search (one word).** DFS from each cell. Mark the board cell as visited (or with a sentinel) before the recursive calls and restore it after. This is backtracking on a grid; the trie version in section 9 is the same idea against many words.

**Traps.**

- Appending `path` without copying. Every answer then aliases the same list and ends up empty or identical.
- Reuse: combination-sum "unlimited coins" passes `i`, not `i + 1`. That single argument is the whole difference.
- Pruning too late. If the remaining sum is already negative or the remaining length cannot reach K, return before the loop.
- Sorting is part of the duplicate-skip invariant. The `a[i] == a[i-1]` check is meaningless on unsorted input.

---

## 12. Graphs — traversal

**Signal.** Cells, rooms, islands, "spread in 4 directions", shortest path when every edge has the same cost, or a clone of a graph.

**Intuition.** A graph DFS/BFS is a tree DFS/BFS plus a **visited set**, because there are cycles and multiple parents. Mark a node when you **push** it, not when you pop it, or the queue fills with duplicates.

```cpp
const int DIRS[4][2] = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};

int gridBfs(vector<vector<int>>& grid, vector<pair<int, int>>& sources) {
    int rows = grid.size(), cols = grid[0].size();
    queue<pair<int, int>> q;
    vector<vector<int>> seen(rows, vector<int>(cols));
    for (auto [r, c] : sources) {
        q.push({r, c});
        seen[r][c] = 1;
    }
    const int BLOCKED = 1; // whatever the problem treats as a wall
    int dist = 0;
    while (!q.empty()) {
        int layer = q.size();
        while (layer--) {
            auto [r, c] = q.front();
            q.pop();
            // process (r, c) at this dist
            for (auto& d : DIRS) {
                int nr = r + d[0], nc = c + d[1];
                if (nr < 0 || nc < 0 || nr >= rows || nc >= cols) continue;
                if (seen[nr][nc] || grid[nr][nc] == BLOCKED) continue;
                seen[nr][nc] = 1;
                q.push({nr, nc});
            }
        }
        dist++;
    }
    return dist;
}
```

### Which traversal

| Ask | Use |
|---|---|
| number of islands, flood fill, clone graph, detect cycle in undirected (parent check) | DFS or BFS, either is fine |
| shortest path, unweighted, or "minimum steps" | BFS, first time you reach the target |
| multi-source ("rotting oranges", "01 matrix", walls and gates) | push **all** sources at dist 0, then one BFS |
| 0-1 weights (edge is 0 or 1) | deque: appendleft on a 0-edge, append on a 1-edge |
| cheapest path, weights `> 0` | Dijkstra, section 13 |
| visit in dependency order | topological sort, section 13 |

**Implicit graphs.** Sometimes nobody hands you an adjacency list. The nodes are states: lock combinations (`0000` → 4 dials × 2 directions), word ladder (words one letter apart — build the adjacency via generic patterns `h*t`), or a board. Name the state, name the edge, then run ordinary BFS. The distance is the number of edges.

**Cycle in a directed graph.** Three colors. White = unseen, gray = on the current DFS stack, black = finished. An edge to a gray node is a back edge, which means a cycle. An edge to a black node is a cross edge and is fine.

**Traps.**

- Marking visited on pop. Two paths enqueue the same cell and you do exponential work.
- Forgetting that BFS distance is only shortest when edges are equal (or you use 0-1 BFS / Dijkstra).
- Mutating the input grid as your visited marks. That is legal and fast, but only if the caller does not need the grid intact, and only if the blocked value and the "empty" value are distinct from your mark.
- Island count: a BFS/DFS that forgets to iterate every cell as a potential start. The traversal only covers one component.

---

## 13. Graphs — structure

### Topological sort (Kahn)

**Signal.** Prerequisites, "order of courses", alien dictionary, "can you finish". Directed, and a cycle means impossible.

**Intuition.** A node with indegree 0 can be taken now. Take it, reduce its neighbors' indegrees, repeat. If you cannot take everyone, something is stuck in a cycle.

```cpp
vector<int> topo(int n, vector<pair<int, int>>& edges) {
    vector<vector<int>> g(n);
    vector<int> indeg(n);
    for (auto [u, v] : edges) { // u must come before v
        g[u].push_back(v);
        indeg[v]++;
    }
    queue<int> q;
    for (int i = 0; i < n; i++)
        if (indeg[i] == 0) q.push(i);
    vector<int> order;
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        order.push_back(u);
        for (int v : g[u])
            if (--indeg[v] == 0) q.push(v);
    }
    if ((int)order.size() != n) return {}; // empty means a cycle
    return order;
}
```

Build the graph in the direction "earlier → later". Reversing an edge reverses the answer and is the most common bug on these problems.

Alien dictionary: consecutive words, find the first position they differ, that is an edge `word1[i] → word2[i]`. If `word1` is longer and `word2` is a prefix of it, the order is illegal. Then topo-sort the letters that appeared.

### Union-find (disjoint set union)

**Signal.** "Are these two in the same component?", "how many components?", "redundant connection", "accounts merge", dynamic connectivity, Kruskal. Edges are **undirected** and you do not need the path, only the component.

**Intuition.** Each component has a root. `find` walks to the root and compresses the path. `union` links one root under the other. If `union` is asked to join two nodes that already share a root, that edge is redundant (it closes a cycle).

```cpp
struct DSU {
    vector<int> parent, rank;
    int components;
    DSU(int n) : parent(n), rank(n), components(n) {
        iota(parent.begin(), parent.end(), 0);
    }
    int find(int x) {
        while (parent[x] != x) {
            parent[x] = parent[parent[x]];
            x = parent[x];
        }
        return x;
    }
    bool unite(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return false;
        if (rank[ra] < rank[rb]) swap(ra, rb);
        parent[rb] = ra;
        if (rank[ra] == rank[rb]) rank[ra]++;
        components--;
        return true;
    }
};
```

Number of provinces = `components` after unioning every edge. Earliest moment the network becomes connected = the first time `components == 1` while scanning edges in time order. A grid's islands can also be counted this way; BFS is usually simpler.

### Dijkstra

**Signal.** Shortest path, edge weights are **non-negative**, and you may need the actual distance rather than the hop count.

```cpp
vector<long long> dijkstra(int n, vector<vector<pair<int, int>>>& g, int src) {
    const long long INF = 1e18;
    vector<long long> dist(n, INF);
    dist[src] = 0;
    priority_queue<pair<long long, int>, vector<pair<long long, int>>, greater<>> pq;
    pq.push({0, src});
    while (!pq.empty()) {
        auto [d, u] = pq.top();
        pq.pop();
        if (d != dist[u]) continue; // stale entry
        for (auto [v, w] : g[u]) {
            if (d + w < dist[v]) {
                dist[v] = d + w;
                pq.push({dist[v], v});
            }
        }
    }
    return dist;
}
```

The stale-entry check is lazy deletion. You may push several distances for `u`; only the one equal to `dist[u]` is real.

**Path reconstruction:** also store `parent[v] = u` when you relax. Walk parents from the target back to the source, then reverse.

**When not Dijkstra.**

- Unweighted → BFS.
- Weights 0 or 1 → 0-1 BFS.
- Negative edges, no negative cycle → Bellman-Ford, `O(VE)`, relax every edge `V-1` times. One more successful relaxation means a negative cycle.
- All-pairs, small `V` (`≤ 400`) → Floyd-Warshall, the triple loop `for k, for i, for j: dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])`. `k` is the outermost loop. That is the whole algorithm.

### Minimum spanning tree

Kruskal: sort edges by weight, add an edge if `union` says its ends are in different components. Stop at `n - 1` edges. Prim: Dijkstra-like, but the heap key is the edge weight into the growing tree, not the path cost from the source.

### Bipartite

Color the graph with two colors via BFS or DFS. If a neighbor already has your color, it is not bipartite. Useful for "possible bipartition" and for some assignment problems.

---

## 14. Intervals

**Signal.** Ranges `[start, end]`. Merge, insert, how many rooms, erase the minimum to kill overlaps, "can one person attend".

**Intuition.** Sort. After sorting by start (or by end — the choice depends on the question), overlaps are adjacent and a single scan settles them.

### Merge

Sort by start. If the next interval starts before the current one ends, extend the end to the max. Otherwise start a new group.

```cpp
vector<vector<int>> mergeIntervals(vector<vector<int>>& intervals) {
    if (intervals.empty()) return {};
    sort(intervals.begin(), intervals.end());
    vector<vector<int>> merged{intervals[0]};
    for (int i = 1; i < (int)intervals.size(); i++) {
        if (intervals[i][0] <= merged.back()[1])
            merged.back()[1] = max(merged.back()[1], intervals[i][1]);
        else
            merged.push_back(intervals[i]);
    }
    return merged;
}
```

Insert a new interval: walk, copy everything that ends before the new one starts, merge everything that overlaps it, copy the rest.

### Meeting rooms

- **Can one person attend all?** Sort by start. If any meeting starts before the previous one ends, no.
- **Minimum rooms.** Sort by start. Min-heap of end times. For each meeting, if the earliest ending meeting is done (`pq.top() <= start`), pop it and reuse the room. Push this meeting's end. The heap size is the room count. A sweep line (start is `+1`, end is `-1`, sort events, end before start at the same timestamp if a room frees in time) answers the same question.

### Remove the minimum intervals to make the rest non-overlapping

Sort by **end**, not start. Take a meeting if it starts at or after the last taken end. This is the classic "earliest finish frees the resource soonest" greedy, and it is optimal.

```cpp
int eraseOverlap(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end(), [](auto& a, auto& b) {
        return a[1] < b[1];
    });
    int keptEnd = INT_MIN, removed = 0;
    for (auto& iv : intervals) {
        if (iv[0] >= keptEnd) keptEnd = iv[1]; // use > if touching counts as overlap
        else removed++;
    }
    return removed;
}
```

Confirm whether `[1,2]` and `[2,3]` overlap. The problem statement's equality decides `<=` vs `<`. Half of interval bugs are this comparison.

---

## 15. Greedy

**Signal.** You build the answer left to right, and a local choice (take the earliest end, jump as far as you can see, give the extra cookie to the greediest child you can still satisfy) never needs to be undone.

**Intuition.** Greedy is legal only when a swap argument holds: if an optimal solution disagrees with your choice at the first difference, you can swap in your choice without making it worse. If you cannot tell that story, do not be greedy — write DP.

You do not need to recite the proof in an interview, but you do need to know the canonical shapes, because they are the ones where the swap argument is already known to work.

| Shape | Rule |
|---|---|
| activity selection / non-overlapping intervals | sort by finish, take if it fits |
| jump game II (min jumps) | scan the current jump's range, record the farthest you can reach, jump when you hit the end of the range |
| gas station | if total gas `<` total cost, impossible. Otherwise the unique start is just after the lowest prefix of `gas - cost` |
| assign cookies | sort both, satisfy the smallest child you can with the smallest fitting cookie |
| partition labels | last index of each char; extend the current partition to cover every char inside it |
| hand of straights | sort, map of counts, always start a group at the smallest remaining number |
| candy | two passes: left to right for the ascending constraint, right to left for the descending one, take max |
| Huffman-style merge files | always merge the two cheapest (a heap) |

**Jump game (can you reach the end?)** is greedy: track the farthest index reachable. If `i` ever passes that farthest, you are stuck. The DP version (`dp[i] = can I land here`) is correct and slower. When both exist and `n` is `10^5`, take the greedy.

**When greedy fails.** Coin systems that are not canonical (coins `[1,3,4]`, amount `6`: greedy takes `4+1+1`, optimal is `3+3`). Any "count the ways" question. Any split where today's cheap choice blocks a much better pair tomorrow and you cannot state a swap argument — that is knapsack, which is DP.

---

## 16. 1D dynamic programming

**Signal.** An answer about a prefix, a suffix, or "the first `i` items". The same sub-question (`best using the first i`, `can I make sum s`) is asked many times. A choice at `i` depends only on answers at smaller indexes.

**Intuition.** Do not recompute a subproblem. Name it, store it, and make sure when you need `dp[j]` it is already final. One dimension means the subproblem has **one** changing parameter: an index, a remaining amount, or a count.

### The contract, written before any code

Say these five lines or you are guessing:

1. **State.** `dp[i]` = …
2. **Choice.** At this state I may …
3. **Recurrence.** `dp[i]` = combine(those choices)
4. **Base.** `dp[0]` or the empty prefix = …
5. **Answer.** `dp[n]`, or `max(dp)`, or `any(dp)`

If you cannot finish line 1, you do not have a DP yet. Forcing a table anyway produces nonsense transitions.

Direction of the recurrence decides the loop. If `dp[i]` reads smaller indexes, loop `i` upward. If it reads larger indexes, loop downward.

### A — linear recurrence

Climbing stairs, fibonacci, min cost climbing stairs, decode ways.

`dp[i]` depends on a fixed handful of earlier cells, usually `i-1` and `i-2`.

```cpp
int climbStairs(int n) {
    int prev2 = 1, prev1 = 1; // ways to stand on step 0 and step 1
    for (int i = 2; i <= n; i++) {
        int cur = prev1 + prev2;
        prev2 = prev1;
        prev1 = cur;
    }
    return prev1;
}
```

**Decode ways.** `dp[i]` = ways to decode `s[:i]`.

- always: if `s[i-1] != '0'`, add `dp[i-1]` (take one digit)
- if `10 <= int(s[i-2:i]) <= 26`, add `dp[i-2]` (take two digits)

A leading zero inside a chunk kills that chunk. `"06"` is not a code. `"10"` is only the two-digit take, because the one-digit take would leave a `0`.

### B — take or skip on a line

House robber: you cannot take neighbors.

`dp[i]` = best loot from the first `i` houses = `max(skip this, take this)`.

```cpp
int rob(vector<int>& nums) {
    int skip = 0, take = 0;
    for (int x : nums) {
        int nextTake = skip + x;
        skip = max(skip, take);
        take = nextTake;
    }
    return max(skip, take);
}
```

**Circular houses** (first and last are neighbors): the answer is the max of `rob` on `nums` without the first house and `rob` on `nums` without the last. Run the linear routine twice. Do not invent a third state.

**Delete and earn:** houses whose values are equal are not adjacent indexes, they are adjacent *values*. Bucket the points by value (`points[v] += v`) and rob that array. Missing values are houses of `0`.

### C — best subarray ending here (Kadane)

Maximum subarray, maximum product subarray.

`dp[i]` = best sum of a subarray that **ends at i**. Either extend the previous one or start fresh.

```cpp
int maxSubarray(vector<int>& nums) {
    int best = nums[0], cur = nums[0];
    for (int i = 1; i < (int)nums.size(); i++) {
        cur = max(nums[i], cur + nums[i]);
        best = max(best, cur);
    }
    return best;
}
```

Only the running `cur` is required. The answer is the best `cur` ever seen, not `cur` at the end.

**Maximum product** needs two running values, because a negative times a negative flips: track `max_end` and `min_end`. A new negative swaps their roles.

```cpp
int maxProduct(vector<int>& nums) {
    int best = nums[0], hi = nums[0], lo = nums[0];
    for (int i = 1; i < (int)nums.size(); i++) {
        int x = nums[i];
        if (x < 0) swap(hi, lo);
        hi = max(x, hi * x);
        lo = min(x, lo * x);
        best = max(best, hi);
    }
    return best;
}
```

### D — amount / dictionary on one axis

Coin change, coin change II, perfect squares, word break, combination sum IV.

The index is a **sum** or a **string length**, not an element. `dp[s]` = best way to build exactly `s`.

**Minimum coins** (each coin unlimited):

```cpp
int coinChange(vector<int>& coins, int amount) {
    const int INF = amount + 1;
    vector<int> dp(amount + 1, INF);
    dp[0] = 0;
    for (int a = 1; a <= amount; a++)
        for (int c : coins)
            if (c <= a) dp[a] = min(dp[a], dp[a - c] + 1);
    return dp[amount] == INF ? -1 : dp[amount];
}
```

**The loop-order switch** is the bug factory. It decides whether order matters.

| Ask | Outer loop | Inner loop | Why |
|---|---|---|---|
| min coins, unlimited | either | either | min does not count duplicates |
| number of **combinations** (order ignored), Coin Change II | coin | amount, forward | each coin extends sums already built from earlier coins, so `{1,2}` is not also counted as `{2,1}` |
| number of **permutations** (order matters), Combination Sum IV | amount | coin | every order of reaching `a` is a different sequence |

```cpp
int coinCombinations(vector<int>& coins, int amount) {
    vector<int> dp(amount + 1);
    dp[0] = 1;
    for (int c : coins)
        for (int a = c; a <= amount; a++)
            dp[a] += dp[a - c];
    return dp[amount];
}

int coinPermutations(vector<int>& coins, int amount) {
    vector<int> dp(amount + 1);
    dp[0] = 1;
    for (int a = 1; a <= amount; a++)
        for (int c : coins)
            if (c <= a) dp[a] += dp[a - c];
    return dp[amount];
}
```

**Word break.** `dp[i]` is true if `s[:i]` can be segmented.

```cpp
bool wordBreak(string s, vector<string>& wordDict) {
    unordered_set<string> words(wordDict.begin(), wordDict.end());
    int n = s.size();
    vector<int> dp(n + 1);
    dp[0] = 1;
    for (int i = 1; i <= n; i++) {
        for (int j = 0; j < i; j++) {
            if (dp[j] && words.count(s.substr(j, i - j))) {
                dp[i] = 1;
                break;
            }
        }
    }
    return dp[n];
}
```

If the max word length is `L`, start `j` at `max(0, i - L)` instead of `0`.

### E — longest increasing subsequence

`O(n^2)`: `dp[i]` = length of the best LIS **ending at i**. For every `j < i` with `nums[j] < nums[i]`, `dp[i] = max(dp[i], dp[j] + 1)`.

`O(n log n)`, patience sorting: `tails[len]` is the smallest tail of any increasing subsequence of that length. A new number replaces the first tail it can extend (lower bound), or appends.

```cpp
int lengthOfLIS(vector<int>& nums) {
    vector<int> tails;
    for (int x : nums) {
        auto it = lower_bound(tails.begin(), tails.end(), x); // upper_bound if non-decreasing
        if (it == tails.end()) tails.push_back(x);
        else *it = x;
    }
    return (int)tails.size();
}
```

`tails` is **not** the subsequence. It is only good for the length. To rebuild one subsequence you need the `O(n^2)` parent pointers, or extra bookkeeping.

Russian doll envelopes and "maximum height by stacking cuboids" are LIS after a sort: sort by one dimension ascending and the other descending, so two items that tie on the first dimension cannot both be taken by the LIS.

### F — stock state machine

Prices over days, with a constraint on how you may trade. The state is where you are in the transaction, not a sum.

One pass, a few rolling variables:

| Version | States to carry |
|---|---|
| one transaction | `min_price`, `best` profit |
| unlimited | add every ascent: `if p[i] > p[i-1]: profit += p[i] - p[i-1]`. Holding across a valley is never better |
| cooldown (must skip a day after selling) | `hold`, `sold`, `rest`. `hold = max(hold, rest - price)`, `sold = hold + price`, `rest = max(rest, sold_yesterday)` |
| at most K transactions | `buy[t]`, `sell[t]` for `t` in `1..k`. `buy[t] = max(buy[t], sell[t-1] - price)`, `sell[t] = max(sell[t], buy[t] + price)` |
| fee | same as unlimited, but selling pays `fee` |

Initialize `buy` states to `-prices[0]` (or `-inf` for transactions you cannot have opened yet) and `sell` states to `0`.

If `k >= n // 2`, "at most K" is the same as unlimited — take the fast path, because the `O(nk)` loop is unnecessary and `k` can be `10^9`.

### G — reachability on a line

Jump game as DP: `dp[i]` = can I reach `i`. True if some earlier `j` with `dp[j]` and `j + nums[j] >= i`. This is correct. The greedy farthest-reach scan in section 15 is the same fact, tightened. Use DP when the reach rule is weirder than "I can land anywhere in range" (minimum jumps with a cost, odd/even jump to a bigger/smaller value).

**Odd-even jump:** from `i`, an odd jump goes to the smallest `nums[j] >= nums[i]` with `j > i`; an even jump goes to the largest `nums[j] <= nums[i]`. Precompute those two next-indexes with a sorted map of value → index, scanned right to left, then `dp[i][parity]` is a boolean.

### 1D checklist when you are stuck

- "Number of ways" and your recurrence uses `min` — you answered a different question.
- `dp[0] = 0` on a counting problem. The empty way to make 0 is **one** way, so `dp[0] = 1`. On a min-cost problem, `dp[0] = 0` and everything else starts at infinity.
- Updating a cell from a value that the current iteration already changed. That is the 0/1 vs unbounded distinction. On a pure 1D prefix it usually means your loop runs the wrong way.
- Negative numbers inside Kadane: starting fresh (`max(x, cur + x)`) already handles them. Do not special-case by skipping negatives; a negative can be a bridge to a larger positive in the **product** variant only if you kept `min_end`.
- Index meaning. Decide "`dp[i]` talks about `nums[i]`" or "`dp[i]` talks about the first `i` items (`nums[i-1]`)". Mixing them is an off-by-one, not a logic error, and it is the usual reason a recurrence "almost" works.

---

## 17. 2D DP and knapsack

**Signal.** Two sequences (LCS, edit distance), a grid walk, or a capacity plus an item index. The state needs **two** parameters, so the table is a grid and the answer is usually the far corner.

Same five-line contract as 1D. The new failure mode is filling a cell from a neighbor that is not final yet.

### Grid paths

Unique paths, minimum path sum, dungeon, cherry pickup (harder: two people at once, so the state grows).

```cpp
int minPathSum(vector<vector<int>>& grid) {
    int rows = grid.size(), cols = grid[0].size();
    vector<int> dp(cols);
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (r == 0 && c == 0) dp[c] = grid[0][0];
            else if (r == 0) dp[c] = dp[c - 1] + grid[r][c];
            else if (c == 0) dp[c] = dp[c] + grid[r][c];
            else dp[c] = min(dp[c], dp[c - 1]) + grid[r][c];
        }
    }
    return dp.back();
}
```

Walking only right and down is why each cell has two parents. If you can walk up or left too, this table has cycles and you want Dijkstra or BFS instead of DP.

**Paths with obstacles:** an obstacle cell is `0` ways (or `inf` cost) and is not computed from its parents.

### 0/1 knapsack vs unbounded knapsack

Items have weight and value, capacity is `W`. `dp[cap]` = best value using exactly that capacity (or at most — same table if values are non-negative, because a smaller load is still available inside a larger cap).

**Each item at most once.** Iterate capacity **downward**, so the update reads capacities that have not yet seen this item.

```cpp
int knapsack01(vector<pair<int, int>>& items, int W) { // (weight, value)
    vector<int> dp(W + 1);
    for (auto [weight, value] : items)
        for (int cap = W; cap >= weight; cap--)
            dp[cap] = max(dp[cap], dp[cap - weight] + value);
    return dp[W];
}
```

**Each item unlimited times.** Iterate capacity **upward**, so this item can build on a capacity that already used it.

```cpp
int knapsackUnbounded(vector<pair<int, int>>& items, int W) {
    vector<int> dp(W + 1);
    for (auto [weight, value] : items)
        for (int cap = weight; cap <= W; cap++)
            dp[cap] = max(dp[cap], dp[cap - weight] + value);
    return dp[W];
}
```

Target sum, partition equal subset sum: this is 0/1 knapsack where value and weight are the number itself, and you ask whether capacity `total // 2` is reachable. Use a boolean array (or a bitset). If `total` is odd, stop.

Ones and zeroes (strings of 0s and 1s, at most `m` zeros and `n` ones): two capacities. Loop both dimensions downward.

```cpp
for (auto [zeros, ones] : items)
    for (int z = m; z >= zeros; z--)
        for (int o = n; o >= ones; o--)
            dp[z][o] = max(dp[z][o], dp[z - zeros][o - ones] + 1);
```

### Longest common subsequence

`dp[i][j]` = LCS of `a[:i]` and `b[:j]`.

```cpp
int lcs(string& a, string& b) {
    int na = a.size(), nb = b.size();
    vector<vector<int>> dp(na + 1, vector<int>(nb + 1));
    for (int i = 1; i <= na; i++) {
        for (int j = 1; j <= nb; j++) {
            if (a[i - 1] == b[j - 1]) dp[i][j] = dp[i - 1][j - 1] + 1;
            else dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
        }
    }
    return dp[na][nb];
}
```

Equal characters **consume both** (diagonal). Unequal characters discard exactly one side. Longest common substring is different: on a mismatch reset to 0, and the answer is the max cell anywhere, because the match must be contiguous and need not end at the ends of the strings.

Edit distance adds the third option:

```cpp
int editDistance(string& a, string& b) {
    int nb = b.size();
    vector<int> prev(nb + 1), cur(nb + 1);
    iota(prev.begin(), prev.end(), 0); // dp[0][j] = j insertions
    for (int i = 1; i <= (int)a.size(); i++) {
        cur[0] = i;
        for (int j = 1; j <= nb; j++) {
            if (a[i - 1] == b[j - 1]) cur[j] = prev[j - 1];
            else cur[j] = 1 + min({prev[j], cur[j - 1], prev[j - 1]}); // delete, insert, replace
        }
        swap(prev, cur);
    }
    return prev[nb];
}
```

Distinct subsequences, interleaving strings, regular expression matching, wildcard matching: same grid. The recurrence is "if this char matches, take the diagonal; also, `*` or `.` fans out". Write the match rule on paper for `*` before coding it — `*` consumes the previous token zero or more times, which is "stay in this row/column", not "skip one char".

### Longest palindromic subsequence / substring

Subsequence: it is LCS of `s` with `s` reversed. Or `dp[i][j]` on the range, filled by length.

Substring (contiguous): expand around centers. There are `2n - 1` centers (each index, and each gap between indexes). Expand while the ends match. This is `O(n^2)` time, `O(1)` extra memory, and simpler than the DP table. Use it unless you need every palindromic substring's count, in which case the boolean table `dp[i][j] = s[i] == s[j] and (j - i < 2 or dp[i+1][j-1])`, filled by increasing length, is the boilerplate.

---

## 18. Interval DP, tree DP, bitmask DP

These are still DP. The state is no longer "a prefix".

### Interval DP

**Signal.** Burst balloons, matrix chain multiplication, minimum cost to merge stones, optimal BST, "best way to parenthesize `i..j`".

**Intuition.** The subproblem is a **span**, not a prefix. A span's answer tries every place it can be split, and the two sides are smaller spans. If you fill by start index you will read spans that are not ready. Fill by **length**.

```cpp
int intervalDp(int n) {
    vector<vector<int>> dp(n, vector<int>(n));
    for (int length = 2; length <= n; length++) { // or 1, depending on the base
        for (int i = 0; i + length - 1 < n; i++) {
            int j = i + length - 1;
            dp[i][j] = INT_MAX;
            for (int k = i; k < j; k++)
                dp[i][j] = min(dp[i][j], dp[i][k] + dp[k + 1][j] + cost(i, k, j));
        }
    }
    return dp[0][n - 1];
}
```

Burst balloons specifically: `dp[i][j]` is the best score bursting everything **strictly between** `i` and `j`, with `i` and `j` still alive (they are the walls). Add sentinels `nums = [1] + nums + [1]`. Last balloon burst inside `(i, j)` is `k`, worth `nums[i] * nums[k] * nums[j]`, plus the two open intervals.

```cpp
int burst(vector<int>& nums) {
    vector<int> a{1};
    a.insert(a.end(), nums.begin(), nums.end());
    a.push_back(1);
    int n = a.size();
    vector<vector<int>> dp(n, vector<int>(n));
    for (int length = 2; length < n; length++) {
        for (int i = 0; i + length < n; i++) {
            int j = i + length;
            for (int k = i + 1; k < j; k++)
                dp[i][j] = max(dp[i][j], dp[i][k] + dp[k][j] + a[i] * a[k] * a[j]);
        }
    }
    return dp[0][n - 1];
}
```

`k` is the **last** balloon you burst in the open interval, not the first. That is why the walls `a[i]` and `a[j]` are still there to multiply. Getting this backwards is the whole difficulty of the problem.

### Tree DP

**Signal.** Best independent set on a tree, diameter (already in section 8), "longest path with a constraint downward".

**Intuition.** The subtree return value is a **tuple**, one slot per choice the parent still cares about. House robber III returns `(rob_this, skip_this)`. Binary tree cameras often returns three states: this node has a camera, this node is covered, this node needs its parent to cover it.

Compute children first (postorder). Combine. There is no table; the call stack is the table.

### Bitmask DP

**Signal.** `n ≤ 20`, and a subset of items is part of the state. Traveling salesman, assign workers to jobs, shortest path visiting every node, "can these subsets each form a square side".

**Intuition.** A mask is a subset. `mask | (1 << j)` adds `j`. `mask & (1 << j)` tests `j`. Iterate masks in numeric order and every submask is already done, **if** adding a bit only moves you to a larger mask. If the transition goes to a smaller mask, loop downward.

```cpp
long long tsp(vector<vector<int>>& dist) {
    int n = dist.size();
    const long long INF = 1e18;
    int FULL = (1 << n) - 1;
    vector<vector<long long>> dp(1 << n, vector<long long>(n, INF));
    dp[1][0] = 0; // start at city 0, only city 0 visited
    for (int mask = 0; mask < (1 << n); mask++) {
        for (int u = 0; u < n; u++) {
            if (dp[mask][u] == INF || (mask & (1 << u)) == 0) continue;
            for (int v = 0; v < n; v++) {
                if (mask & (1 << v)) continue;
                int nxt = mask | (1 << v);
                dp[nxt][v] = min(dp[nxt][v], dp[mask][u] + dist[u][v]);
            }
        }
    }
    long long ans = INF;
    for (int u = 0; u < n; u++) ans = min(ans, dp[FULL][u] + dist[u][0]);
    return ans;
}
```

`O(2^n * n^2)`. Fine at `n = 16`, dead at `n = 25`. If `n` is 40, this is the wrong section — look at meet-in-the-middle (split into halves of 20, enumerate each half, sort one side, binary search the complement).

**Submask enumeration** when you need every subset of a fixed mask:

```cpp
for (int sub = mask; sub; sub = (sub - 1) & mask) {
    // use sub
}
```

---

## 19. Bit manipulation

**Signal.** The problem mentions XOR, "every element appears twice except one", subsets of a small set, or "O(1) memory" on integers.

**Intuition.** XOR is addition without carry. `a ^ a = 0`, `a ^ 0 = a`, and it is commutative, so pairs cancel.

| Task | Idiom |
|---|---|
| single number, all others twice | XOR everything |
| single number, all others three times | per-bit count mod 3, or the two-variable FSM `ones`, `twos` |
| two single numbers, rest twice | XOR all, the lowest set bit (`xor & -xor`) splits them into two groups, XOR inside each group |
| count a bit | `n & 1`, then `n >>= 1` |
| drop the lowest set bit | `n & (n - 1)` |
| lowest set bit only | `n & -n` |
| is power of two | `n > 0 and n & (n - 1) == 0` |
| all subsets of a mask / of `n` items | `for mask in range(1 << n)` |
| iterate bits of a mask | `while mask: bit = mask & -mask; mask -= bit` |
| reverse bits | build an answer, 32 times: `ans = (ans << 1) \| (n & 1); n >>= 1` |

Missing number in `0..n`: XOR every index with every value, and XOR `n` in at the end. The paired ones cancel. (Sum also works; XOR does not overflow.)

**Bit DP counting** (count bits from `0` to `n`, total hamming distance): do not loop every number if `n` is `10^9`. Each bit position contributes `count_of_numbers_with_this_bit_set * (1 << b)`. Group the range into complete blocks of size `2^(b+1)`.

---

## 20. Strings

Most string problems are already covered: sliding window (section 4), trie (section 9), two pointers for palindromes, DP for edit distance and LCS (section 17). What remains is matching and small string recipes.

### Expand around center

Longest palindromic substring. For each center, walk outward. Handles both odd and even lengths. `O(n^2)`, and it is the right default.

### Anagram windows

"Find all anagrams of `p` in `s`": fixed window of `len(p)`, a frequency delta array of size 26, and a `need` counter of how many chars are currently satisfied. Update the counter by `±1` only when a char's count crosses the target. Do not compare the whole frequency array on every index.

### Repeated pattern

`s` is made of a repeated substring if `s` appears in `(s + s)` with the first and last characters removed: `(s + s).substr(1, s.size() * 2 - 2)`. The KMP failure function explains why; you do not have to implement KMP to use this check.

### When to actually write KMP

You search for `pattern` inside many texts, or you need the longest prefix that is also a suffix (shortest palindrome, repeated substring pattern from scratch).

```cpp
vector<int> prefixFunction(const string& s) {
    int n = s.size();
    vector<int> pi(n);
    for (int i = 1; i < n; i++) {
        int j = pi[i - 1];
        while (j && s[i] != s[j]) j = pi[j - 1];
        if (s[i] == s[j]) j++;
        pi[i] = j;
    }
    return pi;
}
```

`pi[i]` is the longest proper prefix of `s[:i+1]` that is also a suffix. To search, run the same automaton over the text and count every time `j` reaches `len(pattern)`.

Rabin-Karp (rolling hash) is the alternative when you have many patterns or you want a simple `O(n)` average search. Use a large mod (or double hash) so collisions do not become wrong answers.

---

## 21. Sorting, selection, and simulation

**Signal.** The problem is really "put each item where it belongs" or "produce the next configuration", and a comparison sort is either the whole solution or the setup for two pointers / intervals / greedy.

| Shape | Move |
|---|---|
| sort colors, 0/1/2 | Dutch flag, section 3 |
| Kth element once | quickselect, average `O(n)` |
| merge K sorted | heap, section 10 |
| count inversions | merge sort, count the cross pairs while merging |
| largest number (arrange nums to form it) | sort with comparator `(a, b) -> a+b > b+a` |
| custom order ("alien sorted" check, not rebuild) | map char to rank, then ordinary compare |
| next permutation | find the longest non-increasing suffix, swap the pivot with the next-larger suffix element, reverse the suffix |
| spiral matrix, rotate image, set matrix zeroes | simulation with four bounds, or transpose + reverse rows. Mark zeros in the first row/column as the flag so you use `O(1)` memory |

**Quickselect:**

```cpp
int kthLargestSelect(vector<int>& nums, int k) {
    int target = (int)nums.size() - k; // index in sorted order
    function<int(int, int)> select = [&](int lo, int hi) -> int {
        int pivot = nums[lo + rand() % (hi - lo + 1)];
        int i = lo, j = lo, m = hi;
        while (j <= m) { // Dutch-flag partition around pivot
            if (nums[j] < pivot) swap(nums[i++], nums[j++]);
            else if (nums[j] > pivot) swap(nums[j], nums[m--]);
            else j++;
        }
        if (target < i) return select(lo, i - 1);
        if (target > m) return select(m + 1, hi);
        return nums[target];
    };
    return select(0, (int)nums.size() - 1);
}
```

**Counting sort** when the value range is small (heights `1..100`, colors, "sort by frequency" buckets). Bucket the items, then walk the buckets. `O(n + R)`.

---

## 22. Design

**Signal.** The prompt is a class with operations and target complexities, not a single function. Implement the structure whose operations match the calls you will get.

| Asked operations | Structure |
|---|---|
| get/put in `O(1)`, evict least recently used | hash map + doubly linked list. Map key → node. List order is recency. Every get/put moves the node to the front. A dummy head and tail remove the empty-list edge cases |
| evict least frequently used | same, but one doubly linked list (or bucket) **per frequency**, plus a `min_freq` counter. On use, move the node to the `freq + 1` bucket |
| push/pop/min in `O(1)` | two stacks: values, and the min at or below this depth |
| median of a stream | two heaps, section 10 |
| insert/search/startswith | trie, section 9 |
| snapshot array, set(index, val) and snap() | per index, a list of `(snap_id, value)`. `get` binary-searches the last snap `≤ id` |
| time-based key-value store | map key → list of `(timestamp, value)`, append-only so each list is sorted, binary search |
| randomized set with insert/delete/getRandom in `O(1)` | list (for random index) + map value → index. Delete by swapping with the last element |
| peeking iterator, iterator over BST | stack the control state explicitly so `peek` does not consume |

LRU is the one to be able to write cold. The bugs are always: forgetting to remove the node from its old position before inserting it at the front, and not updating the map when the tail is evicted.

```cpp
struct Node {
    int key = 0, val = 0;
    Node *prev = nullptr, *next = nullptr;
};

class LRUCache {
    int cap;
    unordered_map<int, Node*> mp;
    Node head, tail;
    void remove(Node* node) {
        node->prev->next = node->next;
        node->next->prev = node->prev;
    }
    void addFront(Node* node) {
        node->next = head.next;
        node->prev = &head;
        head.next->prev = node;
        head.next = node;
    }
public:
    LRUCache(int capacity) : cap(capacity) {
        head.next = &tail;
        tail.prev = &head;
    }
    int get(int key) {
        if (!mp.count(key)) return -1;
        Node* node = mp[key];
        remove(node);
        addFront(node);
        return node->val;
    }
    void put(int key, int value) {
        if (mp.count(key)) {
            Node* node = mp[key];
            node->val = value;
            remove(node);
            addFront(node);
            return;
        }
        Node* node = new Node{key, value};
        mp[key] = node;
        addFront(node);
        if ((int)mp.size() > cap) {
            Node* victim = tail.prev;
            remove(victim);
            mp.erase(victim->key);
            delete victim;
        }
    }
};
```

---

## 23. When you are stuck

Run this in order. Stop at the first one that changes the approach.

1. **Restate the brute force** and its complexity. If it already fits `n`, code it. Interviewers would rather see a correct `O(n^2)` at `n ≤ 500` than a broken linear scan.
2. **Name what you recompute.** "Every index re-scans to its left" → prefix, stack, or hash. "Every window re-counts" → sliding window. "Every subset is solved twice" → DP.
3. **Sorted?** If you are allowed to sort, two pointers, intervals, and greedy open up. If relative order must be preserved, you are in hash / stack / DP land.
4. **Contiguous?** Yes → window or Kadane. No (subsequence) → DP or greedy, and a window will be wrong.
5. **Answer type.** Count, min/max, or yes/no. Write the combine step (`+`, `min`, `or`) before the loop.
6. **One parameter or two?** One → 1D DP or a pointer. Two strings or (index, capacity) → 2D. A span `i..j` → interval DP by length. A subset with `n ≤ 20` → bitmask.
7. **Does the local choice survive a swap?** If you can swap your greedy choice into any optimal solution, be greedy. If you can construct a counterexample in thirty seconds, it is DP.
8. **Draw n = 4.** Fill the table by hand. The recurrence is whatever you did with your pencil. Then code that, not a prettier version you have not checked.
9. **Off-by-one audit, after it works on the sample.** Empty input, one element, all equal, all negative, touching intervals (`end == start`), even vs odd length, head deletion, `k = 0`, `k = n`.
10. **Still stuck.** Code the brute force, then add the cache on the arguments you keep passing into the recursion. That is top-down DP, and it finds the state for you. You can rebuild it bottom-up afterward if you want the table.

### Recursion to DP, mechanically

If the recursive function is `f(i, j)` and it calls `f` on smaller `i` or `j`:

- the cache key is `(i, j)`
- the table is those bounds
- the base cases of the recursion are the base cases of the table
- the loop order is whatever makes the callees run first

Memoizing a correct recursion is always correct. It may overflow the stack or lose the tight memory bound, which is why the templates in this guide are written as loops. Start from the recursion when you do not yet know the loop.
