# Find Kth Smallest / Largest Element

## 1. Problem Statement

Given an integer array `nums` and an integer `k`, return the $k$-th largest element in the array.

Note that it is the $k$-th largest element in the sorted order, not the $k$-th distinct element.

*(Note: Finding the $k$-th smallest element is the symmetric problem: the $k$-th smallest element is equivalent to the $(n - k + 1)$-th largest element.)*

### Input
* `nums`: A vector of integers where $1 \le k \le \text{nums.length} \le 10^5$.
* `k`: An integer representing the target order.

### Output
* Returns the $k$-th largest integer.

### Constraints
* $-10^4 \le \text{nums}[i] \le 10^4$

---

## 2. Interview Clarification

Before writing any code, clarify these points with your interviewer:
1. **Should the solution handle duplicates?**
   * *Answer*: Yes, we want the $k$-th largest element in sorted order, meaning duplicates are counted. For example, in `[3, 2, 3, 1, 2, 4, 5, 5, 6]` and $k = 4$, the sorted order is `[1, 2, 2, 3, 3, 4, 5, 5, 6]`. The 4th largest is `4`.
2. **Can we modify the input array?**
   * *Answer*: Yes, in-place modification is generally allowed. If it isn't, we can make a copy, which increases space complexity to $O(n)$.
3. **What is the expected complexity?**
   * *Answer*: You should target average $O(n)$ time complexity.

---

## 3. Thinking Process

### First Instinct (Brute Force)
* Sort the array in descending order and return the element at index `k - 1` (or sort in ascending order and return `nums[n - k]`).
* Sorting takes $O(n \log n)$ time.

### Better Instinct (Heaps - Priority Queue)
* We want to find the $k$-th largest element. We can maintain a **Min-Heap** of size $K$.
* For each number `num` in the array:
  * Push `num` into the min-heap.
  * If the size of the min-heap exceeds $K$, pop the smallest element (which is at the top).
* After iterating through all elements, the min-heap will contain the $K$ largest elements of the array. The top of the min-heap will be the smallest of these $K$ largest elements, which is the $K$-th largest element.
* This takes $O(n \log k)$ time and $O(k)$ auxiliary space.

### Optimal Insight (QuickSelect Algorithm)
* QuickSelect is a selection algorithm based on the QuickSort partitioning step.
* **Concept**:
  * Pick a pivot. Partition the array around the pivot such that all elements larger than the pivot are to its left (or right, depending on implementation), and smaller elements are to the other side.
  * After partitioning, the pivot will be at its final sorted position, say index `p`.
  * If `p == k - 1` (for 0-indexed largest element), we found our element!
  * If `p > k - 1`, the $k$-th largest element lies in the left subarray. Recursively partition the left side.
  * If `p < k - 1`, the $k$-th largest element lies in the right subarray. Recursively partition the right side.
* **Time Complexity**:
  * On average, each step halves the search space: $n + n/2 + n/4 + \dots = O(n)$ time.
  * In the worst case (e.g., already sorted array and bad pivot choice), it reduces by only 1 element at a time: $O(n^2)$ time.
  * We can use a random pivot to avoid the $O(n^2)$ worst-case scenario.

---

## 4. Pattern Recognition

* **Topic**: Sorting and Selection.
* **Pattern**: Heap (for streaming / constrained space) or QuickSelect (for average linear time).
* **Why this works**: Partitioning allows us to discard half of the array at each step without sorting the unneeded half, achieving average linear time.

---

## 5. Brute Force Solution

### Algorithm
1. Sort the vector `nums` using `std::sort`.
2. Return `nums[nums.size() - k]`.

### Complexity Analysis
* **Time Complexity**: $O(n \log n)$
* **Space Complexity**: $O(1)$ auxiliary space (if sorting in-place).

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class SolutionBrute {
public:
    int findKthLargest(std::vector<int>& nums, int k) {
        std::sort(nums.begin(), nums.end());
        return nums[nums.size() - k];
    }
};
```

---

## 6. Better Solution

### Algorithm (Min-Heap)
1. Initialize a min-heap (priority queue with greater comparator).
2. For each element `num` in `nums`:
   * Push `num` into the min-heap.
   * If heap size is $> k$, pop the top element.
3. Return the top of the heap.

### Complexity Analysis
* **Time Complexity**: $O(n \log k)$ because insertion/deletion in a heap of size $k$ takes $O(\log k)$ time.
* **Space Complexity**: $O(k)$ to store $k$ elements in the heap.

### C++17 Code
```cpp
#include <vector>
#include <queue>

class SolutionBetter {
public:
    int findKthLargest(const std::vector<int>& nums, int k) {
        // Min-heap
        std::priority_queue<int, std::vector<int>, std::greater<int>> min_heap;
        
        for (int num : nums) {
            min_heap.push(num);
            if (min_heap.size() > k) {
                min_heap.pop();
            }
        }
        
        return min_heap.top();
    }
};
```

---

## 7. Optimal Solution

### Algorithm (QuickSelect with Random Pivot)
1. We want to find the element at index `target = n - k` in the sorted array (which is the $k$-th largest).
2. Define a function `quickSelect(left, right, target)`:
   * Choose a random pivot index between `left` and `right`. Swap it with `nums[right]`.
   * Partition the array from `left` to `right` using Lomuto's partition scheme:
     * Iterate `i` from `left` to `right - 1`. If `nums[i] <= pivot`, swap `nums[i]` with `nums[store_index]` and increment `store_index`.
     * Swap `nums[right]` with `nums[store_index]`.
   * Now the pivot is at index `store_index`.
   * If `store_index == target`, return `nums[store_index]`.
   * If `store_index > target`, recursively call `quickSelect(left, store_index - 1, target)`.
   * If `store_index < target`, recursively call `quickSelect(store_index + 1, right, target)`.

### Why this is Optimal
* On average, we do not partition both sides of the array (unlike QuickSort). This reduces the recurrence relation to $T(n) = T(n/2) + O(n)$, which solves to $O(n)$ by Master's Theorem.
* Space complexity is $O(1)$ auxiliary (ignoring recursion stack space which is $O(\log n)$ on average).

### C++17 Code
```cpp
#include <vector>
#include <algorithm>
#include <cstdlib>
#include <ctime>

class SolutionOptimal {
private:
    int partition(std::vector<int>& nums, int left, int right) {
        // Choose a random pivot to avoid O(n^2) worst case on sorted inputs
        int pivot_index = left + std::rand() % (right - left + 1);
        std::swap(nums[pivot_index], nums[right]);
        
        int pivot = nums[right];
        int store_index = left;
        
        for (int i = left; i < right; ++i) {
            if (nums[i] < pivot) {
                std::swap(nums[i], nums[store_index]);
                store_index++;
            }
        }
        std::swap(nums[store_index], nums[right]);
        return store_index;
    }

    int quickSelect(std::vector<int>& nums, int left, int right, int target) {
        if (left == right) {
            return nums[left];
        }
        
        int pivot_index = partition(nums, left, right);
        
        if (pivot_index == target) {
            return nums[pivot_index];
        } 
        else if (pivot_index < target) {
            return quickSelect(nums, pivot_index + 1, right, target);
        } 
        else {
            return quickSelect(nums, left, pivot_index - 1, target);
        }
    }

public:
    int findKthLargest(std::vector<int>& nums, int k) {
        std::srand(std::time(nullptr)); // Seed random number generator
        int n = nums.size();
        return quickSelect(nums, 0, n - 1, n - k);
    }
};
```

---

## 8. Code Walkthrough

* **Line 33**: `findKthLargest` calculates the target index as `n - k`. For instance, in sorted ascending array of size 6, the 2nd largest is at index $6 - 2 = 4$.
* **Line 20**: `quickSelect` takes boundaries `left`, `right`, and the target index.
* **Line 7**: `partition` chooses a random pivot inside the range `[left, right]`, swaps it with the element at `right`, and does standard partition.
* **Line 14**: `if (nums[i] < pivot)`
  We move all elements smaller than pivot to the left side of `store_index`.
* **Line 26**: `if (pivot_index == target)`
  If the pivot falls exactly on the target index, we have found our $k$-th largest element and return it immediately.
* **Line 29-31**: If `pivot_index` is smaller than `target`, we search in the right partition. Else, we search in the left partition.

---

## 9. Dry Run

Input: `nums = [3, 2, 1, 5, 6, 4]`, `k = 2`.
`n = 6`, `target = 6 - 2 = 4`.
We search for the element that will end up at index `4` when sorted.

Let's assume our random choices result in the following steps:
1. **Initial Call**: `quickSelect(nums, 0, 5, 4)`
   * Pivot chosen: `nums[5] = 4` (value = 4).
   * Partitioning:
     * Elements smaller than 4: `3, 2, 1`.
     * Elements larger than 4: `5, 6`.
     * Array becomes: `[3, 2, 1, 4, 6, 5]`.
   * Pivot index `4` is returned (`nums[4] = 6`, wait: let's do partition steps precisely):
     * `nums = [3, 2, 1, 5, 6, 4]`, pivot = 4.
     * `i = 0`: 3 < 4, swap `nums[0]` with `nums[0]`. `store_index = 1`.
     * `i = 1`: 2 < 4, swap `nums[1]` with `nums[1]`. `store_index = 2`.
     * `i = 2`: 1 < 4, swap `nums[2]` with `nums[2]`. `store_index = 3`.
     * `i = 3`: 5 not < 4.
     * `i = 4`: 6 not < 4.
     * End loop. Swap `nums[store_index]` (5) with `nums[right]` (4).
     * Array: `[3, 2, 1, 4, 6, 5]`. `store_index` returned is `3`.
2. **Evaluate pivot index**:
   * Returned `pivot_index = 3`.
   * Since `3 < target (4)`, call `quickSelect(nums, 4, 5, 4)`.
3. **Next Call**: `quickSelect(nums, 4, 5, 4)`
   * Range: `[4, 5]` (`nums[4] = 6`, `nums[5] = 5`).
   * Assume pivot chosen: `5` (swapped to right position `nums[5]`).
   * Partitioning:
     * `i = 4`: 6 not < 5.
     * Swap `nums[store_index]` (6) with `nums[right]` (5).
     * Array: `[3, 2, 1, 4, 5, 6]`. `store_index` returned is `4`.
4. **Evaluate pivot index**:
   * Returned `pivot_index = 4`.
   * Since `4 == target`, return `nums[4] = 5`.

Result: `5` (Correct!)

---

## 10. Complexity Analysis

| Approach | Time Complexity (Best/Avg) | Time (Worst Case) | Space Complexity | Notes |
| :--- | :--- | :--- | :--- | :--- |
| Brute Force (Sort) | $O(n \log n)$ | $O(n \log n)$ | $O(1)$ | Simple, modifies array |
| Heap | $O(n \log k)$ | $O(n \log k)$ | $O(k)$ | Good for stream inputs |
| **QuickSelect** | **$O(n)$** | **$O(n^2)$** | **$O(1)$ auxiliary** | **Linear time on average** |

*Note: Worst-case $O(n^2)$ time in QuickSelect happens when the pivot chosen is always the minimum or maximum element. Randomizing the pivot makes this case highly improbable ($O(2^{-n})$ probability).*

---

## 11. Common Mistakes

1. **Calculating target index incorrectly**:
   * For the $k$-th largest element, the target index in ascending order is `n - k`. If looking for $k$-th smallest, the target index is `k - 1`.
2. **Not randomizing the pivot**:
   * If the input is already sorted, choosing the first or last element as pivot will result in $O(n^2)$ time complexity. This often leads to a Time Limit Exceeded (TLE) error in interviews.
3. **Modifying `mid` inside recursion range incorrectly**:
   * When calling recursion on left/right partitions, exclude the pivot itself: `quickSelect(nums, pivot_index + 1, right, target)` or `quickSelect(nums, left, pivot_index - 1, target)`.

---

## 12. Edge Cases

| Edge Case | Example | Why it matters | Solution Behavior |
| :--- | :--- | :--- | :--- |
| $K = 1$ (Find Max) | `[2, 1, 5, 3]`, $k=1$ | Boundary condition for $K$. | Returns `5` (Correct) |
| $K = N$ (Find Min) | `[2, 1, 5, 3]`, $k=4$ | Boundary condition for $K$. | Returns `1` (Correct) |
| Array size 1 | `[10]`, $k=1$ | Minimum array size. | Returns `10` (Correct) |
| Duplicate elements | `[2, 2, 2, 2]`, $k=2$ | Duplicate values must be sorted correctly. | Returns `2` (Correct) |

---

## 13. Interview Explanation

"To find the $k$-th largest element in an array, we have three main approaches. The brute force way is to sort the array and return the element at index `n - k`, which takes $O(n \log n)$ time. 

A better approach, especially if the array is very large or elements are streamed, is to use a Min-Heap of size $k$. We push elements onto the heap and pop when the size exceeds $k$. At the end, the top of the heap is the $k$-th largest element. This takes $O(n \log k)$ time and $O(k)$ space.

The optimal approach is to use the QuickSelect algorithm, which is based on QuickSort partitioning. Instead of sorting both halves, we partition the array around a random pivot. If the pivot lands on the target index `n - k`, we return it. Otherwise, we recursively partition only the half that must contain the target. This runs in $O(n)$ average time complexity and requires $O(1)$ extra space."

---

## 14. Follow-up Questions

1. **How do you guarantee $O(n)$ worst-case time complexity?**
   * *Answer*: You can use the **Median of Medians** algorithm (also known as the BFPRT algorithm) to choose the pivot. It divides the array into groups of 5, finds the median of each group, and then recursively finds the median of those medians to use as the pivot. This guarantees $O(n)$ worst-case time, but has a high constant factor.
2. **What if the input array is too large to fit in memory?**
   * *Answer*: If the data is streamed, we must use the Heap approach. We keep a min-heap of size $k$ in memory, which uses only $O(k)$ memory space.

---

## 15. Variations

1. **K-th Smallest Element**: Same logic, but target index is `k - 1` instead of `n - k`.
2. **K Closest Points to Origin**: Uses heap or QuickSelect to find the $K$ points with smallest Euclidean distance.

---

## 16. Revision Notes

* **QuickSelect Recurrence**:
  * Average: $T(n) = T(n/2) + O(n) \implies O(n)$
  * Worst: $T(n) = T(n-1) + O(n) \implies O(n^2)$
* **Heap vs QuickSelect**:
  * Use Heap when data is streamed ($O(k)$ memory).
  * Use QuickSelect when data is fully in memory and $O(n)$ time is preferred.

---

## 17. Memorization Version

```cpp
// O(n) Average Time, O(1) Space. QuickSelect selection.
int partition(vector<int>& nums, int l, int r) {
    int pIdx = l + rand() % (r - l + 1);
    swap(nums[pIdx], nums[r]);
    int pivot = nums[r], idx = l;
    for (int i = l; i < r; ++i) {
        if (nums[i] < pivot) swap(nums[i], nums[idx++]);
    }
    swap(nums[idx], nums[r]);
    return idx;
}
int quickSelect(vector<int>& nums, int l, int r, int target) {
    if (l == r) return nums[l];
    int pIdx = partition(nums, l, r);
    if (pIdx == target) return nums[pIdx];
    return pIdx < target ? quickSelect(nums, pIdx + 1, r, target) : quickSelect(nums, l, pIdx - 1, target);
}
int findKthLargest(vector<int>& nums, int k) {
    return quickSelect(nums, 0, nums.size() - 1, nums.size() - k);
}
```
