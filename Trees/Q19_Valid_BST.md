# Check if Valid BST

## 1. Problem Statement

Given the root of a binary tree, determine if it is a valid Binary Search Tree (BST).

A valid BST is defined as follows:
1.  The left subtree of a node contains only nodes with keys **less than** the node's key.
2.  The right subtree of a node contains only nodes with keys **greater than** the node's key.
3.  Both the left and right subtrees must also be binary search trees.

*   **Input**:
    *   A pointer to the root of the binary tree: `TreeNode* root`.
*   **Output**:
    *   A boolean `bool`: `true` if the tree is a valid BST, otherwise `false`.
*   **Constraints**:
    *   $1 \le N \le 10^4$ (Number of nodes in the tree)
    *   $-2^{31} \le \text{Node value} \le 2^{31} - 1$ (Full 32-bit signed integer range)

---

## 2. Interview Clarification

1.  **Q**: Can a node have a value equal to its parent (i.e. are duplicate values allowed)?
    *   *A*: In this standard version of the problem, strictly less than (`<`) and strictly greater than (`>`) are required. Thus, duplicate values are **not** allowed.
2.  **Q**: What is the range of node values?
    *   *A*: Values can span the full range of 32-bit signed integers (`INT_MIN` to `INT_MAX`).
3.  **Q**: Can the tree be empty?
    *   *A*: Yes. An empty tree is a valid BST.

---

## 3. Thinking Process

*   **First Instinct (Common Trap)**:
    *   Just check if the left child value is less than the current node and the right child value is greater than the current node.
    *   *Why this fails*: Consider this tree:
        ```
              5
             / \
            3   7
               / \
              4   8
        ```
        Checking immediate children for node `7`: `4 < 7` and `8 > 7` is true.
        Checking immediate children for root `5`: `3 < 5` and `7 > 5` is true.
        However, this is **not** a valid BST because `4` is in the right subtree of `5` but is less than `5`.
        Therefore, we must ensure that *all* nodes in the left subtree are less than the ancestor, and *all* nodes in the right subtree are greater than the ancestor.

*   **Optimal Insight**:
    *   **Approach 1: Pass min/max boundaries**.
        *   We can traverse down the tree, maintaining a valid range `(min_val, max_val)` for each node.
        *   For the root, the range is `(long long MIN, long long MAX)` to prevent integer overflow.
        *   When going left, the maximum allowable value becomes the current node's value: range is updated to `(min_val, curr->val)`.
        *   When going right, the minimum allowable value becomes the current node's value: range is updated to `(curr->val, max_val)`.
    *   **Approach 2: Inorder Traversal**.
        *   The inorder traversal of a valid BST must be strictly increasing.
        *   We can perform an inorder traversal and verify that each visited node has a value strictly greater than the previously visited node.

---

## 4. Pattern Recognition

*   **Topic**: BST Properties & Validation.
*   **Pattern**: Recursive range bounding (DFS) or Sorted property validation (Inorder).
*   **Why**: A node's validity depends on constraints inherited from all its ancestors, not just its parent. Passing bounds down (DFS) or checking output sequence order (Inorder) are the direct ways to enforce these ancestor constraints.

---

## 5. Brute Force Solution

*   **Algorithm**:
    *   For every node, find the maximum value in its left subtree and the minimum value in its right subtree.
    *   Check if `left_max < node->val` and `right_min > node->val`.
    *   Recursively repeat this for all nodes.
*   **Complexity**:
    *   **Time Complexity**: $O(N^2)$ for a skewed tree, as finding min/max in subtrees takes $O(N)$ and we do it for all $N$ nodes.
    *   **Space Complexity**: $O(H)$ recursion stack space.
*   **Pros/Cons**:
    *   *Pros*: Straightforward translation of the mathematical definition of BST.
    *   *Cons*: Redundant traversals lead to unacceptable quadratic time complexity.

### Brute Force C++17 Code
```cpp
#include <iostream>
#include <algorithm>
#include <climits>

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

int getMaxValue(TreeNode* root) {
    if (!root) return INT_MIN;
    return std::max({root->val, getMaxValue(root->left), getMaxValue(root->right)});
}

int getMinValue(TreeNode* root) {
    if (!root) return INT_MAX;
    return std::min({root->val, getMinValue(root->left), getMinValue(root->right)});
}

bool isValidBSTBrute(TreeNode* root) {
    if (!root) return true;
    if (root->left && getMaxValue(root->left) >= root->val) return false;
    if (root->right && getMinValue(root->right) <= root->val) return false;
    return isValidBSTBrute(root->left) && isValidBSTBrute(root->right);
}
```

---

## 6. Better Solution

*   **Inorder Traversal approach**:
    *   We track the previously visited node pointer `prev`.
    *   Do an inorder traversal (Left, Root, Right).
    *   At the root visit, if `prev` is not null and `curr->val <= prev->val`, return `false`.
    *   Otherwise, update `prev = curr` and continue.
*   **Complexity**:
    *   **Time Complexity**: $O(N)$
    *   **Space Complexity**: $O(H)$ for stack space.
*   **Pros/Cons**:
    *   Highly elegant, but requires maintaining a stateful pointer/reference to `prev` across recursive calls.

### Inorder C++17 Code
```cpp
class SolutionInorder {
private:
    TreeNode* prev = nullptr;

public:
    bool isValidBST(TreeNode* root) {
        if (root == nullptr) return true;
        
        // Recurse left
        if (!isValidBST(root->left)) return false;
        
        // Validate current node
        if (prev != nullptr && root->val <= prev->val) {
            return false;
        }
        prev = root; // Update previous visited node
        
        // Recurse right
        return isValidBST(root->right);
    }
};
```

---

## 7. Optimal Solution

The **Recursive Range Bounding** approach is generally preferred because it is stateless (does not rely on global/member variables) and easily handles short-circuiting.

### Algorithm
1.  Define a helper function `validate(TreeNode* node, long long minVal, long long maxVal)`.
2.  If `node` is `nullptr`, return `true` (base case).
3.  Check if current node's value violates boundaries:
    *   If `node->val <= minVal` or `node->val >= maxVal`, return `false`.
4.  Recursively validate left and right subtrees:
    *   Left subtree: `validate(node->left, minVal, node->val)` (limits values to be strictly less than `node->val`).
    *   Right subtree: `validate(node->right, node->val, maxVal)` (limits values to be strictly greater than `node->val`).
5.  Return `true` only if both subtrees are valid.

### Optimal C++17 Code
```cpp
#include <iostream>
#include <climits>

/**
 * Definition for a binary tree node.
 */
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
private:
    /**
     * Helper function validating that nodes in the subtree rooted at 'node'
     * fall strictly within the range (minVal, maxVal).
     */
    bool validate(TreeNode* node, long long minVal, long long maxVal) {
        // Base case: An empty tree/leaf child is always valid
        if (node == nullptr) {
            return true;
        }

        // Check range constraints
        if (node->val <= minVal || node->val >= maxVal) {
            return false;
        }

        // Left subtree must be within (minVal, node->val)
        // Right subtree must be within (node->val, maxVal)
        return validate(node->left, minVal, node->val) && 
               validate(node->right, node->val, maxVal);
    }

public:
    /**
     * Validates if a binary tree is a valid Binary Search Tree.
     */
    bool isValidBST(TreeNode* root) {
        // Use LLONG_MIN and LLONG_MAX to prevent integer overflow
        // when node value is INT_MIN or INT_MAX.
        return validate(root, LLONG_MIN, LLONG_MAX);
    }
};
```

---

## 8. Code Walkthrough

1.  **Use of `long long`**:
    *   `LLONG_MIN` and `LLONG_MAX` are used for initial bounds. If we used `INT_MIN` and `INT_MAX`, and the root node itself had a value of `INT_MAX`, the check `root->val >= maxVal` would incorrectly fail or overflow.
2.  **Strict Inequalities**:
    *   `node->val <= minVal || node->val >= maxVal` enforces that duplicate values are invalid.
3.  **Boundary Update**:
    *   For `node->left`: upper bound is set to `node->val`.
    *   For `node->right`: lower bound is set to `node->val`.
4.  **Short-Circuit Evaluation**:
    *   Using the `&&` operator means if `validate(left)` returns `false`, `validate(right)` is never called, saving execution time.

---

## 9. Dry Run

Consider the invalid tree:
```
      5
     / \
    3   7
       / \
      4   8
```

*   **Call 1**: `validate(5, LLONG_MIN, LLONG_MAX)`
    *   `5` is within range.
    *   Left branch: `validate(3, LLONG_MIN, 5)`.
    *   Right branch: `validate(7, 5, LLONG_MAX)`.

*   **Call 2 (Left child of 5)**: `validate(3, LLONG_MIN, 5)`
    *   `3` is within range.
    *   Children of `3` are null $\rightarrow$ Returns `true`.

*   **Call 3 (Right child of 5)**: `validate(7, 5, LLONG_MAX)`
    *   `7` is within range.
    *   Left branch: `validate(4, 5, 7)`.
    *   Right branch: `validate(8, 7, LLONG_MAX)`.

*   **Call 4 (Left child of 7)**: `validate(4, 5, 7)`
    *   `4` is **not** within range (since `4 <= 5` is true).
    *   Returns `false`.

*   **Result Propagation**: Call 3 returns `false`, which bubbles up to Call 1. Final result: `false`.

---

## 10. Complexity Analysis

*   **Time Complexity**: $\mathcal{O}(N)$
    *   Each node in the tree is visited at most once.
*   **Space Complexity**: $\mathcal{O}(H)$
    *   $H$ is the height of the tree.
    *   The space is occupied by the recursion stack.
    *   For a balanced tree, $H = \log N$. For a skewed tree, $H = N$.

---

## 11. Common Mistakes

1.  **Only checking immediate children**: Assumed that if `root->left->val < root->val` and `root->right->val > root->val` for all subtrees, the tree is valid. (See Thinking Process for counter-example).
2.  **Integer Overflow**: Using `INT_MIN` and `INT_MAX` directly as limits. If the tree contains `INT_MIN` as a value, the strict inequality check will fail.
3.  **Allowing Duplicates**: Using `<` and `>` instead of `<=` and `>=` for boundary checks, which incorrectly allows duplicates.

---

## 12. Edge Cases

| Edge Case | Input Tree | Expected Output | Importance |
| :--- | :--- | :--- | :--- |
| **Empty Tree** | `nullptr` | `true` | Base case validation. |
| **Single Node** | `[2147483647]` (INT_MAX) | `true` | Tests integer bounds. |
| **Duplicate Values** | `[2, 2, 2]` | `false` | Verifies duplicates are marked invalid. |
| **Out-of-range descendant** | `[10, 5, 15, null, null, 6, 20]` | `false` | `6` is in the right subtree of `5` but is a descendant of `15` (violates root boundary `10`). |

---

## 13. Interview Explanation

"To validate a Binary Search Tree, checking only immediate parent-child relationships is insufficient because a node must respect constraints set by all of its ancestors. For example, every node in the root's left subtree must be strictly less than the root.

To implement this correctly and efficiently, we perform a recursive DFS traversal while passing down allowable ranges. We initialize the range for the root node as negative infinity to positive infinity using 64-bit integers to prevent integer overflow. As we traverse to the left child, we update the upper limit of the range to the parent's value. As we traverse to the right child, we update the lower limit to the parent's value. If any node's value falls outside its range, we immediately return `false`. This approach visits each node once, resulting in $O(N)$ time complexity and $O(H)$ stack space complexity."

---

## 14. Follow-up Questions

1.  **Q**: Can we solve this without recursion or extra space?
    *   *A*: Yes, we can perform a Morris Inorder Traversal. Morris traversal utilizes thread links (temporary parent links) to achieve $O(N)$ time complexity and $O(1)$ auxiliary space complexity.
2.  **Q**: What changes if we are told that duplicate values are allowed in the left subtree?
    *   *A*: The range updates would change: going left would update the range to `(minVal, node->val]` instead of `(minVal, node->val)`.

---

## 15. Variations

1.  **Find Inorder Successor in BST**:
    *   Requires traversing based on target value comparison.
2.  **Recover BST**: Two nodes of a BST are swapped by mistake. Find and recover them.
    *   *Solution*: Perform inorder traversal, identify the two nodes violating sorted order, and swap their values.

---

## 16. Revision Notes

*   **The Trap**: Local check `left < root < right` is NOT enough.
*   **The Rule**: Pass ranges `(min, max)` down. Left: update `max = root->val`, Right: update `min = root->val`.
*   **Type Hint**: Use `long long` limits to avoid `INT_MIN`/`INT_MAX` boundary bugs.

---

## 17. Memorization Version

```cpp
bool validate(TreeNode* root, long long minVal, long long maxVal) {
    if (!root) return true;
    if (root->val <= minVal || root->val >= maxVal) return false;
    return validate(root->left, minVal, root->val) && 
           validate(root->right, root->val, maxVal);
}
bool isValidBST(TreeNode* root) {
    return validate(root, LLONG_MIN, LLONG_MAX);
}
```
*   **Mnemonic**: DFS, root bounds `LLONG_MIN` to `LLONG_MAX`. Left updates max, right updates min.
*   **Complexity**: $O(N)$ time, $O(H)$ space.
