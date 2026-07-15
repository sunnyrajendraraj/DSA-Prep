# Lowest Common Ancestor of a Binary Tree

## 1. Problem Statement

Given a binary tree, find the **Lowest Common Ancestor (LCA)** of two given nodes, `p` and `q`.

According to the definition of LCA on Wikipedia: “The lowest common ancestor is defined between two nodes $p$ and $q$ as the lowest node in $T$ that has both $p$ and $q$ as descendants (where we allow a node to be a descendant of itself).”

### Input
- `root`: A pointer to the root node of a general binary tree.
- `p`: Pointer to the first node.
- `q`: Pointer to the second node.

### Output
- A pointer to the TreeNode representing the lowest common ancestor of `p` and `q`.

### Constraints
- The number of nodes in the tree is in the range `[2, 10^5]`.
- Node values: `-10^9 <= Node.val <= 10^9`.
- All `Node.val` are unique.
- `p` and `q` exist in the tree and `p != q`.

### Observations
- Unlike a BST, we cannot use node values to decide which subtree to search. We must search the entire tree using DFS.
- For any node in the tree:
  - If one target node lies in its left subtree and the other target node lies in its right subtree, then the current node **must** be the LCA.
  - If both target nodes lie in its left subtree, the LCA must also be in the left subtree.
  - If both target nodes lie in its right subtree, the LCA must also be in the right subtree.

---

## 2. Interview Clarification

Before starting, verify:
1. **Node Existence**: "Are `p` and `q` guaranteed to exist in the tree?"
   - *Interviewer response:* Yes.
2. **Nodes pointing to themselves**: "Can a node be an ancestor of itself?"
   - *Interviewer response:* Yes, if `p` is a parent of `q`, then `p` is the LCA.
3. **Empty Tree**: "Can the root be null?"
   - *Interviewer response:* Technically yes, though the constraints say at least 2 nodes exist. You should still handle the null check.

---

## 3. Thinking Process

### First Instinct (Recursive DFS)
- Let's search for `p` and `q` bottom-up using recursive DFS.
- When we visit a node:
  - If the node is null, we return null.
  - If the node matches either `p` or `q`, we return the node itself.
  - Recursively search the left subtree: `leftResult = lowestCommonAncestor(node->left, p, q)`.
  - Recursively search the right subtree: `rightResult = lowestCommonAncestor(node->right, p, q)`.
  - Evaluate the results returned from the subtrees:
    - If both `leftResult` and `rightResult` are non-null, it means one target was found on the left and the other on the right. The current node is the LCA, so we return it.
    - If only one of the results is non-null, it means both targets lie within that subtree (or one target was found and the other is a descendant of it). We propagate the non-null result upward.
    - If both results are null, we return null.

---

## 4. Pattern Recognition

- **Category**: Binary Tree Bottom-Up DFS.
- **Pattern**: Recursion with Dual-Branch Convergence.
- **Why**: Finding the LCA requires identifying where two separate search paths converge. In a bottom-up DFS, this convergence point is the node where both subtree calls return non-null values.

---

## 5. Brute Force Solution

*Note: Since the recursive DFS solution is the standard approach and runs in optimal $\mathcal{O}(N)$ time, there is no slower "brute force" variation; we will detail the recursive version below as the Optimal Solution.*

---

## 6. Better Solution

*Note: The recursive DFS solution is the direct optimal solution for this problem. We will proceed to detail it.*

---

## 7. Optimal Solution (Recursive DFS)

The recursive DFS solution is clean, elegant, and handles all path split and parent-child scenarios.

### Algorithm
1. Base cases:
   - If `root` is null, return `nullptr`.
   - If `root` is equal to `p` or `q`, return `root`.
2. Recursively search subtrees:
   - `leftLCA = lowestCommonAncestor(root->left, p, q)`
   - `rightLCA = lowestCommonAncestor(root->right, p, q)`
3. Evaluate results:
   - If both `leftLCA` and `rightLCA` are non-null, return `root` (split point).
   - If `leftLCA` is not null (and `rightLCA` is null), return `leftLCA`.
   - If `rightLCA` is not null (and `leftLCA` is null), return `rightLCA`.
   - If both are null, return `nullptr`.

---

### Why this is Optimal
It visits each node at most once, running in $\mathcal{O}(N)$ time. The space complexity is $\mathcal{O}(H)$ stack frames, which is the minimum required to traverse a general tree without parent pointers.

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
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        // Base case: if we hit a null node, or find either target node, return it
        if (root == nullptr || root == p || root == q) {
            return root;
        }
        
        // Recurse into left and right subtrees
        TreeNode* leftLCA = lowestCommonAncestor(root->left, p, q);
        TreeNode* rightLCA = lowestCommonAncestor(root->right, p, q);
        
        // If both subtrees returned a non-null node, the current node is the LCA
        if (leftLCA != nullptr && rightLCA != nullptr) {
            return root;
        }
        
        // Otherwise, return the non-null result (if any)
        return (leftLCA != nullptr) ? leftLCA : rightLCA;
    }
};
```

---

## 8. Code Walkthrough

Let's trace the recursive decision logic:
- `if (root == nullptr || root == p || root == q) return root;`: This is the base case. If we find either `p` or `q`, we return it. We don't need to search its descendants because even if the other target node is a descendant, the current node is still the LCA.
- `leftLCA = ...` and `rightLCA = ...`: Searches both subtrees.
- `if (leftLCA != nullptr && rightLCA != nullptr) return root;`: If one target is found on the left and the other on the right, the current node is the LCA.
- `return (leftLCA != nullptr) ? leftLCA : rightLCA;`: If only one side returns a node, it means both targets lie within that subtree. We propagate this node upward.

---

## 9. Dry Run

Let's dry run the optimal solution with this tree:
```
        3
       / \
      5   1
     / \
    6   2
```
Targets: `p = 5`, `q = 1`.

- Call `lowestCommonAncestor(3, 5, 1)`:
  - `3` is not null, `3 != 5`, `3 != 1`.
  - Call `lowestCommonAncestor(3->left (5), 5, 1)`:
    - Current node is `5` which matches `p`.
    - Returns `5`.
  - Call `lowestCommonAncestor(3->right (1), 5, 1)`:
    - Current node is `1` which matches `q`.
    - Returns `1`.
  - Back to `3`:
    - `leftLCA = 5` (non-null), `rightLCA = 1` (non-null).
    - Both are non-null $\to$ Returns `3`.
- Final LCA returned: `3`. Correct.

---

## 10. Complexity Analysis

| Metric | Recursive DFS (Optimal) |
|:---|:---|
| **Time Complexity (Best)** | $\mathcal{O}(H)$ (targets found near the root) |
| **Time Complexity (Avg)** | $\mathcal{O}(N)$ |
| **Time Complexity (Worst)**| $\mathcal{O}(N)$ |
| **Space Complexity (Worst)**| $\mathcal{O}(N)$ (skewed tree stack) |
| **Space Complexity (Best)** | $\mathcal{O}(\log N)$ (balanced tree) |

---

## 11. Common Mistakes

1. **Unnecessary Descendant Searching**: Continuing to search the descendants of a node after finding `p` or `q`. This is unnecessary because if the other target node is a descendant, the current node is still the LCA.
2. **Incorrect Return Evaluation**: Forgetting to check the case where both child subtrees return non-null values.

---

## 12. Edge Cases

| Edge Case | Input Tree | Why it matters | How Code Handles It |
|:---|:---|:---|:---|
| **One Target is Root** | `p` is root | Split check validation | Returns `root` immediately via base case. |
| **Targets in Same Subtree** | `p` and `q` are on the left | Unilateral propagation | Returns `leftLCA` correctly. |
| **Skewed Tree** | Line tree `3 -> 2 -> 1` | Stack depth matches height $N$ | Handled correctly. Uses $\mathcal{O}(N)$ stack memory. |

---

## 13. Interview Explanation

"To find the Lowest Common Ancestor (LCA) in a general binary tree, we can use a bottom-up recursive DFS traversal.

We search for the target nodes `p` and `q` recursively. 

At each node:
- If we reach a null node, or if we find either target node, we return it.
- We then recursively search the left and right subtrees.
- If both subtree searches return non-null nodes, it means one target node lies in the left subtree and the other in the right subtree. The current node must be the LCA, so we return it.
- If only one subtree search returns a non-null node, it means both target nodes lie within that subtree, so we propagate that node upward.

This recursive approach runs in $\mathcal{O}(N)$ time and uses $\mathcal{O}(H)$ call stack space."

---

## 14. Follow-up Questions

1. **Q**: What if the nodes `p` and `q` are not guaranteed to exist in the tree?
   - **A**: We would first need to verify if both nodes exist. If they do, we run the LCA algorithm; if not, we return `nullptr`.
2. **Q**: Can you implement this iteratively using parent pointers?
   - **A**: Yes. If we have parent pointers, we can traverse upwards from `p` to the root, storing all ancestors in a hash set. Then, we traverse upwards from `q` and return the first ancestor that exists in the hash set.

---

## 15. Variations

1. **Lowest Common Ancestor in a BST**: Solve using the sorted property of BSTs for improved efficiency.
2. **LCA with Parent Pointers**: Solve using ancestor tracking in a hash set.

---

## 16. Revision Notes

- **LCA Decision Criteria**:
  - `leftLCA && rightLCA` $\to$ Current node is LCA.
  - `leftLCA || rightLCA` $\to$ Propagate the non-null result.
- **Base Case**: Return `root` if it is null or matches either target node.

---

## 17. Memorization Version

```cpp
// Recursive BT LCA: Returns node if found, propagates results, returns root if split.
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    if (!root || root == p || root == q) return root;
    TreeNode* left = lowestCommonAncestor(root->left, p, q);
    TreeNode* right = lowestCommonAncestor(root->right, p, q);
    if (left && right) return root;
    return left ? left : right;
}
```
