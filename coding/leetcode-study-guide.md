# LeetCode Study Guide — MAANG ML/MLE/DS Roles

Curated question list organized by pattern. Every question here has been reported at MAANG for ML Engineer, Applied Scientist, or Senior SWE-ML roles.

**Time budget**: 6–8 weeks, ~1.5 hours/day. Target 120–150 questions total.

---

## How to Use This Guide

1. Work through patterns in order — later patterns build on earlier ones
2. For each question: solve independently first, then review the optimal solution
3. Track your time: target ≤ 20 min for Medium, ≤ 35 min for Hard
4. After solving: write the pattern name and time/space complexity in a notes doc
5. Re-do questions you got wrong after 3 days (spaced repetition)

**Company priority filter:**
- 🟢 = reported at Google
- 🔵 = reported at Meta
- 🟠 = reported at Amazon
- 🟣 = reported at Apple/Netflix
- ⭐ = reported at multiple; highest priority

---

## Pattern 1: Arrays & Hashing

**Core idea**: use a hash map to trade O(n) time for O(n) space. Convert O(n²) brute force to O(n).

### Must-solve
| # | Problem | Difficulty | Companies | Key insight |
|---|---------|-----------|-----------|-------------|
| 1 | Two Sum | Easy | ⭐ All | HashMap of value → index |
| 49 | Group Anagrams | Medium | ⭐ 🟢🔵 | Sort each string as key |
| 347 | Top K Frequent Elements | Medium | ⭐ 🟢🔵🟠 | Bucket sort or heap |
| 238 | Product of Array Except Self | Medium | ⭐ 🟢🔵 | Prefix + suffix product |
| 128 | Longest Consecutive Sequence | Medium | ⭐ 🟢🔵 | HashSet, only start from sequence beginning |
| 217 | Contains Duplicate | Easy | 🟠 | Set |
| 242 | Valid Anagram | Easy | 🟢🔵 | Counter / sorted |
| 271 | Encode and Decode Strings | Medium | 🔵 | Length-prefixed encoding |

### Good practice
| # | Problem | Difficulty |
|---|---------|-----------|
| 36 | Valid Sudoku | Medium |
| 659 | Split Array into Consecutive Subsequences | Medium |

---

## Pattern 2: Two Pointers

**Core idea**: use two indices that move toward each other or in the same direction to avoid nested loops.

### Must-solve
| # | Problem | Difficulty | Companies | Key insight |
|---|---------|-----------|-----------|-------------|
| 125 | Valid Palindrome | Easy | 🔵🟠 | Left/right pointers, skip non-alphanum |
| 167 | Two Sum II (sorted array) | Medium | ⭐ | Left/right pointers |
| 15 | 3Sum | Medium | ⭐ 🟢🔵 | Sort, fix one, two-pointer for rest |
| 11 | Container With Most Water | Medium | ⭐ 🟢🔵🟠 | Move pointer with smaller height |
| 42 | Trapping Rain Water | Hard | ⭐ 🟢🔵🟠 | Left/right max arrays or two pointers |
| 26 | Remove Duplicates from Sorted Array | Easy | 🟠 | Slow/fast pointer |
| 75 | Sort Colors | Medium | 🟢🔵 | Dutch flag (3 pointers) |

---

## Pattern 3: Sliding Window

**Core idea**: expand/contract a window to find an optimal subarray/substring in O(n).

### Must-solve
| # | Problem | Difficulty | Companies | Key insight |
|---|---------|-----------|-----------|-------------|
| 121 | Best Time to Buy and Sell Stock | Easy | ⭐ All | Track min price so far |
| 3 | Longest Substring Without Repeating Characters | Medium | ⭐ All | HashMap last seen index |
| 424 | Longest Repeating Character Replacement | Medium | ⭐ 🟢🔵 | Track max freq char in window |
| 567 | Permutation in String | Medium | 🟢🔵 | Fixed-size window + char count |
| 76 | Minimum Window Substring | Hard | ⭐ 🟢🔵🟠 | Shrink window when all chars covered |
| 239 | Sliding Window Maximum | Hard | 🟢🔵 | Monotonic deque |

---

## Pattern 4: Stack

**Core idea**: LIFO structure for problems involving matching pairs, previous/next greater elements, or nested structures.

### Must-solve
| # | Problem | Difficulty | Companies | Key insight |
|---|---------|-----------|-----------|-------------|
| 20 | Valid Parentheses | Easy | ⭐ All | Push opens, pop on close |
| 155 | Min Stack | Medium | 🟢🔵🟠 | Parallel stack tracking min |
| 150 | Evaluate Reverse Polish Notation | Medium | 🟠 | Push nums, pop on operator |
| 739 | Daily Temperatures | Medium | ⭐ 🟢🔵🟠 | Monotonic decreasing stack |
| 853 | Car Fleet | Medium | 🟢 | Sort by position, stack of speeds |
| 84 | Largest Rectangle in Histogram | Hard | ⭐ 🟢🔵 | Monotonic increasing stack |

---

## Pattern 5: Binary Search

**Core idea**: whenever the search space is sorted or has a monotonic property, binary search gives O(log n).

### Must-solve
| # | Problem | Difficulty | Companies | Key insight |
|---|---------|-----------|-----------|-------------|
| 704 | Binary Search | Easy | All | Baseline |
| 74 | Search a 2D Matrix | Medium | 🟢🔵 | Treat as 1D sorted array |
| 875 | Koko Eating Bananas | Medium | ⭐ 🟢🟠 | Binary search on answer |
| 33 | Search in Rotated Sorted Array | Medium | ⭐ 🟢🔵🟠 | Identify sorted half first |
| 153 | Find Minimum in Rotated Sorted Array | Medium | ⭐ 🟢🔵🟠 | Pivot is where order breaks |
| 981 | Time Based Key-Value Store | Medium | 🟢🔵 | Binary search on timestamps |
| 4 | Median of Two Sorted Arrays | Hard | ⭐ 🟢 | Binary search on partition |

---

## Pattern 6: Linked Lists

**Core idea**: use slow/fast pointers for cycle detection and midpoint finding; use dummy nodes for clean insertion/deletion.

### Must-solve
| # | Problem | Difficulty | Companies | Key insight |
|---|---------|-----------|-----------|-------------|
| 206 | Reverse Linked List | Easy | ⭐ All | Three pointers: prev, curr, next |
| 21 | Merge Two Sorted Lists | Easy | ⭐ All | Dummy head |
| 141 | Linked List Cycle | Easy | ⭐ All | Floyd's tortoise and hare |
| 143 | Reorder List | Medium | 🔵🟠 | Find mid, reverse 2nd half, merge |
| 19 | Remove Nth Node From End | Medium | ⭐ 🟢🔵🟠 | Two pointers N apart |
| 138 | Copy List with Random Pointer | Medium | ⭐ 🟢🔵 | HashMap old → new nodes |
| 23 | Merge K Sorted Lists | Hard | ⭐ 🟢🔵🟠 | Min-heap of (val, list_index) |
| 25 | Reverse Nodes in K-Group | Hard | 🟢🔵 | Reverse sub-lists iteratively |

---

## Pattern 7: Trees

**Core idea**: most tree problems are recursive DFS. Know preorder/inorder/postorder and BFS (level-order).

### Must-solve
| # | Problem | Difficulty | Companies | Key insight |
|---|---------|-----------|-----------|-------------|
| 226 | Invert Binary Tree | Easy | ⭐ All | Swap children recursively |
| 104 | Maximum Depth of Binary Tree | Easy | ⭐ All | 1 + max(left, right) |
| 543 | Diameter of Binary Tree | Easy | ⭐ 🟢🔵🟠 | Track max at each node |
| 110 | Balanced Binary Tree | Easy | 🟢🟠 | Check depth difference |
| 100 | Same Tree | Easy | 🟢🟠 | Structural + value equality |
| 572 | Subtree of Another Tree | Easy | 🟢🔵 | Same tree check at every node |
| 235 | LCA of BST | Medium | ⭐ 🟢🔵🟠 | Traverse based on values |
| 102 | Binary Tree Level Order Traversal | Medium | ⭐ All | BFS with queue |
| 199 | Binary Tree Right Side View | Medium | ⭐ 🟢🔵🟠 | BFS, take last of each level |
| 1448 | Count Good Nodes in Binary Tree | Medium | 🔵 | Track max along path |
| 98 | Validate BST | Medium | ⭐ 🟢🔵🟠 | Pass min/max bounds |
| 230 | Kth Smallest Element in BST | Medium | ⭐ 🟢🔵 | Inorder traversal |
| 105 | Construct BT from Preorder/Inorder | Medium | ⭐ 🟢🔵 | HashMap for inorder indices |
| 124 | Binary Tree Maximum Path Sum | Hard | ⭐ 🟢🔵🟠 | Track contribution vs path sum |
| 297 | Serialize and Deserialize BT | Hard | ⭐ 🟢🔵 | BFS or DFS with null markers |

---

## Pattern 8: Tries

**Core idea**: prefix tree for efficient string search, autocomplete, prefix matching.

### Must-solve
| # | Problem | Difficulty | Companies | Key insight |
|---|---------|-----------|-----------|-------------|
| 208 | Implement Trie | Medium | ⭐ 🟢🔵🟠 | Node class with children dict + is_end |
| 211 | Design Add and Search Words | Medium | 🟢🔵 | DFS with '.' wildcard |
| 212 | Word Search II | Hard | 🟢🔵 | Trie + DFS backtracking on grid |

---

## Pattern 9: Graphs

**Core idea**: represent as adjacency list. BFS for shortest path; DFS for connected components; Union-Find for dynamic connectivity.

### Must-solve — BFS/DFS
| # | Problem | Difficulty | Companies | Key insight |
|---|---------|-----------|-----------|-------------|
| 200 | Number of Islands | Medium | ⭐ All | DFS/BFS, mark visited |
| 133 | Clone Graph | Medium | ⭐ 🟢🔵🟠 | HashMap old → new, BFS |
| 695 | Max Area of Island | Medium | 🟢🔵🟠 | DFS, return size |
| 417 | Pacific Atlantic Water Flow | Medium | 🟢🔵 | BFS from both oceans inward |
| 130 | Surrounded Regions | Medium | 🟢🔵 | BFS from border O's |
| 994 | Rotting Oranges | Medium | ⭐ 🟢🔵🟠 | Multi-source BFS |
| 286 | Walls and Gates | Medium | 🟢🔵 | Multi-source BFS from gates |

### Must-solve — Topological Sort / Cycle Detection
| # | Problem | Difficulty | Companies | Key insight |
|---|---------|-----------|-----------|-------------|
| 207 | Course Schedule | Medium | ⭐ 🟢🔵🟠 | Detect cycle in directed graph (DFS) |
| 210 | Course Schedule II | Medium | ⭐ 🟢🔵🟠 | Topological sort (Kahn's BFS) |
| 684 | Redundant Connection | Medium | 🟢🔵 | Union-Find; find edge creating cycle |

### Must-solve — Shortest Path
| # | Problem | Difficulty | Companies | Key insight |
|---|---------|-----------|-----------|-------------|
| 743 | Network Delay Time | Medium | ⭐ 🟢🔵🟠 | Dijkstra's algorithm |
| 778 | Swim in Rising Water | Hard | 🟢 | Binary search + BFS, or Dijkstra |
| 787 | Cheapest Flights Within K Stops | Medium | 🟢🔵🟠 | Bellman-Ford (K+1 rounds) |

### Must-solve — Union-Find
| # | Problem | Difficulty | Companies | Key insight |
|---|---------|-----------|-----------|-------------|
| 323 | Number of Connected Components | Medium | 🟢🔵 | Union-Find or DFS |
| 261 | Graph Valid Tree | Medium | 🟢🔵 | n-1 edges + no cycle |

---

## Pattern 10: Dynamic Programming

**Core idea**: break into subproblems; store solutions to avoid recomputation. Most DP has two steps: define state, write recurrence.

### 1D DP — Must-solve
| # | Problem | Difficulty | Companies | Key insight |
|---|---------|-----------|-----------|-------------|
| 70 | Climbing Stairs | Easy | ⭐ All | dp[i] = dp[i-1] + dp[i-2] |
| 746 | Min Cost Climbing Stairs | Easy | 🟢🟠 | dp[i] = cost[i] + min(dp[i-1], dp[i-2]) |
| 198 | House Robber | Medium | ⭐ 🟢🔵🟠 | dp[i] = max(dp[i-1], dp[i-2]+nums[i]) |
| 213 | House Robber II (circular) | Medium | ⭐ 🟢🔵🟠 | Run twice: [0..n-2] and [1..n-1] |
| 5 | Longest Palindromic Substring | Medium | ⭐ 🟢🔵 | Expand from center |
| 647 | Palindromic Substrings | Medium | 🟢🔵 | Expand from center, count |
| 91 | Decode Ways | Medium | ⭐ 🟢🔵🟠 | dp with single + double digit check |
| 322 | Coin Change | Medium | ⭐ All | dp[amount] = min(dp[amount - coin] + 1) |
| 139 | Word Break | Medium | ⭐ 🟢🔵🟠 | dp[i] = any dp[j] and s[j:i] in dict |
| 300 | Longest Increasing Subsequence | Medium | ⭐ 🟢🔵🟠 | dp[i] = max(dp[j]+1) for j<i, nums[j]<nums[i] |

### 2D DP — Must-solve
| # | Problem | Difficulty | Companies | Key insight |
|---|---------|-----------|-----------|-------------|
| 62 | Unique Paths | Medium | ⭐ 🟢🔵🟠 | dp[i][j] = dp[i-1][j] + dp[i][j-1] |
| 1143 | Longest Common Subsequence | Medium | ⭐ 🟢🔵🟠 | Classic 2D DP |
| 309 | Best Time to Buy/Sell with Cooldown | Medium | 🟢🔵 | State machine DP |
| 518 | Coin Change II (ways) | Medium | 🟢🔵🟠 | Unbounded knapsack |
| 494 | Target Sum | Medium | ⭐ 🟢🔵🟠 | 0/1 knapsack or DFS+memo |
| 72 | Edit Distance | Hard | ⭐ 🟢🔵 | Classic 2D DP; learn the recurrence cold |

---

## Pattern 11: Heap / Priority Queue

**Core idea**: use a heap when you need the K smallest/largest, or repeated min/max extraction.

### Must-solve
| # | Problem | Difficulty | Companies | Key insight |
|---|---------|-----------|-----------|-------------|
| 703 | Kth Largest Element in Stream | Easy | ⭐ 🟢🔵🟠 | Min-heap of size K |
| 1046 | Last Stone Weight | Easy | 🟠 | Max-heap |
| 973 | K Closest Points to Origin | Medium | ⭐ 🟢🔵🟠 | Max-heap of size K |
| 215 | Kth Largest Element in Array | Medium | ⭐ 🟢🔵🟠 | Quickselect or min-heap |
| 621 | Task Scheduler | Medium | ⭐ 🟢🔵🟠 | Max-heap + idle calculation |
| 355 | Design Twitter | Medium | 🔵 | Heap merge of K sorted tweet lists |
| 295 | Find Median from Data Stream | Hard | ⭐ 🟢🔵 | Two heaps: max-heap lower, min-heap upper |

---

## Pattern 12: Backtracking

**Core idea**: explore all possibilities via DFS; prune branches that can't lead to a valid solution.

### Must-solve
| # | Problem | Difficulty | Companies | Key insight |
|---|---------|-----------|-----------|-------------|
| 78 | Subsets | Medium | ⭐ 🟢🔵🟠 | Include/exclude each element |
| 39 | Combination Sum | Medium | ⭐ 🟢🔵🟠 | Reuse elements; start from current index |
| 40 | Combination Sum II | Medium | 🟢🔵 | Skip duplicates at same level |
| 46 | Permutations | Medium | ⭐ 🟢🔵🟠 | Swap elements in place |
| 90 | Subsets II | Medium | 🟢🔵 | Sort + skip duplicates |
| 79 | Word Search | Medium | ⭐ 🟢🔵🟠 | DFS + in-place visited marking |
| 131 | Palindrome Partitioning | Medium | ⭐ 🟢🔵 | Backtrack + palindrome check |
| 17 | Letter Combinations of Phone Number | Medium | ⭐ 🟢🔵🟠 | Map digits to chars, backtrack |
| 51 | N-Queens | Hard | ⭐ 🟢🔵 | Track columns + diagonals |

---

## Pattern 13: Intervals

**Core idea**: sort by start, then merge/find overlaps.

### Must-solve
| # | Problem | Difficulty | Companies | Key insight |
|---|---------|-----------|-----------|-------------|
| 57 | Insert Interval | Medium | ⭐ 🟢🔵🟠 | Three phases: before, overlap, after |
| 56 | Merge Intervals | Medium | ⭐ All | Sort by start; extend end |
| 435 | Non-overlapping Intervals | Medium | ⭐ 🟢🔵🟠 | Greedy: sort by end, keep earliest-ending |
| 252 | Meeting Rooms | Easy | 🔵🟠 | Sort, check adjacent overlap |
| 253 | Meeting Rooms II | Medium | ⭐ 🟢🔵🟠 | Min-heap of end times |
| 1851 | Minimum Interval to Include Each Query | Hard | 🟢 | Sort intervals + queries, min-heap |

---

## Pattern 14: Greedy

**Core idea**: make the locally optimal choice at each step. Prove it leads to global optimum.

### Must-solve
| # | Problem | Difficulty | Companies | Key insight |
|---|---------|-----------|-----------|-------------|
| 53 | Maximum Subarray | Medium | ⭐ All | Kadane's algorithm |
| 55 | Jump Game | Medium | ⭐ 🟢🔵🟠 | Track max reachable index |
| 45 | Jump Game II | Medium | ⭐ 🟢🔵🟠 | BFS layers = jumps |
| 134 | Gas Station | Medium | 🟢🔵🟠 | If total gas ≥ total cost, solution exists |
| 846 | Hand of Straights | Medium | 🟢🔵 | Counter + greedy grouping |
| 763 | Partition Labels | Medium | ⭐ 🔵🟠 | Last occurrence of each char |

---

## ML-Specific Coding Questions

These appear in ML Engineer / Applied Scientist rounds specifically. Not just LeetCode — these test ML implementation skills.

| Problem | Where it appears | Key skill |
|---------|-----------------|-----------|
| Implement k-means clustering | Google, Uber, Apple | NumPy, k-means++, convergence |
| Compute AUC-ROC from scratch | Uber, Meta | Sorting, trapezoidal rule |
| Implement gradient descent (linear regression) | Startup, Google | Vectorized ops, partial derivatives |
| Implement backpropagation for 2-layer net | Google, Apple | Chain rule, matrix dimensions |
| Find top-K items from a stream | Amazon, Meta | Min-heap |
| Compute running median from a stream | Google, Meta | Two heaps |
| Implement cosine similarity at scale | Pinterest, Apple | Vectorization, ANN concepts |
| Design a feature store lookup | Meta, Google | HashMap, TTL, caching |
| Implement BM25 scoring | Search teams | IDF, term frequency |
| Serialize/deserialize a decision tree | ML interviews | Tree traversal, string encoding |
| Given a DataFrame, compute churn cohort | Netflix, Meta DS | Pandas groupby, time-series |

See [ml-coding-from-scratch.md](ml-coding-from-scratch.md) for complete implementations.

---

## Company-Specific Question Sets

### Google (LC Medium–Hard bar)
Focus on: arrays, trees, graphs, DP. Expect unusual/novel problems not just standard patterns.

High-frequency:
- 42 Trapping Rain Water ⭐
- 297 Serialize/Deserialize BT
- 23 Merge K Sorted Lists
- 4 Median of Two Sorted Arrays
- 76 Minimum Window Substring
- 212 Word Search II
- 124 Binary Tree Maximum Path Sum
- 51 N-Queens

### Meta (LC Medium bar, some Hard for E6+)
Focus on: arrays, trees, graphs, strings. Often have a "stream processing" or "feed data" flavor.

High-frequency:
- 1 Two Sum (yes, really — often with a twist)
- 56 Merge Intervals
- 986 Interval List Intersections
- 523 Continuous Subarray Sum
- 973 K Closest Points to Origin
- 50 Pow(x, n)
- 199 Binary Tree Right Side View
- 721 Accounts Merge
- 426 Convert BST to Sorted Doubly Linked List

### Amazon (LC Easy–Medium bar)
Focus on: arrays, trees, BFS/DFS. LP-connected problems common ("design a feature for Amazon").

High-frequency:
- 1 Two Sum
- 200 Number of Islands
- 42 Trapping Rain Water
- 238 Product of Array Except Self
- 146 LRU Cache
- 127 Word Ladder
- 329 Longest Increasing Path in a Matrix
- 297 Serialize/Deserialize BT

### Apple (LC Medium bar, ML rounds may have ML-specific questions)
Focus on: clean code, edge cases, ML-flavored data problems.

High-frequency:
- 15 3Sum
- 11 Container With Most Water
- 73 Set Matrix Zeroes
- 253 Meeting Rooms II
- Implement ML primitives (k-means, cosine similarity)

### Netflix (LC Medium, occasional Hard for Staff)
Focus on: data structure design, streaming data problems.

High-frequency:
- 295 Find Median from Data Stream
- 146 LRU Cache
- 460 LFU Cache
- 23 Merge K Sorted Lists
- 380 Insert Delete GetRandom O(1)

---

## System Design / Data Structure Design Questions

These are distinct from ML system design — focus on implementing data structures or scalable systems.

| Problem | Difficulty | Companies | Pattern |
|---------|-----------|-----------|---------|
| 146 LRU Cache | Medium | ⭐ All | HashMap + Doubly Linked List |
| 460 LFU Cache | Hard | 🟢🔵🟣 | Two HashMaps + Doubly Linked List |
| 380 Insert Delete GetRandom O(1) | Medium | 🔵🟠 | HashMap + Array |
| 155 Min Stack | Medium | ⭐ All | Parallel stack |
| 208 Implement Trie | Medium | ⭐ 🟢🔵🟠 | Tree of nodes |
| 355 Design Twitter | Medium | 🔵 | Heap + HashMap |
| 981 Time Based Key-Value Store | Medium | 🟢🔵 | HashMap + binary search |
| 1472 Design Browser History | Medium | 🟢🔵🟠 | Doubly linked list |

---

## 12-Week Coding Schedule

### Week 1–2: Foundations
- Day 1–2: Arrays + Hashing (all must-solve)
- Day 3–4: Two Pointers + Sliding Window
- Day 5–6: Stack + Binary Search
- Weekend: review weak areas, redo problems you got wrong

### Week 3–4: Trees and Linked Lists
- Day 1–3: Linked Lists
- Day 4–7: Trees (complete all must-solve)
- Weekend: timed practice — 2 problems in 45 minutes

### Week 5–6: Graphs + DP Part 1
- Day 1–3: Graphs (BFS/DFS, topological sort)
- Day 4–7: 1D Dynamic Programming
- Weekend: Mock interview (coding only, 45 min)

### Week 7–8: DP Part 2 + Advanced
- Day 1–3: 2D DP
- Day 4–5: Heap + Intervals
- Day 6–7: Greedy + Backtracking
- Weekend: Mock interview

### Week 9–10: ML Coding + Company-Specific
- Day 1–3: ML coding from scratch (see ml-coding-from-scratch.md)
- Day 4–7: Company-specific question sets (target 15 questions per company)
- Weekend: Timed contest (LeetCode weekly contest)

### Week 11: Interview Prep Mode
- Daily: 2–3 problems timed (no hints)
- Focus on your weakest pattern
- Do 2 full mock interviews (coding only)

### Week 12: Maintenance
- Daily: 1–2 warm-up problems
- Review notes from all previous problems
- No new problems — don't learn new patterns right before interviews

---

## Tips

**On timing yourself**: don't look at hints for the first 20 minutes. After 20 minutes, it's OK to check the pattern hint (not the full solution). After 35 minutes, read the solution — learn it, implement it from scratch the next day.

**On reviewing solutions**: after solving, always check the editorial for:
1. Is your time complexity optimal?
2. Is there a cleaner implementation?
3. What edge cases does the editorial mention that you missed?

**The "explain it" test**: after solving, try to explain the solution out loud in 60 seconds. If you can't, you don't really understand it.

**Recognize patterns fast**: in real interviews, you have 45 minutes total. You need to recognize the pattern within 5 minutes. Practice by seeing the problem title and trying to name the pattern before reading the problem.
