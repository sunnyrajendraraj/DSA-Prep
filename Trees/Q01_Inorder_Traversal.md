# Inorder Traversal of Binary Tree

## 1. Problem Statement

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

---

## 2. Interview Clarification

Before writing any code, clarify these points with your interviewer:
1. **Empty Tree**: "Can the root be null? If yes, should we return an empty list/vector?" 
   - *Interviewer response:* Yes, return an empty vector.
2. **Output Modification**: "Do you want us to return the values in a collection (like `std::vector`), or print them?"
   - *Interviewer response:* Return them as a vector.
3. **Tree Size/Height**: "Is the tree balanced or skewed? This will affect the worst-case space complexity of the recursive stack."
   - *Interviewer response:* It could be skewed, so plan for $O(N)$ space in the worst case.
4. **Extra Constraints**: "Are we allowed to use extra space, or should we optimize for $O(1)$ auxiliary space?"
   - *Interviewer response:* First implement the standard recursive and iterative approaches, then we can discuss $O(1)$ space.

---

## 3. Thinking Process

### First Instinct (Recursive Approach)
- A tree is recursively defined. To traverse a tree in `Left-Root-Right` order:
  - If the current node is null, do nothing (base case).
  - Recursively visit the left child.
  - Record the value of the current root.
  - Recursively visit the right child.
- This is simple but uses implicit stack frames for recursion.

### Transition to Iterative Approach
- To do this iteratively, we must simulate the call stack manually.
- We need to go as far left as possible, saving visited nodes along the way so we can backtrack to them.
- A stack (`std::stack<TreeNode*>`) is the ideal data structure to remember nodes we have passed but not yet processed.
- **Algorithm design**:
  1. Initialize an empty stack and set a pointer `curr = root`.
  2. Loop while `curr != nullptr` or the stack is not empty:
     - Keep pushing `curr` to the stack and moving to `curr->left` until `curr` becomes null.
     - Pop the top node from the stack, visit it (add to output).
     - Set `curr = popped_node->right` to traverse the right subtree.

### Optimal/Bonus Insight: Morris Traversal
- Can we traverse without a stack or recursion?
- The reason we need a stack is to backtrack to the root/parent after visiting the left subtree.
- **Morris Traversal** establishes temporary links (threads) from the predecessor (rightmost node of the left subtree) back to the current node.
- When we finish visiting the left subtree, we use the thread to return to the current node, then we destroy the thread to restore the tree's original structure.

---

## 4. Pattern Recognition

- **Category**: Binary Tree Traversal (DFS).
- **Pattern**: Depth-First Search (DFS) / Stack-based simulation / Tree threading.
- **Why**: Tree structures do not contain parent pointers. Thus, to back-track upwards, we must either use a call stack (recursive), an explicit stack (iterative), or modify the leaf nodes to temporarily point to ancestors (Morris Traversal).

---

## 5. Brute Force Solution (Recursive Approach)

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

---

## 6. Better Solution (Iterative Approach)

To avoid call stack overflow, we simulate recursion using an explicit `std::stack`.

### Algorithm
1. Initialize an empty stack of `TreeNode*` and a pointer `curr` pointing to `root`.
2. Loop while `curr != nullptr` or the stack is not empty:
   - While `curr != nullptr`, push `curr` to stack and update `curr = curr->left` (go deep left).
   - Pop the top node from stack.
   - Add its value to the result vector.
   - Update `curr = popped_node->right` (move to the right subtree).

### Dry Run
Using the same tree:
```
    1
     \
      2
     /
    3
```
- `curr = 1`, Stack `S = []`, `result = []`
- `curr != nullptr` -> Push `1`, `S = [1]`, `curr = 1->left = nullptr`.
- `curr == nullptr`, `S` is not empty:
  - Pop `1`, `S = []`, `result = [1]`.
  - `curr = 1->right = 2`.
- `curr = 2 != nullptr` -> Push `2`, `S = [2]`, `curr = 2->left = 3`.
- `curr = 3 != nullptr` -> Push `3`, `S = [2, 3]`, `curr = 3->left = nullptr`.
- `curr == nullptr`, `S` is not empty:
  - Pop `3`, `S = [2]`, `result = [1, 3]`.
  - `curr = 3->right = nullptr`.
- `curr == nullptr`, `S` is not empty:
  - Pop `2`, `S = []`, `result = [1, 3, 2]`.
  - `curr = 2->right = nullptr`.
- Loop terminates since `curr == nullptr` and `S` is empty. Output: `[1, 3, 2]`.

### Complexity Analysis
- **Time Complexity**: $\mathcal{O}(N)$ since each node is pushed onto and popped from the stack exactly once.
- **Space Complexity**: $\mathcal{O}(H)$ for the stack, where $H$ is the height of the tree. Worst case is $\mathcal{O}(N)$ for skewed trees.

### C++17 Implementation
```cpp
#include <vector>
#include <stack>

class SolutionIterative {
public:
    std::vector<int> inorderTraversal(TreeNode* root) {
        std::vector<int> result;
        std::stack<TreeNode*> s;
        TreeNode* curr = root;

        while (curr != nullptr || !s.empty()) {
            // Keep going left as far as possible
            while (curr != nullptr) {
                s.push(curr);
                curr = curr->left;
            }
            
            // Current is null, pop the top node from stack
            curr = s.top();
            s.pop();
            
            // Visit the node
            result.push_back(curr->val);
            
            // Move to the right subtree
            curr = curr->right;
        }
        
        return result;
    }
};
```

---

## 7. Optimal Solution (Morris Traversal)

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

---

### Why Morris Traversal is Optimal
Unlike other solutions, Morris traversal achieves the theoretical minimum space bound of $\mathcal{O}(1)$ auxiliary space (not counting the output vector). It does this by leveraging the unused `nullptr` right pointers of leaf nodes to create temporary return links.

---

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

---

## 8. Code Walkthrough

Let's trace the Morris Traversal logic inside the `while(curr != nullptr)` loop:
- `TreeNode* curr = root;` sets up the runner pointer.
- `curr->left == nullptr`: Since there is no left child, we have completed the left side traversal for this node's context. We capture the value and move right.
- `pred = curr->left`: We step left once, then we keep moving right (`pred = pred->right`) to find the rightmost node of the left subtree.
- `pred->right != curr`: Crucial check. If we didn't have this, we would loop infinitely once the thread is established.
- `pred->right = curr`: This is where the temporary backlink is made.
- `pred->right = nullptr`: Cleans up the backlink once we finish visiting the left subtree and return to `curr`.

---

## 9. Dry Run

Let's dry run Morris Traversal with this tree:
```
      4
     / \
    2   5
   / \
  1   3
```

| Step | `curr` node | `curr->left`? | Predecessor (`pred`) | Action / Thread Change | Result Vector | Next `curr` |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | 4 | Yes | 3 (Rightmost of 2) | `pred->right` was null. Thread `3->4`. | `[]` | 2 |
| 2 | 2 | Yes | 1 (Rightmost of 1) | `pred->right` was null. Thread `1->2`. | `[]` | 1 |
| 3 | 1 | No | — | Visit `1`. | `[1]` | 2 (via thread `1->2`) |
| 4 | 2 | Yes | 1 | `pred->right` is `2`. Unthread `1->2`. Visit `2`. | `[1, 2]` | 3 |
| 5 | 3 | No | — | Visit `3`. | `[1, 2, 3]` | 4 (via thread `3->4`) |
| 6 | 4 | Yes | 3 | `pred->right` is `4`. Unthread `3->4`. Visit `4`. | `[1, 2, 3, 4]` | 5 |
| 7 | 5 | No | — | Visit `5`. | `[1, 2, 3, 4, 5]` | `nullptr` (end) |

---

## 10. Complexity Analysis

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

## 11. Common Mistakes

1. **Forgetting to break Morris threads**: If you do not set `pred->right = nullptr` on the second visit, the tree structure remains modified, leading to cycles and infinite loops if someone traverses the tree later.
2. **Infinite loop in finding predecessor**: In Morris traversal, forgeting the condition `pred->right != curr` inside the while loop will cause an infinite loop because it will follow the thread back to `curr`.
3. **Stack Overflow on Skewed Trees**: Assuming the recursive solution is always "good enough." Highly deep trees will crash the runtime stack.

---

## 12. Edge Cases

| Edge Case | Input Tree | Why it matters | How Code Handles It |
|:---|:---|:---|:---|
| **Null Tree** | `nullptr` | Can throw NullPointerException | Guard condition `curr != nullptr` or recursion base case immediately exits. |
| **Single Node** | `[1]` | Minimal tree size | Standard loops visit once and exit. |
| **Left-Skewed** | `3 -> 2 -> 1` | Stack size grows to max | Iterative stack handles $\mathcal{O}(N)$ frames. Morris handles it efficiently. |
| **Right-Skewed**| `1 -> 2 -> 3` | Stack size is $\mathcal{O}(1)$ | Easily handled by moving `curr = curr->right`. |

---

## 13. Interview Explanation

"To traverse a binary tree inorder, we need to visit the left subtree, then the root, and finally the right subtree. 

The most straightforward way to do this is **recursively** which requires $\mathcal{O}(H)$ call stack space where $H$ is the height of the tree. To avoid call-stack limitations, we can write an **iterative** version using an explicit `std::stack`, traversing left nodes to the bottom, popping, recording, and transitioning to their right subtrees. 

If we want to optimize for space, we can use **Morris Traversal**. By finding the inorder predecessor of the current node, we can temporarily link its right pointer to the current node. This lets us return to the current node after completing the left subtree without using extra memory. Once we return, we restore the tree by removing this thread. This gives us an optimal $\mathcal{O}(N)$ time and $\mathcal{O}(1)$ auxiliary space complexity."

---

## 14. Follow-up Questions

1. **Q**: Can you implement Morris Traversal without modifying the tree?
   - **A**: No, the definition of Morris Traversal inherently relies on modifying the child pointer fields temporarily. If the tree is read-only (shared among threads), you must use the stack-based iterative approach.
2. **Q**: How would you traverse a BST in reverse inorder (Right-Root-Left)?
   - **A**: Simply swap the left and right traversal steps in the recursive, iterative stack, or Morris algorithms.

---

## 15. Variations

1. **Binary Search Tree Iterator**: Build a class that returns `next()` elements inorder. The iterative stack solution forms the exact foundation for this problem, where you yield nodes lazily.
2. **Kth Smallest Element in a BST**: Since inorder traversal of a BST is sorted, you perform inorder traversal and stop at the $K$-th visited element.

---

## 16. Revision Notes

- **Inorder Visit Order**: Left $\to$ Root $\to$ Right.
- **BST Property**: Inorder of a BST = Sorted Array.
- **Iterative Tip**: Keep push left, pop, go right.
- **Morris Traversal Formula**:
  - `pred = curr->left`
  - Find rightmost child of `pred` that doesn't point to `curr`.
  - Set `pred->right = curr` (first visit), or `pred->right = nullptr` (second visit).

---

## 17. Memorization Version

```cpp
// Iterative: Push all left. Pop, record, go right.
vector<int> inorder(TreeNode* root) {
    vector<int> res; stack<TreeNode*> s; TreeNode* curr = root;
    while (curr || !s.empty()) {
        while (curr) { s.push(curr); curr = curr->left; }
        curr = s.top(); s.pop();
        res.push_back(curr->val);
        curr = curr->right;
    }
    return res;
}
```
