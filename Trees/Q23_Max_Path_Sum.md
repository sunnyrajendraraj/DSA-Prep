# Binary Tree Maximum Path Sum

## 1. Problem Statement

A **path** in a binary tree is a sequence of nodes where each pair of adjacent nodes in the sequence has an edge connecting them. A node can only appear in the sequence at most once. Note that the path does not need to pass through the root.

The **path sum** of a path is the sum of the node's values in the path.

Given the root of a binary tree, return the maximum path sum of any non-empty path.

*   **Input**:
    *   A pointer to the root of the binary tree: `TreeNode* root`.
*   **Output**:
    *   An integer representing the maximum path sum.
*   **Constraints**:
    *   The number of nodes in the tree is in the range $[1, 3 \cdot 10^4]$.
    *   $-1000 \le \text{Node value} \le 1000$.

*   **Observations**:
    *   Node values can be negative.
    *   A path can start and end at any node.
    *   For any node acting as the highest point (LCA) of a path, the path sum is:
        $$\text{Path Sum} = \text{Node Value} + \text{Max Gain from Left Subtree} + \text{Max Gain from Right Subtree}$$
    *   If the max gain from a subtree is negative, we should ignore that subtree (i.e., take a gain of `0` instead).
    *   To find the overall maximum path sum, we must calculate this "highest point path sum" for every node and record the global maximum.
    *   When returning a value to the node's parent, the node can only extend a path through *one* of its children. So it returns:
        $$\text{Return Value} = \text{Node Value} + \max(\text{Max Gain from Left Subtree}, \text{Max Gain from Right Subtree})$$

---

## 2. Interview Clarification

1.  **Q**: Can the path consist of only a single node?
    *   *A**: Yes, the path must be non-empty and can contain a single node.
2.  **Q**: What if all nodes in the tree have negative values?
    *   *A**: The answer will be the value of the single node with the least negative value (maximum node value).
3.  **Q**: Can the path span across the root node?
    *   *A*: Yes, but it doesn't have to. The maximum path could lie entirely within the left or right subtree.

---

## 3. Thinking Process

*   **First Instinct**:
    *   For each node, we want to find the maximum path that has this node as its highest point.
    *   The maximum path through node `u` is `u->val + maxPathStartingAt(u->left) + maxPathStartingAt(u->right)`.
    *   To do this efficiently, we can use a post-order traversal (DFS) since we need the child subtrees' gains before calculating the parent's value.
    *   At each node, we calculate:
        1.  `leftGain = max(0, dfs(node->left))` (ignore if negative).
        2.  `rightGain = max(0, dfs(node->right))` (ignore if negative).
        3.  Current path sum through this node = `node->val + leftGain + rightGain`.
        4.  Update our global maximum with this current path sum.
        5.  Return `node->val + max(leftGain, rightGain)` to the parent. This represents the maximum single-branch path starting at this node.

---

## 4. Pattern Recognition

*   **Topic**: Tree DFS / Post-Order Traversal.
*   **Pattern**: Recursive Gain Accumulation with Global State Update.
*   **Why**: Calculating path sums requires combining values from left and right subtrees. Post-order DFS (`left` then `right` then `current`) naturally returns values bottom-up. Updating a global maximum during this bottom-up phase allows us to check all possible path turn-points in a single pass.

---

## 5. Brute Force Solution

*   **Algorithm**:
    *   For every node `u` in the tree:
        *   Find the maximum path starting at `u->left` going downwards.
        *   Find the maximum path starting at `u->right` going downwards.
        *   Compute path sum through `u`: `u->val + left + right`.
        *   Update global max.
    *   Repeat this recursively for all nodes.
*   **Complexity**:
    *   **Time Complexity**: $O(N^2)$ in the worst case (skewed tree) because we run a downward path sum search of size $O(N)$ for each of the $O(N)$ nodes.
    *   **Space Complexity**: $O(H)$ recursion stack.
*   **Pros/Cons**:
    *   *Cons*: Redundant calculations make it too slow.

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

// Returns max path starting at node going down (one-way)
int maxPathDown(TreeNode* root) {
    if (!root) return 0;
    int left = std::max(0, maxPathDown(root->left));
    int right = std::max(0, maxPathDown(root->right));
    return root->val + std::max(left, right);
}

class SolutionBrute {
private:
    int maxPathSumVal = INT_MIN;

    void traverse(TreeNode* root) {
        if (!root) return;
        int leftGain = std::max(0, maxPathDown(root->left));
        int rightGain = std::max(0, maxPathDown(root->right));
        maxPathSumVal = std::max(maxPathSumVal, root->val + leftGain + rightGain);
        traverse(root->left);
        traverse(root->right);
    }

public:
    int maxPathSum(TreeNode* root) {
        traverse(root);
        return maxPathSumVal;
    }
};
```

---

## 6. Better Solution

*   *Note*: The transition from the $O(N^2)$ brute-force traversal directly to the single-pass $O(N)$ post-order traversal represents the optimal jump. There is no intermediate "better" solution.

---

## 7. Optimal Solution

*   **Approach**: Single-pass post-order DFS.
*   **Algorithm**:
    1.  Initialize a global/member variable `maxSum = INT_MIN`.
    2.  Define a recursive function `maxGain(TreeNode* node)` that returns the maximum path sum starting at `node` and extending into at most one of its subtrees.
    3.  In `maxGain(node)`:
        *   If `node` is `nullptr`, return `0` (base case).
        *   Recursively calculate max gain of left child: `leftGain = max(0, maxGain(node->left))`. We clamp to `0` so we don't include paths that sum to a negative value.
        *   Recursively calculate max gain of right child: `rightGain = max(0, maxGain(node->right))`.
        *   Calculate the path sum if we choose `node` as the turn-point (highest point of the path):
            $$\text{currentPathSum} = \text{node->val} + \text{leftGain} + \text{rightGain}$$
        *   Update `maxSum = max(maxSum, currentPathSum)`.
        *   Return the maximum gain the parent node can obtain by choosing this node:
            $$\text{returnVal} = \text{node->val} + \max(\text{leftGain}, \text{rightGain})$$

### Optimal C++17 Code
```cpp
#include <iostream>
#include <algorithm>
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
    int maxSum;

    /**
     * Helper function that calculates the maximum single-branch path gain 
     * starting from the current node. It also updates the global maxSum.
     */
    int maxGain(TreeNode* node) {
        if (node == nullptr) {
            return 0;
        }

        // Recursively compute the maximum path sum of the left and right subtrees.
        // If the path sum is negative, we ignore it by clamping to 0.
        int leftGain = std::max(0, maxGain(node->left));
        int rightGain = std::max(0, maxGain(node->right));

        // The maximum path sum using the current node as the highest point (LCA)
        int currentPathSum = node->val + leftGain + rightGain;

        // Update the global maximum path sum found so far
        maxSum = std::max(maxSum, currentPathSum);

        // Return the maximum gain the parent node can achieve by choosing 
        // to continue the path through the current node
        return node->val + std::max(leftGain, rightGain);
    }

public:
    /**
     * Computes the maximum path sum of any non-empty path in the binary tree.
     */
    int maxPathSum(TreeNode* root) {
        maxSum = INT_MIN;
        maxGain(root);
        return maxSum;
    }
};
```

---

## 8. Code Walkthrough

1.  **Global Variable `maxSum`**:
    *   Initialized to `INT_MIN` to handle trees containing only negative numbers.
2.  **Left and Right Gain Checks**:
    *   `std::max(0, maxGain(node->left))`
    *   Clamping to `0` acts as a decision: "If a subtree's path sum is negative, we choose not to extend our path into it."
3.  **Path Union**:
    *   `currentPathSum = node->val + leftGain + rightGain;`
    *   This is the path sum where the path starts in the left subtree, goes up to the current node, and down into the right subtree. This node is the peak.
4.  **State Update**:
    *   `maxSum = std::max(maxSum, currentPathSum);` compares the peak path sum to the global maximum.
5.  **Single Branch Return**:
    *   `return node->val + std::max(leftGain, rightGain);`
    *   To be a valid path, the parent node can only link the current node to *either* its left child or its right child, but not both.

---

## 9. Dry Run

Consider the tree:
```
     -10
     /  \
    9   20
       /  \
      15   7
```

*   **Execution**:
    *   `maxSum = INT_MIN`.

1.  `maxGain(-10)`:
    *   Calls `maxGain(9)`.
2.  `maxGain(9)`:
    *   Children are null $\rightarrow$ `leftGain = 0`, `rightGain = 0`.
    *   `currentPathSum = 9 + 0 + 0 = 9`.
    *   `maxSum = max(INT_MIN, 9) = 9`.
    *   Returns `9 + 0 = 9`.
3.  Back to `-10`. `leftGain = max(0, 9) = 9`.
    *   Calls `maxGain(20)`.
4.  `maxGain(20)`:
    *   Calls `maxGain(15)`.
5.  `maxGain(15)`:
    *   Children are null $\rightarrow$ `leftGain = 0`, `rightGain = 0`.
    *   `currentPathSum = 15 + 0 + 0 = 15`.
    *   `maxSum = max(9, 15) = 15`.
    *   Returns `15`.
6.  `maxGain(20)`: `leftGain = max(0, 15) = 15`.
    *   Calls `maxGain(7)`.
7.  `maxGain(7)`:
    *   Children null $\rightarrow$ returns `7`. `maxSum` remains `15` (since `7 < 15`).
8.  `maxGain(20)` resumes:
    *   `leftGain = 15`, `rightGain = max(0, 7) = 7`.
    *   `currentPathSum = 20 + 15 + 7 = 42`.
    *   `maxSum = max(15, 42) = 42`.
    *   Returns `20 + max(15, 7) = 35`.
9.  `maxGain(-10)` resumes:
    *   `leftGain = 9`, `rightGain = max(0, 35) = 35`.
    *   `currentPathSum = -10 + 9 + 35 = 34`.
    *   `maxSum = max(42, 34) = 42`.
    *   Returns `-10 + 35 = 25`.
*   **Final Output**: `maxSum = 42` (corresponding to the path `15 -> 20 -> 7`).

---

## 10. Complexity Analysis

*   **Time Complexity**: $\mathcal{O}(N)$
    *   We perform a post-order traversal, visiting each node in the tree exactly once.
*   **Space Complexity**: $\mathcal{O}(H)$
    *   $H$ is the height of the tree.
    *   The space is occupied by the recursion stack.
    *   For a balanced tree, $H = \log N$. For a skewed tree, $H = N$.

---

## 11. Common Mistakes

1.  **Forgetting to return only one branch**: A common bug is returning `node->val + leftGain + rightGain` to the parent. This forms a split path, which is invalid because a path cannot branch into two children of a node and then go up to its parent.
2.  **Not clamping to zero**: Failing to use `std::max(0, ...)` for subtree gains. If a subtree returns a negative sum, we should not include it in our path.
3.  **Initializing `maxSum` to 0**: If all node values are negative (e.g. `[-3]`), the answer is `-3`. If `maxSum` was initialized to `0`, the code would return `0` which is incorrect.

---

## 12. Edge Cases

| Edge Case | Input Tree | Expected Output | Importance |
| :--- | :--- | :--- | :--- |
| **All Negative Values** | `[-3, -2, -1]` | `-1` | Checks if `maxSum` handles negative values correctly. |
| **Single Node** | `[5]` | `5` | Verify base case recursion values. |
| **Skewed Positive Path**| `1 -> 2 -> 3` | `6` | Simple linear sum check. |

---

## 13. Interview Explanation

"To solve the Maximum Path Sum problem, we use a post-order DFS traversal. For any node, a path can use it as the highest point (turn-point) and extend into both its left and right subtrees. The sum of this path is the node's value plus the maximum gains from both children.

However, to be a valid path, we cannot split into both children and also propagate up to the node's parent. Therefore, when returning a path sum to the parent, we must return only the single best branch: the node's value plus the maximum of the left or right subtree gains.

During our recursion, we calculate the max gain for the left and right subtrees. If a subtree's gain is negative, we ignore it by clamping it to 0. We compute the sum of the path peaking at the current node and update a global maximum variable. Finally, we return the single-branch gain to the parent. This runs in $O(N)$ time and uses $O(H)$ space."

---

## 14. Follow-up Questions

1.  **Q**: How can we print the actual path nodes that form the maximum path sum?
    *   *A*: We can keep track of parent pointers or save the nodes that yield the updated global maximum inside our recursive checks.
2.  **Q**: What if we are only allowed to find the max path sum from root to any node?
    *   *A*: The problem becomes simpler; we do not update a global maximum using `left + right`. Instead, we return the maximum path starting from the root going down.

---

## 15. Variations

1.  **Path Sum III**: Find paths that sum to a target value.
2.  **Diameter of Binary Tree**:
    *   *Connection*: Similar logic. The diameter is the maximum path length between two leaf nodes. The formula is `left_depth + right_depth` at each node, returning `1 + max(left, right)`.

---

## 16. Revision Notes

*   **Formula for turn-point**: `current = node->val + leftGain + rightGain`.
*   **Formula for return value**: `return node->val + max(leftGain, rightGain)`.
*   **Negative Filter**: `std::max(0, child_gain)` prevents negative branches from degrading the sum.
*   **Initial State**: Initialize `maxSum` to `INT_MIN`.

---

## 17. Memorization Version

```cpp
class Solution {
    int max_sum = INT_MIN;
    int dfs(TreeNode* root) {
        if (!root) return 0;
        int l = max(0, dfs(root->left));
        int r = max(0, dfs(root->right));
        max_sum = max(max_sum, root->val + l + r);
        return root->val + max(l, r);
    }
public:
    int maxPathSum(TreeNode* root) {
        dfs(root);
        return max_sum;
    }
};
```
*   **Mnemonic**: Post-order DFS. Calculate `l` and `r` clamped to `0`. Update `max_sum` with `root + l + r`. Return `root + max(l, r)`.
*   **Complexity**: $O(N)$ time, $O(H)$ space.
