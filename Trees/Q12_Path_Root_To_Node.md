# Path from Root to Node

## 1. Problem Statement

Given a Binary Tree and a target node value `B`, find the path from the root of the binary tree to the node with value `B`. 

*   **Input**:
    *   A pointer to the root of the binary tree: `TreeNode* root`.
    *   An integer `B` representing the target node's value.
*   **Output**:
    *   A list of integers representing the values of nodes along the path from the root to the target node `B`. If the target is not present, return an empty list.
*   **Constraints**:
    *   $1 \le N \le 10^5$ (Number of nodes in the tree)
    *   $1 \le \text{Node value} \le 10^9$
    *   All node values are distinct (or assume we want the first path to a node with value `B`).
*   **Observations**:
    *   A binary tree path is unique for any node since there are no cycles and every node except the root has exactly one parent.
    *   To find the path, we must explore nodes and dynamically track the current path.
    *   If a path leads to a dead end (a leaf that is not the target), we must backtrack (remove the current node from our path) and explore other branches.

---

## 2. Interview Clarification

A candidate should ask the following questions to clarify requirements:
1.  **Q**: What should we return if the target node `B` does not exist in the tree?
    *   *A*: Return an empty array/vector.
2.  **Q**: Are the values of nodes in the tree guaranteed to be unique?
    *   *A*: Yes, you can assume values are unique. If there are duplicates, find the path to the first node with value `B` in preorder traversal.
3.  **Q**: Can the tree be empty (`nullptr`)?
    *   *A*: Yes, handle the empty tree case by returning an empty path.
4.  **Q**: Do we need to return a path of `TreeNode*` pointers or node values?
    *   *A*: Return a list of node values (integers).

---

## 3. Thinking Process

*   **First Instinct**:
    *   We can traverse the tree in a standard DFS (preorder) manner.
    *   As we visit a node, we add it to our path list.
    *   If the current node's value matches the target `B`, we are done! We return `true` to signal success.
    *   If we reach a null node, we return `false`.
    *   If the target is not in the left subtree and not in the right subtree of the current node, then the current node cannot be part of the path. We must remove it from the path (backtrack) and return `false`.

*   **Optimal Insight**:
    *   By returning a boolean flag (`true`/`false`) from our recursive helper, we can immediately stop exploring other subtrees as soon as the target is found. This prevents unnecessary traversal after finding the node.

---

## 4. Pattern Recognition

*   **Topic**: Binary Tree Traversal.
*   **Pattern**: DFS with Backtracking.
*   **Why**: We need to construct a path from the root. Since a binary tree is recursively structured, DFS naturally builds the path from the ancestor down to the descendant. Backtracking is needed because we only know if a path is invalid after fully exploring it.

---

## 5. Brute Force Solution

*   **Algorithm**:
    *   Generate all paths from the root to all leaf nodes.
    *   Search each path for the target value `B`.
    *   If found, truncate the path at `B` and return it.
*   **Dry Run**:
    *   If the tree is `1 -> 2 -> 3`, we generate paths `[1, 2, 3]`. If `B = 2`, we search `[1, 2, 3]`, find `2` at index 1, and return `[1, 2]`.
*   **Complexity**:
    *   **Time Complexity**: $O(N \cdot H)$ because there can be $O(N)$ leaves, and each path can be up to $O(H)$ in length. Storing/generating all paths is highly redundant.
    *   **Space Complexity**: $O(N \cdot H)$ to store all paths.
*   **Pros/Cons**:
    *   *Pros*: Easy to conceptualize if you already have a "generate all paths" helper.
    *   *Cons*: Extremely inefficient; does way more work than needed.

### Brute Force C++17 Code
```cpp
#include <iostream>
#include <vector>

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

// Helper to generate all paths
void generateAllPaths(TreeNode* root, std::vector<int>& currentPath, std::vector<std::vector<int>>& allPaths) {
    if (!root) return;
    currentPath.push_back(root->val);
    if (!root->left && !root->right) {
        allPaths.push_back(currentPath);
    } else {
        generateAllPaths(root->left, currentPath, allPaths);
        generateAllPaths(root->right, currentPath, allPaths);
    }
    currentPath.pop_back(); // Backtrack
}

std::vector<int> getPathBruteForce(TreeNode* root, int B) {
    std::vector<std::vector<int>> allPaths;
    std::vector<int> currentPath;
    generateAllPaths(root, currentPath, allPaths);
    
    for (const auto& path : allPaths) {
        for (size_t i = 0; i < path.size(); ++i) {
            if (path[i] == B) {
                return std::vector<int>(path.begin(), path.begin() + i + 1);
            }
        }
    }
    return {};
}
```

---

## 6. Better Solution

*   *Note*: The transition from the brute-force "generate all paths" approach directly goes to the single-pass DFS with backtracking. There is no intermediate "better" solution that is not already the optimal $O(N)$ time and $O(H)$ space backtracking approach. We will jump directly to the optimal solution.

---

## 7. Optimal Solution

*   **Approach**: Single-pass DFS with early exit and backtracking.
*   **Algorithm**:
    1.  Create a helper function `getPathHelper(TreeNode* root, int B, vector<int>& path)` that returns a boolean.
    2.  If `root` is `nullptr`, return `false`.
    3.  Push the current node value `root->val` into `path`.
    4.  If `root->val == B`, return `true` (target found).
    5.  Recursively check if the target exists in the left subtree: `getPathHelper(root->left, B, path)`. If it returns `true`, return `true` immediately to bubble up the success.
    6.  Recursively check if the target exists in the right subtree: `getPathHelper(root->right, B, path)`. If it returns `true`, return `true` immediately.
    7.  If both subtrees return `false`, then the current node is not part of the path. Pop the current node value from `path` (backtrack) and return `false`.

### Optimal C++17 Code
```cpp
#include <iostream>
#include <vector>

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
     * Helper function to find path using DFS & backtracking.
     * @param root Current node in the binary tree
     * @param target Target value to find
     * @param path Vector to accumulate the path elements
     * @return true if the target is found in the current subtree, false otherwise
     */
    bool getPathHelper(TreeNode* root, int target, std::vector<int>& path) {
        // Base case: If root is null, target cannot be found in this path
        if (root == nullptr) {
            return false;
        }

        // Action: Add the current node to the path
        path.push_back(root->val);

        // Base case: If the current node is the target, we return true
        if (root->val == target) {
            return true;
        }

        // Recurse: Search in the left subtree. If found, stop searching and return true.
        if (getPathHelper(root->left, target, path)) {
            return true;
        }

        // Recurse: Search in the right subtree. If found, stop searching and return true.
        if (getPathHelper(root->right, target, path)) {
            return true;
        }

        // Backtrack: If not found in left or right subtree, the current node is not
        // part of the path to the target. Remove it and return false.
        path.pop_back();
        return false;
    }

public:
    /**
     * Main function to find the path from root to the node with value B.
     */
    std::vector<int> solve(TreeNode* root, int B) {
        std::vector<int> path;
        if (root == nullptr) {
            return path;
        }
        getPathHelper(root, B, path);
        return path;
    }
};
```

---

## 8. Code Walkthrough

1.  **Function Signature**: `bool getPathHelper(TreeNode* root, int target, vector<int>& path)`
    *   We pass `path` by reference to modify it in-place.
2.  **Base Case (`root == nullptr`)**:
    *   Returns `false`, stopping exploration of this empty branch.
3.  **Path Extension (`path.push_back(root->val)`)**:
    *   Hypothesizes that `root` is on the valid path and appends it.
4.  **Target Match Check (`root->val == target`)**:
    *   If true, returns `true` to immediately bubble up the recursion chain, preventing any pop operations on nodes in the current path.
5.  **Subtree Traversals**:
    *   `if (getPathHelper(root->left, target, path))` checks the left child.
    *   `if (getPathHelper(root->right, target, path))` checks the right child.
    *   The `if` condition allows the success signal (`true`) to propagate immediately to the caller without executing the backtracking step.
6.  **Backtracking (`path.pop_back()`)**:
    *   Executed only if both subtrees failed to find the target. This ensures the path contains only the correct ancestors.

---

## 9. Dry Run

Consider the following tree and target `B = 5`:
```
      1
     / \
    2   3
   / \
  4   5
```

| Step | Call / Node | `path` (state before check) | Action / Check | Return Value | `path` (state after check) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | `Helper(1)` | `[]` | Push `1` | Pending | `[1]` |
| 2 | `Helper(2)` | `[1]` | Push `2` | Pending | `[1, 2]` |
| 3 | `Helper(4)` | `[1, 2]` | Push `4`; `4 != 5` | Pending | `[1, 2, 4]` |
| 4 | `Helper(4->left)` | `[1, 2, 4]` | `nullptr` check | `false` | `[1, 2, 4]` |
| 5 | `Helper(4->right)`| `[1, 2, 4]` | `nullptr` check | `false` | `[1, 2, 4]` |
| 6 | Backtrack `4` | `[1, 2, 4]` | `path.pop_back()` | `false` | `[1, 2]` |
| 7 | `Helper(5)` | `[1, 2]` | Push `5`; `5 == 5` | `true` | `[1, 2, 5]` |
| 8 | Return to `2` | `[1, 2, 5]` | Left child was `false`, Right is `true` | `true` | `[1, 2, 5]` |
| 9 | Return to `1` | `[1, 2, 5]` | Left child was `true` | `true` | `[1, 2, 5]` |

---

## 10. Complexity Analysis

*   **Time Complexity**: $\mathcal{O}(N)$
    *   In the worst case (where the target is at the last leaf or not present), we visit every node in the binary tree exactly once.
*   **Space Complexity**: $\mathcal{O}(H)$
    *   $H$ is the height of the tree.
    *   The recursion stack uses $\mathcal{O}(H)$ space.
    *   The `path` vector stores at most $H + 1$ elements since the maximum length of a root-to-node path is bounded by the height of the tree.
    *   For a balanced tree, $H = \log_2(N)$, resulting in $\mathcal{O}(\log N)$ space. For a skewed tree, $H = N$, resulting in $\mathcal{O}(N)$ space.

---

## 11. Common Mistakes

1.  **Forgetting to Backtrack**: Candidates often forget `path.pop_back()`, leaving dead-end node values in the path.
2.  **Missing Early Exit**: Not using the boolean return value to stop searching. Instead, they do a complete traversal of the tree, which degrades performance and is harder to implement correctly.
3.  **Incorrect Null Handling**: Accessing `root->val` before checking if `root` is `nullptr`.

---

## 12. Edge Cases

| Edge Case | Input Tree / Values | Expected Path | Importance |
| :--- | :--- | :--- | :--- |
| **Empty Tree** | `root = nullptr`, `B = 5` | `[]` | Avoid null pointer dereference. |
| **Root is Target** | `root = [5]`, `B = 5` | `[5]` | Single node path should return immediately. |
| **Target Not Present** | `root = [1, 2, 3]`, `B = 5` | `[]` | Verify backtracking cleans up the path completely. |
| **Target in Skewed Tree**| `1 -> 2 -> 3 -> 4`, `B = 4` | `[1, 2, 3, 4]` | Test performance with worst-case height. |

---

## 13. Interview Explanation

"To find the path from the root to a target node in a binary tree, we can use a Depth First Search (DFS) with backtracking. We maintain a list to store the current path of nodes from the root. As we traverse each node, we add it to the path and check if it matches the target. 

If it does, we return `true` immediately to stop the recursion. If not, we recursively search its left and right subtrees. If both subtrees return `false`, we know that the current node is a dead end. We then remove the node from our path list (backtrack) and return `false`. This ensures we only return the nodes that lead directly to the target. The time complexity is $O(N)$ as we visit each node at most once, and the space complexity is $O(H)$ representing the height of the recursion stack and the path list."

---

## 14. Follow-up Questions

1.  **Q**: How can we use this path helper to find the Lowest Common Ancestor (LCA) of two nodes?
    *   *A*: Find the path from root to `node1` and root to `node2`. Iterate through both paths simultaneously. The last common element in both paths is the LCA.
2.  **Q**: What if the tree has millions of nodes and we need to query multiple targets?
    *   *A*: We could precompute parent pointers for each node using BFS or DFS, storing them in a hash map: `map[node_val] = parent_node`. Then, we can reconstruct the path from any target back to the root in $O(H)$ time by traversing parent pointers.

---

## 15. Variations

1.  **All Root-to-Leaf Paths**: Instead of finding one target, generate all paths from the root to every leaf node.
    *   *Solution change*: We do not exit early. When a leaf is reached, we copy the path to a global list of paths.
2.  **Path Sum**: Find a path from root to leaf that sums up to a target value.
    *   *Solution change*: Track the sum along the path; verify the sum matches target when a leaf is reached.

---

## 16. Revision Notes

*   **Backtracking Principle**: Add before recursing, remove after recursing (if recursion did not find the target).
*   **Early Return Rule**: Use the recursive function's boolean return value to bubble up the result immediately.
*   **Time/Space**: $O(N)$ time / $O(H)$ space.

---

## 17. Memorization Version

```cpp
bool getPath(TreeNode* root, int target, vector<int>& path) {
    if (!root) return false;
    path.push_back(root->val);
    if (root->val == target) return true;
    if (getPath(root->left, target, path) || getPath(root->right, target, path)) 
        return true;
    path.pop_back(); // backtrack
    return false;
}
```
*   **Remember**: DFS, push current, check match, recurse left/right with short-circuit OR, pop on fail.
*   **Complexity**: $O(N)$ time, $O(H)$ space.
