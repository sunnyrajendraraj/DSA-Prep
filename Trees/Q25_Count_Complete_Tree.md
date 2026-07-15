# Count Nodes in a Complete Binary Tree

## 1. Problem Statement

Given the root of a **complete** binary tree, return the number of nodes in the tree.

According to Wikipedia, every level, except possibly the last, is completely filled in a complete binary tree, and all nodes in the last level are as far left as possible. It can have between $1$ and $2^h$ nodes inclusive at the last level $h$.

Design an algorithm that runs in less than $O(N)$ time complexity.

### Input
- The root of a complete binary tree.

### Output
- An integer representing the total count of nodes.

### Constraints
- The number of nodes in the tree is in the range $[0, 5 \cdot 10^4]$.
- $0 \le \text{Node.val} \le 5 \cdot 10^4$.
- The tree is guaranteed to be **complete**.

---

## 2. Interview Clarification

1. **What is the time complexity of the trivial solution?**
   - *Answer:* The trivial solution traverses every node, taking $O(N)$ time. We need to do better by leveraging the "complete" property.
2. **Can we assume the input tree is always a valid complete binary tree?**
   - *Answer:* Yes, you can assume it is complete.
3. **Should we handle integer overflow?**
   - *Answer:* The maximum number of nodes is $5 \cdot 10^4$, so standard 32-bit integers will not overflow.

---

## 3. Thinking Process

- **First instinct:** Standard traversal (DFS or BFS) visits every node. This is $O(N)$ time.
- **Can we do better?** Since it is a *complete binary tree*, many subtrees are actually *perfect binary trees* (every level is fully filled).
- **Property of a Perfect Binary Tree:** If a tree is perfect and has height $h$ (where root is height 1), the number of nodes is exactly $2^h - 1$.
- **How to check if a tree is perfect?**
  - Walk down the leftmost path and compute height $L$.
  - Walk down the rightmost path and compute height $R$.
  - If $L == R$, the tree is perfect! We can immediately calculate its nodes as $2^L - 1$ in $O(L)$ time, without visiting the middle nodes.
- **What if $L \neq R$?**
  - If they are not equal, then the tree is not perfect. However, at least one of its subtrees (either left or right) must be perfect, and the other will be complete.
  - In this case, we can recursively compute:
    $$\text{nodes}(root) = 1 + \text{nodes}(root\to left) + \text{nodes}(root\to right)$$
  - Because the tree is complete, one of the recursive calls will find a perfect subtree and return instantly. The other call will recurse down.
  - Since we discard half of the tree at each level of recursion, the height decreases by 1 at each step, and we perform height calculations ($O(H)$) at each step. This gives $O(H^2)$ which is $O(\log^2 N)$ time.

---

## 4. Pattern Recognition

- **Divide and Conquer (Pruning):** We use the structural properties of complete binary trees to skip traversal of entire subtrees.
- **Tree Height Calculation:** Measuring left and right boundaries to determine structure.

---

## 5. Brute Force Solution

A simple traversal of the tree using DFS or BFS to count all nodes.

### Algorithm
1. If root is null, return 0.
2. Return $1 + \text{countNodes}(root->left) + \text{countNodes}(root->right)$.

### C++17 Code
```cpp
class SolutionBrute {
public:
    int countNodes(TreeNode* root) {
        if (root == nullptr) return 0;
        return 1 + countNodes(root->left) + countNodes(root->right);
    }
};
```

### Complexity Analysis
- **Time Complexity:** $O(N)$ because we visit every single node.
- **Space Complexity:** $O(H)$ where $H$ is the height of the tree (call stack). For a complete tree, $H = O(\log N)$.

---

## 6. Better Solution

Since brute force is $O(N)$ and the optimal is $O(\log^2 N)$, there is no intermediate "better" solution. We move straight to the optimal solution.

---

## 7. Optimal Solution

### Algorithm
1. Write helper functions to compute the left height and right height of a node:
   - For left height: traverse `curr = curr->left` and count.
   - For right height: traverse `curr = curr->right` and count.
2. In the main function `countNodes(root)`:
   - If `root` is null, return 0.
   - Calculate left height `lh` and right height `rh`.
   - If `lh == rh`, the tree is a perfect binary tree. Return $(1 \ll lh) - 1$.
   - If `lh != rh`, return $1 + \text{countNodes}(root->left) + \text{countNodes}(root->right)$.

### C++17 Code

```cpp
#include <iostream>

// Definition for a binary tree node.
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
private:
    // Helper to find the left-most height (1-based)
    int getLeftHeight(TreeNode* node) {
        int height = 0;
        while (node != nullptr) {
            height++;
            node = node->left;
        }
        return height;
    }

    // Helper to find the right-most height (1-based)
    int getRightHeight(TreeNode* node) {
        int height = 0;
        while (node != nullptr) {
            height++;
            node = node->right;
        }
        return height;
    }

public:
    int countNodes(TreeNode* root) {
        if (root == nullptr) {
            return 0;
        }

        int lh = getLeftHeight(root);
        int rh = getRightHeight(root);

        // If left height and right height are equal, it's a perfect binary tree.
        if (lh == rh) {
            // Number of nodes is (2^lh) - 1
            // 1 << lh is equivalent to 2^lh
            return (1 << lh) - 1;
        }

        // Otherwise, recurse on left and right subtrees
        return 1 + countNodes(root->left) + countNodes(root->right);
    }
};
```

---

## 8. Code Walkthrough

- **`getLeftHeight(TreeNode* node)`**:
  - Initializes `height = 0`.
  - Continuously traverses `node = node->left` until it reaches `nullptr`, incrementing `height`.
- **`getRightHeight(TreeNode* node)`**:
  - Continuously traverses `node = node->right` until it reaches `nullptr`, incrementing `height`.
- **`countNodes(TreeNode* root)`**:
  - Base case: if `root` is null, returns 0.
  - Otherwise, checks if left-most path length `lh` equals right-most path length `rh`.
  - If equal, returns $(2^{lh} - 1)$ via `(1 << lh) - 1`.
  - If unequal, computes recursively: `1` (for current root) + `countNodes(root->left)` + `countNodes(root->right)`.

---

## 9. Dry Run

Let's dry run the algorithm with the following complete binary tree:

```
        1
       / \
      2   3
     / \  /
    4   5 6
```

### Recursive Level 0: `root = 1`
1. `getLeftHeight(1)`: Path is $1 \to 2 \to 4 \to \text{null}$. Height `lh = 3`.
2. `getRightHeight(1)`: Path is $1 \to 3 \to \text{null}$ (since 3 has no right child). Height `rh = 2`.
3. `lh != rh` ($3 \neq 2$). Recurse:
   $$\text{return } 1 + \text{countNodes}(2) + \text{countNodes}(3)$$

### Recursive Level 1 (Left): `root = 2`
1. `getLeftHeight(2)`: Path is $2 \to 4 \to \text{null}$. Height `lh = 2`.
2. `getRightHeight(2)`: Path is $2 \to 5 \to \text{null}$. Height `rh = 2`.
3. `lh == rh` ($2 == 2$).
4. Return $(1 \ll 2) - 1 = 4 - 1 = 3$.

### Recursive Level 1 (Right): `root = 3`
1. `getLeftHeight(3)`: Path is $3 \to 6 \to \text{null}$. Height `lh = 2`.
2. `getRightHeight(3)`: Path is $3 \to \text{null}$. Height `rh = 1`.
3. `lh != rh` ($2 \neq 1$). Recurse:
   $$\text{return } 1 + \text{countNodes}(6) + \text{countNodes}(nullptr)$$

### Recursive Level 2 (Left of 3): `root = 6`
1. `lh = 1`, `rh = 1`. Equal!
2. Return $(1 \ll 1) - 1 = 1$.

### Recursive Level 2 (Right of 3): `root = nullptr`
1. Returns `0`.

### Combining Results
- For `root = 3`: $1 + 1 + 0 = 2$.
- For `root = 1`: $1 + 3 + 2 = 6$.

Total nodes: **6**. Correct!

---

## 10. Complexity Analysis

| Metric | Complexity | Reasoning |
| :--- | :--- | :--- |
| **Time Complexity** | $O(\log^2 N)$ | In each step, we calculate height which takes $O(\log N)$ time. We only recurse down one path because one of the subtrees is always perfect and returns in $O(1)$ time. Thus, recursion depth is $O(\log N)$. Total time: $O(\log N \cdot \log N) = O(\log^2 N)$. |
| **Space Complexity** | $O(\log N)$ | The recursion stack height is at most $O(\log N)$ (since the tree is complete). |

---

## 11. Common Mistakes

1. **Incorrect bit shift operator precedence:** Write `(1 << lh) - 1` with correct parentheses. Writing `1 << lh - 1` evaluates to `1 << (lh - 1)` in C++ due to precedence rules, which is incorrect.
2. **Incorrect height definition:** Mixing up 0-based and 1-based heights can lead to calculations like $2^h$ off by one.

---

## 12. Edge Cases

| Edge Case | Example | Handling |
| :--- | :--- | :--- |
| **Empty Tree** | `nullptr` | Returns `0`. |
| **Single Node** | `[1]` | `lh = 1`, `rh = 1`. Returns `(1 << 1) - 1 = 1`. |
| **Perfect Binary Tree** | `1 / \ 2 3` | `lh = 2`, `rh = 2`. Returns `(1 << 2) - 1 = 3`. No recursion. |

---

## 13. Interview Explanation

"To count the nodes of a complete binary tree in less than $O(N)$ time, we leverage the property that a complete binary tree contains many perfect subtrees. 

For any node, we find the height of its leftmost path and the height of its rightmost path. If these two heights are equal, it means that the subtree is a perfect binary tree, and we can immediately compute its total nodes as $2^{\text{height}} - 1$ in constant time. 

If the heights are different, we recursively call the function on both the left and right subtrees and add 1 for the root. Since the tree is complete, at least one of these subtrees is guaranteed to be perfect, terminating that recursive branch immediately. This limits the total traversal to a single path of recursion of depth $O(\log N)$, and at each step we calculate height in $O(\log N)$ time, giving an overall optimal time complexity of $O(\log^2 N)$."

---

## 14. Follow-up Questions

1. **Can you solve this in $O(\log N)$ time?**
   - *Answer:* Yes! We can perform a binary search on the leaves. Since the depth of the tree is $d$, the number of leaf positions is $2^d$. We can binary search the index of the last leaf. Checking if a leaf exists at a certain index takes $O(\log N)$ time by traversing from the root using binary encoding of the path. This yields $O(\log N \cdot \log N) = O(\log^2 N)$ time. There is no strictly faster way than $O(\log^2 N)$ or $O(\log N)$ binary search.

---

## 15. Variations

1. **Find depth of a complete binary tree:** Simply go leftmost.
2. **Check if a binary tree is complete:** Use level-order traversal; once a non-full node is seen, all subsequent nodes must be leaves.

---

## 16. Revision Notes

- **Key Trick:** `lh == rh` check allows pruning an entire subtree.
- **Math Bitwise Operation:** `(1 << h) - 1` calculates $2^h - 1$.
- **Time Complexity:** $O(\log^2 N)$ is much faster than $O(N)$. For $N = 10^6$, $\log_2(N) \approx 20$, and $20^2 = 400$ operations.

---

## 17. Memorization Version

```text
Count Complete Tree Nodes:
1. If root is null, return 0.
2. Compute left height (lh) by going left-only.
3. Compute right height (rh) by going right-only.
4. If lh == rh, return (1 << lh) - 1.
5. Else, return 1 + countNodes(left) + countNodes(right).
Complexity: Time O(log^2 N), Space O(log N).
```
