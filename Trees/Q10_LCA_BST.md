# Lowest Common Ancestor of a Binary Search Tree (BST)

## 1. Problem Statement

Given a binary search tree (BST), find the **Lowest Common Ancestor (LCA)** of two given nodes in the BST.

According to the definition of LCA on Wikipedia: “The lowest common ancestor is defined between two nodes $p$ and $q$ as the lowest node in $T$ that has both $p$ and $q$ as descendants (where we allow a node to be a descendant of itself).”

### Input
- `root`: A pointer to the root node of a BST.
- `p`: Pointer to the first node.
- `q`: Pointer to the second node.

### Output
- A pointer to the TreeNode representing the lowest common ancestor of `p` and `q`.

### Constraints
- The number of nodes in the tree is in the range `[2, 10^5]`.
- Node values: `-10^9 <= Node.val <= 10^9`.
- All `Node.val` are unique.
- `p` and `q` will exist in the BST and `p != q`.

### Observations
- In a BST, for every node:
  - All values in the left subtree are smaller than the node's value.
  - All values in the right subtree are larger than the node's value.
- This BST property allows us to locate the LCA without traversing the entire tree.

---

## 2. Interview Clarification

Before starting, verify:
1. **Node existence**: "Can we assume `p` and `q` are guaranteed to exist in the BST?"
   - *Interviewer response:* Yes.
2. **BST Properties**: "Can we leverage the sorted property of the BST, or should we write a general binary tree solution?"
   - *Interviewer response:* Leverage the BST property to make it as efficient as possible.
3. **Space constraints**: "Can we write it iteratively to achieve $\mathcal{O}(1)$ auxiliary space?"
   - *Interviewer response:* Yes, that is the ideal solution.

---

## 3. Thinking Process

### First Instinct (General Tree LCA)
- We could search the subtrees for `p` and `q` recursively. This is the general binary tree LCA algorithm.
- However, this does not take advantage of the BST property and would traverse the entire tree, taking $\mathcal{O}(N)$ time.

### Leveraging the BST Property
- Let the current node be `curr`.
- If both `p` and `q` have values smaller than `curr->val`, then both nodes must lie in the **left subtree**. We should move `curr` to `curr->left`.
- If both `p` and `q` have values larger than `curr->val`, then both nodes must lie in the **right subtree**. We should move `curr` to `curr->right`.
- If one value is smaller and the other is larger (or if `curr` is equal to either `p` or `q`), then the paths to `p` and `q` split at `curr`. This means `curr` is the **Lowest Common Ancestor**.

---

## 4. Pattern Recognition

- **Category**: Binary Search Tree (BST) Traversal.
- **Pattern**: Binary Search / Directional Splitting.
- **Why**: The BST property provides a clear sorting criterion at each step. This allows us to make a binary decision (go left, go right, or split) at each node, reducing search time to $\mathcal{O}(H)$ (where $H$ is the height of the tree) and space to $\mathcal{O}(1)$ if done iteratively.

---

## 5. Brute Force Solution

The brute force solution is to use the general Binary Tree LCA algorithm, which traverses the entire tree and does not use the BST property. We will skip implementing the general algorithm here and jump straight to the BST-specific implementations.

---

## 6. Better Solution (Recursive BST LCA)

We can implement the BST splitting logic recursively.

### Algorithm
1. If `root` is null, return `nullptr`.
2. If `p->val < root->val` and `q->val < root->val`, recurse left: `lowestCommonAncestor(root->left, p, q)`.
3. If `p->val > root->val` and `q->val > root->val`, recurse right: `lowestCommonAncestor(root->right, p, q)`.
4. Else, the paths split or one node is the ancestor of the other. Return `root`.

### Complexity Analysis
- **Time Complexity**: $\mathcal{O}(H)$ where $H$ is the height of the tree. In the worst case (skewed BST), $H = N$. In a balanced BST, $H = \log N$.
- **Space Complexity**: $\mathcal{O}(H)$ call stack space.

### C++17 Implementation
```cpp
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
};

class SolutionRecursive {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if (root == nullptr) {
            return nullptr;
        }
        
        // If both nodes are smaller, LCA must be in the left subtree
        if (p->val < root->val && q->val < root->val) {
            return lowestCommonAncestor(root->left, p, q);
        }
        
        // If both nodes are larger, LCA must be in the right subtree
        if (p->val > root->val && q->val > root->val) {
            return lowestCommonAncestor(root->right, p, q);
        }
        
        // We have found the split point, which is the LCA
        return root;
    }
};
```

---

## 7. Optimal Solution (Iterative BST LCA)

By using a simple while loop, we can eliminate the recursion stack entirely, achieving $\mathcal{O}(1)$ auxiliary space.

### Algorithm
1. Set a runner pointer `curr = root`.
2. While `curr` is not null:
   - If both `p->val` and `q->val` are less than `curr->val`, move `curr = curr->left`.
   - If both `p->val` and `q->val` are greater than `curr->val`, move `curr = curr->right`.
   - Else, we have found the split point. Return `curr`.
3. Return `nullptr` if the loop exits without finding the LCA (though constraints guarantee `p` and `q` exist).

---

### Why this is Optimal
It uses the BST property to run in $\mathcal{O}(H)$ time and avoids recursive call stack overhead, keeping auxiliary space at a constant $\mathcal{O}(1)$.

---

### Beautiful C++17 Code
```cpp
class SolutionOptimal {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        TreeNode* curr = root;
        
        while (curr != nullptr) {
            // If both targets are smaller than current node, go left
            if (p->val < curr->val && q->val < curr->val) {
                curr = curr->left;
            }
            // If both targets are larger than current node, go right
            else if (p->val > curr->val && q->val > curr->val) {
                curr = curr->right;
            }
            // We've found the split point (or curr is p or q)
            else {
                return curr;
            }
        }
        
        return nullptr;
    }
};
```

---

## 8. Code Walkthrough

Let's dissect the optimal iterative loop:
- `while (curr != nullptr)`: Traverses down the tree.
- `if (p->val < curr->val && q->val < curr->val)`: Both values are in the left subtree, so we discard the current node and the entire right subtree.
- `else if (p->val > curr->val && q->val > curr->val)`: Both values are in the right subtree, so we discard the current node and the entire left subtree.
- `else { return curr; }`: Handles the split condition where one value is on the left and the other is on the right, or where `curr` matches either `p` or `q`. This node is our LCA.

---

## 9. Dry Run

Let's dry run the iterative solution with this BST:
```
        6
       / \
      2   8
     / \
    0   4
       / \
      3   5
```
Targets: `p = 2`, `q = 4`.

- `curr = 6` (root):
  - `p->val (2) < 6` and `q->val (4) < 6`.
  - Both are smaller, so we move left: `curr = curr->left = 2`.
- `curr = 2`:
  - `p->val (2)` is equal to `curr->val (2)`.
  - The conditions `p->val < 2` and `p->val > 2` are both false.
  - We hit the `else` block and return `curr` (`2`).
- Final LCA returned: `2`. Correct.

---

## 10. Complexity Analysis

| Metric | Recursive DFS | Iterative (Optimal) |
|:---|:---|:---|
| **Time Complexity (Best)** | $\mathcal{O}(\log N)$ (balanced tree) | $\mathcal{O}(\log N)$ |
| **Time Complexity (Worst)**| $\mathcal{O}(N)$ (skewed tree) | $\mathcal{O}(N)$ |
| **Space Complexity (Worst)**| $\mathcal{O}(N)$ | $\mathcal{O}(1)$ auxiliary |
| **Space Complexity (Best)** | $\mathcal{O}(\log N)$ | $\mathcal{O}(1)$ auxiliary |

---

## 11. Common Mistakes

1. **Not Using the BST Property**: Implementing the general binary tree LCA algorithm. While correct, it is inefficient because it runs in $\mathcal{O}(N)$ time even for balanced trees, whereas the BST-specific approach runs in $\mathcal{O}(H)$ time.
2. **Missing the Split Condition**: Forgetting that if `curr` matches either `p` or `q`, it is the LCA. The `else` block handles this case correctly because the inequality checks fail.

---

## 12. Edge Cases

| Edge Case | Input Tree | Why it matters | How Code Handles It |
|:---|:---|:---|:---|
| **One Target is Root** | `p` is root | Split check validation | Returns `root` immediately since inequality checks fail. |
| **Skewed Tree** | Line tree `5 -> 4 -> 3` | Stack overflow concern | Iterative approach handles this with $\mathcal{O}(1)$ space. |
| **Immediate Parent** | `p` is child of `q` | Parent-child relationship | Returns `q` (which is parent of `p`) correctly. |

---

## 13. Interview Explanation

"To find the Lowest Common Ancestor (LCA) in a Binary Search Tree, we can leverage the sorted property of the BST. 

At any node, if both target nodes have values smaller than the current node's value, we know the LCA must reside in the left subtree. If both have values greater than the current node's value, the LCA must reside in the right subtree. 

The first node where the paths to the target nodes split—meaning one target is smaller and the other is larger than the current node—is the LCA. 

We can implement this iteratively using a simple loop to traverse down the tree. This achieves an optimal time complexity of $\mathcal{O}(H)$, where $H$ is the height of the tree, and an auxiliary space complexity of $\mathcal{O}(1)$."

---

## 14. Follow-up Questions

1. **Q**: What if the nodes `p` and `q` are not guaranteed to exist in the BST?
   - **A**: We would first need to verify if both nodes exist in the tree. If they do, we run the LCA algorithm; if not, we return `nullptr`.
2. **Q**: How would you find the LCA if the tree was a general binary tree without BST properties?
   - **A**: We would have to traverse the entire tree recursively, returning the nodes when found, and checking if they meet at a common ancestor.

---

## 15. Variations

1. **Lowest Common Ancestor in a Binary Tree**: Solving the same problem for general binary trees (no sorting property).
2. **Inorder Successor in BST**: Find the next node in BST inorder traversal.

---

## 16. Revision Notes

- **BST Traversal Criterion**:
  - `p->val < curr->val && q->val < curr->val` $\to$ Go Left.
  - `p->val > curr->val && q->val > curr->val` $\to$ Go Right.
  - Else $\to$ Split point (LCA found).
- **Space complexity**: Iterative version uses $\mathcal{O}(1)$ auxiliary space.

---

## 17. Memorization Version

```cpp
// Iterative BST LCA: Go left if both small, right if both large, else split.
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    TreeNode* curr = root;
    while (curr) {
        if (p->val < curr->val && q->val < curr->val) curr = curr->left;
        else if (p->val > curr->val && q->val > curr->val) curr = curr->right;
        else return curr;
    }
    return nullptr;
}
```
