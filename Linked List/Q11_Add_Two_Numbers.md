# Add Two Numbers as Linked Lists

## 1. Problem Statement

You are given two non-empty linked lists representing two non-negative integers. The digits are stored in **reverse order**, and each of their nodes contains a single digit. Add the two numbers and return the sum as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

### Input
- `l1`: Head pointer to the first singly linked list representing number 1.
- `l2`: Head pointer to the second singly linked list representing number 2.

### Output
- Head pointer to a new singly linked list representing the sum of the two numbers.

### Constraints
- The number of nodes in each linked list is in the range $[1, 100]$.
- $0 \le \text{Node.val} \le 9$.

### Observations
1. The digits are stored in reverse order. This is a major advantage! Why? Because standard addition starts from the least significant digit (ones place) and moves to the most significant digit. Since the heads of the lists represent the ones place, we can traverse both lists from left to right and perform addition naturally.
2. We must handle the carry digit that propagates from right to left.
3. The lists can be of different lengths.
4. There might be a carry left over after processing all nodes from both lists (e.g., $99 + 1 = 100$). We must create a new node for this final carry.

---

## 2. Interview Clarification

Before starting, ask the interviewer:
1. **Are we allowed to modify the input lists?**
   * *Answer*: No, do not modify the input lists. Create a new list for the result.
2. **How should we handle lists of different sizes?**
   * *Answer*: Treat the missing digits of the shorter list as 0.
3. **Can the numbers contain leading zeros?**
   * *Answer*: No, except for the number 0 itself (e.g., a single node containing `0`).
4. **What are the time and space complexity constraints?**
   * *Answer*: $O(\max(M, N))$ time, where $M$ and $N$ are the sizes of the two lists. The auxiliary space should be $O(1)$ (excluding the space for the output list).

---

## 3. Thinking Process

Let's think through how to solve this:
* **First Instinct (Brute Force)**:
  * Convert the first linked list to a standard integer (e.g., `1 -> 2 -> 3` represents `321`).
  * Convert the second linked list to a standard integer (e.g., `4 -> 5` represents `54`).
  * Add the two integers: `321 + 54 = 375`.
  * Convert `375` back into a linked list: `5 -> 7 -> 3`.
  * *Critique*: While this works for small numbers, it will fail due to integer overflow for lists with more than 18 digits (since even `unsigned long long` in C++ has a limit of $1.8 \times 10^{19}$). Linked lists can represent numbers with hundreds of digits.
* **Better Solution (String/Vector Addition)**:
  * To prevent overflow, we can traverse both lists, copy their values into strings or vectors, and perform school-method big-integer addition on the strings.
  * This avoids overflow, but it requires $O(M + N)$ extra space to store the strings/digits.
* **Optimal Solution (Simultaneous Direct Addition)**:
  * Since we traverse the lists starting from the units digit, we don't need to copy values. We can add them on the fly.
  * We maintain a pointer `p1` for `l1` and `p2` for `l2`.
  * We also maintain a `carry` variable initialized to 0.
  * In a loop, as long as `p1 != nullptr` or `p2 != nullptr` or `carry != 0`:
    * Calculate sum: `sum = (p1 ? p1->val : 0) + (p2 ? p2->val : 0) + carry`.
    * Update carry: `carry = sum / 10`.
    * Create a new node with value `sum % 10` and append it to our result list.
    * Move `p1` and `p2` forward if they are not null.
  * This is highly efficient and operates in a single pass.

---

## 4. Pattern Recognition

1. **Simultaneous Traversal**: Iterating through two pointer sequences in lockstep.
2. **Carry Propagation**: A standard pattern in arithmetic operations where extra value from one position is carried over to the next.
3. **Dummy Head Pattern**: Using a dummy node to easily construct a linked list without checking if the head is null in every step.

---

## 5. Brute Force Solution (Integer Conversion)

### Algorithm
1. Traverse `l1`, compute the integer `num1` (reversing digits back).
2. Traverse `l2`, compute the integer `num2`.
3. Let `sum = num1 + num2`.
4. If `sum == 0`, return a node with value `0`.
5. Reconstruct the list by taking `sum % 10` as node values and setting `sum = sum / 10`.

### Complexity Analysis
* **Time Complexity**: $O(M + N)$ where $M, N$ are the lengths of the lists.
* **Space Complexity**: $O(1)$ auxiliary space.
* **Pros**: Simple arithmetic.
* **Cons**: Fails completely on overflow when list length $> 19$.

### Commented C++17 Code
```cpp
// Definition for singly-linked list.
struct ListNode {
    int val;
    ListNode *next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode *next) : val(x), next(next) {}
};

class BruteAddTwoNumbers {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        long long num1 = 0, num2 = 0;
        long long factor = 1;
        
        // Convert l1 to integer
        while (l1 != nullptr) {
            num1 += l1->val * factor;
            factor *= 10;
            l1 = l1->next;
        }
        
        // Convert l2 to integer
        factor = 1;
        while (l2 != nullptr) {
            num2 += l2->val * factor;
            factor *= 10;
            l2 = l2->next;
        }
        
        long long sum = num1 + num2;
        if (sum == 0) return new ListNode(0);
        
        ListNode dummy(0);
        ListNode* curr = &dummy;
        
        // Build the linked list
        while (sum > 0) {
            curr->next = new ListNode(sum % 10);
            curr = curr->next;
            sum /= 10;
        }
        
        return dummy.next;
    }
};
```

---

## 6. Better Solution (Vector/String Arithmetic)

### Improvement over Brute Force
Handles arbitrarily large integers by using string-based addition, avoiding hardware integer overflow.

### Algorithm
1. Read `l1` values into a string `s1` (e.g. "123").
2. Read `l2` values into a string `s2` (e.g. "45").
3. Perform standard string-addition starting from index 0.
4. Construct a new linked list from the resulting sum string.

### Complexity Analysis
* **Time Complexity**: $O(M + N)$.
* **Space Complexity**: $O(M + N)$ to store the strings/digits.

### Commented C++17 Code
```cpp
#include <string>
#include <algorithm>

class BetterAddTwoNumbers {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        std::string s1 = "", s2 = "";
        while (l1) { s1 += std::to_string(l1->val); l1 = l1->next; }
        while (l2) { s2 += std::to_string(l2->val); l2 = l2->next; }
        
        ListNode dummy(0);
        ListNode* curr = &dummy;
        int i = 0, j = 0;
        int carry = 0;
        
        while (i < s1.length() || j < s2.length() || carry) {
            int val1 = (i < s1.length()) ? (s1[i] - '0') : 0;
            int val2 = (j < s2.length()) ? (s2[j] - '0') : 0;
            
            int sum = val1 + val2 + carry;
            carry = sum / 10;
            curr->next = new ListNode(sum % 10);
            curr = curr->next;
            
            if (i < s1.length()) i++;
            if (j < s2.length()) j++;
        }
        
        return dummy.next;
    }
};
```

---

## 7. Optimal Solution (Direct Simultaneous Addition)

### Best Approach
Instead of transferring values to intermediate strings or vectors, perform the addition in a single pass directly on the linked list nodes. We sum the matching digits and propagate the carry variable forward.

### Algorithm
1. Initialize a `dummy` node to simplify the creation of the output list.
2. Initialize `curr` to point to `dummy`.
3. Set `carry` to `0`.
4. Run a loop while `l1 != nullptr` or `l2 != nullptr` or `carry != 0`:
   * Determine digit values: `x = (l1 != nullptr) ? l1->val : 0;` and `y = (l2 != nullptr) ? l2->val : 0;`.
   * Compute sum: `sum = x + y + carry;`.
   * Update carry: `carry = sum / 10;`.
   * Create a new node with `sum % 10` and link it: `curr->next = new ListNode(sum % 10);`.
   * Advance `curr`.
   * Advance `l1` and `l2` pointers if they are not null.
5. Return `dummy.next`.

### Why it is Optimal
* **Time**: $O(\max(M, N))$ since we traverse the longer list exactly once.
* **Space**: $O(1)$ auxiliary space (excluding the output list) because we only keep a few pointers and a single `carry` variable.

### C++17 Code
```cpp
class OptimalAddTwoNumbers {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode dummy(0); // Dummy node to simplify head initialization
        ListNode* curr = &dummy;
        int carry = 0;

        // Loop until lists are exhausted AND there is no remaining carry
        while (l1 != nullptr || l2 != nullptr || carry != 0) {
            int sum = carry;

            if (l1 != nullptr) {
                sum += l1->val;
                l1 = l1->next; // Move to next node
            }
            if (l2 != nullptr) {
                sum += l2->val;
                l2 = l2->next; // Move to next node
            }

            carry = sum / 10;                     // Compute new carry
            curr->next = new ListNode(sum % 10);  // Store digit
            curr = curr->next;
        }

        return dummy.next;
    }
};
```

---

## 8. Code Walkthrough

* `ListNode dummy(0);`
  * Creates a stack-allocated dummy node. This is a very clean trick in C++: instead of using `new ListNode(0)`, we stack-allocate it to avoid manual deletion, and return `dummy.next`.
* `while (l1 != nullptr || l2 != nullptr || carry != 0)`
  * Crucial loop condition: even if both lists are finished, if we have a leftover `carry` (like `9 + 1 = 10`), the loop runs one more time to append the `1` digit.
* `if (l1 != nullptr) { sum += l1->val; l1 = l1->next; }`
  * If `l1` is not empty, we add its value and move its pointer to the next node. If it is null, we do nothing (acting as 0).
* `carry = sum / 10;`
  * Arithmetic division: if `sum` is 18, `carry` becomes 1. If `sum` is 8, `carry` becomes 0.
* `curr->next = new ListNode(sum % 10);`
  * Appends the remainder digit to the new list.
* `return dummy.next;`
  * Returns the actual head pointer of our result list.

---

## 9. Dry Run

Let's dry run the optimal solution on:
`l1 = [2, 4, 3]` (representing 342)
`l2 = [5, 6, 4]` (representing 465)

| Step | `l1` node | `l2` node | `carry` | `sum` | Digit created (`sum % 10`) | New `carry` | Result List |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---|
| Setup | `2` | `5` | `0` | - | - | - | `dummy` |
| **1** | `2` | `5` | `0` | `2 + 5 + 0 = 7` | `7` | `0` | `7` |
| **2** | `4` | `6` | `0` | `4 + 6 + 0 = 10`| `0` | `1` | `7 -> 0` |
| **3** | `3` | `4` | `1` | `3 + 4 + 1 = 8` | `8` | `0` | `7 -> 0 -> 8` |
| End | `nullptr` | `nullptr` | `0` | - | - | - | Loop terminates |

**Result**: Return `[7, 0, 8]` (representing 807).

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Description |
|:---|:---:|:---:|:---|
| **Brute Force** | $O(M + N)$ | $O(1)$ | Fails due to arithmetic overflow. |
| **Better (Strings)** | $O(M + N)$ | $O(M + N)$ | Works for all, but duplicates digits in memory. |
| **Optimal (Direct)** | $O(\max(M, N))$ | $O(1)$ auxiliary | Best. Minimal memory overhead. |

* **Time Complexity**: $O(\max(M, N))$ because we perform a single loop that runs for a number of iterations equal to the length of the longer list, plus at most 1 additional iteration for a trailing carry.
* **Space Complexity**: $O(1)$ auxiliary space since we do not allocate any temporary structures (vectors/strings). The $O(\max(M, N))$ space to build the result list is required by the problem statement.

---

## 11. Common Mistakes

1. **Losing Final Carry**: Forgetting to process the carry after the lists are exhausted. E.g., adding `5` and `5` results in `0`, forgetting to add the carry `1` at the end.
2. **Null Pointer Dereference**: Accessing `l1->val` or `l2->val` without checking if `l1` or `l2` is `nullptr`.
3. **Modifying Input Lists**: Modifying the original nodes of `l1` or `l2` to store the result. Unless requested by the interviewer to optimize output space, always create a new list.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it Matters |
|:---|:---|:---|:---|
| **Varying Lengths** | `[9, 9] + [1]` | `[0, 0, 1]` | Tests carry propagation when one list ends early. |
| **Single Element Zero** | `[0] + [0]` | `[0]` | Verifies the base single digit zero condition. |
| **Carry Propagates to End**| `[9, 9] + [9, 9]` | `[8, 9, 1]` | Ensures a new node is created for the final carry. |
| **Large Numbers** | 100-digit numbers | Correct sum list | Tests overflow prevention (C++ native limits). |

---

## 13. Interview Explanation

"To add two numbers represented as linked lists in reverse order, we can traverse both lists starting from their heads. Because they are stored in reverse, the heads represent the ones place, which matches the starting point of standard addition.

A conversion approach where we change the lists to integers will fail due to overflow on large numbers.

Instead, we can implement an optimal single-pass approach:
1. We initialize a dummy node to build our result list, and a carry variable set to 0.
2. We loop while either list has elements, or if there is a leftover carry.
3. In each iteration, we add the current node values (or 0 if a list is exhausted) along with the carry.
4. We extract the new carry using division by 10, and append a new node with the digit (`sum % 10`) to our result.
5. Finally, we advance the pointers and return the merged list.
This runs in $O(\max(M, N))$ time and uses $O(1)$ auxiliary space."

---

## 14. Follow-up Questions

1. **What if the digits are stored in normal (forward) order instead of reverse? (e.g. `3 -> 4 -> 2` represents 342)**
   * *Answer*: If they are in forward order, we cannot add them from left to right because the heads represent the most significant digits. We have two main ways to solve this:
     1. Reverse both input lists, add them using the reverse method, and reverse the resulting list.
     2. Use two stacks to store the nodes, and then pop elements to add them from right to left.
2. **How would you optimize this if we had to perform this in-place (no new allocations)?**
   * *Answer*: We could reuse the nodes of the longer list (`l1` or `l2`) to write the sum digits, and only allocate a new node if a final carry extends the list size.

---

## 15. Variations

1. **Add Two Numbers II (LeetCode 445)**:
   * *Difference*: Numbers are represented in forward order (MSB at the head).
   * *Solution*: Reverse lists or use stacks.
2. **Plus One Linked List**:
   * *Difference*: Add 1 to a linked list representing a number.

---

## 16. Revision Notes

* **Core Mnemonic**: `while(l1 || l2 || carry)` is the loop boundary.
* **Math Refresher**:
  * Digit: `sum % 10`
  * Carry: `sum / 10`
* **Complexity summary**: Time: $O(\max(M, N))$ | Space: $O(1)$ auxiliary.

---

## 17. Memorization Version

```cpp
// 1-min Review: Add Two Numbers LL
ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
    ListNode dummy(0);
    ListNode* curr = &dummy;
    int carry = 0;
    while (l1 || l2 || carry) {
        int sum = carry;
        if (l1) { sum += l1->val; l1 = l1->next; }
        if (l2) { sum += l2->val; l2 = l2->next; }
        carry = sum / 10;
        curr->next = new ListNode(sum % 10);
        curr = curr->next;
    }
    return dummy.next;
}
```
