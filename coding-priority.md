# LinkedIn Staff (Marketplace Integrity): coding priority list

Source: two reported-question lists (44 problems). List A is recent (Jun to Nov 2026). List B is older (Jan 2025 to Apr 2026).
Ranking = report count x recency. The 44 problems reduce to about 12 patterns.

## Tier 1: must code cold (high frequency, recent)

| # | Problem | Reports | Pattern | What they're really testing |
|---|---------|---------|---------|-----------------------------|
| 716 | Max Stack | **24** | DLL + sorted map/heap, lazy delete | popMax in O(log n), not O(n). Start with two stacks, then upgrade |
| 364 | Nested List Weight Sum II | **19** | BFS by level / two-pass DFS | Reverse depth without computing maxDepth first (running-sum trick) |
| 1004 | Max Consecutive Ones III | 10 | Sliding window | "at most k zeros" window |
| 380 | Insert Delete GetRandom O(1) | 10 | Array + index map, swap-with-last | Follow-up: 381 (duplicates allowed) |
| 432 | All O`one Data Structure | 10 | DLL of count-buckets + key map | Hardest design one. Bucket insert/remove edge cases |
| 605 | Can Place Flowers | 7 | Greedy scan | Easy warm-up, must be bug-free in 5 min |
| 244 | Shortest Word Distance II | 7 | Preprocess index lists, two-pointer merge | Also know 243 and 245 (same word) |

## Tier 2: LinkedIn-custom / domain-flavored (not verbatim on LeetCode)

These are the ones Premium won't hand you. Most likely place for a Staff-level follow-up.

| Problem | Reports | Pattern | Closest LC |
|---------|---------|---------|-----------|
| Mutual Connections (mutual friends, 2nd degree, shortest path) | 2 | Set intersection, BFS | 1129-ish, plain BFS |
| LinkedIn Connection Distance (influencer nodes) | 1 (Staff) | **Bidirectional BFS**, expand the smaller frontier | 127 Word Ladder (bidirectional) |
| TaskList with prerequisites + cycle detection (addTask / pollNext) | 2 | Incremental topo sort (Kahn), DFS 3-color cycle check | 210 Course Schedule II |
| LZ78 compress / decompress | 2 | Trie or dict of prefixes | none |
| Restaurant election (instant runoff) | 1 | Simulation with counts + elimination | none |
| Mouse finds cheese / Robot Room Cleaner | 1 + 1 (Staff) | Blind DFS with backtrack moves, visited by relative coords | 489 |
| Tallest tower stacking blocks | 1 (Staff) | Sort by (weight + capacity), greedy/heap | none [?] |

## Tier 3: DP family

| # | Problem | Reports | Note |
|---|---------|---------|------|
| 265 / 256 / 1473 | Paint House II / I / III | 4 / 2 / 1 | Learn 265 (min + second-min, O(nk)); 256 falls out. 1473 is stretch |
| 53 / 152 | Max Subarray / Max Product Subarray | 3 / 2 | Kadane; track min and max for product |
| 121 | Best Time to Buy/Sell Stock | 1 | 2-minute problem |
| 135 | Candy | 1 | Two-pass greedy |

## Tier 4: graphs / union-find

| # | Problem | Reports | Note |
|---|---------|---------|------|
| 200 | Number of Islands | 5 | DFS/BFS. Most recent report (Mid Nov 2026) |
| 305 | Number of Islands II | 2 | Union-find with rank + path compression |
| 1135 | Connecting Cities (MST) | 1 | Kruskal reusing the same union-find |
| 210 | Course Schedule II | 2 | Covered by TaskList above |
| 277 | Find the Celebrity | 4 | Candidate elimination + verify pass |

## Tier 5: trees, heaps, iterators, design

| # | Problem | Reports | Note |
|---|---------|---------|------|
| 341 | Flatten Nested List Iterator | 3 (Staff) | Stack of iterators. Same family as 364 |
| 272 | Closest BST Value II | 3 | Two stacks (pred/succ) or inorder + sliding window |
| 366 | Find Leaves of Binary Tree | 2 | Post-order height bucketing |
| 215 | Kth Largest | 4 | Heap of size k; quickselect as follow-up |
| 373 | K Pairs Smallest Sums | 2 | Min-heap k-way merge |
| 146 | LRU Cache | 1 (Staff) | DLL + hashmap. Shares code with 716 / 432 |
| 156, 235, 104 | Upside Down, LCA BST, Max Depth | 1 each | Quick reviews |

## Tier 6: easy / skim

17 Letter Combinations (5), 20 Valid Parentheses, 21 Merge Two Lists, 367 Valid Perfect Square (binary search), 65 Valid Number (2, state machine, tedious), 76 / 727 Min Window Substring / Subsequence (sliding-window family with 1004).

## Shared building blocks (write once, reuse)

1. **Doubly linked list + hashmap**: 146, 432, 716
2. **Union-find**: 200, 305, 1135
3. **BFS / bidirectional BFS**: 200, Mutual Connections, Connection Distance
4. **Topo sort + cycle detection**: 210, TaskList
5. **Sliding window**: 1004, 76, 727
6. **Nested structure recursion**: 364, 341

## Progress

| Problem | Solved | Timed (<25 min) | Notes |
|---------|--------|-----------------|-------|
| 716 Max Stack | | | |
| 364 Nested List Weight Sum II | | | |
| 1004 Max Consecutive Ones III | | | |
| 380 Insert Delete GetRandom | | | |
| 432 All O`one | | | |
| 605 Can Place Flowers | | | |
| 244 Shortest Word Distance II | | | |
| Connection Distance (bidirectional BFS) | | | |
| TaskList + cycle detection | | | |
| LZ78 | | | |
| 265 Paint House II | | | |
| 200 / 305 Islands I + II | | | |
