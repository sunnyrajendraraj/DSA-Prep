# Delete All Occurrences of a Key in a Linked List

## 1. Problem Statement

Given the `head` of a linked list and an integer `val`, remove all the nodes of the linked list that has `Node.val == val`, and return the new head of the modified linked list.

### Input
- `head`: A pointer to the head of a singly linked list.
- `val`: An integer representing the target key to be deleted.

### Output
- A pointer to the head of the modified linked list.

### Constraints
- The number of nodes in the list is in the range $[0, 10^4]$.
- $1 \le \text{Node.val} \le 50$
- $0 \le val \le 50$

### Observations
1. The target value `val` might appear at the head of the list, possibly multiple times consecutively (e.g., `2 -> 2 -> 3`, deleting `2`).
2. The target value might appear at the tail of the list.
3. The list might become empty after deleting all nodes (e.g., `2 -> 2`, deleting `2`).
4. In languages with manual memory management like C++, we should properly deallocate (delete) the removed nodes to prevent memory leaks.

---

## 2. Interview Clarification

Before coding, clarify:
1. **Q:** Should we delete the matching nodes from memory in C++?
   **A:** Yes, explicitly using the `delete` operator is best practice in C++ interviews.
2. **Q:** Can the list be empty initially?
   **A:** Yes, constraints indicate the size can be 0.
3. **Q:** What should we return if all nodes are deleted?
   **A:** Return `nullptr`.

---

## 3. Thinking Process

Let's build the intuition step-by-step:
1. **How to delete a node?**
   - If we have a node `curr` and its predecessor `prev`, we delete `curr` by setting `prev->next = curr->next`.
2. **Handling Head Deletions:**
   - If the head node contains the target value, we don't have a `prev` node.
   - We would have to write special logic: update `head = head->next` continuously until the head node does not contain `val`.
   - This approach is error-prone and verbose.
3. **The Dummy Node Trick:**
   - What if we create a temporary node `dummy` and set `dummy->next = head`?
   - Now, the head node is no longer a special case! It has a predecessor `dummy`.
   - We set `prev = dummy` and `curr = head`.
   - If `curr->val == val`, we update `prev->next = curr->next`, delete `curr`, and advance `curr` to `prev->next`.
   - If `curr->val != val`, we move `prev = curr` and `curr = curr->next`.
   - At the end, we return `dummy->next` and clean up `dummy`.

---

## 4. Pattern Recognition

This problem utilizes the **Dummy Node Pattern**.
- **Why this pattern?** It eliminates special cases for modifying, deleting, or inserting nodes at the beginning of a linked list.

---

## 5. Brute Force Solution

### Algorithm
1. Traverse the linked list and copy the values of all nodes that are NOT equal to `val` into an array/vector.
2. Create a new linked list using the values from the array.
3. Delete the original linked list to prevent memory leaks.
4. Return the head of the new list.

### Complexity Analysis
- **Time Complexity:** $O(N)$ because we traverse the list and rebuild it.
- **Space Complexity:** $O(N)$ auxiliary space to store the values.

### C++17 Code
```cpp
#include <vector>

struct ListNode {
    int val;
    ListNode* next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode* next) : val(x), next(next) {}
};

class SolutionBrute {
public:
    ListNode* removeElements(ListNode* head, int val) {
        std::vector<int> remaining_vals;
        ListNode* curr = head;
        
        // Step 1: Collect non-target values
        while (curr != nullptr) {
            if (curr->val != val) {
                remaining_vals.push_back(curr->val);
            }
            curr = curr->next;
        }
        
        // Free original list memory
        curr = head;
        while (curr != nullptr) {
            ListNode* temp = curr;
            curr = curr->next;
            delete temp;
        }
        
        // Step 2: Reconstruct list
        ListNode dummy(0);
        ListNode* tail = &dummy;
        for (int v : remaining_vals) {
            tail->next = new ListNode(v);
            tail = tail->next;
        }
        
        return dummy.next;
    }
};
```

---

## 6. Better Solution (In-place without Dummy Node)

### Algorithm
1. Loop while `head != nullptr` and `head->val == val`. In each step, move `head = head->next` (and delete the old head).
2. If `head` is now `nullptr`, return `nullptr`.
3. Otherwise, set `curr = head`.
4. While `curr->next != nullptr`:
   - If `curr->next->val == val`, we link `curr->next = curr->next->next` (and delete the skipped node).
   - Otherwise, advance `curr = curr->next`.
5. Return `head`.

### Complexity Analysis
- **Time Complexity:** $O(N)$ to traverse the list once.
- **Space Complexity:** $O(1)$ auxiliary space.

### C++17 Code
```cpp
class SolutionBetter {
public:
    ListNode* removeElements(ListNode* head, int val) {
        // Step 1: Handle head nodes with target value
        while (head != nullptr && head->val == val) {
            ListNode* temp = head;
            head = head->next;
            delete temp;
        }
        
        if (head == nullptr) {
            return nullptr;
        }
        
        // Step 2: Handle internal nodes
        ListNode* curr = head;
        while (curr->next != nullptr) {
            if (curr->next->val == val) {
                ListNode* temp = curr->next;
                curr->next = curr->next->next;
                delete temp;
            } else {
                curr = curr->next;
            }
        }
        
        return head;
    }
};
```

---

## 7. Optimal Solution (In-place with Dummy Node)

### Algorithm
1. Create a `dummy` node on the stack or heap: `ListNode dummy(0); dummy.next = head;`.
2. Initialize `prev = &dummy` and `curr = head`.
3. Loop while `curr != nullptr`:
   - If `curr->val == val`:
     - Connect the previous node to the next: `prev->next = curr->next`.
     - Save the next node: `ListNode* next_node = curr->next`.
     - Delete current node: `delete curr`.
     - Move current pointer: `curr = next_node`.
   - If `curr->val != val`:
     - Advance `prev = curr`.
     - Advance `curr = curr->next`.
4. Return `dummy.next`.

### Why Optimal?
- Cleans up edge cases (deleting head, deleting all nodes) without nested loops or custom head modifications.
- Performs deletion in a single pass ($O(N)$ time) with zero heap-allocation overhead ($O(1)$ space).

### C++17 Code
```cpp
class Solution {
public:
    ListNode* removeElements(ListNode* head, int val) {
        // Dummy node initialized on stack to avoid memory leak
        ListNode dummy(0);
        dummy.next = head;
        
        ListNode* prev = &dummy;
        ListNode* curr = head;
        
        while (curr != nullptr) {
            if (curr->val == val) {
                // Link prev node to curr's next node
                prev->next = curr->next;
                
                // Store curr for deletion, then advance
                ListNode* to_delete = curr;
                curr = curr->next;
                
                // Deallocate memory
                delete to_delete;
            } else {
                // Move prev and curr forward
                prev = curr;
                curr = curr->next;
            }
        }
        
        return dummy.next;
    }
};
```

---

## 8. Code Walkthrough

1. `ListNode dummy(0); dummy.next = head;`
   - Allocates `dummy` on the stack. Its `next` pointer holds the current head.
2. `ListNode* prev = &dummy;`
   - Points to the dummy node. `prev` will always refer to the node immediately before `curr`.
3. `prev->next = curr->next;`
   - Since `curr->val == val`, we bypass `curr` by pointing `prev->next` to `curr->next`.
4. `curr = curr->next;`
   - Updates our traversal pointer to the next node.
5. `delete to_delete;`
   - Properly frees memory of the deleted node.
6. `prev = curr; curr = curr->next;`
   - If the value does not match `val`, we simply step both pointers forward.

---

## 9. Dry Run

List: `1 -> 2 -> 6 -> 3 -> 6`, `val = 6`.

| Step | `prev` Node | `curr` Node | `curr->val` | Action | Updated List Structure | Next Pointer |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Start** | `dummy(0)` | `1` | `1` | Keep | `dummy -> 1 -> 2 -> 6 -> 3 -> 6` | `prev = 1`, `curr = 2` |
| **1** | `1` | `2` | `2` | Keep | `dummy -> 1 -> 2 -> 6 -> 3 -> 6` | `prev = 2`, `curr = 6` |
| **2** | `2` | `6` | `6` | Delete | `dummy -> 1 -> 2 -> 3 -> 6` | `prev = 2`, `curr = 3` |
| **3** | `2` | `3` | `3` | Keep | `dummy -> 1 -> 2 -> 3 -> 6` | `prev = 3`, `curr = 6` |
| **4** | `3` | `6` | `6` | Delete | `dummy -> 1 -> 2 -> 3 -> nullptr`| `prev = 3`, `curr = nullptr` |

Loop ends because `curr == nullptr`. Return `dummy.next` $\implies$ `1 -> 2 -> 3`.

---

## 10. Complexity Analysis

| Solution | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Vector Copy** | $O(N)$ | $O(N)$ | Extra memory of size $N$ is allocated. |
| **Better (No Dummy)** | $O(N)$ | $O(1)$ | No extra memory, but complex head traversal. |
| **Optimal (Dummy)** | $O(N)$ | $O(1)$ | No extra memory, clean single pass. |

---

## 11. Common Mistakes

1. **Memory Leak:**
   - Detaching a node (`prev->next = curr->next`) but forgetting to call `delete curr` in C++.
2. **Invalid Pointer Access:**
   - Advancing `prev = curr` after deleting `curr`. If you delete `curr`, the memory it pointed to is gone, and using it to update `prev` leads to undefined behavior.
3. **Skipping Consecutive Elements:**
   - Advancing `prev` when a deletion occurs. For example, in `2 -> 2` with `val = 2`, if you delete the first `2` and advance `prev` to it, you will miss deleting the second `2`. Only advance `prev` if no deletion occurs.

---

## 12. Edge Cases

| Edge Case | Input | Expected Output | Why It Matters |
| :--- | :--- | :--- | :--- |
| **Empty list** | `nullptr`, `val = 5` | `nullptr` | Loop does not execute, returns null safely. |
| **All nodes match** | `1 -> 1 -> 1`, `val = 1` | `nullptr` | All nodes are deleted; head becomes null. |
| **Target at head** | `6 -> 1 -> 2`, `val = 6` | `1 -> 2` | Dummy node handles head deletion smoothly. |
| **Target at tail** | `1 -> 2 -> 6`, `val = 6` | `1 -> 2` | Verifies correct termination of list. |
| **No matches** | `1 -> 2 -> 3`, `val = 4` | `1 -> 2 -> 3` | List structure remains unaltered. |

---

## 13. Interview Explanation

"To delete all occurrences of a key in a linked list in $O(N)$ time and $O(1)$ space, we use a dummy node. 

The dummy node is placed right before the head of the list, which allows us to treat the head of the list exactly like any other node, eliminating edge-case conditionals for deleting the first element. We initialize a `prev` pointer pointing to the dummy, and a `curr` pointer pointing to the head. We traverse the list. If `curr` has the target value, we bypass it by setting `prev->next = curr->next` and deallocate `curr`. If it does not match, we simply advance `prev` and `curr` forward. In C++, we make sure to delete each skipped node to prevent memory leaks."

---

## 14. Follow-up Questions

1. **Q:** Can you write a recursive solution for this?
   **A:** Yes:
   ```cpp
   ListNode* removeElements(ListNode* head, int val) {
       if (head == nullptr) return nullptr;
       head->next = removeElements(head->next, val);
       if (head->val == val) {
           ListNode* temp = head->next;
           delete head;
           return temp;
       }
       return head;
   }
   ```
   *Note:* This takes $O(N)$ call-stack space.

---

## 15. Variations

1. **Remove Duplicates from Sorted List:**
   - Remove duplicates so that each element appears only once.
2. **Remove Duplicates from Sorted List II:**
   - Remove *all* nodes that have duplicate numbers, leaving only distinct numbers. Uses a dummy node and a sub-loop to skip duplicates.

---

## 16. Revision Notes

- **Advancing Rule:** Advance `prev` ONLY if you did not delete `curr`. If you deleted `curr`, `prev` stays where it is, and `curr` updates to `prev->next`.
- **Memory Clean-up:** Keep a `to_delete` temporary pointer before moving `curr` to free the node.
- **Dummy Node:** Saves code complexity when the head node is deleted.

---

## 17. Memorization Version

```cpp
// 1. ListNode dummy(0); dummy.next = head;
// 2. ListNode *prev = &dummy, *curr = head;
// 3. Loop: if (curr->val == val) { prev->next = curr->next; delete curr; curr = prev->next; }
// 4. Else: prev = curr; curr = curr->next;
// 5. Return dummy.next;
```
