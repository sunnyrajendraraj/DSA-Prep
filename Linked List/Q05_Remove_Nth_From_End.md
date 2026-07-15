# Remove Nth Node From End of Linked List

## 1. Problem Statement
Given the `head` of a singly linked list and an integer `n`, remove the `n`-th node from the end of the list and return its head.

### Input
- `head`: A pointer to the first node of a singly linked list.
- `n`: An integer representing the position from the end of the list (1-indexed).

### Output
- A pointer to the head of the modified linked list.

### Constraints
- The number of nodes in the list is $L$ where $1 <= L <= 30$.
- `0 <= Node.val <= 100`
- `1 <= n <= L`

### Observations
- The $n$-th node from the end is the $(L - n + 1)$-th node from the beginning (where $L$ is the length of the list).
- To delete a node, we must position our pointer at the node *immediately before* the target node.

---

## 2. Interview Clarification Questions
1. **Is `n` guaranteed to be valid and within the range of the list length?**
   - *Answer:* Yes, you can assume $1 \le n \le L$.
2. **Should we free the memory of the removed node?**
   - *Answer:* Yes, in C++ it is best practice to delete the node to avoid memory leaks.
3. **What if the list has only one node and we remove it?**
   - *Answer:* The list becomes empty, so we should return `nullptr`.

---

## 3. Thinking Process
- **First Instinct (Two Passes):**
  1. Traverse the entire list to compute its length $L$.
  2. The node to be deleted is at index $L - n$.
  3. Traverse the list a second time to index $L - n - 1$ (the node before the target).
  4. Perform the deletion: `prev->next = prev->next->next`.
- **Can we do it in One Pass?**
  - Yes, by maintaining a fixed gap of size $n$ between two pointers, `slow` and `fast`.
- **Two-Pointer Concept:**
  - Advance `fast` by $n$ steps.
  - Now, the distance between `slow` and `fast` is $n$ nodes.
  - Move both pointers 1 step at a time until `fast` reaches the last node (`fast->next == nullptr`).
  - Because `fast` is $n$ nodes ahead of `slow`, when `fast` is at the last node (index $L-1$), `slow` will be at index $L - n - 1$.
  - Index $L - n - 1$ is exactly the node *before* the $n$-th node from the end! We can safely bypass the target node.
- **Handling the Edge Case (Removing the Head):**
  - If we advance `fast` by $n$ steps and `fast` becomes `nullptr`, it means $n$ is equal to the length of the list $L$. Thus, we must remove the head node. We can handle this by returning `head->next`.
  - Alternatively, we can use a **Dummy Node** pointing to `head` to completely unify all cases (including removing the head).

---

## 4. Pattern Recognition
- **Pattern:** Two Pointers (Fixed-Gap sliding window).
- **Why this topic?** This is the classic way to locate an offset from the end of a singly linked list in a single traversal, without needing to know the list's total size in advance.

---

## 5. Brute Force Solution (Two-Passes)
Calculate length, then traverse to the target.

### Algorithm
1. Traverse the list to calculate the total length $L$.
2. If $n == L$, the target is the head. Return `head->next`.
3. Traverse the list to the $(L - n - 1)$-th node. Let this be `curr`.
4. Store the node to delete: `toDelete = curr->next`.
5. Update: `curr->next = curr->next->next`.
6. Delete `toDelete` and return `head`.

### Dry Run
Input: `1 -> 2 -> 3 -> 4 -> 5`, `n = 2`
1. First pass: $L = 5$.
2. Target index to stop before: $L - n - 1 = 5 - 2 - 1 = 2$ (Node `3` at 0-indexed position 2).
3. Second pass: Stop at Node `3`.
4. `toDelete` = Node `4`.
5. Update `3->next = 5`.
6. Delete Node `4`.
7. Output: `1 -> 2 -> 3 -> 5`.

### Complexity Analysis
- **Time Complexity:** $O(N)$ because we traverse the list twice (once for length, once to locate node).
- **Space Complexity:** $O(1)$ auxiliary space.

### C++17 Brute Force Code
```cpp
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class SolutionBrute {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        int length = 0;
        ListNode* curr = head;
        while (curr) {
            length++;
            curr = curr->next;
        }
        
        // Edge Case: Remove head node
        if (length == n) {
            ListNode* newHead = head->next;
            delete head;
            return newHead;
        }
        
        curr = head;
        // Move to the (length - n - 1)-th node
        for (int i = 0; i < length - n - 1; ++i) {
            curr = curr->next;
        }
        
        ListNode* toDelete = curr->next;
        curr->next = curr->next->next;
        delete toDelete;
        
        return head;
    }
};
```

---

## 6. Better Solution
> [!NOTE]
> The brute force is already $O(N)$ time. The optimization is to do it in a single pass of the list. We jump directly to the optimal one-pass two-pointer solution.

---

## 7. Optimal Solution (One Pass, Two Pointers with Dummy Node)
Using a dummy node simplifies the edge cases, particularly when removing the head node.

### Algorithm
1. Create a `dummy` node and set `dummy->next = head`.
2. Initialize two pointers: `slow = dummy` and `fast = dummy`.
3. Move `fast` pointer $n + 1$ steps forward. This creates a gap of exactly $n$ nodes between `slow` and `fast`.
4. Move both `slow` and `fast` pointers 1 step at a time until `fast` becomes `nullptr`.
5. At this point, `slow` is pointing to the node *before* the target node.
6. Store the target node to delete: `toDelete = slow->next`.
7. Bypass the target node: `slow->next = slow->next->next`.
8. Delete `toDelete` and return `dummy->next` (new head).

### Optimal C++17 Code
```cpp
class SolutionOptimal {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        // Create dummy node to handle head removal seamlessly
        ListNode dummy(0);
        dummy.next = head;
        
        ListNode* slow = &dummy;
        ListNode* fast = &dummy;
        
        // Step 1: Move fast pointer n + 1 steps ahead
        for (int i = 0; i <= n; ++i) {
            fast = fast->next;
        }
        
        // Step 2: Move both pointers until fast reaches the end
        while (fast != nullptr) {
            slow = slow->next;
            fast = fast->next;
        }
        
        // Step 3: slow now points to the node BEFORE the target node
        ListNode* toDelete = slow->next;
        slow->next = slow->next->next;
        delete toDelete; // Clean up memory
        
        return dummy.next;
    }
};
```

---

## 8. Code Walkthrough
- **`ListNode dummy(0); dummy.next = head;`**: Stack-allocated dummy node pointing to `head`. Prevents separate logic for head deletion.
- **`ListNode* slow = &dummy; ListNode* fast = &dummy;`**: Initialize both pointers to point to the dummy node.
- **`for (int i = 0; i <= n; ++i) fast = fast->next;`**: Moves `fast` $n+1$ times. Since we start at `dummy`, moving $n+1$ steps sets `fast` to the $(n)$-th node of the actual list.
- **`while (fast != nullptr)`**: Slides both pointers. When `fast` reaches `nullptr` (past the end), `slow` will have advanced $L - n$ steps from `dummy`, positioning it at index $L - n$ of the extended list (which is the node before the target).
- **`slow->next = slow->next->next;`**: Skips the target node.

---

## 9. Dry Run
Input: `head = [1, 2, 3, 4, 5]`, `n = 2`

- List layout with dummy: `dummy(0) -> 1 -> 2 -> 3 -> 4 -> 5 -> nullptr`

### Phase 1: Advance `fast` by $n+1 = 3$ steps
- Init: `slow = dummy`, `fast = dummy`
- Step 1: `fast` moves to `1`
- Step 2: `fast` moves to `2`
- Step 3: `fast` moves to `3`

### Phase 2: Slide both pointers until `fast == nullptr`
- Loop Iteration 1: `slow` -> `1`, `fast` -> `4`
- Loop Iteration 2: `slow` -> `2`, `fast` -> `5`
- Loop Iteration 3: `slow` -> `3`, `fast` -> `nullptr`
- Exit loop.

### Phase 3: Deletion
- `slow` is at Node `3`.
- `toDelete` = `slow->next` -> Node `4`.
- `slow->next = slow->next->next` -> `3->next = 5`.
- Delete Node `4`.
- Returns `dummy.next` -> `1`. Modified list: `1 -> 2 -> 3 -> 5 -> nullptr`.

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Two-Pass (Brute)** | $O(N)$ | $O(1)$ | Traverses the list twice. |
| **One-Pass (Optimal)**| $O(N)$ | $O(1)$ | Single pass. `fast` traverses exactly $N+1$ steps. `slow` traverses $N-n$ steps. |

---

## 11. Common Mistakes
- **Off-by-one error in spacing:** Moving `fast` by only $n$ steps when using a dummy node. If you do this and loop until `fast != nullptr`, `slow` will end up pointing to the target node itself, rather than the node *before* it.
- **Null pointer dereference on head removal:** If not using a dummy node, failing to handle the case where the head itself is removed (i.e. $n == L$).
- **Memory leaks:** Forgetting to free memory using `delete` in C++.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it matters |
| :--- | :--- | :--- | :--- |
| **Remove Head Node** | `[1, 2]`, `n = 2` | `[2]` | Dummy node handles this seamlessly without custom conditional branching. |
| **Single element list**| `[1]`, `n = 1` | `[]` (`nullptr`) | Target is the head node. Returns `nullptr`. |
| **Remove Tail Node** | `[1, 2]`, `n = 1` | `[1]` | Validates correct pointer bypassing at the end of the list. |

---

## 13. Interview Explanation
> "To remove the $N$-th node from the end of a linked list in a single pass, I use the two-pointer technique with a dummy node. The dummy node is placed right before the head, which helps handle edge cases like removing the head node without separate branching logic. I initialize both `slow` and `fast` pointers at the dummy node. First, I advance the `fast` pointer by $N + 1$ steps to establish a fixed gap of $N$ nodes. Then, I advance both pointers one step at a time. When the `fast` pointer reaches the end of the list (`nullptr`), the `slow` pointer will be positioned exactly at the node prior to the target node. I then update the `slow` pointer's next link to skip the target node, delete the target node to free memory, and return `dummy.next` as the new head. This approach achieves $O(N)$ time complexity and $O(1)$ space complexity."

---

## 14. Follow-up Questions
1. **Can you solve this without using a dummy node?**
   - *Yes. We can advance `fast` by $n$ steps from `head`. If `fast` is `nullptr`, it means we need to remove the head, so we return `head->next`. Otherwise, we advance both until `fast->next == nullptr`, and then update `slow->next = slow->next->next`.*
2. **How would you solve this recursively?**
   - *Use a recursive helper function that returns the post-order index from the end. In the recursive unwind, when the index equals $n+1$, we are at the node before the target, and we perform the deletion.*

---

## 15. Variations
- **Remove Nth Node from start:** Simply loop $n-1$ times and delete the next node.
- **Swap Nth node from start with Nth node from end:** Find both nodes using similar two-pointer techniques and swap their values.

---

## 16. Revision Notes
- **Dummy Node Utility:** Prevents special checks for removing the head node.
- **Pointer Spacing:** `fast` is $n+1$ steps ahead of `slow` starting from `dummy`.
- **Termination:** `while (fast != nullptr)`

---

## 17. Memorization Version (< 1 minute read)
- **Goal:** Remove Nth node from end in 1 pass, $O(1)$ space.
- **Strategy:**
  1. Create `dummy` pointing to `head`.
  2. Set `slow = dummy`, `fast = dummy`.
  3. Move `fast` by $n+1$ steps.
  4. Move both 1 step at a time until `fast == nullptr`.
  5. Delete `slow->next` and update `slow->next = slow->next->next`.
  6. Return `dummy.next`.
- **Complexity:** $O(N)$ Time, $O(1)$ Space.
