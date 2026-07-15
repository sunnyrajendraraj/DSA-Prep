# Check if Binary Tree is Balanced

## 1. Problem Statement

Given a binary tree, determine if it is **height-balanced**.

A **height-balanced** binary tree is defined as:
- A binary tree in which the left and right subtrees of every node differ in height by no more than 1.

### Input
- `root`: A pointer to the root node of a binary tree.

### Output
- A boolean (`true` or `false`) indicating if the tree is height-balanced.

### Constraints
- The number of nodes in the tree is in the range `[0, 5000]`.
- Node values: `-10^4 <= Node.val <= 10^4`.

### Observations
- A tree is balanced if and only if:
  1. The left subtree is balanced.
  2. The right subtree is balanced.
  3. The difference between the heights of the left and right subtrees is at most 1.
- This is a nested condition: we need both the status (balanced/unbalanced) and the height of subtrees.

---

## 2. Interview Clarification

Before starting, clarify:
1. **Empty Tree**: "Is an empty tree considered height-balanced?"
   - *Interviewer response:* Yes, an empty tree (null root) has a height of 0, so it is balanced (returns `true`).
2. **Standard Height definition**: "Are we using the standard node-based height where a single node has a height of 1?"
   - *Interviewer response:* Yes.
3. **Optimizations**: "Should I optimize to prevent redundant height checks?"
   - *Interviewer response:* Yes, first briefly explain why the brute force is slow, then implement the optimal single-pass version.

---

## 3. Thinking Process

### First Instinct (Brute Force Approach)
- For every node, we check if the difference in height between its left and right subtrees is at most 1.
- To do this, we can write a recursive `getHeight(node)` function.
- Then, we write a recursive `isBalanced(node)` function:
  - If node is null, return `true`.
  - Calculate `leftHeight = getHeight(node->left)`.
  - Calculate `rightHeight = getHeight(node->right)`.
  - If `abs(leftHeight - rightHeight) > 1`, return `false`.
  - Otherwise, return `isBalanced(node->left) && isBalanced(node->right)`.
- This is slow because we call `getHeight` repeatedly on the same nodes down the tree.

### Transition to Optimal Solution
- Can we check balance while calculating heights?
- When evaluating the height of a subtree:
  - If any child subtree is already unbalanced, the parent subtree is also unbalanced.
  - If the height difference between the left and right child is greater than 1, the parent subtree is unbalanced.
- To propagate this "unbalanced" state upward, we can use a sentinel value: `-1`.
  - If a subtree is balanced, we return its actual height (a non-negative integer).
  - If a subtree is unbalanced, we return `-1`.
- At each node:
  1. Get left height. If it is `-1`, return `-1`.
  2. Get right height. If it is `-1`, return `-1`.
  3. If `abs(leftHeight - rightHeight) > 1`, return `-1`.
  4. Else, return the actual height: `1 + max(leftHeight, rightHeight)`.

---

## 4. Pattern Recognition

- **Category**: Binary Tree Bottom-Up DFS.
- **Pattern**: Sentinel Return / Failure Propagation.
- **Why**: Like checking for binary tree validity or path constraints, checking for balance requires bubble-up information. Using a special sentinel value (like `-1` or `std::nullopt` or a custom `pair<bool, int>`) avoids multiple passes.

---

## 5. Brute Force Solution

The brute force approach recursively calculates heights from scratch at every single node.

### Algorithm
1. Helper `getHeight(node)` returns height of subtree.
2. Main `isBalanced(node)`:
   - If null, return `true`.
   - If `abs(getHeight(node->left) - getHeight(node->right)) > 1`, return `false`.
   - Return `isBalanced(node->left) && isBalanced(node->right)`.

### Complexity Analysis
- **Time Complexity**: $\mathcal{O}(N^2)$ in the worst case (skewed tree) because we run an $\mathcal{O}(N)$ height calculation at each of the $N$ nodes. For a balanced tree, it is $\mathcal{O}(N \log N)$ (Master Theorem $T(N) = 2T(N/2) + \mathcal{O}(N)$).
- **Space Complexity**: $\mathcal{O}(H)$ stack frames. Worst case is $\mathcal{O}(N)$.

### C++17 Implementation
```cpp
#include <algorithm>
#include <cmath>

struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
};

class SolutionBrute {
private:
    int getHeight(TreeNode* node) {
        if (node == nullptr) return 0;
        return 1 + std::max(getHeight(node->left), getHeight(node->right));
    }
public:
    bool isBalanced(TreeNode* root) {
        if (root == nullptr) return true;
        
        int leftHeight = getHeight(root->left);
        int rightHeight = getHeight(root->right);
        
        if (std::abs(leftHeight - rightHeight) > 1) {
            return false;
        }
        
        return isBalanced(root->left) && isBalanced(root->right);
    }
};
```

---

## 6. Better Solution

*Note: Since the brute force jumps directly to the optimal version by integrating height and balance verification using the `-1` sentinel, we will detail this as the Optimal Solution below.*

---

## 7. Optimal Solution (Single-Pass DFS with Sentinel)

This approach computes height and validates balance simultaneously, returning `-1` if unbalanced.

### Algorithm
1. Define a helper function `checkHeight(TreeNode* node)`:
   - Base case: If `node` is null, return `0`.
   - Recurse left: `leftHeight = checkHeight(node->left)`. If `leftHeight == -1`, return `-1` (propagate failure).
   - Recurse right: `rightHeight = checkHeight(node->right)`. If `rightHeight == -1`, return `-1` (propagate failure).
   - Check balance: If `abs(leftHeight - rightHeight) > 1`, return `-1` (mark unbalanced).
   - Return height: `1 + max(leftHeight, rightHeight)`.
2. In the main function `isBalanced(root)`:
   - Return `checkHeight(root) != -1`.

---

### Why this is Optimal
It traverses each node at most once. It performs a bottom-up calculation: once a subtree is found to be unbalanced, the failure is bubbled up immediately without any further height evaluations. Time complexity is $\mathcal{O}(N)$.

---

### Beautiful C++17 Code
```cpp
#include <algorithm>
#include <cmath>

class SolutionOptimal {
private:
    // Returns height of node if balanced, or -1 if unbalanced
    int checkHeight(TreeNode* node) {
        if (node == nullptr) {
            return 0; // Empty tree is balanced with height 0
        }
        
        // Check left subtree
        int leftHeight = checkHeight(node->left);
        if (leftHeight == -1) {
            return -1; // Left subtree is already unbalanced
        }
        
        // Check right subtree
        int rightHeight = checkHeight(node->right);
        if (rightHeight == -1) {
            return -1; // Right subtree is already unbalanced
        }
        
        // Evaluate balance of current node
        if (std::abs(leftHeight - rightHeight) > 1) {
            return -1; // Current node is unbalanced
        }
        
        // If balanced, return the actual height
        return 1 + std::max(leftHeight, rightHeight);
    }
public:
    bool isBalanced(TreeNode* root) {
        return checkHeight(root) != -1;
    }
};
```

---

## 8. Code Walkthrough

- `int leftHeight = checkHeight(node->left);`: Recursively checks the left subtree.
- `if (leftHeight == -1) return -1;`: This is the early return/pruning step. If the left side was unbalanced, we stop processing the right side and immediately return `-1`.
- `if (std::abs(leftHeight - rightHeight) > 1) return -1;`: This verifies the balance condition for the current node.
- `return 1 + std::max(leftHeight, rightHeight);`: Standard height calculation, executed only if this subtree is balanced.

---

## 9. Dry Run

Let's dry run the optimal solution with this unbalanced tree:
```
       1
      /
     2
    /
   3
```

- Call `checkHeight(1)`:
  - Call `checkHeight(2)`:
    - Call `checkHeight(3)`:
      - Left = 0, Right = 0.
      - Difference = 0. Returns `1`.
    - Right side of 2 is `nullptr` $\to$ Returns `0`.
    - Back to `2`:
      - `leftHeight = 1`, `rightHeight = 0`.
      - Difference = 1. Returns `1 + max(1, 0) = 2`.
  - Right side of 1 is `nullptr` $\to$ Returns `0`.
  - Back to `1`:
    - `leftHeight = 2`, `rightHeight = 0`.
    - Difference = 2. Since `abs(2 - 0) > 1`, return `-1`.
- In main: `checkHeight(1) != -1` evaluates to `false`. Correct.

---

## 10. Complexity Analysis

| Metric | Brute Force | Optimal DFS (Sentinel) |
|:---|:---|:---|
| **Time Complexity (Best)** | $\mathcal{O}(N \log N)$ (balanced tree) | $\mathcal{O}(N)$ |
| **Time Complexity (Worst)**| $\mathcal{O}(N^2)$ (skewed tree) | $\mathcal{O}(N)$ |
| **Space Complexity**| $\mathcal{O}(H)$ stack frames | $\mathcal{O}(H)$ stack frames |
| **Space Complexity (Worst)**| $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |

---

## 11. Common Mistakes

1. **Forgetting to Propagate `-1`**: Writing `checkHeight` but forgetting to verify if the children returned `-1`. If you just do `std::abs(leftHeight - rightHeight) > 1`, you might calculate `abs(-1 - 2) = 3` which returns `-1`, but intermediate nodes might hide the `-1` state by adding 1 to it.
2. **Wrong Sentinel Value**: Using a sentinel value that could represent a valid height (like `0` or positive integers). `-1` is ideal because a tree height can never be negative.
3. **Slow Brute Force**: Implementing the $\mathcal{O}(N^2)$ approach due to familiarity with the height function, without realizing the optimization potential.

---

## 12. Edge Cases

| Edge Case | Input Tree | Why it matters | How Code Handles It |
|:---|:---|:---|:---|
| **Empty Tree** | `nullptr` | Base case validation | Returns `0` (balanced). |
| **Single Node** | `[1]` | Minimal structure | Returns `1` (balanced). |
| **Slightly Unbalanced**| Left height = 2, Right = 0 | Edge of criteria | Returns `-1` since diff > 1. |
| **Deeply Unbalanced** | Left height = 5, Right = 1 | High depth difference | Quickly returns `-1` upwards. |

---

## 13. Interview Explanation

"To determine if a binary tree is height-balanced, the height of the left and right subtrees at every single node must not differ by more than 1.

The naive approach computes the height of each node's children independently. This leads to redundant calculations and an $\mathcal{O}(N^2)$ worst-case time complexity.

To optimize this, we can combine height calculation and balance checking into a single bottom-up DFS traversal. We define a recursive helper function that returns the height of the subtree if it is balanced, or `-1` if it is unbalanced. At each node, we recursively find the left and right heights. If either subtree returns `-1`, or if the height difference between the left and right subtrees exceeds 1, we immediately propagate `-1` up the tree. Otherwise, we return the node's actual height. The main function simply checks if the helper returns a value other than `-1`, running in optimal $\mathcal{O}(N)$ time and $\mathcal{O}(H)$ space."

---

## 14. Follow-up Questions

1. **Q**: Could we use a custom structure or `std::pair<bool, int>` instead of the sentinel value `-1`?
   - **A**: Yes, we could return a pair where the first element is a boolean indicating balance status and the second is the height. The sentinel `-1` is preferred in C++ because it avoids the overhead of creating and copying pair structures on the stack.
2. **Q**: How does height-balance relate to AVL trees?
   - **A**: AVL trees are self-balancing binary search trees that maintain this exact height-balanced property. If an insertion makes the heights differ by more than 1, AVL trees perform rotations to restore balance.

---

## 15. Variations

1. **Maximum Depth of Binary Tree**: Calculated using the same bottom-up height formula.
2. **Minimum Depth of Binary Tree**: Calculates the shortest path to a leaf node.

---

## 16. Revision Notes

- **Balanced Definition**: `abs(leftHeight - rightHeight) <= 1` at every node.
- **Optimal DFS pattern**: Bottom-up traversal with sentinel propagation (`-1`).
- **Pruning**: If child height returns `-1`, return `-1` immediately.

---

## 17. Memorization Version

```cpp
// Returns height if balanced, else -1
int check(TreeNode* node) {
    if (!node) return 0;
    int lh = check(node->left);  if (lh == -1) return -1;
    int rh = check(node->right); if (rh == -1) return -1;
    if (abs(lh - rh) > 1) return -1;
    return 1 + max(lh, rh);
}
bool isBalanced(TreeNode* root) {
    return check(root) != -1;
}
```
