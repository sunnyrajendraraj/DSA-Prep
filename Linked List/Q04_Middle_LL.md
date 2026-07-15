# Find Middle of Linked List

## 1. Problem Statement
Given the `head` of a singly linked list, find and return the middle node of the list.

- If there are two middle nodes (even length list), return the **second middle node** (Standard LeetCode definition).
- We will also cover the variant where we return the **first middle node**.

### Input
- `head`: A pointer to the first node of a singly linked list.

### Output
- A pointer to the middle node of the linked list.

### Constraints
- The number of nodes in the list is in the range `[1, 100]`.
- `1 <= Node.val <= 100`

### Observations
- In a list of length $N$:
  - If $N$ is odd (e.g., $5$ nodes: `1 -> 2 -> 3 -> 4 -> 5`), there is a single middle node at index $2$ (0-indexed): Node `3`.
  - If $N$ is even (e.g., $4$ nodes: `1 -> 2 -> 3 -> 4`), there are two middle nodes: `2` (index 1) and `3` (index 2).
    - Second Middle: Node `3`.
    - First Middle: Node `2`.

---

## 2. Interview Clarification Questions
1. **How should we handle lists of even length?**
   - *Answer:* By default, return the second middle node (e.g., for `[1, 2]`, return `2`). However, I can also implement the variant that returns the first middle node.
2. **Can the list be empty?**
   - *Answer:* The constraints state the number of nodes is at least 1, but we should handle the empty list case (`nullptr`) robustly by returning `nullptr`.

---

## 3. Thinking Process
- **First Instinct (Two Passes):** First, traverse the list completely to count the total number of nodes, $N$. Then, compute the index of the middle node ($N/2$). Finally, perform a second traversal from the head to this index and return the node.
- **Can we do it in One Pass?** Yes, by using the two-pointer technique.
- **Fast and Slow Pointers (Tortoise and Hare):**
  - If we have a `slow` pointer moving 1 step at a time and a `fast` pointer moving 2 steps at a time.
  - By the time `fast` reaches the end of the list, `slow` will have covered half the distance, landing exactly at the middle node.

---

## 4. Pattern Recognition
- **Pattern:** Fast and Slow Pointers (Tortoise and Hare).
- **Why this topic?** This pattern is standard for finding split points, checking palindromes, or doing merge-sort on linked lists, as it avoids counting the length of the list.

---

## 5. Brute Force Solution (Length Counting)
Count the length, then traverse to the middle index.

### Algorithm
1. Initialize `count = 0` and a pointer `curr = head`.
2. Traverse the list to count the total nodes: `count++`.
3. Calculate the target index: `target = count / 2`.
4. Reset `curr = head`.
5. Loop `target` times: `curr = curr->next`.
6. Return `curr`.

### Dry Run
Input: `1 -> 2 -> 3 -> 4 -> nullptr` (Even length $N = 4$)
1. First pass: count = 4.
2. Target index: `4 / 2 = 2`.
3. Second pass starting at `1`:
   - Step 1: `curr` moves to `2`.
   - Step 2: `curr` moves to `3`.
4. Return Node `3` (Second middle).

### Complexity Analysis
- **Time Complexity:** $O(N)$ because we traverse the list 1.5 times ($N$ steps for count, $N/2$ steps to reach the middle).
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros/Cons
- **Pros:** Conceptually simple and easy to debug.
- **Cons:** Requires two passes. We visit elements multiple times.

### C++17 Brute Force Code
```cpp
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class SolutionBrute {
public:
    ListNode* middleNode(ListNode* head) {
        if (!head || !head->next) return head;
        
        int length = 0;
        ListNode* curr = head;
        while (curr) {
            length++;
            curr = curr->next;
        }
        
        int mid = length / 2;
        curr = head;
        for (int i = 0; i < mid; ++i) {
            curr = curr->next;
        }
        
        return curr;
    }
};
```

---

## 6. Better Solution
> [!NOTE]
> The brute force is already $O(N)$ time and $O(1)$ space. The optimization is to reduce the constant factor (doing it in a single pass). We jump directly to the optimal single-pass solution.

---

## 7. Optimal Solution (Fast and Slow Pointers)

### Case 1: Returning the Second Middle Node (Standard)
- Condition: `while (fast != nullptr && fast->next != nullptr)`
- For odd length `[1, 2, 3, 4, 5]`:
  - `slow` and `fast` start at `1`.
  - `slow` = `2`, `fast` = `3`.
  - `slow` = `3`, `fast` = `5`.
  - Loop terminates because `fast->next` is `nullptr`. `slow` is at `3` (middle).
- For even length `[1, 2, 3, 4]`:
  - `slow` and `fast` start at `1`.
  - `slow` = `2`, `fast` = `3`.
  - `slow` = `3`, `fast` = `nullptr`.
  - Loop terminates because `fast` is `nullptr`. `slow` is at `3` (second middle).

### Case 2: Returning the First Middle Node (Variant)
- Condition: `while (fast->next != nullptr && fast->next->next != nullptr)`
- For odd length `[1, 2, 3, 4, 5]`:
  - `slow` = `3` (same).
- For even length `[1, 2, 3, 4]`:
  - `slow` and `fast` start at `1`.
  - `slow` = `2`, `fast` = `3`.
  - Loop terminates because `fast->next->next` (which is `nullptr`) is checked. `slow` stays at `2` (first middle).

### Optimal C++17 Code
```cpp
class SolutionOptimal {
public:
    // Standard: Returns Second Middle Node for Even Length
    ListNode* middleNode(ListNode* head) {
        if (!head) return nullptr;
        
        ListNode* slow = head;
        ListNode* fast = head;
        
        while (fast != nullptr && fast->next != nullptr) {
            slow = slow->next;       // Moves 1 step
            fast = fast->next->next; // Moves 2 steps
        }
        
        return slow;
    }

    // Variant: Returns First Middle Node for Even Length
    ListNode* middleNodeFirst(ListNode* head) {
        if (!head || !head->next) return head;
        
        ListNode* slow = head;
        ListNode* fast = head;
        
        while (fast->next != nullptr && fast->next->next != nullptr) {
            slow = slow->next;
            fast = fast->next->next;
        }
        
        return slow;
    }
};
```

---

## 8. Code Walkthrough (Standard Case)
- **`ListNode* slow = head; ListNode* fast = head;`**: Both pointers start at the head.
- **`while (fast != nullptr && fast->next != nullptr)`**: 
  - `fast != nullptr` ensures we haven't reached the end (relevant for even-length lists).
  - `fast->next != nullptr` ensures we can safely move `fast` by two steps.
- **`slow = slow->next;`**: Moves the slow pointer one step forward.
- **`fast = fast->next->next;`**: Moves the fast pointer two steps forward.
- **`return slow;`**: Once the loop breaks, `slow` points to the middle node.

---

## 9. Dry Run (Standard Case)

### Dry Run 1: Odd Length `[1, 2, 3, 4, 5]`
- **Init:** `slow = 1`, `fast = 1`
- **Step 1:**
  - `slow = slow->next` -> Node `2`
  - `fast = fast->next->next` -> Node `3`
  - Check: `fast` is `3`, `fast->next` is `4`. Continue.
- **Step 2:**
  - `slow = slow->next` -> Node `3`
  - `fast = fast->next->next` -> Node `5`
  - Check: `fast` is `5`, `fast->next` is `nullptr`. Loop terminates.
- **Result:** Returns `3`.

### Dry Run 2: Even Length `[1, 2, 3, 4]`
- **Init:** `slow = 1`, `fast = 1`
- **Step 1:**
  - `slow = slow->next` -> Node `2`
  - `fast = fast->next->next` -> Node `3`
  - Check: `fast` is `3`, `fast->next` is `4`. Continue.
- **Step 2:**
  - `slow = slow->next` -> Node `3`
  - `fast = fast->next->next` -> `nullptr`
  - Check: `fast` is `nullptr`. Loop terminates.
- **Result:** Returns `3` (Second middle).

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Length Counting (Brute)** | $O(N)$ | $O(1)$ | 1.5 passes over the list. No extra memory. |
| **Fast & Slow (Optimal)** | $O(N)$ | $O(1)$ | Exactly 1 pass. `fast` pointer moves $N$ steps, loop runs $N/2$ times. |

---

## 11. Common Mistakes
- **Null pointer dereference:** Writing `while (fast->next != nullptr && fast != nullptr)` which evaluates `fast->next` before verifying `fast` is not null. Order matters in logical `&&` evaluation.
- **Confusing the two variants:** Implementing the `fast->next != nullptr && fast->next->next != nullptr` loop but expecting the second middle node as output.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it matters |
| :--- | :--- | :--- | :--- |
| **Single node** | `1 -> nullptr` | Node `1` | Loop does not execute. Handled immediately. |
| **Two nodes (Odd/Even)** | `1 -> 2 -> nullptr` | Node `2` (Standard) | Verifies standard returns second node. |
| **Three nodes** | `1 -> 2 -> 3 -> nullptr` | Node `2` | Verifies odd length middle behavior. |

---

## 13. Interview Explanation
> "To find the middle of a linked list in a single pass, I use the fast and slow pointer technique. I initialize both pointers at the head of the list. In a loop, I advance the slow pointer by one node and the fast pointer by two nodes. By moving at twice the speed, when the fast pointer reaches the end of the list, the slow pointer will be exactly at the middle node. If the list has an even number of nodes, using the condition `while (fast != nullptr && fast->next != nullptr)` ensures we return the second of the two middle nodes, which is the standard definition on platforms like LeetCode. This runs in $O(N)$ time and uses $O(1)$ space."

---

## 14. Follow-up Questions
1. **How would you delete the middle node?**
   - *Maintain a `prev` pointer that tracks the node right behind `slow`. When the loop finishes, set `prev->next = slow->next` and delete `slow`.*
2. **How is this used in Merge Sort for Linked Lists?**
   - *In merge sort, we split the list into two halves. We use the 'first middle' variant to locate the split point, set `mid->next = nullptr` to disconnect the halves, and recursively sort both.*

---

## 15. Variations
- **Delete Middle of Linked List:** Finding the middle and removing it.
- **Check Palindrome Linked List:** Uses middle node to find where to start reversing the second half of the list.

---

## 16. Revision Notes
- **Second Middle Guard:** `while (fast && fast->next)`
- **First Middle Guard:** `while (fast->next && fast->next->next)` (requires `head && head->next` check beforehand).

---

## 17. Memorization Version (< 1 minute read)
- **Goal:** Find middle node in 1 pass, $O(1)$ space.
- **Strategy:**
  - `slow = head`, `fast = head`.
  - Loop: `while (fast && fast->next)`.
  - Advance: `slow = slow->next`, `fast = fast->next->next`.
  - Return `slow`.
- **Complexity:** $O(N)$ Time, $O(1)$ Space.
