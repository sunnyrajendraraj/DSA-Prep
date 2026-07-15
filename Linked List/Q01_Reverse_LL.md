# Reverse a Linked List

## 1. Problem Statement
Given the `head` of a singly linked list, reverse the list in-place and return the new head of the reversed list.

### Input
- `head`: A pointer to the first node of a singly linked list.

### Output
- A pointer to the new head of the reversed singly linked list.

### Constraints
- The number of nodes in the list is in the range `[0, 5000]`.
- `-5000 <= Node.val <= 5000`

### Observations
- A singly linked list can only be traversed in the forward direction because each node only has a pointer to its `next` node.
- Reversing the list requires modifying the `next` pointer of each node to point to its predecessor instead of its successor.
- We must be careful not to lose the reference to the rest of the list when we reassign a node's `next` pointer.

---

## 2. Interview Clarification Questions
1. **Can the input list be empty or contain only one node?**
   - *Answer:* Yes, if `head` is `nullptr` or `head->next` is `nullptr`, we should return the head as-is.
2. **Can we modify the values inside the nodes instead of modifying the pointers?**
   - *Answer:* No, in interviews, you are expected to modify the pointers (manipulate the structure) rather than copying or swapping node values.
3. **Is it acceptable to use extra memory, or should it be done in-place?**
   - *Answer:* We should strive for an in-place solution with $O(1)$ auxiliary space.

---

## 3. Thinking Process
- **First Instinct:** If we store all node values in a stack or array during a forward traversal, we can then traverse the list again and overwrite the node values in reverse order. This is a simple two-pass approach but uses $O(N)$ extra space.
- **In-Place Shift:** To reverse the list in-place with $O(1)$ extra space, we need to reverse the pointers. For any node `curr`, its `next` pointer should point to the previous node `prev`.
- **The Challenge:** As soon as we execute `curr->next = prev`, we lose the reference to the original next node. Hence, we must store the next node in a temporary pointer (`nextNode = curr->next`) *before* breaking the link.
- **Pointer Transition:** After reversing the link, we shift our pointers forward:
  - `prev` becomes `curr`
  - `curr` becomes `nextNode`
- **Recursion Concept:** Alternatively, to reverse a list recursively, we can assume that the sub-list from the second node onward is already reversed. Then, we make the second node point back to the first node, and set the first node's next pointer to `nullptr`.

---

## 4. Pattern Recognition
- **Pattern:** Pointer Manipulation / Three-Pointer Slide.
- **Why this topic?** This is the fundamental problem of Linked Lists. It teaches pointer re-assignment, managing temporary references, and recursion on inductive data structures.

---

## 5. Brute Force Solution
The brute force solution involves copying the values of the linked list into an auxiliary container (like a `std::vector` or `std::stack`), reversing the container, and writing the values back to the list.

### Algorithm
1. Traverse the linked list and push all node values onto a stack.
2. Traverse the linked list a second time, pop values from the stack, and update the nodes' values.
3. Return the original head.

### Dry Run
Input: `1 -> 2 -> 3 -> nullptr`
1. First pass: Stack = `[1, 2, 3]` (top is 3).
2. Second pass:
   - Node 1: set val to pop() -> `3`. Stack = `[1, 2]`.
   - Node 2: set val to pop() -> `2`. Stack = `[1]`.
   - Node 3: set val to pop() -> `1`. Stack = `[]`.
3. Output list: `3 -> 2 -> 1 -> nullptr`.

### Complexity Analysis
- **Time Complexity:** $O(N)$ because we traverse the list twice.
- **Space Complexity:** $O(N)$ to store the node values in the stack.

### Pros/Cons
- **Pros:** Extremely simple to implement; does not involve complex pointer redirection.
- **Cons:** High space complexity ($O(N)$); does not actually reverse the physical node connections (violates the constraint of actual list structure reversal).

### C++17 Brute Force Code
```cpp
#include <stack>

struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class SolutionBrute {
public:
    ListNode* reverseList(ListNode* head) {
        if (!head || !head->next) return head;
        
        std::stack<int> values;
        ListNode* curr = head;
        
        // Step 1: Store all values in a stack
        while (curr) {
            values.push(curr->val);
            curr = curr->next;
        }
        
        // Step 2: Write values back in reverse order
        curr = head;
        while (curr) {
            curr->val = values.top();
            values.pop();
            curr = curr->next;
        }
        
        return head;
    }
};
```

---

## 6. Better Solution
> [!NOTE]
> Since the jump from a stack-based value copy to an in-place pointer reversal is direct, there is no intermediate "better" solution that is distinct from the Optimal Iterative approach. We will jump directly to the Optimal solutions (Iterative and Recursive).

---

## 7. Optimal Solution

### Approach 1: Iterative (In-place, Three-pointer)
We maintain three pointers: `prev` (initially `nullptr`), `curr` (initially `head`), and `nextNode` (temporary). We iterate through the list, redirecting pointers one by one.

#### Algorithm
1. Initialize `prev = nullptr` and `curr = head`.
2. While `curr` is not `nullptr`:
   - Store the next node: `nextNode = curr->next`.
   - Reverse the pointer: `curr->next = prev`.
   - Move `prev` forward: `prev = curr`.
   - Move `curr` forward: `curr = nextNode`.
3. Return `prev` (which now points to the new head of the reversed list).

#### Iterative C++17 Code
```cpp
class SolutionIterative {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode* prev = nullptr;
        ListNode* curr = head;
        
        while (curr != nullptr) {
            ListNode* nextNode = curr->next; // 1. Save the next node
            curr->next = prev;              // 2. Reverse the link
            prev = curr;                    // 3. Move prev one step forward
            curr = nextNode;                // 4. Move curr one step forward
        }
        
        return prev; // prev will be pointing to the new head
    }
};
```

### Approach 2: Recursive
Recursively reverse the rest of the list and then fix the pointers at the current level.

#### Algorithm
1. Base cases: If `head` is null or there's only one node, return `head` (it's already reversed).
2. Recursively reverse the list starting from `head->next`. Let the head of this reversed sub-list be `newHead`.
3. Make the next node (`head->next`) point back to the current node: `head->next->next = head`.
4. Disconnect the current node's next pointer: `head->next = nullptr` (preventing cycles).
5. Return `newHead`.

#### Recursive C++17 Code
```cpp
class SolutionRecursive {
public:
    ListNode* reverseList(ListNode* head) {
        // Base case: empty list or single node
        if (head == nullptr || head->next == nullptr) {
            return head;
        }
        
        // Reverse the rest of the list
        ListNode* newHead = reverseList(head->next);
        
        // Link the next node's next back to current head
        head->next->next = head;
        
        // Disconnect the current head's outgoing link to avoid cycles
        head->next = nullptr;
        
        return newHead;
    }
};
```

---

## 8. Code Walkthrough (Iterative)
- **`ListNode* prev = nullptr;`**: We need a pointer to track the reversed portion. Since the head will become the last node, its new `next` must be `nullptr`.
- **`ListNode* curr = head;`**: Pointer to traverse the original list.
- **`while (curr != nullptr)`**: Loop runs until we have processed all nodes.
- **`ListNode* nextNode = curr->next;`**: Save reference to `curr`'s next node so we can traverse to it after modifying `curr->next`.
- **`curr->next = prev;`**: Point the current node back to `prev`.
- **`prev = curr;`** and **`curr = nextNode;`**: Progress the sliding window of pointers.

---

## 9. Dry Run (Iterative)
Input List: `1 -> 2 -> 3 -> nullptr`

| Step | `curr` | `nextNode` (Saved) | `curr->next` (Updated) | `prev` (After update) | `curr` (After update) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Start** | `1` | - | - | `nullptr` | `1` |
| **Iteration 1** | `1` | `2` | `nullptr` | `1` | `2` |
| **Iteration 2** | `2` | `3` | `1` | `2` | `3` |
| **Iteration 3** | `3` | `nullptr` | `2` | `3` | `nullptr` |
| **Exit** | `nullptr` | - | - | `3` | `nullptr` |

**Returned Head:** Node `3` (Reversed list: `3 -> 2 -> 1 -> nullptr`).

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Iterative** | $O(N)$ | $O(1)$ | Single pass over $N$ nodes. Only three pointers are used. |
| **Recursive** | $O(N)$ | $O(N)$ | Visits each node once. The recursion stack uses $O(N)$ frames. |

---

## 11. Common Mistakes
- **Losing the next pointer:** Forgetting to store `curr->next` before setting `curr->next = prev`, which breaks the traversal.
- **Creating a cycle:** Forgetting to set the original head's next to `nullptr`, leading to an infinite cycle between node 1 and node 2.
- **Returning `curr` instead of `prev`:** When the loop terminates, `curr` is `nullptr`. The new head is pointed to by `prev`.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it matters |
| :--- | :--- | :--- | :--- |
| **Empty list** | `nullptr` | `nullptr` | Ensures code doesn't crash on null dereference. |
| **Single node** | `1 -> nullptr` | `1 -> nullptr` | Loop terminates immediately; base checks must handle this. |
| **Two nodes** | `1 -> 2 -> nullptr` | `2 -> 1 -> nullptr` | Verifies pointer swapping updates the head correctly. |

---

## 13. Interview Explanation
> "To reverse a singly linked list in-place, I use an iterative approach with three pointers: `prev`, `curr`, and `nextNode`. We start with `prev` as null and `curr` pointing to the head. In a single pass, we temporarily store `curr->next` in `nextNode` to preserve the rest of the list. We then reverse the connection by pointing `curr->next` to `prev`. Finally, we move `prev` to `curr` and `curr` to `nextNode`. Once `curr` becomes null, `prev` points to the new head of the reversed list. This runs in $O(N)$ time and uses $O(1)$ auxiliary space. Alternatively, we can use recursion, which has the same time complexity but requires $O(N)$ space due to the call stack."

---

## 14. Follow-up Questions
1. **Can you reverse a doubly linked list?**
   - *Yes, we swap the `next` and `prev` pointers for every node.*
2. **How would you reverse a sub-list from position $L$ to $R$?**
   - *Traverse to position $L-1$, keep track of the boundary node, reverse nodes between $L$ and $R$ iteratively, and reconnect the boundaries.*
3. **Can you do it in groups of $K$ nodes?**
   - *Use a helper function to reverse $K$ nodes, and link the groups recursively or iteratively.*

---

## 15. Variations
- **Reverse a Doubly Linked List:** Swap `next` and `prev` pointers for all nodes.
- **Palindrome Linked List:** Find the middle, reverse the second half, and compare values from both ends.

---

## 16. Revision Notes
- **Mnemonic:** **"Save next, point to prev, shift prev, shift curr"** (S-P-S-S).
- **Core Loop:**
  ```cpp
  ListNode* nextNode = curr->next;
  curr->next = prev;
  prev = curr;
  curr = nextNode;
  ```

---

## 17. Memorization Version (< 1 minute read)
- **Goal:** Reverse singly linked list in-place.
- **Iterative Strategy:** Use `prev` (starts as null) and `curr` (starts as head). In a loop:
  1. Save `nextNode = curr->next`.
  2. Set `curr->next = prev`.
  3. Update `prev = curr` and `curr = nextNode`.
- **Termination:** Loop stops when `curr == nullptr`. Return `prev`.
- **Complexity:** $O(N)$ Time, $O(1)$ Space. Keep stack space in mind for recursion ($O(N)$ space).
