# Check if Tree is Symmetric

## 1. Problem Statement

Given the root of a binary tree, check whether it is a mirror of itself (i.e., symmetric around its center).

### Input
- The root of a binary tree.

### Output
- A boolean: `true` if the tree is symmetric, `false` otherwise.

### Constraints
- The number of nodes in the tree is in the range $[1, 1000]$.
- $-100 \le \text{Node.val} \le 100$.

---

## 2. Interview Clarification

1. **What is an empty tree considered?**
   - *Answer:* Symmetric. An empty tree is a mirror of itself.
2. **Can we solve this both recursively and iteratively?**
   - *Answer:* Yes. Interviewers often ask for the recursive solution first, and then follow up by asking for the iterative version to avoid recursion stack overflow.
3. **What defines two trees as mirrors of each other?**
   - *Answer:* Two trees $A$ and $B$ are mirrors if:
     - Their root values are equal.
     - The left subtree of $A$ is a mirror of the right subtree of $B$.
     - The right subtree of $A$ is a mirror of the left subtree of $B$.

---

## 3. Thinking Process

- **First instinct:** A symmetric tree looks identical when folded in half.
- If we look at the root's left subtree and right subtree, they must be mirrors of each other.
- Let's define a helper function `isMirror(t1, t2)`:
  - If both `t1` and `t2` are `nullptr`, they are symmetric (return `true`).
  - If only one of them is `nullptr`, they are not symmetric (return `false`).
  - If their values are different (`t1->val != t2->val`), they are not symmetric (return `false`).
  - Otherwise, we must check if:
    - `t1->left` is a mirror of `t2->right` AND
    - `t1->right` is a mirror of `t2->left`.
- **Iterative Alternative:**
  - We can achieve this using a queue. Instead of a standard level-order BFS, we push pairs of nodes that need to be symmetric.
  - Push `(root->left, root->right)` to start.
  - In each iteration:
    - Pop two nodes `n1` and `n2`.
    - If both are null, continue.
    - If one is null or values differ, return `false`.
    - Push `n1->left` and `n2->right` to the queue.
    - Push `n1->right` and `n2->left` to the queue.

---

## 4. Pattern Recognition

- **Double Tree Recursion:** Traversing two tree structures simultaneously (often comparing structural equality, subtree relationship, or mirror properties).
- **BFS with Pair Processing:** Managing symmetric nodes in pairs in a queue.

---

## 5. Brute Force Solution

There is no "naive" brute force other than converting the tree to structural lists and verifying mirroring, which is messy. The recursive solution is the natural and most direct approach. Thus, we jump directly to the optimal implementations.

---

## 6. Better Solution (Iterative Approach)

Using a queue to traverse the tree iteratively.

### C++17 Code
```cpp
#include <queue>

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class SolutionIterative {
public:
    bool isSymmetric(TreeNode* root) {
        if (root == nullptr) return true;
        
        std::queue<TreeNode*> q;
        q.push(root->left);
        q.push(root->right);
        
        while (!q.empty()) {
            TreeNode* t1 = q.front(); q.pop();
            TreeNode* t2 = q.front(); q.pop();
            
            if (t1 == nullptr && t2 == nullptr) continue;
            if (t1 == nullptr || t2 == nullptr) return false;
            if (t1->val != t2->val) return false;
            
            // Push children in mirror order
            q.push(t1->left);
            q.push(t2->right);
            q.push(t1->right);
            q.push(t2->left);
        }
        
        return true;
    }
};
```

### Complexity Analysis
- **Time Complexity:** $O(N)$ because we visit each node at most once.
- **Space Complexity:** $O(N)$ since the queue can hold up to $O(N)$ nodes at the lowest level in the worst case.

---

## 7. Optimal Solution (Recursive Approach)

### C++17 Code

```cpp
#include <iostream>

class Solution {
private:
    // Helper to check if t1 and t2 are mirror images
    bool isMirror(TreeNode* t1, TreeNode* t2) {
        // Base Case 1: Both are null
        if (t1 == nullptr && t2 == nullptr) {
            return true;
        }
        // Base Case 2: One is null, the other is not
        if (t1 == nullptr || t2 == nullptr) {
            return false;
        }
        // Base Case 3: Values must be equal
        if (t1->val != t2->val) {
            return false;
        }

        // Check if outer subtrees (t1->left and t2->right) are mirrors
        // AND inner subtrees (t1->right and t2->left) are mirrors
        return isMirror(t1->left, t2->right) && isMirror(t1->right, t2->left);
    }

public:
    bool isSymmetric(TreeNode* root) {
        if (root == nullptr) {
            return true;
        }
        return isMirror(root->left, root->right);
    }
};
```

---

## 8. Code Walkthrough

- **`isSymmetric(root)`**:
  - Checks if the root is null. If yes, returns `true`.
  - Otherwise, delegates structural comparison to `isMirror(root->left, root->right)`.
- **`isMirror(t1, t2)`**:
  - Checks if both nodes are null. If so, they are symmetric mirrors; returns `true`.
  - Checks if only one is null. If so, they cannot be mirrors; returns `false`.
  - Checks if `t1->val != t2->val`. If so, returns `false`.
  - Recurses symmetrically:
    - Compares `t1->left` (left subtree outer) with `t2->right` (right subtree outer).
    - Compares `t1->right` (left subtree inner) with `t2->left` (right subtree inner).
  - Returns `true` if and only if both comparisons return `true`.

---

## 9. Dry Run

Let's dry run the recursive solution with this symmetric tree:
```
        1
       / \
      2   2
     / \ / \
    3  4 4  3
```

### Initial Call
`isSymmetric(root)` $\Rightarrow$ Calls `isMirror(node(2_left), node(2_right))`

### Step 1: `isMirror(2_left, 2_right)`
- Neither is null.
- Values match ($2 == 2$).
- Make two recursive calls:
  1. `isMirror(2_left->left, 2_right->right)` $\Rightarrow$ `isMirror(3_left, 3_right)`
  2. `isMirror(2_left->right, 2_right->left)` $\Rightarrow$ `isMirror(4_left, 4_right)`

### Step 2: `isMirror(3_left, 3_right)`
- Neither is null. Values match ($3 == 3$).
- Recursive calls:
  - `isMirror(nullptr, nullptr)` $\Rightarrow$ returns `true`.
  - `isMirror(nullptr, nullptr)` $\Rightarrow$ returns `true`.
- Returns `true`.

### Step 3: `isMirror(4_left, 4_right)`
- Neither is null. Values match ($4 == 4$).
- Recursive calls return `true` (both sub-children are null).
- Returns `true`.

### Final Evaluation
- Both recursive sub-calls in Step 1 returned `true`.
- Result is `true`.

---

## 10. Complexity Analysis

| Metric | Complexity | Reasoning |
| :--- | :--- | :--- |
| **Time Complexity** | $O(N)$ | We visit every node in the tree exactly once. |
| **Space Complexity** | $O(H)$ | The height of the recursion stack is at most $O(H)$, where $H$ is the height of the tree. In the worst case (skewed tree), $H = O(N)$; in the best/balanced case, $H = O(\log N)$. |

---

## 11. Common Mistakes

1. **Incorrect mirror mapping:** Comparing `t1->left` with `t2->left` (which checks if the subtrees are identical, not mirrors).
2. **Missing null pointer checks:** Accessing `t1->val` before verifying that `t1` is not `nullptr`.

---

## 12. Edge Cases

| Edge Case | Input | Handling |
| :--- | :--- | :--- |
| **Empty Tree** | `nullptr` | Returns `true`. |
| **Single Node** | `[1]` | Returns `true` (calls `isMirror(nullptr, nullptr)`). |
| **Asymmetric Structure**| `[1, 2, 2, null, 3, null, 3]` | Correctly fails when comparing `nullptr` with `3`. |

---

## 13. Interview Explanation

"To determine if a binary tree is symmetric, we check if its left subtree is a mirror image of its right subtree. 

We can define a helper function `isMirror` that takes two nodes, `t1` and `t2`. Two nodes are mirror images of each other if:
1. Both are null, which is symmetric.
2. Neither is null, their values are equal, and their subtrees are cross-mirrored. Specifically, `t1`'s left child must mirror `t2`'s right child, and `t1`'s right child must mirror `t2`'s left child.

We can solve this recursively using DFS or iteratively using a queue to process pairs of nodes in mirror order. Both approaches run in $O(N)$ time, while the recursive approach uses $O(H)$ space and the iterative approach uses $O(W)$ space (where $W$ is the maximum width of the tree)."

---

## 14. Follow-up Questions

1. **What is the difference in space complexity between the recursive and iterative solutions?**
   - *Answer:* The recursive solution uses space proportional to the tree height $H$ due to the call stack. The iterative solution uses space proportional to the maximum width of the tree $W$ in its queue. For a skewed tree, recursive space is $O(N)$ and iterative is $O(1)$. For a perfect binary tree, recursive space is $O(\log N)$ and iterative is $O(N)$.

---

## 15. Variations

1. **Same Tree (LeetCode 100):** Check if two trees are identical (not mirrors). Compare `t1->left` with `t2->left`, and `t1->right` with `t2->right`.
2. **Flip Equivalent Binary Trees (LeetCode 951):** Check if trees can match by flipping left and right children of any nodes.

---

## 16. Revision Notes

- **Symmetric Rule:** Compare `left` with `right` and `right` with `left`.
- **Base cases:**
  ```cpp
  if (!t1 && !t2) return true;
  if (!t1 || !t2) return false;
  if (t1->val != t2->val) return false;
  ```

---

## 17. Memorization Version

```text
Symmetric Tree:
1. isSymmetric(root) -> return !root || isMirror(root->left, root->right)
2. isMirror(t1, t2):
   - If both null, return true.
   - If one null or values differ, return false.
   - Return isMirror(t1->left, t2->right) && isMirror(t1->right, t2->left).
Complexity: Time O(N), Space O(H).
```
