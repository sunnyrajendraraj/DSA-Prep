# Diameter of a Binary Tree

## 1. Problem Statement

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

---

## 2. Interview Clarification

Before writing any code, clarify:
1. **Edges vs. Nodes**: "Is the diameter defined by the number of edges or the number of nodes?"
   - *Interviewer response:* Edges. So a tree with two connected nodes has a diameter of 1.
2. **Path through Root**: "Does the longest path have to pass through the root?"
   - *Interviewer response:* No. The longest path can reside entirely within the left or right subtrees.
3. **Empty Tree / Single Node**: "What is the diameter of an empty tree or a single node?"
   - *Interviewer response:* The diameter of a single node or empty tree is 0.

---

## 3. Thinking Process

### First Instinct (Brute Force Approach)
- The longest path passing through node $X$ is $\text{height}(X\text{.left}) + \text{height}(X\text{.right})$.
- We can compute this value for every node in the tree.
- For each node:
  1. Compute height of its left subtree.
  2. Compute height of its right subtree.
  3. Calculate path length: $\text{leftHeight} + \text{rightHeight}$.
  4. Recurse for the left child and right child to find their respective local diameters.
  5. The global diameter is the maximum of all these local diameters.
- This is inefficient because we recalculate the height of the same nodes multiple times.

### Transition to Optimal Solution
- Can we find the height and diameter in a single pass?
- Yes, while computing the height of a subtree recursively, we already have access to the heights of the left and right children.
- At each node, we can:
  1. Calculate the left subtree height ($L$) and right subtree height ($R$).
  2. Update a global diameter tracker: `diameter = max(diameter, L + R)`.
  3. Return the height of the current node: `1 + max(L, R)` to the caller.
- This lets us compute both values simultaneously in a single bottom-up DFS postorder pass.

---

## 4. Pattern Recognition

- **Category**: Binary Tree Bottom-Up DFS.
- **Pattern**: State accumulation during recursive traversal.
- **Why**: Many tree properties (like diameter, balance, sum path) require combining child states. Instead of separate passes, we return one value (height) and update another (diameter) via a referenced variable or helper struct.

---

## 5. Brute Force Solution

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

---

## 6. Better Solution

*Note: Since the brute force jumps directly to the single-pass optimal solution by adding a tracker variable, there is no intermediate "better" solution.*

---

## 7. Optimal Solution (Single-Pass DFS)

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

---

### Why this is Optimal
This solution visits each node exactly once. It computes node heights bottom-up and updates the diameter in-place, eliminating redundant traversal and keeping the runtime strictly $\mathcal{O}(N)$.

---

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

---

## 8. Code Walkthrough

- `int leftHeight = calculateHeight(node->left, diameter);`: Computes left height first.
- `int rightHeight = calculateHeight(node->right, diameter);`: Computes right height.
- `diameter = std::max(diameter, leftHeight + rightHeight);`: This is the critical optimization. The number of edges on the path using the current node as the peak is `leftHeight + rightHeight`. If this is larger than the previously observed max path, update our tracker.
- `return 1 + std::max(leftHeight, rightHeight);`: Continues returning the height of the current node back up the tree.

---

## 9. Dry Run

Let's dry run the optimal solution with this tree:
```
        1
       / \
      2   3
     / \
    4   5
```

- Initialize `diameter = 0`.
- Call `calculateHeight(1)`:
  - Call `calculateHeight(2)`:
    - Call `calculateHeight(4)`:
      - Left = 0, Right = 0.
      - `diameter = max(0, 0 + 0) = 0`.
      - Returns `1`.
    - Call `calculateHeight(5)`:
      - Left = 0, Right = 0.
      - `diameter = max(0, 0 + 0) = 0`.
      - Returns `1`.
    - Back to `2`:
      - LeftHeight = 1, RightHeight = 1.
      - `diameter = max(0, 1 + 1) = 2`.
      - Returns `1 + max(1, 1) = 2`.
  - Call `calculateHeight(3)`:
    - Left = 0, Right = 0.
    - `diameter = max(2, 0 + 0) = 2`.
    - Returns `1`.
  - Back to `1`:
    - LeftHeight = 2 (from node 2), RightHeight = 1 (from node 3).
    - `diameter = max(2, 2 + 1) = 3`.
    - Returns `1 + max(2, 1) = 3`.
- Final `diameter = 3`. (Path is `4 -> 2 -> 1 -> 3` or `5 -> 2 -> 1 -> 3`, length = 3 edges).

---

## 10. Complexity Analysis

| Metric | Brute Force | Optimal DFS |
|:---|:---|:---|
| **Time Complexity (Best)** | $\mathcal{O}(N \log N)$ | $\mathcal{O}(N)$ |
| **Time Complexity (Worst)**| $\mathcal{O}(N^2)$ (skewed tree) | $\mathcal{O}(N)$ |
| **Space Complexity**| $\mathcal{O}(H)$ stack frames | $\mathcal{O}(H)$ stack frames |
| **Space Complexity (Worst)**| $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |

---

## 11. Common Mistakes

1. **Incorrect Edge Counting**: Writing `leftHeight + rightHeight + 1` for the diameter path length. That counts the *number of nodes* in the path, whereas the problem asks for the *number of edges* (which is always $Nodes - 1$, so it is simply `leftHeight + rightHeight`).
2. **Missing Recursion on Paths**: Assuming the diameter *must* pass through the root, calculating only `height(left) + height(right)` at the root. This fails for trees like:
   ```
          1
         /
        2
       / \
      3   4
     /     \
    5       6
   ```
   Here, the diameter path `5 -> 3 -> 2 -> 4 -> 6` (length 4) does not pass through the root `1`.

---

## 12. Edge Cases

| Edge Case | Input Tree | Why it matters | How Code Handles It |
|:---|:---|:---|:---|
| **Single Node** | `[1]` | Minimum nodes | Left/Right are null $\to$ `leftHeight=0`, `rightHeight=0`. Diameter is `0`. Correct. |
| **Deep Skewed Tree**| `1 -> 2 -> 3 -> 4` | Deep recursion | Time remains $\mathcal{O}(N)$. Stack uses $\mathcal{O}(N)$ space. |
| **Two Nodes** | `1 -> 2` | One edge | Left height = 1, Right height = 0. Diameter is `1`. Correct. |

---

## 13. Interview Explanation

"To find the diameter of a binary tree, which is the longest path between any two nodes measured in edges, we can observe that at any node, the longest path using this node as the highest point is the sum of the heights of its left and right subtrees.

The brute force approach calculates heights for every node separately, leading to redundant traversals and an $\mathcal{O}(N^2)$ runtime.

To optimize this, we can compute the height and update the diameter in a single bottom-up DFS traversal. At each node, we recursively get the left and right subtree heights, update a global diameter variable with `leftHeight + rightHeight`, and return `1 + max(leftHeight, rightHeight)` to calculate the height for the parent. This runs in $\mathcal{O}(N)$ time and uses $\mathcal{O}(H)$ stack space, making it highly efficient."

---

## 14. Follow-up Questions

1. **Q**: What if we needed to return the actual path (a list of node values) instead of just the length?
   - **A**: We would modify our recursive function to return the longest path as a vector of nodes. At each node, we would check if the combined paths of left and right children are longer than the best path found so far, and keep track of that path.
2. **Q**: How does this change if we want the longest path in a General Tree (N-ary Tree)?
   - **A**: At each node, instead of two children, we would find the heights of all children, select the top two highest children, and add them together to evaluate the diameter at that node.

---

## 15. Variations

1. **Binary Tree Maximum Path Sum**: Find the path with the maximum sum of node values. Instead of heights (edges count), we accumulate maximum node value sums bottom-up.
2. **Longest Univalue Path**: Find the length of the longest path where all nodes have the same value.

---

## 16. Revision Notes

- **Diameter Equation**: `max(diameter, leftHeight + rightHeight)`.
- **Height Propagated Up**: `1 + max(leftHeight, rightHeight)`.
- **Single Pass DFS Pattern**: Modifies a reference tracker variable while returning helper metrics.

---

## 17. Memorization Version

```cpp
// Single-pass DFS: Returns height, updates diameter reference.
int height(TreeNode* node, int& d) {
    if (!node) return 0;
    int lh = height(node->left, d);
    int rh = height(node->right, d);
    d = max(d, lh + rh);
    return 1 + max(lh, rh);
}
int diameterOfBinaryTree(TreeNode* root) {
    int d = 0; height(root, d); return d;
}
```
