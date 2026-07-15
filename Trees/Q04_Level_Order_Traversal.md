# Level Order Traversal of Binary Tree

## 1. Problem Statement

Given the root of a binary tree, return the **level order traversal** of its nodes' values. (i.e., from left to right, level by level).

### Input
- `root`: A pointer to the root node of a binary tree.

### Output
We will cover two common output formats:
1. **Flat List**: A 1D vector of integers: `[level0, level1, ...]`.
2. **List of Lists (Grouped)**: A 2D vector of integers: `[[level0], [level1], ...]`.

### Constraints
- The number of nodes in the tree is in the range `[0, 2000]` (or up to `10^5`).
- Node values: `-1000 <= Node.val <= 1000`.

### Observations
- Level order traversal is equivalent to **Breadth-First Search (BFS)** on a tree.
- Unlike Depth-First Search (DFS) which uses a stack, BFS uses a **queue** (FIFO) to visit nodes in horizontal bands.
- The maximum size of the queue depends on the **maximum width** of the binary tree.

---

## 2. Interview Clarification

Before starting, clarify:
1. **Output Format**: "Do you want the output as a single flat array of values, or grouped level-by-level in a 2D array?"
   - *Interviewer response:* Provide both, but make sure the primary solution returns a 2D array `vector<vector<int>>` representing each level separately.
2. **Empty Tree**: "How should we handle an empty tree?"
   - *Interviewer response:* Return an empty 2D vector.
3. **Queue Limit**: "Should we assume the tree can be extremely wide, affecting memory limits?"
   - *Interviewer response:* Yes, keep track of queue memory usage.

---

## 3. Thinking Process

### First Instinct (BFS using Queue)
- To traverse level-by-level, we must process nodes in the order they are discovered (FIFO). A `std::queue<TreeNode*>` is the perfect structure.
- **Basic BFS**:
  - Push root to queue.
  - While queue is not empty:
    - Pop front node.
    - Visit it (add to output).
    - Push its left child, then its right child.
- This creates the **Flat List** output.

### Transition to Level-by-Level (Grouped List)
- To group nodes level-by-level, we need to know where each level starts and ends.
- At the beginning of each level loop, the queue contains **exactly** the nodes of the current level.
- By storing the size of the queue at the start of the level loop (`int levelSize = q.size()`), we can run an inner loop exactly `levelSize` times. This processes all nodes of the current level, and pushes all of their children (the next level) into the queue.

---

## 4. Pattern Recognition

- **Category**: Binary Tree BFS.
- **Pattern**: Queue-based Level Processing.
- **Why**: Breadth-First Search requires visiting nodes closest to the root first. The FIFO queue preserves this ordering. By reading the queue size dynamically, we partition the queue elements into discrete levels.

---

## 5. Brute Force Solution (Recursive DFS with Level Tracking)

We can actually perform level order traversal using DFS by passing the depth/level as an argument.

### Algorithm
1. Initialize an empty 2D vector `result`.
2. Define a helper function `dfs(TreeNode* node, int level, vector<vector<int>>& result)`.
3. If `node` is null, return.
4. If `level == result.size()`, it means we are visiting this level for the first time. Add a new empty vector `result.push_back({})`.
5. Add `node->val` to `result[level]`.
6. Recurse left: `dfs(node->left, level + 1, result)`.
7. Recurse right: `dfs(node->right, level + 1, result)`.

### Complexity Analysis
- **Time Complexity**: $\mathcal{O}(N)$ since we visit every node once.
- **Space Complexity**: $\mathcal{O}(H)$ stack space, where $H$ is the height of the tree. In the worst case (skewed tree), this is $\mathcal{O}(N)$.

### C++17 Implementation
```cpp
#include <vector>

struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
};

class SolutionDFS {
private:
    void dfs(TreeNode* node, int level, std::vector<std::vector<int>>& result) {
        if (node == nullptr) return;
        
        // If we reach a new level, create a new sub-vector
        if (level == result.size()) {
            result.push_back({});
        }
        
        result[level].push_back(node->val);
        dfs(node->left, level + 1, result);
        dfs(node->right, level + 1, result);
    }
public:
    std::vector<std::vector<int>> levelOrder(TreeNode* root) {
        std::vector<std::vector<int>> result;
        dfs(root, 0, result);
        return result;
    }
};
```

---

## 6. Better Solution (Flat List Output)

If the interviewer only needs a flat list of node values, we do a basic queue traversal without tracking level size.

### C++17 Implementation
```cpp
#include <vector>
#include <queue>

class SolutionFlat {
public:
    std::vector<int> levelOrderFlat(TreeNode* root) {
        std::vector<int> result;
        if (root == nullptr) return result;
        
        std::queue<TreeNode*> q;
        q.push(root);
        
        while (!q.empty()) {
            TreeNode* curr = q.front();
            q.pop();
            
            result.push_back(curr->val);
            
            if (curr->left != nullptr) q.push(curr->left);
            if (curr->right != nullptr) q.push(curr->right);
        }
        return result;
    }
};
```

---

## 7. Optimal Solution (Grouped Level Order Traversal)

The optimal standard BFS using a queue to process level-by-level.

### Algorithm
1. If `root` is null, return an empty 2D vector.
2. Initialize a queue `q` of `TreeNode*` and push `root` into it.
3. While `q` is not empty:
   - Determine `levelSize = q.size()`.
   - Create a temporary vector `currentLevel` to store the values at the current depth.
   - Loop `levelSize` times:
     - Pop `curr` from queue.
     - Add `curr->val` to `currentLevel`.
     - Push `curr->left` (if not null) to `q`.
     - Push `curr->right` (if not null) to `q`.
   - Add `currentLevel` to `result`.

---

### Why this is Optimal
It processes the tree strictly level by level in linear time ($\mathcal{O}(N)$). The space complexity is $\mathcal{O}(W)$ where $W$ is the maximum width of the tree. This is the optimal queue size for BFS on a tree.

---

### Beautiful C++17 Code
```cpp
#include <vector>
#include <queue>

class SolutionOptimal {
public:
    std::vector<std::vector<int>> levelOrder(TreeNode* root) {
        std::vector<std::vector<int>> result;
        if (root == nullptr) {
            return result;
        }
        
        std::queue<TreeNode*> q;
        q.push(root);
        
        while (!q.empty()) {
            int levelSize = q.size(); // Number of nodes at current level
            std::vector<int> currentLevel;
            currentLevel.reserve(levelSize); // Pre-allocate for optimization
            
            for (int i = 0; i < levelSize; ++i) {
                TreeNode* curr = q.front();
                q.pop();
                
                currentLevel.push_back(curr->val);
                
                // Add children of current node to queue for the next level
                if (curr->left != nullptr) {
                    q.push(curr->left);
                }
                if (curr->right != nullptr) {
                    q.push(curr->right);
                }
            }
            
            result.push_back(currentLevel);
        }
        
        return result;
    }
};
```

---

## 8. Code Walkthrough

- `int levelSize = q.size();`: Captured before the inner loop begins. The queue contains exactly the nodes of the current level.
- `currentLevel.reserve(levelSize)`: Avoids multiple dynamic reallocations of the vector.
- `for (int i = 0; i < levelSize; ++i)`: Processes exactly the number of nodes at this level.
- `q.push(curr->left)` and `q.push(curr->right)`: Queue size increases, but since we captured `levelSize` beforehand, these new nodes will not be processed in the current level's loop. They are deferred to the next level's loop.

---

## 9. Dry Run

Let's dry run the optimal solution with this tree:
```
      3
     / \
    9  20
      /  \
     15   7
```

| Loop Iteration | Queue at start of level | `levelSize` | Nodes Popped (and visited) | Next level elements pushed | `currentLevel` vector | Result Vector |
|:---:|:---|:---:|:---|:---|:---|:---|
| **Init** | `[3]` | — | — | — | — | `[]` |
| **Level 0** | `[3]` | `1` | `3` | `9`, `20` | `[3]` | `[[3]]` |
| **Level 1** | `[9, 20]` | `2` | `9`, then `20` | `15`, `7` (from 20) | `[9, 20]` | `[[3], [9, 20]]` |
| **Level 2** | `[15, 7]` | `2` | `15`, then `7` | None | `[15, 7]` | `[[3], [9, 20], [15, 7]]` |
| **End** | `[]` | — | Queue empty, exit loop | — | — | — |

---

## 10. Complexity Analysis

| Metric | DFS-based (Brute) | BFS-based (Optimal) |
|:---|:---|:---|
| **Time Complexity** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Space Complexity (Worst)**| $\mathcal{O}(N)$ (skewed tree) | $\mathcal{O}(W)$ (complete tree: $W \approx N/2$) |
| **Space Complexity (Best)** | $\mathcal{O}(\log N)$ (balanced tree)| $\mathcal{O}(1)$ (skewed list tree) |

### Why is Space Complexity $\mathcal{O}(W)$?
In BFS, the maximum memory is occupied by the queue which holds nodes at the widest level of the tree. For a perfect binary tree, the leaf level contains $N/2$ nodes. Thus, the worst-case space is $\mathcal{O}(W)$ where $W$ is the maximum width of the tree.

---

## 11. Common Mistakes

1. **Recalculating Queue Size**: Writing `for (int i = 0; i < q.size(); ++i)` directly in the loop. Since `q.push(...)` increases the size of the queue dynamically, this loop will process newly added children in the same level, leading to incorrect levels.
2. **Pushing `nullptr`**: Not checking if `left` or `right` are null before pushing to the queue. This causes crash/null-dereferences in subsequent pops.
3. **Improper Empty Tree Handling**: Not checking `root == nullptr`, causing undefined behavior when popping/checking front from an empty queue.

---

## 12. Edge Cases

| Edge Case | Input Tree | Why it matters | How Code Handles It |
|:---|:---|:---|:---|
| **Empty Tree** | `nullptr` | Can cause null-dereference | Checked via `if (root == nullptr) return result;`. |
| **Single Node** | `[1]` | Minimal structure | Loop runs once, outputs `[[1]]`. |
| **Skewed Tree (Left)**| `3 -> 2 -> 1` | Queue size is always 1 | Space is optimal $\mathcal{O}(1)$. |
| **Complete Binary Tree**| Perfect tree | Max width leaf level | Correctly scales queue size to $N/2$ nodes. |

---

## 13. Interview Explanation

"Level order traversal of a binary tree is equivalent to Breadth-First Search (BFS). We traverse the tree level-by-level, from left to right.

To implement this, we use a **FIFO queue**. 

To group the nodes level-by-level, we capture the number of nodes at the current level using `q.size()` before we start processing that level. We run an inner loop for exactly that count, popping each node, recording its value, and pushing its non-null children onto the queue. Because we captured the count beforehand, any children pushed during this step are deferred to the next level. This ensures we process levels correctly, giving us a time complexity of $\mathcal{O}(N)$ and a space complexity of $\mathcal{O}(W)$, where $W$ is the maximum width of the tree."

---

## 14. Follow-up Questions

1. **Q**: How would you modify this to output levels in reverse order (bottom-up)?
   - **A**: Perform standard level order traversal and reverse the outer `result` vector at the end, or insert each level at the beginning of the result list (though vector insertion at the front is $\mathcal{O}(N)$).
2. **Q**: How would you implement Zigzag Level Order Traversal?
   - **A**: Use a flag to reverse the `currentLevel` vector before pushing it to `result` for alternate levels, or use a deque to construct each level.

---

## 15. Variations

1. **Binary Tree Zigzag Level Order Traversal**: Traverse left-to-right on one level, then right-to-left on the next.
2. **Binary Tree Right Side View**: Return the rightmost node of each level. You can solve this by performing level order traversal and only saving the last element of each level.

---

## 16. Revision Notes

- **Level Order**: Breadth-First Search (BFS).
- **Core Tool**: `std::queue<TreeNode*>`.
- **Level Separator Trick**: Store `q.size()` before processing the current level.
- **Space complexity**: $\mathcal{O}(W)$ where $W$ is maximum width of the tree.

---

## 17. Memorization Version

```cpp
// Grouped Level Order: Capture size, process size elements, push children.
vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> res; if (!root) return res;
    queue<TreeNode*> q; q.push(root);
    while (!q.empty()) {
        int sz = q.size(); vector<int> lvl;
        for (int i = 0; i < sz; i++) {
            TreeNode* curr = q.front(); q.pop();
            lvl.push_back(curr->val);
            if (curr->left) q.push(curr->left);
            if (curr->right) q.push(curr->right);
        }
        res.push_back(lvl);
    }
    return res;
}
```
