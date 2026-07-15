# Fenwick Tree / Binary Indexed Tree (BIT)

## 1. Problem Statement

Implement a **Fenwick Tree** (also known as a **Binary Indexed Tree - BIT**) to support:
- **Point Update**: Add a value `delta` to the element at a given index `i`.
- **Prefix Sum Query**: Compute the sum of the elements from the beginning of the array up to index `i` (inclusive).
- **Range Sum Query**: Compute the sum of the elements in a range $[L, R]$ (inclusive).

All operations (excluding range sum which uses two prefix queries) must run in $O(\log N)$ time. The implementation should be 1-indexed internally.

### Input
- Array size $N$.
- A series of update and query requests.

### Constraints
- $1 \le N \le 10^5$.
- values: $-10^5 \le \text{val} \le 10^5$.

---

## 2. Interview Clarification

1. **Does the Fenwick Tree support Range Minimum Queries (RMQ) easily?**
   - *Answer:* No. Fenwick Trees require the merge operation to be invertible (like addition/subtraction or multiplication/division) to compute range queries as `prefix(R) - prefix(L-1)`. Operations like minimum or maximum are not easily invertible, so Segment Trees are preferred for RMQ.
2. **What is the space complexity compared to Segment Trees?**
   - *Answer:* Fenwick Tree uses exactly $N+1$ space, whereas a Segment Tree typically uses $4N$ space.
3. **Are input indices 0-indexed?**
   - *Answer:* Yes, we will receive 0-indexed operations, but we will convert them to 1-indexed inside our Fenwick Tree class.

---

## 3. Thinking Process

### The Core Concept of BIT
Every positive integer can be represented as a sum of powers of 2. 
Similarly, a range sum can be represented as a sum of sub-ranges.
In a Fenwick Tree, the node at index `i` stores the sum of a range of length equal to the value of the **least significant set bit** of `i`.

### The Lowbit Trick (`i & -i`)
How do we isolate the lowest set bit of an integer `i`?
In two's complement, `-i` is equivalent to `~i + 1`. 
Performing a bitwise AND between `i` and `-i` isolates the rightmost set bit:
$$\text{lowbit}(i) = i \ \& \ (-i)$$

### Traversal Logic
1. **Query (Prefix Sum up to `i`):**
   - We start at index `i`.
   - Add `tree[i]` to our sum.
   - Strip off the lowest set bit: `i -= (i & -i)`.
   - Repeat until `i <= 0`.
2. **Update (Add `delta` at index `i`):**
   - We start at index `i`.
   - Add `delta` to `tree[i]`.
   - Move to the next index containing this range by adding the lowest set bit: `i += (i & -i)`.
   - Repeat until `i > N`.

---

## 4. Pattern Recognition

- **Binary Representation & Bitwise Operations:** Using properties of two's complement and bitwise operations to compute range splits.
- **Prefix Computations:** Transforming range queries into differences of prefixes.

---

## 5. Brute Force Solution

Using a standard array:
- **Update**: `arr[i] += delta` $\Rightarrow O(1)$
- **Prefix Query**: Loop from $0$ to $i$ summing elements $\Rightarrow O(N)$

---

## 6. Better Solution

There is no intermediate representation between the brute force array/prefix-array and the optimal BIT. We proceed directly to the optimal implementation.

---

## 7. Optimal Solution

### C++17 Code

```cpp
#include <vector>
#include <iostream>

class FenwickTree {
private:
    int size;
    std::vector<int> tree;

public:
    // Constructor initialized with size N
    FenwickTree(int n) {
        this->size = n;
        // 1-indexed tree of size N + 1
        tree.assign(n + 1, 0);
    }

    // Constructor initialized with a given 0-indexed array
    FenwickTree(const std::vector<int>& arr) : FenwickTree(static_cast<int>(arr.size())) {
        for (int i = 0; i < static_cast<int>(arr.size()); ++i) {
            update(i, arr[i]);
        }
    }

    // 1-indexed Point Update: Add delta to index 'i' (0-indexed externally)
    void update(int i, int delta) {
        int idx = i + 1; // Convert to 1-based index
        while (idx <= size) {
            tree[idx] += delta;
            idx += idx & (-idx); // Add lowbit
        }
    }

    // 1-indexed Prefix Sum Query: sum from index 0 to 'i' (0-indexed externally)
    int query(int i) const {
        int idx = i + 1; // Convert to 1-based index
        int sum = 0;
        while (idx > 0) {
            sum += tree[idx];
            idx -= idx & (-idx); // Subtract lowbit
        }
        return sum;
    }

    // Range Sum Query: sum from index L to R (inclusive)
    int queryRange(int L, int R) const {
        if (L > R) return 0;
        return query(R) - query(L - 1);
    }
};
```

---

## 8. Code Walkthrough

- **`tree.assign(n + 1, 0)`**: Allocates the flat vector structure. Fenwick Trees must be 1-indexed because `idx & (-idx)` equals `0` when `idx` is `0`, leading to an infinite loop.
- **`update(i, delta)`**:
  - `idx = i + 1` shifts 0-indexed inputs to 1-indexed.
  - While `idx <= size`, we increment `tree[idx]` by `delta` and add the least significant bit (`idx += idx & (-idx)`). This cascades the update upwards to all containing parent ranges.
- **`query(i)`**:
  - `idx = i + 1` converts to 1-indexed.
  - We accumulate `sum += tree[idx]` and step down by removing the least significant bit (`idx -= idx & (-idx)`). This traverses the tree upwards towards the root, skipping unnecessary ranges.
- **`queryRange(L, R)`**:
  - Returns `query(R) - query(L - 1)` using prefix sum properties.

---

## 9. Dry Run

Let's dry run the initialization and a query with `arr = [3, 2, -1, 6, 5]`. $N = 5$.

### Step 1: Build the tree using updates
- **`update(0, 3)`**: `idx = 1`.
  - `tree[1] += 3`. `idx += 1 & -1` (1) $\Rightarrow$ `idx = 2`.
  - `tree[2] += 3`. `idx += 2 & -2` (2) $\Rightarrow$ `idx = 4`.
  - `tree[4] += 3`. `idx += 4 & -4` (4) $\Rightarrow$ `idx = 8` ($> 5$, terminate).
- **`update(1, 2)`**: `idx = 2`.
  - `tree[2] += 2`. `idx = 4`.
  - `tree[4] += 2`. `idx = 8` ($> 5$, terminate).
- **`update(2, -1)`**: `idx = 3`.
  - `tree[3] += -1`. `idx += 3 & -3` (1) $\Rightarrow$ `idx = 4`.
  - `tree[4] += -1`. `idx = 8` ($> 5$, terminate).
- **`update(3, 6)`**: `idx = 4`.
  - `tree[4] += 6`. `idx = 8` ($> 5$, terminate).
- **`update(4, 5)`**: `idx = 5`.
  - `tree[5] += 5`. `idx += 5 & -5` (1) $\Rightarrow$ `idx = 6` ($> 5$, terminate).

### Resulting `tree` array (1-indexed):
- `tree[1] = 3`
- `tree[2] = 3 + 2 = 5`
- `tree[3] = -1`
- `tree[4] = 3 + 2 + (-1) + 6 = 10`
- `tree[5] = 5`

---

### Step 2: Query Prefix of index 4 (values `arr[0]` to `arr[4]`)
Expected Sum: $3 + 2 - 1 + 6 + 5 = 15$.
- `query(4)` $\Rightarrow$ `idx = 5`.
  - `sum += tree[5]` ($5$). `idx -= 5 & -5` (1) $\Rightarrow$ `idx = 4`.
  - `sum += tree[4]` ($10$). `idx -= 4 & -4` (4) $\Rightarrow$ `idx = 0` (terminate).
- Total Sum: $5 + 10 = \mathbf{15}$. Correct!

---

## 10. Complexity Analysis

| Operation | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Point Update** | $O(\log N)$ | $O(N)$ | The number of set bits in any integer $\le N$ is at most $O(\log N)$. |
| **Prefix Query** | $O(\log N)$ | $O(N)$ | Similar to update, we clear bits at each step, taking at most $O(\log N)$ steps. |
| **Space Complexity** | — | $O(N)$ | Uses an array of size $N+1$ to store segment data. |

---

## 11. Common Mistakes

1. **0-based indexing:** Forgetting to offset to 1-based indexing. If `idx = 0`, `idx & (-idx)` is `0`, and `idx += 0` or `idx -= 0` causes an infinite loop.
2. **Confusing update direction:** Remember that `update` goes **up** (`idx += idx & -idx`) and `query` goes **down** (`idx -= idx & -idx`).

---

## 12. Edge Cases

| Edge Case | Input | Handling |
| :--- | :--- | :--- |
| **$N = 1$** | `arr = [10]` | Tree size = 2. Update and query steps exit immediately. |
| **Negative updates**| `delta = -10` | The arithmetic handles addition of negative values properly. |

---

## 13. Interview Explanation

"A Fenwick Tree, or Binary Indexed Tree, is a highly space-efficient data structure used to perform prefix sum queries and point updates in logarithmic time. 

It is implemented as a flat, 1-indexed array of size $N + 1$. Each index $i$ stores the cumulative sum of a range of length equal to its least significant set bit, which we isolate using the expression `i & -i`. 

To update a value, we propagate the change to all parent segments by adding the least significant bit to our index at each step until we exceed the array size. 

To query a prefix sum, we accumulate the value at the current index and subtract its least significant bit to move to the next disjoint segment, continuing until the index becomes zero. 

A Fenwick Tree is faster and uses less memory than a Segment Tree, though it is limited to invertible operations like sums."

---

## 14. Follow-up Questions

### Segment Tree vs. Fenwick Tree Comparison Table

| Feature | Segment Tree | Fenwick Tree (BIT) |
| :--- | :--- | :--- |
| **Space Complexity** | $4N$ nodes | $N + 1$ nodes |
| **Implementation Complexity**| Moderate / High | Very simple |
| **Range Operations** | Sum, Min, Max, GCD | Primarily Sum (or invertible ops) |
| **Lazy Propagation** | Supported | Complex / Not standard |
| **Constant Factor** | High | Extremely low |

---

## 15. Variations

1. **Range Update & Point Query:** Store differences in the BIT. To add $X$ in range $[L, R]$, update $(L, X)$ and update $(R+1, -X)$. Point query at $i$ is just the prefix sum up to $i$.
2. **Range Update & Range Query:** Can be solved using two helper BIT arrays.

---

## 16. Revision Notes

- **Lowbit expression:** `x & -x`.
- **Query Traversal:** Subtract lowbit (`idx -= idx & -idx`).
- **Update Traversal:** Add lowbit (`idx += idx & -idx`).
- **Memory footprint:** $O(N)$ auxiliary space compared to $O(4N)$ of Segment Tree.

---

## 17. Memorization Version

```text
Fenwick Tree (BIT):
1. Use 1-indexed array of size N + 1.
2. Lowbit: i & -i
3. Update(i, delta):
   - idx = i + 1
   - while idx <= N: tree[idx] += delta; idx += idx & -idx
4. Query(i):
   - idx = i + 1, sum = 0
   - while idx > 0: sum += tree[idx]; idx -= idx & -idx
   - return sum
Complexity: Time O(log N), Space O(N).
```
