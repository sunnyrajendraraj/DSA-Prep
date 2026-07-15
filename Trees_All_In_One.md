# Trees — Complete Topic Reference

## Q01 Inorder Traversal

### 1. Problem Explanation

Given the root of a binary tree, return the **inorder traversal** of its nodes' values. Inorder traversal visit order is:
1. Traverse the left subtree.
2. Visit the root.
3. Traverse the right subtree.

### Input
- `root`: A pointer to the root node of a binary tree.

### Output
- A vector of integers representing the inorder traversal of the binary tree.

### Constraints
- The number of nodes in the tree is in the range `[0, 100]` (or more generally up to `10^4` or `10^5` in competitive programming).
- Node values: `-100 <= Node.val <= 100` (or fits within a standard 32-bit signed integer).

### Observations
- Inorder traversal on a Binary Search Tree (BST) visits nodes in ascending sorted order.
- Since it is a depth-first traversal (DFS), the space complexity depends on the height of the tree due to recursion/call stack memory.


### 2. Brute Solution

The recursive approach is simple and direct. It represents the natural way of thinking about DFS traversals.

### Algorithm
1. Define a helper function `inorderHelper(TreeNode* node, vector<int>& result)`.
2. Base Case: If `node` is `nullptr`, return.
3. Recurse: `inorderHelper(node->left, result)`.
4. Visit: `result.push_back(node->val)`.
5. Recurse: `inorderHelper(node->right, result)`.

### Dry Run
Given Tree:
```
    1
     \
      2
     /
    3
```
- `inorderHelper(1)`:
  - left is null.
  - visit `1` -> `result = [1]`.
  - call `inorderHelper(2)`:
    - call `inorderHelper(3)`:
      - left is null.
      - visit `3` -> `result = [1, 3]`.
      - right is null. Returns to `2`.
    - visit `2` -> `result = [1, 3, 2]`.
    - right is null. Returns to `1`.
- Traversal completes. Output: `[1, 3, 2]`.

### Complexity Analysis
- **Time Complexity**: $\mathcal{O}(N)$, where $N$ is the number of nodes in the tree. We visit each node exactly once.
- **Space Complexity**: $\mathcal{O}(H)$, where $H$ is the height of the tree. This is for the recursive call stack. In the worst case (skewed tree), $H = N$. In the best/average case (balanced tree), $H = \log N$.

### Pros and Cons
- **Pros**: Extremely simple to read, write, and verify.
- **Cons**: Can cause Stack Overflow (segmentation fault) if the tree is highly skewed and very deep (e.g., $N > 10^5$).

### C++17 Implementation
```cpp
#include <vector>

// Definition for a binary tree node.
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
};

class SolutionRecursive {
private:
    void inorderHelper(TreeNode* root, std::vector<int>& result) {
        if (root == nullptr) {
            return;
        }
        // 1. Traverse Left
        inorderHelper(root->left, result);
        // 2. Visit Root
        result.push_back(root->val);
        // 3. Traverse Right
        inorderHelper(root->right, result);
    }
public:
    std::vector<int> inorderTraversal(TreeNode* root) {
        std::vector<int> result;
        inorderHelper(root, result);
        return result;
    }
};
```


### 3. Optimal Solution & Complexities

Morris Traversal is the optimal solution because it achieves $\mathcal{O}(N)$ time and $\mathcal{O}(1)$ auxiliary space. It does this by temporarily modifying the tree structure.

### Algorithm
1. Initialize `curr` to `root`.
2. While `curr != nullptr`:
   - If `curr` has no left child:
     - Visit `curr` (add to result).
     - Go right: `curr = curr->right`.
   - Else (if `curr` has a left child):
     - Find the **inorder predecessor** of `curr` (rightmost node in the left subtree).
     - If the predecessor's `right` pointer is null:
       - Thread it: Set predecessor's `right` pointer to `curr`.
       - Move `curr` to its left child: `curr = curr->left`.
     - Else (predecessor's `right` pointer is already pointing to `curr`):
       - Unthread it: Set predecessor's `right` pointer back to `nullptr`.
       - Visit `curr` (add to result).
       - Move `curr` to its right child: `curr = curr->right`.

### Why Morris Traversal is Optimal
Unlike other solutions, Morris traversal achieves the theoretical minimum space bound of $\mathcal{O}(1)$ auxiliary space (not counting the output vector). It does this by leveraging the unused `nullptr` right pointers of leaf nodes to create temporary return links.

### Beautiful C++17 Code
```cpp
#include <vector>

class SolutionMorris {
public:
    std::vector<int> inorderTraversal(TreeNode* root) {
        std::vector<int> result;
        TreeNode* curr = root;

        while (curr != nullptr) {
            if (curr->left == nullptr) {
                // If there is no left child, visit this node and go right
                result.push_back(curr->val);
                curr = curr->right;
            } else {
                // Find the inorder predecessor (rightmost node in left subtree)
                TreeNode* pred = curr->left;
                while (pred->right != nullptr && pred->right != curr) {
                    pred = pred->right;
                }

                if (pred->right == nullptr) {
                    // Create thread pointing to current node (ancestor)
                    pred->right = curr;
                    curr = curr->left;
                } else {
                    // Thread already exists, break it to restore tree structure
                    pred->right = nullptr;
                    result.push_back(curr->val);
                    curr = curr->right;
                }
            }
        }
        return result;
    }
};
```


#### Complexity Summary

| Metric | Recursive | Iterative (Stack) | Morris Traversal |
|:---|:---|:---|:---|
| **Time Complexity (Best)** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Time Complexity (Avg)** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Time Complexity (Worst)**| $\mathcal{O}(N)$ | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Space Complexity (Worst)**| $\mathcal{O}(N)$ (skewed tree) | $\mathcal{O}(N)$ (skewed tree) | $\mathcal{O}(1)$ auxiliary |
| **Space Complexity (Best)** | $\mathcal{O}(\log N)$ (balanced tree)| $\mathcal{O}(\log N)$ (balanced tree)| $\mathcal{O}(1)$ auxiliary |

### Why is Morris Traversal $\mathcal{O}(N)$ time?
Although we find the predecessor for nodes multiple times, no edge is traversed more than 3 times (once to find predecessor, once to establish thread, and once to break the thread). Thus, the time complexity remains strictly linear.


---

## Q02 Preorder Traversal

### 1. Problem Explanation

Given the root of a binary tree, return the **preorder traversal** of its nodes' values. Preorder traversal visit order is:
1. Visit the root.
2. Traverse the left subtree.
3. Traverse the right subtree.

### Input
- `root`: A pointer to the root node of a binary tree.

### Output
- A vector of integers representing the preorder traversal of the binary tree.

### Constraints
- The number of nodes in the tree is in the range `[0, 100]` (or more generally up to `10^4` or `10^5` in competitive programming).
- Node values: `-100 <= Node.val <= 100` (or fits within a standard 32-bit signed integer).

### Observations
- Preorder traversal processes the current node before traversing its subtrees.
- It is a natural choice when cloning or printing a tree since parents are processed/created before their children.
- Like all DFS variations, the auxiliary space is bound by the height of the tree.


### 2. Brute Solution

The recursive approach processes nodes by making implicit call-stack frames.

### Algorithm
1. Create a helper function `preorderHelper(TreeNode* node, vector<int>& result)`.
2. Base case: If `node` is `nullptr`, return.
3. Visit: `result.push_back(node->val)`.
4. Recurse left: `preorderHelper(node->left, result)`.
5. Recurse right: `preorderHelper(node->right, result)`.

### Dry Run
Given Tree:
```
    1
   / \
  2   3
```
- `preorderHelper(1)`:
  - Visit `1` $\to$ `result = [1]`.
  - Recurse left: calls `preorderHelper(2)`:
    - Visit `2` $\to$ `result = [1, 2]`.
    - Recurse left (null) and right (null). Returns to `1`.
  - Recurse right: calls `preorderHelper(3)`:
    - Visit `3` $\to$ `result = [1, 2, 3]`.
    - Recurse left/right (null). Returns.
- Complete. Output: `[1, 2, 3]`.

### Complexity Analysis
- **Time Complexity**: $\mathcal{O}(N)$ since we visit every node exactly once.
- **Space Complexity**: $\mathcal{O}(H)$, where $H$ is the tree height, due to the implicit recursion stack. In a skewed tree, $H = N$.

### Pros and Cons
- **Pros**: Short, simple, and self-documenting.
- **Cons**: Vulnerable to stack overflow on very deep trees.

### C++17 Implementation
```cpp
#include <vector>

// Definition for a binary tree node.
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
};

class SolutionRecursive {
private:
    void preorderHelper(TreeNode* root, std::vector<int>& result) {
        if (root == nullptr) {
            return;
        }
        // 1. Visit Root
        result.push_back(root->val);
        // 2. Traverse Left
        preorderHelper(root->left, result);
        // 3. Traverse Right
        preorderHelper(root->right, result);
    }
public:
    std::vector<int> preorderTraversal(TreeNode* root) {
        std::vector<int> result;
        preorderHelper(root, result);
        return result;
    }
};
```


### 3. Optimal Solution & Complexities

The iterative approach uses an explicit stack to achieve $\mathcal{O}(N)$ time and $\mathcal{O}(H)$ space, eliminating call stack limits.

### Algorithm
1. Initialize an empty stack and push `root` if it is not null.
2. While stack is not empty:
   - Pop the top node `curr`.
   - Visit `curr` (add `curr->val` to result).
   - If `curr->right` is not null, push `curr->right` to the stack.
   - If `curr->left` is not null, push `curr->left` to the stack.
3. Return the result vector.

### Why this is Optimal
This solution uses an explicit stack, preventing stack overflow on deeply nested trees while running in linear time. We push the right child first so that the left child is popped and processed first, matching the Root-Left-Right order.

### Beautiful C++17 Code
```cpp
#include <vector>
#include <stack>

class SolutionOptimal {
public:
    std::vector<int> preorderTraversal(TreeNode* root) {
        std::vector<int> result;
        if (root == nullptr) {
            return result;
        }
        
        std::stack<TreeNode*> s;
        s.push(root);
        
        while (!s.empty()) {
            TreeNode* curr = s.top();
            s.pop();
            
            // Visit current node
            result.push_back(curr->val);
            
            // Push right child first so left child is processed first (LIFO)
            if (curr->right != nullptr) {
                s.push(curr->right);
            }
            if (curr->left != nullptr) {
                s.push(curr->left);
            }
        }
        
        return result;
    }
};
```


#### Complexity Summary

| Metric | Recursive | Iterative (Optimal) |
|:---|:---|:---|
| **Time Complexity (Best)** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Time Complexity (Avg)** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Time Complexity (Worst)**| $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Space Complexity (Worst)**| $\mathcal{O}(N)$ (skewed tree) | $\mathcal{O}(N)$ (skewed tree) |
| **Space Complexity (Best)** | $\mathcal{O}(\log N)$ (balanced tree)| $\mathcal{O}(\log N)$ (balanced tree)|


---

## Q03 Postorder Traversal

### 1. Problem Explanation

Given the root of a binary tree, return the **postorder traversal** of its nodes' values. Postorder traversal visit order is:
1. Traverse the left subtree.
2. Traverse the right subtree.
3. Visit the root.

### Input
- `root`: A pointer to the root node of a binary tree.

### Output
- A vector of integers representing the postorder traversal of the binary tree.

### Constraints
- The number of nodes in the tree is in the range `[0, 100]` (or up to `10^5` in competitive programming).
- Node values: `-100 <= Node.val <= 100`.

### Observations
- In postorder traversal, a parent is visited **after** both of its subtrees have been fully traversed.
- Because of this property, postorder is the default choice for operations like **deleting a tree** (freeing child nodes before deleting the parent) or calculating attributes like tree height, size, and diameter where child info is required.


### 2. Brute Solution

### Algorithm
1. Helper: `postorderHelper(TreeNode* node, vector<int>& result)`.
2. Base case: If `node` is `nullptr`, return.
3. Recurse left: `postorderHelper(node->left, result)`.
4. Recurse right: `postorderHelper(node->right, result)`.
5. Visit: `result.push_back(node->val)`.

### Complexity Analysis
- **Time Complexity**: $\mathcal{O}(N)$ where $N$ is the number of nodes.
- **Space Complexity**: $\mathcal{O}(H)$ stack space, where $H$ is the height of the tree. Worst case is $\mathcal{O}(N)$.

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

class SolutionRecursive {
private:
    void postorderHelper(TreeNode* root, std::vector<int>& result) {
        if (root == nullptr) return;
        postorderHelper(root->left, result);
        postorderHelper(root->right, result);
        result.push_back(root->val);
    }
public:
    std::vector<int> postorderTraversal(TreeNode* root) {
        std::vector<int> result;
        postorderHelper(root, result);
        return result;
    }
};
```


### 3. Optimal Solution & Complexities

Using a single stack optimizes space usage by avoiding the storage of all $N$ elements in a second stack.

### Algorithm
1. Initialize an empty stack `s`, set pointer `curr = root` and `lastVisited = nullptr`.
2. Loop while `curr != nullptr` or stack is not empty:
   - If `curr != nullptr`, push `curr` to stack and move to `curr = curr->left` (go deep left).
   - Else:
     - Peek the top node: `TreeNode* peekNode = s.top()`.
     - If `peekNode->right` is not null and `lastVisited != peekNode->right`:
       - Move `curr = peekNode->right` to visit right subtree.
     - Else:
       - Visit `peekNode` (add to result).
       - Record `lastVisited = peekNode`.
       - Pop `peekNode` from stack.

### Why this is Optimal
This approach uses only a single stack, meaning its space complexity is strictly limited to the height of the tree $\mathcal{O}(H)$, rather than storing all $N$ elements in a second stack.

### Beautiful C++17 Code
```cpp
#include <vector>
#include <stack>

class SolutionOptimal {
public:
    std::vector<int> postorderTraversal(TreeNode* root) {
        std::vector<int> result;
        std::stack<TreeNode*> s;
        TreeNode* curr = root;
        TreeNode* lastVisited = nullptr;

        while (curr != nullptr || !s.empty()) {
            if (curr != nullptr) {
                s.push(curr);
                curr = curr->left; // Keep going left
            } else {
                TreeNode* peekNode = s.top();
                // If right child exists and traversing from left, go right
                if (peekNode->right != nullptr && lastVisited != peekNode->right) {
                    curr = peekNode->right;
                } else {
                    // Right child is processed or doesn't exist, visit parent
                    result.push_back(peekNode->val);
                    lastVisited = peekNode;
                    s.pop();
                }
            }
        }
        return result;
    }
};
```


#### Complexity Summary

| Metric | Recursive | Iterative (Two Stacks) | Iterative (Optimal One Stack) |
|:---|:---|:---|:---|
| **Time Complexity** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Space Complexity**| $\mathcal{O}(H)$ call stack | $\mathcal{O}(N)$ storage | $\mathcal{O}(H)$ explicit stack |
| **Space Complexity (Worst)**| $\mathcal{O}(N)$ | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Space Complexity (Best)** | $\mathcal{O}(\log N)$ | $\mathcal{O}(N)$ | $\mathcal{O}(\log N)$ |


---

## Q04 Level Order Traversal

### 1. Problem Explanation

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


### 2. Brute Solution

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


### 3. Optimal Solution & Complexities

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

### Why this is Optimal
It processes the tree strictly level by level in linear time ($\mathcal{O}(N)$). The space complexity is $\mathcal{O}(W)$ where $W$ is the maximum width of the tree. This is the optimal queue size for BFS on a tree.

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


#### Complexity Summary

| Metric | DFS-based (Brute) | BFS-based (Optimal) |
|:---|:---|:---|
| **Time Complexity** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Space Complexity (Worst)**| $\mathcal{O}(N)$ (skewed tree) | $\mathcal{O}(W)$ (complete tree: $W \approx N/2$) |
| **Space Complexity (Best)** | $\mathcal{O}(\log N)$ (balanced tree)| $\mathcal{O}(1)$ (skewed list tree) |

### Why is Space Complexity $\mathcal{O}(W)$?
In BFS, the maximum memory is occupied by the queue which holds nodes at the widest level of the tree. For a perfect binary tree, the leaf level contains $N/2$ nodes. Thus, the worst-case space is $\mathcal{O}(W)$ where $W$ is the maximum width of the tree.


---

## Q05 Height Depth

### 1. Problem Explanation

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


### 2. Brute Solution

*Note: Since the recursive DFS is the most straightforward representation, we will present the Iterative BFS as the "Better" solution and the Recursive DFS as the "Optimal" solution due to its minimal code overhead and performance.*


### 3. Optimal Solution & Complexities

The recursive DFS is highly optimized, has no queue overhead, and is the cleanest way to solve this problem.

### Algorithm
1. Base case: If `root` is `nullptr`, return `0`.
2. Find left height: `leftHeight = maxDepth(root->left)`.
3. Find right height: `rightHeight = maxDepth(root->right)`.
4. Return `1 + max(leftHeight, rightHeight)`.

### Why this is Optimal
It requires no auxiliary structures like queues, saving allocation overhead. It runs in linear time $\mathcal{O}(N)$ and uses stack space proportional to the height of the tree $\mathcal{O}(H)$.

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


#### Complexity Summary

| Metric | Recursive DFS (Optimal) | Iterative BFS |
|:---|:---|:---|
| **Time Complexity (Best)** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Time Complexity (Avg)** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Time Complexity (Worst)**| $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Space Complexity (Worst)**| $\mathcal{O}(N)$ (skewed tree stack) | $\mathcal{O}(N)$ (perfect tree queue size) |
| **Space Complexity (Best)** | $\mathcal{O}(\log N)$ (balanced tree)| $\mathcal{O}(1)$ (skewed tree) |


---

## Q06 Diameter

### 1. Problem Explanation

Given the root of a binary tree, return the **length of the diameter** of the tree.

The **diameter** of a binary tree is the length of the longest path between any two nodes in a tree. This path may or may not pass through the root.

The length of a path between two nodes is represented by the number of edges between them.

### Input
- `root`: A pointer to the root node of a binary tree.

### Output
- An integer representing the length of the diameter (number of edges).

### Constraints
- The number of nodes in the tree is in the range `[1, 10^4]`.
- Node values: `-100 <= Node.val <= 100`.

### Observations
- For any node in the tree, the longest path that passes through it as the highest point (curving back down) is the sum of the maximum heights of its left and right subtrees.
- Since height counts nodes, the number of edges along this path is:
  $$\text{edges} = \text{height}(\text{left}) + \text{height}(\text{right})$$
- The diameter of the tree is the maximum of this value calculated across all nodes in the tree.


### 2. Brute Solution

The brute force solution computes the height of every node individually, resulting in redundant calculations.

### Algorithm
1. Define a helper `getHeight(node)` that returns the height of a subtree.
2. Define `diameterOfBinaryTree(node)`:
   - Base case: If `node` is null, return 0.
   - Calculate path through root: `rootPath = getHeight(node->left) + getHeight(node->right)`.
   - Recurse left: `leftDiag = diameterOfBinaryTree(node->left)`.
   - Recurse right: `rightDiag = diameterOfBinaryTree(node->right)`.
   - Return `max({rootPath, leftDiag, rightDiag})`.

### Complexity Analysis
- **Time Complexity**: $\mathcal{O}(N^2)$ in the worst case (skewed tree) because we call `getHeight` which is $\mathcal{O}(N)$ for all $N$ nodes. In a balanced tree, it is $\mathcal{O}(N \log N)$.
- **Space Complexity**: $\mathcal{O}(H)$ for the stack frames. Worst case is $\mathcal{O}(N)$.

### C++17 Implementation
```cpp
#include <algorithm>

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
    int diameterOfBinaryTree(TreeNode* root) {
        if (root == nullptr) return 0;
        
        // Longest path passing through the root
        int rootPath = getHeight(root->left) + getHeight(root->right);
        
        // Longest paths not passing through the root
        int leftDiameter = diameterOfBinaryTree(root->left);
        int rightDiameter = diameterOfBinaryTree(root->right);
        
        return std::max({rootPath, leftDiameter, rightDiameter});
    }
};
```


### 3. Optimal Solution & Complexities

By combining height calculation and diameter tracking into a single recursion pass, we achieve linear time complexity.

### Algorithm
1. Initialize a global variable (or reference parameter) `diameter = 0`.
2. Define a helper function `calculateHeight(TreeNode* node, int& diameter)`:
   - Base case: If `node` is null, return `0`.
   - Recursively calculate left subtree height: `leftHeight = calculateHeight(node->left, diameter)`.
   - Recursively calculate right subtree height: `rightHeight = calculateHeight(node->right, diameter)`.
   - Update `diameter` with the maximum path found so far:
     `diameter = max(diameter, leftHeight + rightHeight)`.
   - Return the height of the current node: `1 + max(leftHeight, rightHeight)`.
3. Call `calculateHeight` on the root, and return `diameter`.

### Why this is Optimal
This solution visits each node exactly once. It computes node heights bottom-up and updates the diameter in-place, eliminating redundant traversal and keeping the runtime strictly $\mathcal{O}(N)$.

### Beautiful C++17 Code
```cpp
#include <algorithm>

class SolutionOptimal {
private:
    // Helper function that returns height of node and updates diameter
    int calculateHeight(TreeNode* node, int& diameter) {
        if (node == nullptr) {
            return 0;
        }
        
        // Bottom-up recursion: Calculate height of children first
        int leftHeight = calculateHeight(node->left, diameter);
        int rightHeight = calculateHeight(node->right, diameter);
        
        // Update diameter: path = left edges + right edges
        diameter = std::max(diameter, leftHeight + rightHeight);
        
        // Return height of current node
        return 1 + std::max(leftHeight, rightHeight);
    }
public:
    int diameterOfBinaryTree(TreeNode* root) {
        int diameter = 0;
        calculateHeight(root, diameter);
        return diameter;
    }
};
```


#### Complexity Summary

| Metric | Brute Force | Optimal DFS |
|:---|:---|:---|
| **Time Complexity (Best)** | $\mathcal{O}(N \log N)$ | $\mathcal{O}(N)$ |
| **Time Complexity (Worst)**| $\mathcal{O}(N^2)$ (skewed tree) | $\mathcal{O}(N)$ |
| **Space Complexity**| $\mathcal{O}(H)$ stack frames | $\mathcal{O}(H)$ stack frames |
| **Space Complexity (Worst)**| $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |


---

## Q07 Balanced Tree

### 1. Problem Explanation

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


### 2. Brute Solution

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


### 3. Optimal Solution & Complexities

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

### Why this is Optimal
It traverses each node at most once. It performs a bottom-up calculation: once a subtree is found to be unbalanced, the failure is bubbled up immediately without any further height evaluations. Time complexity is $\mathcal{O}(N)$.

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


#### Complexity Summary

| Metric | Brute Force | Optimal DFS (Sentinel) |
|:---|:---|:---|
| **Time Complexity (Best)** | $\mathcal{O}(N \log N)$ (balanced tree) | $\mathcal{O}(N)$ |
| **Time Complexity (Worst)**| $\mathcal{O}(N^2)$ (skewed tree) | $\mathcal{O}(N)$ |
| **Space Complexity**| $\mathcal{O}(H)$ stack frames | $\mathcal{O}(H)$ stack frames |
| **Space Complexity (Worst)**| $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |


---

## Q08 Mirror Invert

### 1. Problem Explanation

Given the root of a binary tree, **invert the tree**, and return its root.

Inverting a binary tree means that for every node in the tree, we swap its left and right children.

### Input
- `root`: A pointer to the root node of a binary tree.

### Output
- A pointer to the root node of the inverted binary tree.

### Constraints
- The number of nodes in the tree is in the range `[0, 100]` (or up to `1000` or `10^4`).
- Node values: `-100 <= Node.val <= 100`.

### Observations
- Inverting a tree is equivalent to creating its mirror image.
- At any given node, its left child becomes its right child, and its right child becomes its left child. This must happen recursively for all subtrees.
- This is the classic problem made famous by Max Howell's tweet: *"Google: 90% of our engineers use the software you wrote (Homebrew), but you can’t invert a binary tree on a whiteboard so fuck off."*


### 2. Brute Solution

*Note: Since the recursive DFS and iterative BFS are both $\mathcal{O}(N)$ time complexity and represent the standards, there is no "brute force" in terms of complexity. We will present the Recursive DFS as the primary clean optimal solution and Iterative BFS as the alternative robust implementation.*


### 3. Optimal Solution & Complexities

The recursive approach is clean, uses minimal lines of code, and modifies the pointers in-place.

### Algorithm
1. Base case: If `root` is null, return `nullptr`.
2. Save children: Keep pointers to the recursively inverted subtrees:
   - `TreeNode* leftInverted = invertTree(root->left);`
   - `TreeNode* rightInverted = invertTree(root->right);`
3. Swap children:
   - `root->left = rightInverted;`
   - `root->right = leftInverted;`
4. Return `root`.

### Why this is Optimal
It processes each node in constant time $\mathcal{O}(1)$ (excluding the traversal steps). The space complexity is $\mathcal{O}(H)$ call stack frames, making it highly memory-efficient for balanced trees.

### Beautiful C++17 Code
```cpp
#include <algorithm>

struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
};

class SolutionOptimal {
public:
    TreeNode* invertTree(TreeNode* root) {
        // Base case: empty node
        if (root == nullptr) {
            return nullptr;
        }
        
        // Recursively invert subtrees
        TreeNode* leftInverted = invertTree(root->left);
        TreeNode* rightInverted = invertTree(root->right);
        
        // Swap the pointers
        root->left = rightInverted;
        root->right = leftInverted;
        
        return root;
    }
};
```

### Alternate Solution (Iterative BFS Queue)
```cpp
#include <queue>
#include <algorithm>

class SolutionIterative {
public:
    TreeNode* invertTree(TreeNode* root) {
        if (root == nullptr) {
            return nullptr;
        }
        
        std::queue<TreeNode*> q;
        q.push(root);
        
        while (!q.empty()) {
            TreeNode* curr = q.front();
            q.pop();
            
            // Swap left and right children
            std::swap(curr->left, curr->right);
            
            // Push children to queue if they exist
            if (curr->left != nullptr) {
                q.push(curr->left);
            }
            if (curr->right != nullptr) {
                q.push(curr->right);
            }
        }
        
        return root;
    }
};
```


#### Complexity Summary

| Metric | Recursive DFS (Optimal) | Iterative BFS |
|:---|:---|:---|
| **Time Complexity** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Space Complexity (Worst)**| $\mathcal{O}(N)$ (skewed tree stack) | $\mathcal{O}(W) \approx \mathcal{O}(N)$ (widest level queue) |
| **Space Complexity (Best)** | $\mathcal{O}(\log N)$ (balanced tree)| $\mathcal{O}(1)$ (skewed tree) |


---

## Q09 Identical Trees

### 1. Problem Explanation

Given the roots of two binary trees `p` and `q`, write a function to check if they are the same or **identical**.

Two binary trees are considered identical if they are structurally identical, and the nodes have the exact same values at corresponding positions.

### Input
- `p`: A pointer to the root of the first binary tree.
- `q`: A pointer to the root of the second binary tree.

### Output
- A boolean (`true` or `false`) indicating whether the two trees are identical.

### Constraints
- The number of nodes in both trees is in the range `[0, 100]` (or up to `10^4`).
- Node values: `-10^4 <= Node.val <= 10^4`.

### Observations
- Two trees are identical if:
  1. Their roots have the same value (or both are null).
  2. Their left subtrees are identical.
  3. Their right subtrees are identical.
- This is a structural recursion check, meaning both the node values and the shape of the trees must match.


### 2. Brute Solution

*Note: The recursive DFS solution is the standard approach and runs in optimal $\mathcal{O}(N)$ time. There is no slower "brute force" variation; the iterative approach is an alternative that avoids recursive call stack limits.*


### 3. Optimal Solution & Complexities

The recursive DFS solution is clean, direct, and performs early pruning on mismatches.

### Algorithm
1. Base cases:
   - If both `p` and `q` are `nullptr`, return `true`.
   - If one of `p` or `q` is `nullptr` (but not both), return `false`.
2. Value check: If `p->val != q->val`, return `false`.
3. Recurse: Return `isSameTree(p->left, q->left) && isSameTree(p->right, q->right)`.

### Why this is Optimal
It visits each node at most once, performing a constant number of comparisons. It exits immediately as soon as a mismatch in structure or value is detected, minimizing execution time.

### Beautiful C++17 Code
```cpp
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
};

class SolutionOptimal {
public:
    bool isSameTree(TreeNode* p, TreeNode* q) {
        // Base case: both nodes are null (reached leaf boundaries together)
        if (p == nullptr && q == nullptr) {
            return true;
        }
        
        // Base case: one node is null and the other is not (structural mismatch)
        if (p == nullptr || q == nullptr) {
            return false;
        }
        
        // Value check: node values must match
        if (p->val != q->val) {
            return false;
        }
        
        // Recursively check left and right subtrees
        return isSameTree(p->left, q->left) && isSameTree(p->right, q->right);
    }
};
```

### Alternate Solution (Iterative BFS Queue)
```cpp
#include <queue>
#include <utility>

class SolutionIterative {
public:
    bool isSameTree(TreeNode* p, TreeNode* q) {
        std::queue<std::pair<TreeNode*, TreeNode*>> todo;
        todo.push({p, q});
        
        while (!todo.empty()) {
            auto [currP, currQ] = todo.front();
            todo.pop();
            
            // Both are null - valid match
            if (currP == nullptr && currQ == nullptr) {
                continue;
            }
            
            // One is null, or values don't match - invalid
            if (currP == nullptr || currQ == nullptr) {
                return false;
            }
            if (currP->val != currQ->val) {
                return false;
            }
            
            // Push children pairs to verify
            todo.push({currP->left, currQ->left});
            todo.push({currP->right, currQ->right});
        }
        
        return true;
    }
};
```


#### Complexity Summary

| Metric | Recursive DFS (Optimal) | Iterative BFS |
|:---|:---|:---|
| **Time Complexity** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Space Complexity (Worst)**| $\mathcal{O}(N)$ (skewed tree stack) | $\mathcal{O}(W) \approx \mathcal{O}(N)$ (widest level queue) |
| **Space Complexity (Best)** | $\mathcal{O}(\log N)$ (balanced tree)| $\mathcal{O}(1)$ (skewed tree) |

*Note: Here, $N$ is the number of nodes in the smaller of the two trees, as the traversal stops early upon finding any structural differences.*


---

## Q10 LCA BST

### 1. Problem Explanation

Given a binary search tree (BST), find the **Lowest Common Ancestor (LCA)** of two given nodes in the BST.

According to the definition of LCA on Wikipedia: “The lowest common ancestor is defined between two nodes $p$ and $q$ as the lowest node in $T$ that has both $p$ and $q$ as descendants (where we allow a node to be a descendant of itself).”

### Input
- `root`: A pointer to the root node of a BST.
- `p`: Pointer to the first node.
- `q`: Pointer to the second node.

### Output
- A pointer to the TreeNode representing the lowest common ancestor of `p` and `q`.

### Constraints
- The number of nodes in the tree is in the range `[2, 10^5]`.
- Node values: `-10^9 <= Node.val <= 10^9`.
- All `Node.val` are unique.
- `p` and `q` will exist in the BST and `p != q`.

### Observations
- In a BST, for every node:
  - All values in the left subtree are smaller than the node's value.
  - All values in the right subtree are larger than the node's value.
- This BST property allows us to locate the LCA without traversing the entire tree.


### 2. Brute Solution

The brute force solution is to use the general Binary Tree LCA algorithm, which traverses the entire tree and does not use the BST property. We will skip implementing the general algorithm here and jump straight to the BST-specific implementations.


### 3. Optimal Solution & Complexities

By using a simple while loop, we can eliminate the recursion stack entirely, achieving $\mathcal{O}(1)$ auxiliary space.

### Algorithm
1. Set a runner pointer `curr = root`.
2. While `curr` is not null:
   - If both `p->val` and `q->val` are less than `curr->val`, move `curr = curr->left`.
   - If both `p->val` and `q->val` are greater than `curr->val`, move `curr = curr->right`.
   - Else, we have found the split point. Return `curr`.
3. Return `nullptr` if the loop exits without finding the LCA (though constraints guarantee `p` and `q` exist).

### Why this is Optimal
It uses the BST property to run in $\mathcal{O}(H)$ time and avoids recursive call stack overhead, keeping auxiliary space at a constant $\mathcal{O}(1)$.

### Beautiful C++17 Code
```cpp
class SolutionOptimal {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        TreeNode* curr = root;
        
        while (curr != nullptr) {
            // If both targets are smaller than current node, go left
            if (p->val < curr->val && q->val < curr->val) {
                curr = curr->left;
            }
            // If both targets are larger than current node, go right
            else if (p->val > curr->val && q->val > curr->val) {
                curr = curr->right;
            }
            // We've found the split point (or curr is p or q)
            else {
                return curr;
            }
        }
        
        return nullptr;
    }
};
```


#### Complexity Summary

| Metric | Recursive DFS | Iterative (Optimal) |
|:---|:---|:---|
| **Time Complexity (Best)** | $\mathcal{O}(\log N)$ (balanced tree) | $\mathcal{O}(\log N)$ |
| **Time Complexity (Worst)**| $\mathcal{O}(N)$ (skewed tree) | $\mathcal{O}(N)$ |
| **Space Complexity (Worst)**| $\mathcal{O}(N)$ | $\mathcal{O}(1)$ auxiliary |
| **Space Complexity (Best)** | $\mathcal{O}(\log N)$ | $\mathcal{O}(1)$ auxiliary |


---

## Q11 LCA BT

### 1. Problem Explanation

Given a binary tree, find the **Lowest Common Ancestor (LCA)** of two given nodes, `p` and `q`.

According to the definition of LCA on Wikipedia: “The lowest common ancestor is defined between two nodes $p$ and $q$ as the lowest node in $T$ that has both $p$ and $q$ as descendants (where we allow a node to be a descendant of itself).”

### Input
- `root`: A pointer to the root node of a general binary tree.
- `p`: Pointer to the first node.
- `q`: Pointer to the second node.

### Output
- A pointer to the TreeNode representing the lowest common ancestor of `p` and `q`.

### Constraints
- The number of nodes in the tree is in the range `[2, 10^5]`.
- Node values: `-10^9 <= Node.val <= 10^9`.
- All `Node.val` are unique.
- `p` and `q` exist in the tree and `p != q`.

### Observations
- Unlike a BST, we cannot use node values to decide which subtree to search. We must search the entire tree using DFS.
- For any node in the tree:
  - If one target node lies in its left subtree and the other target node lies in its right subtree, then the current node **must** be the LCA.
  - If both target nodes lie in its left subtree, the LCA must also be in the left subtree.
  - If both target nodes lie in its right subtree, the LCA must also be in the right subtree.


### 2. Brute Solution

*Note: Since the recursive DFS solution is the standard approach and runs in optimal $\mathcal{O}(N)$ time, there is no slower "brute force" variation; we will detail the recursive version below as the Optimal Solution.*


### 3. Optimal Solution & Complexities

The recursive DFS solution is clean, elegant, and handles all path split and parent-child scenarios.

### Algorithm
1. Base cases:
   - If `root` is null, return `nullptr`.
   - If `root` is equal to `p` or `q`, return `root`.
2. Recursively search subtrees:
   - `leftLCA = lowestCommonAncestor(root->left, p, q)`
   - `rightLCA = lowestCommonAncestor(root->right, p, q)`
3. Evaluate results:
   - If both `leftLCA` and `rightLCA` are non-null, return `root` (split point).
   - If `leftLCA` is not null (and `rightLCA` is null), return `leftLCA`.
   - If `rightLCA` is not null (and `leftLCA` is null), return `rightLCA`.
   - If both are null, return `nullptr`.

### Why this is Optimal
It visits each node at most once, running in $\mathcal{O}(N)$ time. The space complexity is $\mathcal{O}(H)$ stack frames, which is the minimum required to traverse a general tree without parent pointers.

### Beautiful C++17 Code
```cpp
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
};

class SolutionOptimal {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        // Base case: if we hit a null node, or find either target node, return it
        if (root == nullptr || root == p || root == q) {
            return root;
        }
        
        // Recurse into left and right subtrees
        TreeNode* leftLCA = lowestCommonAncestor(root->left, p, q);
        TreeNode* rightLCA = lowestCommonAncestor(root->right, p, q);
        
        // If both subtrees returned a non-null node, the current node is the LCA
        if (leftLCA != nullptr && rightLCA != nullptr) {
            return root;
        }
        
        // Otherwise, return the non-null result (if any)
        return (leftLCA != nullptr) ? leftLCA : rightLCA;
    }
};
```


#### Complexity Summary

| Metric | Recursive DFS (Optimal) |
|:---|:---|
| **Time Complexity (Best)** | $\mathcal{O}(H)$ (targets found near the root) |
| **Time Complexity (Avg)** | $\mathcal{O}(N)$ |
| **Time Complexity (Worst)**| $\mathcal{O}(N)$ |
| **Space Complexity (Worst)**| $\mathcal{O}(N)$ (skewed tree stack) |
| **Space Complexity (Best)** | $\mathcal{O}(\log N)$ (balanced tree) |


---

## Q12 Path Root To Node

### 1. Problem Explanation

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


### 2. Brute Solution

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


### 3. Optimal Solution & Complexities

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


#### Complexity Summary

*   **Time Complexity**: $\mathcal{O}(N)$
    *   In the worst case (where the target is at the last leaf or not present), we visit every node in the binary tree exactly once.
*   **Space Complexity**: $\mathcal{O}(H)$
    *   $H$ is the height of the tree.
    *   The recursion stack uses $\mathcal{O}(H)$ space.
    *   The `path` vector stores at most $H + 1$ elements since the maximum length of a root-to-node path is bounded by the height of the tree.
    *   For a balanced tree, $H = \log_2(N)$, resulting in $\mathcal{O}(\log N)$ space. For a skewed tree, $H = N$, resulting in $\mathcal{O}(N)$ space.


---

## Q13 Zigzag Traversal

### 1. Problem Explanation

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


### 2. Brute Solution

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


### 3. Optimal Solution & Complexities

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


#### Complexity Summary

*   **Time Complexity**: $\mathcal{O}(N)$
    *   Every node in the tree is visited exactly once.
    *   Index assignments take $\mathcal{O}(1)$ time per node.
*   **Space Complexity**: $\mathcal{O}(W)$
    *   $W$ is the maximum width of the tree.
    *   In the worst case (a complete binary tree), the maximum number of nodes at the lowest level is $W \approx N/2$, so the queue takes $\mathcal{O}(N)$ space in the worst case.
    *   For a highly skewed tree, $W = 1$, requiring $\mathcal{O}(1)$ auxiliary space.


---

## Q14 Vertical Order Traversal

### 1. Problem Explanation

Given the root of a binary tree, calculate the vertical order traversal of the binary tree.

For each node at position `(row, col)`, its left child will be at `(row + 1, col - 1)` and its right child will be at `(row + 1, col + 1)`. The root of the tree is at `(0, 0)`.

The vertical order traversal is a list of top-to-bottom orderings for each column index starting from the left-most column and ending on the right-most column. There may be multiple nodes in the same row and same column. In such a case, sort these nodes by their values.

*   **Input**:
    *   A pointer to the root of the binary tree: `TreeNode* root`.
*   **Output**:
    *   A 2D vector of integers: `vector<vector<int>>` representing the vertical order traversal.
*   **Constraints**:
    *   $1 \le N \le 1000$ (Number of nodes in the tree)
    *   $0 \le \text{Node value} \le 1000$

*   **Observations**:
    *   Nodes are categorized by their vertical column (`col`).
    *   Columns are visited from left to right (minimum `col` to maximum `col`).
    *   Within each column, nodes are processed from top to bottom (minimum `row` to maximum `row`).
    *   If two nodes share both the same column and row, they must be sorted in ascending order of their values.


### 2. Brute Solution

*   **Algorithm**:
    *   Perform a DFS. Collect all nodes into a flat list of tuples: `(col, row, value)`.
    *   Sort the flat list of size $N$ using a custom comparator:
        *   First by `col` ascending.
        *   Then by `row` ascending.
        *   Then by `value` ascending.
    *   Group elements by `col` and populate the output list.
*   **Dry Run**:
    *   Flat list representation: `[(0, 0, 1), (-1, 1, 2), (1, 1, 3)]`.
    *   Sorted list: `[(-1, 1, 2), (0, 0, 1), (1, 1, 3)]`.
    *   Grouped: `[[2], [1], [3]]`.
*   **Complexity**:
    *   **Time Complexity**: $O(N \log N)$ due to sorting the flat list.
    *   **Space Complexity**: $O(N)$ to store all nodes in a list.
*   **Pros/Cons**:
    *   *Pros*: Easy to write standard sort logic.
    *   *Cons*: Custom comparators can be error-prone; sorting a large vector at once may have cache miss overhead.

### Brute Force C++17 Code
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

struct NodeInfo {
    int col;
    int row;
    int val;
};

void dfsBrute(TreeNode* root, int row, int col, std::vector<NodeInfo>& list) {
    if (!root) return;
    list.push_back({col, row, root->val});
    dfsBrute(root->left, row + 1, col - 1, list);
    dfsBrute(root->right, row + 1, col + 1, list);
}

std::vector<std::vector<int>> verticalTraversalBrute(TreeNode* root) {
    std::vector<NodeInfo> list;
    dfsBrute(root, 0, 0, list);

    std::sort(list.begin(), list.end(), [](const NodeInfo& a, const NodeInfo& b) {
        if (a.col != b.col) return a.col < b.col;
        if (a.row != b.row) return a.row < b.row;
        return a.val < b.val;
    });

    std::vector<std::vector<int>> result;
    if (list.empty()) return result;

    int current_col = list[0].col;
    std::vector<int> col_vals;
    for (const auto& node : list) {
        if (node.col != current_col) {
            result.push_back(col_vals);
            col_vals.clear();
            current_col = node.col;
        }
        col_vals.push_back(node.val);
    }
    result.push_back(col_vals);
    return result;
}
```


### 3. Optimal Solution & Complexities

*   **Approach**: BFS with nested `std::map` mapping.
*   **Data Structure**: `map<int, map<int, multiset<int>>> nodes;`
*   **Algorithm**:
    1.  Initialize a queue of pairs: `queue<pair<TreeNode*, pair<int, int>>> q;` storing `(node, (row, col))`.
    2.  Push `(root, (0, 0))` to `q`.
    3.  While `q` is not empty:
        *   Pop the front element: `curr`, `row`, `col`.
        *   Insert `curr->val` into `nodes[col][row]`.
        *   If `curr->left` exists, push `(curr->left, (row + 1, col - 1))` to `q`.
        *   If `curr->right` exists, push `(curr->right, (row + 1, col + 1))` to `q`.
    4.  Traverse the map `nodes` (since `map` is ordered, `col` keys will be iterated from lowest to highest, and nested `row` keys will also be iterated from lowest to highest):
        *   For each column, create a vector `col_vals`.
        *   Iterate through the row map of that column.
        *   Extract all values from the `multiset` and append them to `col_vals`.
        *   Push `col_vals` into the `result` vector.

### Optimal C++17 Code
```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <map>
#include <set>

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
     * Computes the vertical order traversal of a binary tree.
     */
    std::vector<std::vector<int>> verticalTraversal(TreeNode* root) {
        std::vector<std::vector<int>> result;
        if (root == nullptr) {
            return result;
        }

        // map<col, map<row, multiset<val>>>
        // Ordered map maintains sorted columns and sorted rows.
        // Multiset maintains sorted values for duplicate coordinates.
        std::map<int, std::map<int, std::multiset<int>>> nodes;
        
        // Queue stores pair of: (Node, (row, col))
        std::queue<std::pair<TreeNode*, std::pair<int, int>>> q;
        q.push({root, {0, 0}});

        while (!q.empty()) {
            auto [curr, coords] = q.front();
            q.pop();

            int row = coords.first;
            int col = coords.second;

            // Insert value into the structured coordinate map
            nodes[col][row].insert(curr->val);

            // Left child goes to row + 1, col - 1
            if (curr->left != nullptr) {
                q.push({curr->left, {row + 1, col - 1}});
            }
            // Right child goes to row + 1, col + 1
            if (curr->right != nullptr) {
                q.push({curr->right, {row + 1, col + 1}});
            }
        }

        // Extract values from ordered map structure
        for (auto const& [col, row_map] : nodes) {
            std::vector<int> col_vals;
            for (auto const& [row, vals] : row_map) {
                // Insert all sorted values from the multiset
                col_vals.insert(col_vals.end(), vals.begin(), vals.end());
            }
            result.push_back(col_vals);
        }

        return result;
    }
};
```


#### Complexity Summary

*   **Time Complexity**: $\mathcal{O}(N \log N)$
    *   For each of the $N$ nodes, we push and pop from the queue: $\mathcal{O}(N)$.
    *   We insert each value into `std::map` (nesting level 2) and `std::multiset`.
    *   The depth of the outer map is $W$ (width of tree), and inner map is $H$ (height of tree).
    *   Inserting into map of maps with multiset takes $\mathcal{O}(\log W + \log H + \log N)$ which simplifies to $\mathcal{O}(\log N)$ per node.
    *   Thus, total time is $\mathcal{O}(N \log N)$.
*   **Space Complexity**: $\mathcal{O}(N)$
    *   To store the nodes in the map and the queue.


---

## Q15 Top View

### 1. Problem Explanation

Given a Binary Tree, return the values of nodes as they would be seen from the top of the tree, ordered from left to right.

A node is visible from the top if it is the first node encountered in its vertical column when looking from the top down.

*   **Input**:
    *   A pointer to the root of the binary tree: `TreeNode* root`.
*   **Output**:
    *   A vector of integers: `vector<int>` representing the top view values ordered from the leftmost column to the rightmost column.
*   **Constraints**:
    *   $1 \le N \le 10^5$ (Number of nodes in the tree)
    *   $1 \le \text{Node value} \le 10^5$
*   **Observations**:
    *   We use horizontal distance (column index) to determine top view visibility.
    *   The root has a horizontal distance of `0`.
    *   Its left child is at `-1`, right child is at `+1`.
    *   The first node we visit at a specific horizontal distance (using BFS) will be the one visible from the top.
    *   Subsequent nodes at the same horizontal distance are hidden under the top-most node.


### 2. Brute Solution

*   **Algorithm**:
    *   Perform a complete vertical order traversal of the tree (e.g., using DFS or BFS to collect all coordinates).
    *   Store all nodes grouped by their column index.
    *   For each column, sort the nodes by their level (row) and value, then pick the first element.
    *   Sort the columns from left to right.
*   **Complexity**:
    *   **Time Complexity**: $O(N \log N)$ to store and sort all node coordinates.
    *   **Space Complexity**: $O(N)$ to store all nodes in memory.
*   **Pros/Cons**:
    *   *Pros*: Heavy reuse of the "Vertical Order Traversal" solution.
    *   *Cons*: Inefficient, because we store all nodes of the tree, even though we only care about the top node of each column.

### Brute Force C++17 Code
```cpp
#include <iostream>
#include <vector>
#include <map>
#include <algorithm>

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

struct DFSNode {
    int row;
    int val;
};

// DFS helper to collect nodes
void collectAllNodes(TreeNode* root, int row, int col, std::map<int, std::vector<DFSNode>>& colMap) {
    if (!root) return;
    colMap[col].push_back({row, root->val});
    collectAllNodes(root->left, row + 1, col - 1, colMap);
    collectAllNodes(root->right, row + 1, col + 1, colMap);
}

std::vector<int> topViewBrute(TreeNode* root) {
    if (!root) return {};
    std::map<int, std::vector<DFSNode>> colMap;
    collectAllNodes(root, 0, 0, colMap);
    
    std::vector<int> result;
    for (auto& [col, nodeList] : colMap) {
        // Sort by row (level) ascending
        std::sort(nodeList.begin(), nodeList.end(), [](const DFSNode& a, const DFSNode& b) {
            return a.row < b.row;
        });
        result.push_back(nodeList[0].val); // Grab the top-most node
    }
    return result;
}
```


### 3. Optimal Solution & Complexities

*   **Approach**: BFS with standard Queue and `std::map`.
*   **Algorithm**:
    1.  Initialize a `std::map<int, int> topNodeMap` where the key is horizontal distance `hd` and value is the node's value.
    2.  Initialize a queue of pairs: `queue<pair<TreeNode*, int>> q;` storing `(node, hd)`.
    3.  Push `(root, 0)` into `q`.
    4.  While `q` is not empty:
        *   Pop the front element: `curr` and `hd`.
        *   If `hd` is not in `topNodeMap`, insert it: `topNodeMap[hd] = curr->val`.
        *   If `curr->left` exists, push `(curr->left, hd - 1)` into `q`.
        *   If `curr->right` exists, push `(curr->right, hd + 1)` into `q`.
    5.  Iterate through `topNodeMap` and extract the values into the result vector.

### Optimal C++17 Code
```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <map>

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
     * Function to return the top view of a binary tree.
     */
    std::vector<int> topView(TreeNode* root) {
        std::vector<int> result;
        if (root == nullptr) {
            return result;
        }

        // Ordered map to store first node value at each horizontal distance (hd)
        std::map<int, int> topNodeMap;

        // Queue to store pair of (Node, Horizontal Distance)
        std::queue<std::pair<TreeNode*, int>> q;
        q.push({root, 0});

        while (!q.empty()) {
            auto [curr, hd] = q.front();
            q.pop();

            // If the horizontal distance is visited for the first time, 
            // it is the top-most node of this vertical column.
            if (topNodeMap.find(hd) == topNodeMap.end()) {
                topNodeMap[hd] = curr->val;
            }

            // Push left child with horizontal distance = hd - 1
            if (curr->left != nullptr) {
                q.push({curr->left, hd - 1});
            }
            // Push right child with horizontal distance = hd + 1
            if (curr->right != nullptr) {
                q.push({curr->right, hd + 1});
            }
        }

        // Extract values in sorted order of horizontal distance
        for (auto const& [hd, val] : topNodeMap) {
            result.push_back(val);
        }

        return result;
    }
};
```


#### Complexity Summary

*   **Time Complexity**: $\mathcal{O}(N \log N)$
    *   We visit each of the $N$ nodes exactly once: $\mathcal{O}(N)$.
    *   For each node, we check and insert into a `std::map`. The size of the map is bounded by the width of the tree $W$ (which is at most $N$).
    *   Map lookup and insertion take $\mathcal{O}(\log W)$ time.
    *   Total time complexity: $\mathcal{O}(N \log W) \approx \mathcal{O}(N \log N)$.
*   **Space Complexity**: $\mathcal{O}(N)$
    *   The map contains at most $W$ elements.
    *   The queue contains at most the maximum width of the tree $W$ at any level.
    *   In the worst case (a broad complete tree), $W \approx N/2$. Hence, $\mathcal{O}(N)$ space is required.


---

## Q16 Bottom View

### 1. Problem Explanation

Given a Binary Tree, return the values of nodes as they would be seen from the bottom of the tree, ordered from left to right.

A node is visible from the bottom if it is the last node encountered in its vertical column when looking from the top down. 

If there are multiple bottom-most nodes at the same horizontal distance and same level, the node that is processed later in BFS (from left to right) is visible.

*   **Input**:
    *   A pointer to the root of the binary tree: `TreeNode* root`.
*   **Output**:
    *   A vector of integers: `vector<int>` representing the bottom view values ordered from the leftmost column to the rightmost column.
*   **Constraints**:
    *   $1 \le N \le 10^5$ (Number of nodes in the tree)
    *   $1 \le \text{Node value} \le 10^5$
*   **Observations**:
    *   Similar to the top view, we track the horizontal distance (column index) `hd`.
    *   Root is at `0`, left is `hd - 1`, right is `hd + 1`.
    *   Unlike the top view where the *first* node seen in a column wins, for the bottom view, the *last* node seen in a column wins.
    *   BFS naturally processes nodes from top to bottom. Hence, overwriting the value in our map for a given `hd` ensures that nodes lower down in the tree will overwrite higher ones.


### 2. Brute Solution

*   **Algorithm**:
    *   Perform a complete vertical order traversal of the tree (e.g., using DFS or BFS to collect all coordinates).
    *   Store all nodes grouped by their column index.
    *   For each column, sort the nodes by their level (row) and horizontal coordinate, then pick the last element.
    *   Sort the columns from left to right.
*   **Complexity**:
    *   **Time Complexity**: $O(N \log N)$ to store and sort all node coordinates.
    *   **Space Complexity**: $O(N)$ to store all nodes in memory.
*   **Pros/Cons**:
    *   *Pros*: Standard reuse of Vertical Traversal logic.
    *   *Cons*: Too much overhead; stores all nodes instead of just updating the latest.

### Brute Force C++17 Code
```cpp
#include <iostream>
#include <vector>
#include <map>
#include <algorithm>

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

struct DFSNode {
    int row;
    int val;
};

// DFS helper to collect nodes
void collectAllNodes(TreeNode* root, int row, int col, std::map<int, std::vector<DFSNode>>& colMap) {
    if (!root) return;
    colMap[col].push_back({row, root->val});
    collectAllNodes(root->left, row + 1, col - 1, colMap);
    collectAllNodes(root->right, row + 1, col + 1, colMap);
}

std::vector<int> bottomViewBrute(TreeNode* root) {
    if (!root) return {};
    std::map<int, std::vector<DFSNode>> colMap;
    collectAllNodes(root, 0, 0, colMap);
    
    std::vector<int> result;
    for (auto& [col, nodeList] : colMap) {
        // Sort by row (level) ascending
        std::sort(nodeList.begin(), nodeList.end(), [](const DFSNode& a, const DFSNode& b) {
            return a.row < b.row;
        });
        // Grab the bottom-most node (last in sorted list)
        result.push_back(nodeList.back().val); 
    }
    return result;
}
```


### 3. Optimal Solution & Complexities

*   **Approach**: BFS with standard Queue and `std::map` (overwriting values).
*   **Algorithm**:
    1.  Initialize a `std::map<int, int> bottomNodeMap` where the key is horizontal distance `hd` and value is the node's value.
    2.  Initialize a queue of pairs: `queue<pair<TreeNode*, int>> q;` storing `(node, hd)`.
    3.  Push `(root, 0)` into `q`.
    4.  While `q` is not empty:
        *   Pop the front element: `curr` and `hd`.
        *   Overwrite the map: `bottomNodeMap[hd] = curr->val`. (Every new node seen at `hd` will replace the older one).
        *   If `curr->left` exists, push `(curr->left, hd - 1)` into `q`.
        *   If `curr->right` exists, push `(curr->right, hd + 1)` into `q`.
    5.  Iterate through `bottomNodeMap` and extract the values into the result vector.

### Optimal C++17 Code
```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <map>

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
     * Function to return the bottom view of a binary tree.
     */
    std::vector<int> bottomView(TreeNode* root) {
        std::vector<int> result;
        if (root == nullptr) {
            return result;
        }

        // Ordered map to store the last node value seen at each horizontal distance (hd)
        std::map<int, int> bottomNodeMap;

        // Queue to store pair of (Node, Horizontal Distance)
        std::queue<std::pair<TreeNode*, int>> q;
        q.push({root, 0});

        while (!q.empty()) {
            auto [curr, hd] = q.front();
            q.pop();

            // Overwrite the horizontal distance with the current node value.
            // This ensures that the lowest node processed in BFS for each column remains.
            bottomNodeMap[hd] = curr->val;

            // Push left child with horizontal distance = hd - 1
            if (curr->left != nullptr) {
                q.push({curr->left, hd - 1});
            }
            // Push right child with horizontal distance = hd + 1
            if (curr->right != nullptr) {
                q.push({curr->right, hd + 1});
            }
        }

        // Extract values in sorted order of horizontal distance
        for (auto const& [hd, val] : bottomNodeMap) {
            result.push_back(val);
        }

        return result;
    }
};
```


#### Complexity Summary

*   **Time Complexity**: $\mathcal{O}(N \log N)$
    *   Each of the $N$ nodes is pushed and popped from the queue once: $\mathcal{O}(N)$.
    *   For each node, we insert/overwrite in the map: $\mathcal{O}(\log W)$ where $W$ is the tree width.
    *   Total time complexity: $\mathcal{O}(N \log W) \approx \mathcal{O}(N \log N)$.
*   **Space Complexity**: $\mathcal{O}(N)$
    *   Queue and map store at most $O(N)$ elements.


---

## Q17 Left Right View

### 1. Problem Explanation

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


### 2. Brute Solution

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


### 3. Optimal Solution & Complexities

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


#### Complexity Summary

*   **Time Complexity**: $\mathcal{O}(N)$
    *   We visit every node in the binary tree exactly once.
*   **Space Complexity**: $\mathcal{O}(H)$
    *   $H$ is the height of the tree.
    *   The space is taken by the recursion stack.
    *   In the worst case (skewed tree), height $H = N$, giving $\mathcal{O}(N)$ space.
    *   In the best case (balanced tree), height $H = \log N$, giving $\mathcal{O}(\log N)$ space.
    *   Note that this is more space-efficient than BFS, which takes $\mathcal{O}(W)$ space where $W \approx N/2$ in the worst case.


---

## Q18 BST Operations

### 1. Problem Explanation

Implement the core operations of a Binary Search Tree (BST):
1.  **Search**: Determine if a key exists in the BST. Return the node if found, otherwise `nullptr`.
2.  **Insert**: Insert a key into the BST while maintaining the BST property. If the key already exists, do nothing or handle duplicates. Return the root of the modified BST.
3.  **Delete**: Remove a key from the BST while maintaining the BST property. Return the root of the modified BST.

*   **BST Properties**:
    *   The left subtree of a node contains only nodes with keys lesser than the node's key.
    *   The right subtree of a node contains only nodes with keys greater than the node's key.
    *   The left and right subtrees must each also be binary search trees.

*   **Input / Output**:
    *   `TreeNode* search(TreeNode* root, int key)`
    *   `TreeNode* insert(TreeNode* root, int key)`
    *   `TreeNode* deleteNode(TreeNode* root, int key)`
*   **Constraints**:
    *   $0 \le N \le 10^5$ (Number of nodes in the tree)
    *   $-10^9 \le \text{Key, Node value} \le 10^9$


### 2. Brute Solution

*   *Note*: There is no "brute force" for BST operations other than treating the BST as a regular binary tree (e.g., searching linearly, reconstructing the tree after insertion/deletion). Since we are working with BST properties, we jump straight to the standard recursive and iterative optimal operations.


### 3. Optimal Solution & Complexities

### Optimal C++17 Code
```cpp
#include <iostream>

/**
 * Definition for a binary tree node.
 */
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class BSTOperations {
public:
    // ==========================================
    // 1. SEARCH OPERATIONS
    // ==========================================

    /**
     * Recursive Search: O(H) Time, O(H) Space
     */
    TreeNode* searchRecursive(TreeNode* root, int key) {
        if (root == nullptr || root->val == key) {
            return root;
        }
        if (key < root->val) {
            return searchRecursive(root->left, key);
        }
        return searchRecursive(root->right, key);
    }

    /**
     * Iterative Search: O(H) Time, O(1) Space
     */
    TreeNode* searchIterative(TreeNode* root, int key) {
        TreeNode* curr = root;
        while (curr != nullptr && curr->val != key) {
            if (key < curr->val) {
                curr = curr->left;
            } else {
                curr = curr->right;
            }
        }
        return curr;
    }

    // ==========================================
    // 2. INSERT OPERATIONS
    // ==========================================

    /**
     * Recursive Insert: O(H) Time, O(H) Space
     */
    TreeNode* insertRecursive(TreeNode* root, int key) {
        if (root == nullptr) {
            return new TreeNode(key);
        }
        if (key < root->val) {
            root->left = insertRecursive(root->left, key);
        } else if (key > root->val) {
            root->right = insertRecursive(root->right, key);
        }
        return root;
    }

    /**
     * Iterative Insert: O(H) Time, O(1) Space
     */
    TreeNode* insertIterative(TreeNode* root, int key) {
        TreeNode* newNode = new TreeNode(key);
        if (root == nullptr) {
            return newNode;
        }

        TreeNode* curr = root;
        TreeNode* parent = nullptr;

        while (curr != nullptr) {
            parent = curr;
            if (key < curr->val) {
                curr = curr->left;
            } else if (key > curr->val) {
                curr = curr->right;
            } else {
                delete newNode; // Key already exists, clean up memory
                return root;
            }
        }

        if (key < parent->val) {
            parent->left = newNode;
        } else {
            parent->right = newNode;
        }

        return root;
    }

    // ==========================================
    // 3. DELETE OPERATIONS
    // ==========================================

    /**
     * Helper to find the inorder successor (min in right subtree)
     */
    TreeNode* findMin(TreeNode* root) {
        while (root->left != nullptr) {
            root = root->left;
        }
        return root;
    }

    /**
     * Recursive Delete: O(H) Time, O(H) Space
     */
    TreeNode* deleteRecursive(TreeNode* root, int key) {
        if (root == nullptr) {
            return nullptr;
        }

        // 1. Locate the node
        if (key < root->val) {
            root->left = deleteRecursive(root->left, key);
        } else if (key > root->val) {
            root->right = deleteRecursive(root->right, key);
        } else {
            // Found the node!

            // Case 1 & 2: Leaf or single child
            if (root->left == nullptr) {
                TreeNode* temp = root->right;
                delete root;
                return temp;
            } else if (root->right == nullptr) {
                TreeNode* temp = root->left;
                delete root;
                return temp;
            }

            // Case 3: Two children
            // Find inorder successor (min in right subtree)
            TreeNode* temp = findMin(root->right);
            // Replace value
            root->val = temp->val;
            // Delete successor
            root->right = deleteRecursive(root->right, temp->val);
        }
        return root;
    }

    /**
     * Iterative Delete: O(H) Time, O(1) Space
     */
    TreeNode* deleteIterative(TreeNode* root, int key) {
        TreeNode* curr = root;
        TreeNode* parent = nullptr;

        // Locate node to delete
        while (curr != nullptr && curr->val != key) {
            parent = curr;
            if (key < curr->val) {
                curr = curr->left;
            } else {
                curr = curr->right;
            }
        }

        // Key not found
        if (curr == nullptr) {
            return root;
        }

        // Case 1 & 2: At most one child
        if (curr->left == nullptr || curr->right == nullptr) {
            TreeNode* newCurr = (curr->left == nullptr) ? curr->right : curr->left;

            if (parent == nullptr) {
                delete curr;
                return newCurr; // Deleting root node
            }

            if (curr == parent->left) {
                parent->left = newCurr;
            } else {
                parent->right = newCurr;
            }
            delete curr;
        } 
        // Case 3: Two children
        else {
            // Find successor and its parent
            TreeNode* successorParent = curr;
            TreeNode* successor = curr->right;

            while (successor->left != nullptr) {
                successorParent = successor;
                successor = successor->left;
            }

            // Copy successor value
            curr->val = successor->val;

            // Delete successor node
            if (successorParent->left == successor) {
                successorParent->left = successor->right;
            } else {
                successorParent->right = successor->right;
            }
            delete successor;
        }

        return root;
    }
};
```


#### Complexity Summary

| Operation | Average Case Time | Worst Case Time | Space (Recursive) | Space (Iterative) |
| :--- | :--- | :--- | :--- | :--- |
| **Search** | $\mathcal{O}(\log N)$ | $\mathcal{O}(N)$ | $\mathcal{O}(H)$ | $\mathcal{O}(1)$ |
| **Insert** | $\mathcal{O}(\log N)$ | $\mathcal{O}(N)$ | $\mathcal{O}(H)$ | $\mathcal{O}(1)$ |
| **Delete** | $\mathcal{O}(\log N)$ | $\mathcal{O}(N)$ | $\mathcal{O}(H)$ | $\mathcal{O}(1)$ |

*   *Note*: The worst case occurs when the tree is skewed (linear list), where height $H = N$. Average case occurs when the tree is balanced, where height $H = \log_2 N$.


---

## Q19 Valid BST

### 1. Problem Explanation

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


### 2. Brute Solution

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


### 3. Optimal Solution & Complexities

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


#### Complexity Summary

*   **Time Complexity**: $\mathcal{O}(N)$
    *   Each node in the tree is visited at most once.
*   **Space Complexity**: $\mathcal{O}(H)$
    *   $H$ is the height of the tree.
    *   The space is occupied by the recursion stack.
    *   For a balanced tree, $H = \log N$. For a skewed tree, $H = N$.


---

## Q20 Kth Smallest BST

### 1. Problem Explanation

Given the root of a binary search tree (BST) and an integer `k`, return the `k`-th smallest element (1-indexed) of all the values of the nodes in the tree.

*   **Input**:
    *   A pointer to the root of the binary search tree: `TreeNode* root`.
    *   An integer `k`.
*   **Output**:
    *   An integer representing the `k`-th smallest element in the BST.
*   **Constraints**:
    *   The number of nodes in the tree is $N$.
    *   $1 \le k \le N \le 10^4$
    *   $0 \le \text{Node value} \le 10^4$

*   **Observations**:
    *   An inorder traversal of a Binary Search Tree (Left $\rightarrow$ Root $\rightarrow$ Right) visits nodes in ascending order.
    *   The $k$-th node visited during the inorder traversal will be the $k$-th smallest element.


### 2. Brute Solution

*   **Algorithm**:
    *   Perform a full recursive inorder traversal and append elements to a list.
    *   Return the $(k-1)$-th element of the list.
*   **Complexity**:
    *   **Time Complexity**: $O(N)$ because we visit all $N$ nodes.
    *   **Space Complexity**: $O(N)$ to store all values in a vector.
*   **Pros/Cons**:
    *   *Pros*: Straightforward to implement.
    *   *Cons*: Inefficient for small values of $k$; uses $O(N)$ memory.

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

void inorderBrute(TreeNode* root, std::vector<int>& nodes) {
    if (!root) return;
    inorderBrute(root->left, nodes);
    nodes.push_back(root->val);
    inorderBrute(root->right, nodes);
}

int kthSmallestBrute(TreeNode* root, int k) {
    std::vector<int> nodes;
    inorderBrute(root, nodes);
    return nodes[k - 1];
}
```


### 3. Optimal Solution & Complexities

The **Iterative Inorder Traversal using a Stack** is the cleanest optimal solution because it avoids recursion state variables and explicitly demonstrates early termination.

### Algorithm
1.  Initialize an empty stack `s` of `TreeNode*`.
2.  Initialize `curr = root`.
3.  While `curr != nullptr` or the stack is not empty:
    *   While `curr != nullptr` (go as deep left as possible):
        *   Push `curr` to stack.
        *   `curr = curr->left`.
    *   Pop the top node from stack: `curr = s.top(); s.pop();`.
    *   Decrement `k`.
    *   If `k == 0`, we have reached the $k$-th smallest element. Return `curr->val`.
    *   Set `curr = curr->right` (explore right subtree).

### Optimal C++17 Code
```cpp
#include <iostream>
#include <stack>

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
     * Finds the kth smallest element in a BST using iterative inorder traversal.
     */
    int kthSmallest(TreeNode* root, int k) {
        std::stack<TreeNode*> s;
        TreeNode* curr = root;

        while (curr != nullptr || !s.empty()) {
            // Push all left children of the current node to the stack
            while (curr != nullptr) {
                s.push(curr);
                curr = curr->left;
            }

            // Process the current node
            curr = s.top();
            s.pop();
            
            k--;
            if (k == 0) {
                return curr->val;
            }

            // Move to the right child
            curr = curr->right;
        }

        return -1; // Should not be reached under problem constraints
    }
};
```


#### Complexity Summary

*   **Time Complexity**: $\mathcal{O}(H + k)$
    *   We spend $\mathcal{O}(H)$ time to reach the leftmost leaf of the tree.
    *   From there, we process $k$ nodes.
    *   In the worst case (e.g. skewed tree and $k = N$), this is $\mathcal{O}(N)$.
    *   On average for a balanced tree, height $H = \log N$, so time complexity is $\mathcal{O}(\log N + k)$.
*   **Space Complexity**: $\mathcal{O}(H)$
    *   The stack stores at most $H$ elements at any time.
    *   For a balanced tree, space is $\mathcal{O}(\log N)$. For a skewed tree, it is $\mathcal{O}(N)$.


---

## Q21 BST To DLL

### 1. Problem Explanation

Convert a Binary Search Tree (BST) to a sorted Doubly Linked List (DLL) in-place. 

The left pointer of a tree node should behave as the previous pointer (`prev`) in the DLL, and the right pointer of a tree node should behave as the next pointer (`next`) in the DLL.

The conversion must be done in-place, meaning you cannot allocate new nodes. The order of elements in the DLL must be sorted in ascending order (which corresponds to the inorder traversal of the BST).

*   **Input**:
    *   A pointer to the root of the BST: `TreeNode* root`.
*   **Output**:
    *   A pointer to the head node of the sorted Doubly Linked List.
*   **Constraints**:
    *   $0 \le N \le 10^5$ (Number of nodes in the tree)
    *   $-10^9 \le \text{Node value} \le 10^9$

*   **Observations**:
    *   Since the DLL must be sorted, we must perform an inorder traversal (Left $\rightarrow$ Root $\rightarrow$ Right).
    *   We need to link the current node to the previously visited node in our traversal.
    *   Left pointer `left` becomes `prev`.
    *   Right pointer `right` becomes `next`.
    *   We must keep track of a global/reference pointer `prevNode` to establish these bidirectional links.


### 2. Brute Solution

*   **Algorithm**:
    *   Perform inorder traversal to collect nodes in a `std::vector<TreeNode*> nodes`.
    *   If vector is empty, return `nullptr`.
    *   Loop from `i = 0` to `nodes.size() - 1`:
        *   `nodes[i]->left = (i == 0) ? nullptr : nodes[i-1]`.
        *   `nodes[i]->right = (i == nodes.size() - 1) ? nullptr : nodes[i+1]`.
    *   Return `nodes[0]`.
*   **Complexity**:
    *   **Time Complexity**: $O(N)$
    *   **Space Complexity**: $O(N)$ to store pointers in the vector.
*   **Pros/Cons**:
    *   *Pros*: Easy to reason about, no complex pointer side-effects during traversal.
    *   *Cons*: Fails the "strict in-place space constraint" if $O(N)$ extra space is forbidden.

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

void collectNodes(TreeNode* root, std::vector<TreeNode*>& nodes) {
    if (!root) return;
    collectNodes(root->left, nodes);
    nodes.push_back(root);
    collectNodes(root->right, nodes);
}

TreeNode* bstToDLLBrute(TreeNode* root) {
    if (!root) return nullptr;
    std::vector<TreeNode*> nodes;
    collectNodes(root, nodes);

    int n = nodes.size();
    for (int i = 0; i < n; ++i) {
        nodes[i]->left = (i == 0) ? nullptr : nodes[i - 1];
        nodes[i]->right = (i == n - 1) ? nullptr : nodes[i + 1];
    }
    return nodes[0];
}
```


### 3. Optimal Solution & Complexities

*   **Approach**: In-place recursive inorder traversal using a tracking pointer `prevNode`.

### Algorithm
1.  Define a helper function `convertToDLL(TreeNode* curr, TreeNode*& prevNode, TreeNode*& headNode)`.
2.  If `curr` is `nullptr`, return.
3.  Recursively convert left subtree: `convertToDLL(curr->left, prevNode, headNode)`.
4.  Process current node `curr`:
    *   If `prevNode` is `nullptr`, this is the leftmost (smallest) node. Set `headNode = curr`.
    *   Else, set `curr->left = prevNode` and `prevNode->right = curr`.
    *   Update `prevNode = curr`.
5.  Recursively convert right subtree: `convertToDLL(curr->right, prevNode, headNode)`.
6.  In the main function:
    *   Initialize `prevNode = nullptr`, `headNode = nullptr`.
    *   Call `convertToDLL(root, prevNode, headNode)`.
    *   Return `headNode`.

### Optimal C++17 Code
```cpp
#include <iostream>

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
     * Helper function to convert BST to DLL in-place.
     * @param curr Current node being processed
     * @param prevNode Reference to the pointer of the previously visited node
     * @param headNode Reference to the pointer of the DLL head node
     */
    void convertToDLL(TreeNode* curr, TreeNode*& prevNode, TreeNode*& headNode) {
        if (curr == nullptr) {
            return;
        }

        // 1. Recurse Left
        convertToDLL(curr->left, prevNode, headNode);

        // 2. Process Current Node
        if (prevNode == nullptr) {
            // If prevNode is null, we are at the leftmost node (smallest value)
            // This is the head of our Doubly Linked List
            headNode = curr;
        } else {
            // Establish link between prevNode and current node
            curr->left = prevNode;       // curr->prev = prevNode
            prevNode->right = curr;     // prevNode->next = curr
        }
        
        // Update prevNode to current node before moving right
        prevNode = curr;

        // 3. Recurse Right
        convertToDLL(curr->right, prevNode, headNode);
    }

public:
    /**
     * Converts a Binary Search Tree to a sorted Doubly Linked List in-place.
     */
    TreeNode* bstToDoublyList(TreeNode* root) {
        if (root == nullptr) {
            return nullptr;
        }

        TreeNode* prevNode = nullptr;
        TreeNode* headNode = nullptr;

        convertToDLL(root, prevNode, headNode);

        // Optional: If you need to make it a Circular Doubly Linked List, 
        // link headNode and prevNode (which represents the tail at the end) here:
        // headNode->left = prevNode;
        // prevNode->right = headNode;

        return headNode;
    }
};
```


#### Complexity Summary

*   **Time Complexity**: $\mathcal{O}(N)$
    *   Every node in the tree is visited exactly once.
*   **Space Complexity**: $\mathcal{O}(H)$
    *   $H$ is the height of the tree.
    *   No extra memory is allocated for list nodes. The space is used by the recursion stack.
    *   For a balanced tree, $H = \log N$, resulting in $\mathcal{O}(\log N)$ space. For a skewed tree, $H = N$, resulting in $\mathcal{O}(N)$ space.


---

## Q22 Serialize Deserialize

### 1. Problem Explanation

Serialization is the process of converting a data structure or object into a sequence of bits so that it can be stored in a file or memory buffer, or transmitted across a network connection link to be reconstructed later in the same or another computer environment.

Design an algorithm to serialize and deserialize a binary tree. There is no restriction on how your serialization/deserialization algorithm should work. You just need to ensure that a binary tree can be serialized to a string and this string can be deserialized to the original tree structure.

*   **Input / Output**:
    *   `string serialize(TreeNode* root)`: Converts a binary tree to a string.
    *   `TreeNode* deserialize(string data)`: Reconstructs the binary tree from the string.
*   **Constraints**:
    *   The number of nodes in the tree is in the range $[0, 10^4]$.
    *   $-1000 \le \text{Node value} \le 1000$.

*   **Observations**:
    *   A normal traversal (like inorder or preorder alone) cannot uniquely reconstruct a binary tree if null pointers are omitted.
    *   However, if we explicitly include markers for `nullptr` (e.g., `#` or `null`) in our traversal representation, we can uniquely reconstruct the binary tree with just a single traversal sequence.
    *   Preorder traversal (Root $\rightarrow$ Left $\rightarrow$ Right) is ideal here because the root node is serialized first. When deserializing, we can construct the tree top-down, which is highly natural.


### 2. Brute Solution

*   *Note*: The standard BFS/Level-order traversal with queue can also be used to serialize/deserialize, but it requires tracking queue states during deserialization and is generally more verbose. DFS preorder with explicit null values is already the cleanest and most optimal solution in terms of code complexity and execution. There is no simpler brute force. We will jump directly to the optimal preorder traversal solution.


### 3. Optimal Solution & Complexities

### Optimal C++17 Code
```cpp
#include <iostream>
#include <string>
#include <sstream>
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

class Codec {
private:
    /**
     * Helper function to serialize tree recursively (Preorder).
     */
    void serializeHelper(TreeNode* root, std::stringstream& ss) {
        if (root == nullptr) {
            ss << "#,";
            return;
        }

        // Preorder: Root -> Left -> Right
        ss << root->val << ",";
        serializeHelper(root->left, ss);
        serializeHelper(root->right, ss);
    }

    /**
     * Helper function to deserialize tree recursively (Preorder).
     */
    TreeNode* deserializeHelper(std::queue<std::string>& q) {
        if (q.empty()) {
            return nullptr;
        }

        // Get the current token
        std::string val = q.front();
        q.pop();

        // If the token is '#', this indicates a null node
        if (val == "#") {
            return nullptr;
        }

        // Create the node (Root)
        TreeNode* root = new TreeNode(std::stoi(val));

        // Reconstruct subtrees (Left -> Right)
        root->left = deserializeHelper(q);
        root->right = deserializeHelper(q);

        return root;
    }

public:
    // Encodes a tree to a single string.
    std::string serialize(TreeNode* root) {
        std::stringstream ss;
        serializeHelper(root, ss);
        return ss.str();
    }

    // Decodes your encoded data to tree.
    TreeNode* deserialize(std::string data) {
        std::queue<std::string> q;
        std::stringstream ss(data);
        std::string token;

        // Split the string by comma and push tokens to queue
        while (std::getline(ss, token, ',')) {
            if (!token.empty()) {
                q.push(token);
            }
        }

        return deserializeHelper(q);
    }
};
```


#### Complexity Summary

*   **Time Complexity**: $\mathcal{O}(N)$
    *   During serialization, we visit each of the $N$ nodes (and their null pointers) once.
    *   During deserialization, we split the string in $\mathcal{O}(N)$ time and process each token exactly once.
*   **Space Complexity**: $\mathcal{O}(N)$
    *   For serialization: the output string and recursion stack use $\mathcal{O}(N)$ space.
    *   For deserialization: the queue of tokens and recursion stack use $\mathcal{O}(N)$ space.


---

## Q23 Max Path Sum

### 1. Problem Explanation

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


### 2. Brute Solution

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


### 3. Optimal Solution & Complexities

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


#### Complexity Summary

*   **Time Complexity**: $\mathcal{O}(N)$
    *   We perform a post-order traversal, visiting each node in the tree exactly once.
*   **Space Complexity**: $\mathcal{O}(H)$
    *   $H$ is the height of the tree.
    *   The space is occupied by the recursion stack.
    *   For a balanced tree, $H = \log N$. For a skewed tree, $H = N$.


---

## Q24 Boundary Traversal

### 1. Problem Explanation

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


### 2. Brute Solution

A naive way would be to perform a standard traversal (like level-order or DFS), capture the coordinates (x, y) of each node, and try to construct the boundary from the convex hull or external nodes. This is extremely overcomplicated and slow, requiring $O(N \log N)$ or $O(N^2)$ time with auxiliary coordinates.

Since brute force is not practical here, we jump directly to the standard modular solution, which is already highly intuitive.


### 3. Optimal Solution & Complexities

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


#### Complexity Summary

| Metric | Complexity | Reasoning |
| :--- | :--- | :--- |
| **Time Complexity** | $O(N)$ | We visit every node in the tree at most a constant number of times (leaves traversal visits all $N$ nodes, left and right boundaries visit at most height $H$ nodes). |
| **Space Complexity** | $O(H)$ | The recursion stack for `addLeaves` takes $O(H)$ space. The temporary array for the right boundary takes $O(H)$ space. $H$ is the height of the tree. |


---

## Q25 Count Complete Tree

### 1. Problem Explanation

Given the root of a **complete** binary tree, return the number of nodes in the tree.

According to Wikipedia, every level, except possibly the last, is completely filled in a complete binary tree, and all nodes in the last level are as far left as possible. It can have between $1$ and $2^h$ nodes inclusive at the last level $h$.

Design an algorithm that runs in less than $O(N)$ time complexity.

### Input
- The root of a complete binary tree.

### Output
- An integer representing the total count of nodes.

### Constraints
- The number of nodes in the tree is in the range $[0, 5 \cdot 10^4]$.
- $0 \le \text{Node.val} \le 5 \cdot 10^4$.
- The tree is guaranteed to be **complete**.


### 2. Brute Solution

A simple traversal of the tree using DFS or BFS to count all nodes.

### Algorithm
1. If root is null, return 0.
2. Return $1 + \text{countNodes}(root->left) + \text{countNodes}(root->right)$.

### C++17 Code
```cpp
class SolutionBrute {
public:
    int countNodes(TreeNode* root) {
        if (root == nullptr) return 0;
        return 1 + countNodes(root->left) + countNodes(root->right);
    }
};
```

### Complexity Analysis
- **Time Complexity:** $O(N)$ because we visit every single node.
- **Space Complexity:** $O(H)$ where $H$ is the height of the tree (call stack). For a complete tree, $H = O(\log N)$.


### 3. Optimal Solution & Complexities

### Algorithm
1. Write helper functions to compute the left height and right height of a node:
   - For left height: traverse `curr = curr->left` and count.
   - For right height: traverse `curr = curr->right` and count.
2. In the main function `countNodes(root)`:
   - If `root` is null, return 0.
   - Calculate left height `lh` and right height `rh`.
   - If `lh == rh`, the tree is a perfect binary tree. Return $(1 \ll lh) - 1$.
   - If `lh != rh`, return $1 + \text{countNodes}(root->left) + \text{countNodes}(root->right)$.

### C++17 Code

```cpp
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
    // Helper to find the left-most height (1-based)
    int getLeftHeight(TreeNode* node) {
        int height = 0;
        while (node != nullptr) {
            height++;
            node = node->left;
        }
        return height;
    }

    // Helper to find the right-most height (1-based)
    int getRightHeight(TreeNode* node) {
        int height = 0;
        while (node != nullptr) {
            height++;
            node = node->right;
        }
        return height;
    }

public:
    int countNodes(TreeNode* root) {
        if (root == nullptr) {
            return 0;
        }

        int lh = getLeftHeight(root);
        int rh = getRightHeight(root);

        // If left height and right height are equal, it's a perfect binary tree.
        if (lh == rh) {
            // Number of nodes is (2^lh) - 1
            // 1 << lh is equivalent to 2^lh
            return (1 << lh) - 1;
        }

        // Otherwise, recurse on left and right subtrees
        return 1 + countNodes(root->left) + countNodes(root->right);
    }
};
```


#### Complexity Summary

| Metric | Complexity | Reasoning |
| :--- | :--- | :--- |
| **Time Complexity** | $O(\log^2 N)$ | In each step, we calculate height which takes $O(\log N)$ time. We only recurse down one path because one of the subtrees is always perfect and returns in $O(1)$ time. Thus, recursion depth is $O(\log N)$. Total time: $O(\log N \cdot \log N) = O(\log^2 N)$. |
| **Space Complexity** | $O(\log N)$ | The recursion stack height is at most $O(\log N)$ (since the tree is complete). |


---

## Q26 Flatten To LL

### 1. Problem Explanation

Given the root of a binary tree, flatten the tree into a "linked list":
- The "linked list" should use the same `TreeNode` class where the `right` child pointer points to the next node in the list and the `left` child pointer is always `nullptr`.
- The "linked list" should be in the same order as a **preorder traversal** of the binary tree.

### Input
- The root of a binary tree.

### Output
- Modify the tree in-place; do not return anything.

### Constraints
- The number of nodes in the tree is in the range $[0, 2000]$.
- $-100 \le \text{Node.val} \le 100$.


### 2. Brute Solution

### Algorithm
1. Perform standard preorder DFS. Store node pointers in a vector.
2. Iterate through the vector:
   - Set current node's `left` to `nullptr`.
   - Set current node's `right` to the next node in the vector.

### C++17 Code
```cpp
#include <vector>

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class SolutionBrute {
private:
    void preorder(TreeNode* root, std::vector<TreeNode*>& nodes) {
        if (root == nullptr) return;
        nodes.push_back(root);
        preorder(root->left, nodes);
        preorder(root->right, nodes);
    }
public:
    void flatten(TreeNode* root) {
        if (root == nullptr) return;
        std::vector<TreeNode*> nodes;
        preorder(root, nodes);
        
        for (size_t i = 0; i < nodes.size() - 1; ++i) {
            nodes[i]->left = nullptr;
            nodes[i]->right = nodes[i + 1];
        }
        nodes.back()->left = nullptr;
        nodes.back()->right = nullptr;
    }
};
```

### Complexity Analysis
- **Time Complexity:** $O(N)$ since we visit every node twice.
- **Space Complexity:** $O(N)$ for the vector storing node pointers.


### 3. Optimal Solution & Complexities

### Algorithm
1. Initialize a pointer `curr = root`.
2. While `curr` is not null:
   - If `curr` has a left child:
     - Find the rightmost node in `curr`'s left subtree. Let this node be `pred`.
     - Connect `pred->right = curr->right`.
     - Move the left subtree to the right: `curr->right = curr->left`.
     - Set `curr->left = nullptr`.
   - Move to the next node: `curr = curr->right`.

### C++17 Code

```cpp
#include <iostream>

class Solution {
public:
    void flatten(TreeNode* root) {
        TreeNode* curr = root;
        while (curr != nullptr) {
            if (curr->left != nullptr) {
                // Find the rightmost node in the left subtree
                TreeNode* pred = curr->left;
                while (pred->right != nullptr) {
                    pred = pred->right;
                }
                
                // Connect the predecessor's right to curr's right
                pred->right = curr->right;
                
                // Move left subtree to the right
                curr->right = curr->left;
                curr->left = nullptr;
            }
            // Move to the next node in the flattened chain
            curr = curr->right;
        }
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(N)$ | $O(N)$ | Storing preorder nodes in a vector. |
| **Recursive (Better)**| $O(N)$ | $O(H)$ | Recursion stack height equal to tree height. |
| **Morris-like (Optimal)**| $O(N)$ | $O(1)$ | We visit each edge at most a constant number of times and use only temporary pointer variables. |


---

## Q27 Construct Inorder Preorder

### 1. Problem Explanation

Given two integer arrays `preorder` and `inorder` where `preorder` is the preorder traversal of a binary tree and `inorder` is the inorder traversal of the same tree, construct and return the binary tree.

### Input
- `preorder`: A vector of integers.
- `inorder`: A vector of integers.

### Output
- The root of the constructed binary tree.

### Constraints
- $1 \le \text{preorder.length} \le 3000$.
- `inorder.length == preorder.length`.
- $-3000 \le \text{preorder[i]}, \text{inorder[i]} \le 3000$.
- `preorder` and `inorder` consist of **unique** values.
- Each value of `inorder` also appears in `preorder`.
- `preorder` is guaranteed to be the preorder traversal of the tree.
- `inorder` is guaranteed to be the inorder traversal of the tree.


### 2. Brute Solution

The brute force solution is to search for the root index linearly inside `inorder` at each recursive step, and make copies of the sub-arrays.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class SolutionBrute {
public:
    TreeNode* buildTree(std::vector<int>& preorder, std::vector<int>& inorder) {
        if (preorder.empty() || inorder.empty()) {
            return nullptr;
        }
        
        int rootVal = preorder[0];
        TreeNode* root = new TreeNode(rootVal);
        
        // Find index linearly
        auto it = std::find(inorder.begin(), inorder.end(), rootVal);
        int index = std::distance(inorder.begin(), it);
        
        // Split arrays
        std::vector<int> leftInorder(inorder.begin(), inorder.begin() + index);
        std::vector<int> rightInorder(inorder.begin() + index + 1, inorder.end());
        
        std::vector<int> leftPreorder(preorder.begin() + 1, preorder.begin() + 1 + index);
        std::vector<int> rightPreorder(preorder.begin() + 1 + index, preorder.end());
        
        root->left = buildTree(leftPreorder, leftInorder);
        root->right = buildTree(rightPreorder, rightInorder);
        
        return root;
    }
};
```

### Complexity Analysis
- **Time Complexity:** $O(N^2)$ due to linear search and vector copies.
- **Space Complexity:** $O(N^2)$ due to creating new sub-vectors at every recursive step.


### 3. Optimal Solution & Complexities

Using indices and a hash map to lookup values in the `inorder` array in $O(1)$ time.

### C++17 Code

```cpp
#include <vector>
#include <unordered_map>

class Solution {
private:
    TreeNode* build(const std::vector<int>& preorder, int preStart, int preEnd,
                    const std::vector<int>& inorder, int inStart, int inEnd,
                    const std::unordered_map<int, int>& inMap) {
        // Base case: empty ranges
        if (preStart > preEnd || inStart > inEnd) {
            return nullptr;
        }

        // 1. The first element in current preorder range is the root
        int rootVal = preorder[preStart];
        TreeNode* root = new TreeNode(rootVal);

        // 2. Locate the root value in inorder to partition subtrees
        int inRoot = inMap.at(rootVal);
        
        // 3. Count elements in left subtree
        int numsLeft = inRoot - inStart;

        // 4. Recurse for Left and Right subtrees
        root->left = build(preorder, preStart + 1, preStart + numsLeft,
                           inorder, inStart, inRoot - 1, inMap);
                           
        root->right = build(preorder, preStart + numsLeft + 1, preEnd,
                            inorder, inRoot + 1, inEnd, inMap);

        return root;
    }

public:
    TreeNode* buildTree(std::vector<int>& preorder, std::vector<int>& inorder) {
        // Map inorder values to their indices for O(1) lookup
        std::unordered_map<int, int> inMap;
        for (int i = 0; i < static_cast<int>(inorder.size()); ++i) {
            inMap[inorder[i]] = i;
        }

        return build(preorder, 0, static_cast<int>(preorder.size()) - 1,
                     inorder, 0, static_cast<int>(inorder.size()) - 1,
                     inMap);
    }
};
```


#### Complexity Summary

| Metric | Complexity | Reasoning |
| :--- | :--- | :--- |
| **Time Complexity** | $O(N)$ | Building the hash map takes $O(N)$ time. We create exactly $N$ nodes, each taking $O(1)$ work because the root lookup in inorder is $O(1)$. |
| **Space Complexity** | $O(N)$ | The hash map occupies $O(N)$ auxiliary space. The recursive call stack takes $O(H)$ space, which is $O(N)$ in the worst case (skewed tree). |


---

## Q28 Construct Inorder Postorder

### 1. Problem Explanation

Given two integer arrays `inorder` and `postorder` where `inorder` is the inorder traversal of a binary tree and `postorder` is the postorder traversal of the same tree, construct and return the binary tree.

### Input
- `inorder`: A vector of integers.
- `postorder`: A vector of integers.

### Output
- The root of the constructed binary tree.

### Constraints
- $1 \le \text{inorder.length} \le 3000$.
- `postorder.length == inorder.length`.
- $-3000 \le \text{inorder[i]}, \text{postorder[i]} \le 3000$.
- `inorder` and `postorder` consist of **unique** values.
- Each value of `postorder` also appears in `inorder`.
- `inorder` is guaranteed to be the inorder traversal of the tree.
- `postorder` is guaranteed to be the postorder traversal of the tree.


### 2. Brute Solution

Construct by splitting vectors via copy operations and scanning linearly for the root index.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class SolutionBrute {
public:
    TreeNode* buildTree(std::vector<int>& inorder, std::vector<int>& postorder) {
        if (inorder.empty() || postorder.empty()) {
            return nullptr;
        }

        int rootVal = postorder.back();
        TreeNode* root = new TreeNode(rootVal);

        auto it = std::find(inorder.begin(), inorder.end(), rootVal);
        int index = std::distance(inorder.begin(), it);

        std::vector<int> leftInorder(inorder.begin(), inorder.begin() + index);
        std::vector<int> rightInorder(inorder.begin() + index + 1, inorder.end());

        std::vector<int> leftPostorder(postorder.begin(), postorder.begin() + index);
        std::vector<int> rightPostorder(postorder.begin() + index, postorder.end() - 1);

        root->left = buildTree(leftInorder, leftPostorder);
        root->right = buildTree(rightInorder, rightPostorder);

        return root;
    }
};
```

### Complexity Analysis
- **Time Complexity:** $O(N^2)$ due to linear searches and subarray copy operations.
- **Space Complexity:** $O(N^2)$ due to vector allocations in recursion.


### 3. Optimal Solution & Complexities

Using index-based boundary arguments and a hash map for $O(1)$ index lookup of inorder elements.

### C++17 Code

```cpp
#include <vector>
#include <unordered_map>

class Solution {
private:
    TreeNode* build(const std::vector<int>& inorder, int inStart, int inEnd,
                    const std::vector<int>& postorder, int postStart, int postEnd,
                    const std::unordered_map<int, int>& inMap) {
        // Base case: empty boundaries
        if (inStart > inEnd || postStart > postEnd) {
            return nullptr;
        }

        // 1. The last element in current postorder range is the root
        int rootVal = postorder[postEnd];
        TreeNode* root = new TreeNode(rootVal);

        // 2. Locate root index in inorder array
        int inRoot = inMap.at(rootVal);

        // 3. Count elements in the left subtree
        int numsLeft = inRoot - inStart;

        // 4. Recurse for Left and Right subtrees
        root->left = build(inorder, inStart, inRoot - 1,
                           postorder, postStart, postStart + numsLeft - 1,
                           inMap);
                           
        root->right = build(inorder, inRoot + 1, inEnd,
                            postorder, postStart + numsLeft, postEnd - 1,
                            inMap);

        return root;
    }

public:
    TreeNode* buildTree(std::vector<int>& inorder, std::vector<int>& postorder) {
        // Build map for O(1) index queries
        std::unordered_map<int, int> inMap;
        for (int i = 0; i < static_cast<int>(inorder.size()); ++i) {
            inMap[inorder[i]] = i;
        }

        return build(inorder, 0, static_cast<int>(inorder.size()) - 1,
                     postorder, 0, static_cast<int>(postorder.size()) - 1,
                     inMap);
    }
};
```


#### Complexity Summary

| Metric | Complexity | Reasoning |
| :--- | :--- | :--- |
| **Time Complexity** | $O(N)$ | Building `inMap` takes $O(N)$ time. The recursive function runs $N$ times, each taking $O(1)$ work because index lookup in the map is $O(1)$. |
| **Space Complexity** | $O(N)$ | The index map uses $O(N)$ memory. The recursion stack uses $O(H)$ space, which is $O(N)$ in the worst case (skewed tree). |


---

## Q29 Segment Tree

### 1. Problem Explanation

Implement a **Segment Tree** to support efficient range queries and point updates on an array of numbers. 
Specifically, design a Segment Tree for **Range Sum Queries**:
- **Build**: Construct the tree from a given array of size $N$.
- **Query**: Get the sum of elements in a range $[L, R]$ (inclusive).
- **Update**: Modify a single element of the array at index $idx$ to a new value $val$, and update the tree accordingly.

### Operations and Complexities
- **Build**: $O(N)$
- **Query**: $O(\log N)$
- **Update**: $O(\log N)$

### Constraints
- $1 \le N \le 10^5$.
- Node values: $-10^4 \le \text{arr[i]} \le 10^4$.
- Number of queries/updates: $1 \le Q \le 10^5$.


### 2. Brute Solution

Using a simple array.
- **Query**: Loop from $L$ to $R$ and sum.
- **Update**: Directly modify `arr[idx] = val`.

### Complexity Analysis
- **Time Complexity**: Build: $O(1)$, Query: $O(N)$, Update: $O(1)$.
- **Space Complexity**: $O(N)$ to store the array.
- **Downside**: Extremely slow for a large number of queries.


### 3. Optimal Solution & Complexities

Using a Segment Tree.

### C++17 Code

```cpp
#include <vector>
#include <iostream>

class SegmentTree {
private:
    int n;
    std::vector<int> tree;

    // Helper to build the segment tree
    void build(const std::vector<int>& arr, int node, int start, int end) {
        // Base Case: Leaf node stores single element of array
        if (start == end) {
            tree[node] = arr[start];
            return;
        }

        int mid = start + (end - start) / 2;
        int leftChild = 2 * node;
        int rightChild = 2 * node + 1;

        // Recursively build left and right subtrees
        build(arr, leftChild, start, mid);
        build(arr, rightChild, mid + 1, end);

        // Internal node stores the sum of its children
        tree[node] = tree[leftChild] + tree[rightChild];
    }

    // Helper for range query
    int queryRange(int node, int start, int end, int L, int R) {
        // Case 1: No overlap
        if (R < start || L > end) {
            return 0; // Identity element for sum
        }

        // Case 2: Complete overlap
        if (L <= start && end <= R) {
            return tree[node];
        }

        // Case 3: Partial overlap
        int mid = start + (end - start) / 2;
        int leftSum = queryRange(2 * node, start, mid, L, R);
        int rightSum = queryRange(2 * node + 1, mid + 1, end, L, R);

        return leftSum + rightSum;
    }

    // Helper for point update
    void updatePoint(int node, int start, int end, int idx, int val) {
        // Base Case: Reached leaf node to update
        if (start == end) {
            tree[node] = val;
            return;
        }

        int mid = start + (end - start) / 2;
        int leftChild = 2 * node;
        int rightChild = 2 * node + 1;

        if (idx <= mid) {
            // Update left child
            updatePoint(leftChild, start, mid, idx, val);
        } else {
            // Update right child
            updatePoint(rightChild, mid + 1, end, idx, val);
        }

        // Recalculate parent value after child update
        tree[node] = tree[leftChild] + tree[rightChild];
    }

public:
    // Constructor
    SegmentTree(const std::vector<int>& arr) {
        n = static_cast<int>(arr.size());
        // Size of tree is at most 4 * n
        tree.resize(4 * n, 0);
        if (n > 0) {
            build(arr, 1, 0, n - 1);
        }
    }

    // Public Query API
    int query(int L, int R) {
        return queryRange(1, 0, n - 1, L, R);
    }

    // Public Update API
    void update(int idx, int val) {
        updatePoint(1, 0, n - 1, idx, val);
    }
};
```


#### Complexity Summary

| Operation | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Build** | $O(N)$ | $O(N)$ | We visit every node in the Segment Tree once. The number of nodes is $< 4N$. |
| **Query** | $O(\log N)$ | $O(\log N)$ | At each level of the tree, we visit at most 4 nodes. The height of the tree is $O(\log N)$. |
| **Update** | $O(\log N)$ | $O(\log N)$ | We traverse down a single path from the root to the leaf node of length $O(\log N)$. |


---

## Q30 Fenwick Tree BIT

### 1. Problem Explanation

Implement a **Fenwick Tree** (also known as a **Binary Indexed Tree - BIT**) to support:
- **Point Update**: Add a value `delta` to the element at a given index `i`.
- **Prefix Sum Query**: Compute the sum of the elements from the beginning of the array up to index `i` (inclusive).
- **Range Sum Query**: Compute the sum of the elements in a range $[L, R]$ (inclusive).

All operations (excluding range sum which uses two prefix queries) must run in $O(\log N)$ time. The implementation should be 1-indexed internally.

### Input
- Array size $N$.
- A series of update and query requests.

### Constraints
- $1 \le N \le 10^5$.
- values: $-10^5 \le \text{val} \le 10^5$.


### 2. Brute Solution

Using a standard array:
- **Update**: `arr[i] += delta` $\Rightarrow O(1)$
- **Prefix Query**: Loop from $0$ to $i$ summing elements $\Rightarrow O(N)$


### 3. Optimal Solution & Complexities

### C++17 Code

```cpp
#include <vector>
#include <iostream>

class FenwickTree {
private:
    int size;
    std::vector<int> tree;

public:
    // Constructor initialized with size N
    FenwickTree(int n) {
        this->size = n;
        // 1-indexed tree of size N + 1
        tree.assign(n + 1, 0);
    }

    // Constructor initialized with a given 0-indexed array
    FenwickTree(const std::vector<int>& arr) : FenwickTree(static_cast<int>(arr.size())) {
        for (int i = 0; i < static_cast<int>(arr.size()); ++i) {
            update(i, arr[i]);
        }
    }

    // 1-indexed Point Update: Add delta to index 'i' (0-indexed externally)
    void update(int i, int delta) {
        int idx = i + 1; // Convert to 1-based index
        while (idx <= size) {
            tree[idx] += delta;
            idx += idx & (-idx); // Add lowbit
        }
    }

    // 1-indexed Prefix Sum Query: sum from index 0 to 'i' (0-indexed externally)
    int query(int i) const {
        int idx = i + 1; // Convert to 1-based index
        int sum = 0;
        while (idx > 0) {
            sum += tree[idx];
            idx -= idx & (-idx); // Subtract lowbit
        }
        return sum;
    }

    // Range Sum Query: sum from index L to R (inclusive)
    int queryRange(int L, int R) const {
        if (L > R) return 0;
        return query(R) - query(L - 1);
    }
};
```


#### Complexity Summary

| Operation | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Point Update** | $O(\log N)$ | $O(N)$ | The number of set bits in any integer $\le N$ is at most $O(\log N)$. |
| **Prefix Query** | $O(\log N)$ | $O(N)$ | Similar to update, we clear bits at each step, taking at most $O(\log N)$ steps. |
| **Space Complexity** | — | $O(N)$ | Uses an array of size $N+1$ to store segment data. |


---

## Q31 Trie Insert Search

### 1. Problem Explanation

Implement a **Trie (Prefix Tree)** data structure. A Trie is a tree-like data structure used to efficiently store and retrieve keys in a dataset of strings. 

You need to implement the `Trie` class with the following methods:
- `void insert(string word)`: Inserts the string `word` into the trie.
- `bool search(string word)`: Returns `true` if the string `word` is in the trie (i.e., was inserted before), and `false` otherwise.
- `bool startsWith(string prefix)`: Returns `true` if there is a previously inserted string `word` that has the prefix `prefix`, and `false` otherwise.

### Constraints
- $1 \le \text{word.length, prefix.length} \le 2000$.
- `word` and `prefix` consist only of lowercase English letters.
- At most $3 \cdot 10^4$ calls in total will be made to `insert`, `search`, and `startsWith`.


### 2. Brute Solution

Using a simple list or set of strings.
- **Insert**: Insert into `std::unordered_set<std::string>`.
- **Search**: `set.count(word)`.
- **StartsWith**: Iterate through all strings in the set and check if they start with the prefix.

### Complexity Analysis
- **Time Complexity:** 
  - `insert`: $O(L)$
  - `search`: $O(L)$
  - `startsWith`: $O(N \cdot L)$ where $N$ is the number of words.
- **Downside:** `startsWith` is extremely slow when the dictionary is large.


### 3. Optimal Solution & Complexities

### C++17 Code

```cpp
#include <string>
#include <vector>
#include <iostream>

class TrieNode {
public:
    TrieNode* children[26];
    bool isEndOfWord;

    TrieNode() {
        isEndOfWord = false;
        for (int i = 0; i < 26; ++i) {
            children[i] = nullptr;
        }
    }

    ~TrieNode() {
        for (int i = 0; i < 26; ++i) {
            delete children[i];
        }
    }
};

class Trie {
private:
    TrieNode* root;

    // Helper to find node matching a string path
    TrieNode* findNode(const std::string& str) const {
        TrieNode* curr = root;
        for (char c : str) {
            int idx = c - 'a';
            if (curr->children[idx] == nullptr) {
                return nullptr;
            }
            curr = curr->children[idx];
        }
        return curr;
    }

public:
    Trie() {
        root = new TrieNode();
    }

    ~Trie() {
        delete root;
    }

    // Inserts a word into the trie
    void insert(std::string word) {
        TrieNode* curr = root;
        for (char c : word) {
            int idx = c - 'a';
            if (curr->children[idx] == nullptr) {
                curr->children[idx] = new TrieNode();
            }
            curr = curr->children[idx];
        }
        curr->isEndOfWord = true;
    }

    // Returns true if the word is in the trie
    bool search(std::string word) const {
        TrieNode* node = findNode(word);
        return (node != nullptr) && node->isEndOfWord;
    }

    // Returns true if there is any word in the trie that starts with the given prefix
    bool startsWith(std::string prefix) const {
        return findNode(prefix) != nullptr;
    }
};
```


#### Complexity Summary

| Operation | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Insert** | $O(L)$ | $O(L \times \Sigma)$ | Where $L$ is the word length and $\Sigma$ is the alphabet size (26). We allocate at most $L$ nodes. |
| **Search** | $O(L)$ | $O(1)$ | We only traverse existing node paths without allocating memory. |
| **StartsWith** | $O(L)$ | $O(1)$ | Similar to search, we traverse the prefix path. |


---

## Q32 Trie Longest Prefix

### 1. Problem Explanation

Given a dictionary of words and a query string, find the **longest prefix** of the query string that exists in the dictionary. 

If no prefix of the query string is present in the dictionary, return an empty string.

### Input
- `dictionary`: A list of strings.
- `query`: A string.

### Output
- A string representing the longest prefix match.

### Constraints
- $1 \le \text{dictionary.length} \le 10^4$.
- $1 \le \text{word.length} \le 100$.
- $1 \le \text{query.length} \le 1000$.
- All strings consist of lowercase English letters.


### 2. Brute Solution

Iterating through all prefixes of the query string and scanning the dictionary.

### C++17 Code
```cpp
#include <string>
#include <vector>
#include <algorithm>

class SolutionBrute {
public:
    std::string longestPrefix(const std::vector<std::string>& dictionary, const std::string& query) {
        std::string result = "";
        for (int i = 1; i <= static_cast<int>(query.length()); ++i) {
            std::string prefix = query.substr(0, i);
            if (std::find(dictionary.begin(), dictionary.end(), prefix) != dictionary.end()) {
                result = prefix;
            }
        }
        return result;
    }
};
```

### Complexity Analysis
- **Time Complexity:** $O(N \cdot L^2)$ where $N = \text{dictionary.size()}$ and $L = \text{query.length()}$.
- **Space Complexity:** $O(L)$ for the substring copies.


### 3. Optimal Solution & Complexities

### C++17 Code

```cpp
#include <string>
#include <vector>
#include <iostream>

class TrieNode {
public:
    TrieNode* children[26];
    bool isEndOfWord;

    TrieNode() {
        isEndOfWord = false;
        for (int i = 0; i < 26; ++i) {
            children[i] = nullptr;
        }
    }

    ~TrieNode() {
        for (int i = 0; i < 26; ++i) {
            delete children[i];
        }
    }
};

class Trie {
private:
    TrieNode* root;

public:
    Trie() {
        root = new TrieNode();
    }

    ~Trie() {
        delete root;
    }

    // Insert a word into the Trie
    void insert(const std::string& word) {
        TrieNode* curr = root;
        for (char c : word) {
            int idx = c - 'a';
            if (curr->children[idx] == nullptr) {
                curr->children[idx] = new TrieNode();
            }
            curr = curr->children[idx];
        }
        curr->isEndOfWord = true;
    }

    // Finds the longest prefix of the query that matches a complete word in the Trie
    std::string findLongestPrefix(const std::string& query) const {
        TrieNode* curr = root;
        int longestMatchIndex = -1; // -1 denotes no match found yet
        
        for (int i = 0; i < static_cast<int>(query.length()); ++i) {
            int idx = query[i] - 'a';
            if (curr->children[idx] == nullptr) {
                break; // No further match possible
            }
            curr = curr->children[idx];
            if (curr->isEndOfWord) {
                longestMatchIndex = i; // Update with the index of the last valid word character
            }
        }

        if (longestMatchIndex == -1) {
            return "";
        }
        return query.substr(0, longestMatchIndex + 1);
    }
};

class Solution {
public:
    std::string longestPrefixMatching(const std::vector<std::string>& dictionary, const std::string& query) {
        Trie trie;
        for (const std::string& word : dictionary) {
            trie.insert(word);
        }
        return trie.findLongestPrefix(query);
    }
};
```


#### Complexity Summary

| Operation / Phase | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Trie Build** | $O(N \cdot W)$ | $O(N \cdot W \cdot \Sigma)$ | Where $N$ is dictionary size, $W$ is average word length, and $\Sigma = 26$. |
| **Prefix Query** | $O(L)$ | $O(1)$ | Where $L$ is the length of the query string. We only traverse the query string once. |


---

## Q33 Symmetric Tree

### 1. Problem Explanation

Given the root of a binary tree, check whether it is a mirror of itself (i.e., symmetric around its center).

### Input
- The root of a binary tree.

### Output
- A boolean: `true` if the tree is symmetric, `false` otherwise.

### Constraints
- The number of nodes in the tree is in the range $[1, 1000]$.
- $-100 \le \text{Node.val} \le 100$.


### 2. Brute Solution

There is no "naive" brute force other than converting the tree to structural lists and verifying mirroring, which is messy. The recursive solution is the natural and most direct approach. Thus, we jump directly to the optimal implementations.


### 3. Optimal Solution & Complexities

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


#### Complexity Summary

| Metric | Complexity | Reasoning |
| :--- | :--- | :--- |
| **Time Complexity** | $O(N)$ | We visit every node in the tree exactly once. |
| **Space Complexity** | $O(H)$ | The height of the recursion stack is at most $O(H)$, where $H$ is the height of the tree. In the worst case (skewed tree), $H = O(N)$; in the best/balanced case, $H = O(\log N)$. |


---
