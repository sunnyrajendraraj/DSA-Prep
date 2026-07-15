# Segregate Even and Odd Nodes in a Linked List

## 1. Problem Statement

Given the `head` of a singly linked list, rearrange the nodes such that all even-valued nodes appear before all odd-valued nodes, while maintaining the relative order of the even and odd nodes as they appeared in the original list.

### Input
- `head`: A pointer to the head of a singly linked list.

### Output
- A pointer to the head of the rearranged linked list.

### Constraints
- The number of nodes in the list is in the range $[0, 10^5]$.
- $-10^5 \le \text{Node.val} \le 10^5$

### Observations
1. We must segregate based on the **values** of the nodes (i.e. whether `val` is even or odd), not their indices.
2. The relative order of the elements within the even group and within the odd group must remain unchanged. This implies the segregation must be **stable**.
3. Reordering should be done in-place by changing node pointers rather than creating new nodes or copying values.

---

## 2. Interview Clarification

Before coding, clarify:
1. **Q:** Are negative numbers possible in the list? If yes, how do we determine if they are even or odd?
   **A:** Yes, negative numbers are possible. A number is even if `val % 2 == 0`, and odd otherwise.
2. **Q:** Do we need to preserve the original order of even/odd elements?
   **A:** Yes, the segregation must be stable.
3. **Q:** What should we return if the list contains only even numbers or only odd numbers?
   **A:** Return the list as-is.

---

## 3. Thinking Process

Let's build the intuition step-by-step:
1. **Splitting the List:**
   To segregate the nodes stably without allocating new ones, we can dynamically build two separate lists during a single traversal:
   - An **Even List** to hold all even-valued nodes.
   - An **Odd List** to hold all odd-valued nodes.
2. **Dynamic Sublist Creation:**
   - To make list appending simple and avoid handling null checks for heads continually, we use **Dummy Nodes** for both list starts: `even_dummy` and `odd_dummy`.
   - Maintain two pointers, `even_tail` and `odd_tail`, initially pointing to their respective dummy nodes.
3. **Traversing and Appending:**
   - Iterate through the list. If `curr->val` is even, link it to `even_tail->next` and advance `even_tail`.
   - If `curr->val` is odd, link it to `odd_tail->next` and advance `odd_tail`.
4. **Stitching the Lists:**
   - After traversing all nodes, link the end of the even list to the beginning of the odd list: `even_tail->next = odd_dummy.next`.
   - Terminate the odd list to avoid circular loops: `odd_tail->next = nullptr`.
   - The head of the new list is `even_dummy.next`. If there are no even nodes, `even_dummy.next` will be null, so we must return `odd_dummy.next`. Connecting them this way handles all combinations automatically.

---

## 4. Pattern Recognition

This problem uses the **Sublist Partitioning Pattern** (also called the Dummy Pointer Partitioning pattern).
- **Why this pattern?** We are dividing a linked list into two disjoint sub-sequences and then merging them. Dummy nodes prevent null pointer checks when starting new sub-lists.

---

## 5. Brute Force Solution

### Algorithm
1. Traverse the linked list and store even-valued nodes in an array/list, and odd-valued nodes in another array/list.
2. Create a new linked list by iterating through the even array first, then the odd array.
3. Return the new list.

### Complexity Analysis
- **Time Complexity:** $O(N)$ because we traverse the list and the arrays.
- **Space Complexity:** $O(N)$ auxiliary space to store the nodes in the arrays.

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
    ListNode* segregateEvenOdd(ListNode* head) {
        if (head == nullptr || head->next == nullptr) {
            return head;
        }
        
        std::vector<ListNode*> evens;
        std::vector<ListNode*> odds;
        
        ListNode* curr = head;
        while (curr != nullptr) {
            if (curr->val % 2 == 0) {
                evens.push_back(curr);
            } else {
                odds.push_back(curr);
            }
            curr = curr->next;
        }
        
        ListNode dummy(0);
        ListNode* tail = &dummy;
        
        for (auto* node : evens) {
            tail->next = node;
            tail = node;
        }
        for (auto* node : odds) {
            tail->next = node;
            tail = node;
        }
        tail->next = nullptr; // Terminate list
        
        return dummy.next;
    }
};
```

---

## 6. Better Solution

There is no intermediate solution; the brute force jumps directly to the optimal $O(1)$ space solution.

---

## 7. Optimal Solution

### Algorithm
1. Initialize two dummy nodes: `even_dummy` and `odd_dummy`.
2. Keep two pointers: `even_tail` pointing to `even_dummy`, and `odd_tail` pointing to `odd_dummy`.
3. Traverse the list using a `curr` pointer:
   - If `curr->val % 2 == 0`, append to the even list: `even_tail->next = curr`, then `even_tail = even_tail->next`.
   - Else, append to the odd list: `odd_tail->next = curr`, then `odd_tail = odd_tail->next`.
   - Move `curr = curr->next`.
4. Connect the two lists: `even_tail->next = odd_dummy.next`.
5. Terminate the odd list tail: `odd_tail->next = nullptr`.
6. Return `even_dummy.next` if it is not null; otherwise, return `odd_dummy.next`.

### C++17 Code
```cpp
class Solution {
public:
    ListNode* segregateEvenOdd(ListNode* head) {
        if (head == nullptr || head->next == nullptr) {
            return head;
        }
        
        ListNode even_dummy(0);
        ListNode odd_dummy(0);
        
        ListNode* even_tail = &even_dummy;
        ListNode* odd_tail = &odd_dummy;
        
        ListNode* curr = head;
        while (curr != nullptr) {
            // Check for even numbers (works correctly for negative integers too)
            if (curr->val % 2 == 0) {
                even_tail->next = curr;
                even_tail = even_tail->next;
            } else {
                odd_tail->next = curr;
                odd_tail = odd_tail->next;
            }
            curr = curr->next;
        }
        
        // Stitch the lists together
        even_tail->next = odd_dummy.next;
        odd_tail->next = nullptr; // Crucial: break any cycles
        
        return even_dummy.next;
    }
};
```

---

## 8. Code Walkthrough

1. `ListNode even_dummy(0); ListNode odd_dummy(0);`
   - Allocated on the stack. They act as anchors for the start of the even and odd sublists.
2. `if (curr->val % 2 == 0)`
   - Checks if the value is even. In C++, modulo operations on negative numbers return negative remainders (e.g. `-3 % 2 = -1`), but `-4 % 2 = 0`. Hence, checking `val % 2 == 0` is completely safe for both positive and negative numbers.
3. `even_tail->next = odd_dummy.next;`
   - Links the last even node to the first odd node. If no even nodes exist, `even_tail` remains pointing to `even_dummy`, meaning `even_dummy.next` is set to `odd_dummy.next`, which is correct.
4. `odd_tail->next = nullptr;`
   - Prevents cycles. If the last node in the original list was an even node, the last odd node (`odd_tail`) would still point to its original next node (which is now part of the even list), creating a cycle. Clearing this next pointer is critical.

---

## 9. Dry Run

List: `17 -> 15 -> 8 -> 12 -> 5`

| Step | Node Checked (`curr`) | Condition | Even List State | Odd List State | Next Node |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Start** | - | - | `even_dummy(0)` | `odd_dummy(0)` | `17` |
| **1** | `17` | Odd | `even_dummy` | `odd_dummy -> 17` | `15` |
| **2** | `15` | Odd | `even_dummy` | `odd_dummy -> 17 -> 15` | `8` |
| **3** | `8` | Even | `even_dummy -> 8` | `odd_dummy -> 17 -> 15` | `12` |
| **4** | `12` | Even | `even_dummy -> 8 -> 12` | `odd_dummy -> 17 -> 15` | `5` |
| **5** | `5` | Odd | `even_dummy -> 8 -> 12` | `odd_dummy -> 17 -> 15 -> 5` | `nullptr` |

- **Stitching:** `even_tail->next = odd_dummy.next` $\implies$ `12->next = 17`.
- **Termination:** `odd_tail->next = nullptr` $\implies$ `5->next = nullptr`.
- **Result:** `8 -> 12 -> 17 -> 15 -> 5`.

---

## 10. Complexity Analysis

| Metric | Complexity | Reasoning |
| :--- | :--- | :--- |
| **Time Complexity** | $O(N)$ | We traverse the list exactly once. |
| **Space Complexity** | $O(1)$ | Stack-allocated dummy nodes, no dynamic heap allocations. |

---

## 11. Common Mistakes

1. **Cycle in the List:**
   - Forgetting to set `odd_tail->next = nullptr`. If the last node of the list was even, the odd tail still points to that even node, creating a cycle.
2. **Negative Modulo Error:**
   - Writing `if (curr->val % 2 == 1)` to find odd numbers. In C++, `-3 % 2` evaluates to `-1`. Therefore, checking for odd nodes using `== 1` will fail for negative numbers. Always check evenness with `val % 2 == 0` and use `else` for odds.
3. **Losing the Head Pointer:**
   - Navigating using the `head` pointer directly without a `curr` pointer, losing references to the start of the list.

---

## 12. Edge Cases

| Edge Case | Input | Expected Output | Why It Matters |
| :--- | :--- | :--- | :--- |
| **Empty list** | `nullptr` | `nullptr` | Prevent null pointer crash. |
| **All Even values** | `2 -> 4 -> 6` | `2 -> 4 -> 6` | Test behavior when one sublist is empty. |
| **All Odd values** | `1 -> 3 -> 5` | `1 -> 3 -> 5` | Test behavior when the other sublist is empty. |
| **Negative numbers** | `-2 -> -3 -> 4` | `-2 -> 4 -> -3` | Checks correct modulo identification. |

---

## 13. Interview Explanation

"To segregate even and odd nodes in a linked list while keeping their original relative order, we can use two dummy nodes: one for the even-valued nodes and one for the odd-valued nodes. 

We traverse the list using a single pointer. For every node, we check if its value is even using `val % 2 == 0`. If even, we append it to the even sublist; if odd, we append it to the odd sublist. After reaching the end of the list, we connect the tail of the even sublist to the head of the odd sublist, and set the tail of the odd sublist to `nullptr` to prevent any cycles. This process sorts the elements stably in $O(N)$ time and $O(1)$ space."

---

## 14. Follow-up Questions

1. **Q:** What if we want to segregate based on the *positions* (index 1 is odd, index 2 is even, etc.) instead of the values?
   **A:** The logic is very similar, but instead of checking `curr->val % 2`, we maintain a counter or toggle a boolean variable on each step to track whether the current node occupies an even or odd position.
2. **Q:** Can we solve this problem without using dummy nodes?
   **A:** Yes, but it requires writing several conditionals to initialize the heads of the even and odd lists, making the code more error-prone.

---

## 15. Variations

1. **Odd Even Linked List (LeetCode 328):**
   - Rearrange nodes based on their indices (1-indexed). The first node is odd, second is even, third is odd, etc.
   - *Solution:* Keep `odd` and `even` pointers, link `odd->next = even->next`, step them alternately.

---

## 16. Revision Notes

- **Modulo Check:** Always use `val % 2 == 0` (even check) rather than `val % 2 == 1` (odd check) to support negative values.
- **Stitching Step:** `even_tail->next = odd_dummy.next; odd_tail->next = nullptr;`
- **Memory safety:** Use stack-allocated local dummy objects `ListNode dummy(0)` rather than heap allocations to prevent leaks.

---

## 17. Memorization Version

```cpp
// 1. ListNode even_dummy(0), odd_dummy(0);
// 2. Loop: if (curr->val % 2 == 0) append to even_tail, else append to odd_tail.
// 3. Connect: even_tail->next = odd_dummy.next.
// 4. Terminate: odd_tail->next = nullptr.
// 5. Return even_dummy.next.
```
