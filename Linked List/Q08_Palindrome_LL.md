# Check Palindrome Linked List

## 1. Problem Statement

Given the head of a singly linked list, determine if it is a palindrome. A linked list is a palindrome if it reads the same forward and backward.

### Input
- `head`: A head pointer to a singly linked list.

### Output
- A boolean: `true` if the list is a palindrome, otherwise `false`.

### Constraints
- The number of nodes in the list is in the range $[1, 10^5]$.
- $0 \le \text{Node.val} \le 9$.

### Observations
1. Palindromes read identical from left-to-right and right-to-left.
2. In a singly linked list, we can only traverse forward easily. We cannot traverse backward from the tail node because there are no backward pointers.
3. If the list is empty or has a single node, it is trivially a palindrome.

---

## 2. Interview Clarification

Before writing code, ask the interviewer:
1. **Can we modify the input linked list?**
   * *Answer*: Yes, but you should restore it to its original structure before returning the result. This is a best practice for API design.
2. **What are the time and space complexity expectations?**
   * *Answer*: Ideally, you should solve this in $O(N)$ time and $O(1)$ auxiliary space.
3. **What values can the nodes have?**
   * *Answer*: Typically characters or single digits, but your solution should work for any integer values.

---

## 3. Thinking Process

Let's think through how to solve this:
* **First Instinct (Brute Force)**:
  * Since we can't traverse backward in a singly linked list, why not copy all the elements of the list into a dynamic array (like `std::vector` in C++)?
  * Once the values are in an array, we can use a two-pointer approach (one pointer at the start, one at the end) and check if they meet in the middle while pointing to identical elements.
  * This is very straightforward. The time complexity is $O(N)$ because we traverse the list once and then traverse the array. The space complexity is $O(N)$ to store the array elements.
* **Can we improve the space complexity? (Better/Alternative Approach)**:
  * What if we use recursion?
  * We can use a recursive function that goes all the way to the end of the list. As the recursion unwinds (backtracks), we compare the nodes from the end with the nodes from the beginning.
  * To do this, we keep a pointer to the left node (starting at `head`) as a member/ref variable, and let recursion traverse to the right node.
  * While this avoids explicitly allocating an array, the recursion stack takes $O(N)$ space. So it's still $O(N)$ space in the worst case.
* **Finding the Optimal Approach (O(1) Space)**:
  * To do it in $O(1)$ space, we cannot use arrays or recursion stacks. We must modify the pointers of the linked list itself.
  * A palindrome is symmetric about the middle.
  * If we can:
    1. Find the middle of the linked list.
    2. Reverse the second half of the linked list.
    3. Compare the first half and the reversed second half node-by-node.
    4. Restore the list to its original state (by reversing the second half back).
    5. Return the comparison result.
  * Let's trace this:
    * For list `1 -> 2 -> 3 -> 2 -> 1`:
      * Middle is `3`.
      * Second half starting after middle is `2 -> 1`.
      * Reversed second half: `1 -> 2`.
      * Compare `1 -> 2 -> ...` (first half) with `1 -> 2` (reversed second half). They match!
      * Reverse the second half back to `2 -> 1` and reconnect.
      * Return `true`.
    * For list `1 -> 2 -> 2 -> 1`:
      * Middle is the first `2`.
      * Second half starts at the second `2`.
      * Reversed second half: `1 -> 2`.
      * Compare `1 -> 2 -> ...` with `1 -> 2`. They match!
      * Return `true`.
  * Finding middle is $O(N)$ using slow & fast pointers. Reversing is $O(N)$ using three pointers. Comparing is $O(N)$. Restoring is $O(N)$.
  * Total time: $O(N)$. Total auxiliary space: $O(1)$. This is optimal!

---

## 4. Pattern Recognition

1. **Slow and Fast Pointers (Tortoise and Hare)**: Used to find the middle of the linked list in a single pass.
2. **In-place Reversal of a Linked List**: Reversing a sub-segment of a linked list is a fundamental building block.
3. **Restore Before Return**: Modifying input structures and restoring them back before exiting is a common production-ready software pattern.

---

## 5. Brute Force Solution (Copy to Array)

### Algorithm
1. Traverse the linked list and push every node's value into a vector.
2. Initialize two pointers: `left = 0`, `right = vector.size() - 1`.
3. While `left < right`:
   * If `vector[left] != vector[right]`, return `false`.
   * Increment `left`, decrement `right`.
4. Return `true`.

### Dry Run
Let list be `1 -> 2 -> 3 -> 2 -> 1`.
* Traverse: Vector `v = [1, 2, 3, 2, 1]`.
* Two pointers: `left = 0`, `right = 4`.
  * `v[0] == v[4]` (1 == 1) $\rightarrow$ `left = 1`, `right = 3`.
  * `v[1] == v[3]` (2 == 2) $\rightarrow$ `left = 2`, `right = 2`.
* Loop terminates because `left < right` is false.
* Return `true`.

### Complexity Analysis
* **Time Complexity**: $O(N)$ where $N$ is the number of nodes. We traverse the list once and the array once.
* **Space Complexity**: $O(N)$ to store the node values in the vector.

### Pros and Cons
* **Pros**: Simple, highly readable, robust. No mutation of the original list.
* **Cons**: Uses linear $O(N)$ extra memory, which is inefficient for large datasets.

### Commented C++17 Code
```cpp
#include <vector>

// Definition for singly-linked list.
struct ListNode {
    int val;
    ListNode *next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode *next) : val(x), next(next) {}
};

class BrutePalindromeLL {
public:
    bool isPalindrome(ListNode* head) {
        std::vector<int> values;
        ListNode* current = head;
        
        // Copy list values to array
        while (current != nullptr) {
            values.push_back(current->val);
            current = current->next;
        }

        // Two pointer comparison
        int left = 0;
        int right = values.size() - 1;
        while (left < right) {
            if (values[left] != values[right]) {
                return false;
            }
            left++;
            right--;
        }
        return true;
    }
};
```

---

## 6. Better Solution (Recursive Stack Comparison)

### Improvement over Brute Force
Instead of allocating an external array explicitly, we use the call stack of a recursive function to traverse back. We compare nodes by maintaining a reference to the front of the list.

### Algorithm
1. Define a global or helper class member variable `left` initialized to `head`.
2. Implement a recursive helper function `check(ListNode* right)`:
   * Base case: if `right` is `nullptr`, return `true`.
   * Recursively call `check(right->next)`. Let this return a boolean `isSubPalindrome`.
   * If `isSubPalindrome` is `false`, return `false`.
   * Compare `left->val` with `right->val`.
   * Move `left` to `left->next`.
   * Return the result of the comparison.

### Dry Run
Let list be `1 -> 2 -> 3`. `left` points to `1`.
* `check(1)` calls `check(2)`.
* `check(2)` calls `check(3)`.
* `check(3)` calls `check(nullptr)` $\rightarrow$ returns `true`.
* Back in `check(3)`:
  * Compare `left->val` (1) with `right->val` (3) $\rightarrow$ mismatch! Return `false`.
* `check(2)` receives `false`, returns `false`.
* `check(1)` receives `false`, returns `false`.
* Final result is `false`.

### Complexity Analysis
* **Time Complexity**: $O(N)$ as we visit each node once.
* **Space Complexity**: $O(N)$ due to the implicit recursion stack depth.

### Commented C++17 Code
```cpp
class BetterPalindromeLL {
private:
    ListNode* leftPointer;

    bool check(ListNode* rightPointer) {
        if (rightPointer == nullptr) {
            return true;
        }

        // Traverse to the end of the list recursively
        bool isSubPalindrome = check(rightPointer->next);
        if (!isSubPalindrome) {
            return false;
        }

        // Compare values
        bool valuesMatch = (leftPointer->val == rightPointer->val);
        
        // Move left pointer forward
        leftPointer = leftPointer->next;

        return valuesMatch;
    }

public:
    bool isPalindrome(ListNode* head) {
        leftPointer = head;
        return check(head);
    }
};
```

---

## 7. Optimal Solution (In-place Reversal of Second Half)

### Best Approach
This method halves the space requirement to $O(1)$ by finding the midpoint of the linked list, reversing the second half in-place, performing a comparison, and then reversing it back to restore the original list structure.

### Algorithm
1. **Find Middle**: Initialize `slow` and `fast` pointers at the `head`. Move `slow` by 1 step and `fast` by 2 steps. When `fast->next` or `fast->next->next` is `nullptr`, `slow` points to the end of the first half.
2. **Reverse Second Half**: Reverse the list starting from `slow->next`. Let the head of this reversed second half be `secondHalfHead`.
3. **Compare**: Compare nodes from `head` and `secondHalfHead` one by one. If any values mismatch, set a flag `isPalin = false`.
4. **Restore**: Reverse the second half again to restore it, reconnecting it back to `slow->next`.
5. **Return**: Return `isPalin`.

### Why it is Optimal
It runs in $O(N)$ time with no extra memory allocation, fulfilling the $O(1)$ auxiliary space constraint. No recursive stack frames or temporary arrays are created.

### C++17 Code
```cpp
class OptimalPalindromeLL {
private:
    // Helper function to reverse a linked list in-place
    ListNode* reverseList(ListNode* head) {
        ListNode* prev = nullptr;
        ListNode* curr = head;
        while (curr != nullptr) {
            ListNode* nextNode = curr->next;
            curr->next = prev;
            prev = curr;
            curr = nextNode;
        }
        return prev;
    }

public:
    bool isPalindrome(ListNode* head) {
        if (head == nullptr || head->next == nullptr) {
            return true;
        }

        // 1. Find the end of the first half (slow pointer)
        ListNode* slow = head;
        ListNode* fast = head;
        while (fast->next != nullptr && fast->next->next != nullptr) {
            slow = slow->next;
            fast = fast->next->next;
        }

        // 2. Reverse the second half
        ListNode* secondHalfHead = reverseList(slow->next);

        // 3. Compare values
        ListNode* p1 = head;
        ListNode* p2 = secondHalfHead;
        bool isPalin = true;
        while (p2 != nullptr) {
            if (p1->val != p2->val) {
                isPalin = false;
                break;
            }
            p1 = p1->next;
            p2 = p2->next;
        }

        // 4. Restore the original list structure
        slow->next = reverseList(secondHalfHead);

        return isPalin;
    }
};
```

---

## 8. Code Walkthrough

1. **`reverseList(ListNode* head)`**:
   * Uses three pointers: `prev` (starts as `nullptr`), `curr` (starts as `head`), and `nextNode` (stores the next pointer before cutting it).
   * Iterates through the list, setting `curr->next = prev`, shifting `prev` to `curr`, and `curr` to `nextNode`.
   * Returns `prev` which becomes the new head of the reversed list.
2. **`isPalindrome(ListNode* head)`**:
   * **Fast/Slow traversal**: `fast` moves twice as fast as `slow`. For odd length, e.g., `1 -> 2 -> 3 -> 2 -> 1`, `slow` stops at `3`. For even length, e.g., `1 -> 2 -> 2 -> 1`, `slow` stops at the first `2`.
   * **Reversing second half**: The list starting at `slow->next` is reversed. For `1 -> 2 -> 2 -> 1`, we reverse `2 -> 1` into `1 -> 2`.
   * **Comparison**: `p1` traverses from the main head, and `p2` traverses from the reversed second half head. They compare up to the end of the second half (`p2 != nullptr`).
   * **Restoration**: To leave the list exactly as we found it, we reverse `secondHalfHead` again and attach it to `slow->next`.

---

## 9. Dry Run

Let's dry run the optimal solution on the list: `1 -> 2 -> 3 -> 2 -> 1`.

* **Step 1: Finding Middle**
  * Initial: `slow = 1`, `fast = 1`
  * Iteration 1: `slow = 2`, `fast = 3` (moves to `fast->next->next`)
  * Iteration 2: `fast->next` is `2`, `fast->next->next` is `1` (both not null). `slow = 3`, `fast = 1` (last node).
  * Loop condition fails: `fast->next` is null.
  * First half ends at `slow = 3`.

* **Step 2: Reversing second half**
  * `slow->next` points to `2` (the node after `3`).
  * Reversing `2 -> 1` results in `1 -> 2`.
  * `secondHalfHead = 1` (value is 1, next is 2).

* **Step 3: Comparing**
  * `p1 = 1` (start of list), `p2 = 1` (start of reversed second half).
  * Match! `p1 = 2`, `p2 = 2`.
  * Match! `p1 = 3`, `p2 = nullptr`.
  * Loop ends. `isPalin` is `true`.

* **Step 4: Restoring**
  * `reverseList(secondHalfHead)` reverses `1 -> 2` back to `2 -> 1`.
  * `slow->next = 2`.
  * List is restored to `1 -> 2 -> 3 -> 2 -> 1`.

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Description |
|:---|:---:|:---:|:---|
| **Brute Force (Array)** | $O(N)$ | $O(N)$ | Simple but uses linear memory. |
| **Better (Recursive Stack)** | $O(N)$ | $O(N)$ | Avoids array but uses stack space. |
| **Optimal (In-place Reversal)** | $O(N)$ | $O(1)$ | Constant memory, restores original list. |

* **Time Complexity**: $O(N)$ since finding the middle takes $N/2$ steps, reversing takes $N/2$ steps, comparison takes $N/2$ steps, and restoration takes $N/2$ steps. The sum of these parts is $2N = O(N)$.
* **Space Complexity**: $O(1)$ because we only use a constant number of pointers (`slow`, `fast`, `prev`, `curr`, `nextNode`, `p1`, `p2`) and no extra memory.

---

## 11. Common Mistakes

1. **Not Restoring the List**: In interviews, developers often modify the list and return `true`/`false` directly. Interviewers consider it bad practice to modify input lists permanently unless instructed.
2. **Off-by-One in Middle Finding**: Stopping too early or too late. Ensure the condition is `fast->next != nullptr && fast->next->next != nullptr`.
3. **Pointer Loss**: While reversing the second half, if not careful, you might lose reference to the nodes, causing segmentation faults.
4. **Incorrect Loop Condition for Comparison**: Comparing until `p1 != nullptr` instead of `p2 != nullptr`. Since the first half might contain the middle node (for odd lengths) and be longer, traversing until `p2 != nullptr` ensures we only compare the reversed half length.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it Matters |
|:---|:---|:---|:---|
| **Single Node** | `1` | `true` | A single node is always a palindrome. |
| **Two Nodes (Different)** | `1 -> 2` | `false` | Tests simple failure cases. |
| **Two Nodes (Same)** | `1 -> 1` | `true` | Tests simple success cases. |
| **Even Length Palindrome** | `1 -> 2 -> 2 -> 1` | `true` | Verifies even length parity splitting. |
| **Odd Length Palindrome** | `1 -> 2 -> 3 -> 2 -> 1` | `true` | Verifies odd length parity splitting. |

---

## 13. Interview Explanation

"To check if a singly linked list is a palindrome, the simplest approach is to copy all node values into an array and use two pointers to verify if the array is symmetric. This takes $O(N)$ time and $O(N)$ space.

To optimize the space complexity to $O(1)$, we can implement an in-place approach:
1. First, we find the middle of the linked list using slow and fast pointers.
2. Next, we reverse the second half of the list in-place.
3. Then, we compare the first half and the reversed second half node-by-node. If all values match, it is a palindrome.
4. Finally, to keep the data structure unchanged for callers, we reverse the second half again to restore the original list structure and return the result.
This optimal strategy runs in $O(N)$ time and uses $O(1)$ extra space."

---

## 14. Follow-up Questions

1. **How would you solve this if you could not write to or mutate the list at all?**
   * *Answer*: If we are strictly forbidden from writing to the nodes, we are forced to use $O(N)$ space (using either recursion stack or copying values to an array/stack).
2. **If this list were a Doubly Linked List, how would that change the solution?**
   * *Answer*: For a doubly linked list, we can keep one pointer at the head and traverse forward, and another pointer at the tail (found in one $O(N)$ traversal) and traverse backward. We compare their values until they meet, using $O(1)$ space and $O(N)$ time without modifying the list structure.

---

## 15. Variations

1. **Reorganize/Reorder List**:
   * *Relation*: Requires finding the middle of the list and reversing the second half, then interweaving them (`1 -> 2 -> 3 -> 4` becomes `1 -> 4 -> 2 -> 3`).
2. **Merge Sort on Linked List**:
   * *Relation*: Uses the slow/fast pointer technique to divide the list into two halves before sorting and merging them.

---

## 16. Revision Notes

* **Core Mnemonic**: "Find Mid $\rightarrow$ Reverse Right Half $\rightarrow$ Compare $\rightarrow$ Restore".
* **Middle Pointer Trick**:
  ```cpp
  while (fast->next && fast->next->next) {
      slow = slow->next;
      fast = fast->next->next;
  }
  ```
  This leaves `slow` pointing to the exact boundary node of the first half.
* **Complexity summary**: Time: $O(N)$ | Space: $O(1)$ (Optimal) vs $O(N)$ (Brute).

---

## 17. Memorization Version

```cpp
// 1-min Review: Palindrome Linked List O(1) Space
bool isPalindrome(ListNode* head) {
    if (!head || !head->next) return true;
    ListNode *slow = head, *fast = head;
    while (fast->next && fast->next->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    ListNode* secondHead = reverse(slow->next);
    ListNode *p1 = head, *p2 = secondHead;
    bool isPalin = true;
    while (p2) {
        if (p1->val != p2->val) { isPalin = false; break; }
        p1 = p1->next; p2 = p2->next;
    }
    slow->next = reverse(secondHead); // restore
    return isPalin;
}
```
