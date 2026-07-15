# Check if Two Trees are Identical

## 1. Problem Statement

Given the roots of two binary trees `p` and `q`, write a function to check if they are the same or **identical**.

Two binary trees are considered identical if they are structurally identical, and the nodes have the exact same values at corresponding positions.

### Input
- `p`: A pointer to the root of the first binary tree.
- `q`: A pointer to the root of the second binary tree.

### Output
- A boolean (`true` or `false`) indicating whether the two trees are identical.

### Constraints
- The number of nodes in both trees is in the range `[0, 100]` (or up to `10^4`).
- Node values: `-10^4 <= Node.val <= 10^4`.

### Observations
- Two trees are identical if:
  1. Their roots have the same value (or both are null).
  2. Their left subtrees are identical.
  3. Their right subtrees are identical.
- This is a structural recursion check, meaning both the node values and the shape of the trees must match.

---

## 2. Interview Clarification

Before starting, verify:
1. **Empty Trees**: "If both trees are null, are they considered identical?"
   - *Interviewer response:* Yes, returns `true`.
2. **One Empty Tree**: "If one tree is null but the other is not, does it return `false`?"
   - *Interviewer response:* Yes.
3. **Iterative Solution**: "Do you want to see an iterative approach using queues/stacks, or is the recursive DFS approach sufficient?"
   - *Interviewer response:* Start with the recursive DFS since it is highly intuitive. If we have time, explain the iterative version using a queue (like a double-BFS).

---

## 3. Thinking Process

### First Instinct (Recursive DFS)
- We compare the current nodes of both trees:
  - If both `p` and `q` are null, they are identical (`true`).
  - If only one of `p` or `q` is null, they cannot be identical (`false`).
  - If their values differ (`p->val != q->val`), they cannot be identical (`false`).
  - Otherwise, recursively check their left children and right children:
    `isSameTree(p->left, q->left) && isSameTree(p->right, q->right)`.

### Transition to Iterative Approach (Double BFS)
- To do this iteratively, we can use a single queue (or two queues) to perform BFS.
- Let's use a single queue to store pairs of nodes that should be compared: `std::queue<pair<TreeNode*, TreeNode*>>`.
- **Algorithm (BFS)**:
  1. Push `{p, q}` to the queue.
  2. While the queue is not empty:
     - Pop the front pair `{currP, currQ}`.
     - If both are null, continue (valid leaf boundary).
     - If only one is null, or their values are different, return `false`.
     - Push `{currP->left, currQ->left}`.
     - Push `{currP->right, currQ->right}`.
  3. If the loop completes without finding differences, return `true`.

---

## 4. Pattern Recognition

- **Category**: Binary Tree Comparison.
- **Pattern**: Dual-Pointer Traversal / Structural Recursion.
- **Why**: When comparing two trees, we traverse them simultaneously. Any mismatch in structure (null vs non-null) or content (different values) triggers an immediate early return.

---

## 5. Brute Force Solution

*Note: The recursive DFS solution is the standard approach and runs in optimal $\mathcal{O}(N)$ time. There is no slower "brute force" variation; the iterative approach is an alternative that avoids recursive call stack limits.*

---

## 6. Better Solution

*Note: Since the recursive method is the baseline, we will detail it as the Optimal Solution below. We will also provide the iterative version under the Iterative section.*

---

## 7. Optimal Solution (Recursive DFS)

The recursive DFS solution is clean, direct, and performs early pruning on mismatches.

### Algorithm
1. Base cases:
   - If both `p` and `q` are `nullptr`, return `true`.
   - If one of `p` or `q` is `nullptr` (but not both), return `false`.
2. Value check: If `p->val != q->val`, return `false`.
3. Recurse: Return `isSameTree(p->left, q->left) && isSameTree(p->right, q->right)`.

---

### Why this is Optimal
It visits each node at most once, performing a constant number of comparisons. It exits immediately as soon as a mismatch in structure or value is detected, minimizing execution time.

---

### Beautiful C++17 Code
```cpp
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
};

class SolutionOptimal {
public:
    bool isSameTree(TreeNode* p, TreeNode* q) {
        // Base case: both nodes are null (reached leaf boundaries together)
        if (p == nullptr && q == nullptr) {
            return true;
        }
        
        // Base case: one node is null and the other is not (structural mismatch)
        if (p == nullptr || q == nullptr) {
            return false;
        }
        
        // Value check: node values must match
        if (p->val != q->val) {
            return false;
        }
        
        // Recursively check left and right subtrees
        return isSameTree(p->left, q->left) && isSameTree(p->right, q->right);
    }
};
```

---

### Alternate Solution (Iterative BFS Queue)
```cpp
#include <queue>
#include <utility>

class SolutionIterative {
public:
    bool isSameTree(TreeNode* p, TreeNode* q) {
        std::queue<std::pair<TreeNode*, TreeNode*>> todo;
        todo.push({p, q});
        
        while (!todo.empty()) {
            auto [currP, currQ] = todo.front();
            todo.pop();
            
            // Both are null - valid match
            if (currP == nullptr && currQ == nullptr) {
                continue;
            }
            
            // One is null, or values don't match - invalid
            if (currP == nullptr || currQ == nullptr) {
                return false;
            }
            if (currP->val != currQ->val) {
                return false;
            }
            
            // Push children pairs to verify
            todo.push({currP->left, currQ->left});
            todo.push({currP->right, currQ->right});
        }
        
        return true;
    }
};
```

---

## 8. Code Walkthrough

Let's trace the recursive DFS base cases:
- `if (p == nullptr && q == nullptr) return true;`: Stops the recursion when both pointers reach the end of a branch.
- `if (p == nullptr || q == nullptr) return false;`: If one node is null and the other is not, it means the tree shapes differ. We return `false` immediately.
- `if (p->val != q->val) return false;`: Checks if the node values are different.
- `isSameTree(p->left, q->left) && isSameTree(p->right, q->right)`: Ensures both the left and right subtrees are identical before returning `true`.

---

## 9. Dry Run

Let's dry run the recursive solution with these two trees:
`p`:
```
    1
   /
  2
```
`q`:
```
    1
     \
      2
```

- Call `isSameTree(p, q)` (nodes `1` and `1`):
  - Values match (`1 == 1`).
  - Calls `isSameTree(p->left, q->left)` (node `2` and `nullptr`):
    - `p` is not null, but `q` is null.
    - Matches the second base case: `p == nullptr || q == nullptr`.
    - Returns `false`.
  - The logical AND operator `&&` short-circuits, skipping the right subtree check.
- Returns `false`. Correct.

---

## 10. Complexity Analysis

| Metric | Recursive DFS (Optimal) | Iterative BFS |
|:---|:---|:---|
| **Time Complexity** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Space Complexity (Worst)**| $\mathcal{O}(N)$ (skewed tree stack) | $\mathcal{O}(W) \approx \mathcal{O}(N)$ (widest level queue) |
| **Space Complexity (Best)** | $\mathcal{O}(\log N)$ (balanced tree)| $\mathcal{O}(1)$ (skewed tree) |

*Note: Here, $N$ is the number of nodes in the smaller of the two trees, as the traversal stops early upon finding any structural differences.*

---

## 11. Common Mistakes

1. **Incorrect Base Case Ordering**: Placing the check `p == nullptr || q == nullptr` *before* the check `p == nullptr && q == nullptr`. If both are null, the first check will evaluate to true and incorrectly return `false`. Always check if both are null first.
2. **Missing Logical Short-Circuit**: Writing code that continues to traverse the rest of the tree even after a mismatch is found, wasting execution time.

---

## 12. Edge Cases

| Edge Case | Input Trees | Why it matters | How Code Handles It |
|:---|:---|:---|:---|
| **Both Trees Empty** | `nullptr`, `nullptr` | Minimal edge case | Handled by the first base case, returns `true`. |
| **One Tree Empty** | `[1]`, `nullptr` | Structural mismatch | Handled by the second base case, returns `false`. |
| **Same Shape, Different Values** | `[1,2]`, `[1,3]` | Value mismatch | Handled by value comparison, returns `false`. |
| **Different Shape, Same Values** | `[1,2,null]`, `[1,null,2]` | Shape mismatch | Handled by structural null check, returns `false`. |

---

## 13. Interview Explanation

"To determine if two binary trees are identical, we must verify that they have the exact same structure and the same values at every position.

We can solve this recursively using a dual-pointer DFS traversal. 

At each step, we compare the current nodes of both trees:
- If both nodes are null, we have reached the end of matching branches, so we return `true`.
- If only one of the nodes is null, or if their values do not match, we have found a structural or value mismatch, and we return `false`.
- Otherwise, we recursively check if their left subtrees are identical, and if their right subtrees are identical.

This recursive approach runs in $\mathcal{O}(N)$ time, where $N$ is the number of nodes, and uses $\mathcal{O}(H)$ call stack space. 

Alternatively, we can implement this iteratively using a queue to perform a double BFS, checking pairs of nodes popped from the queue."

---

## 14. Follow-up Questions

1. **Q**: Can you check if one tree is a subtree of another?
   - **A**: Yes (the "Subtree of Another Tree" problem). We can traverse the main tree, and at each node, call `isSameTree` to check if the subtree matches the target tree.
2. **Q**: How would you optimize this if we had to compare very large trees frequently?
   - **A**: We could pre-compute cryptographic hashes (like SHA or Merkle tree hashes) for all subtrees. Comparing two subtrees would then become a simple $\mathcal{O}(1)$ hash comparison.

---

## 15. Variations

1. **Subtree of Another Tree**: Check if tree $S$ is structurally identical to a subtree of tree $T$.
2. **Symmetric Tree**: Check if a tree is a mirror image of itself (calls a variation of `isSameTree(root->left, root->right)` where we compare left to right).

---

## 16. Revision Notes

- **Identity Conditions**: Structural match (null checks) AND value match.
- **Base Case Order**: Check `both null` before `either null`.
- **Short-circuiting**: Using `&&` ensures that once a mismatch is found, further recursion is stopped.

---

## 17. Memorization Version

```cpp
// Recursive Identical: Check nulls, check values, recurse children.
bool isSameTree(TreeNode* p, TreeNode* q) {
    if (!p && !q) return true;
    if (!p || !q) return false;
    return (p->val == q->val) && 
           isSameTree(p->left, q->left) && 
           isSameTree(p->right, q->right);
}
```
