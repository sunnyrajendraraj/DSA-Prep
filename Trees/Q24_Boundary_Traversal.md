# Boundary Traversal of Binary Tree

## 1. Problem Statement

Given a Binary Tree, return its **boundary traversal** in a counter-clockwise direction, starting from the root. 

The boundary traversal consists of:
1. **Left Boundary:** The sequence of nodes from the root to the left-most node, excluding leaf nodes.
2. **Leaf Nodes:** All leaf nodes visited from left to right.
3. **Right Boundary:** The sequence of nodes from the right-most node back to the root (excluding leaf nodes and root), listed in bottom-up order.

### Input
- The root of a binary tree.

### Output
- A list of integers representing the boundary nodes in counter-clockwise order.

### Constraints
- The number of nodes in the tree is in the range $[0, 10^5]$.
- Node values: $-10^5 \le \text{Node.val} \le 10^5$.

### Observations
- We must avoid duplicates. For example, the root shouldn't be added twice (once for left, once for right), and leaf nodes shouldn't be duplicated if they are also part of the left/right boundaries.
- The root is processed first.
- The right boundary needs to be reversed (or traversed bottom-up).

---

## 2. Interview Clarification

Before writing any code, a candidate should ask the interviewer:
1. **How should a single-node tree be handled?** 
   - *Answer:* Just return the root value itself.
2. **What if the tree is empty?**
   - *Answer:* Return an empty list.
3. **If a node in the left boundary does not have a left child, does the left boundary continue through its right child?**
   - *Answer:* Yes. The boundary follows the path we would see from the outside. If there's no left child, we move to the right child. Same applies to the right boundary (if no right child, go to left).
4. **Should leaf nodes be collected via standard depth-first search?**
   - *Answer:* Yes, a simple preorder or inorder traversal can collect all leaves from left to right.

---

## 3. Thinking Process

Let's trace how we can traverse the boundary of a tree:
- **First instinct:** We cannot do this with a single standard DFS or BFS traversal. The left boundary goes top-down, leaves go left-to-right, and the right boundary goes bottom-up.
- **Deconstruction:**
  - Let's break the problem into three sub-problems:
    1. Collect the left boundary (excluding leaves).
    2. Collect all leaf nodes (left-to-right).
    3. Collect the right boundary (excluding leaves) and reverse it.
- **Left Boundary Logic:**
  - Start from `root->left`.
  - Traverse down: if `curr` has a left child, go left; otherwise, go right.
  - Stop when we reach a leaf node (do not include it, as it will be covered by the leaf traversal).
- **Leaf Nodes Logic:**
  - A simple preorder/inorder traversal of the entire tree.
  - If a node is a leaf (`!node->left && !node->right`), add it to our result.
- **Right Boundary Logic:**
  - Start from `root->right`.
  - Traverse down: if `curr` has a right child, go right; otherwise, go left.
  - Stop when we reach a leaf node.
  - Since we need the right boundary bottom-up, we can either:
    - Push nodes onto a stack or a temporary list, then reverse and add to the final result.
    - Collect them recursively: make the recursive call first, then add the current node to the list.

---

## 4. Pattern Recognition

- **Tree Decomposition:** This problem uses modular decomposition of a binary tree. Instead of treating the tree as a single entity, we traverse specific "sub-paths".
- **Standard Traversals Used:**
  - Left/right boundary traversal: Path traversal (similar to finding a path to a node).
  - Leaf traversal: Standard DFS (Preorder/Inorder).

---

## 5. Brute Force Solution

A naive way would be to perform a standard traversal (like level-order or DFS), capture the coordinates (x, y) of each node, and try to construct the boundary from the convex hull or external nodes. This is extremely overcomplicated and slow, requiring $O(N \log N)$ or $O(N^2)$ time with auxiliary coordinates.

Since brute force is not practical here, we jump directly to the standard modular solution, which is already highly intuitive.

---

## 6. Better Solution

There is no intermediate "better" solution between a coordinate-based convex hull approach and the standard $O(N)$ three-pass traversal. We proceed directly to the Optimal Solution.

---

## 7. Optimal Solution

The optimal approach divides the traversal into three clear, independent helper functions:
1. **`addLeftBoundary`**: Start from `root->left`. While the node is not a leaf, add it to the result. If a left child exists, go left; else go right.
2. **`addLeaves`**: Recursively traverse the tree. If a node is a leaf, append it to the result.
3. **`addRightBoundary`**: Start from `root->right`. While the node is not a leaf, push it to a temporary stack/vector. If a right child exists, go right; else go left. Finally, append the elements in reverse order.

### Algorithm
1. Check if the tree is empty. If so, return.
2. If the root is not a leaf, add the root's value to the result list.
3. Call `addLeftBoundary(root->left)`.
4. Call `addLeaves(root)` to find all leaf nodes.
5. Call `addRightBoundary(root->right)`.

### C++17 Code

```cpp
#include <vector>
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
    // Helper to check if a node is a leaf
    bool isLeaf(TreeNode* node) {
        return (node != nullptr) && (node->left == nullptr) && (node->right == nullptr);
    }

    // 1. Add Left Boundary (excluding leaf nodes)
    void addLeftBoundary(TreeNode* root, std::vector<int>& res) {
        TreeNode* curr = root;
        while (curr != nullptr) {
            if (!isLeaf(curr)) {
                res.push_back(curr->val);
            }
            if (curr->left != nullptr) {
                curr = curr->left;
            } else {
                curr = curr->right;
            }
        }
    }

    // 2. Add Leaf Nodes (left-to-right)
    void addLeaves(TreeNode* root, std::vector<int>& res) {
        if (root == nullptr) {
            return;
        }
        if (isLeaf(root)) {
            res.push_back(root->val);
            return;
        }
        addLeaves(root->left, res);
        addLeaves(root->right, res);
    }

    // 3. Add Right Boundary (excluding leaf nodes, bottom-up)
    void addRightBoundary(TreeNode* root, std::vector<int>& res) {
        TreeNode* curr = root;
        std::vector<int> temp;
        while (curr != nullptr) {
            if (!isLeaf(curr)) {
                temp.push_back(curr->val);
            }
            if (curr->right != nullptr) {
                curr = curr->right;
            } else {
                curr = curr->left;
            }
        }
        // Append in reverse order to get bottom-up
        for (int i = static_cast<int>(temp.size()) - 1; i >= 0; --i) {
            res.push_back(temp[i]);
        }
    }

public:
    std::vector<int> boundaryTraversal(TreeNode* root) {
        std::vector<int> res;
        if (root == nullptr) {
            return res;
        }

        // Add root if it is not a leaf
        if (!isLeaf(root)) {
            res.push_back(root->val);
        }

        // Process Left Boundary, Leaves, and Right Boundary
        addLeftBoundary(root->left, res);
        addLeaves(root, res);
        addRightBoundary(root->right, res);

        return res;
    }
};
```

---

## 8. Code Walkthrough

- **`isLeaf(TreeNode* node)`**: A helper utility returning `true` if both `left` and `right` child pointers are `nullptr`.
- **`addLeftBoundary(TreeNode* root, vector<int>& res)`**:
  - We initialize `curr` to the left child of the root.
  - We loop while `curr` is not null. Inside, we check `!isLeaf(curr)` before appending to avoid adding leaf nodes here (since the leaf traversal will capture them).
  - We prioritize moving left: `curr = curr->left`. If `curr->left` is null, we move right: `curr = curr->right`.
- **`addLeaves(TreeNode* root, vector<int>& res)`**:
  - This is a standard preorder-like DFS.
  - If the node is a leaf, we add its value and return immediately.
  - Otherwise, recursively call on `root->left` then `root->right`.
- **`addRightBoundary(TreeNode* root, vector<int>& res)`**:
  - We initialize `curr` to the right child of the root.
  - We traverse downward: prioritizing `curr->right` over `curr->left`.
  - Non-leaf nodes are stored in a temporary vector `temp`.
  - Finally, we loop backwards through `temp` and push elements into `res` to obtain the bottom-up order.

---

## 9. Dry Run

Let's dry run the algorithm with the following binary tree:

```
        1
       / \
      2   3
     / \   \
    4   5   6
       / \
      7   8
```

### Initial State
`res = []`

### Step 1: Add root
Root is `1`. It is not a leaf.
`res = [1]`

### Step 2: Left Boundary
Start at `root->left` which is `2`.
- `curr = 2`: Not a leaf. `res = [1, 2]`. Left child exists (`4`), so `curr = 4`.
- `curr = 4`: Leaf! Terminate loop.
`res` remains `[1, 2]`.

### Step 3: Leaves Traversal
DFS on root `1`:
- Visit `1` (not leaf) -> go to `2`.
- Visit `2` (not leaf) -> go to `4`.
- Visit `4` (leaf) -> `res = [1, 2, 4]`.
- Visit `5` (not leaf) -> go to `7`.
- Visit `7` (leaf) -> `res = [1, 2, 4, 7]`.
- Visit `8` (leaf) -> `res = [1, 2, 4, 7, 8]`.
- Visit `3` (not leaf) -> go to `6`.
- Visit `6` (leaf) -> `res = [1, 2, 4, 7, 8, 6]`.

### Step 4: Right Boundary
Start at `root->right` which is `3`.
- `curr = 3`: Not a leaf. `temp = [3]`. Right child exists (`6`), so `curr = 6`.
- `curr = 6`: Leaf! Terminate loop.
- Reverse `temp`: `[3]`.
- Append to `res`: `res = [1, 2, 4, 7, 8, 6, 3]`.

### Final Output
`[1, 2, 4, 7, 8, 6, 3]`

---

## 10. Complexity Analysis

| Metric | Complexity | Reasoning |
| :--- | :--- | :--- |
| **Time Complexity** | $O(N)$ | We visit every node in the tree at most a constant number of times (leaves traversal visits all $N$ nodes, left and right boundaries visit at most height $H$ nodes). |
| **Space Complexity** | $O(H)$ | The recursion stack for `addLeaves` takes $O(H)$ space. The temporary array for the right boundary takes $O(H)$ space. $H$ is the height of the tree. |

---

## 11. Common Mistakes

1. **Including the root node multiple times:** Make sure you only add the root once and do not re-add it during the left or right boundary traversal.
2. **Including boundary nodes as leaf nodes:** Handled by check `!isLeaf(curr)` in boundary traversals.
3. **Wrong order for right boundary:** The right boundary must be printed bottom-up. Forgetting to reverse the right boundary is a common bug.
4. **Incorrect boundary progression:** If a left boundary node has no left child, you must go right (do not terminate!).

---

## 12. Edge Cases

| Edge Case | Example | Handling |
| :--- | :--- | :--- |
| **Empty Tree** | `nullptr` | Return empty vector immediately. |
| **Single Node** | `[1]` | Handled correctly; `isLeaf(root)` is true, so root is not added in step 1. `addLeaves` will add it. |
| **Skewed Left Tree** | `1 -> 2 -> 3` | Left boundary collects `1, 2`, leaf is `3`, right boundary is empty. |
| **Skewed Right Tree**| `1 -> 2 -> 3` (all right) | Left boundary is empty, leaf is `3`, right boundary is `2` (reversed). |

---

## 13. Interview Explanation

"To solve the Boundary Traversal of a Binary Tree, we divide the problem into three distinct, counter-clockwise phases:
1. First, we traverse the left boundary starting from the root's left child. We follow the leftmost path, adding nodes to our result only if they are not leaf nodes.
2. Next, we perform a standard depth-first search (like an inorder traversal) on the entire tree to collect all the leaf nodes in order from left to right.
3. Finally, we traverse the right boundary starting from the root's right child, following the rightmost path. We collect these nodes in a temporary list, excluding leaf nodes, and append them in reverse order to ensure a bottom-up direction.

Combining these three phases gives us the complete boundary traversal without duplicates in $O(N)$ time and $O(H)$ space."

---

## 14. Follow-up Questions

1. **How would you modify the solution to traverse in clockwise order?**
   - *Answer:* Reverse the operations: first traverse the right boundary top-down, then collect leaf nodes from right to left (by visiting `right` before `left` in DFS), and finally traverse the left boundary bottom-up.
2. **Can you do this iteratively without recursion for leaf traversal?**
   - *Answer:* Yes, we can use an iterative DFS using a stack to collect all leaf nodes.

---

## 15. Variations

1. **Clockwise Boundary Traversal:** Described in follow-up 1.
2. **Boundary Traversal of a N-ary Tree:** The leaves are collected similarly; the left boundary follows the first child, and the right boundary follows the last child of each node.

---

## 16. Revision Notes

- **Crucial Rule:** The boundary traversal is counter-clockwise.
- **Root Node:** Handle it carefully as a special case to avoid duplicates.
- **Leaf exclusion:** Left and Right boundary traversals must skip leaf nodes using an `isLeaf()` check.
- **Complexity:** $Time: O(N)$, $Space: O(H)$ where $H$ is the height of the tree.

---

## 17. Memorization Version

```text
Boundary Traversal:
1. If root is null, return.
2. Add root if not leaf.
3. Traverse left boundary: go left if possible, else right. Skip leaves.
4. Traverse leaves: DFS (Inorder/Preorder) to add all leaves left-to-right.
5. Traverse right boundary: go right if possible, else left. Skip leaves. Push to temp, append reversed.
Complexity: Time O(N), Space O(H).
```
