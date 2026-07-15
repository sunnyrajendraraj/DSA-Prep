# Merge Two Sorted Linked Lists

## 1. Problem Statement
Given the heads of two sorted linked lists, `list1` and `list2`, merge them into a single sorted linked list. The merged list should be made by splicing together the nodes of the first two lists. Return the head of the merged linked list.

### Input
- `list1`: A pointer to the head of the first sorted singly linked list.
- `list2`: A pointer to the head of the second sorted singly linked list.

### Output
- A pointer to the head of the merged sorted singly linked list.

### Constraints
- The number of nodes in both lists is in the range `[0, 50]`.
- `-100 <= Node.val <= 100`
- Both `list1` and `list2` are sorted in non-decreasing order.

### Observations
- Since the input lists are already sorted, we can build the merged list in a single pass by comparing the head nodes of both lists.
- We should splice the existing nodes in-place rather than allocating new nodes to save space.

---

## 2. Interview Clarification Questions
1. **Can we modify the original lists?**
   - *Answer:* Yes, we should merge them in-place by changing the node pointers (splicing). No new nodes should be created.
2. **What if one of the input lists is empty?**
   - *Answer:* Return the other list as-is. If both are empty, return `nullptr`.
3. **Is it possible that the node values are duplicated?**
   - *Answer:* Yes, the list should preserve all duplicates in sorted order.

---

## 3. Thinking Process
- **First Instinct (Collect & Sort):** Copy all node values from both lists into an array, sort the array, and then create a brand new linked list. This is very easy but uses $O(M+N)$ extra space and runs in $O((M+N) \log(M+N))$ time.
- **Splicing Pointers (Two-Pointer Merge):**
  - We can maintain a pointer for `list1` and a pointer for `list2`.
  - Compare the values at both pointers:
    - Whichever is smaller is appended to the merged list.
    - Advance that list's pointer.
  - Repeat this until one of the lists is exhausted.
  - Append the remainder of the non-empty list directly to the merged list.
- **Simplifying Head Assignment (Dummy Node):**
  - To avoid checking if the merged head has been set yet, we use a `dummy` node.
  - A tail pointer `tail` starts at `dummy` and tracks the end of our newly merged list.
- **Recursive Thinking:**
  - The subproblem is: Merge the remaining portions of both lists.
  - If `list1->val < list2->val`, then `list1` is the head of the merged list. The next node of `list1` will be the result of merging `list1->next` and `list2`.
  - Conversely, if `list2->val <= list1->val`, then `list2` is the head of the merged list. The next node of `list2` will be the result of merging `list1` and `list2->next`.

---

## 4. Pattern Recognition
- **Pattern:** Two Pointers (Merge Step).
- **Why this topic?** This is the core merging logic used in Merge Sort. It teaches how to construct a new sequence in-place from two sorted sequences with constant space overhead.

---

## 5. Brute Force Solution (Collect & Sort)
Collect all node values, sort them, and rebuild the list.

### Algorithm
1. Traverse `list1` and push all values into a vector.
2. Traverse `list2` and push all values into the same vector.
3. Sort the vector in ascending order.
4. Create a new dummy node. Traverse the sorted vector, creating a new node for each value and appending it to the list.
5. Return the dummy's next node.

### Complexity Analysis
- **Time Complexity:** $O((M+N) \log(M+N))$ where $M$ and $N$ are the lengths of `list1` and `list2` respectively, due to the sorting step.
- **Space Complexity:** $O(M+N)$ to store values in the vector and to allocate the new list nodes.

### Pros/Cons
- **Pros:** Conceptually simple; doesn't require modifying existing pointer links.
- **Cons:** High space complexity; does not modify the lists in-place; slow due to sorting.

### C++17 Brute Force Code
```cpp
#include <vector>
#include <algorithm>

struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class SolutionBrute {
public:
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        std::vector<int> values;
        
        // Step 1: Collect values
        ListNode* curr = list1;
        while (curr) {
            values.push_back(curr->val);
            curr = curr->next;
        }
        curr = list2;
        while (curr) {
            values.push_back(curr->val);
            curr = curr->next;
        }
        
        // Step 2: Sort
        std::sort(values.begin(), values.end());
        
        // Step 3: Rebuild
        ListNode dummy(0);
        curr = &dummy;
        for (int val : values) {
            curr->next = new ListNode(val);
            curr = curr->next;
        }
        
        return dummy.next;
    }
};
```

---

## 6. Better Solution
> [!NOTE]
> The brute force collects values and sorts them. The in-place two-pointer approaches (Iterative and Recursive) are optimal. We jump directly to them.

---

## 7. Optimal Solution

### Approach 1: Iterative (In-place with Dummy Node)
We maintain a dummy node to build the list, and comparison pointers for `list1` and `list2`.

#### Algorithm
1. Create a `dummy` node. Initialize `tail = &dummy`.
2. While `list1 != nullptr` and `list2 != nullptr`:
   - Compare `list1->val` and `list2->val`.
   - If `list1->val <= list2->val`:
     - Link `tail->next = list1`.
     - Advance `list1 = list1->next`.
   - Else:
     - Link `tail->next = list2`.
     - Advance `list2 = list2->next`.
   - Move `tail` forward: `tail = tail->next`.
3. If one list is exhausted, append the other list: `tail->next = (list1 != nullptr) ? list1 : list2`.
4. Return `dummy.next`.

#### Iterative C++17 Code
```cpp
class SolutionIterative {
public:
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        ListNode dummy(0); // Stack-allocated dummy node
        ListNode* tail = &dummy;
        
        while (list1 != nullptr && list2 != nullptr) {
            if (list1->val <= list2->val) {
                tail->next = list1;
                list1 = list1->next;
            } else {
                tail->next = list2;
                list2 = list2->next;
            }
            tail = tail->next;
        }
        
        // Link the remaining nodes of the non-empty list
        tail->next = (list1 != nullptr) ? list1 : list2;
        
        return dummy.next;
    }
};
```

### Approach 2: Recursive (In-place)
Recursively merge the lists.

#### Algorithm
1. Base cases: If `list1` is null, return `list2`. If `list2` is null, return `list1`.
2. Compare head values:
   - If `list1->val <= list2->val`:
     - `list1->next = mergeTwoLists(list1->next, list2)`.
     - Return `list1`.
   - Else:
     - `list2->next = mergeTwoLists(list1, list2->next)`.
     - Return `list2`.

#### Recursive C++17 Code
```cpp
class SolutionRecursive {
public:
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        // Base Cases
        if (list1 == nullptr) return list2;
        if (list2 == nullptr) return list1;
        
        // Compare heads and merge recursively
        if (list1->val <= list2->val) {
            list1->next = mergeTwoLists(list1->next, list2);
            return list1;
        } else {
            list2->next = mergeTwoLists(list1, list2->next);
            return list2;
        }
    }
};
```

---

## 8. Code Walkthrough (Iterative)
- **`ListNode dummy(0); ListNode* tail = &dummy;`**: Initialize a temporary dummy node. `tail` is used to append nodes during traversal.
- **`while (list1 != nullptr && list2 != nullptr)`**: Iterate while both lists have nodes remaining.
- **`if (list1->val <= list2->val) { tail->next = list1; list1 = list1->next; }`**: If the value in `list1` is smaller or equal, append `list1` to `tail->next`, and advance `list1` by one step.
- **`tail = tail->next;`**: Advance the `tail` pointer.
- **`tail->next = (list1 != nullptr) ? list1 : list2;`**: Once the loop terminates, at least one list is empty. We link the remaining elements of the non-empty list directly.
- **`return dummy.next;`**: Return the node after `dummy`, which is the head of the merged list.

---

## 9. Dry Run (Iterative)
Input: `list1 = [1, 3, 5]`, `list2 = [2, 4, 6]`

- **Init:** `dummy(0)`, `tail = dummy`, `list1 = 1`, `list2 = 2`

| Step | Comparison | Action | `tail->next` updated to | Pointers Advanced |
| :--- | :--- | :--- | :--- | :--- |
| **1** | `1 <= 2` (True) | Append `list1` | Node `1` | `list1` to `3`, `tail` to `1` |
| **2** | `3 <= 2` (False) | Append `list2` | Node `2` | `list2` to `4`, `tail` to `2` |
| **3** | `3 <= 4` (True) | Append `list1` | Node `3` | `list1` to `5`, `tail` to `3` |
| **4** | `5 <= 4` (False) | Append `list2` | Node `4` | `list2` to `6`, `tail` to `4` |
| **5** | `5 <= 6` (True) | Append `list1` | Node `5` | `list1` to `nullptr`, `tail` to `5` |
| **Loop Exit** | `list1 == nullptr` | - | - | - |
| **Final Link**| - | Append `list2` | Node `6` (rest of `list2`)| `tail->next` set to Node `6` |

**Returned list:** `1 -> 2 -> 3 -> 4 -> 5 -> 6 -> nullptr`.

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Iterative** | $O(M+N)$ | $O(1)$ | Single pass over both lists. Spliced in-place; uses only dummy and tail pointers. |
| **Recursive** | $O(M+N)$ | $O(M+N)$ | Visits each node once. The recursion call stack size is $O(M+N)$ frames. |

---

## 11. Common Mistakes
- **Forgetting to append the remaining list:** Leaving out the step `tail->next = list1 ? list1 : list2` results in losing the tail end of the longer list.
- **Allocating new nodes:** Creating a new node for each element instead of splicing existing ones. This increases space complexity to $O(M+N)$.
- **Null dereference in loop check:** Writing `list1->val` or `list2->val` without verifying that `list1 != nullptr` and `list2 != nullptr`.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it matters |
| :--- | :--- | :--- | :--- |
| **One empty list** | `list1 = []`, `list2 = [1, 2]` | `[1, 2]` | Loop doesn't execute; final link handles appending the rest. |
| **Both empty lists** | `list1 = []`, `list2 = []` | `[]` (`nullptr`) | Dummy's next remains `nullptr`, returns correctly. |
| **No overlap in values**| `[1, 2]`, `[3, 4]` | `[1, 2, 3, 4]` | One list is completely traversed first, then the other is appended. |

---

## 13. Interview Explanation
> "To merge two sorted linked lists in-place, I use an iterative two-pointer approach with a dummy node. The dummy node simplifies head node assignment by giving us a starting point. We maintain a `tail` pointer starting at `dummy`. We then compare the current nodes of both lists. The smaller node is attached to `tail->next`, and we advance the corresponding list pointer and the `tail` pointer. We repeat this process until one list is exhausted. Once the loop ends, we append the remainder of the non-empty list to `tail->next`. Finally, we return `dummy.next` as the head of the merged list. This runs in $O(M+N)$ time and uses $O(1)$ auxiliary space. A recursive approach is also possible with the same time complexity but requires $O(M+N)$ space due to the call stack."

---

## 14. Follow-up Questions
1. **How would you merge $K$ sorted lists?**
   - *We can use a divide-and-conquer strategy, pairing up and merging lists. Alternatively, we can use a min-heap (priority queue) storing the head of each list. Both methods take $O(N \log K)$ time.*
2. **How does this relate to Merge Sort on linked lists?**
   - *This merge routine is the combine step of Merge Sort. Once a list is split into halves, we recursively sort both halves and use this function to merge them back.*

---

## 15. Variations
- **Merge $K$ Sorted Lists (LeetCode 23):** Merging multiple sorted linked lists.
- **Merge Sorted Array (LeetCode 88):** Merging two arrays in-place from the back.

---

## 16. Revision Notes
- **Key trick:** Use a stack-allocated dummy node to avoid checking `if (mergedHead == nullptr)`.
- **Iterative step:** Compare values, redirect link, advance pointer.
- **Cleanup step:** `tail->next = list1 ? list1 : list2`

---

## 17. Memorization Version (< 1 minute read)
- **Goal:** Merge 2 sorted lists in-place, $O(1)$ space.
- **Strategy:**
  1. Create `dummy` node and `tail = &dummy`.
  2. While both lists are not null, link `tail->next` to the node with the smaller value, advance that list's pointer and `tail`.
  3. After loop, set `tail->next` to the remaining non-empty list.
  4. Return `dummy.next`.
- **Complexity:** $O(M+N)$ Time, $O(1)$ Space.
