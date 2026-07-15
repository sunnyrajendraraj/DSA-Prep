# Sort Linked List (Merge Sort)

## 1. Problem Statement

Given the `head` of a singly linked list, return the list after sorting it in ascending order. The solution must run in $O(N \log N)$ time and use $O(\log N)$ stack space (or $O(1)$ auxiliary space if done iteratively).

### Input
- `head`: A pointer to the head of a singly linked list.

### Output
- A pointer to the head of the sorted singly linked list.

### Constraints
- The number of nodes in the list is in the range $[0, 5 \cdot 10^4]$.
- $-10^5 \le \text{Node.val} \le 10^5$

### Observations
1. Unlike arrays, we cannot access elements of a linked list in $O(1)$ time by index.
2. Standard sorting algorithms like Quick Sort or Heap Sort rely heavily on random access, making them less suitable or harder to optimize for linked lists.
3. Merge Sort fits linked lists perfectly because merging two sorted lists can be done in-place by simply adjusting pointers without allocating new nodes.

---

## 2. Interview Clarification

Before coding, clarify:
1. **Q:** Can we modify the values of the nodes directly, or must we rearrange the nodes?
   **A:** Rearranging the pointers (nodes themselves) is the expected approach. Changing values is generally frowned upon as it doesn't demonstrate linked list pointer manipulation.
2. **Q:** What should be returned for an empty list or a list with a single node?
   **A:** Return the head as-is.
3. **Q:** Are there duplicate values in the list?
   **A:** Yes, the sort should be stable (duplicates should retain their original relative order if applicable, though for simple integers stability is trivial).

---

## 3. Thinking Process

Let's build the intuition step-by-step:
1. **How do we sort an array in $O(N \log N)$?**
   - Merge Sort: Divide the array into two halves, recursively sort them, and merge the sorted halves.
   - Quick Sort: Choose a pivot, partition the array, and recursively sort the partitions.
2. **Applying Merge Sort to Linked Lists:**
   - **Divide:** How do we find the middle of a linked list to split it?
     - We can use the **Slow and Fast Pointer** technique. Move `slow` by 1 step and `fast` by 2 steps. When `fast` reaches the end, `slow` will be at the middle.
     - We split the list into two parts: `head` to `slow`, and `slow->next` to the end. We must set the next pointer of the middle node to `nullptr` to detach the two halves.
   - **Conquer:** Recursively call merge sort on both halves.
   - **Combine:** Merge the two sorted lists. This is a standard problem: "Merge Two Sorted Lists." We can do this in-place using a dummy node and a pointer traversal.

---

## 4. Pattern Recognition

This problem utilizes two classic patterns:
- **Divide and Conquer (Merge Sort):** Recursively dividing the problem into subproblems of half the size.
- **Two Pointers (Slow and Fast):** To find the midpoint of the linked list in $O(N)$ time.

---

## 5. Brute Force Solution

### Algorithm
1. Traverse the linked list and copy all node values into a dynamic array (like `std::vector<int>`).
2. Sort the array using the built-in library sorting function (`std::sort`), which uses IntroSort ($O(N \log N)$ time).
3. Traverse the list again, overwriting the values of the nodes with the sorted values from the array.
4. Return the head of the list.

### Dry Run
List: `4 -> 2 -> 1 -> 3`
1. Array: `[4, 2, 1, 3]`
2. Sort Array: `[1, 2, 3, 4]`
3. Rewrite values:
   - Node 1 value becomes `1`.
   - Node 2 value becomes `2`.
   - Node 3 value becomes `3`.
   - Node 4 value becomes `4`.
4. Output: `1 -> 2 -> 3 -> 4`

### Complexity Analysis
- **Time Complexity:** $O(N \log N)$ to sort the array of size $N$, plus $O(N)$ for copying.
- **Space Complexity:** $O(N)$ auxiliary space to store the node values in the array.

### Pros and Cons
- **Pros:** Extremely easy to implement.
- **Cons:** High space complexity ($O(N)$) and does not actually sort the nodes by modifying pointers (violates the spirit of linked list manipulation).

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

struct ListNode {
    int val;
    ListNode* next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode* next) : val(x), next(next) {}
};

class SolutionBrute {
public:
    ListNode* sortList(ListNode* head) {
        if (head == nullptr || head->next == nullptr) {
            return head;
        }
        
        std::vector<int> values;
        ListNode* curr = head;
        while (curr != nullptr) {
            values.push_back(curr->val);
            curr = curr->next;
        }
        
        std::sort(values.begin(), values.end());
        
        curr = head;
        for (int val : values) {
            curr->val = val;
            curr = curr->next;
        }
        
        return head;
    }
};
```

---

## 6. Better Solution

Since there is no meaningful intermediate solution between copying to an array and doing in-place Merge Sort, the brute force approach jumps directly to the optimal Merge Sort solution.

---

## 7. Optimal Solution (Merge Sort)

### Why is Merge Sort preferred over Quick Sort for Linked Lists?
1. **No Random Access:** Quick Sort relies heavily on random access ($O(1)$ index access) to partition the array and select pivots. In linked lists, traversing to a specific index takes $O(N)$ time, which degrades Quick Sort's performance.
2. **$O(1)$ Merge Overhead:** Merging two sorted arrays requires an auxiliary array of size $O(N)$ to copy elements. However, merging two sorted linked lists can be done by simply changing pointers, requiring $O(1)$ auxiliary space.
3. **Worst-Case Guarantee:** Merge Sort guarantees $O(N \log N)$ time complexity in all cases. Quick Sort can degrade to $O(N^2)$ time if pivot selection is poor (e.g., already sorted lists).

### Algorithm
1. **Base Case:** If the list is empty or has only one node, return `head`.
2. **Find Middle:** Use the slow and fast pointer approach to find the midpoint of the list.
   - To split correctly, we want the slow pointer to stop at the node *before* the middle or at the middle.
   - Specifically: `slow` starts at `head`, `fast` starts at `head->next` (or we track `prev` of `slow`).
   - Move `slow` by 1 step and `fast` by 2 steps.
3. **Split List:**
   - The middle node is `slow`.
   - The head of the right sublist is `right_head = slow->next`.
   - Disconnect the lists: `slow->next = nullptr`.
4. **Sort Recursively:**
   - Sort the left half: `left = sortList(head)`.
   - Sort the right half: `right = sortList(right_head)`.
5. **Merge:** Merge the two sorted lists `left` and `right` and return the merged list's head.

### C++17 Code
```cpp
class Solution {
public:
    ListNode* sortList(ListNode* head) {
        // Base case: if list has 0 or 1 elements, it is already sorted
        if (head == nullptr || head->next == nullptr) {
            return head;
        }
        
        // Find the middle of the list
        ListNode* mid = getMid(head);
        ListNode* right_head = mid->next;
        mid->next = nullptr; // Split the list into two halves
        
        // Recursively sort the two halves
        ListNode* left = sortList(head);
        ListNode* right = sortList(right_head);
        
        // Merge the sorted halves
        return merge(left, right);
    }

private:
    // Helper function to find the midpoint
    ListNode* getMid(ListNode* head) {
        ListNode* slow = head;
        ListNode* fast = head->next; // Start fast one step ahead
        
        while (fast != nullptr && fast->next != nullptr) {
            slow = slow->next;
            fast = fast->next->next;
        }
        return slow; // slow will point to the middle node (or first middle in even length)
    }
    
    // Helper function to merge two sorted lists
    ListNode* merge(ListNode* list1, ListNode* list2) {
        ListNode dummy(0);
        ListNode* curr = &dummy;
        
        while (list1 != nullptr && list2 != nullptr) {
            if (list1->val < list2->val) {
                curr->next = list1;
                list1 = list1->next;
            } else {
                curr->next = list2;
                list2 = list2->next;
            }
            curr = curr->next;
        }
        
        // Append remaining nodes
        if (list1 != nullptr) {
            curr->next = list1;
        } else {
            curr->next = list2;
        }
        
        return dummy.next;
    }
};
```

---

## 8. Code Walkthrough

1. `ListNode* mid = getMid(head);`
   - Invokes the helper to find the midpoint of the linked list. If the list has elements `[4, 2, 1, 3]`, `slow` starts at `4`, `fast` at `2`.
   - `fast->next` is `1` (not null), so `slow` moves to `2`, `fast` moves to `3`.
   - In the next check, `fast->next` is `nullptr`. The loop terminates. `slow` (node `2`) is returned.
2. `ListNode* right_head = mid->next; mid->next = nullptr;`
   - `right_head` gets node `1`.
   - The pointer from `2` to `1` is severed by setting `mid->next = nullptr`. We now have two independent sublists: `4 -> 2` and `1 -> 3`.
3. `ListNode* left = sortList(head); ListNode* right = sortList(right_head);`
   - Recursively sorts both sublists.
4. `merge(left, right)`
   - Uses a stack-allocated dummy node `ListNode dummy(0)` to build the merged list.
   - Compares the heads of both lists and links the smaller node next.
   - Returns `dummy.next` which contains the head of the fully merged list.

---

## 9. Dry Run

List: `4 -> 2 -> 1 -> 3`

### Call Tree:
```
              sortList([4,2,1,3])
                 /          \
         sortList([4,2])   sortList([1,3])
           /      \         /       \
      sort([4]) sort([2]) sort([1]) sort([3])
```

### Trace:
1. `sortList([4,2,1,3])`:
   - `mid` is `2`. Split into `[4,2]` and `[1,3]`.
2. `sortList([4,2])`:
   - `mid` is `4`. Split into `[4]` and `[2]`.
   - `sortList([4])` -> returns `[4]` (Base case).
   - `sortList([2])` -> returns `[2]` (Base case).
   - `merge([4], [2])` -> returns `[2, 4]`.
3. `sortList([1,3])`:
   - `mid` is `1`. Split into `[1]` and `[3]`.
   - `sortList([1])` -> returns `[1]`.
   - `sortList([3])` -> returns `[3]`.
   - `merge([1], [3])` -> returns `[1, 3]`.
4. Merge step of the root call:
   - `merge([2, 4], [1, 3])`
     - Compare `2` and `1`. Choose `1`. `curr->next = 1`.
     - Compare `2` and `3`. Choose `2`. `curr->next = 2`.
     - Compare `4` and `3`. Choose `3`. `curr->next = 3`.
     - List 2 is empty. Append remaining List 1 (`4`).
     - Returns `1 -> 2 -> 3 -> 4`.

---

## 10. Complexity Analysis

| Scenario | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Worst Case** | $O(N \log N)$ | $O(\log N)$ | Always splits the list in half. Recursive stack depth is $\log N$. |
| **Average Case** | $O(N \log N)$ | $O(\log N)$ | Standard behavior of Merge Sort. |
| **Best Case** | $O(N \log N)$ | $O(\log N)$ | Even if the list is already sorted, we split and merge. |

*Note: It is possible to implement Merge Sort iteratively using bottom-up merging to achieve $O(1)$ space, but it is highly complex and rarely asked to be coded in full during a 45-minute interview.*

---

## 11. Common Mistakes

1. **Infinite Recursion (Stack Overflow):**
   - If the midpoint is not selected correctly, splitting a list of size 2 (e.g. `4 -> 2`) can result in one half having size 2 and the other size 0, leading to infinite recursion. Starting `fast = head->next` avoids this by ensuring `slow` stops on the first node for size 2 lists.
2. **Forgetting to Sever the Link:**
   - Forgetting `mid->next = nullptr` will keep the two sublists linked, resulting in cycles or double-processing during recursive steps.
3. **Memory Management in C++:**
   - When merging, make sure not to create unnecessary copies of nodes. Modify the pointer references directly.

---

## 12. Edge Cases

| Edge Case | Input | Expected Output | Why It Matters |
| :--- | :--- | :--- | :--- |
| **Empty List** | `nullptr` | `nullptr` | Base case prevention of null pointer dereference. |
| **Single Node** | `5` | `5` | Recursion base case. |
| **Already Sorted** | `1 -> 2 -> 3` | `1 -> 2 -> 3` | Must not disrupt order. |
| **Reverse Sorted** | `3 -> 2 -> 1` | `1 -> 2 -> 3` | Must sort correctly. |
| **All Duplicates** | `2 -> 2 -> 2` | `2 -> 2 -> 2` | Must handle equal elements without infinite loops. |

---

## 13. Interview Explanation

"To sort a linked list in $O(N \log N)$ time, the most suitable algorithm is Merge Sort. We cannot use Quick Sort easily because linked lists don't support random access. 

The algorithm works in three steps: First, we find the middle of the linked list using the slow and fast pointer approach. Second, we split the list into two halves by breaking the link at the middle node, and then recursively call merge sort on both halves. Finally, we merge the two sorted halves back together in-place by comparing values and adjusting pointers. This recursive process takes $O(N \log N)$ time and $O(\log N)$ auxiliary space for the recursion stack."

---

## 14. Follow-up Questions

1. **Q:** Can we implement this in $O(1)$ auxiliary space?
   **A:** Yes, by using a bottom-up iterative merge sort. We merge sublists of sizes 1, 2, 4, 8, etc., iteratively. This avoids recursive stack frames.
2. **Q:** How does Merge Sort compare to Insertion Sort for linked lists?
   **A:** Insertion Sort takes $O(N^2)$ time in the worst case but runs in $O(1)$ space and is efficient for very small or nearly sorted lists. Merge Sort is much faster for larger datasets with $O(N \log N)$ time complexity.

---

## 15. Variations

1. **Sort a Linked List containing only 0s, 1s, and 2s:**
   - Can be solved in $O(N)$ time by counting frequencies or using three dummy pointers to partition the list into 0s, 1s, and 2s.
2. **Merge K Sorted Lists:**
   - A extension where we use a min-heap to merge $K$ sorted linked lists.

---

## 16. Revision Notes

- **Split Rule:** To split a 2-node list safely, `fast` must start at `head->next`.
- **Merge Logic:** Use a local stack-allocated dummy node `ListNode dummy(0)` to avoid heap allocation overhead during merge.
- **Why Merge Sort:** $O(1)$ merge overhead for lists vs $O(N)$ for arrays.

---

## 17. Memorization Version

```cpp
// 1. Base case: if (!head || !head->next) return head;
// 2. Find middle node using slow/fast (fast starts at head->next).
// 3. Split list: right = mid->next, mid->next = nullptr.
// 4. Recursive sort: left = sortList(head), right = sortList(right).
// 5. Merge left and right using helper merge function.
```
