# Mirror / Invert a Binary Tree

## 1. Problem Statement

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

---

## 2. Interview Clarification

Before coding, clarify:
1. **Empty Tree**: "Should we return `nullptr` if the root is null?"
   - *Interviewer response:* Yes.
2. **In-place vs. Copy**: "Do you want us to invert the tree in-place by swapping pointers, or should we allocate a new mirror copy of the tree?"
   - *Interviewer response:* In-place modification is preferred.
3. **Iterative and Recursive**: "Would you like to see both recursive and iterative implementations?"
   - *Interviewer response:* Yes, show both and explain how the iterative queue or stack tracks the swap operation.

---

## 3. Thinking Process

### First Instinct (Recursive Bottom-up / Top-down DFS)
- A tree structure is recursive. To invert a tree:
  - Base Case: If the root is null, return null.
  - Recursively invert the left child.
  - Recursively invert the right child.
  - Swap the left and right pointers of the current root.
  - Return the root.
- The order of operation does not strictly matter (it can be pre-order: swap first then recurse; or post-order: recurse first then swap). Both work perfectly.

### Transition to Iterative Approach (BFS / DFS Queue/Stack)
- To do this iteratively, we must visit every node and swap its children.
- We can use a queue (BFS-style) or a stack (DFS-style).
- **Algorithm (BFS)**:
  1. Initialize a queue and push the `root` (if not null).
  2. While the queue is not empty:
     - Pop the front node `curr`.
     - Swap `curr->left` and `curr->right`.
     - Push `curr->left` (if not null) to the queue.
     - Push `curr->right` (if not null) to the queue.
  3. Return the `root`.

---

## 4. Pattern Recognition

- **Category**: Binary Tree Modification.
- **Pattern**: Recursive Tree Traversal / Queue-based Node Swapping.
- **Why**: Since we must swap the left and right children for *every* node in the tree, we need a complete tree traversal (either DFS or BFS). The swap logic itself is a constant time $\mathcal{O}(1)$ operation per node.

---

## 5. Brute Force Solution

*Note: Since the recursive DFS and iterative BFS are both $\mathcal{O}(N)$ time complexity and represent the standards, there is no "brute force" in terms of complexity. We will present the Recursive DFS as the primary clean optimal solution and Iterative BFS as the alternative robust implementation.*

---

## 6. Better Solution

*Note: The recursive solution jumps directly to the optimal version. We will detail the recursive version below as the Optimal Solution, and the iterative version under the Iterative section.*

---

## 7. Optimal Solution (Recursive DFS)

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

---

### Why this is Optimal
It processes each node in constant time $\mathcal{O}(1)$ (excluding the traversal steps). The space complexity is $\mathcal{O}(H)$ call stack frames, making it highly memory-efficient for balanced trees.

---

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

---

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

---

## 8. Code Walkthrough

Let's dissect the recursive solution:
- `if (root == nullptr) return nullptr;`: Base case. When we reach the bottom of the tree, we return `nullptr` back up.
- `TreeNode* leftInverted = invertTree(root->left);`: Computes the mirrored left subtree.
- `TreeNode* rightInverted = invertTree(root->right);`: Computes the mirrored right subtree.
- `root->left = rightInverted;` & `root->right = leftInverted;`: Performs the swap. Note that we use the cached variables `leftInverted` and `rightInverted` to avoid overwriting pointers before the swap is complete.

---

## 9. Dry Run

Let's dry run the recursive solution with this tree:
```
      4
     / \
    2   7
   / \ / \
  1  3 6  9
```

- **Execution Trace**:
  - `invertTree(4)`:
    - calls `invertTree(2)`:
      - calls `invertTree(1)`: left/right are null $\to$ returns `1`.
      - calls `invertTree(3)`: left/right are null $\to$ returns `3`.
      - swaps `2`'s children: `2->left = 3`, `2->right = 1`. Returns `2`.
    - calls `invertTree(7)`:
      - calls `invertTree(6)`: returns `6`.
      - calls `invertTree(9)`: returns `9`.
      - swaps `7`'s children: `7->left = 9`, `7->right = 6`. Returns `7`.
    - swaps `4`'s children: `4->left = 7`, `4->right = 2`.
- Final tree output root is `4`, with children:
```
      4
     / \
    7   2
   / \ / \
  9  6 3  1
```

---

## 10. Complexity Analysis

| Metric | Recursive DFS (Optimal) | Iterative BFS |
|:---|:---|:---|
| **Time Complexity** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Space Complexity (Worst)**| $\mathcal{O}(N)$ (skewed tree stack) | $\mathcal{O}(W) \approx \mathcal{O}(N)$ (widest level queue) |
| **Space Complexity (Best)** | $\mathcal{O}(\log N)$ (balanced tree)| $\mathcal{O}(1)$ (skewed tree) |

---

## 11. Common Mistakes

1. **Incorrect Swap Order**: Swapping the children directly without caching:
   ```cpp
   root->left = invertTree(root->right);
   root->right = invertTree(root->left); // Bug: root->left has already changed!
   ```
   This will result in the right side being inverted twice, and the left side's original values being overwritten. Always cache the results in temporary variables or use `std::swap` before recursing.
2. **Missing Null Checks in Queue**: In the iterative version, pushing null children to the queue. This causes runtime issues when attempting to read pointers from null elements.

---

## 12. Edge Cases

| Edge Case | Input Tree | Why it matters | How Code Handles It |
|:---|:---|:---|:---|
| **Empty Tree** | `nullptr` | Can cause null-dereference | Checked immediately at start: returns `nullptr`. |
| **Single Node** | `[1]` | Minimum structure | Swaps its null children, returns self. |
| **Skewed Tree** | `1 -> 2 -> 3` | Stack height grows to maximum | Handled correctly. DFS uses $\mathcal{O}(N)$ stack; BFS uses $\mathcal{O}(1)$ queue space. |

---

## 13. Interview Explanation

"To invert or mirror a binary tree, we must swap the left and right children for every node in the tree. 

We can solve this recursively using a depth-first search approach. 

For the current node, we recursively call the inversion function on its left and right subtrees. We store the returned inverted subtrees in temporary variables, and then assign the left child pointer to the inverted right subtree, and the right child pointer to the inverted left subtree. This takes $\mathcal{O}(N)$ time because we visit every node exactly once, and uses $\mathcal{O}(H)$ call stack space where $H$ is the height of the tree. 

Alternatively, we can solve this iteratively using a queue to perform a level-order BFS traversal, popping each node, swapping its child pointers in-place, and pushing the non-null children onto the queue. This avoids stack overflow concerns and has a space complexity of $\mathcal{O}(W)$ where $W$ is the maximum width of the tree."

---

## 14. Follow-up Questions

1. **Q**: Can you implement this in-place without using any auxiliary space (like stacks or queues)?
   - **A**: Standard tree structures do not have parent pointers, so traversing without a stack, queue, or call stack is not possible unless we use a modified Morris traversal. However, Morris traversal modifies the tree, which is difficult when we are actively swapping children. Thus, iterative BFS/DFS or recursive DFS is preferred.
2. **Q**: What if we are given two trees and need to check if they are mirrors of each other?
   - **A**: This is the "Symmetric Tree" problem, where we compare `t1->left` with `t2->right` and `t1->right` with `t2->left` recursively.

---

## 15. Variations

1. **Symmetric Tree**: Check if a tree is a mirror image of itself.
2. **Flip Binary Tree**: Reconstruct a binary tree where the left child becomes the new root and the right child becomes the left child.

---

## 16. Revision Notes

- **Mirror Rule**: Swap `left` and `right` child pointers at every node.
- **Temporary Variable Rule**: Store inverted children in variables before reassigning to avoid overwriting pointers before they are evaluated.
- **Complexity**: $\mathcal{O}(N)$ time, space scales with height (DFS) or width (BFS).

---

## 17. Memorization Version

```cpp
// Recursive Invert: Swap left and right subtrees recursively.
TreeNode* invertTree(TreeNode* root) {
    if (!root) return nullptr;
    TreeNode* temp = root->left;
    root->left = invertTree(root->right);
    root->right = invertTree(temp);
    return root;
}
```
