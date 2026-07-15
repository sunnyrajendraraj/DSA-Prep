# Flatten a Multilevel Linked List

## 1. Problem Statement

Given a linked list where in addition to the `next` pointer, each node has a `child` pointer, which may point to a separate singly linked list. These child lists may also have one or more children of their own, and so on, creating a multilevel data structure.

Flatten the list so that all the nodes appear in a single-level singly linked list. The flattening should follow a **Depth-First Search (DFS)** order: if a node has a child, the entire child list must be flattened and inserted after the current node before we continue with the next node in the original list.

### Input
- `head`: A head pointer to the multilevel linked list.

### Output
- A head pointer to the flattened single-level linked list. All `child` pointers must be set to `nullptr`.

### Constraints
- The number of nodes in the list is in the range $[0, 1000]$.
- $1 \le \text{Node.val} \le 10^5$.

### Observations
1. This is essentially a tree structure where `child` is like a left child and `next` is like a right child.
2. The traversal order is preorder DFS: visit current node, then recurse on child (left), then recurse on next (right).
3. We need to reset the `child` pointers of all nodes to `nullptr` in the final output.

---

## 2. Interview Clarification

Before starting, clarify the following details:
1. **Is the list singly or doubly linked?**
   * *Answer*: Singly linked for this standard problem (using `next` and `child`). If it's doubly linked (like LeetCode 430), we would also need to update `prev` pointers. We will focus on the singly linked version and discuss doubly linked in the variations.
2. **Should we flatten in-place or construct a new list?**
   * *Answer*: In-place. Modify the existing nodes' pointers. Do not allocate new nodes.
3. **What is the order of flattening?**
   * *Answer*: DFS Preorder. We traverse down the child branch before moving to the next node in the current level.

---

## 3. Thinking Process

* **First Instinct (Brute Force)**:
  * Since the order is DFS, we can perform a standard preorder DFS traversal.
  * During the traversal, we can store all the visited nodes in a dynamic array/vector.
  * Once we have all the nodes in the array, we can iterate through the array and link `arr[i]->next = arr[i+1]` and set `arr[i]->child = nullptr`.
  * This is correct and simple. However, it uses $O(N)$ extra space to store the nodes in the array.
* **Better Approach (In-place Iterative Stack)**:
  * Can we link nodes as we traverse, using a stack to keep track of where to return?
  * Yes, when we pop a node `curr` from the stack:
    * If `curr` has a `next` node, we push `curr->next` to the stack.
    * If `curr` has a `child` node, we push `curr->child` to the stack.
    * Then we link `curr->next` to the top of the stack (if it exists) and clear `curr->child`.
  * Because a stack is Last-In-First-Out (LIFO), pushing `next` first and then `child` ensures that `child` is popped first. This matches the DFS preorder.
  * The space complexity is $O(D)$ where $D$ is the maximum depth of the stack (in the worst case of a vertical-only list, $O(N)$).
* **Optimal Approach (In-place O(1) Auxiliary Space)**:
  * Can we avoid using a stack or recursion altogether?
  * Yes, by flattening from left to right as we traverse.
  * At any node `curr`:
    * If `curr` has no child, we simply move to `curr->next`.
    * If `curr` has a child:
      1. Find the tail of the child list (let's call it `childTail`).
      2. Link `childTail->next = curr->next`.
      3. Link `curr->next = curr->child`.
      4. Set `curr->child = nullptr`.
      5. Move `curr` to `curr->next` (which is now the head of the child list).
    * As we traverse forward, we will naturally step into the child list nodes and flatten any child lists they might have!
  * This requires only a few pointer variables, yielding $O(1)$ auxiliary space.

---

## 4. Pattern Recognition

1. **Depth-First Search (DFS) / Preorder Traversal**: The child list has priority over the next list, matching tree preorder traversal.
2. **In-place Pointer Rewriting**: Manipulating pointers to stitch sublists together without using extra memory.
3. **Iterative Flattening**: Pushing child branches forward along the traverse path to process them sequentially.

---

## 5. Brute Force Solution (Collection Array)

### Algorithm
1. Create a helper function `dfs(Node* node, vector<Node*>& nodes)`:
   * If `node` is null, return.
   * Push `node` to `nodes`.
   * Recurse on `node->child`.
   * Recurse on `node->next`.
2. In the main function, call `dfs(head, nodes)`.
3. Loop from `i = 0` to `nodes.size() - 2`:
   * Link `nodes[i]->next = nodes[i+1]`.
   * Set `nodes[i]->child = nullptr`.
4. Set the `next` and `child` of the last node to `nullptr`.
5. Return `head`.

### Complexity Analysis
* **Time Complexity**: $O(N)$ where $N$ is the total number of nodes in the multilevel list.
* **Space Complexity**: $O(N)$ for the vector and the recursion stack.

### Commented C++17 Code
```cpp
#include <vector>

// Definition for a Node.
struct Node {
    int val;
    Node* next;
    Node* child;
    Node(int x) : val(x), next(nullptr), child(nullptr) {}
};

class BruteFlattenMultilevel {
private:
    void dfs(Node* node, std::vector<Node*>& nodes) {
        if (node == nullptr) return;
        nodes.push_back(node);
        dfs(node->child, nodes);
        dfs(node->next, nodes);
    }

public:
    Node* flatten(Node* head) {
        if (head == nullptr) return nullptr;
        
        std::vector<Node*> nodes;
        dfs(head, nodes);
        
        // Relink all nodes sequentially
        for (size_t i = 0; i < nodes.size() - 1; ++i) {
            nodes[i]->next = nodes[i+1];
            nodes[i]->child = nullptr;
        }
        nodes.back()->next = nullptr;
        nodes.back()->child = nullptr;
        
        return head;
    }
};
```

---

## 6. Better Solution (Iterative Stack-based DFS)

### Improvement over Brute Force
Instead of storing all nodes in a vector, we perform a DFS iteratively using a stack, updating the pointers on the fly. This reduces space to the maximum depth of branches rather than all nodes.

### Algorithm
1. Push `head` to a stack.
2. Keep a `prev` pointer initialized to `nullptr`.
3. While the stack is not empty:
   * Pop `curr`.
   * If `prev` is not null, link `prev->next = curr` and `prev->child = nullptr`.
   * Push `curr->next` to the stack (if it exists).
   * Push `curr->child` to the stack (if it exists).
   * Update `prev = curr`.
4. Set `prev->next = nullptr` and `prev->child = nullptr` for the final node.
5. Return `head`.

### Complexity Analysis
* **Time Complexity**: $O(N)$ as each node is pushed and popped once.
* **Space Complexity**: $O(D)$ where $D$ is the maximum depth of child hierarchies (at most $O(N)$).

### Commented C++17 Code
```cpp
#include <stack>

class StackFlattenMultilevel {
public:
    Node* flatten(Node* head) {
        if (head == nullptr) return nullptr;

        std::stack<Node*> st;
        st.push(head);
        Node* prev = nullptr;

        while (!st.empty()) {
            Node* curr = st.top();
            st.pop();

            if (prev != nullptr) {
                prev->next = curr;
                prev->child = nullptr; // Reset child pointer
            }

            // Push next first so that child is popped first (DFS)
            if (curr->next != nullptr) {
                st.push(curr->next);
            }
            if (curr->child != nullptr) {
                st.push(curr->child);
            }

            prev = curr;
        }

        return head;
    }
};
```

---

## 7. Optimal Solution (In-place O(1) Space Pointer Stitching)

### Best Approach
Iterate through the list. Whenever a node has a child, find the tail of that child list, link the tail to the current node's next node, make the child node the current node's next node, and set the child pointer to null. Continue traversing forward.

### Algorithm
1. Initialize `curr` to `head`.
2. While `curr` is not null:
   * If `curr` has a `child`:
     * Find the tail of the child list: `Node* temp = curr->child;` and loop `while (temp->next) temp = temp->next;`.
     * Link the child tail's `next` to `curr->next`: `temp->next = curr->next;`.
     * Link `curr->next` to `curr->child`: `curr->next = curr->child;`.
     * Clear the child pointer: `curr->child = nullptr;`.
   * Move `curr` to `curr->next`.
3. Return `head`.

### Why it is Optimal
* **Time**: $O(N)$. Although we have nested loops, each node is visited at most twice (once during main traversal, and at most once when locating the tail of a child list).
* **Space**: $O(1)$ auxiliary space since we do not use recursion stack or stack data structures.

### C++17 Code
```cpp
class OptimalFlattenMultilevel {
public:
    Node* flatten(Node* head) {
        if (head == nullptr) return nullptr;

        Node* curr = head;
        while (curr != nullptr) {
            // If the current node has a child list
            if (curr->child != nullptr) {
                // Find the tail of the child list
                Node* childTail = curr->child;
                while (childTail->next != nullptr) {
                    childTail = childTail->next;
                }

                // Connect the tail of the child list to the next of current
                childTail->next = curr->next;

                // Make the child the next of current
                curr->next = curr->child;

                // Clear the child pointer
                curr->child = nullptr;
            }
            // Move to the next node
            curr = curr->next;
        }
        return head;
    }
};
```

---

## 8. Code Walkthrough

* `Node* curr = head;`
  * Start traversing from the root list head.
* `if (curr->child != nullptr)`
  * Check if the current node branches downwards.
* `Node* childTail = curr->child;`
  * Initialize a pointer to scan the child list.
* `while (childTail->next != nullptr) childTail = childTail->next;`
  * Walk until we reach the end node of the child list.
* `childTail->next = curr->next;`
  * Link the last node of the child list to the node that followed `curr`.
* `curr->next = curr->child;`
  * Link `curr` to the head of the child list.
* `curr->child = nullptr;`
  * Clean up the child pointer.
* `curr = curr->next;`
  * Move forward. Note that `curr->next` now points to the child head we just inserted. In subsequent loops, we will traverse and flatten any child lists nested inside this child list.

---

## 9. Dry Run

Let's dry run the optimal solution on the following structure:
```
1 -> 2 -> 3
|
4 -> 5
```
* **Step 1**: `curr = 1`.
  * `curr->child` is `4`.
  * Find tail of `4 -> 5` $\rightarrow$ `childTail = 5`.
  * Link `childTail->next = curr->next` $\rightarrow$ `5->next = 2`.
  * Link `curr->next = curr->child` $\rightarrow$ `1->next = 4`.
  * `curr->child = nullptr`.
  * List becomes: `1 -> 4 -> 5 -> 2 -> 3`.
  * Move `curr = curr->next` $\rightarrow$ `curr = 4`.
* **Step 2**: `curr = 4`.
  * `curr->child` is null.
  * Move `curr = curr->next` $\rightarrow$ `curr = 5`.
* **Step 3**: `curr = 5`.
  * `curr->child` is null.
  * Move `curr = curr->next` $\rightarrow$ `curr = 2`.
* **Step 4**: `curr = 2`.
  * `curr->child` is null.
  * Move `curr = curr->next` $\rightarrow$ `curr = 3`.
* **Step 5**: `curr = 3`.
  * `curr->child` is null.
  * Move `curr = curr->next` $\rightarrow$ `curr = nullptr`. Loop terminates.
* Return `1 -> 4 -> 5 -> 2 -> 3`.

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Description |
|:---|:---:|:---:|:---|
| **Brute Force** | $O(N)$ | $O(N)$ | Simple but uses linear memory. |
| **Better (Stack)** | $O(N)$ | $O(D)$ | Standard DFS, uses stack proportional to depth. |
| **Optimal (In-place)** | $O(N)$ | $O(1)$ | No extra memory, highly efficient. |

* **Time Complexity**: $O(N)$ where $N$ is the number of nodes. Even though we find the tail of child lists using a nested loop, we traverse each node link a constant number of times. Specifically, no node's `next` link is traversed more than twice.
* **Space Complexity**: $O(1)$ auxiliary space as we only redirect pointers.

---

## 11. Common Mistakes

1. **Dangling Child Pointer**: Forgetting to set `curr->child = nullptr` after stitching. This leaves the child pointer active, violating the flat constraint.
2. **Losing the Next Reference**: In recursive or stack implementations, overriding `curr->next` before saving its value, causing the rest of the list to be lost.
3. **Infinite Loops**: Advancing the pointer incorrectly or looping indefinitely while looking for the child tail.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it Matters |
|:---|:---|:---|:---|
| **Empty List** | `nullptr` | `nullptr` | Guard clause prevents null-pointer dereference. |
| **No Child Links** | `1 -> 2 -> 3` | `1 -> 2 -> 3` | Code should pass through without changes. |
| **Only Child Links** | `1` with child `2` with child `3` | `1 -> 2 -> 3` | Verifies vertical depth-first stitching. |
| **Nested Childs** | `1` child `2` child `3`, next `4` | `1 -> 2 -> 3 -> 4` | Verifies multiple levels of deep stitching. |

---

## 13. Interview Explanation

"To flatten a multilevel linked list in DFS order, we need to insert the child list of any node directly after it, before we continue with the next node in the original list.

A stack-based iterative DFS approach uses $O(N)$ time and $O(D)$ stack space, where $D$ is the depth.

To optimize space to $O(1)$, we can perform an in-place traversal. As we iterate through the list:
1. If the current node has a child, we locate the tail of this child list.
2. We link the child tail's `next` pointer to the current node's `next` node.
3. We then set the current node's `next` to the child node, and clear the `child` pointer to `nullptr`.
4. We move forward to the next node.
Because we insert child lists inline, our traversal will naturally step into them and flatten nested children on the fly. This runs in $O(N)$ time and uses $O(1)$ extra space."

---

## 14. Follow-up Questions

1. **What if the list is Doubly Linked? (e.g. LeetCode 430)**
   * *Answer*: We must update the `prev` pointers. When linking the child head to `curr`, set `child->prev = curr`. When linking the child tail to `curr->next` (if it exists), set `curr->next->prev = childTail`.
2. **Can you solve this recursively?**
   * *Answer*: Yes, we can write a function `Node* flatten_rec(Node* head)` that flattens the list and returns the tail of the flattened list, making it easy to link with the next node. It takes $O(N)$ space due to the stack.

---

## 15. Variations

1. **Flattening a Linked List (Sorted Column/Row)**:
   * *Difference*: Nodes have `next` and `bottom` pointers, where each column list is sorted, and we want the final list to be sorted.
   * *Solution*: Use 2-way merging recursively from right to left (similar to merge sort merge step).

---

## 16. Revision Notes

* **Core Mnemonic**: "Stitch child's tail to current's next; redirect current to child."
* **Visualization**:
  ```
  curr -> (child head) -> ... -> (child tail) -> (curr->next)
  ```
* **Complexity summary**: Time: $O(N)$ | Space: $O(1)$ (Optimal) vs $O(N)$ (Brute).

---

## 17. Memorization Version

```cpp
// 1-min Review: Flatten Multilevel LL O(1) Space
Node* flatten(Node* head) {
    Node* curr = head;
    while (curr) {
        if (curr->child) {
            Node* tail = curr->child;
            while (tail->next) tail = tail->next; // Find tail
            tail->next = curr->next;              // Link tail to next
            curr->next = curr->child;             // Link curr to child
            curr->child = nullptr;                // Clear child
        }
        curr = curr->next;
    }
    return head;
}
```
