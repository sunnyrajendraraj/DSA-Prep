# Left View and Right View of a Binary Tree

## 1. Problem Statement

Given a Binary Tree, return the **Left View** and **Right View** of its nodes.

*   **Left View**: The set of nodes visible when the tree is visited from the left side. (i.e., the first node at each level).
*   **Right View**: The set of nodes visible when the tree is visited from the right side. (i.e., the last node at each level).

*   **Input**:
    *   A pointer to the root of the binary tree: `TreeNode* root`.
*   **Output**:
    *   For Left View: A vector of integers `vector<int>` containing the first node of each level from top to bottom.
    *   For Right View: A vector of integers `vector<int>` containing the last node of each level from top to bottom.
*   **Constraints**:
    *   $0 \le N \le 10^5$ (Number of nodes in the tree)
    *   $1 \le \text{Node value} \le 10^5$
*   **Observations**:
    *   The depth of the output vectors is equal to the height of the tree.
    *   Left view contains the first node of each level.
    *   Right view contains the last node of each level.
    *   We can compute these either using Level Order Traversal (BFS) or using Depth First Search (DFS) with level tracking.

---

## 2. Interview Clarification

1.  **Q**: What should we return if the tree is empty?
    *   *A*: Return an empty vector `std::vector<int>{}`.
2.  **Q**: Can we compute both views using a single traversal?
    *   *A*: Yes, but typically the interviewer will ask for one or the other. We will provide clean, independent implementations for both, utilizing the most optimal DFS approach.
3.  **Q**: Why is DFS preferred over BFS by interviewers for this problem?
    *   *A*: DFS only uses $O(H)$ space (recursion stack) where $H$ is the height of the tree, which is $O(\log N)$ on average. BFS uses $O(W)$ space where $W$ is the maximum width of the tree, which can be up to $O(N)$ for a balanced tree.

---

## 3. Thinking Process

*   **First Instinct (BFS)**:
    *   Perform a standard level-order traversal (BFS) using a queue.
    *   For each level, track the size of the queue.
    *   **Left View**: Record the value of the first node processed at each level (when `i == 0`).
    *   **Right View**: Record the value of the last node processed at each level (when `i == size - 1`).
    *   This is highly intuitive, but uses $\mathcal{O}(W)$ auxiliary space.

*   **Optimal Instinct (DFS - Recursive View)**:
    *   We can use DFS to traverse the tree while tracking the current depth level.
    *   We maintain our result vector. If the size of the result vector matches the current level, it means we are visiting this level for the **very first time**.
    *   **Right View**:
        *   If we visit the right subtree first (Root $\rightarrow$ Right $\rightarrow$ Left), then the first node we encounter at any level `L` will be the rightmost node of that level.
        *   If `level == result.size()`, push `root->val` to the result.
        *   Recurse right, then recurse left.
    *   **Left View**:
        *   If we visit the left subtree first (Root $\rightarrow$ Left $\rightarrow$ Right), then the first node we encounter at any level `L` will be the leftmost node of that level.
        *   If `level == result.size()`, push `root->val` to the result.
        *   Recurse left, then recurse right.

---

## 4. Pattern Recognition

*   **Topic**: Tree Traversal.
*   **Pattern**: Recursive DFS with level-tracking (Preorder variant).
*   **Why**: By matching the size of the output array to the traversal depth, we can selectively record only the first node visited at each level. Controlling the order of recursive calls (left-first vs. right-first) allows us to toggle between Left and Right views.

---

## 5. Brute Force Solution

*   **Algorithm**:
    *   Perform a standard Level Order Traversal (BFS) of the tree.
    *   Store all levels as a 2D vector: `vector<vector<int>> levels`.
    *   **Left View**: Extract the first element of each inner vector: `levels[i][0]`.
    *   **Right View**: Extract the last element of each inner vector: `levels[i].back()`.
*   **Complexity**:
    *   **Time Complexity**: $O(N)$ to traverse all nodes.
    *   **Space Complexity**: $O(N)$ to store all nodes level by level.
*   **Pros/Cons**:
    *   *Pros*: Easy to explain and build on top of level order traversal.
    *   *Cons*: Redundant space usage. We store all nodes of the tree in a 2D vector just to extract one node per level.

### Brute Force C++17 Code
```cpp
#include <iostream>
#include <vector>
#include <queue>

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

std::vector<int> rightViewBFS(TreeNode* root) {
    if (!root) return {};
    std::vector<int> view;
    std::queue<TreeNode*> q;
    q.push(root);

    while (!q.empty()) {
        int size = q.size();
        for (int i = 0; i < size; ++i) {
            TreeNode* curr = q.front();
            q.pop();
            // Record last node of this level
            if (i == size - 1) {
                view.push_back(curr->val);
            }
            if (curr->left) q.push(curr->left);
            if (curr->right) q.push(curr->right);
        }
    }
    return view;
}
```

---

## 6. Better Solution

*   *Note*: The transition from storing a full 2D level-order array to the space-optimized DFS recursion forms our optimal solution.

---

## 7. Optimal Solution

*   **Approach**: Depth First Search (DFS) with level-tracking.
*   **Space Savings**: Reduces auxiliary space to $O(H)$ (recursion stack) compared to $O(W)$ in BFS.

### Left View Implementation
*   **Order**: Root $\rightarrow$ Left $\rightarrow$ Right.
*   **Condition**: If `level == result.size()`, append `root->val`.

### Right View Implementation
*   **Order**: Root $\rightarrow$ Right $\rightarrow$ Left.
*   **Condition**: If `level == result.size()`, append `root->val`.

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
     * Helper for Left View: Preorder (Root -> Left -> Right)
     */
    void getLeftView(TreeNode* root, int level, std::vector<int>& view) {
        if (root == nullptr) {
            return;
        }

        // If this level is visited for the first time
        if (level == view.size()) {
            view.push_back(root->val);
        }

        // Traverse Left first, then Right
        getLeftView(root->left, level + 1, view);
        getLeftView(root->right, level + 1, view);
    }

    /**
     * Helper for Right View: Reverse Preorder (Root -> Right -> Left)
     */
    void getRightView(TreeNode* root, int level, std::vector<int>& view) {
        if (root == nullptr) {
            return;
        }

        // If this level is visited for the first time
        if (level == view.size()) {
            view.push_back(root->val);
        }

        // Traverse Right first, then Left
        getRightView(root->right, level + 1, view);
        getRightView(root->left, level + 1, view);
    }

public:
    /**
     * Returns the Left View of the binary tree.
     */
    std::vector<int> leftSideView(TreeNode* root) {
        std::vector<int> view;
        getLeftView(root, 0, view);
        return view;
    }

    /**
     * Returns the Right View of the binary tree.
     */
    std::vector<int> rightSideView(TreeNode* root) {
        std::vector<int> view;
        getRightView(root, 0, view);
        return view;
    }
};
```

---

## 8. Code Walkthrough

1.  **Level Parameter**: We pass `level` starting at `0`. When recursing to children, we pass `level + 1`.
2.  **Size Check**: `if (level == view.size())`
    *   Initially, `view` is empty (size `0`). At `level = 0`, `0 == 0`, so we push the root value. Now size is `1`.
    *   At the next depth (`level = 1`), `1 == 1` (since size is `1`), so we push the first node encountered. Size becomes `2`.
    *   Any other node visited at `level = 1` will find that `1 != 2` (size is `2`), and will not be added.
3.  **Recursion Order**:
    *   For **Left View**: We call `left` first. This guarantees that the leftmost node at each level is processed first and gets added.
    *   For **Right View**: We call `right` first. This guarantees that the rightmost node at each level is processed first and gets added.

---

## 9. Dry Run

Consider the tree:
```
      1
     / \
    2   3
     \   \
      4   5
```

### Dry Run for Right View (Root $\rightarrow$ Right $\rightarrow$ Left):
*   `view = []`

1.  `getRightView(1, level = 0)`:
    *   `level (0) == view.size() (0)` $\rightarrow$ Push `1`. `view = [1]`.
    *   Recurse Right: `getRightView(3, 1)`.
2.  `getRightView(3, level = 1)`:
    *   `level (1) == view.size() (1)` $\rightarrow$ Push `3`. `view = [1, 3]`.
    *   Recurse Right: `getRightView(5, 2)`.
3.  `getRightView(5, level = 2)`:
    *   `level (2) == view.size() (2)` $\rightarrow$ Push `5`. `view = [1, 3, 5]`.
    *   Recurse Right: `nullptr` (returns).
    *   Recurse Left: `nullptr` (returns).
4.  Unwind to node `3`:
    *   Recurse Left: `nullptr` (returns).
5.  Unwind to node `1`:
    *   Recurse Left: `getRightView(2, 1)`.
6.  `getRightView(2, level = 1)`:
    *   `level (1) == view.size() (3)` $\rightarrow$ `1 != 3` $\rightarrow$ Skip push.
    *   Recurse Right: `getRightView(4, 2)`.
7.  `getRightView(4, level = 2)`:
    *   `level (2) == view.size() (3)` $\rightarrow$ `2 != 3` $\rightarrow$ Skip push.
    *   Recurse Right/Left: `nullptr` (returns).
*   **Final Right View**: `[1, 3, 5]`

---

## 10. Complexity Analysis

*   **Time Complexity**: $\mathcal{O}(N)$
    *   We visit every node in the binary tree exactly once.
*   **Space Complexity**: $\mathcal{O}(H)$
    *   $H$ is the height of the tree.
    *   The space is taken by the recursion stack.
    *   In the worst case (skewed tree), height $H = N$, giving $\mathcal{O}(N)$ space.
    *   In the best case (balanced tree), height $H = \log N$, giving $\mathcal{O}(\log N)$ space.
    *   Note that this is more space-efficient than BFS, which takes $\mathcal{O}(W)$ space where $W \approx N/2$ in the worst case.

---

## 11. Common Mistakes

1.  **Wrong Traversal Order**: Forgetting that Left View uses standard preorder (left first) and Right View uses reverse preorder (right first).
2.  **Using BFS space unnecessarily**: Implementing BFS when the interviewer specifically asks to optimize the space complexity to $O(H)$.
3.  **Missing `level == view.size()` condition**: Storing all nodes instead of just the first one seen per level.

---

## 12. Edge Cases

| Edge Case | Input Tree | Left View | Right View | Importance |
| :--- | :--- | :--- | :--- | :--- |
| **Empty Tree** | `nullptr` | `[]` | `[]` | Return early without error. |
| **Single Node** | `[1]` | `[1]` | `[1]` | Verify base level size check. |
| **Skewed Left** | `1 -> 2 -> 3` | `[1, 2, 3]` | `[1, 2, 3]` | Left side is also the right side. |
| **V-Shaped Tree**| `1` with children `2 (L)` and `3 (R)`. `2` has left child `4`. `3` has right child `5`. | `[1, 2, 4]` | `[1, 3, 5]` | Left/Right sides diverge. |

---

## 13. Interview Explanation

"To solve the Left or Right View of a binary tree, we can use an optimized Depth First Search (DFS) with level-tracking. 

For the **Right View**, we want the rightmost node at each level. We perform a reverse preorder traversal where we visit the Root, then the Right child, then the Left child. We pass down a `level` variable. If the current `level` matches the size of our output vector, it means we are visiting this level for the first time. Since we go right first, this first node is guaranteed to be the rightmost node. We add it to our list.

For the **Left View**, the logic is identical, except we visit the Left child before the Right child. This ensures the leftmost node is the first one processed at each level.

This DFS approach is optimal as it visits every node once, yielding a time complexity of $O(N)$. Its space complexity is $O(H)$ for the recursion stack, which is superior to the $O(W)$ space complexity of BFS."

---

## 14. Follow-up Questions

1.  **Q**: Can we solve this iteratively using DFS?
    *   *A*: Yes, using a stack that stores pairs of `(TreeNode*, level)` and simulating the recursive calls.
2.  **Q**: What if we are asked to return the sum of nodes in each view?
    *   *A*: We would run the same algorithm, collect the values, and then sum the resulting list.

---

## 15. Variations

1.  **Deepest Node of a Binary Tree**:
    *   *Solution*: The last element of the Right View (or Left View) is the deepest node.
2.  **Find Largest Value in Each Tree Row**:
    *   *Solution*: Similar level-based tracking, but instead of taking the first node, track and update the maximum value for each level using a map or vector.

---

## 16. Revision Notes

*   **Size Trick**: `level == view.size()` controls level-based entry.
*   **Left View Order**: Root $\rightarrow$ Left $\rightarrow$ Right.
*   **Right View Order**: Root $\rightarrow$ Right $\rightarrow$ Left.
*   **Complexity**: $O(N)$ time, $O(H)$ space.

---

## 17. Memorization Version

```cpp
void rightView(TreeNode* root, int lvl, vector<int>& res) {
    if (!root) return;
    if (lvl == res.size()) res.push_back(root->val);
    rightView(root->right, lvl + 1, res);
    rightView(root->left, lvl + 1, res);
}
void leftView(TreeNode* root, int lvl, vector<int>& res) {
    if (!root) return;
    if (lvl == res.size()) res.push_back(root->val);
    leftView(root->left, lvl + 1, res);
    leftView(root->right, lvl + 1, res);
}
```
*   **Mnemonic**: DFS, match `lvl == res.size()`, Right View recurses Right first, Left View recurses Left first.
*   **Complexity**: $O(N)$ time, $O(H)$ space.
