# Height / Depth of a Binary Tree

## 1. Problem Statement

Given the root of a binary tree, find its **maximum depth / height**. 

The **maximum depth** (or height) is the number of nodes along the longest path from the root node down to the farthest leaf node.
*Note: A tree with only the root node has a height of 1. An empty tree has a height of 0.*

### Input
- `root`: A pointer to the root node of a binary tree.

### Output
- An integer representing the maximum depth of the binary tree.

### Constraints
- The number of nodes in the tree is in the range `[0, 10^4]`.
- Node values: `-100 <= Node.val <= 100`.

### Observations
- The height of a binary tree is defined recursively: it is $1$ (for the current node) plus the maximum of the heights of its left and right subtrees.
- Depth-First Search (DFS) provides a clean recursive bottom-up solution.
- Breadth-First Search (BFS) provides an iterative solution by counting the total number of levels.

---

## 2. Interview Clarification

Before starting, clarify:
1. **Definition of Height/Depth**: "Is the height defined by the number of nodes or the number of edges on the longest path?"
   - *Interviewer response:* Count the number of nodes. So a single root node has a height of 1, and an empty tree has a height of 0.
2. **Empty Tree**: "Should we return 0 for a null root?"
   - *Interviewer response:* Yes.
3. **Iterative Requirement**: "Should I provide both recursive and iterative solutions?"
   - *Interviewer response:* Yes, please provide both.

---

## 3. Thinking Process

### First Instinct (Recursive Bottom-Up / Postorder DFS)
- To find the height of the root, we must first know the height of the left and right subtrees.
- Let's define the recurrence relation:
  $$\text{height}(\text{node}) = 1 + \max(\text{height}(\text{node.left}), \text{height}(\text{node.right}))$$
- Base case: If the node is `nullptr`, its height is `0`.
- This is a postorder traversal because we must evaluate child subtrees before calculating the parent's height.

### Transition to Iterative (Level-by-Level BFS)
- To do this iteratively without a call stack, we can traverse level-by-level using a BFS (Breadth-First Search) queue.
- Every time we finish processing a complete level, we increment our depth counter.
- This is identical to the level order traversal template, but instead of storing node values, we just count the levels.

---

## 4. Pattern Recognition

- **Category**: Binary Tree Properties.
- **Pattern**: Depth-First Search (Bottom-Up) / Level-by-level BFS.
- **Why**: Evaluating tree height requires visiting all paths. Bottom-up recursive DFS is the most direct way to propagate maximum heights upward. BFS is the most natural way to count physical levels.

---

## 5. Brute Force Solution

*Note: Since the recursive DFS is the most straightforward representation, we will present the Iterative BFS as the "Better" solution and the Recursive DFS as the "Optimal" solution due to its minimal code overhead and performance.*

---

## 6. Better Solution (Iterative BFS)

We count the levels of the tree using a queue.

### Algorithm
1. If `root` is null, return 0.
2. Initialize `depth = 0` and queue `q` containing `root`.
3. While `q` is not empty:
   - Increment `depth`.
   - Determine `levelSize = q.size()`.
   - Loop `levelSize` times:
     - Pop `curr`.
     - Push `curr->left` (if not null) to `q`.
     - Push `curr->right` (if not null) to `q`.
4. Return `depth`.

### Complexity Analysis
- **Time Complexity**: $\mathcal{O}(N)$ since we visit every node once.
- **Space Complexity**: $\mathcal{O}(W)$ where $W$ is the maximum width of the tree. In the worst case (perfect tree), the queue holds $N/2$ nodes, which is $\mathcal{O}(N)$ space.

### C++17 Implementation
```cpp
#include <queue>

struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
};

class SolutionIterative {
public:
    int maxDepth(TreeNode* root) {
        if (root == nullptr) {
            return 0;
        }
        
        std::queue<TreeNode*> q;
        q.push(root);
        int depth = 0;
        
        while (!q.empty()) {
            int levelSize = q.size();
            depth++; // We are starting a new level
            
            for (int i = 0; i < levelSize; ++i) {
                TreeNode* curr = q.front();
                q.pop();
                
                if (curr->left != nullptr) {
                    q.push(curr->left);
                }
                if (curr->right != nullptr) {
                    q.push(curr->right);
                }
            }
        }
        return depth;
    }
};
```

---

## 7. Optimal Solution (Recursive DFS)

The recursive DFS is highly optimized, has no queue overhead, and is the cleanest way to solve this problem.

### Algorithm
1. Base case: If `root` is `nullptr`, return `0`.
2. Find left height: `leftHeight = maxDepth(root->left)`.
3. Find right height: `rightHeight = maxDepth(root->right)`.
4. Return `1 + max(leftHeight, rightHeight)`.

---

### Why this is Optimal
It requires no auxiliary structures like queues, saving allocation overhead. It runs in linear time $\mathcal{O}(N)$ and uses stack space proportional to the height of the tree $\mathcal{O}(H)$.

---

### Beautiful C++17 Code
```cpp
#include <algorithm>

class SolutionOptimal {
public:
    int maxDepth(TreeNode* root) {
        // Base case: An empty tree has a height of 0
        if (root == nullptr) {
            return 0;
        }
        
        // Postorder DFS: Compute height of subtrees first
        int leftDepth = maxDepth(root->left);
        int rightDepth = maxDepth(root->right);
        
        // Height of current node is 1 + max height of subtrees
        return 1 + std::max(leftDepth, rightDepth);
    }
};
```

---

## 8. Code Walkthrough

- `if (root == nullptr) return 0;`: Handles the base case, stopping recursion when reaching a leaf node's child.
- `int leftDepth = maxDepth(root->left);`: Recurses down to find the height of the left side.
- `int rightDepth = maxDepth(root->right);`: Recurses down to find the height of the right side.
- `return 1 + std::max(leftDepth, rightDepth);`: Adds the current node to the maximum height found from either subtree.

---

## 9. Dry Run

Let's dry run the recursive optimal solution with this tree:
```
      1
     / \
    2   3
       /
      4
```

- **Execution Trace**:
  - `maxDepth(1)`:
    - calls `maxDepth(2)`:
      - calls `maxDepth(2->left)` $\to$ returns 0.
      - calls `maxDepth(2->right)` $\to$ returns 0.
      - returns `1 + max(0, 0) = 1`.
    - calls `maxDepth(3)`:
      - calls `maxDepth(4)`:
        - calls `maxDepth(4->left)` $\to$ returns 0.
        - calls `maxDepth(4->right)` $\to$ returns 0.
        - returns `1 + max(0, 0) = 1`.
      - calls `maxDepth(3->right)` $\to$ returns 0.
      - returns `1 + max(1, 0) = 2`.
    - returns `1 + max(1, 2) = 3`.
- Final Height: `3`.

---

## 10. Complexity Analysis

| Metric | Recursive DFS (Optimal) | Iterative BFS |
|:---|:---|:---|
| **Time Complexity (Best)** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Time Complexity (Avg)** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Time Complexity (Worst)**| $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Space Complexity (Worst)**| $\mathcal{O}(N)$ (skewed tree stack) | $\mathcal{O}(N)$ (perfect tree queue size) |
| **Space Complexity (Best)** | $\mathcal{O}(\log N)$ (balanced tree)| $\mathcal{O}(1)$ (skewed tree) |

---

## 11. Common Mistakes

1. **Incorrect Base Case**: Setting the base case to return `1` for `nullptr` instead of `0`. This makes the calculated height of a leaf node `2` instead of `1`.
2. **Missing `1 +`**: Returning `std::max(leftDepth, rightDepth)` without adding `1` for the current node. This results in the tree height always evaluating to `0`.
3. **Redundant Recalculations**: Writing `return 1 + std::max(maxDepth(root->left), maxDepth(root->right));` directly inside the return statement. In some languages or compilers, this can cause duplicate calls and exponential time complexity. It is always safer to compute them into variables first.

---

## 12. Edge Cases

| Edge Case | Input Tree | Why it matters | How Code Handles It |
|:---|:---|:---|:---|
| **Null Tree** | `nullptr` | No nodes present | Returns `0` via the base case. |
| **Single Node** | `[1]` | Minimum valid tree | Returns `1 + max(0,0) = 1`. |
| **Left Skewed** | `3 -> 2 -> 1` | Stack depth matches height $N$ | Handled correctly. Uses $\mathcal{O}(N)$ stack memory. |
| **Right Skewed**| `1 -> 2 -> 3` | Stack depth matches height $N$ | Handled correctly. Uses $\mathcal{O}(N)$ stack memory. |

---

## 13. Interview Explanation

"To find the maximum depth or height of a binary tree, we can use a bottom-up recursive approach. 

The height of any node is equal to 1 plus the maximum height of its left and right subtrees. 

By defining our base case where a null node has a height of 0, we can recursively calculate the heights of the left and right children and return `1 + max(leftHeight, rightHeight)`. This represents a postorder depth-first search running in $\mathcal{O}(N)$ time and $\mathcal{O}(H)$ stack space. 

If we wanted to avoid recursion due to stack overflow concerns on very deep trees, we could also use a level-order BFS traversal, incrementing a depth counter each time we finish processing a level of the tree. This would also run in $\mathcal{O}(N)$ time, but would use $\mathcal{O}(W)$ auxiliary space, where $W$ is the maximum width of the tree."

---

## 14. Follow-up Questions

1. **Q**: Can you find the minimum depth of a binary tree?
   - **A**: Yes, but the logic changes slightly. We must find the shortest path to a *leaf* node. If a node has only one child, we cannot return `1 + min(0, childHeight)` (as that would treat the non-leaf as a leaf). We must recursively traverse only the non-null child paths.
2. **Q**: What happens to space complexity in a balanced tree vs. skewed tree for DFS vs. BFS?
   - **A**: In a balanced tree, DFS uses $\mathcal{O}(\log N)$ space while BFS uses $\mathcal{O}(N)$ space. In a skewed tree, DFS uses $\mathcal{O}(N)$ space while BFS uses $\mathcal{O}(1)$ space.

---

## 15. Variations

1. **Minimum Depth of Binary Tree**: Finding the number of nodes along the shortest path from the root node down to the nearest leaf node.
2. **Balanced Binary Tree**: Checking if the tree heights of left and right subtrees at every node differ by at most 1 (uses this height calculation directly).

---

## 16. Revision Notes

- **Height Rule**: `1 + max(leftHeight, rightHeight)`.
- **Base Case**: `nullptr` returns `0`.
- **DFS space**: $\mathcal{O}(H)$.
- **BFS space**: $\mathcal{O}(W)$.

---

## 17. Memorization Version

```cpp
// Recursive Height: 1 + max of child heights.
int maxDepth(TreeNode* root) {
    if (!root) return 0;
    return 1 + max(maxDepth(root->left), maxDepth(root->right));
}
```
