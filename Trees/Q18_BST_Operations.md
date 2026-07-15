# BST Operations: Search, Insert, and Delete

## 1. Problem Statement

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

---

## 2. Interview Clarification

1.  **Q**: What should we do if we insert a key that already exists?
    *   *A*: Typically, BSTs do not allow duplicates. If the key already exists, return the tree unchanged.
2.  **Q**: When deleting a node with two children, should we replace it with the inorder predecessor or inorder successor?
    *   *A*: Either is valid. In standard implementations, replacing with the **inorder successor** (the smallest node in the right subtree) is preferred.
3.  **Q**: Do we need to write both recursive and iterative versions?
    *   *A*: Yes, demonstrating both is highly recommended as iterative versions save recursion stack space.

---

## 3. Thinking Process

*   **Search**:
    *   *Recursive*: If current node is null or matches `key`, return it. If `key < root->val`, search left. If `key > root->val`, search right.
    *   *Iterative*: Use a simple `while` loop: `curr = key < curr->val ? curr->left : curr->right;` until found or null.

*   **Insert**:
    *   *Recursive*: Walk down standard BST search path. When you reach `nullptr`, create and return a new `TreeNode(key)`. Assign returned pointers back: `root->left = insert(root->left, key)` etc.
    *   *Iterative*: Keep track of a `parent` pointer. Walk down until `curr` becomes null. Insert new node as the left or right child of `parent`.

*   **Delete**:
    *   Find the node to delete using search logic. Once found, handle three cases:
        *   **Case 1: Leaf Node (No children)**: Delete the node and return `nullptr`.
        *   **Case 2: One Child**: Return the non-null child to the caller and delete the current node.
        *   **Case 3: Two Children**:
            1.  Find the inorder successor (minimum node in the right subtree).
            2.  Replace current node's value with the successor's value.
            3.  Recursively delete the successor from the right subtree.

---

## 4. Pattern Recognition

*   **Topic**: Binary Search Tree.
*   **Pattern**: Divide and conquer based on property comparison (`key < root->val` vs `key > root->val`).
*   **Why**: The BST property allows us to discard half of the search space at each step, making operations $O(H)$ instead of $O(N)$.

---

## 5. Brute Force Solution

*   *Note*: There is no "brute force" for BST operations other than treating the BST as a regular binary tree (e.g., searching linearly, reconstructing the tree after insertion/deletion). Since we are working with BST properties, we jump straight to the standard recursive and iterative optimal operations.

---

## 6. Better Solution

*   *Note*: The recursive version is clean but uses $O(H)$ auxiliary space for recursion stack. The iterative version is the optimized "better" solution because it runs in $O(1)$ auxiliary space. We will present both recursive and iterative implementations for all three operations under the optimal section.

---

## 7. Optimal Solution

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

---

## 8. Code Walkthrough

### Search Walkthrough (Iterative)
1.  Initialize `curr = root`.
2.  Loop while `curr != nullptr` and `curr->val != key`.
3.  Branch left if `key < curr->val`, else right.
4.  Return `curr` (which is either the matching node or `nullptr`).

### Insert Walkthrough (Recursive)
1.  If tree is empty, create a new node and return it.
2.  Compare key:
    *   If `key < root->val`: recurse left. Reassign: `root->left = insertRecursive(root->left, key)`.
    *   If `key > root->val`: recurse right. Reassign: `root->right = insertRecursive(root->right, key)`.
3.  Return the `root` pointer.

### Delete Walkthrough (Recursive)
1.  If `root` is null, key is not in tree. Return `nullptr`.
2.  Navigate to the target node: `root->left = deleteRecursive(...)` or `root->right = deleteRecursive(...)`.
3.  Once the target node is found (`root->val == key`):
    *   If `left` is null: save `right`, delete `root`, return `right` (handles leaf and right-child-only).
    *   If `right` is null: save `left`, delete `root`, return `left` (handles left-child-only).
    *   If both children exist: find the minimum node in the right subtree (`findMin(root->right)`). Copy its value to `root`. Then delete that minimum node recursively from the right subtree: `root->right = deleteRecursive(root->right, temp->val)`.

---

## 9. Dry Run

Let's trace `deleteRecursive(root, 5)` on the tree:
```
      5
     / \
    3   7
   / \
  2   4
```

### Dry Run steps:
1.  `deleteRecursive(5, 5)`: Node matches!
2.  Has both left (`3`) and right (`7`) children.
3.  Find min of right subtree: `findMin(7)` starts at `7`. Has no left child, so min is `7`.
4.  Overwrite value: `5` becomes `7`. Tree is now:
```
      7
     / \
    3   7
   / \
  2   4
```
5.  Call `root->right = deleteRecursive(7->right, 7)`.
6.  `deleteRecursive(nullptr, 7)` returns `nullptr`.
7.  `7->right` becomes `nullptr`.
8.  Return `7` to caller. Tree matches expected state.

---

## 10. Complexity Analysis

| Operation | Average Case Time | Worst Case Time | Space (Recursive) | Space (Iterative) |
| :--- | :--- | :--- | :--- | :--- |
| **Search** | $\mathcal{O}(\log N)$ | $\mathcal{O}(N)$ | $\mathcal{O}(H)$ | $\mathcal{O}(1)$ |
| **Insert** | $\mathcal{O}(\log N)$ | $\mathcal{O}(N)$ | $\mathcal{O}(H)$ | $\mathcal{O}(1)$ |
| **Delete** | $\mathcal{O}(\log N)$ | $\mathcal{O}(N)$ | $\mathcal{O}(H)$ | $\mathcal{O}(1)$ |

*   *Note*: The worst case occurs when the tree is skewed (linear list), where height $H = N$. Average case occurs when the tree is balanced, where height $H = \log_2 N$.

---

## 11. Common Mistakes

1.  **Forgetting to Link Subtrees**: In recursive insert/delete, failing to update `root->left` and `root->right` with the returned node pointers:
    *   *Incorrect*: `insert(root->left, key);`
    *   *Correct*: `root->left = insert(root->left, key);`
2.  **Memory Leaks**: Deleting a node in C++ without using `delete` operator, leading to memory leaks.
3.  **Losing Successor Children**: In Case 3 of iterative deletion, when replacing a node with its successor, forgetting to link the successor's right child back to the successor parent's left branch.

---

## 12. Edge Cases

| Edge Case | Scenario | Expected Outcome | Importance |
| :--- | :--- | :--- | :--- |
| **Empty Tree** | Root is `nullptr` | Search returns `nullptr`. Insert/Delete return the new root. | Ensure pointer safety. |
| **Single Node Delete**| Tree has only `[5]`, delete `5` | Return `nullptr` as new root. | Clean memory. |
| **Delete Node with Left Skewed Successor**| Successor is deep left in right subtree. | Successor's right child is successfully linked to successor parent. | Tests Case 3 link rewiring. |

---

## 13. Interview Explanation

"BST operations leverage the binary search property to achieve logarithmic performance. 

For **Search**, we compare the key with the current node value and proceed to the left or right child. For **Insertion**, we follow the search path until we hit a null pointer, where we attach the new node. Both operations run in $O(H)$ time.

**Deletion** is slightly more involved. First, we locate the node. If it has no children or one child, we bypass the node and link the parent directly to the child. If the node has two children, we find the inorder successor (the minimum node in its right subtree), overwrite the target node's value with the successor's value, and then delete that successor node.

While recursive versions are cleaner, iterative versions are more optimal in practice because they run in $O(1)$ auxiliary space instead of consuming recursion stack space."

---

## 14. Follow-up Questions

1.  **Q**: How do we guarantee the average-case $O(\log N)$ behavior for these operations?
    *   *A*: By using self-balancing binary search trees such as AVL Trees or Red-Black Trees, which perform rotations during insertion/deletion to keep height balanced.
2.  **Q**: What happens to delete operation if we allow duplicate values in the BST?
    *   *A*: We would need to traverse and delete all occurrences, or use a count variable inside each node (`count++`/`count--`) instead of physically removing the node.

---

## 15. Variations

1.  **Inorder Successor/Predecessor in BST**: Find the next node in order.
2.  **BST Iterator**: Implement `next()` and `hasNext()` in $O(1)$ average time and $O(H)$ space.

---

## 16. Revision Notes

*   **BST Constraint**: Left < Node < Right.
*   **Case 3 Delete**: Swap with min of right subtree, delete min from right subtree.
*   **Pointer Reassignment**: Always assign returned nodes back to parent left/right pointers.

---

## 17. Memorization Version

```cpp
// Search
TreeNode* search(TreeNode* r, int k) {
    if (!r || r->val == k) return r;
    return k < r->val ? search(r->left, k) : search(r->right, k);
}
// Insert
TreeNode* insert(TreeNode* r, int k) {
    if (!r) return new TreeNode(k);
    if (k < r->val) r->left = insert(r->left, k);
    else if (k > r->val) r->right = insert(r->right, k);
    return r;
}
// Delete
TreeNode* deleteNode(TreeNode* r, int k) {
    if (!r) return nullptr;
    if (k < r->val) r->left = deleteNode(r->left, k);
    else if (k > r->val) r->right = deleteNode(r->right, k);
    else {
        if (!r->left) { TreeNode* t = r->right; delete r; return t; }
        if (!r->right) { TreeNode* t = r->left; delete r; return t; }
        TreeNode* min = r->right;
        while (min->left) min = min->left;
        r->val = min->val;
        r->right = deleteNode(r->right, min->val);
    }
    return r;
}
```
*   **Mnemonic**: Search & Insert are simple binary decisions. Delete has 3 cases: leaf/single child returns the other side; two children copy inorder successor value then delete successor.
