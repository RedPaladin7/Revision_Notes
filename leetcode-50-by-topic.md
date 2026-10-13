# LeetCode 50 by Topic

Same idea as the [LeetCode Dynamic Programming study plan](https://leetcode.com/studyplan/dynamic-programming/): each topic is 50 problems, split by the shapes that topic actually takes, drilled from the basic move to the nasty variant.

Dynamic programming is not in this file. The official plan already covers Fibonacci-style, grid, knapsack, LIS, two-string, stock state machines, tree DP, and multidimensional DP. Use that for DP. Use this for everything else.

750 problems. 15 topics. Each problem is listed once, under the pattern you should learn it for.

## How to use this

- Work a topic top to bottom. The first problem in a subtype is the template. The later ones change one rule.
- Short on time: do the first problem of every subtype in a topic, then come back for the rest. That is the "all types" pass. The later problems are the variations.
- `Premium` means LeetCode locks it. Those are here only when the subtype is incomplete without them (Meeting Rooms, Alien Dictionary, Range Addition). Skip them if you do not have Premium. The free problems still cover the move.
- Difficulty is LeetCode's label. A Medium in week 20 of a topic is usually harder than an Easy in week 1.
- A few famous problems have two legitimate solutions. The pattern column names the one to practice here. The other home is the topic that owns the problem.

| Topic | What the 50 is teaching |
| --- | --- |
| [Arrays & Hashing](#1-arrays--hashing) | Frequency, prefix hashes, index-as-hash, scans, matrix layout |
| [Strings](#2-strings) | Palindromes, parsing, construction, tries |
| [Two Pointers](#3-two-pointers) | Opposite ends, same direction, fast/slow |
| [Sliding Window](#4-sliding-window) | Fixed, at most K, exactly K, cover, monotonic deque |
| [Binary Search](#5-binary-search) | Bounds, peaks, rotated arrays, search on the answer |
| [Stack & Monotonic Stack](#6-stack--monotonic-stack) | Parsing, next greater, histogram contributions |
| [Linked Lists](#7-linked-lists) | Reverse, fast/slow, merge, random pointer, design |
| [Trees & BST](#8-trees--bst) | Traversals, path returns, build, BST edit |
| [Graphs & Union-Find](#9-graphs--union-find) | Grid search, topo, shortest paths, components |
| [Heaps](#10-heaps) | Top-K, k-way merge, two heaps, greedy with a heap |
| [Backtracking](#11-backtracking) | Subsets, permutations, constraint search, partition |
| [Greedy](#12-greedy) | Exchange arguments: jumps, digits, grouping, chunks |
| [Intervals](#13-intervals) | Merge, schedule, cover, line sweep |
| [Bit Manipulation](#14-bit-manipulation) | Masks, XOR prefix, range AND, bit DP-free tricks |
| [Math & Geometry](#15-math--geometry) | Number theory, combinatorics, geometry, randomness |

Design problems are not a separate 50. They sit inside the structure they are really about: LRU and LFU under linked lists, trie under strings, min stack and max stack under stacks, calendars and skyline under intervals, running median under heaps, Insert Delete GetRandom under math.

---

## 1. Arrays & Hashing

The array is usually not the hard part. The hard part is the extra structure you hang on it: a count, a prefix, or the index itself used as a bucket.

Tries on binary numbers are under [Strings](#2-strings). Subarray windows that shrink from the left are under [Sliding Window](#4-sliding-window).

### Frequency and complements

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 1 | [1. Two Sum](https://leetcode.com/problems/two-sum/) | Easy | Complement in a hash map |
| 2 | [217. Contains Duplicate](https://leetcode.com/problems/contains-duplicate/) | Easy | Set membership |
| 3 | [219. Contains Duplicate II](https://leetcode.com/problems/contains-duplicate-ii/) | Easy | Last index in range |
| 4 | [128. Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/) | Medium | Expand only from sequence starts |
| 5 | [242. Valid Anagram](https://leetcode.com/problems/valid-anagram/) | Easy | Character counts |
| 6 | [49. Group Anagrams](https://leetcode.com/problems/group-anagrams/) | Medium | Count tuple as the key |
| 7 | [383. Ransom Note](https://leetcode.com/problems/ransom-note/) | Easy | Counts, then consume |
| 8 | [387. First Unique Character in a String](https://leetcode.com/problems/first-unique-character-in-a-string/) | Easy | Count, then first pass |
| 9 | [451. Sort Characters By Frequency](https://leetcode.com/problems/sort-characters-by-frequency/) | Medium | Bucket by frequency |
| 10 | [347. Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) | Medium | Bucket sort, not a heap |
| 11 | [454. 4Sum II](https://leetcode.com/problems/4sum-ii/) | Medium | Count pair sums |

### Prefix sums and difference arrays

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 12 | [1480. Running Sum of 1d Array](https://leetcode.com/problems/running-sum-of-1d-array/) | Easy | Prefix sum |
| 13 | [1732. Find the Highest Altitude](https://leetcode.com/problems/find-the-highest-altitude/) | Easy | Prefix of a gain array |
| 14 | [724. Find Pivot Index](https://leetcode.com/problems/find-pivot-index/) | Easy | Left sum vs total |
| 15 | [303. Range Sum Query - Immutable](https://leetcode.com/problems/range-sum-query-immutable/) | Easy | Prefix, then O(1) ranges |
| 16 | [560. Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/) | Medium | Prefix + count of `sum - k` |
| 17 | [974. Subarray Sums Divisible by K](https://leetcode.com/problems/subarray-sums-divisible-by-k/) | Medium | Prefix modulo |
| 18 | [523. Continuous Subarray Sum](https://leetcode.com/problems/continuous-subarray-sum/) | Medium | Prefix mod, length at least 2 |
| 19 | [525. Contiguous Array](https://leetcode.com/problems/contiguous-array/) | Medium | Balance prefix, 0 as -1 |
| 20 | [1010. Pairs of Songs With Total Durations Divisible by 60](https://leetcode.com/problems/pairs-of-songs-with-total-durations-divisible-by-60/) | Medium | Mod buckets, not a subarray |
| 21 | [1588. Sum of All Odd Length Subarrays](https://leetcode.com/problems/sum-of-all-odd-length-subarrays/) | Easy | How many subarrays cover index i |
| 22 | [238. Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/) | Medium | Prefix product and suffix product |
| 23 | [304. Range Sum Query 2D - Immutable](https://leetcode.com/problems/range-sum-query-2d-immutable/) | Medium | 2D prefix (inclusion-exclusion) |
| 24 | [1314. Matrix Block Sum](https://leetcode.com/problems/matrix-block-sum/) | Medium | 2D prefix, clamped window |
| 25 | [1074. Number of Submatrices That Sum to Target](https://leetcode.com/problems/number-of-submatrices-that-sum-to-target/) | Hard | Fix two rows, then 1D prefix hash |
| 26 | [327. Count of Range Sum](https://leetcode.com/problems/count-of-range-sum/) | Hard | Prefix + merge sort count |
| 27 | [1109. Corporate Flight Bookings](https://leetcode.com/problems/corporate-flight-bookings/) | Medium | Difference array |
| 28 | [1094. Car Pooling](https://leetcode.com/problems/car-pooling/) | Medium | Difference array, then scan |

### Index as a hash, and one-pass scans

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 29 | [268. Missing Number](https://leetcode.com/problems/missing-number/) | Easy | XOR or sum formula |
| 30 | [136. Single Number](https://leetcode.com/problems/single-number/) | Easy | XOR cancel |
| 31 | [137. Single Number II](https://leetcode.com/problems/single-number-ii/) | Medium | Bit count mod 3 |
| 32 | [260. Single Number III](https://leetcode.com/problems/single-number-iii/) | Medium | XOR, then split on a set bit |
| 33 | [645. Set Mismatch](https://leetcode.com/problems/set-mismatch/) | Easy | Index marking |
| 34 | [448. Find All Numbers Disappeared in an Array](https://leetcode.com/problems/find-all-numbers-disappeared-in-an-array/) | Easy | Index marking |
| 35 | [442. Find All Duplicates in an Array](https://leetcode.com/problems/find-all-duplicates-in-an-array/) | Medium | Index marking, values in range |
| 36 | [41. First Missing Positive](https://leetcode.com/problems/first-missing-positive/) | Hard | Cycle values onto their indexes |
| 37 | [287. Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/) | Medium | Floyd on the index graph |
| 38 | [169. Majority Element](https://leetcode.com/problems/majority-element/) | Easy | Boyer-Moore vote |
| 39 | [229. Majority Element II](https://leetcode.com/problems/majority-element-ii/) | Medium | Two candidates |
| 40 | [53. Maximum Subarray](https://leetcode.com/problems/maximum-subarray/) | Medium | Kadane |
| 41 | [152. Maximum Product Subarray](https://leetcode.com/problems/maximum-product-subarray/) | Medium | Track min and max product |
| 42 | [918. Maximum Sum Circular Subarray](https://leetcode.com/problems/maximum-sum-circular-subarray/) | Medium | Kadane, or total minus min subarray |
| 43 | [121. Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) | Easy | Running minimum |
| 44 | [122. Best Time to Buy and Sell Stock II](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/) | Medium | Sum every ascent |

Stock with a fee, cooldown, or k transactions is DP. It lives on the official DP plan, not here. [714](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/) is the one exception filed under [Greedy](#12-greedy), because the fee version collapses to a one-pass state.

### In place and matrix layout

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 45 | [189. Rotate Array](https://leetcode.com/problems/rotate-array/) | Medium | Reverse three times |
| 46 | [54. Spiral Matrix](https://leetcode.com/problems/spiral-matrix/) | Medium | Shrink the layer |
| 47 | [59. Spiral Matrix II](https://leetcode.com/problems/spiral-matrix-ii/) | Medium | Write a layer |
| 48 | [48. Rotate Image](https://leetcode.com/problems/rotate-image/) | Medium | Four-cycle, or transpose + reverse |
| 49 | [73. Set Matrix Zeroes](https://leetcode.com/problems/set-matrix-zeroes/) | Medium | First row and column as markers |
| 50 | [36. Valid Sudoku](https://leetcode.com/problems/valid-sudoku/) | Medium | Row, column, and box sets |

---

## 2. Strings

Palindrome expansion, parsing, building the answer, and prefix trees. Anagram counts are under [Arrays & Hashing](#1-arrays--hashing). Sliding-window anagrams are under [Sliding Window](#4-sliding-window). Parenthesis calculators are under [Stack](#6-stack--monotonic-stack).

### Palindromes

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 1 | [125. Valid Palindrome](https://leetcode.com/problems/valid-palindrome/) | Easy | Skip non-alphanumerics |
| 2 | [9. Palindrome Number](https://leetcode.com/problems/palindrome-number/) | Easy | Reverse half the digits |
| 3 | [680. Valid Palindrome II](https://leetcode.com/problems/valid-palindrome-ii/) | Easy | One deletion |
| 4 | [409. Longest Palindrome](https://leetcode.com/problems/longest-palindrome/) | Easy | Pairs from a frequency map |
| 5 | [5. Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring/) | Medium | Expand around centers |
| 6 | [647. Palindromic Substrings](https://leetcode.com/problems/palindromic-substrings/) | Medium | Count every expansion |
| 7 | [214. Shortest Palindrome](https://leetcode.com/problems/shortest-palindrome/) | Hard | KMP on `s + # + reverse(s)` |
| 8 | [336. Palindrome Pairs](https://leetcode.com/problems/palindrome-pairs/) | Hard | Hash words, check the split |

### Match, map, subsequence

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 9 | [151. Reverse Words in a String](https://leetcode.com/problems/reverse-words-in-a-string/) | Medium | Split, or reverse the whole string |
| 10 | [557. Reverse Words in a String III](https://leetcode.com/problems/reverse-words-in-a-string-iii/) | Easy | Reverse each word |
| 11 | [14. Longest Common Prefix](https://leetcode.com/problems/longest-common-prefix/) | Easy | Vertical scan |
| 12 | [28. Find the Index of the First Occurrence in a String](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/) | Easy | Naive scan; KMP is the follow-up |
| 13 | [459. Repeated Substring Pattern](https://leetcode.com/problems/repeated-substring-pattern/) | Easy | `s` in `(s + s)[1:-1]` |
| 14 | [686. Repeated String Match](https://leetcode.com/problems/repeated-string-match/) | Medium | How many copies cover `b` |
| 15 | [796. Rotate String](https://leetcode.com/problems/rotate-string/) | Easy | `goal` in `s + s` |
| 16 | [205. Isomorphic Strings](https://leetcode.com/problems/isomorphic-strings/) | Easy | Two-way last-seen map |
| 17 | [290. Word Pattern](https://leetcode.com/problems/word-pattern/) | Easy | Bijection, pattern to word |
| 18 | [890. Find and Replace Pattern](https://leetcode.com/problems/find-and-replace-pattern/) | Medium | Normalize to the first-seen encoding |
| 19 | [392. Is Subsequence](https://leetcode.com/problems/is-subsequence/) | Easy | Walk the target |
| 20 | [524. Longest Word in Dictionary through Deleting](https://leetcode.com/problems/longest-word-in-dictionary-through-deleting/) | Medium | Subsequence check per word |
| 21 | [792. Number of Matching Subsequences](https://leetcode.com/problems/number-of-matching-subsequences/) | Medium | Bucket words by the next character they need |

### Parse and build

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 22 | [13. Roman to Integer](https://leetcode.com/problems/roman-to-integer/) | Easy | Subtractive pairs |
| 23 | [12. Integer to Roman](https://leetcode.com/problems/integer-to-roman/) | Medium | Greedy denomination list |
| 24 | [8. String to Integer (atoi)](https://leetcode.com/problems/string-to-integer-atoi/) | Medium | State while scanning |
| 25 | [67. Add Binary](https://leetcode.com/problems/add-binary/) | Easy | Add from the right, carry |
| 26 | [415. Add Strings](https://leetcode.com/problems/add-strings/) | Easy | Same, in base 10 |
| 27 | [43. Multiply Strings](https://leetcode.com/problems/multiply-strings/) | Medium | Grade-school into an array |
| 28 | [165. Compare Version Numbers](https://leetcode.com/problems/compare-version-numbers/) | Medium | Split on dots |
| 29 | [38. Count and Say](https://leetcode.com/problems/count-and-say/) | Medium | Run-length the previous term |
| 30 | [443. String Compression](https://leetcode.com/problems/string-compression/) | Medium | Read pointer, write pointer |
| 31 | [6. Zigzag Conversion](https://leetcode.com/problems/zigzag-conversion/) | Medium | Row simulation |
| 32 | [68. Text Justification](https://leetcode.com/problems/text-justification/) | Hard | Pack a line, then distribute spaces |
| 33 | [93. Restore IP Addresses](https://leetcode.com/problems/restore-ip-addresses/) | Medium | 4 segments, leading-zero rules |
| 34 | [17. Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/) | Medium | Cartesian product of digit letters |

### Build the smallest or the balanced string

These are greedy or stack-shaped, but the thing you are constructing is a string, so they stay here. The general monotonic-stack catalog is under [Stack](#6-stack--monotonic-stack).

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 35 | [763. Partition Labels](https://leetcode.com/problems/partition-labels/) | Medium | Last index of each char |
| 36 | [767. Reorganize String](https://leetcode.com/problems/reorganize-string/) | Medium | Most frequent char first |
| 37 | [316. Remove Duplicate Letters](https://leetcode.com/problems/remove-duplicate-letters/) | Medium | Stack + last occurrence |
| 38 | [402. Remove K Digits](https://leetcode.com/problems/remove-k-digits/) | Medium | Pop larger digits while you can |
| 39 | [678. Valid Parenthesis String](https://leetcode.com/problems/valid-parenthesis-string/) | Medium | Balance range, `*` is wild |
| 40 | [921. Minimum Add to Make Parentheses Valid](https://leetcode.com/problems/minimum-add-to-make-parentheses-valid/) | Medium | Balance counter |
| 41 | [1963. Minimum Number of Swaps to Make the String Balanced](https://leetcode.com/problems/minimum-number-of-swaps-to-make-the-string-balanced/) | Medium | Unmatched `]` / 2 |
| 42 | [1653. Minimum Deletions to Make String Balanced](https://leetcode.com/problems/minimum-deletions-to-make-string-balanced/) | Medium | Count of `b` before each `a` |

### Trie

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 43 | [208. Implement Trie (Prefix Tree)](https://leetcode.com/problems/implement-trie-prefix-tree/) | Medium | Insert, search, startsWith |
| 44 | [720. Longest Word in Dictionary](https://leetcode.com/problems/longest-word-in-dictionary/) | Easy | Word built one char at a time |
| 45 | [648. Replace Words](https://leetcode.com/problems/replace-words/) | Medium | Shortest root that is a prefix |
| 46 | [211. Design Add and Search Words Data Structure](https://leetcode.com/problems/design-add-and-search-words-data-structure/) | Medium | `.` branches to every child |
| 47 | [1268. Search Suggestions System](https://leetcode.com/problems/search-suggestions-system/) | Medium | Walk the prefix, take 3 |
| 48 | [676. Implement Magic Dictionary](https://leetcode.com/problems/implement-magic-dictionary/) | Medium | Exactly one mismatch |
| 49 | [212. Word Search II](https://leetcode.com/problems/word-search-ii/) | Hard | Trie pruned by the board DFS |
| 50 | [421. Maximum XOR of Two Numbers in an Array](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/) | Medium | Binary trie, prefer the opposite bit |

---

## 3. Two Pointers

Three different algorithms share the name.

- Opposite ends: sorted array, palindrome, container.
- Same direction: write index lags the read index.
- Fast and slow: a cycle, or the middle.

Shrinkable subarray windows are under [Sliding Window](#4-sliding-window). Fast/slow on a real linked list is under [Linked Lists](#7-linked-lists).

### Opposite ends

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 1 | [167. Two Sum II - Input Array Is Sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/) | Easy | Move the end that fixes the sum |
| 2 | [15. 3Sum](https://leetcode.com/problems/3sum/) | Medium | Sort, fix one, two-sum the rest |
| 3 | [16. 3Sum Closest](https://leetcode.com/problems/3sum-closest/) | Medium | Same, track the best |
| 4 | [18. 4Sum](https://leetcode.com/problems/4sum/) | Medium | Fix two, two-sum the rest |
| 5 | [923. 3Sum With Multiplicity](https://leetcode.com/problems/3sum-with-multiplicity/) | Medium | Count duplicates as you close in |
| 6 | [611. Valid Triangle Number](https://leetcode.com/problems/valid-triangle-number/) | Medium | Fix the largest side |
| 7 | [11. Container With Most Water](https://leetcode.com/problems/container-with-most-water/) | Medium | Advance the shorter side |
| 8 | [42. Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) | Hard | Inward, with left-max and right-max |
| 9 | [977. Squares of a Sorted Array](https://leetcode.com/problems/squares-of-a-sorted-array/) | Easy | Largest square is at one end |
| 10 | [881. Boats to Save People](https://leetcode.com/problems/boats-to-save-people/) | Medium | Lightest with heaviest, if it fits |
| 11 | [1877. Minimize Maximum Pair Sum in Array](https://leetcode.com/problems/minimize-maximum-pair-sum-in-array/) | Medium | Pair smallest with largest |
| 12 | [1679. Max Number of K-Sum Pairs](https://leetcode.com/problems/max-number-of-k-sum-pairs/) | Medium | Sort, then pair to k |
| 13 | [1498. Number of Subsequences That Satisfy the Given Sum Condition](https://leetcode.com/problems/number-of-subsequences-that-satisfy-the-given-sum-condition/) | Medium | Min + max, then `2^(j-i)` |
| 14 | [948. Bag of Tokens](https://leetcode.com/problems/bag-of-tokens/) | Medium | Buy cheap score, sell expensive |
| 15 | [633. Sum of Square Numbers](https://leetcode.com/problems/sum-of-square-numbers/) | Medium | `a^2 + b^2 = c` |
| 16 | [1577. Number of Ways Where Square of Number Is Equal to Product of Two Numbers](https://leetcode.com/problems/number-of-ways-where-square-of-number-is-equal-to-product-of-two-numbers/) | Medium | Two ends on a sorted array |
| 17 | [825. Friends Of Appropriate Ages](https://leetcode.com/problems/friends-of-appropriate-ages/) | Medium | Sort, then a sliding right bound |
| 18 | [870. Advantage Shuffle](https://leetcode.com/problems/advantage-shuffle/) | Medium | Smallest number that still beats |
| 19 | [826. Most Profit Assigning Work](https://leetcode.com/problems/most-profit-assigning-work/) | Medium | Sort jobs and workers together |
| 20 | [2563. Count the Number of Fair Pairs](https://leetcode.com/problems/count-the-number-of-fair-pairs/) | Medium | Sort, count a range of partners |
| 21 | [1855. Maximum Distance Between a Pair of Values](https://leetcode.com/problems/maximum-distance-between-a-pair-of-values/) | Medium | Both pointers only move forward |

### Same direction, partition, and strings

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 22 | [26. Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/) | Easy | Write index |
| 23 | [80. Remove Duplicates from Sorted Array II](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/) | Medium | Allow two copies |
| 24 | [27. Remove Element](https://leetcode.com/problems/remove-element/) | Easy | Write the keepers |
| 25 | [283. Move Zeroes](https://leetcode.com/problems/move-zeroes/) | Easy | Write the non-zeros |
| 26 | [75. Sort Colors](https://leetcode.com/problems/sort-colors/) | Medium | Dutch national flag, three bands |
| 27 | [88. Merge Sorted Array](https://leetcode.com/problems/merge-sorted-array/) | Easy | Fill from the back |
| 28 | [905. Sort Array By Parity](https://leetcode.com/problems/sort-array-by-parity/) | Easy | Partition evens left |
| 29 | [922. Sort Array By Parity II](https://leetcode.com/problems/sort-array-by-parity-ii/) | Easy | Even indexes and odd indexes |
| 30 | [228. Summary Ranges](https://leetcode.com/problems/summary-ranges/) | Easy | Start of a run, end of a run |
| 31 | [349. Intersection of Two Arrays](https://leetcode.com/problems/intersection-of-two-arrays/) | Easy | Set, or two pointers after sort |
| 32 | [350. Intersection of Two Arrays II](https://leetcode.com/problems/intersection-of-two-arrays-ii/) | Easy | Counts, multiplicity matters |
| 33 | [345. Reverse Vowels of a String](https://leetcode.com/problems/reverse-vowels-of-a-string/) | Easy | Swap vowels from both ends |
| 34 | [917. Reverse Only Letters](https://leetcode.com/problems/reverse-only-letters/) | Easy | Skip the non-letters |
| 35 | [1750. Minimum Length of String After Deleting Similar Ends](https://leetcode.com/problems/minimum-length-of-string-after-deleting-similar-ends/) | Medium | Eat the same char from both ends |
| 36 | [844. Backspace String Compare](https://leetcode.com/problems/backspace-string-compare/) | Easy | Walk both strings from the end |
| 37 | [925. Long Pressed Name](https://leetcode.com/problems/long-pressed-name/) | Easy | Runs must match, typed run can be longer |
| 38 | [809. Expressive Words](https://leetcode.com/problems/expressive-words/) | Medium | A run of 3+ can stretch |
| 39 | [777. Swap Adjacent in LR String](https://leetcode.com/problems/swap-adjacent-in-lr-string/) | Medium | `L` only moves left, `R` only right |

### Runs, mountains, cycles

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 40 | [845. Longest Mountain](https://leetcode.com/problems/longest-mountain-in-array/) | Medium | Climb, then descend |
| 41 | [581. Shortest Unsorted Continuous Subarray](https://leetcode.com/problems/shortest-unsorted-continuous-subarray/) | Medium | Find the segment that breaks sorted order |
| 42 | [1574. Shortest Subarray to be Removed to Make Array Sorted](https://leetcode.com/problems/shortest-subarray-to-be-removed-to-make-array-sorted/) | Medium | Longest prefix + suffix that can join |
| 43 | [1658. Minimum Operations to Reduce X to Zero](https://leetcode.com/problems/minimum-operations-to-reduce-x-to-zero/) | Medium | Ends sum to x, so the middle sums to total - x |
| 44 | [795. Number of Subarrays With Bounded Maximum](https://leetcode.com/problems/number-of-subarrays-with-bounded-maximum/) | Medium | Count `atMost(right) - atMost(left-1)` |
| 45 | [2444. Count Subarrays With Fixed Bounds](https://leetcode.com/problems/count-subarrays-with-fixed-bounds/) | Hard | Last min, last max, last out-of-range |
| 46 | [1793. Maximum Score of a Good Subarray](https://leetcode.com/problems/maximum-score-of-a-good-subarray/) | Hard | Expand from `k` toward the larger side |
| 47 | [475. Heaters](https://leetcode.com/problems/heaters/) | Medium | Each house takes the nearer heater |
| 48 | [324. Wiggle Sort II](https://leetcode.com/problems/wiggle-sort-ii/) | Medium | Median, then map indexes |
| 49 | [202. Happy Number](https://leetcode.com/problems/happy-number/) | Easy | Floyd cycle on the digit-square sequence |
| 50 | [457. Circular Array Loop](https://leetcode.com/problems/circular-array-loop/) | Medium | Floyd, same direction, length > 1 |

---

## 4. Sliding Window

A window is two pointers plus a reason to move the left edge. Learn them in this order: fixed length, grow-and-shrink ("at most K"), exactly K as a difference of two at-most windows, then a window that must cover a multiset, then a deque when the window needs its min or max.

### Fixed length

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 1 | [643. Maximum Average Subarray I](https://leetcode.com/problems/maximum-average-subarray-i/) | Easy | Sum of length k, slide by one |
| 2 | [1343. Number of Sub-arrays of Size K and Average Greater than or Equal to Threshold](https://leetcode.com/problems/number-of-sub-arrays-of-size-k-and-average-greater-than-or-equal-to-threshold/) | Medium | Fixed sum vs threshold |
| 3 | [1456. Maximum Number of Vowels in a Substring of Given Length](https://leetcode.com/problems/maximum-number-of-vowels-in-a-substring-of-given-length/) | Medium | Fixed window, vowel count |
| 4 | [1876. Substrings of Size Three with Distinct Characters](https://leetcode.com/problems/substrings-of-size-three-with-distinct-characters/) | Easy | Every window of 3 |
| 5 | [2379. Minimum Recolors to Get K Consecutive Black Blocks](https://leetcode.com/problems/minimum-recolors-to-get-k-consecutive-black-blocks/) | Easy | Fixed window, count white |
| 6 | [1052. Grumpy Bookstore Owner](https://leetcode.com/problems/grumpy-bookstore-owner/) | Medium | Fixed window of extra customers |
| 7 | [1652. Defuse the Bomb](https://leetcode.com/problems/defuse-the-bomb/) | Easy | Circular fixed window |
| 8 | [2090. K Radius Subarray Averages](https://leetcode.com/problems/k-radius-subarray-averages/) | Medium | Window of length `2k+1` |
| 9 | [2461. Maximum Sum of Distinct Subarrays With Length K](https://leetcode.com/problems/maximum-sum-of-distinct-subarrays-with-length-k/) | Medium | Fixed window plus a frequency map |
| 10 | [2841. Maximum Sum of Almost Unique Subarray](https://leetcode.com/problems/maximum-sum-of-almost-unique-subarray/) | Medium | Fixed window, at least m distinct |
| 11 | [1151. Minimum Swaps to Group All 1's Together](https://leetcode.com/problems/minimum-swaps-to-group-all-1s-together/) | Medium | Window length = number of ones |
| 12 | [2134. Minimum Swaps to Group All 1's Together II](https://leetcode.com/problems/minimum-swaps-to-group-all-1s-together-ii/) | Medium | Same, on a circular array |

### At most K

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 13 | [485. Max Consecutive Ones](https://leetcode.com/problems/max-consecutive-ones/) | Easy | Break on a zero. No budget yet |
| 14 | [209. Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/) | Medium | Shrink while sum is enough |
| 15 | [3. Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/) | Medium | Last index, jump the left edge |
| 16 | [904. Fruit Into Baskets](https://leetcode.com/problems/fruit-into-baskets/) | Medium | At most 2 distinct |
| 17 | [1004. Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/) | Medium | At most k zeros |
| 18 | [1493. Longest Subarray of 1's After Deleting One Element](https://leetcode.com/problems/longest-subarray-of-1s-after-deleting-one-element/) | Medium | At most one zero, and you must delete |
| 19 | [487. Max Consecutive Ones II](https://leetcode.com/problems/max-consecutive-ones-ii/) | Medium | At most one zero. Premium |
| 20 | [2024. Maximize the Confusion of an Exam](https://leetcode.com/problems/maximize-the-confusion-of-an-exam/) | Medium | At most k flips, try both targets |
| 21 | [424. Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/) | Medium | Window - max freq <= k |
| 22 | [1838. Frequency of the Most Frequent Element](https://leetcode.com/problems/frequency-of-the-most-frequent-element/) | Medium | Sort, then cost to raise the window |
| 23 | [1208. Get Equal Substrings Within Budget](https://leetcode.com/problems/get-equal-substrings-within-budget/) | Medium | Cost of `|s[i]-t[i]|` |
| 24 | [713. Subarray Product Less Than K](https://leetcode.com/problems/subarray-product-less-than-k/) | Medium | Shrink while product is too big |
| 25 | [1695. Maximum Erasure Value](https://leetcode.com/problems/maximum-erasure-value/) | Medium | Unique elements, maximize sum |
| 26 | [2958. Length of Longest Subarray With at Most K Frequency](https://leetcode.com/problems/length-of-longest-subarray-with-at-most-k-frequency/) | Medium | No frequency above k |
| 27 | [159. Longest Substring with At Most Two Distinct Characters](https://leetcode.com/problems/longest-substring-with-at-most-two-distinct-characters/) | Medium | At most 2 distinct. Premium |
| 28 | [340. Longest Substring with At Most K Distinct Characters](https://leetcode.com/problems/longest-substring-with-at-most-k-distinct-characters/) | Medium | At most k distinct. Premium |

### Exactly K, and cover

Exactly K is almost never a single window. It is `atMost(K) - atMost(K-1)`.

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 29 | [930. Binary Subarrays With Sum](https://leetcode.com/problems/binary-subarrays-with-sum/) | Medium | Exactly k zeros-and-ones sum |
| 30 | [1248. Count Number of Nice Subarrays](https://leetcode.com/problems/count-number-of-nice-subarrays/) | Medium | Exactly k odds |
| 31 | [992. Subarrays with K Different Integers](https://leetcode.com/problems/subarrays-with-k-different-integers/) | Hard | Exactly k distinct |
| 32 | [1358. Number of Substrings Containing All Three Characters](https://leetcode.com/problems/number-of-substrings-containing-all-three-characters/) | Medium | Once the window is valid, every larger end works |
| 33 | [2799. Count Complete Subarrays in an Array](https://leetcode.com/problems/count-complete-subarrays-in-an-array/) | Medium | Window holds every distinct value |
| 34 | [2962. Count Subarrays Where Max Element Appears at Least K Times](https://leetcode.com/problems/count-subarrays-where-max-element-appears-at-least-k-times/) | Medium | At least k copies of the max |
| 35 | [395. Longest Substring with At Least K Repeating Characters](https://leetcode.com/problems/longest-substring-with-at-least-k-repeating-characters/) | Medium | Split on rare chars, or try each distinct-count |
| 36 | [76. Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) | Hard | Cover a need-count, then shrink |
| 37 | [567. Permutation in String](https://leetcode.com/problems/permutation-in-string/) | Medium | Fixed window, anagram counts |
| 38 | [438. Find All Anagrams in a String](https://leetcode.com/problems/find-all-anagrams-in-a-string/) | Medium | Same, record every hit |
| 39 | [30. Substring with Concatenation of All Words](https://leetcode.com/problems/substring-with-concatenation-of-all-words/) | Hard | Word-sized steps, need-count of words |
| 40 | [2516. Take K of Each Character From Left and Right](https://leetcode.com/problems/take-k-of-each-character-from-left-and-right/) | Medium | Minimize the middle you delete |
| 41 | [1156. Swap For Longest Repeated Character Substring](https://leetcode.com/problems/swap-for-longest-repeated-character-substring/) | Medium | A run, plus one swap from elsewhere |

### Monotonic deque

When the window must know its minimum or maximum in O(1), a sorted map works and a deque is the intended tool. The deque stores indexes, decreasing or increasing, and you pop the front when it falls out of the window.

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 42 | [239. Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/) | Hard | Decreasing deque of indexes |
| 43 | [1438. Longest Continuous Subarray With Absolute Diff Less Than or Equal to Limit](https://leetcode.com/problems/longest-continuous-subarray-with-absolute-diff-less-than-or-equal-to-limit/) | Medium | Deque min and deque max |
| 44 | [862. Shortest Subarray with Sum at Least K](https://leetcode.com/problems/shortest-subarray-with-sum-at-least-k/) | Hard | Prefix, increasing deque. Negatives break a normal window |
| 45 | [1696. Jump Game VI](https://leetcode.com/problems/jump-game-vi/) | Medium | Best score in the last k indexes |
| 46 | [1425. Constrained Subsequence Sum](https://leetcode.com/problems/constrained-subsequence-sum/) | Hard | Same idea, take or skip |
| 47 | [480. Sliding Window Median](https://leetcode.com/problems/sliding-window-median/) | Hard | Two heaps, or two multisets, plus lazy delete |
| 48 | [978. Longest Turbulent Subarray](https://leetcode.com/problems/longest-turbulent-subarray/) | Medium | Reset when the up/down pattern breaks |
| 49 | [1446. Consecutive Characters](https://leetcode.com/problems/consecutive-characters/) | Easy | Current run length |
| 50 | [1703. Minimum Adjacent Swaps for K Consecutive Ones](https://leetcode.com/problems/minimum-adjacent-swaps-for-k-consecutive-ones/) | Hard | Median of the positions of ones |

---

## 5. Binary Search

Three families, and they do not feel like the same algorithm until you have done all three.

- Search a sorted array: lower bound, upper bound, first true.
- The array is sorted but broken: rotated, mountain, 2D.
- Binary search on the answer: the predicate "can I do it with capacity x?" is monotonic, so you search x. You never search the input array.

Patience sorting for LIS is here because the O(n log n) solution is a binary search, not the O(n^2) DP.

### Bounds on a sorted array

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 1 | [704. Binary Search](https://leetcode.com/problems/binary-search/) | Easy | The template |
| 2 | [35. Search Insert Position](https://leetcode.com/problems/search-insert-position/) | Easy | Lower bound |
| 3 | [34. Find First and Last Position of Element in Sorted Array](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/) | Medium | Lower bound and upper bound |
| 4 | [278. First Bad Version](https://leetcode.com/problems/first-bad-version/) | Easy | First true in a boolean array |
| 5 | [1539. Kth Missing Positive Number](https://leetcode.com/problems/kth-missing-positive-number/) | Easy | Missing count at mid |
| 6 | [540. Single Element in a Sorted Array](https://leetcode.com/problems/single-element-in-a-sorted-array/) | Medium | Pairs: even index equals the next |
| 7 | [275. H-Index II](https://leetcode.com/problems/h-index-ii/) | Medium | Citations already sorted |
| 8 | [1351. Count Negative Numbers in a Sorted Matrix](https://leetcode.com/problems/count-negative-numbers-in-a-sorted-matrix/) | Easy | Each row is sorted, start from the corner |
| 9 | [1608. Special Array With X Elements Greater Than or Equal X](https://leetcode.com/problems/special-array-with-x-elements-greater-than-or-equal-x/) | Easy | Search x, count how many are >= x |
| 10 | [2300. Successful Pairs of Spells and Potions](https://leetcode.com/problems/successful-pairs-of-spells-and-potions/) | Medium | Sort potions, lower-bound the partner |

### Roots, peaks, rotated, 2D

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 11 | [69. Sqrt(x)](https://leetcode.com/problems/sqrtx/) | Easy | Search the integer root |
| 12 | [367. Valid Perfect Square](https://leetcode.com/problems/valid-perfect-square/) | Easy | Search, do not use `sqrt` |
| 13 | [162. Find Peak Element](https://leetcode.com/problems/find-peak-element/) | Medium | Climb the larger neighbor |
| 14 | [852. Peak Index in a Mountain Array](https://leetcode.com/problems/peak-index-in-a-mountain-array/) | Medium | Bitonic, one peak |
| 15 | [1095. Find in Mountain Array](https://leetcode.com/problems/find-in-mountain-array/) | Hard | Find the peak, search both sides |
| 16 | [153. Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/) | Medium | Which half is unsorted |
| 17 | [154. Find Minimum in Rotated Sorted Array II](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array-ii/) | Hard | Duplicates, shrink one side |
| 18 | [33. Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/) | Medium | Decide which half is sorted |
| 19 | [81. Search in Rotated Sorted Array II](https://leetcode.com/problems/search-in-rotated-sorted-array-ii/) | Medium | Duplicates again |
| 20 | [74. Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/) | Medium | Treat it as one sorted array |
| 21 | [240. Search a 2D Matrix II](https://leetcode.com/problems/search-a-2d-matrix-ii/) | Medium | Start at a corner, discard a row or column |

### Search the answer

Write the predicate first: `ok(x)` is true for every x past some point, or false past some point. Then binary search x. The input array is only there to evaluate `ok`.

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 22 | [875. Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/) | Medium | Min speed that finishes in h hours |
| 23 | [1011. Capacity To Ship Packages Within D Days](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/) | Medium | Min capacity |
| 24 | [1482. Minimum Number of Days to Make m Bouquets](https://leetcode.com/problems/minimum-number-of-days-to-make-m-bouquets/) | Medium | Min day, flowers must be adjacent |
| 25 | [1283. Find the Smallest Divisor Given a Threshold](https://leetcode.com/problems/find-the-smallest-divisor-given-a-threshold/) | Medium | Min divisor |
| 26 | [2064. Minimized Maximum of Products Distributed to Any Store](https://leetcode.com/problems/minimized-maximum-of-products-distributed-to-any-store/) | Medium | Min possible max load |
| 27 | [2226. Maximum Candies Allocated to K Children](https://leetcode.com/problems/maximum-candies-allocated-to-k-children/) | Medium | Max equal share |
| 28 | [1552. Magnetic Force Between Two Balls](https://leetcode.com/problems/magnetic-force-between-two-balls/) | Medium | Max min-distance, greedy place |
| 29 | [1870. Minimum Speed to Arrive on Time](https://leetcode.com/problems/minimum-speed-to-arrive-on-time/) | Medium | Min speed, ceiling on every hop but the last |
| 30 | [2187. Minimum Time to Complete Trips](https://leetcode.com/problems/minimum-time-to-complete-trips/) | Medium | Time t, how many trips |
| 31 | [1760. Minimum Limit of Balls in a Bag](https://leetcode.com/problems/minimum-limit-of-balls-in-a-bag/) | Medium | Min max, cost is the splits |
| 32 | [410. Split Array Largest Sum](https://leetcode.com/problems/split-array-largest-sum/) | Hard | Min largest piece, k cuts |
| 33 | [774. Minimize Max Distance to Gas Station](https://leetcode.com/problems/minimize-max-distance-to-gas-station/) | Hard | Min max gap. Premium |
| 34 | [1802. Maximum Value at a Given Index in a Bounded Array](https://leetcode.com/problems/maximum-value-at-a-given-index-in-a-bounded-array/) | Medium | Peak at index, sum of both slopes |
| 35 | [1300. Sum of Mutated Array Closest to Target](https://leetcode.com/problems/sum-of-mutated-array-closest-to-target/) | Medium | Search the cap |
| 36 | [2439. Minimize Maximum of Array](https://leetcode.com/problems/minimize-maximum-of-array/) | Medium | Prefix average is a lower bound |
| 37 | [2560. House Robber IV](https://leetcode.com/problems/house-robber-iv/) | Medium | Min capability, greedy take |
| 38 | [2141. Maximum Running Time of N Computers](https://leetcode.com/problems/maximum-running-time-of-n-computers/) | Hard | Minutes t, batteries can be shared |
| 39 | [2594. Minimum Time to Repair Cars](https://leetcode.com/problems/minimum-time-to-repair-cars/) | Medium | Time t, rank r repairs `floor(sqrt(t/r))` |
| 40 | [1201. Ugly Number III](https://leetcode.com/problems/ugly-number-iii/) | Medium | Inclusion-exclusion as the count |
| 41 | [878. Nth Magical Number](https://leetcode.com/problems/nth-magical-number/) | Hard | Count multiples of a or b |
| 42 | [793. Preimage Size of Factorial Zeroes Function](https://leetcode.com/problems/preimage-size-of-factorial-zeroes-function/) | Hard | Trailing zeros is monotonic, with gaps |

### Search plus another structure

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 43 | [4. Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/) | Hard | Partition both arrays |
| 44 | [378. Kth Smallest Element in a Sorted Matrix](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/) | Medium | Search the value, count how many are smaller |
| 45 | [668. Kth Smallest Number in Multiplication Table](https://leetcode.com/problems/kth-smallest-number-in-multiplication-table/) | Hard | Search the value, count `min(m, x/i)` |
| 46 | [300. Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/) | Medium | Patience sorting, tails array |
| 47 | [354. Russian Doll Envelopes](https://leetcode.com/problems/russian-doll-envelopes/) | Hard | Sort, then LIS on the other dimension |
| 48 | [981. Time Based Key-Value Store](https://leetcode.com/problems/time-based-key-value-store/) | Medium | Per key, binary search the timestamp |
| 49 | [528. Random Pick with Weight](https://leetcode.com/problems/random-pick-with-weight/) | Medium | Prefix sums, then lower bound a random point |
| 50 | [1044. Longest Duplicate Substring](https://leetcode.com/problems/longest-duplicate-substring/) | Hard | Search the length, rolling hash to test it |

---

## 6. Stack & Monotonic Stack

Two tools.

A plain stack parses nested structure: parentheses, paths, calculators, undo.

A monotonic stack answers "the next greater element" for every index in linear time. The histogram, the contribution of each element as a minimum, and the "remove digits to make the smallest number" family are all that one loop.

String-building versions of the remove-digits stack ([316](https://leetcode.com/problems/remove-duplicate-letters/), [402](https://leetcode.com/problems/remove-k-digits/)) are under [Strings](#2-strings). The deque version of "window minimum" is under [Sliding Window](#4-sliding-window).

### Parse and simulate

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 1 | [20. Valid Parentheses](https://leetcode.com/problems/valid-parentheses/) | Easy | Push opens, match closes |
| 2 | [1614. Maximum Nesting Depth of the Parentheses](https://leetcode.com/problems/maximum-nesting-depth-of-the-parentheses/) | Easy | Depth counter is enough |
| 3 | [1021. Remove Outermost Parentheses](https://leetcode.com/problems/remove-outermost-parentheses/) | Easy | Depth, drop the outer pair |
| 4 | [1541. Minimum Insertions to Balance a Parentheses String](https://leetcode.com/problems/minimum-insertions-to-balance-a-parentheses-string/) | Medium | `(` needs two `)` |
| 5 | [1249. Minimum Remove to Make Valid Parentheses](https://leetcode.com/problems/minimum-remove-to-make-valid-parentheses/) | Medium | Index stack, delete the unmatched |
| 6 | [856. Score of Parentheses](https://leetcode.com/problems/score-of-parentheses/) | Medium | Score stack, or depth |
| 7 | [32. Longest Valid Parentheses](https://leetcode.com/problems/longest-valid-parentheses/) | Hard | Index stack, or two balance scans |
| 8 | [301. Remove Invalid Parentheses](https://leetcode.com/problems/remove-invalid-parentheses/) | Hard | BFS or backtracking, prune by the extra count |
| 9 | [1190. Reverse Substrings Between Each Pair of Parentheses](https://leetcode.com/problems/reverse-substrings-between-each-pair-of-parentheses/) | Medium | Stack of segments |
| 10 | [394. Decode String](https://leetcode.com/problems/decode-string/) | Medium | Count stack and string stack |
| 11 | [71. Simplify Path](https://leetcode.com/problems/simplify-path/) | Medium | Split, `..` pops |
| 12 | [150. Evaluate Reverse Polish Notation](https://leetcode.com/problems/evaluate-reverse-polish-notation/) | Medium | Operand stack |
| 13 | [227. Basic Calculator II](https://leetcode.com/problems/basic-calculator-ii/) | Medium | Deferred `+` and `-`, immediate `*` and `/` |
| 14 | [224. Basic Calculator](https://leetcode.com/problems/basic-calculator/) | Hard | Sign stack across parentheses |
| 15 | [772. Basic Calculator III](https://leetcode.com/problems/basic-calculator-iii/) | Hard | `+ - * /` and parentheses. Premium |
| 16 | [726. Number of Atoms](https://leetcode.com/problems/number-of-atoms/) | Hard | Map stack, multiply on `)` |
| 17 | [636. Exclusive Time of Functions](https://leetcode.com/problems/exclusive-time-of-functions/) | Medium | Start-time stack, subtract the child |
| 18 | [1130. Minimum Cost Tree From Leaf Values](https://leetcode.com/problems/minimum-cost-tree-from-leaf-values/) | Medium | Pop while the next leaf is larger |

### Design and collision

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 19 | [155. Min Stack](https://leetcode.com/problems/min-stack/) | Medium | Parallel stack of minima |
| 20 | [716. Max Stack](https://leetcode.com/problems/max-stack/) | Hard | Stack plus a way to delete the max. Premium |
| 21 | [232. Implement Queue using Stacks](https://leetcode.com/problems/implement-queue-using-stacks/) | Easy | Input stack and output stack |
| 22 | [225. Implement Stack using Queues](https://leetcode.com/problems/implement-stack-using-queues/) | Easy | Push is the expensive direction |
| 23 | [1381. Design a Stack With Increment Operation](https://leetcode.com/problems/design-a-stack-with-increment-operation/) | Medium | Lazy increment on the bottom k |
| 24 | [895. Maximum Frequency Stack](https://leetcode.com/problems/maximum-frequency-stack/) | Hard | Stack per frequency |
| 25 | [946. Validate Stack Sequences](https://leetcode.com/problems/validate-stack-sequences/) | Medium | Simulate the pushes |
| 26 | [735. Asteroid Collision](https://leetcode.com/problems/asteroid-collision/) | Medium | Stack, right-movers wait |
| 27 | [1047. Remove All Adjacent Duplicates In String](https://leetcode.com/problems/remove-all-adjacent-duplicates-in-string/) | Easy | Pop the match |
| 28 | [1544. Make The String Great](https://leetcode.com/problems/make-the-string-great/) | Easy | Pop the case-swap pair |
| 29 | [1209. Remove All Adjacent Duplicates in String II](https://leetcode.com/problems/remove-all-adjacent-duplicates-in-string-ii/) | Medium | Stack of (char, count) |
| 30 | [2390. Removing Stars From a String](https://leetcode.com/problems/removing-stars-from-a-string/) | Medium | Star pops |
| 31 | [1003. Check If Word Is Valid After Substitutions](https://leetcode.com/problems/check-if-word-is-valid-after-substitutions/) | Medium | Reduce `abc` |

### Next greater, and what that element contributes

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 32 | [496. Next Greater Element I](https://leetcode.com/problems/next-greater-element-i/) | Easy | Monotonic stack, then a map |
| 33 | [503. Next Greater Element II](https://leetcode.com/problems/next-greater-element-ii/) | Medium | Circular, scan twice |
| 34 | [739. Daily Temperatures](https://leetcode.com/problems/daily-temperatures/) | Medium | Distance to the next warmer day |
| 35 | [1475. Final Prices With a Special Discount in a Shop](https://leetcode.com/problems/final-prices-with-a-special-discount-in-a-shop/) | Easy | Next smaller or equal |
| 36 | [1019. Next Greater Node In Linked List](https://leetcode.com/problems/next-greater-node-in-linked-list/) | Medium | Same stack, on a list |
| 37 | [901. Online Stock Span](https://leetcode.com/problems/online-stock-span/) | Medium | Stack of (price, span) |
| 38 | [456. 132 Pattern](https://leetcode.com/problems/132-pattern/) | Medium | Stack of candidates for the `3`, track the `2` |
| 39 | [962. Maximum Width Ramp](https://leetcode.com/problems/maximum-width-ramp/) | Medium | Decreasing stack of indexes, then scan right to left |
| 40 | [853. Car Fleet](https://leetcode.com/problems/car-fleet/) | Medium | Sort by position, stack arrival times |
| 41 | [1776. Car Fleet II](https://leetcode.com/problems/car-fleet-ii/) | Hard | Collision time with the next car that is still alive |
| 42 | [1944. Number of Visible People in a Queue](https://leetcode.com/problems/number-of-visible-people-in-a-queue/) | Hard | Next strictly taller, count who you pass |
| 43 | [84. Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/) | Hard | Nearest smaller on both sides |
| 44 | [85. Maximal Rectangle](https://leetcode.com/problems/maximal-rectangle/) | Hard | Histogram per row, then 84 |
| 45 | [907. Sum of Subarray Minimums](https://leetcode.com/problems/sum-of-subarray-minimums/) | Hard | Each index is the min of `(left+1)*(right+1)` windows |
| 46 | [2104. Sum of Subarray Ranges](https://leetcode.com/problems/sum-of-subarray-ranges/) | Medium | Sum of maxes minus sum of mins |
| 47 | [1856. Maximum Subarray Min-Product](https://leetcode.com/problems/maximum-subarray-min-product/) | Medium | Nearest smaller, times the prefix sum of the span |
| 48 | [2281. Sum of Total Strength of Wizards](https://leetcode.com/problems/sum-of-total-strength-of-wizards/) | Hard | Same span, weighted by the prefix of prefixes |
| 49 | [1673. Find the Most Competitive Subsequence](https://leetcode.com/problems/find-the-most-competitive-subsequence/) | Medium | Keep a growing stack, pop while a smaller digit still fits |
| 50 | [321. Create Maximum Number](https://leetcode.com/problems/create-maximum-number/) | Hard | Drop digits to length i, then merge like merge-sort |

---

## 7. Linked Lists

Almost every list problem is one of: reverse a span, find a node with two pointers, merge, or splice. The design problems at the end are linked lists wearing another name. `O(1)` delete and `O(1)` insert in the middle is a doubly linked list plus a hash map.

[23. Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) is under [Heaps](#10-heaps). The two-pointer cycle problems on arrays ([202](https://leetcode.com/problems/happy-number/), [457](https://leetcode.com/problems/circular-array-loop/)) are under [Two Pointers](#3-two-pointers).

### Reverse and reorder

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 1 | [206. Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/) | Easy | prev, cur, next |
| 2 | [92. Reverse Linked List II](https://leetcode.com/problems/reverse-linked-list-ii/) | Medium | Reverse a span, reconnect both ends |
| 3 | [25. Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/) | Hard | Reverse only when k nodes remain |
| 4 | [24. Swap Nodes in Pairs](https://leetcode.com/problems/swap-nodes-in-pairs/) | Medium | k = 2 |
| 5 | [1721. Swapping Nodes in a Linked List](https://leetcode.com/problems/swapping-nodes-in-a-linked-list/) | Medium | Swap values, or splice both nodes |
| 6 | [143. Reorder List](https://leetcode.com/problems/reorder-list/) | Medium | Mid, reverse the second half, merge |
| 7 | [61. Rotate List](https://leetcode.com/problems/rotate-list/) | Medium | Make it circular, cut at n - k |

### Fast / slow and pointers

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 8 | [876. Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list/) | Easy | Slow and fast |
| 9 | [2095. Delete the Middle Node of a Linked List](https://leetcode.com/problems/delete-the-middle-node-of-a-linked-list/) | Medium | Slow, and the node before slow |
| 10 | [19. Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/) | Medium | Gap of n, dummy head |
| 11 | [141. Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/) | Easy | Floyd, meet or not |
| 12 | [142. Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/) | Medium | Meet, then reset one pointer to the head |
| 13 | [234. Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list/) | Easy | Mid, reverse, compare |
| 14 | [160. Intersection of Two Linked Lists](https://leetcode.com/problems/intersection-of-two-linked-lists/) | Easy | Switch heads, equalize the lengths |
| 15 | [2130. Maximum Twin Sum of a Linked List](https://leetcode.com/problems/maximum-twin-sum-of-a-linked-list/) | Medium | Reverse the second half, pair |

### Merge, sort, partition

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 16 | [21. Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/) | Easy | Dummy, take the smaller |
| 17 | [148. Sort List](https://leetcode.com/problems/sort-list/) | Medium | Merge sort, find mid, merge |
| 18 | [147. Insertion Sort List](https://leetcode.com/problems/insertion-sort-list/) | Medium | Insert into a growing sorted prefix |
| 19 | [2046. Sort Linked List Already Sorted Using Absolute Values](https://leetcode.com/problems/sort-linked-list-already-sorted-using-absolute-values/) | Medium | Negatives to the front by prepending |
| 20 | [86. Partition List](https://leetcode.com/problems/partition-list/) | Medium | Two dummy lists, concatenate |
| 21 | [328. Odd Even Linked List](https://leetcode.com/problems/odd-even-linked-list/) | Medium | Odd chain and even chain |
| 22 | [83. Remove Duplicates from Sorted List](https://leetcode.com/problems/remove-duplicates-from-sorted-list/) | Easy | Skip while the next value matches |
| 23 | [82. Remove Duplicates from Sorted List II](https://leetcode.com/problems/remove-duplicates-from-sorted-list-ii/) | Medium | Dummy, skip the whole run |
| 24 | [109. Convert Sorted List to Binary Search Tree](https://leetcode.com/problems/convert-sorted-list-to-binary-search-tree/) | Medium | Mid is the root. Slow/fast, or an index |

### Edit and arithmetic

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 25 | [203. Remove Linked List Elements](https://leetcode.com/problems/remove-linked-list-elements/) | Easy | Dummy head |
| 26 | [237. Delete Node in a Linked List](https://leetcode.com/problems/delete-node-in-a-linked-list/) | Easy | Copy the next value, skip the next node |
| 27 | [2487. Remove Nodes From Linked List](https://leetcode.com/problems/remove-nodes-from-linked-list/) | Medium | Reverse, keep a decreasing stack, reverse back |
| 28 | [1171. Remove Zero Sum Consecutive Nodes from Linked List](https://leetcode.com/problems/remove-zero-sum-consecutive-nodes-from-linked-list/) | Medium | Prefix sum, last node with that sum |
| 29 | [1669. Merge In Between Linked Lists](https://leetcode.com/problems/merge-in-between-linked-lists/) | Medium | Splice out `[a, b]` |
| 30 | [725. Split Linked List in Parts](https://leetcode.com/problems/split-linked-list-in-parts/) | Medium | Width and remainder |
| 31 | [2. Add Two Numbers](https://leetcode.com/problems/add-two-numbers/) | Medium | Add from the head, the lists are reversed |
| 32 | [445. Add Two Numbers II](https://leetcode.com/problems/add-two-numbers-ii/) | Medium | Stacks, or reverse both |
| 33 | [369. Plus One Linked List](https://leetcode.com/problems/plus-one-linked-list/) | Medium | Rightmost non-9. Premium |
| 34 | [2816. Double a Number Represented as a Linked List](https://leetcode.com/problems/double-a-number-represented-as-a-linked-list/) | Medium | Double, carry from the right. Or reverse |
| 35 | [2807. Insert Greatest Common Divisors in Linked List](https://leetcode.com/problems/insert-greatest-common-divisors-in-linked-list/) | Medium | Insert between every pair |
| 36 | [1290. Convert Binary Number in a Linked List to Integer](https://leetcode.com/problems/convert-binary-number-in-a-linked-list-to-integer/) | Easy | `ans = ans * 2 + bit` |
| 37 | [817. Linked List Components](https://leetcode.com/problems/linked-list-components/) | Medium | Count starts of a component |
| 38 | [2058. Find the Minimum and Maximum Number of Nodes Between Critical Points](https://leetcode.com/problems/find-the-minimum-and-maximum-number-of-nodes-between-critical-points/) | Medium | Local min and max, track indexes |
| 39 | [2181. Merge Nodes in Between Zeros](https://leetcode.com/problems/merge-nodes-in-between-zeros/) | Medium | Sum each segment between zeros |
| 40 | [382. Linked List Random Node](https://leetcode.com/problems/linked-list-random-node/) | Medium | Reservoir sampling, one pass |

### Copy, flatten, design

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 41 | [138. Copy List with Random Pointer](https://leetcode.com/problems/copy-list-with-random-pointer/) | Medium | Interleave copies, or a hash map |
| 42 | [430. Flatten a Multilevel Doubly Linked List](https://leetcode.com/problems/flatten-a-multilevel-doubly-linked-list/) | Medium | Child becomes the next span |
| 43 | [707. Design Linked List](https://leetcode.com/problems/design-linked-list/) | Medium | Dummy, size, get to index |
| 44 | [1472. Design Browser History](https://leetcode.com/problems/design-browser-history/) | Medium | Doubly linked, or an array plus an index |
| 45 | [146. LRU Cache](https://leetcode.com/problems/lru-cache/) | Medium | Hash map + doubly linked list |
| 46 | [460. LFU Cache](https://leetcode.com/problems/lfu-cache/) | Hard | Freq to a list of keys, plus min freq |
| 47 | [432. All O`one Data Structure](https://leetcode.com/problems/all-oone-data-structure/) | Hard | Count buckets in a doubly linked list |
| 48 | [1206. Design Skiplist](https://leetcode.com/problems/design-skiplist/) | Hard | Layers of linked lists |
| 49 | [708. Insert into a Sorted Circular Linked List](https://leetcode.com/problems/insert-into-a-sorted-circular-linked-list/) | Medium | Find the seam, including the wrap. Premium |
| 50 | [426. Convert Binary Search Tree to Sorted Doubly Linked List](https://leetcode.com/problems/convert-binary-search-tree-to-sorted-doubly-linked-list/) | Medium | Inorder, stitch prev to cur. Premium |

---

## 8. Trees & BST

A tree problem is a function that returns something to its parent. Decide the return value before you write the recursion: a number, a boolean, a node, or a pair.

Tries are under [Strings](#2-strings). They are trees, but the problems are about prefixes.

### Traversal and view

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 1 | [144. Binary Tree Preorder Traversal](https://leetcode.com/problems/binary-tree-preorder-traversal/) | Easy | Root, left, right. Then the stack version |
| 2 | [94. Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/) | Easy | Left, root, right |
| 3 | [145. Binary Tree Postorder Traversal](https://leetcode.com/problems/binary-tree-postorder-traversal/) | Easy | Left, right, root |
| 4 | [102. Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/) | Medium | BFS, level size |
| 5 | [103. Binary Tree Zigzag Level Order Traversal](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/) | Medium | Alternate direction per level |
| 6 | [199. Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/) | Medium | Last node of each level |
| 7 | [513. Find Bottom Left Tree Value](https://leetcode.com/problems/find-bottom-left-tree-value/) | Medium | First node of the last level |
| 8 | [662. Maximum Width of Binary Tree](https://leetcode.com/problems/maximum-width-of-binary-tree/) | Medium | Heap indexes, width is right - left |
| 9 | [116. Populating Next Right Pointers in Each Node](https://leetcode.com/problems/populating-next-right-pointers-in-each-node/) | Medium | Perfect tree, use the next pointers you just set |
| 10 | [117. Populating Next Right Pointers in Each Node II](https://leetcode.com/problems/populating-next-right-pointers-in-each-node-ii/) | Medium | Same, the tree is not perfect |

### Shape

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 11 | [104. Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/) | Easy | 1 + max of children |
| 12 | [111. Minimum Depth of Binary Tree](https://leetcode.com/problems/minimum-depth-of-binary-tree/) | Easy | A missing child is not a leaf |
| 13 | [543. Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/) | Easy | Height returned, diameter stored on the side |
| 14 | [110. Balanced Binary Tree](https://leetcode.com/problems/balanced-binary-tree/) | Easy | Height, or -1 if unbalanced |
| 15 | [100. Same Tree](https://leetcode.com/problems/same-tree/) | Easy | Both null, or values match and children match |
| 16 | [101. Symmetric Tree](https://leetcode.com/problems/symmetric-tree/) | Easy | Outside pair and inside pair |
| 17 | [572. Subtree of Another Tree](https://leetcode.com/problems/subtree-of-another-tree/) | Easy | Same-tree at every node |
| 18 | [226. Invert Binary Tree](https://leetcode.com/problems/invert-binary-tree/) | Easy | Swap children |
| 19 | [617. Merge Two Binary Trees](https://leetcode.com/problems/merge-two-binary-trees/) | Easy | Add, recurse on both |
| 20 | [951. Flip Equivalent Binary Trees](https://leetcode.com/problems/flip-equivalent-binary-trees/) | Medium | Try both child pairings |

### Paths and ancestors

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 21 | [112. Path Sum](https://leetcode.com/problems/path-sum/) | Easy | Subtract as you go |
| 22 | [113. Path Sum II](https://leetcode.com/problems/path-sum-ii/) | Medium | Backtracking, copy the path at a leaf |
| 23 | [257. Binary Tree Paths](https://leetcode.com/problems/binary-tree-paths/) | Easy | Build the string on the way down |
| 24 | [129. Sum Root to Leaf Numbers](https://leetcode.com/problems/sum-root-to-leaf-numbers/) | Medium | `num * 10 + digit` |
| 25 | [988. Smallest String Starting From Leaf](https://leetcode.com/problems/smallest-string-starting-from-leaf/) | Medium | Compare at the leaves |
| 26 | [437. Path Sum III](https://leetcode.com/problems/path-sum-iii/) | Medium | Prefix sums on the path down |
| 27 | [124. Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/) | Hard | Gain from each child, path through the node is a side answer |
| 28 | [236. Lowest Common Ancestor of a Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/) | Medium | Return the node when both sides report a find |
| 29 | [865. Smallest Subtree with all the Deepest Nodes](https://leetcode.com/problems/smallest-subtree-with-all-the-deepest-nodes/) | Medium | Return (node, depth) |
| 30 | [1123. Lowest Common Ancestor of Deepest Leaves](https://leetcode.com/problems/lowest-common-ancestor-of-deepest-leaves/) | Medium | Same return pair |

### Build, prune, enumerate

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 31 | [105. Construct Binary Tree from Preorder and Inorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/) | Medium | Preorder gives the root, inorder splits |
| 32 | [106. Construct Binary Tree from Inorder and Postorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/) | Medium | Postorder gives the root from the end |
| 33 | [889. Construct Binary Tree from Preorder and Postorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-postorder-traversal/) | Medium | Left subtree size from preorder |
| 34 | [297. Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/) | Hard | Preorder with null markers |
| 35 | [114. Flatten Binary Tree to Linked List](https://leetcode.com/problems/flatten-binary-tree-to-linked-list/) | Medium | Right spine, reverse postorder |
| 36 | [814. Binary Tree Pruning](https://leetcode.com/problems/binary-tree-pruning/) | Medium | Keep a node if it or a child has a 1 |
| 37 | [1325. Delete Leaves With a Given Value](https://leetcode.com/problems/delete-leaves-with-a-given-value/) | Medium | Prune after the children, a node can become a leaf |
| 38 | [108. Convert Sorted Array to Binary Search Tree](https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/) | Easy | Mid is the root |
| 39 | [95. Unique Binary Search Trees II](https://leetcode.com/problems/unique-binary-search-trees-ii/) | Medium | Every i as root, cartesian product of sides |
| 40 | [894. All Possible Full Binary Trees](https://leetcode.com/problems/all-possible-full-binary-trees/) | Medium | Odd node counts, combine left and right |

### BST

The invariant is the whole job: everything left is smaller, everything right is larger. Pass the allowed range down. Do not only check the parent.

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 41 | [98. Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/) | Medium | Pass `(low, high)` |
| 42 | [700. Search in a Binary Search Tree](https://leetcode.com/problems/search-in-a-binary-search-tree/) | Easy | Go left or right |
| 43 | [701. Insert into a Binary Search Tree](https://leetcode.com/problems/insert-into-a-binary-search-tree/) | Medium | Walk to a null child |
| 44 | [450. Delete Node in a BST](https://leetcode.com/problems/delete-node-in-a-bst/) | Medium | 0, 1, or 2 children. Successor replaces 2 |
| 45 | [235. Lowest Common Ancestor of a Binary Search Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/) | Medium | First node between the two values |
| 46 | [230. Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/) | Medium | Inorder, stop at k |
| 47 | [538. Convert BST to Greater Tree](https://leetcode.com/problems/convert-bst-to-greater-tree/) | Medium | Reverse inorder, running sum |
| 48 | [669. Trim a Binary Search Tree](https://leetcode.com/problems/trim-a-binary-search-tree/) | Medium | Cut a side, or replace the node with the kept child |
| 49 | [173. Binary Search Tree Iterator](https://leetcode.com/problems/binary-search-tree-iterator/) | Medium | Stack of the left spine |
| 50 | [653. Two Sum IV - Input is a BST](https://leetcode.com/problems/two-sum-iv-input-is-a-bst/) | Easy | Inorder into a set, or two iterators |

[99. Recover Binary Search Tree](https://leetcode.com/problems/recover-binary-search-tree/) is the natural next problem after 98: inorder should be sorted, so the swapped pair shows up as one or two drops. It did not fit in the 50.

---

## 9. Graphs & Union-Find

Grid search, multi-source BFS, topological order, shortest paths with a cost, and disjoint sets. Dijkstra shows up again under [Heaps](#10-heaps) only when the heap is the thing being practiced. The problems below are here because the graph is the thing being practiced.

### Grid search

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 1 | [733. Flood Fill](https://leetcode.com/problems/flood-fill/) | Easy | DFS or BFS recolor |
| 2 | [200. Number of Islands](https://leetcode.com/problems/number-of-islands/) | Medium | Count components, sink the land as you go |
| 3 | [695. Max Area of Island](https://leetcode.com/problems/max-area-of-island/) | Medium | Size of a component |
| 4 | [463. Island Perimeter](https://leetcode.com/problems/island-perimeter/) | Easy | 4 minus 2 per shared edge |
| 5 | [130. Surrounded Regions](https://leetcode.com/problems/surrounded-regions/) | Medium | Mark border-connected `O`, then flip |
| 6 | [1020. Number of Enclaves](https://leetcode.com/problems/number-of-enclaves/) | Medium | Land that cannot reach the border |
| 7 | [1254. Number of Closed Islands](https://leetcode.com/problems/number-of-closed-islands/) | Medium | Island not touching the border |
| 8 | [1905. Count Sub Islands](https://leetcode.com/problems/count-sub-islands/) | Medium | Every land cell of B must be land in A |
| 9 | [417. Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/) | Medium | Reachability from both oceans, uphill |
| 10 | [994. Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) | Medium | Multi-source BFS, the minute is the layer |
| 11 | [542. 01 Matrix](https://leetcode.com/problems/01-matrix/) | Medium | Multi-source BFS from every 0 |
| 12 | [1091. Shortest Path in Binary Matrix](https://leetcode.com/problems/shortest-path-in-binary-matrix/) | Medium | 8-direction BFS |
| 13 | [934. Shortest Bridge](https://leetcode.com/problems/shortest-bridge/) | Medium | DFS one island, BFS until the other |
| 14 | [959. Regions Cut By Slashes](https://leetcode.com/problems/regions-cut-by-slashes/) | Medium | Union the four triangles of each cell |
| 15 | [827. Making A Large Island](https://leetcode.com/problems/making-a-large-island/) | Hard | Component sizes, try flipping each 0 |
| 16 | [329. Longest Increasing Path in a Matrix](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/) | Hard | DFS with memo. The graph is a DAG |

### Implicit graphs, topo, coloring

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 17 | [133. Clone Graph](https://leetcode.com/problems/clone-graph/) | Medium | DFS, map original node to copy |
| 18 | [841. Keys and Rooms](https://leetcode.com/problems/keys-and-rooms/) | Medium | Visit every room you can unlock |
| 19 | [1192. Critical Connections in a Network](https://leetcode.com/problems/critical-connections-in-a-network/) | Hard | Tarjan, bridges |
| 20 | [332. Reconstruct Itinerary](https://leetcode.com/problems/reconstruct-itinerary/) | Hard | Hierholzer, Euler path. Postorder the edges |
| 21 | [127. Word Ladder](https://leetcode.com/problems/word-ladder/) | Hard | BFS, one letter away. Bidirectional is the speedup |
| 22 | [433. Minimum Genetic Mutation](https://leetcode.com/problems/minimum-genetic-mutation/) | Medium | Word ladder on a bank |
| 23 | [752. Open the Lock](https://leetcode.com/problems/open-the-lock/) | Medium | BFS on 4 digits, plus deadends |
| 24 | [909. Snakes and Ladders](https://leetcode.com/problems/snakes-and-ladders/) | Medium | BFS, teleport on the destination cell |
| 25 | [399. Evaluate Division](https://leetcode.com/problems/evaluate-division/) | Medium | Weighted edges, DFS the product |
| 26 | [207. Course Schedule](https://leetcode.com/problems/course-schedule/) | Medium | Cycle check, Kahn or colors |
| 27 | [210. Course Schedule II](https://leetcode.com/problems/course-schedule-ii/) | Medium | Kahn, record the order |
| 28 | [802. Find Eventual Safe States](https://leetcode.com/problems/find-eventual-safe-states/) | Medium | Reverse edges, or 3-color DFS |
| 29 | [1462. Course Schedule IV](https://leetcode.com/problems/course-schedule-iv/) | Medium | Reachability after the topo, Floyd if it is small |
| 30 | [1466. Reorder Routes to Make All Paths Lead to the City Zero](https://leetcode.com/problems/reorder-routes-to-make-all-paths-lead-to-the-city-zero/) | Medium | DFS from 0, count edges that point away |
| 31 | [310. Minimum Height Trees](https://leetcode.com/problems/minimum-height-trees/) | Medium | Peel leaves until 1 or 2 centers remain |
| 32 | [785. Is Graph Bipartite?](https://leetcode.com/problems/is-graph-bipartite/) | Medium | 2-color |
| 33 | [886. Possible Bipartition](https://leetcode.com/problems/possible-bipartition/) | Medium | Dislikes are edges, then 2-color |
| 34 | [269. Alien Dictionary](https://leetcode.com/problems/alien-dictionary/) | Hard | Compare adjacent words, topo the letters. Premium |
| 35 | [1857. Largest Color Value in a Directed Graph](https://leetcode.com/problems/largest-color-value-in-a-directed-graph/) | Hard | Topo plus a color count on the path |

### Shortest path with a cost

Unweighted graphs use BFS. A weight, a hop limit, or a state (keys held, obstacles left) needs Dijkstra or a Bellman-style relaxation. If you only remember one implementation, remember a min-heap of `(dist, node)`.

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 36 | [743. Network Delay Time](https://leetcode.com/problems/network-delay-time/) | Medium | Dijkstra, answer is the max dist |
| 37 | [787. Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/) | Medium | Bellman, relax at most k+1 times. Or Dijkstra with stops in the state |
| 38 | [1514. Path with Maximum Probability](https://leetcode.com/problems/path-with-maximum-probability/) | Medium | Dijkstra, max-heap of probability |
| 39 | [1631. Path With Minimum Effort](https://leetcode.com/problems/path-with-minimum-effort/) | Medium | Dijkstra, cost is the max edge on the path |
| 40 | [778. Swim in Rising Water](https://leetcode.com/problems/swim-in-rising-water/) | Hard | Dijkstra, cost is the max height seen |
| 41 | [1293. Shortest Path in a Grid with Obstacles Elimination](https://leetcode.com/problems/shortest-path-in-a-grid-with-obstacles-elimination/) | Hard | BFS state is `(r, c, removals left)` |
| 42 | [847. Shortest Path Visiting All Nodes](https://leetcode.com/problems/shortest-path-visiting-all-nodes/) | Hard | BFS state is `(node, bitmask of visited)` |

### Union-Find and a minimum spanning tree

Parent array, rank or size, path compression. The MST problem at the end is Kruskal: sort edges, add one if it does not close a cycle.

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 43 | [547. Number of Provinces](https://leetcode.com/problems/number-of-provinces/) | Medium | Components in an adjacency matrix |
| 44 | [684. Redundant Connection](https://leetcode.com/problems/redundant-connection/) | Medium | The edge that joins two nodes already connected |
| 45 | [685. Redundant Connection II](https://leetcode.com/problems/redundant-connection-ii/) | Hard | A node with two parents, and maybe a cycle |
| 46 | [1319. Number of Operations to Make Network Connected](https://leetcode.com/problems/number-of-operations-to-make-network-connected/) | Medium | Components - 1, if you have spare cables |
| 47 | [721. Accounts Merge](https://leetcode.com/problems/accounts-merge/) | Medium | Union emails that appear together |
| 48 | [990. Satisfiability of Equality Equations](https://leetcode.com/problems/satisfiability-of-equality-equations/) | Medium | Union `==` first, then check `!=` |
| 49 | [947. Most Stones Removed with Same Row or Column](https://leetcode.com/problems/most-stones-removed-with-same-row-or-column/) | Medium | Stones minus components |
| 50 | [1584. Min Cost to Connect All Points](https://leetcode.com/problems/min-cost-to-connect-all-points/) | Medium | Kruskal on Manhattan edges |

Premium follow-ups that did not fit: [323. Number of Connected Components in an Undirected Graph](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/), [261. Graph Valid Tree](https://leetcode.com/problems/graph-valid-tree/), [305. Number of Islands II](https://leetcode.com/problems/number-of-islands-ii/).

---

## 10. Heaps

A heap is a way to repeatedly pull the current best. If you need all k of them and k is small, a heap of size k beats sorting. If you are merging sorted lists, the heap holds one head from each list. If you need a running median, you need two heaps.

### Top K and "do the best thing k times"

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 1 | [703. Kth Largest Element in a Stream](https://leetcode.com/problems/kth-largest-element-in-a-stream/) | Easy | Min-heap of size k |
| 2 | [215. Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/) | Medium | Heap of size k, or quickselect |
| 3 | [1985. Find the Kth Largest Integer in the Array](https://leetcode.com/problems/find-the-kth-largest-integer-in-the-array/) | Medium | Same, the numbers are strings |
| 4 | [973. K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/) | Medium | Max-heap of size k by distance |
| 5 | [692. Top K Frequent Words](https://leetcode.com/problems/top-k-frequent-words/) | Medium | Heap with a tie break on the word |
| 6 | [1046. Last Stone Weight](https://leetcode.com/problems/last-stone-weight/) | Easy | Max-heap, smash the top two |
| 7 | [1962. Remove Stones to Minimize the Total](https://leetcode.com/problems/remove-stones-to-minimize-the-total/) | Medium | Always halve the current largest |
| 8 | [2558. Take Gifts From the Richest Pile](https://leetcode.com/problems/take-gifts-from-the-richest-pile/) | Easy | Replace the max with its square root, k times |
| 9 | [2208. Minimum Operations to Halve Array Sum](https://leetcode.com/problems/minimum-operations-to-halve-array-sum/) | Medium | Halve the current largest until the sum drops enough |
| 10 | [2233. Maximum Product After K Increments](https://leetcode.com/problems/maximum-product-after-k-increments/) | Medium | Always increment the current minimum |
| 11 | [2530. Maximal Score After Applying K Operations](https://leetcode.com/problems/maximal-score-after-applying-k-operations/) | Medium | Take the max, write `ceil(x/3)` back |
| 12 | [2335. Minimum Amount of Time to Fill Cups](https://leetcode.com/problems/minimum-amount-of-time-to-fill-cups/) | Easy | Two largest cups each second |

### K-way merge

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 13 | [23. Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) | Hard | Heap of the current head of each list |
| 14 | [373. Find K Pairs with Smallest Sums](https://leetcode.com/problems/find-k-pairs-with-smallest-sums/) | Medium | Heap, push the neighbor when you pop |
| 15 | [786. K-th Smallest Prime Fraction](https://leetcode.com/problems/k-th-smallest-prime-fraction/) | Medium | Same k-way idea, or binary search the value |
| 16 | [632. Smallest Range Covering Elements from K Lists](https://leetcode.com/problems/smallest-range-covering-elements-from-k-lists/) | Hard | One pointer per list, heap tracks the min |
| 17 | [1439. Find the Kth Smallest Sum of a Matrix With Sorted Rows](https://leetcode.com/problems/find-the-kth-smallest-sum-of-a-matrix-with-sorted-rows/) | Hard | Merge row by row, keep only k partial sums |
| 18 | [264. Ugly Number II](https://leetcode.com/problems/ugly-number-ii/) | Medium | Three pointers, or a heap of `2x 3x 5x` |
| 19 | [313. Super Ugly Number](https://leetcode.com/problems/super-ugly-number/) | Medium | One pointer per prime |

### Two heaps, and greedy that needs "the current best"

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 20 | [295. Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/) | Hard | Max-heap of the low half, min-heap of the high half |
| 21 | [1675. Minimize Deviation in Array](https://leetcode.com/problems/minimize-deviation-in-array/) | Hard | Push the max even down, track the min |
| 22 | [502. IPO](https://leetcode.com/problems/ipo/) | Hard | Affordable projects in a max-heap of profit |
| 23 | [1383. Maximum Performance of a Team](https://leetcode.com/problems/maximum-performance-of-a-team/) | Hard | Sort by efficiency, heap of the speeds you keep |
| 24 | [857. Minimum Cost to Hire K Workers](https://leetcode.com/problems/minimum-cost-to-hire-k-workers/) | Hard | Sort by wage/quality, heap of qualities |
| 25 | [2542. Maximum Subsequence Score](https://leetcode.com/problems/maximum-subsequence-score/) | Medium | Sort by one array, heap the other |
| 26 | [630. Course Schedule III](https://leetcode.com/problems/course-schedule-iii/) | Hard | Sort by end day, swap out the longest course |
| 27 | [621. Task Scheduler](https://leetcode.com/problems/task-scheduler/) | Medium | Heap of frequencies, or the idle formula |
| 28 | [1405. Longest Happy String](https://leetcode.com/problems/longest-happy-string/) | Medium | Always append the most frequent letter you are allowed to |
| 29 | [1054. Distant Barcodes](https://leetcode.com/problems/distant-barcodes/) | Medium | No two equal barcodes adjacent |
| 30 | [1792. Maximum Average Pass Ratio](https://leetcode.com/problems/maximum-average-pass-ratio/) | Medium | Heap by the gain of adding one student |
| 31 | [1642. Furthest Building You Can Reach](https://leetcode.com/problems/furthest-building-you-can-reach/) | Medium | Ladders on the largest climbs, bricks on the rest |
| 32 | [871. Minimum Number of Refueling Stops](https://leetcode.com/problems/minimum-number-of-refueling-stops/) | Hard | Drive, heap the stations you passed |
| 33 | [1834. Single-Threaded CPU](https://leetcode.com/problems/single-threaded-cpu/) | Medium | Available tasks in a heap by processing time |
| 34 | [1882. Process Tasks Using Servers](https://leetcode.com/problems/process-tasks-using-servers/) | Medium | Free servers and busy servers, two heaps |
| 35 | [2402. Meeting Rooms III](https://leetcode.com/problems/meeting-rooms-iii/) | Hard | Free rooms, and rooms by when they free |
| 36 | [1353. Maximum Number of Events That Can Be Attended](https://leetcode.com/problems/maximum-number-of-events-that-can-be-attended/) | Medium | Day by day, heap of events that end soonest |
| 37 | [1801. Number of Orders in the Backlog](https://leetcode.com/problems/number-of-orders-in-the-backlog/) | Medium | Buy max-heap, sell min-heap |
| 38 | [2462. Total Cost to Hire K Workers](https://leetcode.com/problems/total-cost-to-hire-k-workers/) | Medium | Two ends, one heap |
| 39 | [1942. The Number of the Smallest Unoccupied Chair](https://leetcode.com/problems/the-number-of-the-smallest-unoccupied-chair/) | Medium | Free chairs, and when each chair frees |
| 40 | [1845. Seat Reservation Manager](https://leetcode.com/problems/seat-reservation-manager/) | Medium | Min-heap of unreserved seats |
| 41 | [2336. Smallest Number in Infinite Set](https://leetcode.com/problems/smallest-number-in-infinite-set/) | Medium | Heap of added-back numbers, plus a cursor |
| 42 | [407. Trapping Rain Water II](https://leetcode.com/problems/trapping-rain-water-ii/) | Hard | Heap on the border, walk inward |
| 43 | [1354. Construct Target Array With Multiple Sums](https://leetcode.com/problems/construct-target-array-with-multiple-sums/) | Hard | Reverse the operation, max must have been the sum |
| 44 | [1705. Maximum Number of Eaten Apples](https://leetcode.com/problems/maximum-number-of-eaten-apples/) | Medium | Heap of batches by expiry |
| 45 | [1167. Minimum Cost to Connect Sticks](https://leetcode.com/problems/minimum-cost-to-connect-sticks/) | Medium | Huffman, always join the two shortest. Premium |
| 46 | [358. Rearrange String k Distance Apart](https://leetcode.com/problems/rearrange-string-k-distance-apart/) | Hard | Heap plus a cooldown queue. Premium |
| 47 | [505. The Maze II](https://leetcode.com/problems/the-maze-ii/) | Medium | Dijkstra, you roll until a wall. Premium |
| 48 | [499. The Maze III](https://leetcode.com/problems/the-maze-iii/) | Hard | Dijkstra, also track the path string. Premium |
| 49 | [882. Reachable Nodes In Subdivided Graph](https://leetcode.com/problems/reachable-nodes-in-subdivided-graph/) | Hard | Dijkstra, then count how far you got along each edge |
| 50 | [2642. Design Graph With Shortest Path Calculator](https://leetcode.com/problems/design-graph-with-shortest-path-calculator/) | Hard | Dijkstra per query, or Floyd if you add edges first |

Meeting-room heaps that are really about intervals ([253](https://leetcode.com/problems/meeting-rooms-ii/), [218](https://leetcode.com/problems/the-skyline-problem/), [759](https://leetcode.com/problems/employee-free-time/)) are under [Intervals](#13-intervals).

---

## 11. Backtracking

Choose, explore, undo. The state you pass down is the whole algorithm: the start index (combinations, no reuse of earlier items), a used array (permutations), or the board you mutate and restore.

Prune before you recurse. The partition problems at the end are the same search with a feasibility cut.

### Subsets, combinations, permutations

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 1 | [78. Subsets](https://leetcode.com/problems/subsets/) | Medium | Include or skip, start index |
| 2 | [90. Subsets II](https://leetcode.com/problems/subsets-ii/) | Medium | Sort, skip duplicate siblings |
| 3 | [491. Non-decreasing Subsequences](https://leetcode.com/problems/non-decreasing-subsequences/) | Medium | No sort allowed, a set of siblings instead |
| 4 | [1863. Sum of All Subset XOR Totals](https://leetcode.com/problems/sum-of-all-subset-xor-totals/) | Easy | Carry the XOR down |
| 5 | [2044. Count Number of Maximum Bitwise-OR Subsets](https://leetcode.com/problems/count-number-of-maximum-bitwise-or-subsets/) | Medium | Target OR is the OR of everything |
| 6 | [77. Combinations](https://leetcode.com/problems/combinations/) | Medium | Choose k from n, start index |
| 7 | [39. Combination Sum](https://leetcode.com/problems/combination-sum/) | Medium | Reuse the same index |
| 8 | [40. Combination Sum II](https://leetcode.com/problems/combination-sum-ii/) | Medium | Each element once, skip duplicates |
| 9 | [216. Combination Sum III](https://leetcode.com/problems/combination-sum-iii/) | Medium | k numbers from 1..9 |
| 10 | [46. Permutations](https://leetcode.com/problems/permutations/) | Medium | Used array, or swap into place |
| 11 | [47. Permutations II](https://leetcode.com/problems/permutations-ii/) | Medium | Sort, skip a duplicate that the previous sibling did not use |
| 12 | [60. Permutation Sequence](https://leetcode.com/problems/permutation-sequence/) | Hard | Factorial number system, do not generate them all |
| 13 | [784. Letter Case Permutation](https://leetcode.com/problems/letter-case-permutation/) | Medium | Flip or not, digits are fixed |
| 14 | [22. Generate Parentheses](https://leetcode.com/problems/generate-parentheses/) | Medium | Opens used, closes used, never more closes |
| 15 | [401. Binary Watch](https://leetcode.com/problems/binary-watch/) | Easy | Choose bits, hours < 12, minutes < 60 |
| 16 | [526. Beautiful Arrangement](https://leetcode.com/problems/beautiful-arrangement/) | Medium | Place i at a position that divides |
| 17 | [967. Numbers With Same Consecutive Differences](https://leetcode.com/problems/numbers-with-same-consecutive-differences/) | Medium | Append a digit at distance k |
| 18 | [1415. The k-th Lexicographical String of All Happy Strings of Length n](https://leetcode.com/problems/the-k-th-lexicographical-string-of-all-happy-strings-of-length-n/) | Medium | No two adjacent letters equal |
| 19 | [1079. Letter Tile Possibilities](https://leetcode.com/problems/letter-tile-possibilities/) | Medium | Counts, every length |
| 20 | [1286. Iterator for Combination](https://leetcode.com/problems/iterator-for-combination/) | Medium | Next combination in order |

### Boards, splits, and expressions

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 21 | [79. Word Search](https://leetcode.com/problems/word-search/) | Medium | DFS, mark the cell, restore |
| 22 | [1219. Path with Maximum Gold](https://leetcode.com/problems/path-with-maximum-gold/) | Medium | Same, collect the best path |
| 23 | [980. Unique Paths III](https://leetcode.com/problems/unique-paths-iii/) | Hard | Visit every empty cell exactly once |
| 24 | [51. N-Queens](https://leetcode.com/problems/n-queens/) | Hard | Columns, diagonals, anti-diagonals |
| 25 | [52. N-Queens II](https://leetcode.com/problems/n-queens-ii/) | Hard | Count only |
| 26 | [37. Sudoku Solver](https://leetcode.com/problems/sudoku-solver/) | Hard | First empty cell, try a legal digit |
| 27 | [131. Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/) | Medium | Cut where the piece is a palindrome |
| 28 | [140. Word Break II](https://leetcode.com/problems/word-break-ii/) | Hard | Cut on dictionary words. Memoize the suffixes |
| 29 | [282. Expression Add Operators](https://leetcode.com/problems/expression-add-operators/) | Hard | Track the last term so `*` can undo it |
| 30 | [842. Split Array into Fibonacci Sequence](https://leetcode.com/problems/split-array-into-fibonacci-sequence/) | Medium | Next number is the sum, no leading zero |
| 31 | [306. Additive Number](https://leetcode.com/problems/additive-number/) | Medium | Same check, the split is not returned |
| 32 | [1849. Splitting a String Into Descending Consecutive Values](https://leetcode.com/problems/splitting-a-string-into-descending-consecutive-values/) | Medium | Next value is previous - 1 |
| 33 | [1593. Split a String Into the Max Number of Unique Substrings](https://leetcode.com/problems/split-a-string-into-the-max-number-of-unique-substrings/) | Medium | Set of pieces used so far |
| 34 | [1239. Maximum Length of a Concatenated String with Unique Characters](https://leetcode.com/problems/maximum-length-of-a-concatenated-string-with-unique-characters/) | Medium | Bitmask of letters used |
| 35 | [1255. Maximum Score Words Formed by Letters](https://leetcode.com/problems/maximum-score-words-formed-by-letters/) | Hard | Take the word or not, letter counts |
| 36 | [996. Number of Squareful Arrays](https://leetcode.com/problems/number-of-squareful-arrays/) | Hard | Permutation, adjacent sum is square |
| 37 | [1718. Construct the Lexicographically Largest Valid Sequence](https://leetcode.com/problems/construct-the-lexicographically-largest-valid-sequence/) | Medium | Place the largest unused number into the first hole |
| 38 | [638. Shopping Offers](https://leetcode.com/problems/shopping-offers/) | Medium | Apply a special offer or buy one item |
| 39 | [2597. The Number of Beautiful Subsets](https://leetcode.com/problems/the-number-of-beautiful-subsets/) | Medium | No two differ by k |
| 40 | [1981. Minimize the Difference Between Target and Chosen Elements](https://leetcode.com/problems/minimize-the-difference-between-target-and-chosen-elements/) | Hard | One pick per row. Prune once the sum passes the target |

### Partition into groups

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 41 | [473. Matchsticks to Square](https://leetcode.com/problems/matchsticks-to-square/) | Medium | 4 sides, same sum. Sort descending |
| 42 | [698. Partition to K Equal Sum Subsets](https://leetcode.com/problems/partition-to-k-equal-sum-subsets/) | Medium | Fill one bucket, then the next |
| 43 | [1723. Find Minimum Time to Finish All Jobs](https://leetcode.com/problems/find-minimum-time-to-finish-all-jobs/) | Hard | Assign jobs to k workers, prune by the best time |
| 44 | [2305. Fair Distribution of Cookies](https://leetcode.com/problems/fair-distribution-of-cookies/) | Medium | Same assignment, minimize the max |
| 45 | [1986. Minimum Number of Work Sessions to Finish the Tasks](https://leetcode.com/problems/minimum-number-of-work-sessions-to-finish-the-tasks/) | Medium | Session time cap |
| 46 | [465. Optimal Account Balancing](https://leetcode.com/problems/optimal-account-balancing/) | Hard | Settle the non-zero balances. Premium |
| 47 | [254. Factor Combinations](https://leetcode.com/problems/factor-combinations/) | Medium | Factors in non-decreasing order. Premium |
| 48 | [489. Robot Room Cleaner](https://leetcode.com/problems/robot-room-cleaner/) | Hard | DFS with backtrack moves, the robot is the cursor. Premium |
| 49 | [351. Android Unlock Patterns](https://leetcode.com/problems/android-unlock-patterns/) | Medium | Knight-skip rules on the 9 dots. Premium |
| 50 | [294. Flip Game II](https://leetcode.com/problems/flip-game-ii/) | Medium | A move exists that leaves the opponent losing. Premium |

---

## 12. Greedy

A greedy solution is an exchange argument you can say out loud: swapping to this choice never makes the answer worse. If you cannot say that, it is probably DP or a search.

Interval scheduling and interval cover are under [Intervals](#13-intervals), even though they are greedy. Heap-backed greedy is under [Heaps](#10-heaps).

### Reach, exchange, and sort-then-scan

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 1 | [55. Jump Game](https://leetcode.com/problems/jump-game/) | Medium | Farthest reach |
| 2 | [45. Jump Game II](https://leetcode.com/problems/jump-game-ii/) | Medium | BFS layers over the index range |
| 3 | [134. Gas Station](https://leetcode.com/problems/gas-station/) | Medium | If the total works, start after the lowest prefix |
| 4 | [135. Candy](https://leetcode.com/problems/candy/) | Hard | Left-to-right, then right-to-left |
| 5 | [860. Lemonade Change](https://leetcode.com/problems/lemonade-change/) | Easy | Prefer giving a 10 back |
| 6 | [455. Assign Cookies](https://leetcode.com/problems/assign-cookies/) | Easy | Smallest cookie that satisfies |
| 7 | [1710. Maximum Units on a Truck](https://leetcode.com/problems/maximum-units-on-a-truck/) | Easy | Sort by units per box |
| 8 | [1005. Maximize Sum Of Array After K Negations](https://leetcode.com/problems/maximize-sum-of-array-after-k-negations/) | Easy | Flip the smallest, then the leftover flip hits the new smallest |
| 9 | [561. Array Partition](https://leetcode.com/problems/array-partition/) | Easy | Sort, sum every first of a pair |
| 10 | [1403. Minimum Subsequence in Non-Increasing Order](https://leetcode.com/problems/minimum-subsequence-in-non-increasing-order/) | Easy | Take largest until you pass the rest |
| 11 | [1561. Maximum Number of Coins You Can Get](https://leetcode.com/problems/maximum-number-of-coins-you-can-get/) | Medium | Sort, you take every second from the end |
| 12 | [1402. Reducing Dishes](https://leetcode.com/problems/reducing-dishes/) | Hard | Cook the best dishes, stop when the marginal sum drops |
| 13 | [1338. Reduce Array Size to The Half](https://leetcode.com/problems/reduce-array-size-to-the-half/) | Medium | Remove the most frequent values first |
| 14 | [1029. Two City Scheduling](https://leetcode.com/problems/two-city-scheduling/) | Medium | Sort by the cost delta |
| 15 | [1899. Merge Triplets to Form Target Triplet](https://leetcode.com/problems/merge-triplets-to-form-target-triplet/) | Medium | A triplet that exceeds the target is useless |
| 16 | [1921. Eliminate Maximum Number of Monsters](https://leetcode.com/problems/eliminate-maximum-number-of-monsters/) | Medium | Sort by arrival time, you fire once a minute |
| 17 | [1686. Stone Game VI](https://leetcode.com/problems/stone-game-vi/) | Medium | Value of taking a stone is `a[i] + b[i]` |
| 18 | [714. Best Time to Buy and Sell Stock with Transaction Fee](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/) | Medium | Hold or cash, one pass |

### Digits, ranges you must cover, local fixes

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 19 | [179. Largest Number](https://leetcode.com/problems/largest-number/) | Medium | Compare `a+b` against `b+a` |
| 20 | [670. Maximum Swap](https://leetcode.com/problems/maximum-swap/) | Medium | Rightmost larger digit for the leftmost smaller one |
| 21 | [738. Monotone Increasing Digits](https://leetcode.com/problems/monotone-increasing-digits/) | Medium | Find the drop, decrement, fill 9s |
| 22 | [1663. Smallest String With A Given Numeric Value](https://leetcode.com/problems/smallest-string-with-a-given-numeric-value/) | Medium | Fill from the right with as much as you can |
| 23 | [1689. Partitioning Into Minimum Number Of Deci-Binary Numbers](https://leetcode.com/problems/partitioning-into-minimum-number-of-deci-binary-numbers/) | Medium | The answer is the largest digit |
| 24 | [330. Patching Array](https://leetcode.com/problems/patching-array/) | Hard | You can build `[1, miss)`. If `nums[i] > miss`, patch `miss` |
| 25 | [991. Broken Calculator](https://leetcode.com/problems/broken-calculator/) | Medium | Go from target back to start. Halve when even |
| 26 | [397. Integer Replacement](https://leetcode.com/problems/integer-replacement/) | Medium | Prefer the even step. For odds, look at mod 4 |
| 27 | [319. Bulb Switcher](https://leetcode.com/problems/bulb-switcher/) | Medium | Squares stay on. `floor(sqrt(n))` |
| 28 | [672. Bulb Switcher II](https://leetcode.com/problems/bulb-switcher-ii/) | Medium | Four flips, the state space is tiny |
| 29 | [1529. Bulb Switcher IV](https://leetcode.com/problems/bulb-switcher-iv/) | Medium | Count the groups of lights that change |
| 30 | [376. Wiggle Subsequence](https://leetcode.com/problems/wiggle-subsequence/) | Medium | Count every direction change |
| 31 | [1144. Decrease Elements To Make Array Zigzag](https://leetcode.com/problems/decrease-elements-to-make-array-zigzag/) | Medium | Fix the even indexes, or the odd ones |
| 32 | [665. Non-decreasing Array](https://leetcode.com/problems/non-decreasing-array/) | Easy | One drop. Repair `a[i]` or `a[i+1]` |
| 33 | [1827. Minimum Operations to Make the Array Increasing](https://leetcode.com/problems/minimum-operations-to-make-the-array-increasing/) | Easy | Raise `a[i]` to `a[i-1] + 1` |
| 34 | [945. Minimum Increment to Make Array Unique](https://leetcode.com/problems/minimum-increment-to-make-array-unique/) | Medium | Sort, then push up to `prev + 1` |
| 35 | [605. Can Place Flowers](https://leetcode.com/problems/can-place-flowers/) | Easy | A hole of 3 empty plots holds one flower |
| 36 | [944. Delete Columns to Make Sorted](https://leetcode.com/problems/delete-columns-to-make-sorted/) | Easy | A column that decreases is deleted |
| 37 | [861. Score After Flipping Matrix](https://leetcode.com/problems/score-after-flipping-matrix/) | Medium | First column must be 1, then majority per other column |
| 38 | [769. Max Chunks To Make Sorted](https://leetcode.com/problems/max-chunks-to-make-sorted/) | Medium | A chunk ends when max-so-far equals the index |
| 39 | [768. Max Chunks To Make Sorted II](https://leetcode.com/problems/max-chunks-to-make-sorted-ii/) | Hard | Prefix max of the left vs suffix min of the right |

### Groups and consecutive numbers

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 40 | [453. Minimum Moves to Equal Array Elements](https://leetcode.com/problems/minimum-moves-to-equal-array-elements/) | Easy | Incrementing n-1 is decrementing one |
| 41 | [462. Minimum Moves to Equal Array Elements II](https://leetcode.com/problems/minimum-moves-to-equal-array-elements-ii/) | Medium | Move everyone to the median |
| 42 | [1013. Partition Array Into Three Parts With Equal Sum](https://leetcode.com/problems/partition-array-into-three-parts-with-equal-sum/) | Easy | Target is total/3, find two cuts |
| 43 | [846. Hand of Straights](https://leetcode.com/problems/hand-of-straights/) | Medium | Map of counts, start a group at the smallest |
| 44 | [659. Split Array into Consecutive Subsequences](https://leetcode.com/problems/split-array-into-consecutive-subsequences/) | Medium | Counts, and how many chains end at x |
| 45 | [1296. Divide Array in Sets of K Consecutive Numbers](https://leetcode.com/problems/divide-array-in-sets-of-k-consecutive-numbers/) | Medium | Same as hands, group size k |
| 46 | [984. String Without AAA or BBB](https://leetcode.com/problems/string-without-aaa-or-bbb/) | Medium | Write two of the majority, then one of the other |
| 47 | [649. Dota2 Senate](https://leetcode.com/problems/dota2-senate/) | Medium | A senator bans the next opposing senator |
| 48 | [1221. Split a String in Balanced Strings](https://leetcode.com/problems/split-a-string-in-balanced-strings/) | Easy | Every time the balance returns to zero, cut |
| 49 | [1518. Water Bottles](https://leetcode.com/problems/water-bottles/) | Easy | Drink, exchange, repeat |
| 50 | [517. Super Washing Machines](https://leetcode.com/problems/super-washing-machines/) | Hard | The max of the prefix deficit and the single-machine surplus |

---

## 13. Intervals

Sort by start. Then one of four things happens: you merge, you decide whether to keep, you cover a range, or you sweep a vertical line across the endpoints.

[2402. Meeting Rooms III](https://leetcode.com/problems/meeting-rooms-iii/) and [1353. Maximum Number of Events That Can Be Attended](https://leetcode.com/problems/maximum-number-of-events-that-can-be-attended/) are under [Heaps](#10-heaps) because the heap is the lesson. Meeting Rooms II, just below, is the same scheduling question without the extra machinery.

### Merge, insert, intersect

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 1 | [56. Merge Intervals](https://leetcode.com/problems/merge-intervals/) | Medium | Sort, extend the open interval |
| 2 | [57. Insert Interval](https://leetcode.com/problems/insert-interval/) | Medium | Before, overlap, after |
| 3 | [1272. Remove Interval](https://leetcode.com/problems/remove-interval/) | Medium | Cut one interval out of a disjoint list. Premium |
| 4 | [1288. Remove Covered Intervals](https://leetcode.com/problems/remove-covered-intervals/) | Medium | Sort by start, then by longer end |
| 5 | [986. Interval List Intersections](https://leetcode.com/problems/interval-list-intersections/) | Medium | Two pointers, overlap is `[max(l), min(r)]` |
| 6 | [495. Teemo Attacking](https://leetcode.com/problems/teemo-attacking/) | Easy | Merge a fixed-length poisoning window |
| 7 | [758. Bold Words in String](https://leetcode.com/problems/bold-words-in-string/) | Medium | Dictionary words become intervals, then merge. Premium |
| 8 | [163. Missing Ranges](https://leetcode.com/problems/missing-ranges/) | Easy | Gaps, including the ends. Premium |
| 9 | [352. Data Stream as Disjoint Intervals](https://leetcode.com/problems/data-stream-as-disjoint-intervals/) | Hard | Insert a point, merge neighbors. TreeMap |
| 10 | [715. Range Module](https://leetcode.com/problems/range-module/) | Hard | Add, remove, query. Ordered map of disjoint ranges |
| 11 | [370. Range Addition](https://leetcode.com/problems/range-addition/) | Medium | Difference array. Premium |
| 12 | [1854. Maximum Population Year](https://leetcode.com/problems/maximum-population-year/) | Easy | Difference array on years |
| 13 | [1893. Check if All the Integers in a Range Are Covered](https://leetcode.com/problems/check-if-all-the-integers-in-a-range-are-covered/) | Easy | Difference array, or a boolean span |
| 14 | [2848. Points That Intersect With Cars](https://leetcode.com/problems/points-that-intersect-with-cars/) | Easy | Merge, then sum the lengths |
| 15 | [2580. Count Ways to Group Overlapping Ranges](https://leetcode.com/problems/count-ways-to-group-overlapping-ranges/) | Medium | Each merged block has 2 choices |

### Scheduling

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 16 | [252. Meeting Rooms](https://leetcode.com/problems/meeting-rooms/) | Easy | Sort, any overlap? Premium |
| 17 | [253. Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/) | Medium | Min rooms. End-time heap, or a sweep. Premium |
| 18 | [2406. Divide Intervals Into Minimum Number of Groups](https://leetcode.com/problems/divide-intervals-into-minimum-number-of-groups/) | Medium | Free version of Meeting Rooms II |
| 19 | [1229. Meeting Scheduler](https://leetcode.com/problems/meeting-scheduler/) | Medium | Two pointers on both calendars. Premium |
| 20 | [759. Employee Free Time](https://leetcode.com/problems/employee-free-time/) | Hard | Merge everyone's busy time, gaps are free. Premium |
| 21 | [729. My Calendar I](https://leetcode.com/problems/my-calendar-i/) | Medium | No double booking. Tree of intervals |
| 22 | [731. My Calendar II](https://leetcode.com/problems/my-calendar-ii/) | Medium | Allow a double, reject a triple |
| 23 | [732. My Calendar III](https://leetcode.com/problems/my-calendar-iii/) | Hard | Sweep, the answer is the max depth |
| 24 | [436. Find Right Interval](https://leetcode.com/problems/find-right-interval/) | Medium | Sort starts, lower-bound each end |
| 25 | [1851. Minimum Interval to Include Each Query](https://leetcode.com/problems/minimum-interval-to-include-each-query/) | Hard | Sort queries, heap of intervals that cover the current point |
| 26 | [2251. Number of Flowers in Full Bloom](https://leetcode.com/problems/number-of-flowers-in-full-bloom/) | Hard | Sort starts and ends, walk the people |
| 27 | [2276. Count Integers in Intervals](https://leetcode.com/problems/count-integers-in-intervals/) | Hard | Disjoint intervals, add and merge, track the covered count |

### Keep, cover, weighted

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 28 | [435. Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/) | Medium | Sort by end, keep the one that frees the timeline sooner |
| 29 | [452. Minimum Number of Arrows to Burst Balloons](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/) | Medium | Same greedy, an arrow is a point |
| 30 | [646. Maximum Length of Pair Chain](https://leetcode.com/problems/maximum-length-of-pair-chain/) | Medium | Sort by end, take the next that starts after |
| 31 | [757. Set Intersection Size At Least Two](https://leetcode.com/problems/set-intersection-size-at-least-two/) | Hard | Sort by end, place points at the end |
| 32 | [1024. Video Stitching](https://leetcode.com/problems/video-stitching/) | Medium | Jump game on clip coverage |
| 33 | [1326. Minimum Number of Taps to Open to Water a Garden](https://leetcode.com/problems/minimum-number-of-taps-to-open-to-water-a-garden/) | Hard | Same jump, each tap covers a range |
| 34 | [2054. Two Best Non-Overlapping Events](https://leetcode.com/problems/two-best-non-overlapping-events/) | Medium | Sort, best event that ended before this one |
| 35 | [1235. Maximum Profit in Job Scheduling](https://leetcode.com/problems/maximum-profit-in-job-scheduling/) | Hard | DP + binary search the previous job |
| 36 | [1751. Maximum Number of Events That Can Be Attended II](https://leetcode.com/problems/maximum-number-of-events-that-can-be-attended-ii/) | Hard | Same, you may take at most k |
| 37 | [2008. Maximum Earnings From Taxi](https://leetcode.com/problems/maximum-earnings-from-taxi/) | Medium | DP on rides sorted by start |
| 38 | [2589. Minimum Time to Complete All Tasks](https://leetcode.com/problems/minimum-time-to-complete-all-tasks/) | Hard | Sort by end, run the task as late as possible |

### Line sweep and rectangles

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 39 | [218. The Skyline Problem](https://leetcode.com/problems/the-skyline-problem/) | Hard | Enter and leave events, heap of active heights |
| 40 | [850. Rectangle Area II](https://leetcode.com/problems/rectangle-area-ii/) | Hard | Sweep x, segment tree or inclusion on the y spans |
| 41 | [223. Rectangle Area](https://leetcode.com/problems/rectangle-area/) | Medium | Sum minus the overlap |
| 42 | [836. Rectangle Overlap](https://leetcode.com/problems/rectangle-overlap/) | Easy | Overlap on x and overlap on y |
| 43 | [939. Minimum Area Rectangle](https://leetcode.com/problems/minimum-area-rectangle/) | Medium | Diagonal pairs, the other two corners must exist |
| 44 | [963. Minimum Area Rectangle II](https://leetcode.com/problems/minimum-area-rectangle-ii/) | Medium | Center and diagonal length |
| 45 | [391. Perfect Rectangle](https://leetcode.com/problems/perfect-rectangle/) | Hard | Area matches, and exactly 4 corners appear once |
| 46 | [699. Falling Squares](https://leetcode.com/problems/falling-squares/) | Hard | Interval of x, the new height is max in range + side |
| 47 | [554. Brick Wall](https://leetcode.com/problems/brick-wall/) | Medium | The most common edge position is where you cut |
| 48 | [1943. Describe the Painting](https://leetcode.com/problems/describe-the-painting/) | Medium | Sweep color segments, merge equal adjacent mixes |
| 49 | [2158. Amount of New Area Painted Each Day](https://leetcode.com/problems/amount-of-new-area-painted-each-day/) | Hard | Paint a range, report the new length. Premium |
| 50 | [616. Add Bold Tag in String](https://leetcode.com/problems/add-bold-tag-in-string/) | Medium | Intervals of matches, merge, wrap. Premium |

---

## 14. Bit Manipulation

XOR cancels. A mask is a set. A range AND is the common prefix of the two numbers. Most of these are not "DP with bits". Bitmask DP (TSP, assign-jobs) is on the DP plan. Subset enumeration by recursion is under [Backtracking](#11-backtracking). The single-number family is under [Arrays & Hashing](#1-arrays--hashing). The binary trie for maximum XOR is [421](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/), under Strings.

### Bits of one number

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 1 | [191. Number of 1 Bits](https://leetcode.com/problems/number-of-1-bits/) | Easy | `n & (n-1)` drops the lowest set bit |
| 2 | [338. Counting Bits](https://leetcode.com/problems/counting-bits/) | Easy | `bits[i] = bits[i >> 1] + (i & 1)` |
| 3 | [190. Reverse Bits](https://leetcode.com/problems/reverse-bits/) | Easy | Pull from one end, push to the other |
| 4 | [231. Power of Two](https://leetcode.com/problems/power-of-two/) | Easy | One bit set: `n & (n-1)` is 0 |
| 5 | [342. Power of Four](https://leetcode.com/problems/power-of-four/) | Easy | Power of two, and the bit sits on an even position |
| 6 | [693. Binary Number with Alternating Bits](https://leetcode.com/problems/binary-number-with-alternating-bits/) | Easy | `n ^ (n >> 1)` is all ones |
| 7 | [762. Prime Number of Set Bits in Binary Representation](https://leetcode.com/problems/prime-number-of-set-bits-in-binary-representation/) | Easy | Popcount, then a tiny prime check |
| 8 | [476. Number Complement](https://leetcode.com/problems/number-complement/) | Easy | Flip bits inside the width of the number |
| 9 | [1009. Complement of Base 10 Integer](https://leetcode.com/problems/complement-of-base-10-integer/) | Easy | Same, including 0 |
| 10 | [868. Binary Gap](https://leetcode.com/problems/binary-gap/) | Easy | Distance between consecutive set bits |
| 11 | [461. Hamming Distance](https://leetcode.com/problems/hamming-distance/) | Easy | Popcount of XOR |
| 12 | [477. Total Hamming Distance](https://leetcode.com/problems/total-hamming-distance/) | Medium | Per bit, zeros times ones |
| 13 | [1342. Number of Steps to Reduce a Number to Zero](https://leetcode.com/problems/number-of-steps-to-reduce-a-number-to-zero/) | Easy | Subtract 1 if odd, halve if even |
| 14 | [1404. Number of Steps to Reduce a Number in Binary Representation to One](https://leetcode.com/problems/number-of-steps-to-reduce-a-number-in-binary-representation-to-one/) | Medium | The input is a binary string. Carry when you add 1 |
| 15 | [389. Find the Difference](https://leetcode.com/problems/find-the-difference/) | Easy | XOR both strings |

### Arithmetic and bases

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 16 | [371. Sum of Two Integers](https://leetcode.com/problems/sum-of-two-integers/) | Medium | XOR is the sum, AND shifted is the carry |
| 17 | [29. Divide Two Integers](https://leetcode.com/problems/divide-two-integers/) | Medium | Subtract the largest shifted divisor |
| 18 | [201. Bitwise AND of Numbers Range](https://leetcode.com/problems/bitwise-and-of-numbers-range/) | Medium | Common prefix. Shift until left equals right |
| 19 | [393. UTF-8 Validation](https://leetcode.com/problems/utf-8-validation/) | Medium | Leading-ones tell you the width |
| 20 | [89. Gray Code](https://leetcode.com/problems/gray-code/) | Medium | `i ^ (i >> 1)` |
| 21 | [405. Convert a Number to Hexadecimal](https://leetcode.com/problems/convert-a-number-to-hexadecimal/) | Easy | Mask 4 bits at a time |
| 22 | [504. Base 7](https://leetcode.com/problems/base-7/) | Easy | Repeated division |
| 23 | [168. Excel Sheet Column Title](https://leetcode.com/problems/excel-sheet-column-title/) | Easy | Base 26, and it is 1-indexed |
| 24 | [171. Excel Sheet Column Number](https://leetcode.com/problems/excel-sheet-column-number/) | Easy | Horner, base 26 |
| 25 | [1017. Convert to Base -2](https://leetcode.com/problems/convert-to-base-2/) | Medium | Remainder 0 or 1, and the divisor is negative |
| 26 | [1073. Adding Two Negabinary Numbers](https://leetcode.com/problems/adding-two-negabinary-numbers/) | Medium | Carry can be -1 |
| 27 | [1018. Binary Prefix Divisible By 5](https://leetcode.com/problems/binary-prefix-divisible-by-5/) | Easy | Running value mod 5 |

### XOR as a prefix

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 28 | [1486. XOR Operation in an Array](https://leetcode.com/problems/xor-operation-in-an-array/) | Easy | There is a closed form. A loop is fine |
| 29 | [1310. XOR Queries of a Subarray](https://leetcode.com/problems/xor-queries-of-a-subarray/) | Medium | Prefix XOR, `pref[r] ^ pref[l-1]` |
| 30 | [1720. Decode XORed Array](https://leetcode.com/problems/decode-xored-array/) | Easy | `a[i+1] = a[i] ^ encoded[i]` |
| 31 | [2433. Find The Original Array of Prefix Xor](https://leetcode.com/problems/find-the-original-array-of-prefix-xor/) | Medium | `arr[i] = pref[i] ^ pref[i-1]` |
| 32 | [1734. Decode XORed Permutation](https://leetcode.com/problems/decode-xored-permutation/) | Medium | XOR of 1..n, then peel the odds |
| 33 | [1829. Maximum XOR for Each Query](https://leetcode.com/problems/maximum-xor-for-each-query/) | Medium | Prefix XOR from the right, then mask to `2^k - 1` |
| 34 | [2683. Neighboring Bitwise XOR](https://leetcode.com/problems/neighboring-bitwise-xor/) | Medium | Derive the array from `derived`, check the cycle |
| 35 | [2425. Bitwise XOR of All Pairings](https://leetcode.com/problems/bitwise-xor-of-all-pairings/) | Medium | A value survives if the other array has odd length |
| 36 | [2429. Minimize XOR](https://leetcode.com/problems/minimize-xor/) | Medium | Match the high bits of num1, then fill low zeros |
| 37 | [2997. Minimum Number of Operations to Make Array XOR Equal to K](https://leetcode.com/problems/minimum-number-of-operations-to-make-array-xor-equal-to-k/) | Medium | Popcount of `xor(all) ^ k` |
| 38 | [810. Chalkboard XOR Game](https://leetcode.com/problems/chalkboard-xor-game/) | Hard | Alice wins if the XOR is already 0, or the length is even |

### Masks and subarray bits

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 39 | [318. Maximum Product of Word Lengths](https://leetcode.com/problems/maximum-product-of-word-lengths/) | Medium | Bitmask of letters, AND is 0 if they share nothing |
| 40 | [1178. Number of Valid Words for Each Puzzle](https://leetcode.com/problems/number-of-valid-words-for-each-puzzle/) | Hard | Puzzle subsets, first letter must be present |
| 41 | [1371. Find the Longest Substring Containing Vowels in Even Counts](https://leetcode.com/problems/find-the-longest-substring-containing-vowels-in-even-counts/) | Medium | 5-bit parity mask, first index of each mask |
| 42 | [1542. Find Longest Awesome Substring](https://leetcode.com/problems/find-longest-awesome-substring/) | Hard | Digit parity, a palindrome allows one odd |
| 43 | [898. Bitwise ORs of Subarrays](https://leetcode.com/problems/bitwise-ors-of-subarrays/) | Medium | OR only grows. The set of ORs ending here is small |
| 44 | [2401. Longest Nice Subarray](https://leetcode.com/problems/longest-nice-subarray/) | Medium | Window whose OR has no overlapping bits |
| 45 | [2419. Longest Subarray With Maximum Bitwise AND](https://leetcode.com/problems/longest-subarray-with-maximum-bitwise-and/) | Medium | The max AND is the max element. Longest run of it |
| 46 | [2275. Largest Combination With Bitwise AND Greater Than Zero](https://leetcode.com/problems/largest-combination-with-bitwise-and-greater-than-zero/) | Medium | The most crowded bit |
| 47 | [982. Triples with Bitwise AND Equal To Zero](https://leetcode.com/problems/triples-with-bitwise-and-equal-to-zero/) | Hard | Count pairs, then the third |
| 48 | [1835. Find XOR Sum of All Pairs Bitwise AND](https://leetcode.com/problems/find-xor-sum-of-all-pairs-bitwise-and/) | Hard | It collapses to `xor(A) & xor(B)` |
| 49 | [1611. Minimum One Bit Operations to Make Integers Zero](https://leetcode.com/problems/minimum-one-bit-operations-to-make-integers-zero/) | Hard | Gray-code inverse |
| 50 | [1787. Make the XOR of All Segments Equal to Zero](https://leetcode.com/problems/make-the-xor-of-all-segments-equal-to-zero/) | Hard | Positions mod k must agree. DP on the residue |

1787 is the one problem in this section that really is DP. It is here because the constraint is a XOR, and the official DP plan does not group it with the usual knapsack or LIS shapes. If you are saving DP for the official plan, skip 1787 until then.

---

## 15. Math & Geometry

Closed forms, gcd, counting, geometry, and the randomized designs interviewers actually ask. If a problem is "binary search the numeric answer", it is under [Binary Search](#5-binary-search) even when the predicate is number theory (ugly numbers, magical numbers, trailing zeros of n!).

### Digits, powers, fractions

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 1 | [7. Reverse Integer](https://leetcode.com/problems/reverse-integer/) | Medium | Pop digits, watch overflow |
| 2 | [66. Plus One](https://leetcode.com/problems/plus-one/) | Easy | Carry from the end |
| 3 | [989. Add to Array-Form of Integer](https://leetcode.com/problems/add-to-array-form-of-integer/) | Easy | Add k into the array from the right |
| 4 | [50. Pow(x, n)](https://leetcode.com/problems/powx-n/) | Medium | Binary exponentiation |
| 5 | [372. Super Pow](https://leetcode.com/problems/super-pow/) | Medium | Euler / mod at each digit of the exponent |
| 6 | [258. Add Digits](https://leetcode.com/problems/add-digits/) | Easy | Digital root, `1 + (n - 1) % 9` |
| 7 | [326. Power of Three](https://leetcode.com/problems/power-of-three/) | Easy | Divide out 3s, or max power of 3 mod n |
| 8 | [172. Factorial Trailing Zeroes](https://leetcode.com/problems/factorial-trailing-zeroes/) | Medium | Count factors of 5 |
| 9 | [273. Integer to English Words](https://leetcode.com/problems/integer-to-english-words/) | Hard | Groups of 3 digits, a word table |
| 10 | [166. Fraction to Recurring Decimal](https://leetcode.com/problems/fraction-to-recurring-decimal/) | Medium | Long division, map remainder to index |
| 11 | [592. Fraction Addition and Subtraction](https://leetcode.com/problems/fraction-addition-and-subtraction/) | Medium | Parse, gcd the denominator |
| 12 | [640. Solve the Equation](https://leetcode.com/problems/solve-the-equation/) | Medium | Coefficient of x, and the constant |

### Primes, factors, gcd

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 13 | [204. Count Primes](https://leetcode.com/problems/count-primes/) | Medium | Sieve |
| 14 | [263. Ugly Number](https://leetcode.com/problems/ugly-number/) | Easy | Divide out 2, 3, 5 |
| 15 | [507. Perfect Number](https://leetcode.com/problems/perfect-number/) | Easy | Sum of proper divisors |
| 16 | [728. Self Dividing Numbers](https://leetcode.com/problems/self-dividing-numbers/) | Easy | Every digit divides the number |
| 17 | [1492. The kth Factor of n](https://leetcode.com/problems/the-kth-factor-of-n/) | Medium | Walk to sqrt, factors come in pairs |
| 18 | [1390. Four Divisors](https://leetcode.com/problems/four-divisors/) | Medium | Exactly four divisors: p cubed, or a product of two distinct primes |
| 19 | [1979. Find Greatest Common Divisor of Array](https://leetcode.com/problems/find-greatest-common-divisor-of-array/) | Easy | gcd(min, max) is enough |
| 20 | [1071. Greatest Common Divisor of Strings](https://leetcode.com/problems/greatest-common-divisor-of-strings/) | Easy | gcd of the lengths, and `s+t == t+s` |
| 21 | [365. Water and Jug Problem](https://leetcode.com/problems/water-and-jug-problem/) | Medium | Bezout: reachable iff gcd divides the target |
| 22 | [1497. Check If Array Pairs Are Divisible by k](https://leetcode.com/problems/check-if-array-pairs-are-divisible-by-k/) | Medium | Remainder r pairs with k-r |
| 23 | [1015. Smallest Integer Divisible by K](https://leetcode.com/problems/smallest-integer-divisible-by-k/) | Medium | Remainders of 1, 11, 111, ... If a remainder repeats, stop |
| 24 | [650. 2 Keys Keyboard](https://leetcode.com/problems/2-keys-keyboard/) | Medium | Break n into its prime factors. The answer is their sum |
| 25 | [651. 4 Keys Keyboard](https://leetcode.com/problems/4-keys-keyboard/) | Medium | DP / greedy on the best multiply. Premium |

### Counting, games, randomness

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 26 | [118. Pascal's Triangle](https://leetcode.com/problems/pascals-triangle/) | Easy | Edges are 1, inside is the sum |
| 27 | [119. Pascal's Triangle II](https://leetcode.com/problems/pascals-triangle-ii/) | Easy | One row, update from the right |
| 28 | [628. Maximum Product of Three Numbers](https://leetcode.com/problems/maximum-product-of-three-numbers/) | Easy | Three largest, or two smallest (negatives) times the largest |
| 29 | [292. Nim Game](https://leetcode.com/problems/nim-game/) | Easy | Lose if n is a multiple of 4 |
| 30 | [1344. Angle Between Hands of a Clock](https://leetcode.com/problems/angle-between-hands-of-a-clock/) | Medium | 30H - 5.5M, the minimum with 360 |
| 31 | [380. Insert Delete GetRandom O(1)](https://leetcode.com/problems/insert-delete-getrandom-o1/) | Medium | Vector plus index map. Swap-delete |
| 32 | [381. Insert Delete GetRandom O(1) - Duplicates allowed](https://leetcode.com/problems/insert-delete-getrandom-o1-duplicates-allowed/) | Hard | Value to a set of indexes |
| 33 | [384. Shuffle an Array](https://leetcode.com/problems/shuffle-an-array/) | Medium | Fisher-Yates |
| 34 | [398. Random Pick Index](https://leetcode.com/problems/random-pick-index/) | Medium | Reservoir, one pass, no extra memory of the indexes |
| 35 | [470. Implement Rand10() Using Rand7()](https://leetcode.com/problems/implement-rand10-using-rand7/) | Medium | Rejection sampling on a 7x7 grid |
| 36 | [478. Generate Random Point in a Circle](https://leetcode.com/problems/generate-random-point-in-a-circle/) | Medium | Rejection in the square, or sqrt of the radius |
| 37 | [710. Random Pick with Blacklist](https://leetcode.com/problems/random-pick-with-blacklist/) | Hard | Map blacklisted numbers in range onto the free tail |

### Geometry

Rectangles as intervals (overlap, area, skyline) are under [Intervals](#13-intervals). What is left is points, angles, hulls, and circles.

| # | Problem | Diff | Pattern |
| --- | --- | --- | --- |
| 38 | [1232. Check If It Is a Straight Line](https://leetcode.com/problems/check-if-it-is-a-straight-line/) | Easy | Cross product is 0 |
| 39 | [1037. Valid Boomerang](https://leetcode.com/problems/valid-boomerang/) | Easy | The three points are not collinear |
| 40 | [1266. Minimum Time Visiting All Points](https://leetcode.com/problems/minimum-time-visiting-all-points/) | Easy | Chebyshev distance, `max(dx, dy)` |
| 41 | [812. Largest Triangle Area](https://leetcode.com/problems/largest-triangle-area/) | Easy | Shoelace on every triple |
| 42 | [593. Valid Square](https://leetcode.com/problems/valid-square/) | Medium | 4 equal sides, equal diagonals, no zero length |
| 43 | [447. Number of Boomerangs](https://leetcode.com/problems/number-of-boomerangs/) | Medium | For each point, count distances |
| 44 | [149. Max Points on a Line](https://leetcode.com/problems/max-points-on-a-line/) | Hard | Slope as a reduced fraction, from each point |
| 45 | [335. Self Crossing](https://leetcode.com/problems/self-crossing/) | Hard | Three cases of a spiral crossing itself |
| 46 | [587. Erect the Fence](https://leetcode.com/problems/erect-the-fence/) | Hard | Convex hull, monotonic chain |
| 47 | [356. Line Reflection](https://leetcode.com/problems/line-reflection/) | Medium | Reflect over the mid x. Premium |
| 48 | [1828. Queries on Number of Points Inside a Circle](https://leetcode.com/problems/queries-on-number-of-points-inside-a-circle/) | Medium | Distance squared vs r squared |
| 49 | [1453. Maximum Number of Darts Inside of a Circular Dartboard](https://leetcode.com/problems/maximum-number-of-darts-inside-of-a-circular-dartboard/) | Hard | A candidate circle is fixed by 2 points on the boundary |
| 50 | [1610. Maximum Number of Visible Points](https://leetcode.com/problems/maximum-number-of-visible-points/) | Hard | Angles, sort, circular window of the viewing angle |

---

## Where the famous ones went

If you came in looking for a problem by name:

| Problem | Topic |
| --- | --- |
| Two Sum, Group Anagrams, Product Except Self, Kadane | Arrays & Hashing |
| Longest Palindrome, Trie, Word Search II | Strings |
| 3Sum, Container With Most Water, Trapping Rain Water | Two Pointers |
| Longest Substring Without Repeating, Minimum Window, Sliding Window Maximum | Sliding Window |
| Koko, Split Array Largest Sum, Median of Two Sorted Arrays | Binary Search |
| Daily Temperatures, Largest Rectangle in Histogram, Basic Calculator | Stack |
| Reverse Linked List, Copy Random Pointer, LRU, LFU | Linked Lists |
| LCA, Serialize Tree, Validate BST | Trees & BST |
| Number of Islands, Course Schedule, Word Ladder, Alien Dictionary | Graphs |
| Merge k Lists, Median of a Stream, IPO, Task Scheduler | Heaps |
| Subsets, Permutations, N-Queens, Sudoku Solver | Backtracking |
| Jump Game, Gas Station, Candy | Greedy |
| Merge Intervals, Meeting Rooms, Skyline | Intervals |
| Single Number lives in Arrays. Counting Bits, Sum of Two Integers live in Bits | Bit Manipulation |
| Pow, Sieve, Insert Delete GetRandom | Math & Geometry |
| House Robber, Coin Change, LCS, Edit Distance, Knapsack, Stock IV | The official DP study plan, not this file |
