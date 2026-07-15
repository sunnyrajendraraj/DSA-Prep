# Convert BST to Sorted Doubly Linked List

## 1. Problem Statement

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

---

## 2. Interview Clarification

1.  **Q**: Is this a circular doubly linked list or a standard doubly linked list?
    *   *A*: Standard doubly linked list (the `prev` of the head is `nullptr` and the `next` of the tail is `nullptr`). *Note*: If circular, we would simply link the head and tail at the very end. We will solve the standard DLL and show how to make it circular if needed.
2.  **Q**: Can we allocate new nodes?
    *   *A*: No, the conversion must be done entirely in-place by reusing the existing tree node pointers.
3.  **Q**: What should we return if the tree is empty?
    *   *A*: Return `nullptr`.

---

## 3. Thinking Process

*   **First Instinct**:
    *   Perform a standard inorder traversal, store all nodes in a list, and then iterate through the list to link `nodes[i]->right = nodes[i+1]` and `nodes[i]->left = nodes[i-1]`.
    *   *Why this is not optimal*: Storing all nodes in a vector requires $O(N)$ auxiliary space. We should perform the linking dynamically during the traversal to achieve $O(H)$ space.

*   **Optimal Insight**:
    *   We can perform the in-place linking during a recursive inorder traversal using a state variable `prevNode` (initialized to `nullptr`) and `headNode` (initialized to `nullptr`).
    *   When visiting a node `curr`:
        1.  First, recursively convert the left subtree.
        2.  Now, process the current node `curr`:
            *   If `prevNode` is `nullptr`, it means `curr` is the smallest node in the BST. This will be the `head` of our DLL.
            *   If `prevNode` is not `nullptr`, we link them:
                *   `curr->left = prevNode;` (curr's prev pointer points to prevNode)
                *   `prevNode->right = curr;` (prevNode's next pointer points to curr)
            *   Update `prevNode = curr`.
        3.  Finally, recursively convert the right subtree.

---

## 4. Pattern Recognition

*   **Topic**: Tree Pointer Manipulation / Inorder Traversal.
*   **Pattern**: DFS Inorder with state tracking (`prev` pointer tracking).
*   **Why**: Repurposing left/right child pointers into prev/next links during an active traversal requires cautious order of execution. An inorder DFS naturally visits nodes in sorted order, ensuring we link them in the correct sequence.

---

## 5. Brute Force Solution

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

---

## 6. Better Solution

*   *Note*: The transition from the $O(N)$ space array solution straight to the in-place recursive inorder pointer manipulation yields the optimal solution. There is no intermediate "better" solution. We will jump directly to the optimal solution.

---

## 7. Optimal Solution

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

---

## 8. Code Walkthrough

1.  **Reference Parameters (`TreeNode*& prevNode, TreeNode*& headNode`)**:
    *   We pass these pointers by reference to ensure updates to `prevNode` and `headNode` persist across recursive frames.
2.  **Left Subtree Traversal**:
    *   `convertToDLL(curr->left, ...)` executes first, running down to the minimum node in the BST.
3.  **Head Assignment**:
    *   When the leftmost leaf is reached, `prevNode` is still `nullptr`.
    *   `headNode = curr` designates this node as the entry point of the list.
4.  **Pointer Rewiring**:
    *   `curr->left = prevNode` rewires the left pointer to the previous node.
    *   `prevNode->right = curr` rewires the right pointer of the previous node to point to the current node.
5.  **State Update**:
    *   `prevNode = curr` updates the traversal state.
6.  **Right Subtree Traversal**:
    *   `convertToDLL(curr->right, ...)` continues the inorder sequence.

---

## 9. Dry Run

Consider the tree:
```
      4
     / \
    2   5
```

*   **Start**: `prevNode = nullptr`, `headNode = nullptr`.

1.  `curr = 4`: Recurse left $\rightarrow$ `curr = 2`.
2.  `curr = 2`: Recurse left $\rightarrow$ `curr = nullptr` (returns).
3.  `curr = 2`:
    *   `prevNode` is `nullptr` $\rightarrow$ `headNode = 2`.
    *   `prevNode = 2`.
    *   Recurse right $\rightarrow$ `curr = nullptr` (returns).
    *   Return from `curr = 2` frame.
4.  `curr = 4`:
    *   `prevNode` is `2` (not null).
    *   `4->left = 2`.
    *   `2->right = 4`.
    *   `prevNode = 4`.
    *   Recurse right $\rightarrow$ `curr = 5`.
5.  `curr = 5`: Recurse left $\rightarrow$ `curr = nullptr` (returns).
6.  `curr = 5`:
    *   `prevNode` is `4` (not null).
    *   `5->left = 4`.
    *   `4->right = 5`.
    *   `prevNode = 5`.
    *   Recurse right $\rightarrow$ `curr = nullptr` (returns).
    *   Return from `curr = 5` frame.
*   **Result**: Head is `2`.
    *   `2 <-> 4 <-> 5`.

---

## 10. Complexity Analysis

*   **Time Complexity**: $\mathcal{O}(N)$
    *   Every node in the tree is visited exactly once.
*   **Space Complexity**: $\mathcal{O}(H)$
    *   $H$ is the height of the tree.
    *   No extra memory is allocated for list nodes. The space is used by the recursion stack.
    *   For a balanced tree, $H = \log N$, resulting in $\mathcal{O}(\log N)$ space. For a skewed tree, $H = N$, resulting in $\mathcal{O}(N)$ space.

---

## 11. Common Mistakes

1.  **Modifying Right Pointer Before Recursing Right**: If you assign `prevNode->right = curr` without understanding the sequence of recursion, you might overwrite a node's child pointers before you have traversed them. Because our code does the linking step *after* left subtree recursion and *before* right subtree recursion, the right child pointer `curr->right` is only modified *after* we have fully processed the left side and are about to enter `curr->right`.
2.  **Not passing pointers by reference**: If `prevNode` is not passed by reference (`TreeNode*&`), updates to `prevNode` are lost when returning from recursion levels.

---

## 12. Edge Cases

| Edge Case | Input Tree | Expected DLL Output | Importance |
| :--- | :--- | :--- | :--- |
| **Empty Tree** | `nullptr` | `nullptr` | Avoid null pointer dereference. |
| **Single Node** | `[1]` | `1` (prev = null, next = null) | Verify single-node handling. |
| **Skewed Left Tree** | `3 -> 2 -> 1` | `1 <-> 2 <-> 3` | Checks conversion of linear trees. |

---

## 13. Interview Explanation

"To convert a Binary Search Tree to a sorted Doubly Linked List in-place, we must utilize an Inorder Traversal, which visits the nodes in sorted order. 

We can perform this conversion recursively. We maintain two pointers throughout the traversal: `headNode` (which will point to the start of our list) and `prevNode` (which tracks the node visited immediately before the current node).

When we visit a node, we first recursively convert its left subtree. Then, we link the current node with `prevNode`. If `prevNode` is null, it means we are at the leftmost node, which becomes our `headNode`. Otherwise, we set the current node's left pointer to `prevNode` and the `prevNode`'s right pointer to the current node. Finally, we update `prevNode` to the current node and recursively convert the right subtree. This conversion is fully in-place, using $O(N)$ time and $O(H)$ recursion stack space."

---

## 14. Follow-up Questions

1.  **Q**: How would you make this a circular doubly linked list?
    *   *A*: After the recursion completes, we have `headNode` pointing to the first element and `prevNode` pointing to the last element (tail). We simply link them: `headNode->left = prevNode; prevNode->right = headNode;`.
2.  **Q**: Can you implement this iteratively?
    *   *A*: Yes, using a stack to perform iterative inorder traversal. As we pop each node, we perform the same linking logic using a `prevNode` pointer.

---

## 15. Variations

1.  **Convert Sorted DLL to Balanced BST**:
    *   *Solution*: Find the middle element of the DLL (using slow/fast pointers) to act as root, and recursively construct left and right subtrees.
2.  **Flatten Binary Tree to Linked List**:
    *   *Solution*: Flatten a binary tree to a singly linked list in preorder sequence in-place.

---

## 16. Revision Notes

*   **Repurposing**: Left = Prev, Right = Next.
*   **Recursive Order**: Recurse Left $\rightarrow$ Link `curr` to `prevNode` $\rightarrow$ Update `prevNode = curr` $\rightarrow$ Recurse Right.
*   **Reference passing**: `TreeNode*&` is mandatory for tracking state.

---

## 17. Memorization Version

```cpp
void convert(TreeNode* curr, TreeNode*& prev, TreeNode*& head) {
    if (!curr) return;
    convert(curr->left, prev, head);
    if (!prev) head = curr;
    else {
        curr->left = prev;
        prev->right = curr;
    }
    prev = curr;
    convert(curr->right, prev, head);
}
TreeNode* bstToDoublyList(TreeNode* root) {
    if (!root) return nullptr;
    TreeNode *prev = nullptr, *head = nullptr;
    convert(root, prev, head);
    return head;
}
```
*   **Mnemonic**: Inorder traversal structure. In the middle block, if no `prev`, set `head`; else wire `curr <-> prev`. Set `prev = curr`.
*   **Complexity**: $O(N)$ time, $O(H)$ space.
