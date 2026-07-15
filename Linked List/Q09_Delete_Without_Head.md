# Delete Node Without Head Pointer

## 1. Problem Statement

You are given a pointer to a node `node` in a singly linked list. You do not have access to the `head` of the list. You must delete the given node from the list.

### Input
- `node`: A pointer to the node to be deleted.

### Output
- No return value. The node must be deleted in-place by mutating the structure of the linked list.

### Constraints
- The list will contain at least two elements.
- The node to be deleted is guaranteed to be in the list and is **not** the last node of the list.
- node value is an integer.

### Observations
1. In a singly linked list, to delete a node $X$, we typically need to modify the `next` pointer of the node preceding $X$ ($X_{\text{prev}}$) to point to the node succeeding $X$ ($X_{\text{next}}$).
2. Since we do not have the head pointer, we cannot traverse the list from the beginning to find $X_{\text{prev}}$.
3. We are guaranteed that the node is not the last node. This means `node->next` is never `nullptr`.

---

## 2. Interview Clarification

Before starting, ask the interviewer:
1. **Can the node to delete be the last node in the list?**
   * *Answer*: No, the constraints guarantee that it will not be the last node.
2. **What if the node to delete is the only node in the list?**
   * *Answer*: The list is guaranteed to have at least two elements.
3. **Do we need to free the memory of the deleted node?**
   * *Answer*: Yes, in C++ we should deallocate the memory of the node we remove to prevent memory leaks.
4. **Does this operation change the address of the node that was passed?**
   * *Answer*: Since we cannot change the pointers pointing to this node, the node passed remains at the same memory address, but its value and next pointer will be changed.

---

## 3. Thinking Process

Let's think through how to solve this:
* **The Dilemma**:
  * We want to delete `node`.
  * Normally: `prev->next = node->next; delete node;`
  * But we don't have `prev`! We cannot find it. Singly linked list pointers only go forward.
* **The Trick (Insight)**:
  * If we cannot change the pointer of `prev` to skip `node`, can we make `node` look exactly like `node->next`, and then delete `node->next` instead?
  * Yes! We can copy the value of `node->next` into `node->val`.
  * Then, we can skip `node->next` by setting `node->next = node->next->next`.
  * Finally, we free the memory of the original `node->next`.
* **Example**:
  * List: `1 -> 2 -> 3 -> 4 -> nullptr`
  * Node to delete: `2`.
  * Step 1: Copy value of next node `3` into `2`.
    * List becomes: `1 -> 3 -> 3 -> 4 -> nullptr`. (We now have two `3` nodes).
  * Step 2: Skip the next node. Make the first `3` point to `4`.
    * List becomes: `1 -> 3 -> 4 -> nullptr`.
  * Step 3: Deallocate the second `3`.
  * This effectively deleted the node containing value `2` in $O(1)$ time and space!

---

## 4. Pattern Recognition

This problem uses the **Value Copy & Pointer Redirection** pattern.
* When we cannot access the predecessor in a singly linked list, we can shift the values of subsequent nodes (or just the immediate next node) to mimic deletion.
* It highlights that "deleting a node" can be achieved by changing the node's state rather than physical pointer removal from the preceding node.

---

## 5. Brute Force Solution

> [!NOTE]
> Since we do not have access to the head pointer, there is no way to traverse the list from the beginning to find the predecessor node. Thus, a traditional traversal-based brute force solution is impossible. The brute force jumps directly to the optimal trick-based solution.

---

## 6. Better Solution

> [!NOTE]
> There is no intermediate "better" solution. Any solution that attempts to copy values from all subsequent nodes to the left sequentially would take $O(N)$ time. The optimal solution does this in $O(1)$ time by only copying the immediate next node's value.

---

## 7. Optimal Solution (Copy Next Node & Delete Next)

### Best Approach
Copy the data from the next node into the current node, then delete the next node by setting the current node's `next` pointer to point to the next node's `next` pointer.

### Algorithm
1. Store a temporary pointer `temp` pointing to the next node: `ListNode* temp = node->next;`.
2. Copy the value of the next node to the current node: `node->val = temp->val;`.
3. Link the current node to the node after the next node: `node->next = temp->next;`.
4. Free/delete `temp` to prevent memory leaks: `delete temp;`.

### Why it is Optimal
* **Time**: It requires exactly $O(1)$ operations because we only look at the node and its immediate successor.
* **Space**: $O(1)$ auxiliary space since we only use one temporary pointer variable.

### C++17 Code
```cpp
// Definition for singly-linked list.
struct ListNode {
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class OptimalDeleteNode {
public:
    void deleteNode(ListNode* node) {
        // Since we are guaranteed the node is not the last node,
        // node->next is never nullptr.
        ListNode* temp = node->next;
        
        // Copy the value of the next node into the current node
        node->val = temp->val;
        
        // Skip the next node
        node->next = temp->next;
        
        // Free the memory of the skipped node
        delete temp;
    }
};
```

---

## 8. Code Walkthrough

* `ListNode* temp = node->next;`
  * We need to save the address of `node->next` because we are about to overwrite `node->next` in the current node. If we don't save it, we lose the pointer and cannot call `delete` on it, causing a memory leak.
* `node->val = temp->val;`
  * Overwrites the current node's value with the next node's value. For example, if current is `2` and next is `3`, the current node now holds `3`.
* `node->next = temp->next;`
  * Re-routes the current node's next pointer to point to `temp->next` (skipping the next node).
* `delete temp;`
  * Frees the memory of the node we skipped (which was originally `node->next`).

---

## 9. Dry Run

Let's dry run the optimal solution on the list: `4 -> 5 -> 1 -> 9 -> nullptr` and the node to delete is `5`.

| Step | State of List | Variables | Description |
|:---:|:---|:---|:---|
| Initial | `4 -> [5] -> 1 -> 9 -> nullptr` | `node` points to `5` | Start state |
| 1 | `4 -> [5] -> 1 -> 9 -> nullptr` | `temp = node->next` (points to `1`) | Save next node reference |
| 2 | `4 -> [1] -> 1 -> 9 -> nullptr` | `node->val = temp->val` | Copy value `1` into `node` |
| 3 | `4 -> [1] ------> 9 -> nullptr` | `node->next = temp->next` | Skip node `temp` (points node to `9`) |
| 4 | `4 -> 1 -> 9 -> nullptr` | `delete temp` | Free the original node containing value `1` |

**Final List**: `4 -> 1 -> 9 -> nullptr`.

---

## 10. Complexity Analysis

* **Time Complexity**: $O(1)$. We perform a constant number of operations: one value assignment, one pointer assignment, and one memory deallocation.
* **Space Complexity**: $O(1)$ auxiliary space as we only use a single temporary pointer `temp`.

---

## 11. Common Mistakes

1. **Memory Leaks**: Directly setting `node->next = node->next->next` without calling `delete` on the skipped node in languages like C++ that do not have automatic garbage collection.
2. **Dereferencing Null Pointer**: Not checking if `node` or `node->next` is null. Though the problem constraints guarantee it, in real-world scenarios, always guard against null pointer exceptions.
3. **Attempting to Delete Last Node**: If `node` is the last node, `temp = node->next` is `nullptr`. Copying `temp->val` will result in a segmentation fault.

---

## 12. Edge Cases

| Edge Case | Input Example | Why it Matters | How Code Handles It |
|:---|:---|:---|:---|
| **Second to Last Node** | `1 -> 2 -> 3 -> nullptr`, delete `2` | `node->next` is the last node, which points to `nullptr`. | `node->val` becomes `3`, `node->next` becomes `nullptr`, last node is deleted. Works perfectly. |
| **Middle Node** | `1 -> 2 -> 3 -> 4`, delete `2` | Standard case. | Replaces value of `2` with `3`, deletes original `3` node. Works. |
| **Last Node (Limitations)** | `1 -> 2 -> 3 -> nullptr`, delete `3` | Cannot copy value because next is null. | **Unsupported by this algorithm.** Must be handled by clearing pointers from predecessor. |

### Why We Cannot Delete the Last Node
If we are given the last node, we cannot copy the value from `node->next` because `node->next` is `nullptr`. If we simply try to delete the last node, the predecessor node will still point to it, making it a **dangling pointer**. To delete the last node, we must traverse from the head to find the predecessor and set its `next` pointer to `nullptr`. Without the head, this is mathematically impossible in a singly linked list.

---

## 13. Interview Explanation

"To delete a node from a singly linked list without access to the head pointer, we cannot use the standard approach of updating the predecessor's next pointer. Instead, we can copy the value of the next node into the current node, and then skip the next node by setting `node->next` to `node->next->next`. Finally, we free the memory of the skipped node.

This runs in $O(1)$ time and uses $O(1)$ space. However, this approach has a major limitation: it cannot delete the last node of the list because there is no next node from which we can copy the value."

---

## 14. Follow-up Questions

1. **How would you handle deleting the last node if you were forced to?**
   * *Answer*: Without the head pointer, we cannot delete it properly because the second-to-last node will still point to it. One workaround is to mark the last node as a dummy node (like setting its value to a sentinel value like -1 or `INT_MIN`) to represent that it is deleted, but we cannot free its memory or remove it from the pointer chain.
2. **What if the list is a Doubly Linked List?**
   * *Answer*: In a doubly linked list, we can easily delete the last node (or any node) in $O(1)$ time without the head pointer because we can access the predecessor via the `prev` pointer: `node->prev->next = node->next; if (node->next) node->next->prev = node->prev; delete node;`.

---

## 15. Variations

1. **Remove Duplicates from Unsorted Linked List**:
   * *Relation*: Often requires deleting arbitrary nodes. If we use a hash set, we can delete nodes by tracking the predecessor, but if we don't have it, value copying might be used (though tracking the predecessor is much cleaner).
2. **Delete Nodes Having Greater Value on Right**:
   * *Relation*: Requires reversing or traversing to delete nodes that don't match a condition.

---

## 16. Revision Notes

* **Core Mnemonic**: "Become your neighbor, then kill your neighbor".
* **Visual Representation**:
  ```
  [2] -> [3] -> [4]
   ^      ^
  node   temp
  
  Copy value:
  [3] -> [3] -> [4]
  
  Link next:
  [3] --------> [4]
  ```
* **Limitation**: Cannot delete the last node.

---

## 17. Memorization Version

```cpp
// 1-min Review: Delete Node Without Head (O(1) time & space)
void deleteNode(ListNode* node) {
    ListNode* temp = node->next; // Save next node pointer
    node->val = temp->val;       // Copy next node's value
    node->next = temp->next;     // Skip next node
    delete temp;                 // Free memory of skipped node
}
// Note: Only works if node is not the last node!
```
