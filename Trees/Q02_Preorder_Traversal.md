# Preorder Traversal of Binary Tree

## 1. Problem Statement

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

---

## 2. Interview Clarification

Before coding, verify these questions with the interviewer:
1. **Null Input**: "Should we return an empty vector if the root is null?"
   - *Interviewer response:* Yes.
2. **Iterative constraints**: "Do you want to see both the recursive and iterative approaches?"
   - *Interviewer response:* Yes, and please explain how the stack works in the iterative approach, particularly the ordering of pushing children.
3. **Modify Node Structures**: "Can we modify the node structures, or should we treat the tree as read-only?"
   - *Interviewer response:* Keep it read-only for now.

---

## 3. Thinking Process

### First Instinct (Recursive Approach)
- Preorder traversal is defined as `Root-Left-Right`.
- The recursive steps are:
  - If node is null, return.
  - Visit/record current node value.
  - Recurse on left child.
  - Recurse on right child.

### Transition to Iterative Approach
- To perform this traversal iteratively, we can use an explicit stack (`std::stack<TreeNode*>`).
- Since we process the current node immediately:
  - First, push the `root` node onto the stack.
  - While the stack is not empty:
    - Pop the top node, process/visit it.
    - **Crucial step**: We must process the left child before the right child. Since a stack is Last-In-First-Out (LIFO), to process the left child first, we must push the **right child** onto the stack first, and then push the **left child**.

---

## 4. Pattern Recognition

- **Category**: Binary Tree Traversal (DFS).
- **Pattern**: Depth-First Search (DFS) / Iterative Stack.
- **Why**: Preorder is inherently DFS because we dive deep down a path to leaf nodes before backtracking. The stack simulates the recursion call stack, matching the LIFO property of backtracking.

---

## 5. Brute Force Solution (Recursive Approach)

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

---

## 6. Better Solution

*Note: Since the recursive method is the baseline, the iterative stack-based solution is the direct improvement to prevent stack overflow. We will detail this as the Optimal Solution below.*

---

## 7. Optimal Solution (Iterative Approach)

The iterative approach uses an explicit stack to achieve $\mathcal{O}(N)$ time and $\mathcal{O}(H)$ space, eliminating call stack limits.

### Algorithm
1. Initialize an empty stack and push `root` if it is not null.
2. While stack is not empty:
   - Pop the top node `curr`.
   - Visit `curr` (add `curr->val` to result).
   - If `curr->right` is not null, push `curr->right` to the stack.
   - If `curr->left` is not null, push `curr->left` to the stack.
3. Return the result vector.

---

### Why this is Optimal
This solution uses an explicit stack, preventing stack overflow on deeply nested trees while running in linear time. We push the right child first so that the left child is popped and processed first, matching the Root-Left-Right order.

---

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

---

## 8. Code Walkthrough

- `if (root == nullptr) return result;`: Prevents attempting to push null to the stack.
- `s.push(root)`: Places the start point of traversal on the stack.
- `TreeNode* curr = s.top(); s.pop();`: Retrieves the current parent to process.
- `result.push_back(curr->val)`: Visits the parent node before children.
- `if (curr->right) s.push(curr->right);`: Places the right child on the stack. It will stay there until all left-descendant paths are processed.
- `if (curr->left) s.push(curr->left);`: Places the left child on the top of the stack. LIFO ensures it is popped next.

---

## 9. Dry Run

Let's trace this tree:
```
      1
     / \
    2   3
   / \
  4   5
```

| Step | Stack State (Top on Right) | Popped Node | Visit Value | Pushed Children (Right, then Left) | Result Vector |
|:---:|:---|:---:|:---:|:---|:---|
| Init | `[1]` | — | — | — | `[]` |
| 1 | `[]` | 1 | 1 | Push 3, then 2 $\to$ Stack: `[3, 2]` | `[1]` |
| 2 | `[3]` | 2 | 2 | Push 5, then 4 $\to$ Stack: `[3, 5, 4]` | `[1, 2]` |
| 3 | `[3, 5]` | 4 | 4 | None $\to$ Stack: `[3, 5]` | `[1, 2, 4]` |
| 4 | `[3]` | 5 | 5 | None $\to$ Stack: `[3]` | `[1, 2, 4, 5]` |
| 5 | `[]` | 3 | 3 | None $\to$ Stack: `[]` | `[1, 2, 4, 5, 3]` |

---

## 10. Complexity Analysis

| Metric | Recursive | Iterative (Optimal) |
|:---|:---|:---|
| **Time Complexity (Best)** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Time Complexity (Avg)** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Time Complexity (Worst)**| $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Space Complexity (Worst)**| $\mathcal{O}(N)$ (skewed tree) | $\mathcal{O}(N)$ (skewed tree) |
| **Space Complexity (Best)** | $\mathcal{O}(\log N)$ (balanced tree)| $\mathcal{O}(\log N)$ (balanced tree)|

---

## 11. Common Mistakes

1. **Incorrect Push Order**: Pushing the left child before the right child in the iterative stack. This leads to a Root-Right-Left traversal, which is NOT preorder traversal.
2. **Pushing `nullptr`**: Forgetting to check if `curr->right` or `curr->left` are null before pushing them, leading to null pointer exceptions or redundant iterations.
3. **Empty Tree Crashes**: Not verifying if `root == nullptr` before pushing onto the stack at initialization.

---

## 12. Edge Cases

| Edge Case | Input Tree | Why it matters | How Code Handles It |
|:---|:---|:---|:---|
| **Empty Tree** | `nullptr` | Can cause null pointer dereference | `if (root == nullptr) return result;` immediately exits. |
| **Single Node** | `[1]` | Minimal tree size | Popped immediately, no children to push, exits. |
| **Skewed Tree (Left)**| `3 -> 2 -> 1` | Stack size matches height $N$ | Handled correctly. Memory grows linearly. |
| **Skewed Tree (Right)**| `1 -> 2 -> 3` | Stack size is $\mathcal{O}(1)$ | Only right nodes pushed/popped immediately. |

---

## 13. Interview Explanation

"Preorder traversal is a Depth-First Search strategy where we visit the current node first, then traverse its left subtree, and finally its right subtree. 

We can implement this easily using **recursion** which uses the implicit call stack, but it risks stack overflow for very deep trees. 

To make it robust, we can implement it **iteratively** using an explicit `std::stack`. We push the root onto the stack. Then, while the stack is not empty, we pop the top node and record its value. To ensure that we process the left child before the right child, we push the right child first (if it exists) and then the left child. This takes advantage of the stack's LIFO property, ensuring the left subtree is traversed completely before we backtrack to the right subtree. Both methods run in $\mathcal{O}(N)$ time, and the iterative version limits stack depth to the height of the tree."

---

## 14. Follow-up Questions

1. **Q**: How does preorder traversal relate to serialization of a tree?
   - **A**: Preorder traversal is highly useful for serializing a binary tree because when reading the values back, we process the parent nodes first, making it straightforward to reconstruct the tree structure.
2. **Q**: Can we implement preorder traversal with $\mathcal{O}(1)$ auxiliary space?
   - **A**: Yes, using Morris Preorder Traversal (a variation of Morris Inorder). Instead of printing when clearing threads, we print the node value when establishing the thread.

---

## 15. Variations

1. **Construct Binary Tree from Preorder and Inorder Traversal**: Given these two traversals, rebuild the original tree.
2. **Verify Preorder Serialization of a Binary Tree**: Given a string of comma-separated values, check if it represents a valid preorder traversal.

---

## 16. Revision Notes

- **Preorder Visit Order**: Root $\to$ Left $\to$ Right.
- **Iterative Hack**: Push right child first, then left child.
- **Deepest Path First**: DFS nature processes paths down to leaves before backtracking.
- **Space complexity**: Matches tree height $H$.

---

## 17. Memorization Version

```cpp
// Iterative Preorder: Stack-based, push right child first, then left.
vector<int> preorder(TreeNode* root) {
    vector<int> res; if (!root) return res;
    stack<TreeNode*> s; s.push(root);
    while (!s.empty()) {
        TreeNode* curr = s.top(); s.pop();
        res.push_back(curr->val);
        if (curr->right) s.push(curr->right);
        if (curr->left) s.push(curr->left);
    }
    return res;
}
```
