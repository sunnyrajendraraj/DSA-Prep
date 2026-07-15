# Flatten Binary Tree to Linked List

## 1. Problem Statement

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

---

## 2. Interview Clarification

1. **Should we create new nodes or modify existing ones in-place?**
   - *Answer:* You must modify the tree in-place. Do not allocate new nodes.
2. **Should the left pointer of every node in the flattened list be null?**
   - *Answer:* Yes, all left pointers must be set to `nullptr`.
3. **What is the desired space complexity?**
   - *Answer:* An optimal solution runs in $O(1)$ auxiliary space.

---

## 3. Thinking Process

Let's explore the preorder sequence of a node: `[Root, Left Subtree, Right Subtree]`.
When flattening, the root's right pointer should point to the head of the flattened left subtree, and the tail of the flattened left subtree should point to the head of the flattened right subtree.

### Approach 1: Brute Force
- Perform a preorder traversal, storing all node pointers in a list.
- Iterate through the list, setting `list[i]->left = nullptr` and `list[i]->right = list[i+1]`.
- Space complexity: $O(N)$ to store the node pointers.

### Approach 2: Recursive (Reverse Postorder)
- If we traverse the tree in reverse preorder (Right $\to$ Left $\to$ Root), we can keep track of the node we visited most recently using a pointer `prev`.
- When we are at the current node:
  - We first flatten the right subtree, then the left subtree.
  - We set `root->right = prev`.
  - We set `root->left = nullptr`.
  - We update `prev = root`.
- This is incredibly elegant but uses $O(H)$ space due to recursion.

### Approach 3: Optimal (Morris-like / Iterative Threading)
- To achieve $O(1)$ auxiliary space, we can use the concept of a Morris traversal.
- For any node `curr` with a left child:
  1. Find the rightmost node in the left subtree of `curr`. Let's call it `pred` (predecessor).
  2. Connect the right child of `pred` to the right child of `curr` (`pred->right = curr->right`).
  3. Move the entire left subtree to the right child of `curr` (`curr->right = curr->left`).
  4. Set the left child of `curr` to `nullptr` (`curr->left = nullptr`).
  5. Move `curr` to its new right child (`curr = curr->right`).
- If `curr` has no left child, we simply move to the right child (`curr = curr->right`).
- This modifies pointers on the fly and uses zero recursion/stack memory.

---

## 4. Pattern Recognition

- **Tree Threading / Morris Traversal:** Re-linking unused pointers (like rightmost leaf nodes) to structure traversals without stack/queue structures.
- **Reverse Postorder Traversal:** Processing subtrees in reverse sequence to build dependencies bottom-up.

---

## 5. Brute Force Solution

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

---

## 6. Better Solution (Recursive - Reverse Postorder)

### Algorithm
1. Keep a global/member variable `prev` initialized to `nullptr`.
2. Perform a postorder traversal in the order: Right Subtree $\to$ Left Subtree $\to$ Root.
3. For the current node:
   - Call `flatten(root->right)`.
   - Call `flatten(root->left)`.
   - Set `root->right = prev`.
   - Set `root->left = nullptr`.
   - Update `prev = root`.

### C++17 Code
```cpp
class SolutionBetter {
private:
    TreeNode* prev = nullptr;
public:
    void flatten(TreeNode* root) {
        if (root == nullptr) return;
        
        flatten(root->right);
        flatten(root->left);
        
        root->right = prev;
        root->left = nullptr;
        prev = root;
    }
};
```

### Complexity Analysis
- **Time Complexity:** $O(N)$ where $N$ is the number of nodes.
- **Space Complexity:** $O(H)$ recursion stack space, where $H$ is the height of the tree.

---

## 7. Optimal Solution (Morris-like / Iterative $O(1)$ Space)

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

---

## 8. Code Walkthrough

Let's examine the step-by-step logic inside the `while(curr != nullptr)` loop:
- **Condition `curr->left != nullptr`**:
  - We only need to adjust pointers if a left child exists. If there is no left child, `curr` is already flattened locally (its left is null, and its right points to the next node in preorder), so we just advance `curr = curr->right`.
- **Finding Predecessor**:
  - We start at `curr->left` and loop `while(pred->right != nullptr)`. This locates the absolute last node of `curr`'s left subtree in preorder.
- **Thread Re-linking**:
  - `pred->right = curr->right`: This links the last element of the left subtree directly to the start of the right subtree.
  - `curr->right = curr->left`: We overwrite the right child of `curr` to point to the left child, effectively merging the subtrees.
  - `curr->left = nullptr`: The left child pointer is cleared as required.
- **Advance**:
  - We move `curr` down to `curr->right`. Since we updated `curr->right` to point to the left child, this naturally steps into the left subtree next.

---

## 9. Dry Run

Let's dry run the optimal solution on this tree:
```
    1
   / \
  2   5
 / \   \
3   4   6
```

### Initial State
`curr = 1`

### Step 1: `curr = 1`
- `curr->left = 2` (not null).
- Find predecessor of `1` in left subtree:
  - `pred` starts at `2`.
  - Go right: `pred = 4`. `4->right` is null. Predecessor is `4`.
- Re-link:
  - `4->right = 1->right` $\Rightarrow$ `4->right = 5`.
  - `1->right = 1->left` $\Rightarrow$ `1->right = 2`.
  - `1->left = nullptr`.
- Tree now looks like:
```
    1
     \
      2
     / \
    3   4
         \
          5
           \
            6
```
- Advance `curr = curr->right` $\Rightarrow$ `curr = 2`.

### Step 2: `curr = 2`
- `curr->left = 3` (not null).
- Find predecessor of `2` in left subtree:
  - `pred` starts at `3`. `3->right` is null. Predecessor is `3`.
- Re-link:
  - `3->right = 2->right` $\Rightarrow$ `3->right = 4`.
  - `2->right = 2->left` $\Rightarrow$ `2->right = 3`.
  - `2->left = nullptr`.
- Tree now looks like:
```
    1
     \
      2
       \
        3
         \
          4
           \
            5
             \
              6
```
- Advance `curr = curr->right` $\Rightarrow$ `curr = 3`.

### Step 3: `curr = 3`
- `curr->left` is null.
- Advance `curr = curr->right` $\Rightarrow$ `curr = 4`.

### Subsequent Steps
- Nodes `4`, `5`, `6` all have null left children. We simply advance `curr` until `curr = nullptr`.
- Execution terminates. Tree is flattened.

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(N)$ | $O(N)$ | Storing preorder nodes in a vector. |
| **Recursive (Better)**| $O(N)$ | $O(H)$ | Recursion stack height equal to tree height. |
| **Morris-like (Optimal)**| $O(N)$ | $O(1)$ | We visit each edge at most a constant number of times and use only temporary pointer variables. |

---

## 11. Common Mistakes

1. **Forgetting to set `curr->left = nullptr`:** If left pointers are not explicitly set to null, the tree will not be a single right-skewed linked list, leading to cycles or validation failures.
2. **Connecting `pred->right` to the wrong node:** Make sure it is connected to `curr->right`, not `curr`.
3. **Infinite Loops:** If you don't traverse to `curr = curr->right` correctly, or if you create cyclic connections, the program will hang.

---

## 12. Edge Cases

| Edge Case | Input | Handling |
| :--- | :--- | :--- |
| **Empty Tree** | `nullptr` | Handled; loop does not run. |
| **Single Node** | `[1]` | Handled; `curr->left` is null, loop finishes instantly. |
| **Right Skewed Tree**| `1 -> 2 -> 3` | No left children. Merely traverses right. |
| **Left Skewed Tree** | `1 -> 2 -> 3` (left) | Pointers are rotated to the right step-by-step. |

---

## 13. Interview Explanation

"To flatten a binary tree in-place to a right-skewed preorder linked list with $O(1)$ auxiliary space, we can use a Morris-like traversal. 

For any node we visit, if it has a left child, we find the rightmost node in its left subtree. This rightmost node is the inorder predecessor of our current node's right subtree. We link this predecessor's right child to the current node's right child. Then, we move the entire left subtree to the right side of the current node and set the left pointer to null. 

By repeating this adjustment as we step down the right child of each node, we restructure the tree into a linked list in-place without needing any recursion stack or auxiliary vectors."

---

## 14. Follow-up Questions

1. **Can we do this in postorder or inorder?**
   - *Answer:* Yes, but the redirection logic would change. For postorder, we'd need to reconstruct from bottom-up using a different traversal sequence.
2. **Does this modify the structural integrity of the tree?**
   - *Answer:* Yes, it permanently transforms the tree structure into a list. If the original tree must be preserved, we should make a copy first.

---

## 15. Variations

1. **Flatten BST to sorted DLL:** Keep left and right pointers functioning as previous and next pointers.
2. **Flatten Binary Tree to DLL in-place (inorder):** Standard in-place BST to doubly linked list conversion.

---

## 16. Revision Notes

- **Predecessor rule:** `pred->right = curr->right` followed by moving left child to right.
- **Iterative code skeleton:**
  ```cpp
  while(curr) {
      if(curr->left) {
          // find pred
          // link pred->right = curr->right
          // curr->right = curr->left; curr->left = nullptr
      }
      curr = curr->right;
  }
  ```
- **Memory footprint:** Morris traversal reduces space complexity to $O(1)$.

---

## 17. Memorization Version

```text
Flatten Binary Tree:
While curr exists:
  If curr has left child:
    Find rightmost node of left subtree (pred).
    Set pred->right = curr->right.
    Set curr->right = curr->left.
    Set curr->left = nullptr.
  curr = curr->right.
Complexity: Time O(N), Space O(1).
```
