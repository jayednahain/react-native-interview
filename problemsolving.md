# DSA Learning Roadmap — Zero to Confident

> **How to use this:** Complete each stage fully before moving to the next.
> Each stage = Learn → Practice → Real World understanding.

---

## Stage 1 — Arrays & Basic Loops

> **The foundation of everything. Start here.**

### What to Learn

- What is an array and how indexing works
- Iterating with `for` and `while` loops
- Linear search (checking each element one by one)
- Finding max, min, and sum of an array

### Practice Problems

| Problem                      | Difficulty | Link                                                          |
| ---------------------------- | ---------- | ------------------------------------------------------------- |
| Move zeros to end of array   | Easy       | [LeetCode 283](https://leetcode.com/problems/move-zeroes/)    |
| Find max and min in an array | Easy       | Practice on paper first                                       |
| Reverse an array             | Easy       | [LeetCode 344](https://leetcode.com/problems/reverse-string/) |
| Count vowels in a string     | Easy       | Practice on paper first                                       |

### Real World Connection

Arrays store your phone contacts list, a playlist of songs, a shopping cart.
Every app you use relies on arrays constantly.

### Key Concept to Understand Before Moving On

> You should be able to loop through an array and find/change any element confidently.

---

## Stage 2 — Strings & Two Pointers

> **Work with text + learn a smarter loop pattern**

### What to Learn

- How strings work (they are just arrays of characters)
- The **Two Pointer** technique: one pointer at start, one at end, moving inward
- Palindrome checking
- Comparing characters from both ends

### Practice Problems

| Problem                               | Difficulty | Link                                                                      |
| ------------------------------------- | ---------- | ------------------------------------------------------------------------- |
| Check if string is a palindrome       | Easy       | [LeetCode 125](https://leetcode.com/problems/valid-palindrome/)           |
| Find all pairs that sum to X          | Easy       | Practice on paper first                                                   |
| Remove duplicate characters           | Easy       | [LeetCode 316](https://leetcode.com/problems/remove-duplicate-letters/)   |
| Check if two strings are permutations | Easy       | [LeetCode 242](https://leetcode.com/problems/valid-anagram/)              |
| Intersection of two arrays            | Easy       | [LeetCode 349](https://leetcode.com/problems/intersection-of-two-arrays/) |

### Real World Connection

Two pointers power search suggestions, spell-checkers, and form validation.
Any time you compare the start and end of something — that is two pointers.

### Key Concept to Understand Before Moving On

> You should be able to solve a palindrome check using two pointers without extra memory.

---

## Stage 3 — Sliding Window

> **Efficient subarray and substring problems**

### What to Learn

- What a "window" means in an array
- Fixed-size window (window size does not change)
- Dynamic/variable window (window grows and shrinks based on condition)
- When to use sliding window vs two pointers

### Practice Problems

| Problem                                        | Difficulty  | Link                                                                                        |
| ---------------------------------------------- | ----------- | ------------------------------------------------------------------------------------------- |
| Maximum subarray sum (Kadane's Algorithm)      | Easy-Medium | [LeetCode 53](https://leetcode.com/problems/maximum-subarray/)                              |
| Longest substring without repeating characters | Medium      | [LeetCode 3](https://leetcode.com/problems/longest-substring-without-repeating-characters/) |
| Smallest subarray with sum ≥ target            | Medium      | [LeetCode 209](https://leetcode.com/problems/minimum-size-subarray-sum/)                    |

### Real World Connection

- Video streaming buffers use sliding window
- Rate limiting in APIs (max 100 requests per minute) is sliding window
- Network packet analysis and monitoring tools

### Key Concept to Understand Before Moving On

> You should understand when to expand the window and when to shrink it.

---

## Stage 4 — Sorting & Binary Search

> **Order things fast. Find things even faster.**

### What to Learn

- Bubble Sort — understand the concept (not used in production, but teaches comparison)
- Merge Sort — divide and conquer, time complexity O(n log n)
- Binary Search — split the search space in half each time, O(log n)
- Big O Notation — O(1), O(n), O(log n), O(n²) and what they mean

### Practice Problems

| Problem                        | Difficulty | Link                                                                           |
| ------------------------------ | ---------- | ------------------------------------------------------------------------------ |
| Find missing number            | Easy       | [LeetCode 268](https://leetcode.com/problems/missing-number/)                  |
| Binary search on sorted array  | Easy       | [LeetCode 704](https://leetcode.com/problems/binary-search/)                   |
| Search in rotated sorted array | Medium     | [LeetCode 33](https://leetcode.com/problems/search-in-rotated-sorted-array/)   |
| Merge two sorted arrays        | Easy       | [LeetCode 88](https://leetcode.com/problems/merge-sorted-array/)               |
| Find kth largest element       | Medium     | [LeetCode 215](https://leetcode.com/problems/kth-largest-element-in-an-array/) |

### Real World Connection

- Google returns search results in milliseconds — binary search
- Sorting a product list by price or rating
- Autocomplete suggestions in search bars

### Key Concept to Understand Before Moving On

> You should be able to write binary search from memory and explain why it is O(log n).

---

## Stage 5 — Linked Lists, Stacks & Queues

> **Your first real data structures**

### What to Learn

- Linked List: nodes connected by pointers (not stored in continuous memory like arrays)
- Stack: Last In, First Out (LIFO) — like a stack of plates
- Queue: First In, First Out (FIFO) — like a line at a shop
- Floyd's Cycle Detection Algorithm (fast and slow pointer)

### Practice Problems

| Problem                               | Difficulty | Link                                                                            |
| ------------------------------------- | ---------- | ------------------------------------------------------------------------------- |
| Reverse a linked list                 | Easy       | [LeetCode 206](https://leetcode.com/problems/reverse-linked-list/)              |
| Detect cycle in linked list           | Easy       | [LeetCode 141](https://leetcode.com/problems/linked-list-cycle/)                |
| Implement stack using arrays          | Easy       | [LeetCode 155](https://leetcode.com/problems/min-stack/)                        |
| Implement queue using linked list     | Easy       | [LeetCode 232](https://leetcode.com/problems/implement-queue-using-stacks/)     |
| Find intersection of two linked lists | Easy       | [LeetCode 160](https://leetcode.com/problems/intersection-of-two-linked-lists/) |

### Real World Connection

- Browser back/forward history = Stack
- Print queue at an office = Queue
- Music playlist "next song" = Linked List
- Undo/Redo in any text editor = Stack

### Key Concept to Understand Before Moving On

> You should understand why linked lists exist when we already have arrays (hint: insertion and deletion speed).

---

## Stage 6 — Hash Maps & Sets

> **Instant lookups — the most powerful everyday tool**

### What to Learn

- Hash Map / Dictionary: key → value pairs with O(1) lookup
- Set: stores unique values only, O(1) lookup
- How hashing works internally
- Collision handling (chaining vs open addressing)

### Practice Problems

| Problem                            | Difficulty | Link                                                                              |
| ---------------------------------- | ---------- | --------------------------------------------------------------------------------- |
| Two Sum                            | Easy       | [LeetCode 1](https://leetcode.com/problems/two-sum/)                              |
| Find first non-repeating character | Easy       | [LeetCode 387](https://leetcode.com/problems/first-unique-character-in-a-string/) |
| Find majority element              | Easy       | [LeetCode 169](https://leetcode.com/problems/majority-element/)                   |
| Intersection of two arrays II      | Easy       | [LeetCode 350](https://leetcode.com/problems/intersection-of-two-arrays-ii/)      |
| Design LRU Cache                   | Hard       | [LeetCode 146](https://leetcode.com/problems/lru-cache/)                          |

### Real World Connection

- Password lookup and authentication
- Caching web pages (your browser does this)
- Counting word frequency in a document
- Storing user sessions in a web server

### Key Concept to Understand Before Moving On

> You should be able to solve Two Sum in O(n) using a hash map and explain why it is faster than a nested loop.

---

## Stage 7 — Trees & Recursion

> **Hierarchical data and thinking recursively**

### What to Learn

- Recursion: a function that calls itself with a smaller problem
- Binary Tree: each node has at most 2 children
- Binary Search Tree (BST): left < root < right
- Tree Traversals: In-order, Pre-order, Post-order (DFS)
- Level-order Traversal (BFS)

### Practice Problems

| Problem                      | Difficulty | Link                                                                                   |
| ---------------------------- | ---------- | -------------------------------------------------------------------------------------- |
| Maximum depth of binary tree | Easy       | [LeetCode 104](https://leetcode.com/problems/maximum-depth-of-binary-tree/)            |
| Validate binary search tree  | Medium     | [LeetCode 98](https://leetcode.com/problems/validate-binary-search-tree/)              |
| Lowest common ancestor       | Medium     | [LeetCode 236](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/) |
| Level order traversal        | Medium     | [LeetCode 102](https://leetcode.com/problems/binary-tree-level-order-traversal/)       |
| Symmetric tree               | Easy       | [LeetCode 101](https://leetcode.com/problems/symmetric-tree/)                          |

### Real World Connection

- File system on your computer (folders inside folders)
- HTML/CSS DOM structure of every website
- Organisation charts in companies
- Decision trees used in machine learning

### Key Concept to Understand Before Moving On

> You should be able to traverse a tree using recursion and explain the difference between DFS and BFS.

---

## Stage 8 — Graphs & Dynamic Programming

> **Advanced — unlocks solving the hardest problems**

### What to Learn

#### Graphs

- Graph representation: Adjacency List vs Adjacency Matrix
- BFS on graphs (shortest path in unweighted graph)
- DFS on graphs (cycle detection, connected components)
- Dijkstra's Algorithm (shortest path in weighted graph)
- Topological Sort

#### Dynamic Programming (DP)

- Memoization: store results so you don't recompute
- Tabulation: bottom-up DP with a table
- Common DP patterns: 0/1 Knapsack, LCS, Fibonacci

### Practice Problems

| Problem                        | Difficulty | Link                                                                       |
| ------------------------------ | ---------- | -------------------------------------------------------------------------- |
| Fibonacci number (DP)          | Easy       | [LeetCode 509](https://leetcode.com/problems/fibonacci-number/)            |
| Climbing stairs                | Easy       | [LeetCode 70](https://leetcode.com/problems/climbing-stairs/)              |
| Longest common subsequence     | Medium     | [LeetCode 1143](https://leetcode.com/problems/longest-common-subsequence/) |
| Number of islands (graph DFS)  | Medium     | [LeetCode 200](https://leetcode.com/problems/number-of-islands/)           |
| 0/1 Knapsack                   | Medium     | GeeksForGeeks                                                              |
| Detect cycle in directed graph | Medium     | [LeetCode 207](https://leetcode.com/problems/course-schedule/)             |
| Shortest path (Dijkstra)       | Hard       | [LeetCode 743](https://leetcode.com/problems/network-delay-time/)          |

### Real World Connection

- Google Maps routing = Graph + Dijkstra
- Facebook friend suggestions = Graph traversal
- Netflix recommendations = Graph algorithms
- Autocorrect = Dynamic Programming
- DNA sequence matching = LCS algorithm

---

## Quick Reference — Topic to Stage Map

| Topic                 | Stage                    |
| --------------------- | ------------------------ |
| Linear Search         | Stage 1                  |
| Array manipulation    | Stage 1                  |
| String manipulation   | Stage 2                  |
| Two Pointers          | Stage 2                  |
| Sliding Window        | Stage 3                  |
| Sorting algorithms    | Stage 4                  |
| Binary Search         | Stage 4                  |
| Linked List           | Stage 5                  |
| Stack & Queue         | Stage 5                  |
| Hash Map / Set        | Stage 6                  |
| Trees & Recursion     | Stage 7                  |
| Graphs                | Stage 8                  |
| Dynamic Programming   | Stage 8                  |
| Bit Manipulation      | After Stage 4 (optional) |
| Mathematical Problems | Anytime as warm-up       |

---

## Study Tips

1. **Code on paper first** — before typing, write your solution by hand. It slows you down in a good way.
2. **One problem per day minimum** — consistency beats intensity.
3. **Explain it out loud** — if you cannot explain a solution simply, you do not understand it yet.
4. **Read others' solutions after solving** — on LeetCode, check the Discuss tab after you solve.
5. **Do not memorize — understand** — patterns repeat, exact code does not.
6. **Time yourself** — aim for 20–30 minutes per easy problem, 45 minutes for medium.

---

_Start at Stage 1. Do not skip. Good luck._
