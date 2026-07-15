# Reverse Linked List in Groups of K

## 1. Problem Statement

Given the `head` of a singly linked list and an integer `k`, reverse the nodes of the list `k` at a time and return the modified list.

`k` is a positive integer and is less than or equal to the length of the linked list. If the number of nodes is not a multiple of `k` then left-out nodes, in the end, should remain as they are (i.e., not reversed).

You may not alter the values in the list's nodes; only nodes themselves may be changed.

### Input
- `head`: A pointer to the head of a singly linked list.
- `k`: An integer representing the group size.

### Output
- A pointer to the head of the modified linked list.

### Constraints
- The number of nodes in the list is `n`.
- $1 \le k \le n \le 5000$
- $0 \le \text{Node.val} \le 1000$

### Observations
1. If $k = 1$, the list remains unchanged.
2. If $n < k$, no nodes are reversed; the list is returned as-is.
3. Only full groups of size `k` are reversed. Any remaining nodes at the end that form a group of size $< k$ must remain in their original order.
4. The relative connections between the reversed groups must be maintained correctly.

---

## 2. Interview Clarification

Before writing any code, a candidate should clarify the following points with the interviewer:
1. **Q:** What should we do if the list is empty (`nullptr`) or has only one node?
   **A:** Return the head as-is.
2. **Q:** Can we modify the values inside the nodes instead of changing the actual pointers?
   **A:** No, the problem specifies that we must physically reorder the nodes by modifying the pointers, not just copy values.
3. **Q:** How do we handle the last group if it has fewer than `k` nodes?
   **A:** Leave it as-is (do not reverse). For example, if the list is `1 -> 2 -> 3 -> 4 -> 5` and `k = 3`, only the first 3 nodes are reversed: `3 -> 2 -> 1 -> 4 -> 5`.
4. **Q:** Is `k` guaranteed to be a positive integer?
   **A:** Yes, $1 \le k \le n$.

---

## 3. Thinking Process

Let's build the intuition step-by-step:
1. **Understand Singly Linked List Reversal:**
   To reverse a simple linked list, we use three pointers (`prev`, `curr`, `next_node`). We iterate through the list and flip the `curr->next` pointer to point to `prev`.
2. **Reversing in Groups of size K:**
   - If we want to reverse a subsegment of size `k`, we need to first check if there are at least `k` nodes available in this segment.
   - If there are fewer than `k` nodes left, we do nothing and return the current segment head.
   - If there are at least `k` nodes, we reverse them.
3. **Connecting Reversed Segments:**
   - Once we reverse a group of `k` nodes, the original `head` of this group becomes the `tail` of the reversed group.
   - This new `tail` must link to the head of the next reversed (or unreversed) segment.
   - This naturally suggests a **recursive** formulation:
     `new_head_of_current_group->next = reverseKGroup(head_of_next_group, k)`
4. **Eliminating Recursion (Iterative Optimal):**
   - Recursion uses $O(n/k)$ stack space due to active function calls.
   - To make it $O(1)$ space, we must track the tail of the *previous* group and link it to the new head of the *current* group as we iterate.
   - A dummy node helps handle the head of the entire list cleanly.

---

## 4. Pattern Recognition

This problem fits the **Linked List Reversal Pattern**.
- **Why this pattern?** We are modifying the structure of a linked list by reversing subsets of nodes.
- **Key Technique:** Group-by-group pointer manipulation. We count $k$ nodes ahead to check eligibility, reverse the $k$-node sub-list, and link it back to the rest of the list.

---

## 5. Brute Force Solution

### Algorithm
The simplest approach is to use a recursive formulation:
1. Count if there are at least `k` nodes left starting from the current node `head`.
2. If there are fewer than `k` nodes, return `head` as-is.
3. Otherwise, reverse the first `k` nodes using the standard iterative reversal technique.
4. After reversing, the original `head` is now the tail of this segment.
5. Recursively call `reverseKGroup` on the $(k+1)$-th node (which is the node immediately following the original segment).
6. Connect the original `head->next` to the result of the recursive call.
7. Return the new head of this reversed segment (which was the $k$-th node before reversal).

### Dry Run
List: `1 -> 2 -> 3 -> 4 -> 5`, `k = 2`
1. Call on node `1`. Count $2$ nodes: `1`, `2`. Eligible.
2. Reverse `1 -> 2` to get `2 -> 1`.
3. Node `1` is the new tail.
4. Recursively call on node `3`.
5. Call on node `3`. Count $2$ nodes: `3`, `4`. Eligible.
6. Reverse `3 -> 4` to get `4 -> 3`.
7. Node `3` is the new tail.
8. Recursively call on node `5`.
9. Call on node `5`. Count: only 1 node remaining. Less than `k=2`. Return `5`.
10. Unwinding recursion:
    - Node `3->next = 5`. Segment becomes `4 -> 3 -> 5`. Return `4`.
    - Node `1->next = 4`. Segment becomes `2 -> 1 -> 4 -> 3 -> 5`. Return `2`.

### Complexity Analysis
- **Time Complexity:** $O(N)$ because we traverse each node at most twice (once for counting, once for reversing).
- **Space Complexity:** $O(N/k)$ recursion stack space, where $N$ is the total number of nodes.

### Pros and Cons
- **Pros:** Extremely clean code, simple to explain and implement.
- **Cons:** Uses stack space proportional to $N/k$, which is not $O(1)$ auxiliary space.

### C++17 Code
```cpp
#include <iostream>

struct ListNode {
    int val;
    ListNode* next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode* next) : val(x), next(next) {}
};

class SolutionRecursive {
public:
    ListNode* reverseKGroup(ListNode* head, int k) {
        // Base case: check if we have at least k nodes left
        ListNode* curr = head;
        int count = 0;
        while (curr != nullptr && count < k) {
            curr = curr->next;
            count++;
        }
        
        // If we have fewer than k nodes, return head as-is
        if (count < k) {
            return head;
        }
        
        // Reverse first k nodes
        ListNode* prev = nullptr;
        curr = head;
        for (int i = 0; i < k; ++i) {
            ListNode* next_node = curr->next;
            curr->next = prev;
            prev = curr;
            curr = next_node;
        }
        
        // At this point, prev is the new head of the reversed segment,
        // and head is the tail of the reversed segment.
        // Link the tail to the result of the next recursive call.
        head->next = reverseKGroup(curr, k);
        
        return prev;
    }
};
```

---

## 6. Better Solution

There is no intermediate "better" solution; we directly optimize the recursive space complexity to $O(1)$ auxiliary space using an iterative method.

---

## 7. Optimal Solution

### Algorithm
To reverse the list in groups of `k` iteratively with $O(1)$ extra space, we need to carefully track the connections between the boundaries of consecutive groups.
1. Create a `dummy` node and set `dummy->next = head`.
2. Maintain `group_prev` pointing to the node right before the current group being processed. Initially, `group_prev = dummy`.
3. In a loop:
   - Find the $k$-th node from `group_prev`. Let this be `group_end`. If there are fewer than $k$ nodes left, we break and stop.
   - Keep a reference to the node after the group: `next_group_start = group_end->next`.
   - Reverse the current group of $k$ nodes. To do this, we can detach the group temporarily (set `group_end->next = nullptr`) or reverse it in place. Let's reverse it in place:
     - Set `prev = next_group_start` (so the tail of the reversed group points directly to the next group).
     - Set `curr = group_prev->next` (start of the current group).
     - Standard loop to reverse: `next_node = curr->next`, `curr->next = prev`, `prev = curr`, `curr = next_node`. Run this $k$ times.
   - Connect the preceding part: `group_prev->next` must now point to the new head of the reversed group (which is `group_end`).
   - Move `group_prev` to the tail of the current reversed group. The tail of the current reversed group is the node that was the head before reversal. We can find this by holding a reference to `group_prev->next` before changing it, or since we know it, we save `temp = group_prev->next` before pointer updates, then set `group_prev = temp`.

### Why Optimal?
- Reuses existing nodes without any extra heap allocation.
- Iterative execution ensures $O(1)$ stack space.

### C++17 Code
```cpp
#include <iostream>

class Solution {
public:
    ListNode* reverseKGroup(ListNode* head, int k) {
        if (head == nullptr || k == 1) {
            return head;
        }
        
        // Dummy node to handle head updates easily
        ListNode* dummy = new ListNode(0);
        dummy->next = head;
        
        ListNode* group_prev = dummy;
        
        while (true) {
            // Find the k-th node
            ListNode* group_end = group_prev;
            int count = 0;
            while (group_end != nullptr && count < k) {
                group_end = group_end->next;
                count++;
            }
            
            // If fewer than k nodes left, we are done
            if (group_end == nullptr) {
                break;
            }
            
            // Node right after the current group
            ListNode* next_group_start = group_end->next;
            
            // Reverse the group of size k.
            // We initialize 'prev' to 'next_group_start' so the original head 
            // of the group will automatically link to the next group after reversal.
            ListNode* prev = next_group_start;
            ListNode* curr = group_prev->next;
            ListNode* group_start = curr; // Save reference to the start of this group
            
            for (int i = 0; i < k; ++i) {
                ListNode* next_node = curr->next;
                curr->next = prev;
                prev = curr;
                curr = next_node;
            }
            
            // Connect group_prev to the new head of this reversed group (which is 'prev')
            group_prev->next = prev;
            
            // Update group_prev to be the end of the newly reversed group (which is 'group_start')
            group_prev = group_start;
        }
        
        ListNode* new_head = dummy->next;
        delete dummy; // Clean up memory
        return new_head;
    }
};
```

---

## 8. Code Walkthrough

Let's break down the optimal iterative implementation:
1. `ListNode* dummy = new ListNode(0); dummy->next = head;`
   - Setting up a dummy head allows us to not worry about special logic for updating the `head` pointer of the entire list.
2. `ListNode* group_prev = dummy;`
   - Keeps track of the node just before the current group we want to reverse.
3. `ListNode* group_end = group_prev;`
   - Finding the end of the current group of `k` nodes.
4. `if (group_end == nullptr) break;`
   - If we run out of nodes before reaching $k$, we exit the loop, leaving the remaining nodes untouched.
5. `ListNode* next_group_start = group_end->next;`
   - Saves the starting node of the next group so we don't lose it when we start reversing pointers.
6. `ListNode* prev = next_group_start;`
   - By initializing `prev` to `next_group_start`, the first node we reverse (which becomes the last node of the reversed group) will point directly to the start of the next group.
7. `group_prev->next = prev;`
   - Links the previous group's tail to the new head of the currently reversed group.
8. `group_prev = group_start;`
   - Prepares `group_prev` for the next iteration.

---

## 9. Dry Run

List: `1 -> 2 -> 3 -> 4 -> 5`, `k = 2`

| Iteration | `group_prev` | `group_end` (k-th node) | Reversing Group | State of List | `group_prev` Next State |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Initial** | `dummy(0)` | - | - | `dummy -> 1 -> 2 -> 3 -> 4 -> 5` | - |
| **1** | `dummy(0)` | `2` | `1 -> 2` | `dummy -> 2 -> 1 -> 3 -> 4 -> 5` | `1` |
| **2** | `1` | `4` | `3 -> 4` | `dummy -> 2 -> 1 -> 4 -> 3 -> 5` | `3` |
| **3** | `3` | `nullptr` (only `5` left) | - | Loop terminates | - |

Final output: `2 -> 1 -> 4 -> 3 -> 5`

---

## 10. Complexity Analysis

| Solution | Time Complexity | Space Complexity | Details |
| :--- | :--- | :--- | :--- |
| **Recursive** | $O(N)$ | $O(N/k)$ | One traversal for check + one traversal for reverse. Recursion stack size $\approx N/k$. |
| **Iterative (Optimal)** | $O(N)$ | $O(1)$ | Same traversal behavior, but pointer manipulations are done inline without stack allocations. |

---

## 11. Common Mistakes

1. **Incorrect Linkage to Next Group:**
   Forgetting to connect the tail of the current reversed group to the head of the *next* group. If done incorrectly, the list will either get severed or form cycles.
2. **Reversing when $< k$ Nodes Remain:**
   Failing to check if there are at least $k$ nodes remaining *before* performing the reversal.
3. **Memory Leak with Dummy Nodes:**
   Creating a `dummy` node on the heap using `new` in C++ and not deleting it before returning the result.
4. **Pointer Loss:**
   Reversing without saving the reference to the node that follows the group (`next_group_start`), leading to lost access to the remainder of the list.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Behavior | Why It Matters |
| :--- | :--- | :--- | :--- |
| **List length < k** | `1 -> 2`, `k = 3` | `1 -> 2` (No change) | Must not crash or attempt reversal. |
| **List length == k** | `1 -> 2`, `k = 2` | `2 -> 1` | Correctly reverses the entire list. |
| **k = 1** | `1 -> 2 -> 3`, `k = 1` | `1 -> 2 -> 3` | Code should return head immediately. |
| **Empty list** | `nullptr`, `k = 2` | `nullptr` | Must handle null references gracefully. |
| **Exact multiple of k** | `1 -> 2 -> 3 -> 4`, `k = 2` | `2 -> 1 -> 4 -> 3` | All elements are reversed in groups. |

---

## 13. Interview Explanation

"To reverse a linked list in groups of $k$, we can iterate through the list and process it segment by segment. First, we must verify if there are at least $k$ nodes left to reverse. If there are fewer than $k$ nodes, we leave them as-is and finish. If there are at least $k$ nodes, we reverse this subsegment using three pointers. 

To achieve optimal $O(1)$ auxiliary space, we implement this iteratively. We use a dummy head node to simplify connection logic. For every group, we locate its tail (the $k$-th node), disconnect it from the next segment, reverse the group, and then stitch the previous group's tail to this group's new head. Finally, we connect this group's new tail to the remaining unreversed list. This ensures we process the list in $O(N)$ time and $O(1)$ space."

---

## 14. Follow-up Questions

1. **Q:** What if we need to reverse the remaining nodes if they are fewer than `k`?
   **A:** We would modify the algorithm to skip the check for $k$ nodes at the end, and simply reverse whatever remaining nodes are left.
2. **Q:** Can we solve this problem using a Stack?
   **A:** Yes. We can push $k$ nodes onto a stack and pop them to reverse them. However, this would use $O(k)$ extra space.

---

## 15. Variations

1. **Reverse Alternate K Nodes:**
   Reverse the first $k$ nodes, skip the next $k$ nodes, reverse the next $k$ nodes, and so on.
   *Difference:* We change the connection logic so that alternate groups are skipped instead of reversed.
2. **Reverse Linked List in Groups of Given Sizes:**
   Instead of a constant $k$, we are given an array of sizes `[g1, g2, g3...]` and we reverse groups of those sizes.

---

## 16. Revision Notes

- **Counting First:** Always verify that a segment of size $k$ exists before starting pointer operations on it.
- **Initialization Trick:** Initializing `prev = next_group_start` before starting the loop saves a separate step of connecting the group tail to the next group.
- **Dummy Node:** Essential for cleaning up edge cases involving the head of the list.
- **Complexity:** Time $O(N)$, Space $O(1)$.

---

## 17. Memorization Version

```cpp
// 1. Check if k nodes exist. If not, break.
// 2. Keep pointer to next group: next_group_start = group_end->next.
// 3. Initialize prev = next_group_start, curr = group_prev->next.
// 4. Reverse k nodes: curr->next = prev, update prev and curr.
// 5. Connect previous group tail: group_prev->next = prev.
// 6. Update group_prev = group_start.
```
