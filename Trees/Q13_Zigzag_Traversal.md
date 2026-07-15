# Zigzag Level Order Traversal

## 1. Problem Statement

Given a Binary Tree, return the zigzag level order traversal of its nodes' values. (i.e., from left to right, then right to left for the next level and alternate between).

*   **Input**:
    *   A pointer to the root of the binary tree: `TreeNode* root`.
*   **Output**:
    *   A 2D vector of integers: `vector<vector<int>>` where each inner vector represents the values of the nodes at that level, ordered in a zigzag pattern.
*   **Constraints**:
    *   $0 \le N \le 2000$ (Number of nodes in the tree)
    *   $-100 \le \text{Node value} \le 100$
*   **Observations**:
    *   Level 0 (root) is traversed Left-to-Right.
    *   Level 1 is traversed Right-to-Left.
    *   Level 2 is traversed Left-to-Right, and so on.
    *   This is a variant of the standard Level Order Traversal (BFS).

---

## 2. Interview Clarification

1.  **Q**: What should we return if the tree is empty?
    *   *A*: Return an empty 2D vector `std::vector<std::vector<int>>{}`.
2.  **Q**: Do we modify the actual pointers in the tree or just return the values?
    *   *A*: Just return the values in the specified order. Do not modify the tree structure.
3.  **Q**: Is there a preference between using a queue with a reverse step vs. using multiple stacks?
    *   *A*: You can use either; however, an optimal $O(N)$ solution that avoids explicit reversals is preferred.

---

## 3. Thinking Process

*   **First Instinct**:
    *   Perform a standard BFS using a queue.
    *   For each level, collect node values into a vector.
    *   If the level index is odd, reverse the level vector before adding it to the final result.
    *   While simple, reversing a vector takes additional $O(\text{level size})$ time, which can be optimized.

*   **Optimization Instincts**:
    *   **Option 1: Queue with Insertion at Specific Indices**:
        *   Instead of reversing, pre-allocate the vector for the level: `vector<int> level(size)`.
        *   If traversing Left-to-Right, insert values from index `0` to `size-1`.
        *   If traversing Right-to-Left, insert values from index `size-1` down to `0`.
        *   This avoids vector reversal overhead.
    *   **Option 2: Two Stacks**:
        *   Use two stacks: `s1` (for Left-to-Right) and `s2` (for Right-to-Left).
        *   For `s1` (L to R): pop a node and push its children (left then right) onto `s2`.
        *   For `s2` (R to L): pop a node and push its children (right then left) onto `s1`.
        *   This naturally reverses the traversal order without post-processing.

---

## 4. Pattern Recognition

*   **Topic**: Binary Tree Traversal.
*   **Pattern**: Breadth First Search (BFS) / Level Order Traversal.
*   **Why**: We need to process nodes level by level. Any problem requiring level-based grouping is a BFS/Queue candidate.

---

## 5. Brute Force Solution

*   **Algorithm**:
    *   Standard BFS level-order traversal using a queue.
    *   At the end of each level, check a boolean flag `leftToRight`. If it is `false`, reverse the collected vector using `std::reverse`.
    *   Flip the flag after each level.
*   **Dry Run**:
    *   Level 0: `[1]`, flag `true` $\rightarrow$ Result: `[[1]]`
    *   Level 1: `[2, 3]`, flag `false` $\rightarrow$ Reverse to `[3, 2]` $\rightarrow$ Result: `[[1], [3, 2]]`
*   **Complexity**:
    *   **Time Complexity**: $O(N)$ total, but does $O(W)$ operations for reverses where $W$ is the width of each level.
    *   **Space Complexity**: $O(N)$ total space (for queue and output).
*   **Pros/Cons**:
    *   *Pros*: Extremely simple to code.
    *   *Cons*: Extra overhead from copying/reversing vectors.

### Brute Force C++17 Code
```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <algorithm>

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

std::vector<std::vector<int>> zigzagLevelOrderBrute(TreeNode* root) {
    if (!root) return {};
    std::vector<std::vector<int>> result;
    std::queue<TreeNode*> q;
    q.push(root);
    bool leftToRight = true;

    while (!q.empty()) {
        int size = q.size();
        std::vector<int> level;
        for (int i = 0; i < size; ++i) {
            TreeNode* node = q.front();
            q.pop();
            level.push_back(node->val);
            if (node->left) q.push(node->left);
            if (node->right) q.push(node->right);
        }
        if (!leftToRight) {
            std::reverse(level.begin(), level.end());
        }
        result.push_back(level);
        leftToRight = !leftToRight;
    }
    return result;
}
```

---

## 6. Better Solution

*   *Note*: The standard optimization of writing elements directly into their final positions of a pre-sized level vector is highly clean and avoids the reversal step entirely. We will present this queue-based direct indexing approach as the optimal BFS method and also cover the two stacks method.

---

## 7. Optimal Solution

We present the **Queue with Direct Indexing** as it is highly readable and standard.

### Algorithm
1.  Initialize a queue `q` and push the `root` node.
2.  Initialize a boolean flag `leftToRight = true`.
3.  While `q` is not empty:
    *   Get the size of the current level `size = q.size()`.
    *   Create a vector `level` of size `size`.
    *   For `i = 0` to `size - 1`:
        *   Pop the front node `curr`.
        *   Determine the correct index to insert `curr->val`. If `leftToRight` is `true`, `index = i`. Else, `index = size - 1 - i`.
        *   Assign `level[index] = curr->val`.
        *   Push the left and right children of `curr` (if they exist) to `q`.
    *   Flip `leftToRight = !leftToRight`.
    *   Push `level` into the `result` vector.

### Optimal C++17 Code
```cpp
#include <iostream>
#include <vector>
#include <queue>

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
public:
    /**
     * Zigzag level order traversal using BFS with direct indexing.
     */
    std::vector<std::vector<int>> zigzagLevelOrder(TreeNode* root) {
        if (root == nullptr) {
            return {};
        }

        std::vector<std::vector<int>> result;
        std::queue<TreeNode*> q;
        q.push(root);
        
        bool leftToRight = true;

        while (!q.empty()) {
            int size = q.size();
            // Pre-allocate level vector to avoid push_back dynamic resizing
            std::vector<int> level(size);

            for (int i = 0; i < size; ++i) {
                TreeNode* curr = q.front();
                q.pop();

                // Determine position based on direction flag
                int index = leftToRight ? i : (size - 1 - i);
                level[index] = curr->val;

                // Push children to the queue
                if (curr->left != nullptr) {
                    q.push(curr->left);
                }
                if (curr->right != nullptr) {
                    q.push(curr->right);
                }
            }

            // Alternate direction for the next level
            leftToRight = !leftToRight;
            result.push_back(level);
        }

        return result;
    }
};
```

---

## 8. Code Walkthrough

1.  **Queue Initialization**: `queue<TreeNode*> q` handles the FIFO level traversal.
2.  **Direction Flag**: `bool leftToRight = true` tracks the current level's insertion order.
3.  **Pre-sized Vector**: `vector<int> level(size)` allows index-based writing instead of calling `push_back`.
4.  **Index Calculation**:
    *   `int index = leftToRight ? i : (size - 1 - i);`
    *   If Left-to-Right: values are filled sequentially from `0` to `size-1`.
    *   If Right-to-Left: values are filled backwards from `size-1` down to `0`.
5.  **Child Insertion**:
    *   Left child first, then right child. This order in the queue remains constant regardless of the traversal direction.

---

## 9. Dry Run

Consider the following tree:
```
      3
     / \
    9  20
      /  \
     15   7
```

*   **Initialization**: `q = [3]`, `leftToRight = true`, `result = []`

### Level 0 (`size = 1`):
*   `level` allocated: `size = 1`.
*   `i = 0`: Pop `3`.
    *   `leftToRight = true` $\rightarrow$ `index = i = 0`.
    *   `level[0] = 3`.
    *   Push children of `3`: `q = [9, 20]`.
*   End level: Flip `leftToRight = false`. `result = [[3]]`.

### Level 1 (`size = 2`):
*   `level` allocated: `size = 2`. `q = [9, 20]`.
*   `i = 0`: Pop `9`.
    *   `leftToRight = false` $\rightarrow$ `index = size - 1 - i = 2 - 1 - 0 = 1`.
    *   `level[1] = 9`.
    *   Push children of `9`: None. `q = [20]`.
*   `i = 1`: Pop `20`.
    *   `leftToRight = false` $\rightarrow$ `index = 2 - 1 - 1 = 0`.
    *   `level[0] = 20`.
    *   Push children of `20`: `q = [15, 7]`.
*   End level: Flip `leftToRight = true`. `result = [[3], [20, 9]]`.

### Level 2 (`size = 2`):
*   `level` allocated: `size = 2`. `q = [15, 7]`.
*   `i = 0`: Pop `15`.
    *   `leftToRight = true` $\rightarrow$ `index = 0`.
    *   `level[0] = 15`.
    *   Push children of `15`: None. `q = [7]`.
*   `i = 1`: Pop `7`.
    *   `leftToRight = true` $\rightarrow$ `index = 1`.
    *   `level[1] = 7`.
    *   Push children of `7`: None. `q = []`.
*   End level: Flip `leftToRight = false`. `result = [[3], [20, 9], [15, 7]]`.

---

## 10. Complexity Analysis

*   **Time Complexity**: $\mathcal{O}(N)$
    *   Every node in the tree is visited exactly once.
    *   Index assignments take $\mathcal{O}(1)$ time per node.
*   **Space Complexity**: $\mathcal{O}(W)$
    *   $W$ is the maximum width of the tree.
    *   In the worst case (a complete binary tree), the maximum number of nodes at the lowest level is $W \approx N/2$, so the queue takes $\mathcal{O}(N)$ space in the worst case.
    *   For a highly skewed tree, $W = 1$, requiring $\mathcal{O}(1)$ auxiliary space.

---

## 11. Common Mistakes

1.  **Flipping Children Insertion Order**: Do NOT change the order of adding left/right children to the queue based on `leftToRight`. The queue traversal order must remain standard; only the *insertion index* of values in the current level vector changes.
2.  **Off-by-One in Indexing**: Calculating `size - i` instead of `size - 1 - i` for right-to-left.
3.  **No Null Tree Check**: Forgetting to check if `root == nullptr` before pushing to queue.

---

## 12. Edge Cases

| Edge Case | Input Tree | Expected Output | Importance |
| :--- | :--- | :--- | :--- |
| **Empty Tree** | `nullptr` | `[]` | Avoid null pointer exception on queue access. |
| **Single Node** | `[1]` | `[[1]]` | Base case validation. |
| **Left Skewed Tree** | `1 -> 2 -> 3` | `[[1], [2], [3]]` | Test correct alternating flag behavior on $W=1$ levels. |
| **Perfect Tree** | `1` with children `2, 3`, grandchildren `4, 5, 6, 7` | `[[1], [3, 2], [4, 5, 6, 7]]` | Verifies alternating patterns across multiple levels. |

---

## 13. Interview Explanation

"To perform a zigzag level order traversal, we can use a Breadth-First Search (BFS) approach. We start by placing the root in a queue. We track the direction of traversal for each level using a boolean flag, `leftToRight`, which starts as `true`. 

For each level, we determine the number of nodes at that level. We allocate a vector of that exact size. Instead of appending elements and reversing them later, we calculate the insertion index directly. If the flag is `true`, we write elements from left to right. If `false`, we write them from right to left (filling the vector from the back). We push the children of each node into the queue in the standard left-to-right order. After processing each level, we toggle the flag. This gives us $O(N)$ time and space complexity with zero array-reversal overhead."

---

## 14. Follow-up Questions

1.  **Q**: Can we solve this using DFS?
    *   *A*: Yes. We can pass the level depth as a parameter. In our DFS helper, if `level == result.size()`, we add a new vector. If the level is even, we append at the end; if odd, we insert at the beginning (using `deque` or `vector::insert`). However, DFS uses $O(H)$ stack space and $O(N)$ list-insertion overhead, making BFS generally cleaner for level-order problems.
2.  **Q**: How would you implement this using the two-stack method?
    *   *A*: We maintain two stacks, `currentLevel` and `nextLevel`. We push root to `currentLevel`. While either stack is not empty: pop from `currentLevel`, print/save, and push its children to `nextLevel` (left then right for L-to-R; right then left for R-to-L). Swap the stacks when `currentLevel` becomes empty.

---

## 15. Variations

1.  **Bottom-Up Level Order Traversal**: Traverse levels from bottom to top.
    *   *Solution change*: Perform standard level-order BFS, then reverse the outer `result` vector (or push to a stack / insert at index 0).
2.  **Level Order Traversal II (Return only odd/even levels)**:
    *   *Solution change*: Track level counts and omit levels that are not needed.

---

## 16. Revision Notes

*   **Direction Flag**: `leftToRight` toggled after each level.
*   **Direct Indexing Trick**: `index = leftToRight ? i : (size - 1 - i)`.
*   **Time/Space**: $O(N)$ time, $O(W)$ space.

---

## 17. Memorization Version

```cpp
vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
    if (!root) return {};
    vector<vector<int>> res;
    queue<TreeNode*> q;
    q.push(root);
    bool lToR = true;
    while (!q.empty()) {
        int sz = q.size();
        vector<int> level(sz);
        for (int i = 0; i < sz; ++i) {
            TreeNode* curr = q.front(); q.pop();
            int idx = lToR ? i : (sz - 1 - i);
            level[idx] = curr->val;
            if (curr->left) q.push(curr->left);
            if (curr->right) q.push(curr->right);
        }
        res.push_back(level);
        lToR = !lToR;
    }
    return res;
}
```
*   **Mnemonic**: BFS, pre-allocated level vector, direct indexing using `lToR ? i : sz - 1 - i`, toggle `lToR`.
