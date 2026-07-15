# Segment Tree – Build, Query, and Update

## 1. Problem Statement

Implement a **Segment Tree** to support efficient range queries and point updates on an array of numbers. 
Specifically, design a Segment Tree for **Range Sum Queries**:
- **Build**: Construct the tree from a given array of size $N$.
- **Query**: Get the sum of elements in a range $[L, R]$ (inclusive).
- **Update**: Modify a single element of the array at index $idx$ to a new value $val$, and update the tree accordingly.

### Operations and Complexities
- **Build**: $O(N)$
- **Query**: $O(\log N)$
- **Update**: $O(\log N)$

### Constraints
- $1 \le N \le 10^5$.
- Node values: $-10^4 \le \text{arr[i]} \le 10^4$.
- Number of queries/updates: $1 \le Q \le 10^5$.

---

## 2. Interview Clarification

1. **Is the array index 0-indexed or 1-indexed?**
   - *Answer:* The input array is usually 0-indexed. The segment tree's internal nodes can be stored in a 1-indexed vector where the root is at index 1, the left child of node $i$ is at $2i$, and the right child is at $2i + 1$.
2. **What is the maximum size of the Segment Tree array?**
   - *Answer:* For an array of size $N$, the maximum number of nodes in the segment tree is at most $4N$.
3. **What happens if the query range $[L, R]$ is out of bounds or invalid?**
   - *Answer:* We should assume valid ranges, but return 0 (identity element for sum) if there is no overlap.

---

## 3. Thinking Process

- **Why a Segment Tree?** 
  - If we use a simple array:
    - Update is $O(1)$.
    - Range query is $O(N)$ (iterating from $L$ to $R$).
  - If we use a prefix sum array:
    - Range query is $O(1)$.
    - Update is $O(N)$ (rebuilding the prefix sums).
  - A Segment Tree balances both operations to $O(\log N)$ time, making it ideal for frequent updates and queries.
- **Structural Properties:**
  - A Segment Tree is a binary tree where each node represents an interval of the array.
  - The root node represents the interval $[0, N-1]$.
  - If a node represents $[start, end]$, its left child represents $[start, \frac{start+end}{2}]$, and its right child represents $[\frac{start+end}{2} + 1, end]$.
- **Memory Representation:**
  - Since it is a nearly full binary tree, we can represent it using a flat array `tree` of size $4N$.
  - 1-indexed representation:
    - Root node index = `1`.
    - Left child of node `i` = `2 * i`.
    - Right child of node `i` = `2 * i + 1`.
- **Query Intervals Intersection:**
  When querying $[L, R]$ against a node representing $[start, end]$:
  1. **Complete Overlap:** $[start, end]$ is entirely inside $[L, R]$ ($L \le start$ and $end \le R$). We return `tree[node]`.
  2. **No Overlap:** $[start, end]$ is completely outside $[L, R]$ ($end < L$ or $start > R$). We return 0.
  3. **Partial Overlap:** Return `query(left_child) + query(right_child)`.

---

## 4. Pattern Recognition

- **Divide and Conquer / Segment Partitioning:** Representing ranges as combinations of power-of-two subsegments.
- **Tree Flattening:** Storing a binary tree structure in a sequential array.

---

## 5. Brute Force Solution

Using a simple array.
- **Query**: Loop from $L$ to $R$ and sum.
- **Update**: Directly modify `arr[idx] = val`.

### Complexity Analysis
- **Time Complexity**: Build: $O(1)$, Query: $O(N)$, Update: $O(1)$.
- **Space Complexity**: $O(N)$ to store the array.
- **Downside**: Extremely slow for a large number of queries.

---

## 6. Better Solution

Using a Prefix Sum array.
- **Query**: `prefixSum[R] - prefixSum[L-1]`.
- **Update**: Recompute the prefix sum array from index $idx$ to $N-1$.

### Complexity Analysis
- **Time Complexity**: Build: $O(N)$, Query: $O(1)$, Update: $O(N)$.
- **Space Complexity**: $O(N)$ to store prefix sums.
- **Downside**: Slow updates.

---

## 7. Optimal Solution

Using a Segment Tree.

### C++17 Code

```cpp
#include <vector>
#include <iostream>

class SegmentTree {
private:
    int n;
    std::vector<int> tree;

    // Helper to build the segment tree
    void build(const std::vector<int>& arr, int node, int start, int end) {
        // Base Case: Leaf node stores single element of array
        if (start == end) {
            tree[node] = arr[start];
            return;
        }

        int mid = start + (end - start) / 2;
        int leftChild = 2 * node;
        int rightChild = 2 * node + 1;

        // Recursively build left and right subtrees
        build(arr, leftChild, start, mid);
        build(arr, rightChild, mid + 1, end);

        // Internal node stores the sum of its children
        tree[node] = tree[leftChild] + tree[rightChild];
    }

    // Helper for range query
    int queryRange(int node, int start, int end, int L, int R) {
        // Case 1: No overlap
        if (R < start || L > end) {
            return 0; // Identity element for sum
        }

        // Case 2: Complete overlap
        if (L <= start && end <= R) {
            return tree[node];
        }

        // Case 3: Partial overlap
        int mid = start + (end - start) / 2;
        int leftSum = queryRange(2 * node, start, mid, L, R);
        int rightSum = queryRange(2 * node + 1, mid + 1, end, L, R);

        return leftSum + rightSum;
    }

    // Helper for point update
    void updatePoint(int node, int start, int end, int idx, int val) {
        // Base Case: Reached leaf node to update
        if (start == end) {
            tree[node] = val;
            return;
        }

        int mid = start + (end - start) / 2;
        int leftChild = 2 * node;
        int rightChild = 2 * node + 1;

        if (idx <= mid) {
            // Update left child
            updatePoint(leftChild, start, mid, idx, val);
        } else {
            // Update right child
            updatePoint(rightChild, mid + 1, end, idx, val);
        }

        // Recalculate parent value after child update
        tree[node] = tree[leftChild] + tree[rightChild];
    }

public:
    // Constructor
    SegmentTree(const std::vector<int>& arr) {
        n = static_cast<int>(arr.size());
        // Size of tree is at most 4 * n
        tree.resize(4 * n, 0);
        if (n > 0) {
            build(arr, 1, 0, n - 1);
        }
    }

    // Public Query API
    int query(int L, int R) {
        return queryRange(1, 0, n - 1, L, R);
    }

    // Public Update API
    void update(int idx, int val) {
        updatePoint(1, 0, n - 1, idx, val);
    }
};
```

---

## 8. Code Walkthrough

- **`SegmentTree(arr)`**:
  - Sets `n` to the array size and resizes `tree` to `4 * n`.
  - Begins the recursive build at `node = 1` representing interval `[0, n-1]`.
- **`build(arr, node, start, end)`**:
  - If `start == end`, it's a leaf node. We set `tree[node] = arr[start]`.
  - Else, we calculate `mid`, recursively call `build` on the left child (`2 * node`) and the right child (`2 * node + 1`), and store their sum in `tree[node]`.
- **`queryRange(node, start, end, L, R)`**:
  - Resolves overlaps:
    - **No overlap**: Returns `0` if `R < start` or `L > end`.
    - **Complete overlap**: Returns `tree[node]` if `L <= start` and `end <= R`.
    - **Partial overlap**: Recurses on left and right children, returning the sum of the results.
- **`updatePoint(node, start, end, idx, val)`**:
  - Recursively searches for the leaf corresponding to index `idx`.
  - If `idx <= mid`, moves to the left child; else moves to the right child.
  - Updates the leaf, then recalculates `tree[node] = tree[2*node] + tree[2*node+1]` bottom-up during stack unwinding.

---

## 9. Dry Run

Let's dry run on `arr = [1, 3, 5]` of size $N=3$.

### Tree Representation Map
- Node indices mapping intervals:
  - `1` represents `[0, 2]`
  - `2` represents `[0, 1]`
  - `3` represents `[2, 2]` (Leaf: `5`)
  - `4` represents `[0, 0]` (Leaf: `1`)
  - `5` represents `[1, 1]` (Leaf: `3`)

Structure of `tree`:
- `tree[4] = 1`, `tree[5] = 3`
- `tree[2] = tree[4] + tree[5] = 4`
- `tree[3] = 5`
- `tree[1] = tree[2] + tree[3] = 9`

---

### Operations

#### 1. Query Range `[1, 2]` (Expected: 8)
- Call `queryRange(node=1, [0, 2], L=1, R=2)`: Partial overlap.
  - Call `queryRange(node=2, [0, 1], L=1, R=2)`: Partial overlap.
    - Call `queryRange(node=4, [0, 0], L=1, R=2)`: No overlap. Return `0`.
    - Call `queryRange(node=5, [1, 1], L=1, R=2)`: Complete overlap. Return `tree[5] = 3`.
    - Node 2 returns `0 + 3 = 3`.
  - Call `queryRange(node=3, [2, 2], L=1, R=2)`: Complete overlap. Return `tree[3] = 5`.
  - Node 1 returns `3 + 5 = 8`.
- Result: **8** (Correct).

#### 2. Update Index 1 to 10 (`arr` becomes `[1, 10, 5]`)
- Call `updatePoint(node=1, [0, 2], idx=1, val=10)`.
  - `idx = 1 <= mid = 1`. Recurse left child: `updatePoint(node=2, [0, 1])`.
    - `idx = 1 > mid = 0`. Recurse right child: `updatePoint(node=5, [1, 1])`.
      - `start == end`. Set `tree[5] = 10`. Return.
    - Recalculate Node 2: `tree[2] = tree[4] + tree[5] = 1 + 10 = 11`.
  - Recalculate Node 1: `tree[1] = tree[2] + tree[3] = 11 + 5 = 16`.
- Tree successfully updated!

---

## 10. Complexity Analysis

| Operation | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Build** | $O(N)$ | $O(N)$ | We visit every node in the Segment Tree once. The number of nodes is $< 4N$. |
| **Query** | $O(\log N)$ | $O(\log N)$ | At each level of the tree, we visit at most 4 nodes. The height of the tree is $O(\log N)$. |
| **Update** | $O(\log N)$ | $O(\log N)$ | We traverse down a single path from the root to the leaf node of length $O(\log N)$. |

---

## 11. Common Mistakes

1. **Incorrect size array allocation:** Allocating size $2N$ instead of $4N$. If $N$ is not a power of 2, the index path can exceed $2N$. Always use $4N$.
2. **Off-by-one in mid calculation:** Using `mid = (start + end) / 2` is fine, but make sure the left child gets `[start, mid]` and the right child gets `[mid + 1, end]`.
3. **Incorrect identity element:** For range sum, the identity is `0`. For range min queries, the identity is `INT_MAX`. For range max, it is `INT_MIN`.

---

## 12. Edge Cases

| Edge Case | Input | Handling |
| :--- | :--- | :--- |
| **$N = 1$** | `arr = [5]` | Tree of size 4. Build registers `tree[1] = 5`. Query and update work instantly. |
| **Range outside array** | Query `[5, 10]` for $N=3$ | Returns identity `0` since overlap checks fail. |
| **Negative values** | `[-2, -5]` | Sum logic naturally handles negative arithmetic. |

---

## 13. Interview Explanation

"A Segment Tree is a binary tree structure used to store information about intervals or segments of an array. It allows us to perform range queries (like sum, minimum, or maximum) and point updates in logarithmic time.

To build it, we recursively split the array into halves until we reach leaf nodes, which represent individual elements of the array. The internal nodes store the merged results of their left and right children. 

When querying a range $[L, R]$, we traverse the tree and divide the nodes into three scenarios: no overlap (where we return the identity element like 0), complete overlap (where we return the precomputed value stored at that node), and partial overlap (where we recursively query both children and merge their answers). 

For point updates, we traverse down to the specific leaf node representing the index, update its value, and recalculate parent node values on the way back up the call stack. This achieves optimal $O(\log N)$ time for both operations."

---

## 14. Follow-up Questions

1. **What is Lazy Propagation?**
   - *Answer:* Lazy Propagation is an optimization for performing range updates in $O(\log N)$ time instead of $O(N \log N)$. We delay updates to children until those nodes are explicitly accessed, storing pending updates in a `lazy` array.
2. **Can we implement a Segment Tree iteratively?**
   - *Answer:* Yes, we can build and query the segment tree iteratively using a flat array of size $2N$ by placing leaves at indices $N$ to $2N-1$ and traversing upwards.

---

## 15. Variations

1. **Range Minimum/Maximum Query:** Change the merge operation from `left + right` to `std::min(left, right)` or `std::max(left, right)`.
2. **Range GCD Query:** Store the greatest common divisor of the children at each node.

---

## 16. Revision Notes

- **Memory Formula:** Always allocate size $4N$.
- **Indexing Convention:** Left child is `2 * node`, right child is `2 * node + 1`.
- **Identity values table:**
  - Sum: `0`
  - Min: `INT_MAX`
  - Max: `INT_MIN`
  - GCD: `0` (since $\gcd(0, x) = x$)

---

## 17. Memorization Version

```text
Segment Tree (1-indexed):
1. Size = 4 * N. Root at node = 1, range [0, N-1].
2. Build(node, start, end):
   - If start == end: tree[node] = arr[start]; return.
   - build(left), build(right); tree[node] = tree[left] + tree[right].
3. Query(node, start, end, L, R):
   - Out of range: return 0.
   - In range: return tree[node].
   - Partial: return query(left) + query(right).
4. Update(node, start, end, idx, val):
   - If start == end: tree[node] = val; return.
   - Recurse left or right child; tree[node] = tree[left] + tree[right].
Complexity: Build O(N), Query/Update O(log N).
```
