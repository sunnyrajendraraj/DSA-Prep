# Postorder Traversal of Binary Tree

## 1. Problem Statement

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

---

## 2. Interview Clarification

Before coding, clarify:
1. **Empty Tree**: "Should we return an empty vector if the root is null?"
   - *Interviewer response:* Yes.
2. **Iterative Approaches**: "There are multiple iterative strategies, such as using two stacks or a single stack. Which would you prefer to see?"
   - *Interviewer response:* Show me both recursive and iterative. For iterative, explain the two-stack/reversed-preorder method first, as it is highly intuitive, and then discuss if we can optimize it.
3. **Tree Deletion Link**: "Why is postorder used for deleting/destructing a tree?"
   - *Interviewer response:* I expect you to mention this in your explanation; you must process children first so you don't lose access to them.

---

## 3. Thinking Process

### First Instinct (Recursive Approach)
- Postorder is naturally expressed as:
  - If node is null, return.
  - Recurse left.
  - Recurse right.
  - Visit current node.

### Transition to Iterative Approach 1 (Reversed Preorder / Two Stacks)
- Standard preorder traversal: `Root -> Left -> Right`.
- If we change the preorder traversal child push order, we can traverse: `Root -> Right -> Left`.
- Notice the reverse of `Root -> Right -> Left` is `Left -> Right -> Root`, which is exactly **postorder**!
- Therefore, we can do a modified preorder traversal (`Root -> Right -> Left`) and either:
  1. Store the results in a second stack (or vector) and reverse the collection at the end.
  2. Use a second stack to hold values as they are popped, then pop everything out of the second stack to print.

### Transition to Iterative Approach 2 (Single Stack)
- A single stack iterative traversal is harder because we need to know whether we are returning from the left subtree or the right subtree.
- We traverse down the left subtree, pushing nodes. When we reach the bottom, we inspect the top node's right child:
  - If there is a right child and we haven't visited it yet, we must traverse it.
  - If there is no right child, or we have already traversed it, we pop and visit the node. We must keep track of the `lastVisitedNode` to distinguish these two states.

---

## 4. Pattern Recognition

- **Category**: Binary Tree DFS Traversal.
- **Pattern**: Depth-First Search / Stack reversing or state tracking.
- **Why**: Postorder requires processing children before the parent. Therefore, a stack must hold the parent until the children are completed. The modified preorder reverse pattern maps perfectly to stack operations.

---

## 5. Brute Force Solution (Recursive Approach)

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

---

## 6. Better Solution (Iterative using Two Stacks / Reversed Preorder)

This method simulates a modified preorder traversal `Root -> Right -> Left` and stores results in a stack, which is then popped (reversing the order) to get `Left -> Right -> Root`.

### Algorithm
1. If `root` is null, return empty result.
2. Initialize two stacks: `s1` (traversal stack) and `s2` (result collection stack).
3. Push `root` to `s1`.
4. While `s1` is not empty:
   - Pop `curr` from `s1`.
   - Push `curr` to `s2`.
   - Push `curr->left` to `s1` (if not null).
   - Push `curr->right` to `s1` (if not null).
5. Pop all elements from `s2` and add their values to the result vector.

### Complexity Analysis
- **Time Complexity**: $\mathcal{O}(N)$ as each node is pushed/popped twice.
- **Space Complexity**: $\mathcal{O}(N)$ due to stack `s2` storing all nodes of the tree.

### C++17 Implementation
```cpp
#include <vector>
#include <stack>
#include <algorithm>

class SolutionTwoStacks {
public:
    std::vector<int> postorderTraversal(TreeNode* root) {
        std::vector<int> result;
        if (root == nullptr) return result;
        
        std::stack<TreeNode*> s1;
        std::stack<TreeNode*> s2;
        
        s1.push(root);
        while (!s1.empty()) {
            TreeNode* curr = s1.top();
            s1.pop();
            s2.push(curr);
            
            // Push left first so right is popped first from s1
            if (curr->left != nullptr) {
                s1.push(curr->left);
            }
            if (curr->right != nullptr) {
                s1.push(curr->right);
            }
        }
        
        while (!s2.empty()) {
            result.push_back(s2.top()->val);
            s2.pop();
        }
        
        return result;
    }
};
```

---

## 7. Optimal Solution (Iterative using One Stack)

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

---

### Why this is Optimal
This approach uses only a single stack, meaning its space complexity is strictly limited to the height of the tree $\mathcal{O}(H)$, rather than storing all $N$ elements in a second stack.

---

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

---

## 8. Code Walkthrough

Let's dissect the optimal single-stack logic:
- `curr != nullptr`: We are traversing down. We push `curr` to the stack to remember it for later, and dive left.
- `curr == nullptr`: We have hit a dead end on the left.
- `peekNode->right != nullptr && lastVisited != peekNode->right`: We check if there is a right child. If there is, and we did not just come back from processing it (i.e. `lastVisited != peekNode->right`), then we must shift our traversal pointer `curr` to that right child.
- `else` branch: Either the right child doesn't exist, or we just finished processing it (`lastVisited == peekNode->right`). This means both left and right subtrees of `peekNode` are fully evaluated. It is now safe to visit `peekNode`, update `lastVisited`, and pop it from the stack.

---

## 9. Dry Run

Let's dry run the single-stack optimal approach with this tree:
```
    1
   / \
  2   3
```

| Step | `curr` | Stack `s` | `lastVisited` | Action | Result Vector |
|:---:|:---:|:---|:---:|:---|:---|
| Init | `1` | `[]` | `nullptr` | Start | `[]` |
| 1 | `2` | `[1]` | `nullptr` | `curr != null`, push 1, go left | `[]` |
| 2 | `nullptr` | `[1, 2]` | `nullptr` | `curr != null`, push 2, go left | `[]` |
| 3 | `nullptr` | `[1, 2]` | `nullptr` | `peekNode=2`. `2->right` is null. Visit 2. Pop 2. | `[2]` |
| 4 | `nullptr` | `[1]` | `2` | `peekNode=1`. `1->right` (3) is not null and `lastVisited != 3`. Set `curr = 3`. | `[2]` |
| 5 | `nullptr` | `[1, 3]` | `2` | `curr != null`, push 3, go left | `[2]` |
| 6 | `nullptr` | `[1, 3]` | `2` | `peekNode=3`. `3->right` is null. Visit 3. Pop 3. | `[2, 3]` |
| 7 | `nullptr` | `[1]` | `3` | `peekNode=1`. `1->right` (3) is same as `lastVisited`. Visit 1. Pop 1. | `[2, 3, 1]` |

---

## 10. Complexity Analysis

| Metric | Recursive | Iterative (Two Stacks) | Iterative (Optimal One Stack) |
|:---|:---|:---|:---|
| **Time Complexity** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Space Complexity**| $\mathcal{O}(H)$ call stack | $\mathcal{O}(N)$ storage | $\mathcal{O}(H)$ explicit stack |
| **Space Complexity (Worst)**| $\mathcal{O}(N)$ | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ |
| **Space Complexity (Best)** | $\mathcal{O}(\log N)$ | $\mathcal{O}(N)$ | $\mathcal{O}(\log N)$ |

---

## 11. Common Mistakes

1. **Incorrect Backtracking Check**: In the single-stack approach, failing to check `lastVisited != peekNode->right` will cause an infinite loop because the algorithm will continuously traverse the right subtree.
2. **Wrong Pop Order**: In the two-stack approach, pushing the children in the wrong order (`right` then `left` instead of `left` then `right`) will yield the wrong final result.
3. **Resource Leak on Deletion**: In tree deletion, trying to delete root before children. Since child references are stored in root, deleting root first makes children inaccessible, causing memory leaks.

---

## 12. Edge Cases

| Edge Case | Input Tree | Why it matters | How Code Handles It |
|:---|:---|:---|:---|
| **Empty Tree** | `nullptr` | Can throw NullPointerException | Loop conditions immediately fail, returns empty vector. |
| **Single Node** | `[1]` | Minimal tree structure | Pushed and popped in one step. |
| **Skewed Tree (Left)**| `3 -> 2 -> 1` | Stack reaches maximum height $N$ | Handled correctly. Space complexity is $\mathcal{O}(N)$. |
| **Skewed Tree (Right)**| `1 -> 2 -> 3` | Backtracking from right subtree | Checked using `lastVisited` mechanism successfully. |

---

## 13. Interview Explanation

"Postorder traversal is a depth-first search strategy where we visit the left child, then the right child, and finally the parent node. 

This visit order is particularly useful for **tree deletion**. When deleting a binary tree, we must free the child nodes before freeing the parent node. If we deleted the parent first, we would lose the pointers to the left and right children, causing a memory leak.

For implementation, the **recursive** strategy is straightforward but uses implicit call stack space. For the **iterative** strategy, we can use a two-stack approach where we reverse a modified preorder (`Root-Right-Left`) traversal. To optimize space, we use a **single-stack** approach where we keep track of the `lastVisitedNode` to determine if we are returning from traversing the right subtree. This allows us to process the parent node at the correct time, maintaining a space complexity of $\mathcal{O}(H)$."

---

## 14. Follow-up Questions

1. **Q**: Can we perform postorder traversal in $\mathcal{O}(1)$ space?
   - **A**: Yes, by using Morris Traversal. It is more complex than inorder/preorder Morris: we must write a helper to print nodes along the right boundary of the left subtree in reverse order.
2. **Q**: What are other real-world uses of postorder traversal?
   - **A**: Parsing expression trees (evaluating bottom-up mathematical operations), and directory size calculation (evaluating subfolders before calculating the parent directory size).

---

## 15. Variations

1. **Evaluate Expression Tree**: Parse a tree representing operations like `+`, `-`, `*` on operands. Postorder evaluates operands first, then applies the operator.
2. **Find Duplicate Subtrees**: Use postorder to build serialization strings of subtrees bottom-up to detect duplicates.

---

## 16. Revision Notes

- **Postorder Order**: Left $\to$ Right $\to$ Root.
- **Key Use Case**: Tree deletion / Bottom-up calculations (height, size).
- **Two-Stack trick**: Reverse of (Root $\to$ Right $\to$ Left).
- **Single-Stack trick**: Use `lastVisited` node pointer to identify backtracking from the right subtree.

---

## 17. Memorization Version

```cpp
// Iterative Postorder (Reversed Preorder variation)
vector<int> postorder(TreeNode* root) {
    vector<int> res; if (!root) return res;
    stack<TreeNode*> s; s.push(root);
    while (!s.empty()) {
        TreeNode* curr = s.top(); s.pop();
        res.push_back(curr->val);
        if (curr->left) s.push(curr->left);
        if (curr->right) s.push(curr->right);
    }
    reverse(res.begin(), res.end());
    return res;
}
```
