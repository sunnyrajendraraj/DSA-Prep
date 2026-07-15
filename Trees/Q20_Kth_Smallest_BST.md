# Kth Smallest Element in a BST

## 1. Problem Statement

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

---

## 2. Interview Clarification

1.  **Q**: Is `k` guaranteed to be valid (i.e. $1 \le k \le N$)?
    *   *A*: Yes, you can assume $k$ is always valid.
2.  **Q**: What if the BST is modified frequently with insertions and deletions, and we need to find the $k$-th smallest element repeatedly? How would you optimize it?
    *   *A*: In that case, we can augment the BST nodes to store the size of their subtrees. This allows us to find the $k$-th smallest element in $O(H)$ time rather than traversing the tree. We should discuss this augmented design.
3.  **Q**: Can we solve this with $O(1)$ auxiliary space?
    *   *A*: Yes, using Morris Inorder Traversal, which modifies the tree temporarily.

---

## 3. Thinking Process

*   **First Instinct**:
    *   Perform a full recursive inorder traversal, store all node values in a sorted vector, and return the element at index `k - 1`.
    *   *Why this is not optimal*: It requires $O(N)$ space to store all node values and visits every node, even if $k = 1$.

*   **Optimal Insight**:
    *   We don't need to traverse the whole tree. We can stop as soon as we visit the $k$-th node.
    *   Using an **iterative inorder traversal** with a stack:
        *   We push all left children to a stack until we hit a leaf.
        *   We pop a node, decrement `k` by 1.
        *   If `k == 0`, we have found the $k$-th smallest node and return its value.
        *   Otherwise, we move to the right child and repeat.
    *   This only visits $k$ nodes, taking $O(H + k)$ time.

---

## 4. Pattern Recognition

*   **Topic**: BST Properties.
*   **Pattern**: Inorder Traversal (Iterative with Stack).
*   **Why**: BST sorted property maps directly to inorder traversal. An iterative approach using a stack allows us to stop traversal early (lazy evaluation) as soon as $k$ reaches 0, saving time.

---

## 5. Brute Force Solution

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

---

## 6. Better Solution

*   **Recursive Inorder with Early Termination**:
    *   Instead of storing all elements, maintain a global/reference counter `count`.
    *   Recurse left.
    *   At current node: increment `count`. If `count == k`, store the value and return.
    *   Recurse right.
*   **Complexity**:
    *   **Time Complexity**: $O(H + k)$
    *   **Space Complexity**: $O(H)$ recursion stack.
*   **Pros/Cons**:
    *   Better space complexity than brute force ($O(H)$ vs $O(N)$), but requires keeping track of state across recursion.

### Recursive Better C++17 Code
```cpp
class SolutionRecursive {
private:
    int count = 0;
    int result = -1;

    void inorder(TreeNode* root, int k) {
        if (!root || count >= k) return;
        
        inorder(root->left, k);
        
        count++;
        if (count == k) {
            result = root->val;
            return;
        }
        
        inorder(root->right, k);
    }

public:
    int kthSmallest(TreeNode* root, int k) {
        inorder(root, k);
        return result;
    }
};
```

---

## 7. Optimal Solution

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

---

## 8. Code Walkthrough

1.  **Iterative Inorder Template**:
    *   The outer loop `while (curr || !s.empty())` is the standard iterative DFS pattern.
2.  **Leftmost Path Search**:
    *   `while (curr) { s.push(curr); curr = curr->left; }` descends to the leftmost node of the current subtree. This replicates recursive left-first calls.
3.  **Process Node & Decrement**:
    *   We pop `curr`. Popping represents visiting the node (Root).
    *   `k--` reduces the count of remaining nodes to visit.
4.  **Early Exit**:
    *   `if (k == 0) return curr->val` triggers immediately when the target node is popped, preventing traversal of any remaining nodes.
5.  **Right Subtree Branch**:
    *   `curr = curr->right` switches context to the right child.

---

## 9. Dry Run

Consider the tree:
```
      5
     / \
    3   6
   / \
  2   4
```
And `k = 3`.

*   **Initialization**: `s = []`, `curr = 5`.

| Step | Stack `s` | `curr` | Action | `k` | Return Val |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | `[5, 3, 2]` | `nullptr` | Leftmost path of 5 pushed. | 3 | - |
| 2 | `[5, 3]` | `2` | Pop `2`. `curr = 2->right = nullptr`. | 2 | - |
| 3 | `[5]` | `3` | Pop `3`. | 1 | - |
| 4 | `[5]` | `4` | `curr = 3->right = 4`. Push `4` to stack. | 1 | - |
| 5 | `[5, 4]` | `nullptr` | `curr = 4->left = nullptr`. | 1 | - |
| 6 | `[5]` | `4` | Pop `4`. | 0 | **4** |

At Step 6, `k` becomes `0`, and we immediately return `4`.

---

## 10. Complexity Analysis

*   **Time Complexity**: $\mathcal{O}(H + k)$
    *   We spend $\mathcal{O}(H)$ time to reach the leftmost leaf of the tree.
    *   From there, we process $k$ nodes.
    *   In the worst case (e.g. skewed tree and $k = N$), this is $\mathcal{O}(N)$.
    *   On average for a balanced tree, height $H = \log N$, so time complexity is $\mathcal{O}(\log N + k)$.
*   **Space Complexity**: $\mathcal{O}(H)$
    *   The stack stores at most $H$ elements at any time.
    *   For a balanced tree, space is $\mathcal{O}(\log N)$. For a skewed tree, it is $\mathcal{O}(N)$.

---

## 11. Common Mistakes

1.  **1-Indexed vs 0-Indexed Counter**: Decrementing `k` incorrectly or using standard `k` index checks, leading to off-by-one errors.
2.  **Not stopping early**: Performing a full traversal when $k$ is small.
3.  **Losing Track of `curr`**: Forgetting to update `curr = curr->right` after popping, causing the loop to re-process nodes or terminate early.

---

## 12. Edge Cases

| Edge Case | Input Tree | `k` | Expected Output | Importance |
| :--- | :--- | :--- | :--- | :--- |
| **Smallest Element** | Any BST | `1` | Minimum node in BST | Verifies correct early exit at first pop. |
| **Largest Element** | Any BST of size $N$ | $N$ | Maximum node in BST | Verifies full traversal up to last node. |
| **Skewed Left Tree** | `3 -> 2 -> 1` | `2` | `2` | Stack size matches $H = N$. |

---

## 13. Interview Explanation

"To find the $k$-th smallest element in a Binary Search Tree, we take advantage of the BST property: an inorder traversal of a BST visits the nodes in sorted, ascending order. 

Instead of doing a full traversal and saving all values, we can perform an iterative inorder traversal using a stack. We push all left descendants of our current node onto the stack. Then, we pop nodes one by one. Each pop represents visiting a node in sorted order. We decrement `k` with each pop. As soon as `k` reaches `0`, we return the value of the popped node. This approach avoids full traversal and executes in $O(H + k)$ time and uses $O(H)$ auxiliary space for the stack."

---

## 14. Follow-up Questions

### Q: What if the BST is modified frequently (insertions/deletions) and we need to query the $k$-th smallest element often? How would you optimize this?
*   **Answer**: We can augment the `TreeNode` structure to store the size of the subtree rooted at that node:
    ```cpp
    struct TreeNode {
        int val;
        int count; // Number of nodes in this subtree
        TreeNode* left;
        TreeNode* right;
    };
    ```
    This size count is updated during insertions and deletions (taking $O(H)$ time).
    When querying the $k$-th smallest element:
    *   Let `leftSize = root->left ? root->left->count : 0`.
    *   If `k <= leftSize`, the target is in the left subtree. Recurse left.
    *   If `k == leftSize + 1`, the current root is the target. Return `root->val`.
    *   If `k > leftSize + 1`, the target is in the right subtree. Recurse right with `k = k - leftSize - 1`.
    This reduces the query time to $\mathcal{O}(H)$, which is independent of $k$.

---

## 15. Variations

1.  **Kth Largest Element in a BST**:
    *   *Solution*: Traverse in reverse inorder (Right $\rightarrow$ Root $\rightarrow$ Left). The $k$-th node popped will be the $k$-th largest.

---

## 16. Revision Notes

*   **BST Sorted Order**: Inorder traversal is sorted.
*   **Optimized Inorder**: Iterative with stack allows lazy evaluation.
*   **Augmentation Trick**: Store subtree counts for $O(H)$ frequent queries.

---

## 17. Memorization Version

```cpp
int kthSmallest(TreeNode* root, int k) {
    stack<TreeNode*> s;
    TreeNode* curr = root;
    while (curr || !s.empty()) {
        while (curr) {
            s.push(curr);
            curr = curr->left;
        }
        curr = s.top(); s.pop();
        if (--k == 0) return curr->val;
        curr = curr->right;
    }
    return -1;
}
```
*   **Mnemonic**: Iterative inorder pattern. Push all lefts, pop, decrement `k`, check `0`, move right.
*   **Complexity**: $O(H + k)$ time, $O(H)$ space.
