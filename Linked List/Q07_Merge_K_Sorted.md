# Merge K Sorted Linked Lists

## 1. Problem Statement

Given an array of `K` linked lists, where each linked list is sorted in ascending order, merge all the linked lists into one sorted linked list and return it.

### Input
- `lists`: A vector of head pointers to `K` sorted singly linked lists.

### Output
- A single head pointer to the merged sorted linked list containing all the nodes from the input lists.

### Constraints
- $K = \text{lists.length}$
- $0 \le K \le 10^4$
- $0 \le \text{Length of each list} \le 500$
- $-10^4 \le \text{Node.val} \le 10^4$
- The cumulative number of nodes in all lists does not exceed $5 \times 10^5$.
- Each individual list is sorted in strictly non-decreasing order.

### Observations
1. If the input array of lists is empty, we must return `nullptr`.
2. If there's only one list in the array, we can return it directly.
3. The nodes themselves should be rearranged; we should not allocate new nodes to replicate the values unless specified otherwise. We should rearrange the pointers in-place.
4. Since the individual lists are already sorted, we can leverage this order. We do not need to sort the nodes from scratch.

---

## 2. Interview Clarification

Before starting, a candidate should clarify the following with the interviewer:
1. **Are the linked lists singly or doubly linked?**
   * *Answer*: Singly linked lists.
2. **Should we reuse the existing nodes or allocate new nodes for the merged list?**
   * *Answer*: Merge the lists in-place by changing the `next` pointers. Do not allocate new nodes.
3. **Can the array contain empty lists (`nullptr`)?**
   * *Answer*: Yes, some list heads in the array can be `nullptr`. Your code must handle them gracefully.
4. **Is it possible that the input array itself is empty?**
   * *Answer*: Yes, `lists` can have a size of 0.
5. **How should we handle duplicate values across different lists?**
   * *Answer*: Maintain them in any order, as long as the final list is sorted.

---

## 3. Thinking Process

Let's think through how to solve this:
* **First Instinct**: Put all node values from all lists into a single dynamic array, sort the array using standard sorting algorithms like $O(M \log M)$ (where $M$ is the total number of nodes), and then construct a new linked list from it.
  * *Critique*: While correct, this does not utilize the fact that each individual list is already sorted, and it uses $O(M)$ extra space to store the values.
* **Observations**:
  * We already know how to merge two sorted lists in $O(N_1 + N_2)$ time and $O(1)$ space using a dummy node and a two-pointer technique.
  * Can we build on this? Yes, we can merge the first list with the second, then merge the result with the third, and so on.
  * Let's analyze this "pairwise merge" approach. Suppose we have $K$ lists, each of length $N$. The first merge takes $N + N = 2N$ operations. The second merge takes $2N + N = 3N$ operations. The $i$-th merge takes $iN + N = (i+1)N$ operations.
  * Summing this up: $2N + 3N + \dots + KN \approx N \cdot \frac{K^2}{2} = O(N \cdot K^2)$ time. This is slow if $K$ is large.
* **Heap-based approach**:
  * Instead of looking at entire lists, at any point during the merge, we only need to compare the smallest remaining elements of each list.
  * At the start, the smallest elements are the heads of the $K$ lists.
  * If we put the heads of all non-empty lists into a Min-Heap (priority queue), the top of the heap will give us the absolute smallest node.
  * We pop this node, append it to our merged list, and then insert its next node (if it exists) into the heap.
  * Since the heap size is at most $K$, each insertion/deletion takes $O(\log K)$ time. With $M$ total nodes, the time complexity becomes $O(M \log K)$, which is significantly faster. Space complexity is $O(K)$ for the heap.
* **Divide and Conquer approach**:
  * Can we do better on space? What if we merge the lists in pairs, just like in Merge Sort?
  * Merge lists `0` and `1`, `2` and `3`, `4` and `5`, and so on.
  * After the first pass, we reduce the number of lists from $K$ to $K/2$.
  * In the next pass, we merge the resulting lists in pairs, reducing them to $K/4$, then $K/8$, and so on, until only 1 list remains.
  * The height of this merge tree is $O(\log K)$. At each level, the total number of nodes we process is $M$ (all nodes).
  * Thus, the total time complexity is $O(M \log K)$ and if we do it iteratively, we do not even need recursion stack space, reducing extra space complexity to $O(1)$.

---

## 4. Pattern Recognition

This problem fits several classic patterns:
1. **Merge/Two-Pointer Pattern**: Merging two sorted lists is the foundation of this problem.
2. **K-Way Merge Pattern**: When merging $K$ sorted inputs (arrays, lists, streams), we can use a Min-Heap to track the minimum element of each list.
3. **Divide and Conquer Pattern**: Splitting the problem into independent halves, solving them, and combining the results. This is highly effective in reducing the depth of operations from linear $O(K)$ to logarithmic $O(\log K)$.

---

## 5. Brute Force Solution (Pairwise Merge)

### Algorithm
1. If `lists` is empty, return `nullptr`.
2. Initialize `mergedHead` as `lists[0]`.
3. Iterate from `i = 1` to `K - 1`:
   * Merge `mergedHead` with `lists[i]` using the standard two-pointer list merging algorithm.
   * Update `mergedHead` with the result.
4. Return `mergedHead`.

### Dry Run
Let `lists = [[1, 4], [2, 5], [3, 6]]`.
* Initialize `mergedHead = [1, 4]`.
* `i = 1`: Merge `[1, 4]` and `[2, 5]`.
  * Comparing heads: `1 < 2` $\rightarrow$ list starts with `1`.
  * Next compare `4` and `2` $\rightarrow$ next is `2`.
  * Next compare `4` and `5` $\rightarrow$ next is `4`.
  * Remainder is `5`.
  * Merged: `[1, 2, 4, 5]`. Update `mergedHead = [1, 2, 4, 5]`.
* `i = 2`: Merge `[1, 2, 4, 5]` and `[3, 6]`.
  * Comparing elements: `1`, then `2`, then `3`, then `4`, then `5`, then `6`.
  * Merged: `[1, 2, 3, 4, 5, 6]`. Update `mergedHead = [1, 2, 3, 4, 5, 6]`.
* End loop. Return `[1, 2, 3, 4, 5, 6]`.

### Complexity Analysis
* **Time Complexity**: $O(N \cdot K^2)$ where $N$ is the average length of a list and $K$ is the number of lists.
  * Specifically, the total number of operations is $\sum_{i=1}^{K-1} O(i \cdot N) = O(N \cdot \frac{K(K-1)}{2}) = O(N \cdot K^2)$.
* **Space Complexity**: $O(1)$ auxiliary space if the merge is done in-place.

### Pros and Cons
* **Pros**: Simple to implement; uses $O(1)$ extra space.
* **Cons**: Extremely slow when $K$ is large ($10^4$). TLE (Time Limit Exceeded) on typical online judges.

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

class BruteMergeKSorted {
private:
    // Helper function to merge two sorted lists in-place
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        ListNode dummy(0);
        ListNode* tail = &dummy;

        while (l1 != nullptr && l2 != nullptr) {
            if (l1->val <= l2->val) {
                tail->next = l1;
                l1 = l1->next;
            } else {
                tail->next = l2;
                l2 = l2->next;
            }
            tail = tail->next;
        }

        // Attach the remaining non-empty list
        if (l1 != nullptr) {
            tail->next = l1;
        } else {
            tail->next = l2;
        }

        return dummy.next;
    }

public:
    ListNode* mergeKLists(std::vector<ListNode*>& lists) {
        if (lists.empty()) {
            return nullptr;
        }

        ListNode* mergedHead = lists[0];
        for (size_t i = 1; i < lists.size(); ++i) {
            mergedHead = mergeTwoLists(mergedHead, lists[i]);
        }

        return mergedHead;
    }
};
```

---

## 6. Better Solution (Min-Heap / Priority Queue)

### Improvement over Brute Force
Instead of merging sequentially, we keep track of the smallest node among the heads of all $K$ lists at once. Using a Min-Heap reduces the time to choose the minimum element from $O(K)$ to $O(\log K)$.

### Algorithm
1. Create a Min-Heap (priority queue) that compares `ListNode*` based on their values.
2. Push the head of each non-empty list into the heap.
3. Create a `dummy` node to act as the head of the merged list, and a `tail` pointer to track the end.
4. While the heap is not empty:
   * Pop the node with the smallest value from the heap.
   * Append this node to `tail->next`.
   * Move `tail` to `tail->next`.
   * If the popped node has a next node (`node->next`), push `node->next` into the heap.
5. Return `dummy.next`.

### Dry Run
Let `lists = [[1, 4], [2, 5], [3, 6]]`.
* Initialize Heap: Push heads `1`, `2`, `3` into the heap. Heap: `[1 (L0), 2 (L1), 3 (L2)]`.
* **Step 1**: Pop `1 (L0)`. Append to merged list: `1`.
  * `1` has a next node `4`. Push `4` to heap. Heap: `[2 (L1), 3 (L2), 4 (L0)]`.
* **Step 2**: Pop `2 (L1)`. Append: `1 -> 2`.
  * `2` has a next node `5`. Push `5` to heap. Heap: `[3 (L2), 4 (L0), 5 (L1)]`.
* **Step 3**: Pop `3 (L2)`. Append: `1 -> 2 -> 3`.
  * `3` has a next node `6`. Push `6` to heap. Heap: `[4 (L0), 5 (L1), 6 (L2)]`.
* **Step 4**: Pop `4 (L0)`. Append: `1 -> 2 -> 3 -> 4`.
  * `4->next` is `nullptr`. Heap: `[5 (L1), 6 (L2)]`.
* **Step 5**: Pop `5 (L1)`. Append: `1 -> 2 -> 3 -> 4 -> 5`.
  * Heap: `[6 (L2)]`.
* **Step 6**: Pop `6 (L2)`. Append: `1 -> 2 -> 3 -> 4 -> 5 -> 6`.
  * Heap becomes empty.
* Return `dummy.next` which is `1 -> 2 -> 3 -> 4 -> 5 -> 6`.

### Complexity Analysis
* **Time Complexity**: $O(M \log K)$ where $M$ is the total number of nodes in all $K$ lists.
  * Every node is pushed and popped from the heap exactly once. The heap size is bounded by $O(K)$.
* **Space Complexity**: $O(K)$ for the priority queue to store the heads of the $K$ lists.

### Commented C++17 Code
```cpp
#include <vector>
#include <queue>

class HeapMergeKSorted {
private:
    // Custom comparator for the min-heap
    struct CompareNodes {
        bool operator()(const ListNode* lhs, const ListNode* rhs) const {
            return lhs->val > rhs->val; // Min-heap behavior
        }
    };

public:
    ListNode* mergeKLists(std::vector<ListNode*>& lists) {
        // Min-heap storing ListNode pointers
        std::priority_queue<ListNode*, std::vector<ListNode*>, CompareNodes> minHeap;

        // Push the heads of all non-empty lists
        for (ListNode* head : lists) {
            if (head != nullptr) {
                minHeap.push(head);
            }
        }

        ListNode dummy(0);
        ListNode* tail = &dummy;

        // Extract nodes from the min-heap and insert their successors
        while (!minHeap.empty()) {
            ListNode* minNode = minHeap.top();
            minHeap.pop();

            tail->next = minNode;
            tail = tail->next;

            // If there's a next node in the current list, push it to the heap
            if (minNode->next != nullptr) {
                minHeap.push(minNode->next);
            }
        }

        return dummy.next;
    }
};
```

---

## 7. Optimal Solution (Divide and Conquer / Merge with O(1) Space)

### Best Approach
Divide and Conquer processes the array of lists by recursively or iteratively splitting them into halves, merging each half, and combining the results. It matches the $O(M \log K)$ time of the heap solution, but achieves $O(1)$ auxiliary space if done iteratively.

### Algorithm (Iterative Divide and Conquer)
1. If the list array is empty, return `nullptr`.
2. Let $K$ be the number of lists.
3. While $K > 1$:
   * Loop through the lists in pairs: merge list `i` and list `i + interval` and store the result in list `i`.
   * The `interval` starts at `1` and doubles at each step (`interval *= 2`).
   * Each pass reduces the number of active lists to merge by half.
4. Return `lists[0]`.

### Why it is Optimal
* **Time**: It achieves the lower bound of $O(M \log K)$ time since we perform $\log K$ merge phases, and in each phase, we process all $M$ nodes.
* **Space**: By doing this iteratively in-place, we eliminate the call-stack of recursion, leading to $O(1)$ auxiliary space. This is strictly better than the priority queue solution which requires $O(K)$ extra space.

### C++17 Code
```cpp
#include <vector>

class OptimalMergeKSorted {
private:
    // Helper function to merge two sorted lists in-place
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        ListNode dummy(0);
        ListNode* tail = &dummy;

        while (l1 != nullptr && l2 != nullptr) {
            if (l1->val <= l2->val) {
                tail->next = l1;
                l1 = l1->next;
            } else {
                tail->next = l2;
                l2 = l2->next;
            }
            tail = tail->next;
        }

        tail->next = (l1 != nullptr) ? l1 : l2;
        return dummy.next;
    }

public:
    ListNode* mergeKLists(std::vector<ListNode*>& lists) {
        if (lists.empty()) {
            return nullptr;
        }

        int size = lists.size();
        int interval = 1;

        // Perform merge pairwise iteratively
        while (interval < size) {
            for (int i = 0; i + interval < size; i += interval * 2) {
                lists[i] = mergeTwoLists(lists[i], lists[i + interval]);
            }
            interval *= 2;
        }

        return lists[0];
    }
};
```

---

## 8. Code Walkthrough

* **`mergeTwoLists(ListNode* l1, ListNode* l2)`**:
  * Utilizes a `dummy` node to simplify handling the head of the merged list.
  * A `tail` pointer tracks the last node of the merged chain.
  * In the `while` loop, it compares elements from `l1` and `l2`, linking the smaller node to `tail->next`, and advances that pointer.
  * Once one list is exhausted, it links the remainder of the other list directly to `tail->next`.
* **`mergeKLists(std::vector<ListNode*>& lists)`**:
  * First guards against empty inputs by returning `nullptr`.
  * The outer `while (interval < size)` loops control the grouping depth. `interval` starts at 1, meaning we merge adjacent elements: index `i` and `i + 1`.
  * In the next step, `interval` becomes 2, merging indices `i` and `i + 2` (i.e., merging the newly created lists at indices 0, 4, 8...).
  * The step `i += interval * 2` ensures we skip over the list elements we've already merged into the current base index.
  * Finally, the fully merged list is aggregated at `lists[0]`.

---

## 9. Dry Run

Let's dry run the optimal solution on: `lists = [[1, 4], [2, 5], [3, 6], [7]]` (size = 4).

| Phase | `interval` | `i` | `i + interval` | Action | Resulting `lists` |
|:---:|:---:|:---:|:---:|:---|:---|
| Setup | - | - | - | - | `[[1, 4], [2, 5], [3, 6], [7]]` |
| **Phase 1** | 1 | 0 | 1 | Merge `lists[0]` and `lists[1]` | `lists[0]` becomes `[1, 2, 4, 5]` |
| | 1 | 2 | 3 | Merge `lists[2]` and `lists[3]` | `lists[2]` becomes `[3, 6, 7]` |
| | 2 (doubled)| - | - | Phase ends | `[[1, 2, 4, 5], [2, 5], [3, 6, 7], [7]]` |
| **Phase 2** | 2 | 0 | 2 | Merge `lists[0]` and `lists[2]` | `lists[0]` becomes `[1, 2, 3, 4, 5, 6, 7]` |
| | 4 (doubled)| - | - | Phase ends | Loop terminates as `interval >= size` (4 >= 4) |

**Result**: Return `lists[0]` which contains `[1, 2, 3, 4, 5, 6, 7]`.

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Recommendation |
|:---|:---:|:---:|:---|
| **Brute Force (Sequential)** | $O(N \cdot K^2)$ | $O(1)$ | Bad. Suffers from quadratic performance. |
| **Better (Min-Heap)** | $O(M \log K)$ | $O(K)$ | Good. Great for streaming data scenarios. |
| **Optimal (Divide & Conquer)** | $O(M \log K)$ | $O(1)$ | Best. Highly efficient time & space limits. |

* **Time Complexity**: Every node in the system ($M$ nodes total) is merged exactly $\log_2 K$ times. Since merging a node takes $O(1)$ operations per level, the time complexity is strictly $O(M \log K)$.
* **Space Complexity**: The iterative divide and conquer approach updates node pointers in-place and uses only a few pointers for iteration. Thus, the auxiliary space complexity is $O(1)$.

---

## 11. Common Mistakes

1. **Memory Leak**: Forgetting to handle pointer manipulation correctly and losing track of list heads, causing segments of the list to be orphaned.
2. **Priority Queue Comparison**: When using the Min-Heap approach, candidates often write the comparator backwards (using `<` instead of `>`), which creates a Max-Heap instead of a Min-Heap.
3. **Invalid Index Access**: In iterative divide and conquer, exceeding the array bound `i + interval` when the number of lists is not a power of 2. Make sure to check `i + interval < size`.
4. **Altering Input Nodes**: Trying to copy values to new nodes rather than changing pointers. Interviewers will check if the address of the nodes changes.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it Matters |
|:---|:---|:---|:---|
| **Empty Input Array** | `lists = []` | `nullptr` | Guard clause prevents accessing index 0. |
| **Lists containing null pointers** | `lists = [nullptr, [1, 3], nullptr]` | `[1, 3]` | Code must skip `nullptr` heads during heap initialization or merge step. |
| **Odd number of lists** | `lists = [[1], [2], [3]]` | `[1, 2, 3]` | Verifies the pairwise merge handles unpaired lists at the end of a round. |
| **Lists of varying sizes** | `lists = [[1, 2, 3, 4], [5], []]` | `[1, 2, 3, 4, 5]` | Testing balance and remainder handling of the standard 2-list merge. |

---

## 13. Interview Explanation

"To merge $K$ sorted linked lists, our goal is to build a single sorted list efficiently. A sequential merge where we merge lists one by one takes $O(N \cdot K^2)$ time because the size of the merged list increases progressively.

To optimize, we can use a Min-Heap of size $K$ to constantly pull the smallest element among all current list heads in $O(\log K)$ time. This gives us a time complexity of $O(M \log K)$ with $O(K)$ auxiliary space.

However, the most optimal solution is iterative Divide and Conquer, similar to Merge Sort. We pair up the lists and merge them, reducing the problem size by half at each step. By executing this process iteratively, we achieve the same optimal time complexity of $O(M \log K)$ while saving space, using only $O(1)$ auxiliary space."

---

## 14. Follow-up Questions

1. **What if the lists are extremely large and cannot fit in memory?**
   * *Answer*: Use the Min-Heap approach (External Sort). We only need to keep the head elements of each file stream in memory (size $K$).
2. **Can we parallelize the Divide and Conquer approach?**
   * *Answer*: Yes, since the pairwise merges at any given level are completely independent of each other (e.g., merging list 0 and 1 is independent of merging list 2 and 3), we can run these merges concurrently in separate threads.
3. **What is the comparison between Min-Heap and Divide and Conquer if $K$ is very small?**
   * *Answer*: If $K$ is very small, the overhead of heap creation and pointer updates might make Divide and Conquer slightly faster due to cache locality and smaller constants.

---

## 15. Variations

1. **Merge $K$ Sorted Arrays**:
   * *Difference*: Input is arrays instead of linked lists.
   * *Solution adjustment*: We cannot merge in-place easily without shifting. Min-Heap is usually preferred here because random access is constant, tracking indices of elements in arrays.
2. **Merge Sort on a Single Linked List**:
   * *Difference*: The list is unsorted and single.
   * *Solution*: We find the middle of the list, break it, sort both halves recursively, and merge them. This uses Divide and Conquer as well.

---

## 16. Revision Notes

* **Core Mnemonic**: "Divide the lists, merge in pairs". Think of a tournament bracket.
* **Key Visual**:
  ```
  [L0] [L1] [L2] [L3]
    \   /     \   /
   [L0+1]    [L2+3]
       \      /
      [L0+1+2+3]
  ```
* **Trick**: Always use a dummy node when merging two lists to avoid handling the head separately.
* **Complexity Cheat Sheet**:
  * Pairwise: $O(M \cdot K)$ time, $O(1)$ space.
  * Heap: $O(M \log K)$ time, $O(K)$ space.
  * Divide & Conquer: $O(M \log K)$ time, $O(1)$ space.

---

## 17. Memorization Version

```cpp
// 1-min Review: Iterative Divide & Conquer
// Time: O(M log K), Space: O(1)
ListNode* mergeKLists(vector<ListNode*>& lists) {
    if (lists.empty()) return nullptr;
    int size = lists.size(), interval = 1;
    while (interval < size) {
        for (int i = 0; i + interval < size; i += interval * 2) {
            lists[i] = mergeTwoLists(lists[i], lists[i + interval]);
        }
        interval *= 2;
    }
    return lists[0];
}
// Remember: mergeTwoLists is the standard 2-pointer sorted merge.
```
